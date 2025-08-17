# 1 Operations Outside the Universal I/O Model: **ioctl**()
```c
#include <sys/ioctl.h>
int ioctl(int fd, int request, ... /* argp */); 
// Value returned on success depends on request, or –1 on error
```
# 2 File Control Operations: fcntl()

```c
#include <fcntl.h>
int fcntl(int fd, int cmd, ...); 
//Return on success depends on cmd, or –1 on error
```
One use of fcntl() is to retrieve or modify the access mode and open file status flags of an open file. F_GETFL or F_SETFL ( The flags that can be modified are O_APPEND, O_NONBLOCK, O_NOATIME, O_ASYNC, and O_DIRECT. Attempts to modify other flags are ignored.)

# 3 Relationship Between File Descriptors and Open Files

- Three data structures maintained by the kernel:
	- the per-process **file descriptor table**; (A)
	- the **system-wide table** (B) of open file descriptions; 
	- and the file system **i-node** (C) table.
- For each process, the kernel maintains a **table of open file descriptors** (A).Each entry in this table records information about a single file descriptor, including:
	- a set of flags controlling the operation of the file descriptor (there is just one such flag, the close-on-exec flag); 
	- and a reference to the open file description.
- The kernel maintains a **system-wide table** (B) of all open file descriptions:
	- the current file offset (as updated by read() and write(), or explicitly modified using lseek());
	- status flags specified when opening the file (i.e., the flags argument to open());
	- the file access mode (read-only, write-only, or read-write, as specified in open())
	- settings relating to signal-driven I/O
	- a reference to the i-node object for this file
- Each file system has a table of **i-nodes**(C) for all files residing in the file system:
	- file type (e.g., regular file, socket, or FIFO) and permissions;
	- a pointer to a list of locks held on this file; and
	- various properties of the file, including its size and timestamps relating to dif ferent types of file operations.
![[Pasted image 20250608095337.png]]

# 4 Duplicating File Descriptors
- The **dup**() call takes oldfd, an open file descriptor, and returns a new descriptor that refers to the same open file description. The new descriptor is guaranteed to be the lowest unused file descriptor.
```c
#include <unistd.h>
int dup(int oldfd); 
// Returns (new) file descriptor on success, or –1 on error
```

- The **dup2**() system call makes a duplicate of the file descriptor given in oldfd using the descriptor number supplied in newfd. If the file descriptor specified in newfd is already open, dup2() closes it first.
```c
#include <unistd.h>
int dup2(int oldfd, int newfd); 
// Returns (new) file descriptor on success, or –1 on error
```
# 5 File I/O at a Specified Offset: **pread**() and **pwrite**()
- The pread() and pwrite() system calls operate just like read() and write(), except that the file I/O is performed at the location specified by offset, rather than at the cur rent file offset. **The file offset is left unchanged by these calls.**
```c
#include <unistd.h>
ssize_t pread(int fd, void *buf, size_t count, off_t offset);
//Returns number of bytes read, 0 on EOF, or –1 on error 
ssize_t pwrite(int fd, const void *buf, size_t count, off_t offset); 
///Returns number of bytes written, or –1 on error
```
# 6 Scatter-Gather I/O: **readv**() and **writev**()

The set of buffers to be transferred is defined by the array **iov**. The integer count specifies the number of elements in **iovcnt**.
```c
#include <sys/uio.h>
ssize_t readv(int fd, const struct iovec *iov, int iovcnt); 
//Returns number of bytes read, 0 on EOF, or –1 on error
ssize_t writev(int fd, const struct iovec *iov, int iovcnt); 
// Returns number of bytes written, or –1 on error

struct iovec {
void *iov_base; 
size_t iov_len; };

```

# 7 Performing scatter-gather I/O at a specified offset

```c
#define _BSD_SOURCE 
#include<sys/uio.h>
ssize_t preadv(int fd, const struct iovec *iov, int iovcnt, off_t offset); //Returns number of bytes read, 0 on EOF, or –1 on error
ssize_t pwritev(int fd, const struct iovec *iov, int iovcnt, off_t offset); 
// Returns number of bytes written, or –1 on error
```

# 8 Truncating a File: **truncate**() and **ftruncate**()
- The **truncate**() and **ftruncate**() system calls set the size of a file to the value specified by **length**.
```c
#include <unistd.h>
int truncate(const char *pathname, off_t length);
int ftruncate(int fd, off_t length);
// Both return 0 on success, or –1 on error
```

# 9 Nonblocking I/O
- Specifying the O_NONBLOCK flag when opening a file serves two purposes:
	- If the file can’t be opened immediately, then open() returns an error instead of blocking. One case where open() can block is with FIFOs .
	- After a successful open(), subsequent I/O operations are also nonblocking. If an I/O system call can’t complete immediately, then either a partial data trans fer is performed or the system call fails with one of the errors EAGAIN or EWOULDBLOCK.
# 10 I/O on Large Files
fopen64(), open64(), lseek64(), truncate64(), stat64(), mmap64(), and setrlimit64()
The \_FILE_OFFSET_BITS macro
```bash
$ cc -D_FILE_OFFSET_BITS=64 prog.c
```
```c
#define _FILE_OFFSET_BITS 64
```
# 11 Creating Temporary Files
- Some programs need to create temporary files that are used only while the pro gram is running, and these files should be removed when the program terminates.
```c
#include  <stdlib.h>
int mkstemp(char *template); 
// Returns file descriptor on success, or –1 on error
```
The **template** argument takes the form of a pathname in which the last 6 characters must be XXXXXX
```c
char template[] = "/tmp/somestringXXXXXX";
fd = mkstemp(template);
```
