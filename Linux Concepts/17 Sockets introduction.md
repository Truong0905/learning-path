Sockets are a method of IPC that allow data to be exchanged between applications, either on the same host (computer) or on different hosts connected by a network.


# 1 Overview

In a typical client-server scenario, applications communicate using sockets as follows:
- Each application creates a socket. A socket is the “apparatus” that allows communication, and both applications require one
- The server binds its socket to a well-known address (name) so that clients can locate it.

A socket is created using the **socket**() system call, which returns a file descriptor used to refer to the socket in subsequent system calls:
```c
fd = socket(domain, type, protocol);
```

**Communication domains**:

Sockets exist in a **communication domain**, which determines:
- the method of identifying a socket (i.e., the **format** of a socket “address”);
- the range of communication (i.e., either between applications on the **same host** **or** between applications on **different hosts** connected via a network)


Modern operating systems support at least the following domains:
- The UNIX (**AF_UNIX**) domain allows communication between applications on the **same host**.
- The IPv4 (**AF_INET**) domain allows communication between applications running on hosts connected **via an Internet Protocol version 4 (IPv4) network**.
- The IPv6 (**AF_INET6**) domain allows communication between applications running on hosts connected **via an Internet Protocol version 6 (IPv6) network**
![[Pasted image 20250811213123.png]]


**Socket types**

![[Pasted image 20250811213517.png]]

- Stream sockets (**SOCK_STREAM**) provide a reliable, bidirectional, byte-stream communication channel:
	- **Reliable** means that we are guaranteed that either the transmitted data will arrive **intact** at the receiving application.
	- **Bidirectional** means that data may be transmitted in **either** direction between **two sockets**
	- **Byte-stream** means that, as with pipes, there is no concept of message boundaries

	- ["] **Stream sockets** operate in connected pairs. For this reason, stream sockets are described as **connection-oriented**. The term **peer socket** **refers** to **the socket** at **the other end of a connection**; **peer address** denotes **the address of that socket**; and **peer application** denotes t**he application utilizing the peer socket**. Sometimes, the term remote (or foreign) is used synonymously with peer.Analogously, sometimes the term local is used to refer to the application, socket, or address for this end of the connection. A stream socket can be connected to only one peer

- Datagram sockets (**SOCK_DGRAM**) allow data to be exchanged in the form of messages called datagrams. With datagram sockets, message boundaries are preserved, but data transmission is not reliable. Messages may arrive out of order, be duplicated, or not arrive at all

	
- **In the Internet domain**, **datagram sockets** **employ** the User Datagram Protocol (**UDP**), and **stream sockets** (usually) **employ** the Transmission Control Protocol (**TCP**).



**Socket system calls** :

- The **socket**() system call creates a new socket.
- The **bind**() system call binds a socket to an address
- The **listen**() system call allows a stream socket to accept incoming connections from other sockets
-  The **accept**() system call accepts a connection from a peer application on a listening stream socket, and optionally returns the address of the peer socket
- The **connect**() system call establishes a connection with another socket


# 2 Creating a Socket: **socket**()

- The **socket**() system call creates a new socket.
```c
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
// Returns file descriptor on success, or –1 on error
```
- The **domain** argument specifies the communication domain for the socket
- . The **type** argument specifies the socket type. This argument is usually specified as either **SOCK_STREAM**, to create a stream socket, or **SOCK_DGRAM**, to create a datagram socket
- On success, **socket**() returns **a file descriptor** used to refer to the newly created socket in later system calls

# 3 Binding a Socket to an Address: **bind**()

- The **bind**() system call binds a socket to an address.
```c
#include <sys/socket.h>
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
// Returns 0 on success, or –1 on error
```

- The **sockfd** argument is a file descriptor obtained from a previous call to **socket**()
- The **addr** argument is a pointer to a structure specifying the address to which this socket is to be bound. The type of structure passed in this argument **depends on the socket domain**
- The **addrlen** argument specifies the size of the address structure

- ["] Typically, we bind a server’s socket to a well-known address—that is, a fixed address that is known in advance to client applications that need to communicate with that server
# 4 Generic Socket Address Structures: struct sockaddr
- **bind**() are generic to all socket domains, they must be able to accept address structures of any type. In order to permit this, the sockets API defines a generic address structure, struct sockaddr
```c
struct sockaddr {
sa_family_t sa_family; /* Address family (AF_* constant) */
char sa_data[14]; /* Socket address (size varies according to socket domain) */ 
};
```


# 5 Stream Sockets
- The **socket**() system call, which creates a socket
- Calls **bind**() in order to bind the socket to a well-known address
- Calls **listen**() to notify the kernel of its willingness to accept incoming connections
- **The other** application **establishes** the connection **by** calling **connect**(), **specifying** the **address of the socket** to which **the connection is to be made**.
- **The application** that **called** **listen**() then accepts the connection using **accept**()
- If the **accept**() is performed **before** the **peer application** calls **connect**(), then the **accept**() blocks

- Once a connection has been established, data can be transmitted in both directions between the applications until one of them closes the connection using **close**(). Communication is performed using the conventional **read**() and write() **system** calls **or** via a number of socketspecific system calls (such as **send**() and **recv**()) that provide additional functionality

**Active and passive sockets**:
- **By default**, a **socket** that has been **created** using **socket**() **is active.** An active socket can be used in a connect() call to establish a connection to a passive socket. This is referred to as performing an active open.
- **A passive socket** (also called a **listening socket**) is one **that has been marked to allow incoming connections by calling listen()**. Accepting an incoming connection** is referred to as performing a **passive open**.

![[Pasted image 20250811224810.png]]

- In most applications that employ stream sockets, the server performs the passive open, and the client performs the active open


## 5.1 Listening for Incoming Connections: **listen**()


- The **listen**() system call marks the stream socket referred to by the file descriptor **sockfd** as **passive**.
- The socket will subsequently be used to accept connections from other (active) sockets

```c
#include <sys/socket.h> 
int listen(int sockfd, int backlog);
Returns 0 on success, or –1 on error
```
- We can’t apply **listen**() to a connected socket—that is, a socket on which a **connect**() has been successfully performed or a socket returned by a call to **accept**().
- To understand the purpose of the **backlog** argument, we first observe that the **client** may call **connect**() before the **server** calls **accept**(). This could happen, for example, because the server is busy handling some other client(s).
![[Pasted image 20250811225823.png]]

- The **backlog** argument allows us to limit the number of such pending connections

- SUSv3 allows an implementation to place an upper limit on the value that can be specified for **backlog**. this limit by defining the constant **SOMAXCONN** in <sys/socket.h>

## 5.2 Accepting a Connection: **accept**()

- The accept() system call accepts an incoming connection on the listening stream socket referred to by the file descriptor **sockfd**. If there are no pending connections when accept() is called, the call blocks until a connection request arrives.

```c
#include <sys/socket.h>
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
//  Returns file descriptor on success, or –1 on error
```

## 5.3 Connecting to a Peer Socket: **connect**()


The **connect**() system call connects the active socket referred to by the file descriptor **sockfd** to the listening socket whose address is specified by **addr** and **addrlen**

```c
#include <sys/socket.h>
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen); 
// Returns 0 on success, or –1 on error
```

- The **addr** and **addrlen** arguments are specified in the same way as the corresponding arguments to **bind**()

- If **connect**() fails and we wish to reattempt the connection, then SUSv3 specifies that the portable method of doing so is to close the socket, create a new socket, and reattempt the connection with the new socket
## 5.4 I/O on Stream Sockets
- The semantics of I/O on connected stream sockets are similar to those for pipes

## 5.5 Connection Termination: **close**()

# 6 Datagram Sockets

- Each application that wants to send or receive datagrams creates a datagram socket using socket()
- In order to allow another application to send it datagrams , an application uses bind() to bind its socket to a well-known address.
- To send a datagram, an application calls **sendto**(), which takes as one of its arguments the address of the socket to which the datagram is to be sent.
- In order to receive a datagram, an application calls **recvfrom**(), which may block if no datagram has yet arrived. Because **recvfrom**() allows us to obtain the address of the sender, we can send a reply if desired
- When the socket is no longer needed, the application closes it using **close**()

![[Pasted image 20250812214316.png]]
## 6.1 Exchanging Datagrams: **recvfrom**() and **sendto**()

- The **recvfrom**() and **sendto**() system calls receive and send datagrams on a datagram socket

```c
#include <sys/socket.h>
ssize_t recvfrom(int sockfd, void *buffer, size_t length, int flags, struct sockaddr *src_addr, socklen_t *addrlen);
// Returns number of bytes received, 0 on EOF, or –1 on error

ssize_t sendto(int sockfd, const void *buffer, size_t length, int flags, const struct sockaddr *dest_addr, socklen_t addrlen);
// Returns number of bytes sent, or –1 on error
```
## 6.2 Using **connect**() with Datagram Sockets

- Calling **connect**() on a datagram socket **causes** the kernel to record a particular address as this socket’s peer. The term **connected datagram socket** is applied to such a socket

- The term **unconnected datagram socke**t is applied to a datagram socket on which **connect() has not been called**

- After a datagram socket has been connected:
	- **Datagrams** can be sent through the socket using **write**() (or **send**()) and are automatically sent to the same peer socket. As with **sendto**(), each **write**() call results in a separate datagram
	- Only datagrams sent by the peer socket may be read on the socket

- ["] **The above statements apply only to the socket on which connect() has been called, not to the remote socket to which it is connected (unless the peer application also calls connect() on its socket)**











