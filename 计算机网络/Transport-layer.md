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

