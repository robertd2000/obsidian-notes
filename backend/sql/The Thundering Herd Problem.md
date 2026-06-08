https://stenzr.medium.com/the-thundering-herd-problem-2d19e9492cbc

## Introduction

In modern computing systems, performance bottlenecks are common, especially in distributed architectures and high-traffic services. One such bottleneck is known as the **Thundering Herd Problem** — a situation where multiple processes or threads simultaneously attempt to access the same resource, overwhelming the system and degrading its performance.

In this blog, we’ll dive deeper into the mechanics of the Thundering Herd problem, how it manifests in real-world systems, and, most importantly, how we can mitigate its effects to ensure smoother system operation.

## What is the Thundering Herd Problem?

The **Thundering Herd Problem** occurs when a large number of processes or threads wake up or are triggered to compete for a resource that becomes available. This happens in a scenario where:

- Multiple entities are waiting on a lock or resource.
- When the resource becomes free or available, all the waiting entities are notified simultaneously.
- They rush to acquire the resource at the same time, causing a spike in system load.

This **_“herd”_** effect can cause the system to slow down drastically, fail to serve the resource properly, or even crash under the load.

**Technical Breakdown:**

1. **Waiting Processes:** Multiple processes wait on a condition, such as accessing a database row or connecting to a server.
2. **Wake-Up Trigger:** Once the resource (file lock, server connection, etc.) becomes available, the kernel or service wakes up all the processes at once.
3. **Simultaneous Contention:** These processes now contend for the same resource, leading to spikes in CPU usage, I/O, and network calls.
4. **Performance Impact:** The system may experience long delays, high resource utilization, or even a complete failure due to the excessive load.

## How the Thundering Herd Problem Manifests

1. **Resource Contention**  
    In scenarios where multiple threads or processes are dependent on a shared resource, the moment that resource becomes available, a “stampede” of processes tries to acquire it. This usually happens in **file locks**, **database access**, or **network socket availability**.

Imagine 100 workers in a distributed system all trying to acquire the same lock to perform a critical section of code. When the lock is released, all workers wake up and rush to acquire it, causing a sharp increase in CPU usage as each worker tries to acquire the lock, many of which will fail and have to retry.

**2. Cache Expiration in Web Servers**  
In high-traffic web applications, **cache expiration** can often trigger the Thundering Herd problem. When cached data expires, many clients will attempt to retrieve the same data at once, which results in many requests hitting the backend or database, overwhelming the system.

Consider an e-commerce site with a heavily accessed product page. If cached data for this page expires, hundreds or thousands of users may issue requests to regenerate the cache simultaneously, overloading the server with database queries.

**3. Service Downtime and Recovery**  
When a critical service goes down, especially in microservice-based architectures, downstream services often retry the failed requests. Once the failed service comes back online, the queued requests flood the service, which may lead to another crash.

Think of a payment service that goes down during a high-traffic sale event. While the payment gateway is down, thousands of customers’ payment attempts fail, and each service retries these requests. When the payment gateway comes back online, it’s hit with a flood of retry requests, potentially crashing the service again or causing a severe delay.

**4. Distributed Systems and Replication Lags**  
Distributed databases with replica nodes (such as **MongoDB** or **Cassandra**) may face thundering herd scenarios when nodes lag behind or become unavailable. When these nodes recover, they are swamped with data replication tasks as well as client read requests, overwhelming the database’s ability to function.

## Get Rohit Kumar’s stories in your inbox

Join Medium for free to get updates from this writer.

Subscribe

Remember me for faster sign in

In a distributed environment, if a primary replica fails and then comes back online, all the read and write requests previously queued may flood the replica node, causing performance degradation for the entire system.

## Real-Life Examples of the Thundering Herd Problem

1. **The Reddit Hug of Death**

During a Reddit AMA or a viral post, if a link to a smaller website is shared, thousands of users may click the link at the same time. This sudden spike in traffic overloads the smaller website, causing it to crash. The **Reddit Hug of Death** is a perfect illustration of the Thundering Herd problem in the context of internet traffic.

2. **Black Friday E-Commerce Surge**

On e-commerce platforms like Amazon or Flipkart, a major sale event (like Black Friday) triggers high traffic to specific services, such as the checkout process. During such events, if a backend service (e.g., inventory tracking) experiences downtime, it leads to a surge of retry requests when the service is restored. The service is overwhelmed again, potentially causing cascading failures across the entire platform.

3. **Content Delivery Network (CDN) Failures**

When a **CDN** cache node experiences an outage, all requests in that region may suddenly be routed to a backup origin server. If the origin server is not designed to handle such a load, this leads to thundering herd issues where multiple requests come in at once, slowing down the content delivery system.

4. **Financial Trading Systems**

In high-frequency trading, when a critical trading system or market data feed goes down, traders’ systems automatically attempt to reconnect or re-fetch the data. When the system comes back online, it is flooded with connection attempts and data requests, leading to further delays and data feed disruptions.

## Mitigation Techniques for the Thundering Herd Problem

Dealing with the Thundering Herd problem requires proactive architectural and algorithmic strategies. Here’s how you can mitigate its impact:

1. **Exponential Backoff**  
    Rather than allowing processes to retry immediately when a resource becomes available, use an exponential backoff strategy. This method spaces out retries over longer intervals, reducing the spike in system load. In HTTP-based microservices, implementing exponential backoff in retries ensures that retry attempts happen less frequently after each failure, spreading the load over time.

**2. Randomized Retry Delays**  
By introducing randomization in the delay between retries, processes are less likely to all wake up and retry at exactly the same time. This reduces contention for the resource. Instead of retrying a failed database query after exactly 1 second, each process waits for a random delay between 500ms and 1500ms before retrying.

**3. Staggered Cache Expiration**  
For web servers or distributed caches, avoid having all cache entries expire at the same time. Stagger expiration times so that cache misses are spread out over time, reducing load on the backend. In Redis or Memcached, use staggered TTL (time-to-live) values for highly requested data so that not all clients request fresh data at once.

**4. Connection Pooling and Limits**  
In systems that deal with databases, implementing connection pooling and limiting the number of active connections can prevent a surge of requests from overwhelming the system.

**5. Circuit Breaker Pattern**  
The **circuit breaker** design pattern temporarily disables access to a failing service, preventing retries from overwhelming the system. After a cool-down period, requests are gradually allowed through.

In microservices, circuit breakers can be used to stop retry attempts after a certain number of failures. Once the system stabilizes, requests are gradually allowed again.

### Conclusion

The Thundering Herd problem can cripple even the most robust systems if not addressed properly. It often sneaks up in high-traffic environments or in scenarios where services are tightly coupled. Understanding the problem, anticipating it, and implementing strategies such as exponential backoff, randomized retries, and staggered cache expirations can greatly reduce its impact.

By taking preventive measures, you can ensure that your system gracefully handles spikes in load, preventing a catastrophic collapse and keeping your services running smoothly, even under extreme traffic conditions.