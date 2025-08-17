**System V IPC** is the label used to refer to three different mechanisms for interprocess communication:
- **Message queues** : can be used to pass messages between processes
- **Semaphores** : permit multiple processes to synchronize their actions
- **Shared memory**
# 1 API Overview
- Some implementations require the inclusion of <sys/types.h> before including the header files shown in Table 45-1
 ![[Pasted image 20250808230509.png]]

**Creating and opening a System V IPC object**: (**msgget**(), **semget**(), or **shmget**())
- Whereas a file descriptor is a process attribute, an IPC identifier is a property of the object itself and is visible system-wide.
- If we know an IPC object already exists, we can skip the **get** ( also mean to create a new object) call, provided we have some other means of knowing the identifier of the object.  For example, the process that created the object might write the identifier to a file that can then be read by other processes.
**IPC object deletion and object persistence**: (**msgctl**(), **semctl**(), **shmctl**())
- For message queues and semaphores, deletion of the IPC object is immediate, and any information contained within the object is destroyed, regardless of whether any other process is still using the object.
- Deletion of shared memory objects occurs differently. Following the **shmctl**(id, IPC_RMID, NULL) call, the shared memory segment is removed only after all processes using the segment detach it (using **shmdt**()).
- System V IPC objects have kernel persistence. Once created, an object continues to exist until it is explicitly deleted or the system is shut down. This property of System V IPC objects can be advantageous.
# 2 IPC Keys
- The IPC **get** calls translate a key into the corresponding integer IPC identifier. These calls guarantee that if we create a new IPC object, then that object will have a unique identifier. If we specify the key of an existing object, then we’ll always obtain the (same) identifier for that object.
- **how do we provide a unique key—one that guarantees that we won’t accidentally obtain the identifier of an existing IPC object used by some other application?**:
	- Randomly choose some integer key value, which is typically placed in a header file included by all programs using the IPC object. The difficulty with this  approach is that we may accidentally choose a value used by another application.
	- Specify the **IPC_PRIVATE** constant as the **key** value to the get **call** when creating the IPC object, which always results in the creation of a new IPC object that is guaranteed to have a unique key.
	-  Employ the **ftok**() function to generate a (likely unique) key

```c
#include <sys/ipc.h>

key_t ftok(char *pathname, int proj);
// Returns integer key on success, or –1 on error
```


# 3 Associated Data Structure and Object Permissions

- The kernel maintains an associated data structure for each instance of a System V IPC object. The form of this data structure varies according to the IPC mechanism (message queue, semaphore, or shared memory) and is defined in the corresponding header file for the IPC mechanism
- Once the object has been created, a program can obtain a copy of this data structure using the appropriate **ctl** system call, by specifying an operation type of **IPC_STAT**. Conversely, some parts of the data structure can be modified using the **IPC_SET** operation

# 4 IPC Identifiers and Client-Server Applications

- In client-server applications, the server typically creates the System V IPC objects, while the client simply accesses them. In other words, the server performs an IPC get call specifying the flag IPC_CREAT, while the client omits this flag in its get call.
- What happens if the server process crashes or is deliberately halted and then restarted? At this point, it would make no sense to blindly reuse the existing IPC object created by the previous server process, since the new server process has no knowledge of the historical information associated with the current state of the IPC object **= >** In such a scenario, the only option for the server may be to abandon all existing clients, delete the IPC objects created by the previous server process, and create new instances of the IPC objects
= > A newly started server handles the possibility that a previous instance of the server terminated prematurely by first trying to create an IPC object by specifying both the IPC_CREAT and the IPC_EXCL flags within the get call. If the get call fails because an object with the specified key already exists, then the server assumes the object was created by an old server process; it therefore uses the IPC_RMID ctl operation to delete the object, and once more performs a get call to create the object

- Even if a restarted server re-created the IPC objects, there still would be a potential problem if supplying the same key to the get call always generated the same identifier whenever a new IPC object was created. Consider the solution just outlined from the point of view of the client. If the IPC objects re-created by the server use the same identifiers, then the client would have no way of becoming aware that the server has been restarted and that the IPC objects don’t contain the expected historical information.
= > To solve this problem, the kernel employs an algorithm (described in the next section) that normally ensures that when a new IPC object is created, the object’s identifier will be different, even when the same key is supplied.



# 5 Algorithm Employed by System V IPC **get** Calls

- Figure 45-1 shows some of the structures used internally by the kernel to represent information about System V IPC objects (in this case semaphores, but the details are similar for other IPC mechanisms):
![[Pasted image 20250809084436.png]]

- This information includes a dynamically sized array of pointers, **entries**
- The current size of the **entries** array is recorded in the **size** field
- the **max_id** field holding the index of the highest currently in-use element


When an IPC get call is made, the algorithm used on Linux (other systems use similar algorithms) is approximately as follows:
1. The list of associated data structures (pointed to by elements of the **entries** array) is searched for one whose key field matches that specified in the get call.
	a) If no match is found, and IPC_CREAT was not specified, then the error ENOENT 
	is returned.
	b) If a match is found, but both IPC_CREAT and IPC_EXCL were specified, then the	error EEXIST is returned.
	c) Otherwise, if a match is found, then the following step is skipped.
2.  If no match was found, and IPC_CREAT was specified, then a new mechanism-specific associated data structure (semid_ds in Figure 45-1) is allocated and initialized. This also involves updating various fields of the ipc_ids structure, and may involve resizing the entries array. A pointer to the new structure is placed in the first free element of entries. Two substeps are included as part of this initialization:
	a) The key value supplied in the get call is copied into the xxx_perm.\_\_key field of the newly allocated structure.
	b) The current value of the seq field of the ipc_ids structure is copied into the xxx_perm.\_\_seq field of the associated data structure, and the seq field is	incremented by one.
3. The identifier for the IPC object is calculated using the following formula:
	identifier = index + xxx_perm.\_\_seq * **SEQ_MULTIPLIER**
	In the formula used to calculate the IPC identifier, **index** is the index of this object instance within the **entries** array, and **SEQ_MULTIPLIER** is a constant defined with the value 32,768 (**IPCMNI** in the kernel source file **include/linux/ipc.h**).

# 6 The **ipcs** and **ipcrm** Commands
- The **ipcs** and **ipcrm** commands are the System V IPC analogs of the **ls** and rm **file** commands.

# 7 Obtaining a List of All IPC Objects
Linux provides two nonstandard methods of obtaining a list of all IPC objects on the system:
-  files within the /proc/sysvipc directory that list all IPC objects; and
-   the use of Linux-specific ctl calls

Three read-only files in the /proc/sysvipc directory provide the same information as can be obtained via ipcs:
- /proc/sysvipc/msg lists all messages queues and their attributes.
- /proc/sysvipc/sem lists all semaphore sets and their attributes.
 - /proc/sysvipc/shm lists all shared memory segments and their attributes
# 8 IPC Limits


