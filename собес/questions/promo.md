I worked on a backend for an interactive promotional game.

It was a high-load system where users interacted with the server in real time during campaigns.

I developed the backend in Go and implemented WebSocket-based communication to handle live events.

The main challenge was handling traffic spikes and keeping the system stable under load.

I focused on optimizing performance, reducing latency, and improving memory usage.

## 🔥 Follow-up вопросы

### ➤ How did you handle high concurrency?

> I used Go’s concurrency model — goroutines and efficient handling of connections.  
> I also tried to minimize blocking operations.

---

### ➤ What was the biggest challenge?

> Handling traffic spikes and maintaining stability during peak usage.

---

### ➤ How did you optimize performance?

> By improving memory usage, reducing blocking operations, and optimizing how the server handled concurrent connections.

---

### ➤ How did you ensure stability?

> By making the system efficient under load and avoiding resource bottlenecks.

---

### ➤ Did you use any message queues?

👉 если не было:

> No, the system mainly relied on direct real-time communication using WebSockets.

---

### ➤ What would you improve?

> I would introduce more asynchronous processing to better handle peak нагрузку.