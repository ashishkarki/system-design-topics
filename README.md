# System Design - Theoritical Aspects

Various theories and some practical aspects of designing systems in the computer/IT/software world (_In context of Computer/Software/IT_).

- [System Design - Theoritical Aspects](#system-design---theoritical-aspects)
  - [Loading balancing and Load Balancers](#loading-balancing-and-load-balancers)
  - [CDNs (Content Delivery Networks)](#cdns-content-delivery-networks)
  - [Caching and how they work](#caching-and-how-they-work)
  - [Queues (more specifically Message Queues)](#queues-more-specifically-message-queues)

## Loading balancing and Load Balancers

1. Load Balancer is a machine that runs a reverse proxy software (or any software that distributes load and may provide extra features like caching etc). The goal of this software is to distribute the requests amongst multiple servers that host/run the actual application.
    1. [ASIDE] BTW, what is a reverse proxy? See [Ref](https://tinyurl.com/2p8p8cte) and some details below:
       1. A reverse proxy accepts a request from a client, forwards it to a server that can fulfill it, and returns the server’s response to the client.
       2. A reverse proxy facilitates load balancing, web acceleration (by compressing in+out data and caching common responses) and security (since the backend server/s’s identity is kept secret)
       3. Whereas deploying a load balancer makes sense only when you have multiple servers, it often makes sense to deploy a reverse proxy even with just one web server or application server. You can think of the reverse proxy as a website’s “public face.”
2. **(Most common) Strategies for Load Balancing**:

    > Most explanations are from [CloudFlare](https://tinyurl.com/yn2ts2zy)

    1. <u>Round robin</u> (Static load balancing): Round robin load balancing distributes traffic to a list of servers in rotation using the Domain Name System (DNS)
    2. <u>Least Connections</u> (Dynamic load balancing)
    3. <u>Resource based</u>: Distributes load based on what resources each server has available at the time.
       1. Specialized software (called an "agent") running on each server measures that server's available CPU and memory, and the load balancer queries the agent before distributing traffic to that server.
    4. <u>Weighted variants of the above 3 strategies</u>: basically assign a weight to each server. And, then server which has a bigger weight (i.e. is deemed to handle more traffic) will get more requests based on either round robin or least connections etc
    5. <u>Random</u> (Dynamic):
    6. <u>IP Hash</u> (Static load balancing): Combines incoming traffic's source and destination IP addresses and uses a mathematical function to convert it into a hash.
       1. Based on the hash, the connection is assigned to a specific server.

3. **TYPES of Load Balancers**: 2 Types

    > Most explanations from [nginix site](https://tinyurl.com/2p9dv8xj)

    1. <u>Layer 4</u>: operates in the Transport layer (which is 4th layer in the OSI model and has access to TCP or UDP protocols.
       1. Layer 4 load balancers make their <u>routing decisions based on address information extracted from the first few packets in the TCP stream, and do not inspect packet content.</u>
       2. A Layer 4 load balancer is often a dedicated hardware device supplied by a vendor
       3. <u>When the Layer 4 load balancer receives a request and makes the load balancing decision, it also performs Network Address Translation (NAT) on the request packet,</u> changing the recorded destination IP address from its own to that of the content server it has chosen on the internal network.
          1. Similarly, before forwarding server responses to clients, the load balancer changes the source address recorded in the packet header from the server’s IP address to its own
    2. <u>Layer 7</u>: operates in Application layer (which is the topmost layer of OSI model and basically the normal internet)
       1. Has access to everything Layer 4 has plus access to HTTP headers, cookies and payload
       2. Layer 7 load balancers base their routing decisions on various characteristics of the HTTP header and on the actual contents of the message, such as the URL, the type of data (text, video, graphics), or information in a cookie.
       3. Rather than manage traffic on a packet-by-packet basis like Layer 4 load balancers that use NAT, <u>Layer 7 load balancing proxies can read requests and responses in their entirety.</u>
       4. Layer 7 are more powerful that way but are also expensive than layer 4.

4. <u>Advantages or usages of LBs</u>:
    1. <u>Makes our system resilient</u> by routing requests to available servers if one or more servers are down
    2. Makes it easy to make our system <u>Horizontally scalable</u> (that is easily plug new servers to the LBs as required for new requests)

## CDNs (Content Delivery Networks)

1. A CDN allows for the quick transfer of assets needed for loading Internet content including HTML pages, javascript files, stylesheets, images, and videos.
2. Basically, <u>CDNs allow you to cache your static contents like images in a globally distributed network of servers (such servers are CDNs) and quickly provide them to the users of the client website.</u>
3. A requirement or good candidates of stuff served by CDNs are again static content that DO NOT CHANGE often.
4. **Types of CDN**: 2 types of CDN
   1. <u>Push CDN</u>: the client/original website Pushes new content to the CDNs. Might not be very efficient if you have large number of new resource/contents
   2. <u>Pull CDN</u>: is a lazy type CDN - it will only Pull (and hence cache) some content (generally static content) if there is a request for it (on the original or client’s website).
      1. Meaning the first request will be slow since the CDN hasn’t already kept of copy of a particular resource
      2. Subsequent requests will be very fast
5. **Summary of CDNs**:
   1. They help in reducing costs (since you don’t need to build a network of your own servers)
   2. They decrease latency (time by which a content is delivered to the actual user) by providing such contents from CDNs network of globally distributed server (whichever is the closest serves that content)
   3. One disadvantage (may be more) is that implementing CDNs increases the complexity of your system

## Caching and how they work

1. To be cost-effective and to enable efficient use of data, caches must be relatively small.
2. Why caching? To improve access to common data faster based on where you need it like so Code/Program (fastest) > Memory > Disk > Network > …
3. <u>Pros of caching</u>:
   1. Improve read performance (aka <u>lowers Latency</u>)
   2. Reduce the load (aka <u>increases Throughput</u>)
      1. <u>Latency</u> indicates <u>how long it takes</u> for packets to reach their destination. <u>Throughput</u> is the term given to <u>the number of packets</u> that are processed within a specific period of time.
      2. Lower latency goes with faster throughput, and higher latency goes with slower throughput.
4. <u>Cons of caching</u>:
   1. <u>Increases complexity</u>: overhead of adding and maintaining the cache
   2. <u>Introduces inconsistencies</u>: if cache has stale data than actual data store in the disk or server or DB etc
   3. <u>Consumes resources</u>: you need additional space or dedicated hardware or software to maintain a cache within memory or disk or elsewhere
5. <u>Caching strategies</u> (most common are follows:)
   1. <u>For Reads</u>:
      1. **Cache Aside**: if the requested data is not in the cache, get it from storage and then also store it in the cache for future usage. The app can talk to the storage if there is a cache miss.
         1. Pretty common when using a caching mechanism like Redis.
         2. **Pros**: only have to cache what is needed
         3. **Cons**:
            1. this strategy **promotes cache misses** (since we only bring stuff into cache once it is needed) which is expensive
            2. **Data staleness**: since we read from DB first and then into cache, by the time data is stored in cache, the DB data might already be updated
            3. **Implementation complexity**: developers have to work with and possibly implement two separate APIs and/or libraries
      2. **Read Through**: application doesn’t have direct access to storage but always, only talks to the cache.
         1. If there is a cache-miss, the cache’s API will fetch those data from storage into the cache and then provide it to the application.
         2. Pretty common with ORM frameworks
         3. **Pros**:
            1. cache only what’s needed
            2. **transparent** since the developer need not even worry about working with two different libraries (transparent like as if cache were allowing us to looking though to DB)
         4. **Cons**:
            1. this strategy **promotes cache misses** (since we only bring stuff into cache once it is needed) which is expensive
            2. **Data staleness**: since we read from DB first and then into cache, by the time data is stored in cache, the DB data might already be updated
            3. **Reliability**: how reliable is the cache? Since it is being used as the single source of all the data requirements for the application
   2. <u>For Writes</u>
       1. **Write Through**: The application interacts with an API that, for each new write or update to the DB, also store that data in the cache.
          1. **Pros**: data in the cache **will never be stale**
          2. **Cons**:
             1. More writes every time means it is expensive
             2. **Redundadant data**: that might never be read from the cache will also be written to the cache
       2. **Write Behind**: similar to Write through, the difference is that data is not written from the cache to the storage immediately.
          1. Instead, the cache waits for more events or timeouts and then flushes/writes everything (in bulk) to the storage.
          2. **Pros**:
             1. **Writes will look and possibly perform faster**: since we don’t write to two places simultaneously
             2. **Reduced load on storage**: since we do one-time bulk writes rather than multiple writes
       3. **Cons**
            1. **Reliability**: again how reliable is the cache, what if the data is lost before writing to the storage
            2. **Lack of consistency**: if we don’t write data from cache to DB/storage often enough (then DB will have stale data compared to cache)
6. <u>Eviction Policies</u>: when our cache becomes too large meaning it has either too many keys or takes too much memory, we have to remove some stuff. Most common policies for eviction are:
   1. **LRU (Least Recently Used)**: based on a LinkedList where the head points to the next element to be removed. How it works:
      1. Anything that has been recently accessed moves to the tail of that list, meaning its probability of eviction is low.
      2. When there is a cache-miss and a new item is added to the cache, it moves to the tail of that list and pushes the head element out.
      3. **Pros / cons:**
         1. This policy is pretty efficient and uses low resources.
         2. BUT, suffers from an issue called **False Cache Eviction** - meaning if a lot of new keys are requested at once, it may evict (in order to make space) some already popular keys
   2. **LFU (Least frequently used)**: solves the False cache eviction problem but it more complicated than LRU. How it works:
      1. Every key will have a counter which is incremented every once in a while.
      2. When there is a cache hit (that is some item from cache is requested), the counter for that item is reset (to say 0 zero)
      3. When there is a cache miss (meaning something new needs to be inserted into the cache), the algorithm picks the cache item with largest counter value and evict it to make space for the new item
      4. **Pros/cons:**
         1. Has the overhead of keeping track of the counters for each item
         2. LRU is faster, cheaper and is used as default in most cases

## Queues (more specifically Message Queues)
