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



