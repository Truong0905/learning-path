# 1 A Taxonomy of IPC Facilities
- **Communication**: These facilities are concerned with exchanging data between processes.
- **Synchronization**: These facilities are concerned with synchronizing the actions of processes or thread
- **Signals**:  they can be used as a synchronization technique in certain circumstances. More rarely, signals can be used as a communication technique
![[Pasted image 20250731214820.png]]

# 2 Communication Facilities
We can break the communication facilities into two categories:
- **Data-transfer facilities**:These facilities require two data transfers between user memory and kernel memory: one transfer from user memory to kernel memory during writing, and another transfer from kernel memory to user memory during reading
![[Pasted image 20250731215509.png]]
- **Shared memory**: Shared memory allows processes to exchange information by placing it in a region of memory that is shared between the processes. Communication doesn’t require system calls or data transfer between user memory and kernel memory, shared memory can provide very fast communication
![[Pasted image 20250731215732.png]]


# 3 Synchronization Facilities
UNIX systems provide the following synchronization facilities:
- **Semaphores**: A semaphore is a kernel-maintained integer whose value is never permitted to fall below 0. Linux provides both System V semaphores and POSIX semaphores, which have essentially similar functionality
- **File locks**: File locks are a synchronization method explicitly designed to coordinate the actions of multiple processes operating on the same file. They can also be used to coordinate access to other shared resources. Any number of processes can hold a read lock on the same file (or region of a file). However, when one process holds a write lock on a file (or file region), other processes are prevented from holding either read or write locks on that file (or file region).
- **Mutexes and condition variables**: These synchronization facilities are normally used with POSIX threads

![[Pasted image 20250731222154.png]]









