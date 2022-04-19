# System Design - Theoritical Aspects

Various theories and some practical aspects of designing systems in the computer/IT/software world (_In context of Computer/Software/IT_).

- [System Design - Theoritical Aspects](#system-design---theoritical-aspects)
  - [Loading balancing and Load Balancers](#loading-balancing-and-load-balancers)
  - [CDNs (Content Delivery Networks)](#cdns-content-delivery-networks)
  - [Caching and how they work](#caching-and-how-they-work)
  - [Queues (more specifically Message Queues)](#queues-more-specifically-message-queues)
  - [Protocols](#protocols)
  - [Concurrency](#concurrency)
  - [Databases](#databases)

## Loading balancing and Load Balancers

1. Load Balancer is a machine that runs a reverse proxy software (or any software that distributes load and may provide extra features like caching etc). The goal of this software is to distribute the requests amongst multiple servers that host/run the actual application.
    1. [ASIDE] BTW, what is a reverse proxy? See [Ref](https://tinyurl.com/2p8p8cte) and some details below:
       1. A reverse proxy accepts a request from a client, forwards it to a server that can fulfill it, and returns the server’s response to the client.
       2. A reverse proxy facilitates load balancing, web acceleration (by compressing in+out data and caching common responses) and security (since the backend server/s’s identity is kept secret)
       3. Whereas deploying a load balancer makes sense only when you have multiple servers, it often makes sense to deploy a reverse proxy even with just one web server or application server. You can think of the reverse proxy as a website’s “public face.”
2. **(Most common) Strategies for Load Balancing**:

    > Most explanations are from [CloudFlare](https://tinyurl.com/yn2ts2zy)

    1. **Round robin** (Static load balancing): Round robin load balancing distributes traffic to a list of servers in rotation using the Domain Name System (DNS)
    2. **Least Connections** (Dynamic load balancing)
    3. **Resource based**: Distributes load based on what resources each server has available at the time.
       1. Specialized software (called an "agent") running on each server measures that server's available CPU and memory, and the load balancer queries the agent before distributing traffic to that server.
    4. **Weighted variants of the above 3 strategies**: basically assign a weight to each server. And, then server which has a bigger weight (i.e. is deemed to handle more traffic) will get more requests based on either round robin or least connections etc
    5. **Random** (Dynamic):
    6. **IP Hash** (Static load balancing): Combines incoming traffic's source and destination IP addresses and uses a mathematical function to convert it into a hash.
       1. Based on the hash, the connection is assigned to a specific server.

3. **TYPES of Load Balancers**: 2 Types

    > Most explanations from [nginix site](https://tinyurl.com/2p9dv8xj)

    1. **Layer 4**: operates in the Transport layer (which is 4th layer in the OSI model and has access to TCP or UDP protocols.
       1. Layer 4 load balancers make their _routing decisions based on address information extracted from the first few packets in the TCP stream, and do not inspect packet content._
       2. A Layer 4 load balancer is often a dedicated hardware device supplied by a vendor
       3. _When the Layer 4 load balancer receives a request and makes the load balancing decision, it also performs Network Address Translation (NAT) on the request packet,_ changing the recorded destination IP address from its own to that of the content server it has chosen on the internal network.
          1. Similarly, before forwarding server responses to clients, the load balancer changes the source address recorded in the packet header from the server’s IP address to its own
    2. **Layer 7**: operates in Application layer (which is the topmost layer of OSI model and basically the normal internet)
       1. Has access to everything Layer 4 has plus access to HTTP headers, cookies and payload
       2. Layer 7 load balancers base their routing decisions on various characteristics of the HTTP header and on the actual contents of the message, such as the URL, the type of data (text, video, graphics), or information in a cookie.
       3. Rather than manage traffic on a packet-by-packet basis like Layer 4 load balancers that use NAT, _Layer 7 load balancing proxies can read requests and responses in their entirety._
       4. Layer 7 are more powerful that way but are also expensive than layer 4.

4. **Advantages or usages of LBs**:
    1. **Makes our system resilient** by routing requests to available servers if one or more servers are down
    2. Makes it easy to make our system **Horizontally scalable** (that is easily plug new servers to the LBs as required for new requests)

## CDNs (Content Delivery Networks)

1. A CDN allows for the quick transfer of assets needed for loading Internet content including HTML pages, javascript files, stylesheets, images, and videos.
2. Basically, _CDNs allow you to cache your static contents like images in a globally distributed network of servers (such servers are CDNs) and quickly provide them to the users of the client website._
3. A requirement or good candidates of stuff served by CDNs are again static content that DO NOT CHANGE often.
4. **Types of CDN**: 2 types of CDN
   1. **Push CDN**: the client/original website Pushes new content to the CDNs. Might not be very efficient if you have large number of new resource/contents
   2. **Pull CDN**: is a lazy type CDN - it will only Pull (and hence cache) some content (generally static content) if there is a request for it (on the original or client’s website).
      1. Meaning the first request will be slow since the CDN hasn’t already kept of copy of a particular resource
      2. Subsequent requests will be very fast
5. **Summary of CDNs**:
   1. They help in reducing costs (since you don’t need to build a network of your own servers)
   2. They decrease latency (time by which a content is delivered to the actual user) by providing such contents from CDNs network of globally distributed server (whichever is the closest serves that content)
   3. One disadvantage (may be more) is that implementing CDNs increases the complexity of your system

## Caching and how they work

1. To be cost-effective and to enable efficient use of data, caches must be relatively small.
2. Why caching? To improve access to common data faster based on where you need it like so Code/Program (fastest) > Memory > Disk > Network > …
3. **Pros of caching:**
   1. Improve read performance (aka **lowers Latency**)
   2. Reduce the load (aka **increases Throughput**)
      1. **Latency** indicates **how long it takes** for packets to reach their destination. **Throughput** is the term given to **the number of packets** that are processed within a specific period of time.
      2. Lower latency goes with faster throughput, and higher latency goes with slower throughput.
4. **Cons of caching:**
   1. **Increases complexity**: overhead of adding and maintaining the cache
   2. **Introduces inconsistencies**: if cache has stale data than actual data store in the disk or server or DB etc
   3. **Consumes resources**: you need additional space or dedicated hardware or software to maintain a cache within memory or disk or elsewhere
5. **Caching strategies** (most common are follows:)
   1. **For Reads:**
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
   2. **For Writes**
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
6. **Eviction Policies**: when our cache becomes too large meaning it has either too many keys or takes too much memory, we have to remove some stuff. Most common policies for eviction are:
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

1. Similar to a DB, a queue is a sophisticated piece of software that runs on some dedicated hardware
2. Queues are used to deliver messages asynchronously which means that we simply send the message and don’t wait for a response.
3. For example, for a Pizza shop - a queue sits between the “Pizza Service” (possibly a web app with some api) and the “Payments Service”.
   1. Whenever, a customer orders a pizza via the Service, the Service puts a new message on the queue which implies (indirectly) for the Payments service to process it.
   2. This way our Service (web app etc) is de-coupled from the Payments Service and allows for massive ordering
4. When any service connects to a Queue, it must choose one of two roles- it can either be a Producer or a Consumer.
5. This allows for **total de-coupling between Producers and Consumers** because:
   1. A Producer doesn’t care if it is connected to a Consumer and vice-versa, all a producer (or a consumer) care about is if it is connected to a Queue.
   2. Producers and consumers can be implemented in totally different languages and libraries and still be able to communicate.
6. **Queues pros and cons**
   1. **Pros**:
      1. Queue acts as buffer for messages/requests for the consumer
      2. Request **spikes are smoothed** because of this buffering
      3. **Message durability**: as queues are able to preserve messages even if consumers go down and some messages aren’t delivered
   2. **Cons**
      1. **Increased system complexity** to add new hardware and software plus communication with other pieces
      2. **Increases latency** since message now have to go via the Queue
      3. **Lower reliability of sorts**: since the whole communication is asynchronous, we won’t know in real-time what happened to messages we inserted from the Producer (for example, my payment is in the Queue but has it been delivered to the payment system - there is no way to get instant feedback)
7. **Messaging Paradigms or Model**: (or how messages can be delivered from a producer to a consumer using these queues). There are two main ways:
   1. **Message Queue**: when the queue ensures a specific message is sent to only one consumer.
      1. For example we don’t want a user’s payment to be sent to multiple consumers and the user being charged multiple times hence the payment information is sent to one consumer for processing.
      2. **Features of message queue**:
         1. Suited to execute some “Action” like making a payment once
         2. Each message will be delivered exactly once to one Consumer and that is it
         3. The queued messages can arrive Out of Order since this has retry mechanism => that is if the selected consumer is dead, the queue retries (sometime later) to some the same or another available consumer.
   2. **Publisher/Subscriber (aka Pub/Sub)**: used when we want to notify multiple services (I.e. consumers in this case) about an event that happened.
      1. For example, the Payments Service (once the payment is successful) might send messages (via another Queue) to the Receipts Service and the Billing Service.
      2. **Features of pub/sub:**
         1. Suited to provide “Notifications” to multiple consumers
         2. Theory is “At least once delivery” (could send multiple messages to one or more connected consumers).
         3. Messages are always delivered In Order since there are no Re-tries.
8. What is an **Exchange in the Message queuing world**?
   1. It is a router or a load balancer that receives all messages and then puts those messages in the right queue for further processing.
   2. It supports three 3 ways of working:
      1. **Direct**: directly put a given message into the queue with the specified Routing-Key. In this mode a queue, acts like a “Message Queue”.
      2. **Topic/Header**: In this mode a queue, acts like a “Message Queue”.
         1. **Topic exchange** means the queue and consequently the exchange is shared (divided into parts) for example based on a region or state etc. The topic name would be sort of a composite of the <topic-name>:<shard-name>
         2. **Header exchange** is same as topic exchange but the routing is done based on the information in the header rather than the topic-name.
         3. **Fan-out**: spreads the message to all the consumers, with each consumer receiving the same message. In this mode, a Queue becomes a Pub/Sub.
9. **What is a Routing-key?**
   1. It identifies to which queue should the message from a producer go it.
   2. Routing keys can be complex and carry more than the name/id of the queue (like also carry stuff like user data etc)
10. What is a **Channel in queuing world? -> this concept provides Concurrency**
    1. A channel is basically multiple paths or channels built out of a single TCP connection between a Queue and each of it consumer.
    2. This allows a single service to process many more messages without the overhead and costs of building multiple TCP connections.
11. How do **Acknowledgements work in queuing world? -> this provides Reliability**
    1. Basically, when should a message be deleted from a queue. Mainly two ways:
        1. **Automatic**: is when a queue deletes a message after it got delivered.
           1. But, this comes with a con that if delivering a message takes time and the consumer crashes in the meantime and if the queue deleted the message automatically, then that message will be lost.
        2. **Explicit**: is when we have the consumer explicitly tell the queue that it got the message and that the message can now be safely deleted.
           1. This way is Reliable but more Complex than the Automatic way above
12. **Some concrete messaging systems:**
    1. **RabbitMQ**
        1. Based on AMQP protocol. And  best used as a Message Queue.
        2. Messages are stored until consumer retrieves them and then they are deleted
        3. Suited for running heavy-processing tasks in offline fashion like payment processing, manipulating images etc
        4. Can also be used for Distributing tasks (amongst multiple servers)
        5. Please note the _concepts from bullet points 8 (Exchange) to 11 (Acknowledgements) more specifically apply to RabbitMQ_ (but also in general to other queueing systems/products)
    2. **Kafka**
       1. Is the most popular pub/sub system currently. And, it calls itself an Event-stream platform.
       2. It also has concepts of Producers and Consumer but since its pub/sub the messages are not deleted after they are consumed instead they are deleted after a period of time.
       3. **Some important terms in Kafka world:**
           1. In Kafka, queues are called “Topics” and messages are called “events”. The topics are usually sharded or or in Kafka terms “Partitioned”
           2. The consumers of a given topic divide partitions amongst themselves. Some consumer might be subscribed to more than one partitions and other might only subscribe to one partition.
           3. Each event has a key, value and timestamp.
           4. **Consumer groups and how they help in partitioning:**
               1. Basically, Kafka distributes a topic’s partitions across various brokers
               2. A Consumer group is a group of consumers which read from exclusive set of partitions. Basically, a group is formed to read data from various brokers but from different partitions.
                   1. For example, Receipts service might be a consumer group reading from only those partitions (that might be store in different brokers or servers) that are relevant to creating user receipts.
               3. And, Kafka stores the offsets at which each consumer in a consumer group has been reading. As the consumer continues reading the offset value is increased.

## Protocols

1. _**TCP protocol (Transmission Control Protocol)**_
   1. Diagrammatic overview of TCP:
    <img src="/assets/img/TCP-Overview.png" alt="TCP Overview" />
   2. **Pros and Cons**
      1. **Pros**:
         1. **Reliable**: a message from sender to receiver is usually broken down into smaller “packets”.
            1. TCP is reliable in the sense that when the sender sends a packet (using TCP) to the receiver (and it starts a new timer) ->
            2. then, either the receiver acknowledges that it received the packet or if a timeout (of that timer set by the TCP) occurs, the sender will re-send the message again.
         2. **Ordered**: TCP orders (i.e. assigns numbers) to each pieces/packets of the message being sent.
            1. The receiver uses these numbers to identify the packets and if something seems missing and out of order, then it will ask the sender to re-send that particular packet.
         3. **Error-checked**: means Tcp adds a checksum for each packet (to ensure the integrity of each packet).
            1. A checksum is a number that is calculated based on the actual content of the message.
            2. When the receiver gets the packet, it computes its own version of checksum (of course using the same method as sender) and compares it with the checksum from the sender.
            3. If the checksums are different, the receiver can ask the sender to re-send that packet.
      2. **Cons**
         1. Slower than some other protocols
2. **UDP protocol (User Datagram Protocol)**
   1. UDP doesn’t have any complexity of TCP, it doesn’t wait for ACK and doesn’t number the packets
   2. Since it does less work, it is faster than TCP but it less reliable than TCP. That is the trade-off.
   3. **When to use UDP then?**
      1. UDP is good for a constant stream of data where the speed is more important than losing some tiny amount of data
      2. _In a system design interview, “sending a lot of data FAST” most probably means choosing UDP_
      3. **Some example usages:**
         1. **Monitoring metrics**: like CPU usage very fast like 100 times a second which means losing a few readings doesn’t matter much
         2. **Stock trading**: is also a similar example to monitoring from above since in most cases stock values fluctuates many times per second or minute so losing a few data points here and there wouldn’t be that problematic
         3. **Video streaming**: our video gets bit fuzzy at times means the browser might be losing some data but doing its best to make sense of remaining video data
         4. **Gaming**: for example the game or your opponents video/frame freeze for split seconds every now and then
3. _**HTTP (hypertext transfer protocol)**_
   1. Http is based on TCP where hypertext is any text containing links to other documents
   2. It is a request and response based protocol based on Application Layer 7 of OSI model.
   3. **Request contains**: Method, URL, Headers and optionally a Body
   4. **Response contains**: Status, Header and optionally a Body
   5. Sometimes POST is used instead of GET since GET has a limit on how much data it can send via its query string (stuff after ? Symbol) plus since POST puts everything inside a body, it is more secure from prying eyes that way.
   6. **Http Status codes (common ones)**
      1. 100-199 : Informational
         1. 200-299: Successful
            1. 201: Created
         2. 300-399: Redirection
            1. 301: moved permanently (to another webpage address)
         3. 400-499: Client error
            1. 401: Unauthorized - meaning client is not authenticated yet
            2. 403: forbidden - client doesn’t have correct permission for this page even after logging in
            3. 404: Not found - client is asking for a page or entity that doesn’t exist on the server
         4. 500-599: Server error
   7. **HTTP Verbs:**
        1. GET
        2. _POST: is not idempotent_ (repeating a Post will execute a new action for example create a new user)
        3. _PUT: is idempotent_ (and hence more suitable to do things like status (say pending, delivered etc) updates

4. _**REST architecture**_
    1. Technically, REST is not really protocol since it doesn’t have a strict definition - basically tells you how to structure your API
    2. **Features of REST**
        1. Built on Http
        2. It standardizes the URL structure as well as emphasizes on proper use of HTTP verbs
    3. **REST in practice or Restful-ness**
        1. Write URL parameters in pairs
           1. For example DELETE libraries/<library-id>/books/<book-id>
        2. Stick to proper HTTP verbs usage like GET for Read, POST for Create, PUT for Update and so on..
        3. **URL structure (of each pair): first part is resource name is plural** (for example users or accounts) and second part is the ID of the actual resource
        4. BUT, if you need deeper nesting (number of pairs in the URL) than 2, then think about possibly using a different protocol like GraphQL
        5. If required, you should also be able to provide Pagination and Sorting capabilities via your api like so
           1. GET /books?page=2&limit=25&sort=<fieldName>

5. _**WebSockets Protocol**_
    1. It is based on Sockets which sits on the 5th layer of the OSI model called Session layer.
    2. WebSocket is a communication protocol which features bi-directional, full-duplex communication over a persistent TCP connection
    3. **When/why to use web-sockets?**
        1. In such cases where is constant polling or checking to see if something is returned and
        2. HTTP’s way of sending a request and then have to constantly check for response is very wasteful
    4. **How does it work?**
        1. WS is built on top of Tcp and is a Duplex protocol - the client has to still connect to the server but only once
        2. But, once the connection is established, both the server and the client can send messages simultaneously
        3. The server can send messages without client asking (polling) for it and the server doesn’t have to respond
    5. **Cons of WS**
        1. More complicated than http and may not be supported in all languages
        2. NOTE: WS (unlike http) is a Stateful protocol which means if there is a network error, WS connection will be dropped and you will have to reconnect by yourself
        3. Stateful vs Stateless Protocol
            1. <https://www.interviewbit.com/blog/stateful-vs-stateless/>
            2. the major feature of **stateful is that it maintains the state of all its sessions**, be it an authentication session, or a client’s request for information. Stateful are those that may be used repeatedly, such as online banking or email. **They’re carried out in the context of prior transactions** in which the states are stored, and what happened in previous transactions may have an impact on the current transaction
        4. Load balancers may have some troubles since WS create long-lived connections unlike short-lived connections that they are normally used for
        5. Doesn’t have standard verbs like HTTP
    6. **REST vs WS**
       1. Please check this excellent resource on [geekforgeeks](https://www.geeksforgeeks.org/difference-between-rest-api-and-web-socket-api/).

6. _**Long Polling**_
    1. It is not really a protocol but more of an architectural pattern. It serves as an alternative to using WebSockets.
    2. Major problem with using Http for applications like chatting is the constant need to open and close connections. The gap between connections are major waste of time.
    3. **WS vs Long Polling**
        1. The notes below are based on this [webpage](<https://dev.to/kevburnsjr/websockets-vs-long-polling-3a0o>).
        2. **What is long polling?**
            1. Long Polling is a near-real-time data access pattern that predates WebSockets.
            2. _A client initiates a TCP connection (usually an HTTP request) with a maximum duration (ie. 20 seconds). If the server has data to return, it returns the data immediately, usually in batch up to a specified limit._
            3. _If not, the server pauses the request thread until data becomes available at which point it returns the data to the client._
            4. WebSockets are Full-Duplex meaning both the client and the server can send and receive messages across the channel. BUT,
               1. Long Polling is Half-Duplex meaning that a new request-response cycle is required each time the client wants to communicate something to the server.
        3. **Generally, WebSockets will be the better choice because** based on this [link](<https://ably.com/blog/websockets-vs-long-polling>):
            1. Long polling is much more resource intensive on servers whereas WebSockets have an extremely lightweight footprint on servers.
            2. Long polling also requires many hops between servers and devices
    4. **Pros of long polling**
            1. Real time message delivery just like WS
            2. Good alternative to WS if WS aren’t an option
    5. **Cons of long polling**
        1. May be hard to implement in some languages and frameworks.
            1. For example in RoR, the connection is controlled by the framework so there is no easy way to keep connections open until a new message comes in.
        2. If your web server is based on threads or processes, maintaining too many long connections is difficult

7. _**gRPC ()**_
   1. But First what is **RPC (Remote Procedure Call)** ?
        1. Invoke another service as if it is a local function. The function is described in an abstract language called Interface Description Language (IDL)
        2. **Diagrammatic representation** - How does RPC call work?
            <img src="/assets/img/how-RPC-works.png" alt="sample of how RPC works" />
        3. Now, **what is gRPC**?
            1. Developed by Google
            2. It _uses “Protocol Buffer” or ProtoBuf as IDL (to encode and decode the messages) and HTTP2 for transportation of messages_
            3. Main drawback is that it _cannot be used in the Browsers and mobile_
        4. **What is Protobuf ?**
            1. It is a Binary Protocol and so is not human readable (unlike say JSON format)
            2. The main advantage is protobuf messages are much smaller than JSON or XML and hence it is much faster in general
            3. How does gRPC work?
                <img src="/assets/img/how-gRPC-works.png" alt="sample of how gRPC works" />
8. _**GraphQL**_
    1. Developed by Facebook in 2015. **Built to solve two problems that REST has**:
        1. **Overfetching**: for example we fetched a whole user object over the network when we only needed user’s name
        2. **Underfetching**: for example when we need a complete object from a REST api but we are only pulling in the id or name from that API.
           1. Say we pull a user’s name and his friends list. A friends list is most probably only a bunch of IDs of that user’s friends.
    2. **What is GQL?**
            1. GQL is based on Http and its request and response are in JSON format
            2. GQL let you define which fields to return as well as which nested entities to return
            3. **GQL Query** lets you select data whereas **GQL Mutation** lets you update/modify data using Post, Put etc verbs
    3. **Cons of GQL**
        1. Since GQL uses POST requests, the results are less cacheable
            1. **_ASIDE - why are POST requests not cacheable?_**
                1. **POST requests are not cacheable by default** but can be made cacheable if either an Expires header or a Cache-Control header with a directive, to explicitly allows caching, is added to the response.
                2. Responses to PUT and DELETE requests are not cacheable at all
                3. But HTTP **caching is applicable only to idempotent requests**, which makes a lot of sense; only idempotent and nullipotent (an action which has no side effect. Queries are typically nullipotent: they return useful data, but do not change the data structure queried.) requests yield the same result when run multiple times.
                4. When caching is enabled, the data is retrieved from the browser cache instead of from the business object on the server.
        2. Support outside JS ecosystem is still not great
        3. Uses JSON object by default which can get bulky in size

9. _**Also, comparing REST vs GraphQL vs gRPC ?**_
    1. **Roughly, which protocol to use based on following scenarios:**
        1. Based on **whether the API will be external** (meaning are the APIs on the server being used by external clients)
           1. YES (means these are good to use): REST, WebSockets, GQL, UDP, Long Polling
           2. NO (means this protocol is not very suitable): gRPC since external clients are usually browsers and gRPC doesn’t support browser/mobile clients
        2. Based on **if the communication need to be Bi-Directional** or we could just use a request-response model like in Http
           1. YES: Websockets, UDP, (somewhat) Long Polling
           2. NO: Rest, grpc, gql
        3. Based on **if we need High Throughput** (process thousands of messages per second)
           1. YES: websockets, grpc, UDP (faster than other two but less reliable in this case)
           2. NO: Rest, gql, long polling
        4. Based on if you need **Browser and/or Mobile support** ?
           1. YES: rest, websockets, gql, long polling
           2. NO: grpc, UDP
    2. Another excellent article by [danhacks](<https://www.danhacks.com/software/grpc-rest-graphql.html>).

## Concurrency

1. **Concurrency !== Parallelism (since)**
    1. **Parallelism** - _actually doing more than one thing at the same time_. Tasks are done independently from each other.
    2. **Concurrency** - _providing an illusion of doing more than one thing_ at the same time. Tasks are done by switching amongst them with something like a back-and-forth.
2. **Processes**:
    1. Remember, each process has its own separate memory space. One process isn’t allowed to access the memory of another process.
    2. Also, with a programming language like Java and its JVM, you will always see one process no matter many CPU cores your machine has.
         1. BUT, **with a library like NodeJS you may see as many process as the number of  CPU cores**. That means for a processing/CPU intensive task, you will consume a lot more memory than a language or library that uses a single process like Java.
    3. [ASIDE] It means that **too many (or too lengthy) CPU-intensive tasks could keep the main thread too busy to handle other requests, practically blocking it**. The Node. js execution model was designed to cater to the needs of most web servers, which tend to be I/O-intensive.
    4. Please read this superb [article](<https://yarin.dev/nodejs-cpu-bound-tasks-worker-threads/>) on Node, its workings, and writing CPU intensive tasks.
    5. Interprocess communication: few common ways
        1. File (also memory mapped file)
        2. Signal (using something like kill -9 <process-id>)
        3. Socket (also Unix Domain Socket)
        4. Pipe (like so ls | grep ‘hello’)
3. **Threads:**
    1. Is the thing that executes your code.
    2. Every process will create at least one thread but there will usually more than one threads created
    3. **VVIP: JS is said to be single-threaded but it actually means that a developer is able to (for code execution) utilize only a Single Thread per Process. But, there are still other threads for garbage collection, networking etc**
    4. **Stack and Heap**
        1. Stack: **every thread has a stack** which stores the local variables as well as method parameters and the call chain.
        2. **Heap: every process has all its memory** in something called a heap.
            1. A Heap is shared between all running threads and this is what causes a lot of multithreaded problems.
    5. **Threads overhead (or the problem of using the thread-per-request model)**
        1. Spawning a new thread is relatively slow and costly
        2. OS limits the number of threads
        3. Each thread consumes memory and each OS or maybe the language/platform limits the memory allocated for threads. This means you can hit the memory limit even before you hit the number of threads limit
        4. **Thread Contention**: different threads contend or compete for resources with the shared memory space (so called Heap).
            1. Some issues are:
               1. **Race condition**: is when different threads (of the same process) try to write different values to the same memory location at the same time
               2. The solution of race and similar conditions are:
                    1. Locks
                    2. CPU time
                    3. Shared resources:
4. **Thread Pools:**
   1. It is basically the concept wherein threads are created as part of a group/bulk if more threads can be created.
   2. Otherwise, the thread pool will ask the requesting process to wait while a thread within that pool becomes available.

## Databases

   1. **Indexes - what, why and how ?**
        1. An index is created per column (generally a column which accessed more frequently, for example you search the users table by name column)
        2. If there is no index for a commonly accessed column (for example name of the users table), then the DBMS will have to do a full table scan searching the entire table row-by-row
        3. But, if there is an index, the Db will check the index first and get the rows with this index.
        4. [ASIDE] **How does indexing work?** See this [ref](<https://chartio.com/learn/databases/how-does-indexing-work/>) for the details below.
            1. Indexing adds a data structure with columns for the search conditions and a pointer
            2. The pointer is the address on the memory disk of the row with the rest of the information
            3. The index data structure is sorted to optimize query efficiency
            4. The query looks for the specific row in the index; the index refers to the pointer which will find the rest of the information
        5. More on creating indexes taking postgres as example (reference is [tutorialspoint](https://www.tutorialspoint.com/postgresql/postgresql_indexes.htm)):
           1. Simply put, an index is a pointer to data in a table. An index in a database is very similar to an index in the back of a book.
           2. For example, if you want to reference all pages in a book that discusses a certain topic, you have to first refer to the index, which lists all topics alphabetically and then refer to one or more specific page numbers.
           3. An index helps to speed up SELECT queries and WHERE clauses; however, it slows down data input, with UPDATE and INSERT statements.
           4. Indexes can be created or dropped with no effect on the data.
           5. A syntax of multicolumn index creation will look like so:

            ```
                CREATE INDEX index_name
                ON table_name (column1_name, column2_name);
            ```

           6. Whether to create a single-column index or a multicolumn index, take into **consideration the column(s) that you may use very frequently in a query's WHERE clause** as filter conditions.
           7. The following guidelines indicate **when to avoid creating indexes**:
              1. Indexes should not be used on **small tables**.
              2. Tables that have **frequent, large batch update** or insert operations.
              3. Indexes should not be used on columns that contain a **high number of NULL values**.
              4. **Columns that are frequently manipulated** should not be indexed.
   2. **Sharding (in databases)**
        1. Most RDBMS have limitation of how big a single instance is allowed to get, sharding is one of the solutions to that limitation.
        2. Sharding is (see [ref](<https://www.mongodb.com/features/database-sharding-explained>)):
            1. **Sharding is a method for distributing a single dataset across multiple databases, which can then be stored on multiple machines.**
            2. This allows for larger datasets to be split in smaller chunks and stored in multiple data nodes, increasing the total storage capacity of the system.
            3. **Sharding is a form of scaling known as horizontal scaling or scale-out, as additional nodes are brought on to share the load.**
               1. Horizontal scaling allows for near-limitless scalability to handle big data and intense workloads.
        3. Another description of sharding from [digitalocean](https://www.digitalocean.com/community/tutorials/understanding-database-sharding):
           1. Sharding is a database architecture pattern related to horizontal partitioning — the practice of separating one table’s rows into multiple different tables, known as partitions.
           2. Each partition has the same schema and columns, but also entirely different rows. Likewise, the data held in each is unique and independent of the data held in other partitions.
           3. Sharding involves breaking up one’s data into two or more smaller chunks, called logical shards. The logical shards are then distributed across separate database nodes, referred to as physical shards, which can hold multiple logical shards.
           4. Despite this, the data held within all the shards collectively represent an entire logical dataset.
        4. The main problems to solve with sharding is (a) how to split up the datasets, and (b) how does application know which machine/node to get its data from.
        5. There are two main approaches to **sharding of DBs: Tenet based and Hash based sharding**:
            1. **Tenet based**: **also called geo-sharding**, has the pro of being easy to create and reason about
                1. Tenet based is good for systems that have clear separation between entities.
                2. For example, in a ride-hailing app like Uber, a driver will only drive one car but travel between multiple countries. The system thinks the driver has to register for each country -> so we create a DB shard for each country.
                3. Cons of Tenet based is mostly about Uneven Distribution (some shards might still be too big to handle).
            2. **Hash based:**
                1. In this approach, **we create multiple databases, and each time we need to perform an operation on a DB entity, we first calculate in which shard this entity is located**.
                2. We do this by applying a hash function to the primary identifier of this entity like so:
                    1. `hash-fxn(primaryId) % numOfShards`
                3. **Pros** of hash based sharding: (a) gives Even Distribution (b) works well for key-value data
                4. **Cons** are: (a) adding new shards is difficult since we have to change the hashing function.
        6. **Downsides of Sharding**
            1. **Added complexity** (since each shard is a new machine that needs to be allocated memory, CPU etc)
            2. Data may still become **unbalanced** (say using Tenet based sharding)
            3. Cross (multiple querying) shard operations are expensive.
   3. **Consistent Hashing** (What and Why is it needed)
        1. Say we have to create new shards (re-shard) as our data grows (particularly if we are using the hash-based sharding strategy)
        2. This results in a problem wherein a lot of the entities have to be moved around since their hashed values will change.
        3. **With consistent hashing, we insert a new shard in the middle of the currently existing shards so we don’t have to move a lot of entities around**
        4. A useful, quick video intro is <https://www.youtube.com/watch?v=ffE1mQWxyKM>
   4. **What is sharding vs partitioning?** Ref from [hazelcast](<https://hazelcast.com/glossary/sharding/>)
        1. Sharding and partitioning are both about breaking up a large data set into smaller subsets. **The difference is that sharding implies the data is spread across multiple computers while partitioning does not.**
        2. **Partitioning is about grouping subsets of data within a single database instance.**
   5. **Partitioning**
        1. Sharding is breaking one large database into smaller databases whereas **Partitioning is breaking one large table into smaller tables and the splitted tables are still in the same database**.
        2. **Benefits of partitioning** (consider everything like a table or an index like a file)
            1. Smaller files (I.e partitions) means faster queries
            2. The indexes will be smaller for a smaller table and can easily fit into memory
            3. Dropping partitions is faster
        3. **Partitioning strategies: 3 of them are -**
            1. **List of values**: like the status of an order like placed, dispatched, delivered etc.
                1. This partitioning by a list of values might still produce Uneven Data distribution (which is a con).
                2. Another con is having to move data between tables when their values/status changes.
            2. **Range of dates**: stored data based on weeks or months or years as required by the business
                1. **Pro** is that this strategy is great if you want to get rid (totally drop, rather than delete) of old and less useful historical data say from last year
                2. **Con** is data might still be unevenly distributed since some dates might have more data than others for example people buy more during Christmas or Thanksgiving months
            3. **Hash of a key**: similar to sharding by hash method
               1. Lets pick a key say the primary key of a table
               2. We define how many smaller tables we want to have
               3. Then we put the entry/row based on a formula like so:
                 `hashFxn(id) % numOfSmallerTables`
        4. **General downsides of Partitioning**
            1. **Maintenance complexity**: updating/deleting partitions takes extra time. Some RDBMS might not support them at all (for MySQL doesn’t and PostGre does very well)
            2. **Scanning** all partitions is **expensive**
            3. **Harder to maintain uniqueness**: since its a bunch of smaller tables -> some tables might have same primary key using something like UUID
