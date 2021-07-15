# chapter 1 Computer networks and the internet

## what is internet

1. What is the internet: (1)The basic hardware and software components that make up the internet. (2)A networking infrastructure that provides service to the distributed applications.
2. The basic components are: (1)communication links. (2)packat switches: routers and link-layer switches. Link layer switches are typically used in access networks, while routers are typically used in the network core. (3)ISPs: residential ISPs、corporate ISPs、university ISPs、cellular data ISPs and so on.
3. The IETF documents are called requests for comments(RFCs). RFCs are tend to be quite technical and detailed.
4. The applications are said to be distributed since they involve multiple end systems that exchange data with each other.
5. End systems attach to internet via socket interface, whick is a set of rules that the sending program must follow so that the internet can diliver data to the destination program.
6. In human protocol, there are specific messages we send, and specific actions we take in response to the recieved messages or other events. A network protocol defines the format and order of the messages exchanged between two or more communicating entities, as well as the actions taken on the trasmission or receipt of a message or other events.

## the network edge

1. End systems are so called because they sit at the edge of the internet. They are also called hosts, and can be further devided into two categories: clients and servers.
2. Access networks are the networks that physically connects an end system to the first router on a path from the end system to any other distant end system, like digital subscribe line(DSL) and cable.
3. The DSL makes use of the telco's existing local telephone infrastructure, they carries both data and traditional telephone signals simultaneously: a high-speed downstream channel; a medium-speed upstream channel; an ordinary two-way telephone channel.
4. The cable internet access the cable television company's existing cable television infrastracture. They require special modem called cable modems.
5. Fiber to the home(FTTH) is another even higher speeds technique which can provide gigabits per second range.
6. A satellite link can be used to connect a residence to the internet at the speed of more than 1 Mbps.
7. Home access networks: DSL、Cable、FTTH、Dial-up、Satellite. Enterprise access: Ethernet and Wifi.
8. Phisycal media fall into two categories: guided media and unguided media. Guided media includes copper wire、cable and fiber. Unguided media includes terrestrial radio channels and satellite  radio channels.

## the network core

1. The network core: the mesh of packet switches and links that interconnects the internet'send systems.
2. Most packet switches use store-and-forward transmission at the inputs to the links. Store-and-forward means that the switch must receive the entire packet before it can begin to transmit the first bit of the packet.
3. The packet switch has an output buffer. Thus, in addition to the store-and-forward delays, packets suffer output buffer queuing delays. In this case, packet loss will occur: either the arriving packet or the already-queued packet will be dropped.
4. Each rotuer has a forwarding table that maps destination address to that router's outbound links. The internet has a number of special routing protocols that are used to automatically set the forwarding tables.
5. There are two fundamental approaches to moving data through a network of links and switches: circuit switching and packet switching. In circuit switching networks, the paths are reserved for the duration of the communication session between the end systems.
6. Traditional telephone networks are examples of circuit switching networks.
7. The internet makes its best effort to deliver packets in a timely manner, but it does not make any guarantees.
8. A circuit in a link is implemented with either frequency-division multiplexing(FDM) or time-division multiplexing(TDM). With FDM, the frequency spectrum of a link is divided up among the connections established across the link. For a TDM link, time is divided into frames of fixed duration, and each frame is divided into a fixed number of time slots.
9. The access ISPs themselves must be interconnected. This is done by creating a network of networks. Understanding this phrase is the key to understanding the internet.
10. Network structure 1, interconnects all of the access ISPs with a single global transit ISP. Network structure 2, consists of hundreds of thousands of access ISPs and multiple global transit ISPs. Network structure 3, in any given region, there may be a regional ISP to which the access ISPs in the region connect. Network structure 4, we must add points of presence(PoPs), multi-homing, peering, and internet exchange points(IXPs) to the hierarchy. Network structure 5, which is the internet today, adds content-provider networks to the hierarchy. Google is currently one of the leading examples of such a content-provider network.

## delay, loss and thoughput in packet-switchednetworks

1. The most important delays are (1)nodal processing delay, (2)queuing delay, (3)transmission delay, (4)propagation delay. Togather, they give a total nodal delay.
2. The time required to examine the packet's header and determine where to direct the packet is part of the processing delay.
3. At the queue, the packet expriences a queuing delay as it waits to be transmitted onto the link.
4. In packet-switched networks, our packet can be transmitted only after all the packets that have arrived before it have been transmitted.
5. The time required to propagate from the beginning of the link to router B is the propagation delay.
6. Our discussion up to this point has focused on the nodal delay, that is, the delay at a single router. Whereas an end to end delay has N routers.
7. A packet can arrive to find a full queue. with no place to store such a packet, a router will drop that packet; that is, the packet will be lost.
8. The instantaneous throughput at any instant of time is the rate at which Host B is receiving the file. It is the transmission rate of the bottleneck link.

## protocal layers and their service model

1. One way to describe the airline system might be to describe the series of actions you take (or others take for you when you fly on an airline. A layered architecture allows us to discuss a well-defined, specific part of a large and complex system.
2. Each Internet layer provides its service by (1)performing certain actions within that layer and by (2)using the services of the layer directly above it.
3. Two drawbacks of layering are (1)one layer may duplicate lower-layer functionality and (2)functionality at one layer may need information that is present only in another layer.
4. The application layer is where network applications and their application-layer protocols reside. We'll refer to this packet of information at the application layer as a message.
5. The internet's transport layer transports application-layer messages between application endpoints. We'll refer to a transport-layer packet as a segment.
6. The Internet's network layer is responsible for moving network-layer known as datagrams from one host to another. It contains both the IP protocol and numerous routing protocols.
7. To move an entire packet from one node(host or router) to the next node in the route, the network layer relies on the services of the link layer. Examples of link-layer protocols include Ethernet, Wifi, and the cable access network's DOCSIS protocol.
8. The job of the physical layer is to move the individual bits within the frame from one node to another.

## networks under attack

1. The bad guys can (1)put malware in your host via the internet like viruses and worms. (2)attack servers and network infrastructure, like Dos attack: vulnerablity attack, bandwidth flooding, connection flooding. (3)can sniff packets. (4)can masquerade as someone you trust.

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



