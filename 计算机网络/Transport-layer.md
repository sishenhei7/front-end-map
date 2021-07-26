# Transport Layer

## Introduction and transport-layer services

1. Fundamentally important problems in networking: (1)how two entities can communicate reliably over a medium that may lose and corrupt data. (2)cotrolling the transmission rate of trasport-layer entities in order to avoid, or recover from, congestion within the network.
2. Tt's important to note that network routers act only on the network-layer fields of the datagram; that is, they do not examine the fields of the transport-layer segment encapsulated with the datagram. Because transport-layer protocols are implemented in the end systems but not in network routers.
3. The IP service model is a best-effort delivery service.
4. The most fundamental responsibility of UDP and TCP is to extend IP's delivery service between two end systems to a delivery service between two processes running on the end systems. Extending host-to-host delivery to process-to-process delivery is called transport-layer multiplexing and demultiplexing.
5. These two minimal transport-layer services: process-to-process delivery and error checking, are the only two services that UDP provides.
6. TCP, on the other hand, provides reliable data transfer and congestion control. It uses flow control, sequences numbers, acknowledgements, and timers to ensure reliable data transfer.

## Multiplexing and Demultiplexing

1. The transport layer has the responsibility of delivering the data in these segments to the appropriate application process running in the host.
2. The transport layer in the receiving host does not actually deliver data directly to a process, but instead to an intermediary socket. Because at any given time there can be more than one socket in the receiving host, each socket has a unique identifier. The format of the identifier depends on whether the socket is a UDP or a TCP.
3. This job of delivering the data in a transport-layer segment to the correct socket is called demultiplexing. The job of gathering data chunks at the source host from different sockets, encapsulating each data chunk with header information to create segments, and passing the segments to the network layer is called multiplexing.
4. transport-layer multiplexing requires: (1)that sockets have unique identifiers, and (2)that each segment have special fields that indicate the socket to which the segment is to be delivered. These special fields, are the source port number field and destination port number field. Each port number is a 16-bit number, ranging from 0 to 65535. The port numbers ranging from 0 to 1023 are called well-known port numbers and are restricted.
5. Each socket in the host could be asigned a port number, and when a segment arrives at the host, the transport layers examins the destination port number in the segment and directs the segment to the corresponding socket. This is basically how UDP does.
6. It is important to note that a UDP socket is fully identified by a two-tuple consisting of a destination IP address and a destination port number.
7. One subtle different between a TCP socket and a UDP socket is that a TCP socket is identified by a four-tuple: source IP address, source port number, destination IP address, destination port number.
8. If the client and server are using persistant HTTP, then throughout the duration of the persistant connection the client and server exchange HTTP messages via the same server socket. However, if the client and server use non-ponsistent HTTP, then a new TCP connection is created and closed for every request/response, and hence a new socket is created and later closed for every request/response.

## Connectionless Transport: UDP

1. At the very least, the transport layer has to provide a multiplex/demultipexing service in order to pass data between the network layer and the correct application-level process.
2. Aside from the multipexing/demultiplexing function and sone light error checking, UDP adds nothing to IP.
3. TCP maintains connection state in the end systems. This connection state includes receive and send buffers, congestion-control parameters, and sequence and acknoledgement number parameters.
4. UDP is better suited for some application for these reasons: (1)Finer application-level control over what data is sent and when. (2)No connection establishment. (3)No connection state. (4)Small packet header overhead.
5. Tt is possible for an application to have reliable data transfer when using UDP. This can be done if reliablity is built into the application itself, for example, by adding acknoledgement and retransmission mechanism.
6. The UDP header has only four fields, each consisting of two bytes. They are source port, destination port, length and checksum.
7. The UDP checksum provides for error detection. That is, the checksum is used to determine whether bits within the UDP segment have been altered as it move from source to destination. UDP at the sender side performs the 1s complement of the sum of all the 16-bit words in the segment, with any overflow encountered during the sum being wrapped around.
8. Given that neither link-by-link reliability nor in-memory error detection is guarenteed, UDP must provide error detection at the transport layer. Although UDP provides error checking, it does not do anything to recover from error.

## Principles of Reliable data transfer

1. Reliable data transfer is of central importance to networking.
2. rdt1.0, the simplest case, in which the underlying channel is completely reliable: (1)there is no need for the receiver side to provide any feedback to the sender since nothing can go wrong. (2)there is no need for the receiver to ask the sender to slow down.
3. rdt2.0, a more realistic model of the underlying channel is one in which bits in a packet may be corrupted.
4. Acknowledgement messages allow the receiver to let the sender know what has been received correctly, and what has been received in error and thus need repeating.
5. Fundamentally, these capabilities are required to handle the presence of bit errors: (1)Error detection. A mechanism is needed to allow the receiver to detect when bit errors has occured. (2)Receiver feedback. The only way for the sender to learn of the receiver's view of the world is for the receiver to provide explicit feedback to the sender. (3)Retransmission. A packet that is received in error at the receiver will be retransmitted by the sender.
6. rdt2.0 is known as stop-and-wait protocol. But we haven't accounted for the possibility that the ACK or NAK packet counld be corrupted, and, how the protocol should recover from errors in ACK or NAK packets. A simple solution is to put a sequence number into the data packet, the receiver need only check this sequence number to determine whether or not the received packet is a retransmission.
7. rdt3.0, the underlying channel can lose packet as well. So two additional concerns must now be addressed by the protocol: how to delect packet loss and what to do when packet loss occurs.
8. The approach thus adopted in practice is for the sender to judiciously choose a time value such that packet loss is likely, although not guarenteed, to have happened. The sender will thus need to be able to (1) start the timer each time a packet, either a first-time packet or a retransmission packet, is sent, (2)respond to a timer interrupt, (3)stop the timer.
9. In our GBN protocol, an acknowledgement for a packet with sequence number n will be taken to be a cumulative acknoledgement, indicating that all packets with a sequence number up to and including n have been correctly received at the receiver.
10. The receiver's actions in GBN are also simple. If a packet with sequence number n is received correctly and is in order, the receiver sends an Ack for packet n and delivers the data portion of the packet to the upper layer. In all other cases, the receiver discards the packet and resends an ACK for the most recently received in-order packet.
11. In our GBN protocol, the receiver discards out-of-order packets. The advantage of this approach is the simplicity of receiving buffering: the receiver need not buffer any out-of-order packets
12. The drawback of GBN may be that a single packet error can cause GBN to retransmit a large number of packets, many unnecessarily.
13. As the name suggests, selective-repeat protocols avoid unnecessary retransmissions by having the sender retransmit only those packets that it suspects were received in error at the receiver.
14. The SR receiver will acknowledge a correctly received packet whether or not it is in order. Out-of-order packets are buffered until any missing packets are received.
15. It is important to note that in Step 2 in figure 3.25, the receiver reacknoledges(rather than ignores) already received packets with certain sequence numbers below the current window base.
16. An important aspect of SR protocols is that the sender and receiver will not always have an identical view of what has been received correctly and what was not. For SR protocols, this means that the sender and receiver windows will not always coincide, which has important consequences.

## Connection-Oriented transport: TCP

1. We'll see that in order to provide reliable data transfer, TCP relies on many of the underlying principles discussed in the previous section, including error detection, retransmissions, cumulative acknowledgement, timers, and header fields for sequence and acknowledgement numbers.
2. TCP is said to be connection-oriented because before one application process can begin to send data to another, the two process must first 'handshake' with each other. Both sides of the connection will initialize many TCP state variables associated with the TCP connection.
3. A TCP connection provides a full-duplex service, and it is always point-to-point.
4. The maximum amout of data that can be grabbed and placed in a segment is limited by the maximun segment size. The MSS is typically set by first determining the length of the largest link-layer frame that can be sent by the local sending host, MTU, and then setting the MSS to ensure that a TCP segment plus the TCP/IP header length will fit into a single link-layer frame. Both Ethernet and PPP link-layer protocols have an MTU of 1500 bytes. Thus a typical value of MSS is 1400 bytes.
5. The sequence number for a segment is therefore the byte-stream number of the first byte in the segment. The acknowledgement number that Host A puts in its segment is the sequence number of the next byte Host A is expecting from Host B.
6. Because TCP only acknowledges bytes up to the first missing byte in the stream, TCP is said to provide cumulative acknowledgement.
7. What does a host do when it receives out-of-order segments in a TCP connection? The TCP RFCs do not impose any rules here.
8. TCP uses a timeout/retransmission mechanism to recover from lost segments. The most obvious question is the length of the timeout intervals.
9. The SampleRTT is being estimated for approximately every RTT. Also TCP never computes retransmitted segment.
10. TCP maintains an average, called EstimatedRTT, of the SampleRTT values.
11. The RTT variation, DevRTT, is an estimate of how much SampleRTT typically deviates from EstimatedRTT.
12. TimeoutInterval = EstimatedRTT + 4 * DevRTT
13. The recommended TCP timer management procedures use only a single retransmission timer.
14. Doubling the timeout interval: whenever the timeout event occurs, TCP retransmitted the not-yet-acknowledged segment with the smallest sequence number. But each time TCP retransmis, it sets the next timeout interval to twice the previous value. This provides a limited form of congection control.
15. Fast Retransmit: If the TCP sender receives three duplicate ACKs for the same data, it takes this as an indication that the segment following the segment that has been ACKed three times has been lost. In this case, the TCP sender performs a fast transmit.
16. Flow Control is a speed-matching service: matching the rate at which the sender is sending against the rate at which the receiving application is reading.
17. A TCP sender can also be throttled due to congestion within the IP network; this form of sender control is referred to as congestion control.
18. TCP provides flow control by having the sender maintain a variable called the receive window(rwnd = RcvBuffer - (LastByteRcvd - LastByteRead)). By keeping the amount of unacknowledged data less than the value of rwnd, Host A is assured that it is not overflowing the receive buffer at Host B.
19. The TCP specification requires Host A to continue to send segments with one data byte when B's receive window is zero.
20. To establish a TCP connection: (1)The client-side TCP first sends a special TCP segment to the server-side TCP with the SYN bit set to 1 with a client_isn set. For this reason, this special segment is referred to as a SYN segment. (2)The server-side TCP sends a TCP segment with SYN bit set to 1 with a server_isn set. (3)The clients allocates buffers and variables to the connection and send back a TCP segment with SYN bit set to 0.
21. When closing a TCP connection, the FIN bit is set to 1.
22. When a host receive a TCP packet whose destination port number doesn't match with an ongoing UDP socket, the host send a special reset segment with the RST bit set to 1, while the UDP host sends a special ICMP datagram. When the client receives neither of above, this likely means that the SYN segment was blocked by an intervening firwall and never reached the target host.

## Principles of Congestion constrol

1. Packet retransmission treats a symptom of network congestion but does not treat the cause of network congestion: too many sources attempting to send data at too high a rate.
2. Large queuing delays are experienced as the packet-arrival rate nears the link capacity.
3. Because packets can be retransmitted, we must now be more careful with our use of the term sending rate: (1)the sender must perform retransmission in order to compensate for dropped packets due to buffer overflow. (2)unneeded retransmissions by the sender in the face of large delays may cause a router to use its link bandwidth to forward unneeded copies of a packet.
