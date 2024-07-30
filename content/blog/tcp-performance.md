+++
title = 'TCP Performance Considerations'
date = 2024-07-12T18:44:34+05:30
draft = false
showToc = true
cover.image = '/images/tcp.jpeg'
tags = ["Web"]
+++

HTTP is the language of the internet. And it is layered directly on top of TCP, so the performance of HTTP calls depends directly on the performance of the underlying TCP.

This article highlights some significant performance considerations of TCP connections. You'll better appreciate HTTP's connection optimization features after understanding these basic performance characteristics of TCP.

## HTTP Transaction Delays

There are several possible causes of delay in an HTTP transaction:

1. The client must determine the web server's IP address and port from the URI. If the hostname is new, DNS resolution can take several seconds.
2. The client sends a TCP connection request and waits for the server's acceptance. This setup delay usually takes a second or two but can add up with multiple transactions.
3. Once connected, the client sends the HTTP request. The server reads and processes the request, which takes time as the data travels over the Internet.
4. The server then sends back the HTTP response, which also takes time.

The impact of these TCP network delays depends on hardware and server mostly, but these are also significantly affected by technical intricacies of the TCP protocol.

## Performance Focus Areas

In this secion we'll go through the most common TCP related delays.

### 1. TCP Connection Handshake Delays

When you set up a new TCP connection, even before you send any data, the TCP software exchanges a series of IP packages to negotiate the terms of the connection.

These are the steps in the TCP connection handshake:

![TCP connection handshake](/images/tcp_handshake.png)

> _Figure 1. TCP Connection handshake_

1. To request a new TCP connection, the client sends a small TCP packet (40–60 bytes) with a “SYN” flag to the server (see Figure 1a).
2. If the server accepts, it sends back a TCP packet with “SYN” and “ACK” flags, indicating the connection is accepted (see Figure 1b).
3. Finally, the client sends an acknowledgment to the server, confirming the connection is established. Modern TCP stacks allow sending data in this acknowledgment packet (see Figure 1c).

The HTTP programmer never gets to see these packets, they are managed invisibly by the TCP/IP software. All they see is a delay.

### 2. Delayed Acknowledgements

The Internet doesn't guarantee reliable packet delivery, so **TCP uses its own acknowledgment scheme** to ensure data delivery. Each TCP segment has a sequence number and checksum. Receivers send small acknowledgment packets back to the sender for intact segments. If an acknowledgment isn't received in time, the sender resends the data.

**Piggybacking**: TCP can piggyback acknowledgments on outgoing data packets to optimize network use. **Delayed Acknowledgments**: TCP stacks may delay acknowledgments (100–200ms) to piggyback on outgoing data. If no data packet arrives in that time, the acknowledgment is sent alone.

**HTTP Limitation**: HTTP's request-reply nature reduces piggybacking opportunities, potentially causing delays. **Adjusting TCP Settings**: Modifying TCP parameters should be done cautiously. **TCP algorithms protect the Internet from poorly designed applications**. Ensure changes won't create problems.

### 3. TCP Slow Start

**TCP performance** depends on the age of the TCP connection. **TCP connections "tune" themselves** over time, initially limiting the speed and increasing it as data is transmitted successfully. This tuning, called **TCP slow start**, prevents sudden overloading and congestion.

**TCP slow start** throttles the number of packets a TCP endpoint can have in flight. Each time a packet is received successfully, the sender can send two more packets. For large HTTP transactions, data is sent incrementally: one packet, then two, then four, etc., **"opening the congestion window."**

New connections are slower than "tuned" connections that have already exchanged data. Since tuned connections are faster, **HTTP allows reuse of existing connections**.

### 4. Nagle’s Algorithm and TCP_NODELAY

TCP allows streaming of any data size, even a single byte, but this can degrade network performance due to the 40 bytes of flags and headers in each TCP segment. **Nagle’s algorithm**, described in [RFC 896](https://datatracker.ietf.org/doc/html/rfc896), **bundles TCP data** to improve network efficiency by sending full-size packets (~1,500 bytes on a LAN or a few hundred bytes across the Internet). It sends non-full-size packets only if all other packets are acknowledged. Otherwise, data is buffered until enough accumulates for a full packet or pending packets are acknowledged.

**HTTP performance issues** arise because small HTTP messages may be delayed while waiting for additional data. Nagle’s algorithm interacts poorly with delayed acknowledgments, causing further delays. **HTTP applications often disable Nagle’s algorithm** by setting the **TCP_NODELAY** parameter, but this requires sending large data chunks to avoid many small packets.

### 5. TIME_WAIT Accumulation and Port Exhaustion

**TIME_WAIT port exhaustion** is a serious issue affecting performance benchmarking but is rare in real deployments. When a TCP connection closes, a control block with IP addresses and port numbers is kept in memory for about 2MSL (often two minutes) to prevent duplicate packets from interfering with new connections using the same values.

**Benchmarking issues** arise because test load-generation computers often limit the number of client IP addresses connecting to a server. The server usually listens on port 80, restricting available connection combinations. In a setup with one client and one server, only the source port can change, limiting connections to 60,000 / 120 = 500 transactions/sec. If server performance caps at around 500 transactions/sec, **check for TIME_WAIT port exhaustion**. Solutions include using more client load-generator machines or rotating through multiple virtual IP addresses.

**Managing open connections**: Even without port exhaustion, having many open connections or control blocks can slow down some operating systems.

## Conclusion

Understanding the factors that affect the speed of HTTP transactions can help improve their performance. From resolving server addresses to establishing connections and managing data flow, each step can introduce delays. By being aware of these potential slowdowns and taking steps to manage them, developers can make their applications run more efficiently.
