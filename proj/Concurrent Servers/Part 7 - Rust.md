This is part 7 in a series of posts on writing concurrent network servers. In this part, we discuss how the challenges described in earlier parts are tackled in the Rust programming language.

All posts in the series:

- [Part 1 - Introduction](https://eli.thegreenplace.net/2017/concurrent-servers-part-1-introduction/)
- [Part 2 - Threads](https://eli.thegreenplace.net/2017/concurrent-servers-part-2-threads/)
- [Part 3 - Event-driven](https://eli.thegreenplace.net/2017/concurrent-servers-part-3-event-driven/)
- [Part 4 - libuv](https://eli.thegreenplace.net/2017/concurrent-servers-part-4-libuv/)
- [Part 5 - Redis case study](https://eli.thegreenplace.net/2017/concurrent-servers-part-5-redis-case-study/)
- [Part 6 - Callbacks, Promises and async/await](https://eli.thegreenplace.net/2018/concurrent-servers-part-6-callbacks-promises-and-asyncawait/)
- Part 7 - Rust (this part)

Several years have passed since the previous parts were published. I've recently went over them to make sure the information presented is still relevant and all the code samples build and run using modern toolchains. I strongly recommend reviewing the previous parts before reading this one.

This post assumes a basic familiarity with the Rust programming language. It will only explain Rust constructs when we encounter code that wouldn't appear in an introductory book or tutorial.

## Setting the baseline - a sequential state machine server

The first few parts in the series focused on a socket server that implements a simple state machine protocol. See [part 1](https://eli.thegreenplace.net/2017/concurrent-servers-part-1-introduction/) for a complete description of the protocol. Let's start by showing how this protocol is implemented in a basic sequential Rust server:

```rust
use async_socket_server::serve_connection;
use std::net::TcpListener;

fn main() -> std::io::Result<()> {
    let port = match std::env::args().nth(1) {
        Some(s) => s,
        None => "9090".to_string(),
    };
    let addr = format!("127.0.0.1:{port}");

    let listener = TcpListener::bind(addr)?;
    println!("Serving on port {port}");

    loop {
        let (stream, addr) = listener.accept()?;
        println!("connection received from {}", addr);
        if let Err(e) = serve_connection(stream) {
            eprintln!("error serving connection: {}", e);
        } else {
            println!("peer done {addr}");
        }
    }
}
```


With the function serve_connection defined as:

```rust
pub enum ProcessingState {
    WaitForMsg,
    InMsg,
}

pub fn serve_connection(mut stream: TcpStream) -> std::io::Result<()> {
    stream.write_all(b"*")?;

    let mut state = ProcessingState::WaitForMsg;
    let mut buf = [0u8; 1024];
    loop {
        let n = stream.read(&mut buf)?;
        if n == 0 {
            // Connection closed by the client.
            break;
        }

        for byte in &buf[..n] {
            match state {
                ProcessingState::WaitForMsg => {
                    if *byte == b'^' {
                        state = ProcessingState::InMsg;
                    }
                }
                ProcessingState::InMsg => {
                    if *byte == b'$' {
                        state = ProcessingState::WaitForMsg;
                    } else {
                        let newbyte = byte.wrapping_add(1);
                        stream.write_all(&[newbyte])?;
                    }
                }
            }
        }
    }

    Ok(())
}
```


As a reminder, this server version is _sequential_ because it accepts clients one by one; the main loop blocks on serve_connection until it's done (the client closes the connection), and only then goes back to accept the next client.

## One thread per client

Clearly, handling clients one by one won't do. In [part 2](https://eli.thegreenplace.net/2017/concurrent-servers-part-2-threads/), we've discussed approaches that use OS threads to handle clients concurrently. Let's start with the unbounded one-thread-per-client solution in Rust:

```rust
use async_socket_server::serve_connection;
use std::{net::TcpListener, thread};

fn main() -> std::io::Result<()> {
    let port = match std::env::args().nth(1) {
        Some(s) => s,
        None => "9090".to_string(),
    };
    let addr = format!("127.0.0.1:{port}");

    let listener = TcpListener::bind(addr)?;
    println!("Serving on port {port}");

    loop {
        let (stream, addr) = listener.accept()?;
        println!("connection received from {}", addr);

        let res = thread::Builder::new().spawn(move || {
            if let Err(e) = serve_connection(stream) {
                eprintln!("error serving connection: {}", e);
            } else {
                println!("peer done {addr}");
            }
        });

        if let Err(e) = res {
            eprintln!("error spawning thread: {}", e);
        }
    }
}

```

The spawn method returns a Result<JoinHandle<T>>; on success, we allow the handle to be dropped at the end of the loop iteration. In Rust, this _detaches_ the thread; we don't actually wait for it to complete. This is reasonable for our code sample, because the loop is _infinite_; it never terminates anyway. The potential for runaway threads is just one of the issues with the unbounded threads approach discussed in part 2. The solution is to use a fixed thread pool.

Before diving into the code, a quick note on the design: the thread pool is a fixed set of threads that await "jobs" and handle them to completion. In our case a "job" is serve_connection for a specific client. There are many ways to implement a thread pool; for our use case, I went with a set of threads that all get a shared _channel_ to which the main thread sends jobs. A worker thread picks up the next job from the channel, serves it to completion, and goes back to waiting for the next job. Here's how this looks in code:

```rust
struct Job {
    stream: TcpStream,
    addr: SocketAddr,
}

fn worker(receiver: Receiver<Job>) {
    while let Ok(job) = receiver.recv() {
        if let Err(e) = serve_connection(job.stream) {
            eprintln!("error serving connection from {}: {}", job.addr, e);
        } else {
            println!("peer done {}", job.addr);
        }
    }
}

```

What is Receiver? It's a type from the crossbeam_channel crate:

use crossbeam_channel::{Receiver, bounded};

Rust's builtin channels in std are _mpsc_ - multi producer, single consumer, but what we need for our job queue is a channel that supports multiple consumers (the worker threads). While std does have [mpmc](https://doc.rust-lang.org/std/sync/mpmc/fn.channel.html), this is an experimental API only available in nightly versions at the time of writing. Therefore, I've opted to include the crossbeam_channel crate that provides well-tested mpmc channels for this sample [[1]](https://eli.thegreenplace.net/2026/concurrent-servers-part-7-rust/#footnote-1).

And here's the main function:


use async_socket_server::serve_connection;
use crossbeam_channel::{Receiver, bounded};
use std::{
    io,
    net::{SocketAddr, TcpListener, TcpStream},
    thread,
};

const NUM_WORKERS: usize = 256;
const JOB_QUEUE_CAPACITY: usize = NUM_WORKERS;

fn main() -> io::Result<()> {
    let port = match std::env::args().nth(1) {
        Some(s) => s,
        None => "9090".to_string(),
    };
    let addr = format!("127.0.0.1:{port}");

    let listener = TcpListener::bind(addr)?;
    println!("Serving on port {port}");

    let (tx, rx) = bounded::<Job>(JOB_QUEUE_CAPACITY);
    for _ in 0..NUM_WORKERS {
        let receiver = rx.clone();
        thread::Builder::new().spawn(move || worker(receiver))?;
    }
    drop(rx);

    loop {
        let (stream, addr) = listener.accept()?;
        println!("connection received from {addr}");

        // A full queue blocks this loop, preventing it from accepting more
        // connections until a worker becomes available.
        if tx.send(Job { stream, addr }).is_err() {
            return Err(io::Error::other("all connection workers stopped"));
        }
    }
}

Note that our job channel is _bounded_ - it has a fixed size. This helps naturally implement a _backpressure_ mechanism - if too many clients connect, the following clients will have to wait - the main loop blocks on tx.send and won't accept additional clients on the socket until jobs are cleared from the channel.

## Asynchronous, event-driven server

In parts 4, 5 and 6 of the series we've discussed _event-driven_, or _asynchronous_ servers. Let's see how it's done in Rust. Specifically, [part 6](https://eli.thegreenplace.net/2018/concurrent-servers-part-6-callbacks-promises-and-asyncawait/) presented a gradation from callbacks to promises to async/await mechanisms; Rust supports all of these and - as you'd expect - modern code is usually written with async/await while hiding all the details of promises (called _futures_ in Rust) underneath.

Without further ado, here's our simple state machine protocol in asynchronous Rust:

use async_socket_server::ProcessingState;
use tokio::io::{self, AsyncReadExt, AsyncWriteExt};
use tokio::net::TcpListener;

#[tokio::main]
async fn main() -> io::Result<()> {
    let port = match std::env::args().nth(1) {
        Some(s) => s,
        None => "9090".to_string(),
    };
    let addr = format!("127.0.0.1:{port}");

    let listener = TcpListener::bind(addr).await?;
    println!("Serving on port {port}");

    loop {
        let (socket, addr) = listener.accept().await?;
        println!("connection received from {addr:?}");

        tokio::spawn(async move {
            if let Err(e) = serve_connection_async(socket).await {
                eprintln!("error serving connection from {addr:?}: {e}");
            } else {
                println!("peer done {addr:?}");
            }
        });
    }
}

Rust takes an interesting approach to async programming: it supports some of its fundamental building blocks (like futures and the async and await keywords) in the core language, but leaves the actual async engine implementation (the thing that implements the event loop) to external crates. By far the most popular crate for async programming in Rust is is [tokio](https://tokio.rs/), so that's what we're using here.

After reading the JS code in part 6, the Rust snippet above should appear fairly familiar, except perhaps the explicit tokio task "spawn". Instead of enqueuing a callback on the connection returned by listener.accept, the code spawns a tokio task, which can be seen as a [green thread](https://en.wikipedia.org/wiki/Green_thread), and hence uses similar terminology [[2]](https://eli.thegreenplace.net/2026/concurrent-servers-part-7-rust/#footnote-2). These tasks must not issue blocking calls; therefore, they are supposed to use tokio's I/O utilities instead of the usual, blocking std utilities. In fact, we have to implement an async version of serve_connection to make this work:

async fn serve_connection_async(mut stream: tokio::net::TcpStream) -> std::io::Result<()> {
    stream.write_all(b"*").await?;

    let mut state = ProcessingState::WaitForMsg;
    let mut buf = [0u8; 1024];
    loop {
        let n = stream.read(&mut buf).await?;
        if n == 0 {
            // Connection closed by the client.
            break;
        }

        for byte in &buf[..n] {
            match state {
                ProcessingState::WaitForMsg => {
                    if *byte == b'^' {
                        state = ProcessingState::InMsg;
                    }
                }
                ProcessingState::InMsg => {
                    if *byte == b'$' {
                        state = ProcessingState::WaitForMsg;
                    } else {
                        let newbyte = byte.wrapping_add(1);
                        stream.write_all(&[newbyte]).await?;
                    }
                }
            }
        }
    }

    Ok(())
}

Note how similar this code is to serve_connection from earlier; the only real differences are the await calls on socket reads and writes [[3]](https://eli.thegreenplace.net/2026/concurrent-servers-part-7-rust/#footnote-3), and the types involved. For example, instead of a std::net::TcpStream used in the synchronous samples, here we're using tokio::net::TcpStream. Tokio has an underlying dependency called [mio](https://github.com/tokio-rs/mio) to handle non-blocking APIs for all kinds of I/O. It wraps OS-specific event loops like epoll to do so efficiently.

## Asynchronous primality testing servers

While most of the series has been using a simple state machine server as the driving example, part 6 switched focus to a server for primality testing which simulates long compute tasks. Let's see how this is done in Rust with tokio:

use tokio::io::{self, AsyncReadExt, AsyncWriteExt};
use tokio::net::TcpListener;

#[tokio::main]
async fn main() -> io::Result<()> {
    let port = match std::env::args().nth(1) {
        Some(s) => s,
        None => "8070".to_string(),
    };
    let addr = format!("127.0.0.1:{port}");

    let listener = TcpListener::bind(addr).await?;
    println!("Serving on port {port}");

    loop {
        let (socket, addr) = listener.accept().await?;
        println!("connection received from {addr:?}");

        tokio::spawn(async move {
            if let Err(e) = serve_client(socket).await {
                eprintln!("error serving connection from {addr:?}: {e}");
            } else {
                println!("peer done {addr:?}");
            }
        });
    }
}

async fn serve_client(mut stream: tokio::net::TcpStream) -> std::io::Result<()> {
    let mut buf = [0u8; 1024];

    loop {
        let n = stream.read(&mut buf).await?;
        if n == 0 {
            // Connection closed by the client.
            return Ok(());
        }

        // Parse read buf into u64.
        let num = std::str::from_utf8(&buf[..n])
            .map_err(|error| io::Error::new(io::ErrorKind::InvalidData, error))?
            .trim()
            .parse::<u64>()
            .map_err(|error| io::Error::new(io::ErrorKind::InvalidData, error))?;

        let answer = if isprime(num, true) {
            "prime"
        } else {
            "composite"
        };
        stream
            .write_all((answer.to_string() + "\n").as_bytes())
            .await?;
    }
}

This code is very similar to the previous snippet conceptually; isprime is:

// Check if n is prime, returning a boolean. The delay parameter is optional;
// if true, the function will block for n milliseconds before computing the
// answer. This is useful for simulating a long-running computation.
fn isprime(n: u64, delay: bool) -> bool {
    if delay {
        std::thread::sleep(std::time::Duration::from_millis(n));
    }

    if n < 2 {
        return false;
    }
    if n % 2 == 0 {
        return n == 2;
    }

    let mut r = 3;
    while r * r <= n {
        if n % r == 0 {
            return false;
        }
        r += 2;
    }
    true
}

Note that this sample demonstrates a job that can block (simulated with a sleep in this case). This can be problematic in an async context, as the [tokio documentation explains](https://docs.rs/tokio/latest/tokio/task/index.html#blocking-and-yielding).

One potential solution would be to dispatch a blocking task to a separate thread pool and use [tokio channels](https://docs.rs/tokio/1.53.1/tokio/sync/) to communicate with it; this is similar to the approach we've taken in the thread pool sample above.

Part 6 also included a version of this server that caches data on a local Redis instance; the goal was to demonstrate the complexity of event-driven code when additional layers of callbacks are added and how async/await can help mitigate that. Here's our Rust version of this server, using the redis crate (that has a tokio component enabled explicitly to support async calls):

use redis::AsyncTypedCommands;
use redis::aio::MultiplexedConnection;
use tokio::io::{self, AsyncReadExt, AsyncWriteExt};
use tokio::net::TcpListener;

#[derive(Clone)]
struct AppState {
    redis_connection: MultiplexedConnection,
}

const REDIS_URL: &str = "redis://127.0.0.1";

#[tokio::main]
async fn main() -> io::Result<()> {
    let port = match std::env::args().nth(1) {
        Some(s) => s,
        None => "8070".to_string(),
    };
    let addr = format!("127.0.0.1:{port}");

    let app_state = AppState {
        redis_connection: {
            redis::Client::open(REDIS_URL)
                .map_err(io::Error::other)?
                .get_multiplexed_async_connection()
                .await
                .map_err(io::Error::other)?
        },
    };

    let listener = TcpListener::bind(addr).await?;
    println!("Serving on port {port}");

    loop {
        let (socket, addr) = listener.accept().await?;
        println!("connection received from {addr:?}");
        let app_state = app_state.clone();

        tokio::spawn(async move {
            if let Err(e) = serve_client(socket, app_state).await {
                eprintln!("error serving connection from {addr:?}: {e}");
            } else {
                println!("peer done {addr:?}");
            }
        });
    }
}

async fn serve_client(
    mut stream: tokio::net::TcpStream,
    mut app_state: AppState,
) -> std::io::Result<()> {
    let mut buf = [0u8; 1024];

    loop {
        let n = stream.read(&mut buf).await?;
        if n == 0 {
            // Connection closed by the client.
            return Ok(());
        }

        // Parse read buf into u64.
        let num = std::str::from_utf8(&buf[..n])
            .map_err(|error| io::Error::new(io::ErrorKind::InvalidData, error))?
            .trim()
            .parse::<u64>()
            .map_err(|error| io::Error::new(io::ErrorKind::InvalidData, error))?;

        // Try the redis cache first. If found in cache, send the cached
        // answer.
        let cachekey = format!("primecache:{num}");
        match app_state.redis_connection.get(cachekey.clone()).await {
            Ok(Some(cached)) => {
                stream.write_all((cached.clone() + "\n").as_bytes()).await?;
                println!("cached num {num} is {cached}");
                continue;
            }
            Ok(None) => {
                // Not found in cache, continue to compute.
            }
            Err(error) => {
                return Err(io::Error::other(error));
            }
        }

        let answer = if isprime(num, true) {
            "prime"
        } else {
            "composite"
        };

        // Save answer in cache and send it to the client.
        app_state
            .redis_connection
            .set(cachekey, answer)
            .await
            .map_err(io::Error::other)?;
        stream
            .write_all((answer.to_string() + "\n").as_bytes())
            .await?;
    }
}

Notes:

- Because of the [function color problem](https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/), the redis crate has a connection constructor specifically for async: get_multiplexed_async_connection.
- Here we have an example of shared state between tokio tasks - the Redis connection. Note that we don't require any particular synchronization because MultiplexedConnection is Clone; cloning it to different tasks is safe - and in fact that's what we do for each new task. There's no magic here; if you look inside MultiplexedConnection, you'll see that it already has all the synchronization mechanisms implemented internally, as needed.
- Due to the magic of async/await, the code in serve_client is nice and linear. We simply await on the Redis call, and once it's back we continue with the rest of the handler. Since we're using an async Redis connection, in case waiting is required, control will be ceded to some other task that's not currently blocked on I/O.

In conclusion, while Rust provides excellent support for async programming, it doesn't solve its inherent issues like function colors and the need for careful separation between blocking and non-blocking tasks. These issues are typically surmountable with some extra care, and async programming with Tokio in Rust is very popular due to its performance benefits.

## Code

All the code for this post is available [on GitHub](https://github.com/eliben/code-for-blog/tree/main/2026/async-socket-server).

---

|   |   |
|---|---|
|[[1]](https://eli.thegreenplace.net/2026/concurrent-servers-part-7-rust/#footnote-reference-1)|Rust makes a conscious design choice to keep its standard library minimal. In part 2, we've used Python's builtin stdlib thread pools; Rust also has several crates that implement thread pools, but I decided against using them because they obscure the underlying working of the code too much. Our sample implements a simple thread pool, but unfortunately still has to reach for an external crate to use a well-supported multi-consumer channel mechanism.|

|   |   |
|---|---|
|[[2]](https://eli.thegreenplace.net/2026/concurrent-servers-part-7-rust/#footnote-reference-2)|tokio is quite sophisticated; it manages a thread pool with an efficient work-stealing mechanism, onto which new tasks are spawned. Therefore, all shared data must be protected with mutexes or other synchronization primitives, and Rust's traits like Send and Sync play an important role, where applicable.|

|   |   |
|---|---|
|[[3]](https://eli.thegreenplace.net/2026/concurrent-servers-part-7-rust/#footnote-reference-3)|Having to code this alternative just for the sake of async invocation is a classical example of the [function color problem](https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/).|