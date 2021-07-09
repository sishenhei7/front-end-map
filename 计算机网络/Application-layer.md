# Application layer

## Principles of network applications

1. There are two predominant architectural paradims used in modern networks: the client-server architecture and the peer-to-peer architecture.
2. In the jardon of operating systems, it is not actually programs but processes that communicate. Process on the same system can communicate with interprocess communication. And process on two different end systems communicate with each other by exchanging messages across the computer network.
3. A process sends messages into, and receives messages from, the network through a software interface called a socket. The application developer has control of everything on the application-layer side of the socket but has little control of the transport-layer side of the socket.
4. To identify the receiving process, two pieces of information need to be specified: (1)the address of the host and (2)an identifier that specifies the receiving process in the destination host.
5. A socket is the interface between the application process and the transport-layer protocol.
6. Applications that have throughput requirements are said to be bandwith-sensitive applications. Elastic applications can make use of as much, as little, throughput as happens to be available.
7. The internet(and more generally, TCP/IP networks) makes two transport protocols available to applications, UDP and TCP.
8. In our brief description of TCP and UDP, conspicuously missing was any mention of throughput and timing guarentees. But today's internet can often provide satisfatory service to time-sensitive applications, but it cannot provide any timing or throughput guarentees.

## The Web and Http

1. Although HTTP uses persistent connections in its default mode, HTTP clients and servers can be configured to use non-persistent connections instead.
2. The round trip time(RTT) is the time it takes for a small packet to travel from client to server and then back to the client.
3. Non-persistent connections have some shortcomings: (1)For too many connections, TCP buffers must be allocated and TCP variables must be kept in both the client and server. (2)Each object suffers a delivery delay of two RTTs: one to establish the TCP connection and one RTT to request and receive an object.
4. HTTP is written in ordinary ASCII text.
5. A request message includes a request line, some header lines and an entity body.
6. A response message includes an initial status line, some header lines and an entity body.
7. A HTTP request meassage is called conditional GET message if (1)the request message uses the GET method and (2)the request message includes an If-modified-since header line.

## Electronic Mail in the Internet

1. If Alice's server cannot deliver mail to Bob's server, Alice's server holds the message in a message queue and attempts to transfer the message later.
2.





