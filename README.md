# Esp32 Mesh

We designed a protocol to deploy our prototype by considering the lim-
itations of a LoRa radio to exchange control and data information. In this
sense, we adapted the Babel routing protocol (RCF 8966)

# Mesh Packet
The esp32_mesh packet format with 16 bytes of header and 240 bytes
for payload. We consider the following fields as a packet header:

*Time to Live (TTL) [1 byte]: means the packet “lifetime”, which can
be seen as the number of hops allowed before a node drops the message.

*Size [1 byte]: is the length of the packet, i.e., header and payload.

*Source and destination addresses [4 bytes]: identifiers of the node that
originated the packet and the intended packet destination address.

*Next-hop address [4 bytes]: is the identifier of the next node, which the
packet should be forwarded, to reach the destination address.

*Sequence [1 byte]: means the global message counter to determine the
packet loss ratio and duplicated. Moreover, it is used to demultiplex the
fragments received by the two radio interfaces, where both fragments
must have the same sequence to be grouped.

*Type [1 byte]: is an ASCII character, which describes the payload
content. r means route message, h denotes hello message

The original Babel routing protocol includes the next-hop information as a packet
payload, introducing additional processing at the relay node to get such vital
information. In this case, we consider the next-hop address within the packet
header. Each node has a unique 4-byte identifier based on the least significant
bytes of the MAC address of the hardware, which is used to guarantee this
individual address. We used this identifier to keep the compatibility with
the MAC address. We define the broadcast address as 0xFFFFFFFF. It is
fundamental to highlight that we can have a larger node address size at the
cost of reducing the payload size.

During the Discovery phase, the Routing algorithm
creates a hello message (1) at each time interval to present itself to its neigh-
bors, transmitted by a given LoRa radio with low priority (2). The hello
message has a size of 16 byte, which contains only the header information as
a tuple (1, 16, source address, 0xF F F F F F F F , 0xF F F F F F F F , sequence
number, h). In addition, the hello message has a TTL with the value 1 to
avoid flooding, which means that each hello message is broadcasted only to
one-hop neighbors.

During this phase, the Routing algorithm creates a routing
message (1) to share its Routing table with its neighbors (2), transmitted
by a given LoRa radio with low priority (3). The routing message contains
the header information as a tuple (1, s, source address, 0xF F F F F F F F ,
0xF F F F F F F F , sequence number, r), and the Routing table as the payload
data. The routing message has a size of s-byte (i.e., header and payload).
Due to the limited packet size (i.e., 240 bytes of data), the routing message
is limited to 30 routes (i.e., 8 bytes for each route)

The Packet relaying module analyzes the message by overhearing a rout-
ing message coming from a given IoT node (6). The route information within
the message is compared with all existing routes in the Routing table. As
a result, the Packet relaying adds the new route to its table, updates a cur-
rent route with a new or better metric, or eliminates the route, as it has a
better route or the route refers to the local address of the nodes (7). Each
IoT node learns about its neighbors and reaches a given node within the
network. We implemented the Routing table as a list with the following in-
formation: Destination, Next-hop, distance, sequence number, metric, and
priority, wherein each entry has a size of 8-byte. The Learning phase is
probably never complete, as new nodes may appear at any time. After a
determined time configured as a parameter on each node in the network, the
node enters the Forwarding phase
