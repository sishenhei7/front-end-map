# Transport Layer

1. Fundamentally important problems in networking: (1)how two entities can communicate reliably over a medium that may lose and corrupt data. (2)cotrolling the transmission rate of trasport-layer entities in order to avoid, or recover from, congestion within the network.
2. Tt's important to note that network routers act only on the network-layer fields of the datagram; that is, they do not examine the fields of the transport-layer segment encapsulated with the datagram. Because transport-layer protocols are implemented in the end systems but not in network routers.
3. The IP service model is a best-effort delivery service.
4. The most fundamental responsibility of UDP and TCP is to extend IP's delivery service between two end systems to a delivery service between two processes running on the end systems. Extending host-to-host delivery to process-to-process delivery is called transport-layer multiplexing and demultiplexing.
5. These two minimal transport-layer services: process-to-process delivery and error checking, are the only two services that UDP provides.
6. TCP, on the other hand, provides reliable data transfer and congestion control. It uses flow control, sequences numbers, acknowledgements, and timers to ensure reliable data transfer.