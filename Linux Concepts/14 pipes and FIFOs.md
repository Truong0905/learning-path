![[Pasted image 20250806211937.png]]
# 1 Overview
- A pipe is a byte stream : 
	- The process reading from a pipe can read blocks of data of any size, regardless of the size of blocks written by the writing process.
	- The data passes through the pipe sequentially— bytes are read from a pipe in exactly the order they were written
	- It is not possible to randomly access the data in a pipe using **lseek**()
- Reading from a pipe:
	- Attempts to read from a pipe that is currently empty block until at least one byte has been written to the pipe
	- If the write end of a pipe is closed, then a process reading from the pipe will see end-of-file (i.e., **read**() returns 0) once it has read all remaining data in the pipe
- Pipes are unidirectional
	- Data can travel only in one direction through a pipe. One end of the pipe is used for writing, and the other end is used for reading
- Writes of up to **PIPE_BUF** bytes are guaranteed to be atomic:
	- If multiple processes are writing to a single pipe, then it is guaranteed that their data won’t be intermingled if they write no more than **PIPE_BUF** bytes at a time
	- SUSv3 requires that **PIPE_BUF** be at least \**_POSIX_PIPE_BUF*\* (512). An implementation should define PIPE_BUF (in <limits.h>) and/or allow the call **fpathconf**(fd, \_PC_PIPE_BUF) to return the actual upper limit for atomic writes. PIPE_BUF varies across UNIX implementations; for example, it is 512 bytes on FreeBSD 6.0, 4096 bytes on Tru64 5.1, and 5120 bytes on Solaris 8. On Linux, PIPE_BUF has the value 4096
- Pipes have a limited capacity

# 2 Creating and Using Pipes
- The **pipe**() system call creates a new pipe.
```c
#include <unistd.h>
int pipe(int filedes[2]);
// Returns 0 on success, or –1 on error
```
- A successful call to **pipe**() returns two open file descriptors in the array filedes: one for the read end of the pipe ( **filedes**[0]) and one for the write end ( **filedes**[1]).
![[Pasted image 20250806215119.png]]
- To connect two processes using a pipe, we follow the **pipe**() call with a call to **fork**(). During a **fork**(), the child process inherits copies of its parent’s file descriptors
- While it is possible for the parent and child to both read from and write to the pipe, this is not usual. Therefore, immediately after the fork(), one process closes its descriptor for the write end of the pipe, and the other closes its descriptor for the read end
![[Pasted image 20250806215309.png]]

- **Pipes allow communication between related processes**:
	- Pipes can be used for communication between any two (or more) related processes, as long as the pipe was created by a common ancestor before the series of **fork**() calls
	- A common scenario is that a pipe is used for communication between two siblings—their parent creates the pipe, and then creates the two children. This is what the shell does when building a pipeline
	
- **Closing unused pipe file descriptors**:
	- Closing unused pipe file descriptors is more than a matter of ensuring that a process doesn’t exhaust its limited set of file descriptors—it is essential to the correct use of pipes
	- The process reading from the pipe closes its write descriptor for the pipe, so that, when the other process completes its output and closes its write descriptor, the reader sees end-of-file. if not, the reader won’t see end-of-file even after it has read all data from the pipe.
	- The writing process closes its read descriptor for the pipe for a different reason. When a process tries to write to a pipe for which n**o process has an open read descriptor, the kernel sends the SIGPIPE signal to the writing process.** By default, this signal kills a process. Process can instead arrange to catch or ignore this signal, in which case the write() on the pipe fails with the error EPIPE (broken pipe).
	- One final reason for closing unused file descriptors is that it is only after all file descriptors in all processes that refer to a pipe are closed that the pipe is destroyed and its resources released for reuse by other processes
# 3 Pipes as a Method of Process Synchronization

Example:
To perform the synchronization, the parent builds a pipe  before creating the child processes . Each child inherits a file descriptor for the write end of the pipe and closes this descriptor once it has completed its action . After all of the children have closed their file descriptors for the write end of the pipe, the parent’s read()  from the pipe will complete, returning end-of-file (0). At this point, the parent is free to carry on to do other work. (Note that closing the unused write end of the pipe in the parent  is essential to the correct operation of this technique; otherwise, the parent would block forever when trying to read from the pipe.)
# 4 Using Pipes to Connect Filters
# 5 Talking to a Shell Command via a Pipe: **popen**()

```c
#include <stdio.h>

FILE *popen(const char *command, const char *mode);
// Returns file stream, or NULL on error 

int pclose(FILE *stream);
// Returns termination status of child process, or –1 on error
```

- The **popen**() function creates a pipe, and then forks a child process that execs a **shell**, which in turn creates a child process to execute the string given in **command**.
- The **mode** argument is a string that determines whether the calling process will read from the pipe (**mode** is **r**) or write to it (**mode** is **w**)

![[Pasted image 20250806224145.png]]


# 6 Pipes and stdio Buffering

# 7 FIFOs
```c
#include <sys/stat.h>

int mkfifo(const char *pathname, mode_t mode);

// Returns 0 on success, or –1 on error
```

# 8 A Client-Server Application Using FIFOs
- In the example application, all clients send their requests to the server using a single server FIFO.
- The header file defines the well-known name (/tmp/seqnum_sv) that the server uses for its FIFO.
- This name is fixed, so that all clients know how to contact the server.
- the client to generate its FIFO pathname, and then pass the pathname as part of its request message
![[Pasted image 20250807213607.png]]

- sender and receiver must agree on some convention for separating the messages
	- Terminate each message with a delimiter character
	- Include a fixed-size header with a length field in each message specifying the number of bytes in the remaining variable-length component of the message.
	- Use fixed-length messages, and have the server always read messages of this fixed size.

( Be aware that for each of these techniques, the total length of each message must be smaller than **PIPE_BUF** bytes in order to avoid the possibility of messages being broken up by the kernel and interleaved with messages from other writers )
![[Pasted image 20250807213856.png]]

# 9 Nonblocking I/O

- Sometimes, it is desirable not to block, and for this purpose, the O_NONBLOCK flag can be specified when calling open():

```c
fd = open("fifopath", O_RDONLY | O_NONBLOCK); 
if (fd == -1)
	errExit("open");
```
- If the FIFO is being opened for reading, and no process currently has the write end of the FIFO open, then the open() call succeeds immediately ( just as though the other end of the FIFO was already open)
- If the FIFO is being opened FIFO for writing, and the other end of the FIFO is not already open for reading, then open() fails, setting errno to ENXIO.

![[Pasted image 20250807220438.png]]
![[Pasted image 20250807221018.png]]
![[Pasted image 20250807221031.png]]






