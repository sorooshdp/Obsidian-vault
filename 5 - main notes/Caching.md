2025/11/27  -  18:50
status: 

Tags: [[Performance]] [[optimization]] [[SQL]]

# Caching
everything that we do on with our computer requires data from mobile apps to web apps. When designing systems to store and retrieve data, we are faced with trade-offs between **capacity**, **speed**, **cost** , and durability. For a given budget, you can either get a large amount of slower data storage, or a small amount of faster storage. Engineers get around this by combining the two: Pair a large amount of cheaper slow storage with a small amount of expensive fast storage. Data that is accessed more _frequently_ can go in the fast storage, and the other data that we use less often goes in the slow storage. This is the core principle of caching.

Every time we request a piece of data, the request is first sent to the cache. If it's there, we consider it a cache hit and quickly send the data back.

If we don't find it, we call it a cache miss. At this point we pass the request on to the slow storage, find the data, send it back to the cache, and then send it to the requester.

The "hit rate" is the percentage of time we get cache hits. In other words:

`hit_rate = (cache_hits / total_requests) x 100`

to increase the hit rate we can increase the size of cache but it costs too much, it's all about trade-off. faster data lockups means more cost or more size limitations due to how physically close the data needs to be to the requester.

## Temporal locality 
In many apps and websites, the data which is newer and more recent has a more chance of getting a request from the users, so it is best practice to put them in the cache rather than putting the data which is not recently stored, especially in social media apps.

## Spatial locality
In some data storage systems, when a chunk of data is being requested from the user, it is likely that the data which lives before and after that chunk in the memory is going to requested in the future. so fetching the data around it is a strong optimization technique.

# Geospatial
When a data storage is located in the US, users who live in that area will experience a faster response from the server, but as we move away from the main server, the latency of accessing the data goes up. Therefor, engineers use CDNs to provide the data to users. Each user would get it's content from the nearest CDN. one possibly strategy is to copy all of the DB to each CDN, so the hit rate would be nearly 100%.

## Replacement policies
When our cache gets full we need an algorithm to put the new data in it and remove the old ones. 

### FIFO
First In, First Out, or FIFO, is the simplest cache replacement policy. It works like a [queue](https://encore.dev/blog/queueing). New items are added to the beginning (on the left). When the cache queue is full, the least-recently added item gets evicted (right). While simple to implement, FIFO isn't optimal for most caching scenarios because it doesn't consider usage patterns. We have smarter techniques to deal with this.

### LRU
Least Recently Used (LRU) is a popular choice, and the industry standard for many caching systems. Unlike FIFO, LRU always evicts the item that has least-recently been requested, a sensible choice to maximize cache hits. This aligns well with temporal locality in real-world data access patterns.

### Time-Aware LRU

We can augment the LRU algorithm in a number of ways to be time-sensitive. How precisely we do this depends on our use case, but it often involves using LRU plus giving each element in the cache a timer. When time is up, we evict the element!

We might imagine this being useful in cases like:
- For a social network, automatically evicting posts from the cache after 48 hours.
- For a weather app, automatically evicting previous-days weather info from the cache when the clock turns to a new day.
- In an email app, removing the email from the cache after a week, as it's unlikely to be read again after that.

## Postgres and MySQL
Caches are frequently put in front of a database as a mechanism to cache the results of slow queries. However, DBMSs like Postgres and MySQL also use caching within their implementations.

Postgres implements a two-layer caching strategy. First, it uses `shared_buffers`, an internal cache for data pages that store table information. This keeps frequently read row data in memory while less-frequently accessed data stays on disk. Second, Postgres relies heavily on the operating system's filesystem page cache, which caches disk pages at the kernel level. This creates a double-buffering system where data can exist in both Postgres's `shared_buffers` and the OS page cache. Many deployments set `shared_buffers` to around 25% of available RAM and let the filesystem cache handle much of the remaining caching work. The `shared_buffers` value can be configured in the Postgres config file.

MySQL does a similar thing with the buffer pool. Like Postgres, this is an internal cache to keep recently used data in RAM.


# References
- https://planetscale.com/blog/caching
- [Twitter's petabyte-scale caching system](https://www.usenix.org/system/files/osdi20-yang.pdf)
- [How Uber handles 40 million cache requests per-second](https://www.uber.com/blog/how-uber-serves-over-40-million-reads-per-second-using-an-integrated-cache/)
- [TikTok's database handles 100 million QPS (and uses a cache)](https://vldb.org/pvldb/vol15/p3306-li.pdf)