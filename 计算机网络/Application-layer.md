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
2. It is important to observe that SMTP does not normally use intermediate mail servers for sending mail, even when the two mail servers are located at opposite ends of the world.
3. We mention here that SMTP uses persistent connections: if the sending mail server has several messages to send to the same receiving mail server, it can send all of the messages over the same TCP connections.
4. There are some differences between HTTP and SMTP: (1)HTTP is mainly a pull protocol; SMTP is primarily a push protocol. (2)SMTP requires each message, including the body of each message, to be in 7-bit ASCII format. (3)HTTP encapsulate each object in its HTTP response message; SMTP places all of the message's object to one message.
5. SMTP is used to transfer mail from the user agent to the sender's mail server; SMTP is also used to transfer mail from the sender's mail server to the recipient's mail server; A mail access protocol such as POP3、IMAP is used to transfer the recipient's mail server to the recipient's user client.
6. With the TCP connection established, POP3 progress through three phrases: authorization, transaction and update.
7. A user agent using POP3 can often be configured to 'download and delete' or to 'download and keep'.
8. POP3 does not provide any means for a user to create remote folders and assign messages to folders. Whereas the IMAP protocol can. Another feature of IMAP is that it has commands that permit a user agent to obtain components of messages.
9. More and more users today are sending and accessing their e-mail through their Web browsers.

## DNS: The internet's directory service

1. The DNS is (1)a distributed database implemented in a hierarchy of DNS servers. (2)an application-layer protocol that allows hosts to query the distributed database. The DNS protocol runs over UDP and uses port 53.
2. DNS provides a few other important services in addition to translating hostnames to IP addresses: (1)host aliasing. (2)mail server aliasing. (3)load distribution: using DNS rotating.
3. To a first approximation, there are three classes of DNS servers: root DNS servers, top-level domain(TLD) DNS servers and authoritative DNS servers. There is another DNS server called the local DNS server.
4. When a host makes a DNS query, the query is sent to the local DNS server, which acts a proxy, forwarding the query into the DNS server hierachy: (1)The host first sends a DNS query message to its local DNS server. (2)The local DNS server forwards the query message to a root DNS server. (3)The local DNS server then resends the query message to one of the TLD servers. (4)The local DNS server resends the query message to the authoritative DNS server to get ip address. Note that in this process, eight DNS messages are sent.
5. A DNS resource record is a four-tuple: (Name, Value, Type, TTL)
6. TTL is the time to live of the resource record; it determines when a resource should be removed from a cache.
7. When Type=NS, then Name is a domain and Value is the hostname of an authoritative DNS server that knows how to obtain the IP address for hosts in the domain.
8. The nslookup program can send DNS messages.

## Peer-to-Peer File Distribution

1. The distribution time is the time it takes to get a copy of the file to all N peers. In client-server distribution, the distribution time increases linearly with the number of peers N; whereas applications with the P2P architecture can be self-scaling, which is a direct consequence of peers being distributors as well as consumers of bits.
2. In BitTorrent lingo, the collection of all peers participating in the distribution of a particular file is called a torrent. Peers in a torrent download equal-size chunks of the file from one another, with a typical size of 256 KBytes.
3. Each torrent has an infrastructure node called tracker. When a peer joins a torrent, it registers itself with the tracker and periodically informs the tracker that it is still in the torrent. In this manner, the tracker keeps track of the peers that are participating in the torrent.
4. To determine which to request, Alice uses a technique called rarest first. The idea is to determine, from among the chunks she does not have, the chunks that are the rarest among her neighbors and then request those rarest chunks first.
5. To determine which request she respond to, the basic idea is that Alice gives priority that are currently supplying her data at the highest rate.
6. Importantly, every 30 seconds, she also picks one additional neighbour at random and send it chunks, which is refered to as tit-for-tat.

## Video Streaming and Content Distribution Networks

1. A video is a sequence of images, typically being displayed at a constant rate, for example, at 24 or 30 images per second. An important characteristic of video is that it can be compressed, thereby trading off video quality with bit rate.
2. From a networking perspective, perhaps the most salient characteristic of video is its high bit rate.
3. HTTP streaming has a major shortcoming: All clients receive the same encoding of the video, despite the large variations in the amount of bandwidth available to a client, both across different clients and also over time for the same client.
4. In DASH, the video is encoded into several different versions, with each version having a different bit rate and, correspondingly, a different quality level. When the amount of available bandwidth is high, the client naturally selects chunks from a high-rate version; and when the available bandwidth is low, it naturally selects from a low-rate version.
5. In fact, many CDNs do not push videos to their clusters but instead use a simple pull strategy. Most CDNs take advantage of DNS to intercept and redirect requests.
6. At the core of any CDN deployment is a cluster selection strategy, that is, a mechanism for dynamically directing clients to a server cluster or a data center within the CDN.
7. YouTube does not employ adaptive streaming such as DASH, but instead requires the user to manually select a version. It uses the HTTP byte range request to limit the flow of transmitted data after a target amount of video is prefetched.
8. Kankan uses P2P delivery instead of client-server delivery. P2P video streaming is very similar to BitTorrent file downloading. When a peer wants to see a video, it contacts a tracker to discover other peers in the system that have a copy of that video.
9. Recently, Kankan has migrated to a hybrid CDN-P2P streaming system.

## Socket programming: Creating Network Applications

1. A typical network application consists of a pair of programs: a client program and a server program, residing in two different end systems. When these two programs are executed, a client process and a server process are created, and these processes communicate with each other by reading from, and writing to, sockets.
2. There are two type of network applications: (1)One type is an implementation whose operation is specified in a protocol standard, such as an RFC. (2)The other type of network is a proprietary network application. In this case the client and server programs employ an application-layer protocol that has not been openly published in an RFC.
3. When a socket is created, an identifier, called a port number, is assigned to it. So, as you might expect, the packet's destination address all includes the socket's port number.
4. Unlike UDP, TCP is a connection-oriented protocol. This means that before the client and server can start to send data to each other, they first need to handshake and establish a TCP connection.

