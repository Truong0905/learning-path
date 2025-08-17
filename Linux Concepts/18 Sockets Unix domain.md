
**UNIX domain sockets**, which allow communication between processes on the same host system.


# 1 UNIX Domain Socket Addresses: **struct sockaddr_un**
```c
struct sockaddr_un {
	sa_family_t sun_family; /* Always AF_UNIX */
	char sun_path[108]; /* Null-terminated socket pathname */
};
```

![[Pasted image 20250814215342.png]]

 The following points are worth noting about binding a UNIX domain socket:
-  We can’t bind a socket to an existing pathname (bind() fails with the error **EADDRINUSE**)
- It is usual to bind a socket to an **absolute pathname**, so that the socket **resides** at a **fixed address** in the file system.
- A socket may be bound to only one pathname
- We can’t use **open**() to open a socket
- When the socket is no longer required, its pathname entry can (and generally should) be removed using **unlink**() (or **remove**())


# 2 Stream Sockets in the UNIX Domain

# 3 Datagram Sockets in the UNIX Domain

- However, for UNIX domain sockets, datagram transmission is carried out within the kernel, and is reliable. All messages are delivered in order and unduplicated.

- Maximum datagram size for UNIX domain datagram sockets: . On Linux, we can send quite large datagrams. The limits are controlled via the SO_SNDBUF socket option and various /proc files, as described in the socket(7) manual page.
# 4 UNIX Domain Socket Permissions

- The ownership and permissions of the socket file determine which processes are able to communicate with that socket:
	- To connect to a UNIX domain stream socket, write permission is required on the socket file.
	- To send a datagram to a UNIX domain datagram socket, write permission is required on the socket file.
- In addition, execute (search) permission is required on each of the directories in the socket pathname
- By default, a socket is created (by **bind**()) with all permissions granted to owner (user), group, and other. To change this, we can **precede** the **call** to **bind**() with a call to **umask**() to disable the permissions that we do not wish to grant

# 5 Creating a Connected Socket Pair: **socketpair**()
- Sometimes, it is useful for a single process to create a pair of sockets and connect them together. This could be done using two calls to socket(), a call to bind(), and then either calls to listen(), connect(), and accept() (for stream sockets), or a call to connect() (for datagram sockets). The socketpair() system call provides a shorthand for this operation.
```c
#include <sys/socket.h>
int socketpair(int domain, int type, int protocol, int sockfd[2]);
// Returns 0 on success, or –1 on error
```
- This **socketpair**() system call can be used only in the UNIX domain; that is, **domain** **must be** specified as **AF_UNIX**.
- The socket **type** may be specified as either **SOCK_DGRAM** or **SOCK_STREAM**. The **protocol** argument **must** be specified as **0**
- ["] Typically, a socket pair is used in a similar fashion to a pipe. After the socketpair() call, the process then creates a child via fork(). The child inherits copies of the parent’s file descriptors, including the descriptors referring to the socket pair. Thus, the parent and child can use the socket pair for IPC

- ["] One way in which the use of socketpair() differs from creating a pair of connected sockets manually is that the sockets are not bound to any address. This can help us avoid a whole class of security vulnerabilities, since the sockets are not visible to any other process

# 6 The Linux Abstract Socket Namespace

- This provides a few potential advantages:
	- We don’t need to worry about possible collisions with existing names in the file system
	- It is not necessary to unlink the socket pathname when we have finished using the socket. The abstract name is automatically removed when the socket is closed
	- We don’t need to create a file-system pathname for the socket. This may be useful in a chroot environment, or if we don’t have write access to a file system
- To create an abstract binding, we specify the first byte of the **sun_path** field as a **null** byte (\0). T**he remaining bytes** of the **sun_path** field then define **the abstract name** for the socket.


