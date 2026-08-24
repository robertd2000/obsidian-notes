This is part 6 in a series of posts on writing concurrent network servers. Parts 3, 4, and 5 in the series discussed the _event-driven_ approach to building concurrent servers, alternatively known as _asynchronous programming_. In this part, we're going to look at some of the challenges inherent in this style of programming and examine some of the modern solutions available.

This post covers many topics, and as such can't cover all of them in great detail. It comes with sizable, fully-working code samples, so I hope it can serve as a good starting point for learning if these topics interest you.

All posts in the series:

- [Part 1 - Introduction](https://eli.thegreenplace.net/2017/concurrent-servers-part-1-introduction/)
- [Part 2 - Threads](https://eli.thegreenplace.net/2017/concurrent-servers-part-2-threads/)
- [Part 3 - Event-driven](https://eli.thegreenplace.net/2017/concurrent-servers-part-3-event-driven/)
- [Part 4 - libuv](https://eli.thegreenplace.net/2017/concurrent-servers-part-4-libuv/)
- [Part 5 - Redis case study](https://eli.thegreenplace.net/2017/concurrent-servers-part-5-redis-case-study/)
- Part 6 - Callbacks, Promises and async/await (this part)
- [Part 7 - Rust](https://eli.thegreenplace.net/2026/concurrent-servers-part-7-rust/)

## Revisiting the primality testing server with Node.js

So far the series has focused on a simple state-machine protocol, to demonstrate the challenges of keeping client-specific state on the server. In this part, I want to focus on a different challenge - keeping track of waiting for multiple things on the server side. To this end, I'm going to revisit the primality testing server that appeared in [part 4](https://eli.thegreenplace.net/2017/concurrent-servers-part-4-libuv/), where it was implemented in C using libuv.

Here we're going to reimplement this in JavaScript, using the Node.js server-side framework and execution engine. Node.js is a popular server-side programming environment that brought the asynchronous style of programming into the limelight when it appeared in 2009 [[1]](https://eli.thegreenplace.net/2018/concurrent-servers-part-6-callbacks-promises-and-asyncawait/#footnote-1).

The C code for the original primality testing server [is here](https://github.com/eliben/code-for-blog/blob/main/2017/async-socket-server/uv-isprime-server.c). It listens on a socket for numbers to arrive, tests them for primality (using the slow brute-force method) and sends back "prime" or "composite". It optionally uses libuv's work queues to offload the computation itself to a thread, to avoid blocking the main event loop.

Let's reconstruct this server in steps in Node.js, starting with a basic server that does all computations in the main thread (all the code for this post is [available here](https://github.com/eliben/code-for-blog/blob/main/2018/async-socket-server/)):

```js
var net = require('net');
var utils = require('./utils.js');

var portnum = 8070;
if (process.argv.length > 2) {
  portnum = process.argv[2];
}

var server = net.createServer();
server.on('connection', handleConnection);

server.listen(portnum, function() {
  console.log('Serving on port %d', portnum);
});

function handleConnection(conn) {
  var remoteAddress = conn.remoteAddress + ':' + conn.remotePort;
  console.log('peer %s connected', remoteAddress);

  conn.on('data', onConnData);
  conn.once('close', onConnClose);
  conn.on('error', onConnError);

  function onConnData(d) {
    var num = utils.buf2num(d);
    console.log('num %d', num);

    var answer = utils.isPrime(num, true) ? "prime" : "composite";
    conn.write(answer + '\n');
    console.log('... %d is %s', num, answer);
  }

  function onConnClose() {
    console.log('connection from %s closed', remoteAddress);
  }

  function onConnError(err) {
    console.log('connection %s error: %s', remoteAddress, err.message);
  }
}
```


This is standard Node.js fare; the interesting work happens in the onConnData callback, which is called whenever new data arrives on the socket. We're missing a couple of utility functions used by this code - they are in utils.js:

```js
// Check if n is prime, returning a boolean. The delay parameter is optional -
// if it's true the function will block for n milliseconds before computing the
// answer.
exports.isPrime = function(n, delay) {
  if (delay === true) {
    sleep(n);
  }

  if (n % 2 == 0) {
    return n == 2 ? true : false;
  }

  for (var r = 3; r * r <= n; r += 2) {
    if (n % r == 0) {
      return false;
    }
  }
  return true;
}

// Parse the given a buffer into a number. buf is of class Buffer; it stores the
// ascii representation of the number followed by some non-digits (like a
// newline).
exports.buf2num = function(buf) {
  var num = 0;
  var code0 = '0'.charCodeAt(0);
  var code9 = '9'.charCodeAt(0);
  for (var i = 0; i < buf.length; ++i) {
    if (buf[i] >= code0 && buf[i] <= code9) {
      num = num * 10 + buf[i] - code0;
    } else {
      break;
    }
  }
  return num;
}

// Blocking sleep for the given number of milliseconds. Uses a spin-loop to
// block; note that this loads the CPU and is only useful for simulating load.
function sleep(ms) {
  var awake_time = new Date().getTime() + ms;
  while (awake_time > new Date().getTime()) {
  }
}
```


For testing and demonstration purposes, isPrime accepts an optional delay parameter; if true, the function will sleep for the number of milliseconds given by n before computing whether n is a prime [[2]](https://eli.thegreenplace.net/2018/concurrent-servers-part-6-callbacks-promises-and-asyncawait/#footnote-2).

## Offloading CPU-intensive computations

Naturally, the server shown above is poorly designed for concurrency; it has a single thread that will stop listening for new clients while it's busy computing the prime-ness of a large number for an existing client.

The natural way to handle this is to offload the CPU intensive computation to a thread. Alas, JavaScript doesn't support threads and Node.js doesn't either. Node.js _does_ support sub-processes though, with its child_process package. Our [next version of the server](https://github.com/eliben/code-for-blog/blob/main/2018/async-socket-server/primeserver-offload.js) leverages this capability. Here is the relevant part in the new server - the onConnData callback:

```js
function onConnData(d) {
  var num = utils.buf2num(d);
  console.log('num %d', num);

  // Fork off a worker to do this computation, and add a callback to handle
  // the result when it's ready. After the callback is set up, this function
  // returns so the server can resume the event loop.
  var worker = child_process.fork('./primeworker.js');
  worker.send(num);
  worker.on('message', message => {
    var answer = message.result ? "prime" : "composite";
    conn.write(answer + '\n');
    console.log('... %d is %s', num, answer);
  });
}

```

When new data is received from a connected client, this server forks off a sub-process to execute code in primeworker.js, sends it the task using IPC and attaches a callback on new messages received from the worker. It then cedes control to the event loop - so there's no bad blocking happening here. primeworker.js is very simple:

```js
var utils = require('./utils.js');

process.on('message', message => {
  console.log('[child %d] received message from server:', process.pid, message);

  // Compute the result (with emulate ddelay) and send back a message.
  process.send({task: message, result: utils.isPrime(message, true)});
  process.disconnect();
  console.log('[child %d] exiting', process.pid);
  process.exit();
});
```


It waits for a message on its IPC channel, computes the prime-ness of the number received, sends the reply and exits. Let's ignore the fact that it's wasteful to launch a subprocess for each number, since the focus of this article is the callbacks in the server. A more realistic application would have a pool of "worker" processes that persist throughout the server's lifetime; this wouldn't change much on the server side, however.

The important part to notice here is that we have a nested callback within the server's onConnData. The server's architecture is still quite simple - let's see how it handles added complexity.

## Adding caching

Let's grossly over-engineer our silly primality testing server by adding a cache. Not just any cache, but stored in Redis! How about that for a true child of the 2010s? The point of this is educational, of course, so please bear with me for a bit.

We assume a Redis server is running on the local host, listening on the default port. We'll use the redis package to talk to it; the [full code is here](https://github.com/eliben/code-for-blog/blob/main/2018/async-socket-server/primeserver-offload-caching.js), but the interesting part is this:

```js
function onConnData(d) {
  var num = utils.buf2num(d);
  console.log('num %d', num);

  var cachekey = 'primecache:' + num;
  redis_client.get(cachekey, (err, res) => {
    if (err) {
      console.log('redis client error', err);
    } else {
      if (res === null) {
        var worker = child_process.fork('./primeworker.js');
        worker.send(num);
        worker.on('message', message => {
          var answer = message.result ? 'prime' : 'composite';
          redis_client.set(cachekey, answer, (err, res) => {
            if (err) {
              console.log('redis client error', err);
            } else {
              conn.write(answer + '\n');
              console.log('... %d is %s', num, answer);
            }
          });
        });
      } else {
        // The strings 'prime' or 'composite' are stored in the Redis cache.
        console.log('cached num %d is %s', num, res);
        conn.write(res + '\n');
      }
    }
  });
}
```


Let's see what's going on. When a new number is received from the client, we first check to see if it's already in the cache. This involves contacting the Redis server, so naturally it has to be done asynchronously with a callback registered for when the answer is ready. If the number is in the cache, we're pretty much done.

If it's not, we have to spawn a worker to compute it; then, once the answer is ready we want to write it to the cache. If the write is successful, we return the answer [[3]](https://eli.thegreenplace.net/2018/concurrent-servers-part-6-callbacks-promises-and-asyncawait/#footnote-3).

## Callback hell

Taking another look at the last code snippet, we see callbacks nested 3 layers deep. That's inside onConnData, which is itself a callback - so make it 4 layers deep. This style of code is so common and notorious in event-driven programming that it has an epithet - "callback hell".

The problem is often visualized as this deep, deep callback nest, but IMHO that's not the real issue. Callback nesting is just a syntactic convenience JS makes particularly easy, so folks use it. If you look at the C code [in part 4](https://eli.thegreenplace.net/2017/concurrent-servers-part-4-libuv), it has a similar level of _logical_ nesting, but since each function is standalone and not a closure embedded in a surrounding function, it's less visually jarring.

The "just use standalone named functions" solution has issues too; closures have their benefits - for example they easily refer to values from external scopes. In the last code snippet, note how num is used in several nested callbacks but only defined inside onConnData itself. Without this lexical convenience we'd have to pass it explicitly through all the callbacks, and the same for all other common values. It's not the end of the world, but it helps explain why folks gravitate naturally to the tower of nested closures - it's less code to type.

The bigger issue with this way of programming is forcing programmers into _continuation passing style_. It's worth spending some time to explain what I mean.

Traditional, "straight-line" code looks like the following:

```c
a <- run_w()
b <- run_x(a)
c <- run_y()
d <- run_z(b, c)
```


Let's assume that each of run_* can potentially block, but it doesn't concern us because we have our own thread or something. The flow of data here is very straightforward. Now let's see how this would look using asynchronous callbacks:

```js
run_w(a =>
  run_x(a, b =>
    run_y(c =>
      run_z(b, c, ...))))
```


Nothing surprising, but note how much less obvious the flow of data is. Instead of saying "run W and get me an a", we have to say "run W and when a is ready, do ...". This is similar to continuations in programming language theory; [I've written about continuations in the past](https://eli.thegreenplace.net/2017/on-recursion-continuations-and-trampolines/), and it should be easy to find tons of other information online.

Continuation passing style is not bad per-se, but it makes it harder to keep track of the data flow in a program. It's easier to think of functions as taking values and returning values, as opposed to taking values and passing their results forward to other functions [[4]](https://eli.thegreenplace.net/2018/concurrent-servers-part-6-callbacks-promises-and-asyncawait/#footnote-4).

This problem is compounded when we consider error handling in realistic programs. Back to the straight-line code sample - if run_x encounters an error, it returns it. The place where run_x is called is precisely the right place to handle this error, because this is the place that has the full context for the call.

In the asynchronous variant, if run_x encounters an error, there's no natural place to "return" it to, because run_x doesn't really return anything. It feeds its result forward. Node.js has an idiom to support this style of programming - [error-first callbacks](http://fredkschott.com/post/2014/03/understanding-error-first-callbacks-in-node-js/).

You might think that JS's exceptions should be able to help here, but exceptions mix with callbacks even more poorly. The callback is usually invoked in a completely different stack frame from the place where it's passed into an operation. Therefore, there's no natural place to position try blocks.

## Promises

Even though the callback-programming style has some issues, they are by no means fatal. After all, many successful projects were developed with Node.js, even before the fancy new features became available in ES6 and beyond.

People have been well aware of the issues, however, and have worked hard to create solutions or at least mitigations for the most serious problems. The first such solution came to standard JS with ES6: _promises_ (also known as _futures_ in other languages). However, long before becoming a standard, promises were available as libraries. A Promise object is really just syntactic sugar around callbacks - it can be implemented as a library in pure Javascript.

There are plenty of tutorials about promises online; I'll just focus on showing how our over-engineered prime server looks when written with promises instead of naked callbacks. Here's onConnData in the [promise-based version](https://github.com/eliben/code-for-blog/blob/main/2018/async-socket-server/primeserver-promises.js):

```js
function onConnData(d) {
  var num = utils.buf2num(d);
  console.log('num %d', num);

  var cachekey = 'primecache:' + num;
  redisGetAsync(cachekey).then(res => {
    if (res === null) {
      return isPrimeAsync(num);
    } else {
      console.log('cached num %d is %s', num, res);
      return Promise.resolve(res);
    }
  }).then(res => {
    // Using Promise.all to pass 'res' from here to the next .then handler.
    return Promise.all([redisSetAsync(cachekey, res), res]);
  }).then(([set_result, computation_result]) => {
    conn.write(computation_result + '\n');
  }).catch(err => {
    console.log('error:', err);
  });
}
```


There are some missing pieces here. First, the promise-ready versions of the Redis client are defined thus:

```js
const {promisify} = require('util');

// Create a Redis client. This connects to a Redis server running on the local
// machine at the default port.
var redis_client = redis.createClient();

const redisGetAsync = promisify(redis_client.get).bind(redis_client);
const redisSetAsync = promisify(redis_client.set).bind(redis_client);
```


promisify is a Node utility function that takes a callback-based function and returns a promise-returning version. isPrimeAsync is:

```js
function isPrimeAsync(n) {
  return new Promise((resolve, reject) => {
    var child = child_process.fork('./primeworker.js');
    child.send(n);
    child.on('message', message => {
      var result = message.result ? 'prime' : 'composite';
      resolve(result);
    });
    child.on('error', message => {reject(message)});
  });
}
```


Here the Promise protocol is implemented manually. Instead of taking a callback to be invoked when the result is ready (and another to be invoked for errors), isPrimeAsync returns a Promise object wrapping a function. It can then participate in a then chain of Promises, as usual.

Now looking back at the main flow of onConnData, some things become apparent:

1. The nesting is _flattened_, turning into a chain of then calls.
2. Errors can be handled in a single catch at the end of the promise chain. Programming language afficionados will be delighted to discover that in this sense promises behave just like [continuation monads in Haskell](https://hackage.haskell.org/package/mtl-2.2.2/docs/Control-Monad-Cont.html).

Choosing promises over the callback style is a matter of preference; what makes promises really interesting, IMHO, is the next step - await.

## async and await

With ES7, Javascript added support for the async and await keywords, actually modifying the language for more convenient support of asynchronous programming. Functions returning promises can now be marked as async, and invoking these functions can be done with await. When a promise-returning function is invoked with await, what happens behind the scenes is exactly the same as in the callback or promise versions - a callback is registered and control is relinquished to the event loop. However, await lets us express this process in a very natural syntax that addresses some of the biggest issues with callbacks and promises.

[Here is our prime server again](https://github.com/eliben/code-for-blog/blob/main/2018/async-socket-server/primeserver-asyncawait.js), now written with await:

```js
async function onConnData(d) {
  var num = utils.buf2num(d);
  console.log('num %d', num);

  try {
    var cachekey = 'primecache:' + num;
    var cached = await redisGetAsync(cachekey);

    if (cached === null) {
      var computed = await isPrimeAsync(num);
      await redisSetAsync(cachekey, computed);
      conn.write(computed + '\n');
    } else {
      console.log('cached num %d is %s', num, cached);
      conn.write(cached + '\n');
    }
  } catch (err) {
    console.log('error:', err);
  }
}
```


This reads just like a blocking version [[5]](https://eli.thegreenplace.net/2018/concurrent-servers-part-6-callbacks-promises-and-asyncawait/#footnote-5), but in fact there is no blocking here; for example, with this line:

var cached = await redisGetAsync(cachekey);

A "get" request will be issued with the Redis client, and a callback will be registered for when data is ready. Until it's ready, the event loop will be free to do other work (like handle concurrent requests). Once it's ready and the callback fires, the result is assigned into cached. We no longer have to split up our code into a tower of callbacks or a chain of then clauses - we can write it in a natural sequential order. We still have to be mindful of blocking operations and be very careful about what is invoked inside callbacks, but it's a big improvement regardless.

## Conclusion

This post has been a whirlwind tour of some idioms of asynchronous programming, adding modern abstractions on top of the bare-bones libuv based servers of part 4. This information should be sufficient to understand most asynchronous code being written today.

A separate question is - is it worth it? Asynchronous code obviously brings with it some unique programming challenges. Is this the best way to handle high-load concurrency? I'm keenly interested in the comparison of this model of programming with the more "traditional" thread-based model, but this is a large topic I'll have to defer to a future post.

---

|   |   |
|---|---|
|[[1]](https://eli.thegreenplace.net/2018/concurrent-servers-part-6-callbacks-promises-and-asyncawait/#footnote-reference-1)|The main value proposition of Node.js is using the same language on the server and on the client. Client-side programmers are already familiar with JS, by necessity, so not having to learn another language to program server-side is a plus.<br><br>Interestingly, this choice also affects the fundamental architecture and "way" of Node.js; since JS is a single-threaded language, Node.js adopted this model and had to turn to asynchronous APIs to support concurrency. In fact, the libuv framework we covered in part 4 was developed as a portability layer to support Node.js.|

|   |   |
|---|---|
|[[2]](https://eli.thegreenplace.net/2018/concurrent-servers-part-6-callbacks-promises-and-asyncawait/#footnote-reference-2)|Since the idea is emulate CPU-intensive work, this is just a hack to avoid using huge primes as inputs. For anything but very large primes, even this naive algorithm executes extremely quickly so it's hard to see real delays.<br><br>Since Node.js doesn't have a sleep function (the idea of sleep is contrary to the philosophy of Node.js), we simulate it here with a busy loop checking the time. The important bit is to keep the CPU occupied, emulating an intensive computation.|

|   |   |
|---|---|
|[[3]](https://eli.thegreenplace.net/2018/concurrent-servers-part-6-callbacks-promises-and-asyncawait/#footnote-reference-3)|Note that we don't strictly have to wait for the cache write to complete before returning the answer, but this results in the cleanest protocol since it gives us a natural place to return errors.|

|   |   |
|---|---|
|[[4]](https://eli.thegreenplace.net/2018/concurrent-servers-part-6-callbacks-promises-and-asyncawait/#footnote-reference-4)|There's much more to say on the relative merits of synchronous vs. callback-based programming, but I'll leave it to another time.|

|   |   |
|---|---|
|[[5]](https://eli.thegreenplace.net/2018/concurrent-servers-part-6-callbacks-promises-and-asyncawait/#footnote-reference-5)|I wrote a blocking version of this exact server in Python, using a thread pool for concurrency; [the full code is here](https://github.com/eliben/code-for-blog/blob/main/2018/async-socket-server/primeserver-py-blocking.py). Feel free to compare the await based onConnData with the handle_client_data function. I switched to Python for this task because writing blocking code in Node.js is a bit like pissing against the wind.|