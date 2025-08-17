# 1 Overview of **fork**(), **exit**(), **wait**(), and **execve**()
- **fork()** : This is done by making the new child process an (almost) exact duplicate of the parent: the child obtains copies of the parent’s stack, data, heap, and text segments
- The **exit**(status) library function terminates a process, making all resources (memory, open file descriptors, and so on) used by the process available for subsequent reallocation by the kernel. The **status** argument is an integer that determines the termination status for the process. Using the **wait**() system call, the parent can retrieve this status
- The **wait**(**&status**) system call has two purposes. First, if a child of this process has not yet terminated by calling exit(), then wait() suspends execution of the process until one of its children has terminated. Second, the termination status of the child is returned in the status argument of wait().
- **execve(pathname, argv, envp)** : system call loads a new program (pathname, with argument list argv, and environment list envp) into a process’s memory. The existing program text is discarded, and the stack, data, and heap segments are freshly created for the new program.
![[Pasted image 20250617232121.png]]
# 2 Creating a New Process: **fork**()

```c
#include <unistd.h>
pid_t fork(void);
// In parent: returns process ID of child on success, or –1 on error;
// In successfully created child: always returns 0
```
- The two processes are executing the same program text, but they have **separate** copies of the **stack, data, and heap segments**. **The child’s [stack], [data], and [heap] segments** are initially exact **duplicates** of the corresponding parts the **parent’s memory** 

## 2.1 File Sharing Between Parent and Child
- The open file description contains the current file offset (as modified by **read**(), **write**(), and **lseek**()) and the open file status flags (set by **open**() and changed by the **fcntl**() F_SETFL operation). Consequently, these attributes of an open file are shared between the parent and child. For example, if the child updates the file offset, this change is visible through the corresponding descriptor in the parent.
![[Pasted image 20250617233934.png]]

## 2.2 Memory Semantics of fork()

# 3 The **vfork**() System Call


```c
#include <unistd.h>
pid_t vfork(void);

// In parent: returns process ID of child on success, or –1 on error; 
// in successfully created child: always returns 0

```

- Two features distinguish the **vfork**() system call from **fork**() and make it more efficient:
	- No duplication of virtual memory pages or page tables is done for the child process. Instead, the child shares the parent’s memory until it either performs a successful **exec**() or calls **_exit**() to terminate.
	- Execution of the parent process is suspended until the child has performed an **exec**() or _exit().
- ["] Since the child is using the parent’s memory, any changes made by the child to the data, heap, or stack segments will be visible to the parent once it resumes. Furthermore, if the child performs a function return between the vfork() and a later exec() or \_exit(), this will also affect the parent. This is similar to the example described in Section 6.8 of trying to longjmp() into a function from which a return has already been performed. Similar chaos—typically a segmentation fault (SIGSEGV)—is likely to result.
- ["] The semantics of vfork() mean that after the call, the child is guaranteed to be scheduled for the CPU before the parent

# 4 Race Conditions After **fork**()

- **After a fork(), it is indeterminate which process—the parent or the child—next has access to the CPU. Applications that implicitly or explicitly rely on a particular sequence of execution in order to achieve correct results are open to failure due to race conditions**
# 5 Avoiding Race Conditions by Synchronizing with Signals


# 6 Process Accounting

- When process accounting is enabled, the kernel writes an accounting record to **the system-wide process accounting file** as each process terminates
-  Including its termination status and how much CPU time it consumed
- Xan be analyzed by standard tools (**sa**(8) summarizes information from the accounting file, and **lastcomm**(1) lists information about previously executed commands) or by tailored applications
- Enabling and disabling process accounting:
```c
#define _BSD_SOURCE
#include <unistd.h>
int acct(const char *acctfile);
// Returns 0 on success, or –1 on error
```
- The **acct**() system call is used by a privileged (CAP_SYS_PACCT) process to enable and disable process accounting.
- Normally, process accounting is enabled at each system restart by placing appropriate commands in the system boot scripts
- To enable process accounting, we supply the pathname of an existing regular file in **acctfile**. A typical pathname for the accounting file is **/var/log/pacct** or **/usr/account/ pacct**. To disable process accounting, we specify **acctfile** as NULL

- **Process accounting records** : Once process accounting is enabled, an acct record is written to the accounting file as each process terminates. The **acct** structure is defined in **<sys/acct.h>** as follows
```c
typedef u_int16_t comp_t; /* See text */

struct acct {
char ac_flag; /* Accounting flags (see text) */
u_int16_t ac_uid; /* User ID of process */
u_int16_t ac_gid; /* Group ID of process */
u_int16_t ac_tty; /* Controlling terminal for process (may be
0 if none, e.g., for a daemon) */
u_int32_t ac_btime; /* Start time (time_t; seconds since the Epoch) */
comp_t ac_utime; /* User CPU time (clock ticks) */
comp_t ac_stime; /* System CPU time (clock ticks) */
comp_t ac_etime; /* Elapsed (real) time (clock ticks) */
comp_t ac_mem; /* Average memory usage (kilobytes) */
comp_t ac_io; /* Bytes transferred by read(2) and write(2)
(unused) */

comp_t ac_rw; /* Blocks read/written (unused) */
comp_t ac_minflt; /* Minor page faults (Linux-specific) */
comp_t ac_majflt; /* Major page faults (Linux-specific) */
comp_t ac_swaps; /* Number of swaps (unused; Linux-specific) */
u_int32_t ac_exitcode; /* Process termination status */
#define ACCT_COMM 16
char ac_comm[ACCT_COMM+1];
/* (Null-terminated) command name
(basename of last execed file) */
char ac_pad[10]; /* Padding (reserved for future use) */
};
```

![[Pasted image 20250706151816.png]]

- **Process accounting Version 3 file format**: Starting with kernel 2.6.8, Linux introduced an optional alternative version of the process accounting file that addresses some limitations of the traditional accounting file. To use this alternative version, known as Version 3, the **CONFIG_BSD_PROCESS_ACCT_V3** kernel configuration option must be enabled before building the kernel
```c
struct acct_v3 {

char ac_flag; /* Accounting flags */

char ac_version; /* Accounting version (3) */

u_int16_t ac_tty; /* Controlling terminal for process */

u_int32_t ac_exitcode; /* Process termination status */

u_int32_t ac_uid; /* 32-bit user ID of process */

u_int32_t ac_gid; /* 32-bit group ID of process */

u_int32_t ac_pid; /* Process ID */

u_int32_t ac_ppid; /* Parent process ID */

u_int32_t ac_btime; /* Start time (time_t) */

float ac_etime; /* Elapsed (real) time (clock ticks) */

comp_t ac_utime; /* User CPU time (clock ticks) */

comp_t ac_stime; /* System CPU time (clock ticks) */

comp_t ac_mem; /* Average memory usage (kilobytes) */

comp_t ac_io; /* Bytes read/written (unused) */

comp_t ac_rw; /* Blocks read/written (unused) */

comp_t ac_minflt; /* Minor page faults */

comp_t ac_majflt; /* Major page faults */

comp_t ac_swaps; /* Number of swaps (unused; Linux-specific) */

#define ACCT_COMM 16

char ac_comm[ACCT_COMM]; /* Command name */

};
```

# 7 The **clone**() System Call
- Like **fork**() and **vfork**(), the Linux-specific **clone**() system call creates a new process
- Because clone() is not portable, its direct use in application programs should normally be avoided
```c
#define _GNU_SOURCE 
#include <sched.h>
int clone(int (*func) (void *), void *child_stack, int flags, void *func_arg, ...
						/* pid_t *ptid, struct user_desc *tls, pid_t *ctid */ );

Returns process ID of child on success, or –1 on error
```
- Unlike **fork**(), the cloned child doesn’t continue from the point of the call, but instead commences by calling the function specified in the **func** argument => **child function**
- When called, the child function is passed the value specified in **func_arg**
- The cloned child process terminates either when **func** returns (in which case its return value is the exit status of the process) or when the process makes a call to exit() (or \_exit())
- Since a cloned child may (like **vfork**()) share the parent’s memory, it can’t use the parent’s stack. Instead, the caller must allocate a suitably sized block of memory for use as the child’s stack and pass a pointer to that block in the argument **child_stack**. On most hardware architectures, the stack grows downward, so the **child_stack** argument should point to the high end of the allocated block
- The clone() **flags** argument serves two purposes. First, its lower byte specifies the child’s termination signal, which is the signal to be sent to the parent when the child terminates. (If a cloned child is stopped by a signal, the parent still receives **SIGCHLD**.) This byte may be 0, in which case no signal is generated. The remaining bytes of the **flags** argument hold a bit mask that controls the operation of clone()

![[Pasted image 20250706152712.png]]

## 7.1 The clone() flags Argument

The **clone() flags** argument is a combination (ORing) of the bit-mask values

# 8 Speed of Process Creation
![[Pasted image 20250706164144.png]]




=============================================================

# 1 Terminating a Process: _exit() and exit()

```c
#include <unistd.h> 
void _exit(int status);
```
=> Just only exit

```c
#include <stdlib.h> 
void exit(int status);
```
- The following actions are performed by exit():
	- Exit handlers (functions registered with atexit() and on_exit()) are called, in reverse order of their registration
	- The stdio stream buffers are flushed.
	- The \_exit() system call is invoked, using the value supplied in status.

# 2 Details of Process Termination
- Open file descriptors, directory streams (Section 18.8), message catalog descriptors (see the catopen(3) and catgets(3) manual pages), and conversion descriptors (see the iconv_open(3) manual page) are closed.

-  As a consequence of closing file descriptors, any file locks (Chapter 55) held by this process are released.

- Any attached System V shared memory segments are detached, and the  shm_nattch counter corresponding to each segment is decremented by one. (Refer to Section 48.8.)

-  For each System V semaphore for which a semadj value has been set by the process, that semadj value is added to the semaphore value. (Refer to Section 47.8.)

- If this is the controlling process for a controlling terminal, then the SIGHUP signal is sent to each process in the controlling terminal’s foreground process group, and the terminal is disassociated from the session. We consider this point further in Section 34.6.

- Any POSIX named semaphores that are open in the calling process are closed as though sem_close() were called.

-  Any POSIX message queues that are open in the calling process are closed as though mq_close() were called.

- If, as a consequence of this process exiting, a process group becomes orphaned and there are any stopped processes in that group, then all processes in the group are sent a SIGHUP signal followed by a SIGCONT signal. We consider this point further in Section 34.7.4.

- Any memory locks established by this process using mlock() or mlockall() (Section 50.2) are removed.

- Any memory mappings established by this process using mmap() are unmapped.

# 3 Exit Handler

Registering exit handlers
```c
#include <stdlib.h> 
int atexit(void (*func)(void));
// Returns 0 on success, or nonzero on error
```
- The **atexit**() function adds func to a list of functions that are called when the process terminates. The function func should be defined to take no arguments and return no value
- It is possible to register multiple exit handlers (and even the same exit handler multiple times). When the program invokes **exit**(), these functions are called in **reverse order of registration**.
- A child process created via **fork**() inherits a copy of its parent’s exit handler registrations.When a process performs an **exec**(), all exit handler registrations are removed.
- An exit handler doesn’t know what status was passed to exit(). The second limitation is that we can’t specify an argument to the exit handler when it is called
	=> To address these limitations, glibc provides a (nonstandard) alternative method of registering exit handlers: on_exit()
```c
#define _BSD_SOURCE /* Or: #define _SVID_SOURCE */

#include <stdlib.h>

int on_exit(void (*func)(int, void *), void *arg);

// Returns 0 on success, or nonzero on error
```

- When called, **func**() is passed two arguments: the **status** argument supplied to **exit**(), and a copy of the **arg** argument supplied to **on_exit**() at the time the function was registered

# 4 Interactions Between fork(), stdio Buffers, and _exit()


=====================================================================

# 1 Waiting on a Child Process

## 1.1 The **wait**() System Call

- The **wait**() system call waits for one of the **children** of the calling process to **terminate** and returns the termination status of that child in the buffer pointed to by status.
```c
#include <sys/wait.h>
pid_t wait(int *status);
// Returns process ID of terminated child, or –1 on error
```
- The call blocks until one of the children terminates.
- If **status** is not NULL, information about how the child terminated is returned in the integer to which **status** points
- On error, wait() returns –1. One possible error is that the calling process has no (previously unwaited-for) children, which is indicated by the **errno** value **ECHILD**

## 1.2 The **waitpid**() System Call
```c
#include <sys/wait.h>

pid_t waitpid(pid_t pid, int *status, int options);

// Returns process ID of child, 0 (see text), or –1 on error
```
- The return value and **status** arguments of **waitpid**() are the same as for **wait**().
- The **pid** argument enables the selection of the child to be waited for, as follows:
	- If **pid** is greater than 0, wait for the child whose **process** ID equals **pid**.
	- If **pid** equals 0, wait for any child in the **same process group as the caller (parent)**
	- If **pid** is less than –1, wait for any child whose **process group** identifier equals the absolute value of **pid**
	- If **pid** equals –1, wait for any child. The call **wait(&status)** is equivalent to the call **waitpid(–1, &status, 0).**
- The **options** argument is a bit mask that can include (OR) zero or more of the following flags

## 1.3 The Wait Status Value
- The <sys/wait.h> header file defines a standard set of macros that can be used to dissect a wait status value
- Additional macros are provided to further dissect the status value, as noted in the list: 
![[Pasted image 20250628100305.png]]


## 1.4 Process Termination from a Signal Handler

- For example, calling \_exit(EXIT_SUCCESS) from the signal handler will make it appear to the parent process that the child terminated successfully. But If the child needs to inform the parent that it terminated because of a signal, then the child’s signal handler should first disestablish itself, and then raise the same signal once more, which this time will terminate the process. The signal handler would contain code such as the following:
```c
void handler(int sig) {

/* Perform cleanup steps */

signal(sig, SIG_DFL); /* Disestablish handler */

raise(sig); /* Raise signal again */

}
```

## 1.5 The **waitid**() System Call
```c
#include <sys/wait.h>
int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options);
// Returns 0 on success or if WNOHANG was specified and there were no children to wait for, or –1 on error
```
- The **idtype** and **id** arguments specify which child(ren) to wait for, as follows:
	- If **idtype** is P_ALL, wait for any child; **id** is ignored.
	- If **idtype** is P_PID, wait for the child whose process ID equals **id**
	- If **idtype** is P_PGID, wait for any child whose process group ID equals **id**.
- We control this by ORing one or more of the following flags in **options**:
![[Pasted image 20250703211424.png]]
![[Pasted image 20250703211501.png]]


- On succes, The following fields are filled in the siginfo_t structure:
![[Pasted image 20250703211645.png]]


## 1.6 The **wait3**() and **wait4**() System Calls


```C
#define _BSD_SOURCE /* Or #define _XOPEN_SOURCE 500 for wait3() */
#include <sys/resource.h> 
#include <sys/wait.h>

pid_t wait3(int *status, int options, struct rusage *rusage);

pid_t wait4(pid_t pid, int *status, int options, struct rusage *rusage);

// Both return process ID of child, or –1 on error
```
- The principal semantic difference is that **wait3**() and **wait4**() return resource usage information about the terminated child in the structure pointed to by the **rusage** argument.
- The names for these two system calls refer to the number of arguments they each take.

# 2 Orphans and Zombies

- **Who becomes the parent of an orphaned child?**  : **init** with pid is 1.

- **What happens to a child that terminates before its parent has had a chance to perform a wait()?**
	- The kernel deals with this situation by turning the child into a zombie.
	- Most of the resources held by the child are released back to the system to be reused by other processes
	- The only part of the process that remains is an entry in the kernel’s process table recording (among other things) the child’s process ID, termination status, and resource usage statistics
- A zombie process can not be killed by a signal. This ensures prent can always eventually perform a **wait**()
- When the parent does perform a **wait**(), the kernel removes the zombie, since the last remaining information about the child is no longer required.
- if the parent terminates without doing a **wait**(), then the **init** process adopts the child and automatically performs a **wait**(), thus removing the zombie process from the system
- If a parent creates a child, but fails to perform a wait(), then an entry for the zombie child will be maintained indefinitely in the kernel’s process table. If a large number of such zombie children are created, they will eventually fill the kernel process table, preventing the creation of new processes. Since the zombies can’t be killed by a signal, the only way to remove them from the system is to kill their parent (or wait for it to exit), at which time the zombies are adopted and waited on by init, and consequently removed from the system
- If a parent creates a child, but fails to perform a wait(), the only way to remove them from the system is to kill their parent (or wait for it to exit), at which time the zombies are adopted and waited on by init, and consequently removed from the system.

# 3 The **SIGCHLD** Signal
## 3.1 Establishing a Handler for **SIGCHLD**

- The **SIGCHLD** signal is sent to a parent process whenever one of its children terminates
- By default, this signal is ignored, but we can catch it by installing a signal handler.
- Within the signal handler, we can use wait() (or similar) to reap the zombie child.
	- With this approach, we will maybe miss second, third chil terminate due to the signal that caused its invocation is temporarily blocked (unless the sigaction() SA_NODEFER flag was specified), and also that standard signals, of which SIGCHLD is one, are not queued.
	- The solution is to loop inside the SIGCHLD handler, repeatedly calling waitpid() with the WNOHANG flag until there are no more dead children to be reaped.
```c
pid_t pid;
int status;
while ((pid = waitpid(-1, &status, WNOHANG)) > 0);
```
- **we noted that using a system call (e.g., waitpid()) from within a signal handler may change the value of the global variable *errno***
	- For this reason, it is sometimes necessary to code a **SIGCHLD** handler to save **errno** in a local variable on entry to the handler, and then restore the errno value just prior to returning
## 3.2 Delivery of **SIGCHLD** for Stopped Children

- when using **sigaction**() to establish a handler for the **SIGCHLD** signal:
	- If **SA_NOCLDSTOP** flag is omitted, a **SIGCHLD** signal is delivered to the parent when one of its children stops;
	- if the **SA_NOCLDSTOP** flag is present, **SIGCHLD** is not delivered for stopped children. (The implementation of **signal**() given in Section 22.7 doesn’t specify **SA_NOCLDSTOP**.)
## 3.3 Ignoring Dead Child Processes
- There is a further possibility for dealing with dead child processes is to set the disposition of **SIGCHLD** to **SIG_IGN** (ignore) ( any child process that subsequently terminates to be immediately removed from the system instead of being converted into a zombie )
	- => In this case, since the status of the child process is simply discarded, a subsequent call to **wait**() (or similar) can’t return any information for the terminated child. We don;t need to call wait anymore in this case
	- ["] On Linux, as on many UNIX implementations, setting the disposition of SIGCHLD to SIG_IGN doesn’t affect the status of any existing zombie children, which must still be waited upon in the usual way. On some other UNIX implementations (e.g., Solaris 8), setting the disposition of SIGCHLD to SIG_IGN does remove existing zombie children
- The sigaction() SA_NOCLDWAIT flag