# 1 Creating or Opening a Message Queue

The **msgget**() system call creates a new message queue or obtains the identifier of an existing queue

```c
#include <sys/types.h> /* For portability */
#include <sys/msg.h>

int msgget(key_t key, int msgflg);
// Returns message queue identifier on success, or –1 on error
```
- The **msgflg** argument is a bit mask that specifies the permissions (Table 15-4,) to be placed on a new message queue or checked against an existing queue.
![[Pasted image 20250809155137.png]]

- In addition, zero or more of the following flags can be ORed (|) in msgflg to control the operation of **msgget**():

**IPC_CREAT**
	If no message queue with the specified key exists, create a new queue.

**IPC_EXCL**
	If **IPC_CREAT** was also specified, and a queue with the specified key already exists, fail with the error **EEXIST**
# 2 Exchanging Messages

The **msgsnd**() < for reading > and **msgrcv**() < for writing> system calls perform I/O on message queues.

- The first argument to both system calls (**msqid**) is a message queue identifier.

- The second argument, **msgp**, is a pointer to a programmer-defined structure used to hold the message being sent or received.This structure has the following general form:
```c
struct mymsg {
long mtype; /* Message type */
char mtext[]; /* Message body */
}
```

## 2.1 Sending Messages

The **msgsnd**() system call writes a message to a message queue:

```c
#include <sys/types.h> /* For portability */
#include <sys/msg.h>
int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);
// Returns 0 on success, or –1 on error
```

- we must set the **mtype** field of the message structure to a value greater than 0
- The **msgsz** argument specifies the number of bytes contained in the **mtext** field.

- ["] When sending messages with **msgsnd**(), there is no concept of a partial write as with **write**(). This is why a successful **msgsnd**() needs only to return 0, rather than the number of bytes sent. 
The final argument, **msgflg**, is a bit mask of flags controlling the operation of **msgsnd**(). Only one such flag is defined: **IPC_NOWAIT**

## 2.2 Receiving Messages

The **msgrcv**() system call reads (and removes) a message from a message queue, and copies its contents into the buffer pointed to by **msgp**

```c
#include <sys/types.h> /* For portability */
#include <sys/msg.h>
ssize_t msgrcv(int msqid, void *msgp, size_t maxmsgsz, long msgtyp, int msgflg); 
// Returns number of bytes copied into mtext field, or –1 on error
```

- The maximum space available in the **mtext** field of the **msgp** buffer is specified by the argument **maxmsgsz**.
- If the body of the message to be removed from the queue exceeds **maxmsgsz** bytes, then no message is removed from the queue, and **msgrcv**() fails with the error **E2BIG**
-  Messages need not be read in the order in which they were sent. Instead, we can select messages according to the value in the **mtype** field:
	- If **msgtyp** equals 0, the first message from the queue is removed and returned to the calling process.
	- If **msgtyp** is greater than 0, the first message in the queue whose **mtype** equals **msgtyp** is removed and returned to the calling process. By specifying different values for msgtyp, multiple processes can read from a message queue without racing to read the same messages. One useful technique is to have each process select messages matching its process ID.
	- If **msgtyp** is less than 0, treat the waiting messages as a priority queue. The first message of the lowest **mtype** less than or equal to the absolute value of **msgtyp** is removed and returned to the calling process.
- The **msgflg** argument is a bit mask formed by ORing together zero or more of the following flags:
	- **IPC_NOWAIT** : Perform a nonblocking receive.
	- **MSG_EXCEPT**:  This flag has an effect only if msgtyp is greater than 0, in which case it forces the complement of the usual operation; that is, the first message from the queue whose mtype is not equal to msgtyp is removed from the queue and returned to the caller
	- **MSG_NOERROR**: By default, if the size of the mtext field of the message exceeds the space available (as defined by the maxmsgsz argument), msgrcv() fails. If the MSG_NOERROR flag is specified, then msgrcv() instead removes the message from the queue, truncates its mtext field to maxmsgsz bytes, and returns it to the caller. The truncated data is lost

![[Pasted image 20250809162500.png]]

- As with msgsnd(), if a blocked msgrcv() call is interrupted by a signal handler, then the call fails with the error EINTR, regardless of the setting of the SA_RESTART flag when the signal handler was established
# 3 Message Queue Control Operations
- The **msgctl**() system call performs control operations on the message queue identified by **msqid**.
```c
#include <sys/types.h> /* For portability */
#include <sys/msg.h>
int msgctl(int msqid, int cmd, struct msqid_ds *buf);
// Returns 0 on success, or –1 on error
```
- The **cmd** argument specifies the operation to be performed on the queue. It can be one of the following: IPC_RMID, IPC_STAT, IPC_SET
#  4 Message Queue Associated Data Structure

![[Pasted image 20250809163408.png]]



# 5 Message Queue Limits
![[Pasted image 20250809163710.png]]

![[Pasted image 20250809163740.png]]



# 7 Client-Server Programming with Message Queues
In this section, we consider two of various possible designs for client-server applications using System V message queues:
- The use of a single message queue for exchanging messages in both directions between server and client.
![[Pasted image 20250810101622.png]]
- The use of separate message queues for the server and for each client. The server’s queue is used to receive incoming client requests, and responses are sent to clients via the individual client queues

# 8 A File-Server Application Using Message Queues
![[Pasted image 20250810101909.png]]

# 9 Disadvantages of System V Message Queues



