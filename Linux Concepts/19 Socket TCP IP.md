
# 1 internets

- An **internetwork** or, more commonly, **internet** (with a **lowercase** **i**), connects different **computer networks** ( aka **subnet** in this case, is collection of multipe **hosts** which are devices have IP such as PC, Laptop, smart phone,... ), allowing **hosts** on all of the networks to communicate with one another. In other words, **an internet is a network of computer networks.**
- An **internet** aims to **hide** the details of **different physical networks** in order to present a unified network architecture to all hosts on the connected networks.

- The term **Internet** (with an **uppercase I**) is used to refer to the TCP/IP internet that connects millions of computers globally

- a **router** is  **a computer** whose function is to connect one subnetwork to another, transferring data between them.
- ["] As well as understanding the internet protocol being used, a **router** **must** also **understand** the (possibly) **different** **data-link-layer protocols** used on **each of the subnets that it connects** 
- A **router** has multiple network interfaces, one for each of the subnets to which it is connected.
![[Pasted image 20250816083754.png]]

# 2 Networking Protocols and Layers


- A **networking protocol** is a set of rules defining how information is to be transmitted across a network
- The **TCP/IP** **protocol suite** is **a layered networking protocol**. It **includes** the **Internet Protocol** (IP) and **various protocols** layered above it.
![[Pasted image 20250816085935.png]]


# 3 The Data-Link Layer

- The lowest layer in Figure 58-2 is **the data-link layer**, which **consists** of the **device driver** and the **hardware interface** (network card) to the underlying physical medium (e.g., a telephone line, a coaxial cable, or a fiber-optic cable). **The data-link layer** is concerned with **transferring data across a physical link in a network** ( mean in a  **Computer Network (Subnet)**)
- To transfer data, the data-link layer encapsulates datagrams from the network layer into units called frames. In addition to the data to be transmitted, each frame includes a header containing, for example, the destination address and frame size.

- One characteristic of the data-link layer that is important for our discussion of IP is the **maximum transmission unit** (MTU). A data-link layer’s MTU is the **upper limit** that the layer places on the **size of a frame**. Different data-link layers have different MTUs.

![[Pasted image 20250816090817.png]]

# 4 The Network Layer: IP

- Above the data-link layer is the network layer, which is concerned with delivering packets (data) from the source host to the destination host. This layer performs a variety of tasks, including:
	- breaking data into fragments small enough for transmission via the data-link layer (if necessary);
	- routing data across the internet; and
	- providing services to the transport layer.

- The most notable difference between the two versions is that IPv4 identifies subnets and hosts using 32-bit addresses, while IPv6 uses 128-bit addresses, thus providing a much larger range of addresses to be assigned to hosts


- **IP transmits datagrams**
	- IP **transmits** data in the form of **datagrams** (packets).
	- Each datagram sent between two hosts travels independently across the network, possibly taking a different route
	- An IP datagram includes a header, which ranges in size from 20 to 60 bytes. The header contains the address of the target host, so that the datagram can be routed through the network to its destination, and also includes the originating address of the packet, so that the receiving host knows the origin of the datagram.
- **IP is connectionless and unreliable**
	- IP is described as a **connectionless protocol**, since it doesn’t provide the notion of a virtual circuit connecting two hosts
	- IP is also an **unreliable protocol**: it makes a “best effort” to transmit datagrams from the sender to the receiver, but doesn’t guarantee that packets will arrive in the order they were transmitted, that they won’t be duplicated, or even that they will arrive at all

- **IP may fragment datagrams**:
	- IPv4 datagrams can be up to 65,535 bytes. By default, IPv6 allows datagrams of up to 65,575 bytes (40 bytes for the header, 65,535 bytes for data), and provides an option for larger datagrams (so-called **jumbograms**).
	- We noted earlier that most data-link layers impose an upper limit (the MTU) on the size of data frames. For example, this upper limit is 1500 bytes on the commonly used Ethernet network architecture (i.e., much smaller than the maximum size of an IP datagram). IP also defines the notion of the path MTU
	- When an IP datagram is larger than the MTU, IP fragments (breaks up) the datagram into suitably sized units for transmission across the network.


# 5 IP Addresses

- An IP address consists of two parts: a network ID, which specifies the network on which a host resides, and a host ID, which identifies the host within that network

- **IPv4 addresses**
![[Pasted image 20250816130106.png]]
	- **One** of these is the address **whose host ID is all 0 bits**, which **is used to identify the network itself**
	-  **The other** is the address **whose host ID is all 1 bits** which is **the subnet broadcast address**.
	- **Certain IPv4 addresses have special meanings.** The special address **127.0.0.1** is normally defined as the loopback address, and is conventionally **assigned** the hostname **localhost**. (Any address on the network 127.0.0.0/8 can be designated as the IPv4 loopback address, but 127.0.0.1 is the usual choice.)
	- A datagram sent to this address ( **localhost**) never actually reaches the network, but instead automatically loops back to become input to the sending host. **Using this address is convenient for testing client and server programs on the same host**. For use in a C program, the integer constant **INADDR_LOOPBACK** is defined for this address
	- Typically, IPv4 addresses are subnetted.
	- For example , we choose to subnet this address range by splitting the 8 bits of the host ID into a 4-bit subnet ID and a 4-bit host ID.
	![[Pasted image 20250816132124.png]]

- **IPv6 addresses**
	-The principles of IPv6 addresses are similar to IPv4 addresses. The key difference is that IPv6 addresses consist of 128 bits, and the first few bits of the address are a format prefix, indicating the address type
	
# 6 The Transport Layer

- There are two widely used transport-layer protocols in the TCP/IP suite:
	- **User Datagram Protocol** (UDP) is the protocol used for **datagram sockets**
	- **Transmission Control Protocol** (TCP) is the protocol used for **stream sockets**.

## 6.1 Port Numbers
- The **task** of **the transport protocol** is to **provide** an **end-to-end communication service** to **applications** residing on **different hosts** (**or** sometimes on the **same host**). **In order to do this**, the transport layer requires **a method of differentiating the applications on a host**.
- In TCP and UDP, this differentiation is provided by a **16-bit port number**
- **Well-known, registered, and privileged ports**:
	- Some well-known port numbers are permanently assigned to specific applications:
		- the ssh (secure shell) daemon uses the wellknown port 22
		- HTTP (the protocol used for communication between web servers and browsers) uses the well-known port 80
	- **Well-known ports** are assigned numbers **in the range 0 to 1023** by a central authority, the Internet Assigned Numbers Authority (IANA, http://www.iana.org/)
	- IANA also records **registered ports**, which are allocated to application developers on a less stringent basis. **The range of IANA registered ports is 1024 to 41951**
	- In most TCP/IP implementations (including Linux), **the port numbers in the range 0 to 1023** are **also** **privileged**, meaning that only privileged (CAP_NET_BIND_SERVICE) processes may bind to these ports
- **Ephemeral ports**:
	- If an application doesn’t select a particular port (i.e., in sockets terminology, it doesn’t bind() its socket to a particular port), then TCP and UDP assign a unique ephemeral port (i.e., short-lived) number to the socket.
	- IANA specifies the ports in the range **49152 to 65535** as dynamic or private, with the intention that these ports can be used by local applications and assigned as ephemeral ports
	- On Linux, the range is defined by (and can be modified via) two numbers contained in the file **/proc/sys/net/ipv4/ip_local_port_range**


## 6.2 User Datagram Protocol (UDP)

- UDP adds just **two feature**s to IP: **port numbers** and a **data checksum** to allow the detection of errors in the transmitted data
- Like IP, UDP is connectionless.
- **Selecting a UDP datagram size to avoid IP fragmentation**:
	- UDP-based applications that aim to avoid IP fragmentation typically adopt a conservative approach, which is to ensure that the transmitted IP datagram is less than the IPv4 minimum reassembly buffer size of 576 bytes.
	- From these 576 bytes, 8 bytes are required by UDP’s own header, and an additional minimum of 20 bytes are required for the IP header, leaving 548 bytes for the UDP datagram itself. In practice, many UDP-based applications opt for a still lower limit of 512 bytes for their datagrams

## 6.3 Transmission Control Protocol (TCP)

- TCP provides a reliable, connection-oriented, bidirectional, byte-stream communication channel between two endpoints (i.e., applications).

![[Pasted image 20250816140318.png]]

- We use the term **TCP endpoin**t to denote the information maintained by the kernel for one end of a TCP connection. (Often, we abbreviate this term further, for example, writing just “a TCP,” to mean “a TCP endpoint,” or “the client TCP” to mean “the TCP endpoint maintained for the client application.”)


- **Connection establishment**:
	- Before communication can commence, TCP establishes a communication channel between the two endpoints. During connection establishment, the sender and receiver can exchange options to advertise parameters for the connection
- **Packaging of data in segments**:
	- Data is broken into segments, each of which contains a checksum to allow the detection of end-to-end transmission errors. Each segment is transmitted in a single IP datagram
- **Acknowledgements, retransmissions, and timeouts**
	- When a TCP segment arrives at its destination without errors, the receiving TCP sends a positive acknowledgement to the sender, informing it of the successfully delivered data. If a segment arrives with errors, then it is discarded, and no acknowledgement is sent.
	- To handle the possibility of segments that never arrive or are discarded, the sender starts a timer when each segment is transmitted. If an acknowledgement is not received before the timer expires, the segment is retransmitted.

- **Sequencing**:
	- Each byte that is transmitted over a TCP connection is assigned a logical sequence number. This number indicates the position of that byte in the data stream for the connection
- **Flow control**
	- Flow control prevents a fast sender from overwhelming a slow receiver
- **Congestion control: slow-start and congestion-avoidance algorithms**
	- TCP’s congestion-control algorithms are designed to prevent a fast sender from overwhelming a network
# 7 Requests for Comments (RFCs)

Each of the Internet protocols that we discuss in this book is defined in an RFC document—a formal protocol specification. RFCs are published by the RFC Editor (http:// www.rfc-editor.org/), which is funded by the Internet Society (http://www.isoc.org/). RFCs that describe Internet standards are developed under the auspices of the Internet Engineering Task Force (IETF, http://www.ietf.org/), a community of network designers, operators, vendors, and researchers concerned with the evolution and smooth operation of the Internet. Membership of the IETF is open to any interested individual







































