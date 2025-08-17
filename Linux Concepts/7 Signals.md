


# 1  Changing Signal Dispositions: **signal()**

```C
#include <signal.h>

/* Returns previous signal disposition on success, or SIG_ERR on error */
void ( *signal(int sig, void (*handler)(int)) ) (int);

typedef void (*sighandler_t)(int);
sighandler_t signal(int sig, sighandler_t handler);

void handler(int sig) 
{
	/* Code for the handler */
}
```
# 2 Sending Signals: **kill()**
```C
#include <signal.h> 
int kill(pid_t pid, int sig);
/* Returns 0 on success, or –1 on error */

```
![[Pasted image 20250604221450.png]]- **A privileged (CAP_KILL) process may send a signal to any process.**

# 3 Checking for the Existence of a Process
- ["] The kill() system call can serve another purpose. If the sig argument is specified as 0 (the so-called null signal), then no signal is sent. Instead, kill() merely performs error checking to see if the process can be signaled. Read another way, this means we can use the null signal to test if a process with a specific process ID exists. If sending a null signal fails with the error ESRCH, then we know the process doesn’t exist.
# 4 Other Ways of Sending Signals: **raise()** and **killpg()**

- **Sometimes, it is useful for a process to send a signal to itself.**
```c
#include <signal.h>
int raise(int sig);
/* Returns 0 on success, or nonzero on error */

/* In a single-threaded program, a call to raise() is equivalent to the following call to kill():  */
kill(getpid(), sig);

/* On a system that supports threads, raise(sig) is implemented as: */
pthread_kill(pthread_self(), sig);
```

- **The killpg() function sends a signal to all of the members of a process group.**
```c
#include <signal.h>
int killpg(pid_t pgrp, int sig);
// Returns 0 on success, or –1 on error

// A call to killpg() is equivalent to the following call to kill():
kill(-pgrp, sig);

```

#  5 Displaying Signal Descriptions
```c
#define _BSD_SOURCE 
#include <signal.h>

extern const char *const sys_siglist[];

#define _GNU_SOURCE 
#include <string.h>

char *strsignal(int sig);
// Returns pointer to signal description string

```

# 6 Signal Sets
- **sigaction()** and **sigprocmask()** allow a program to specify a group of signals that are to be blocked by a process, while **sigpending()** returns a group of signals that are currently pending for a process.
- The **sigemptyset**() function initializes a signal set to contain no members. The sigfillset() **function** initializes a set to contain all signals (including all realtime signals)
```c
#include <signal.h>

int sigemptyset(sigset_t *set); 
int sigfillset(sigset_t *set);
// Both return 0 on success, or –1 on error
```

- After initialization, individual signals can be added to a set using **sigaddset**() and removed using **sigdelset**(). For both sigaddset() and sigdelset(), the **sig** argument is a signal number
```c
#include <signal.h>
int sigaddset(sigset_t *set, int sig);
int sigdelset(sigset_t *set, int sig);
// Both return 0 on success, or –1 on error
```

- The **sigismember**() function is used to test for membership of a set.
```c
#include <signal.h>

int sigismember(const sigset_t *set, int sig);

// Returns 1 if sig is a member of set, otherwise 0
```

# 7 The Signal Mask (Blocking Signal Delivery)

- For each process, the kernel maintains a *signal mask*—a set of signals whose delivery to the process is currently blocked
- The **sigprocmask**() system call can be used at any time to explicitly add signals to, and remove signals from, the signal mask
```c
#include <signal.h>

int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);

// Returns 0 on success, or –1 on error
```


```c
sigset_t blockSet, prevMask;

/* Initialize a signal set to contain SIGINT */
sigemptyset(&blockSet);
sigaddset(&blockSet, SIGINT);

/* Block SIGINT, save previous signal mask */
if (sigprocmask(SIG_BLOCK, &blockSet, &prevMask) == -1) 
	errExit("sigprocmask1");

/* ... Code that should not be interrupted by SIGINT ... */

/* Restore previous signal mask, unblocking SIGINT */
if (sigprocmask(SIG_SETMASK, &prevMask, NULL) == -1) 
	errExit("sigprocmask2");
```

# 8 Pending Signals
- To determine which signals are pending for a process, we can call **sigpending**(). The **sigpending**() system call returns the set of signals that are pending for the calling process in the sigset_t structure pointed to by **set**.
```c
#include <signal.h> 
int sigpending(sigset_t *set);
// Returns 0 on success, or –1 on error
```
- **If we change the disposition of a pending signal, then, when the signal is later unblocked, it is handled according to its new disposition**
# 9 Changing Signal Dispositions: **sigaction**()

```C
#include <signal.h>

int sigaction(int sig, const struct sigaction *act, struct sigaction *oldact); // Returns 0 on success, or –1 on error

struct sigaction {
void (*sa_handler)(int); /* Address of handler */
sigset_t sa_mask; /* Signals blocked during handler invocation */
int sa_flags; /* Flags controlling handler invocation */
void (*sa_restorer)(void); /* Not for application use */ 
};

struct sigaction { 
union { 
	void (*sa_handler)(int);
	void (*sa_sigaction)(int, siginfo_t *, void *); 
	} __sigaction_handler; 
sigset_t sa_mask;
int void };
```

# 10 Waiting for a Signal: **pause**()
Calling **pause**() suspends execution of the process until the call is interrupted by a signal handler (or until an unhandled signal terminates the process).
```C
#include <unistd.h>

int pause(void);

// Always returns –1 with errno set to EINTR
```


# 11 Designing Signal Handlers
**Two common designs for signal handlers are the following**
- The signal handler sets a global flag and exits. The main program periodically checks this flag and, if it is set, takes appropriate action.
- The signal handler performs some type of cleanup and then either terminates the process or uses a nonlocal goto to unwind the stack and return control to a predetermined location in the main program.
## 11. 1 Signals Are Not Queued (Revisited)
## 11.2 Reentrant and Async-Signal-Safe Functions
- when writing signal handlers, we have **two choices**:
	- Ensure that the code of the signal handler itself is reentrant and that it calls only async-signal-safe functions.
	- Block delivery of signals while executing code in the main program that calls unsafe functions or works with global data structures also updated by the sig nal handler
- Use of **errno** inside signal handlers :
```c
void handler(int sig) {
int savedErrno; 
savedErrno = errno; /* Now we can execute a function that might modify errno */ errno = savedErrno; 
}
```
## 11. 3 Global Variables and the **sig_atomic_t** Data Type
- The C language standards and SUSv3 specify an integer data type, sig_atomic_t, for which reads and writes are guaranteed to be atomic. Thus, a global flag variable that is shared between the main program and a signal handler should be declared as follows:
```c
volatile sig_atomic_t flag;
```

#  12 Other Methods of Terminating a Signal Handler
- Use \_exit() to terminate the process.Beforehand, the handler may carry out some cleanup actions.
- Use kill() or raise() to send a signal that kills the process
- Perform a nonlocal goto from the signal handler ( **sigsetjmp**, **siglongjmp**)
- Use the abort() function to terminate the process with a core dump.
# 13 Handling a Signal on an Alternate Stack: **sigaltstack**()
- The **sigaltstack**() system call both establishes an alternate signal stack and returns information about any alternate signal stack that is already established
```c
#include<signal.h>
int sigaltstack(const stack_t *sigstack, stack_t *old_sigstack); 
// Returns 0 on success, or –1 on error
typedef struct { 
void *ss_sp; /* Starting address of alternate stack */
size_t ss_size; } stack_t;   /* Size of alternate stack */
int ss_flags; /* Size of alternate stack*/
size_t ss_size; } stack_t;   
```
The **sigstack** argument points to a structure specifying the location and attributes of the new alternate signal stack. The **old_sigstack** argument points to a structure used to return information about the previously established alternate signal stack (if there was one).


# 14 The **SA_SIGINFO** Flag

```c
typedef struct {
int si_signo; /* Signal number */
int si_code; /* Signal code */
int si_trapno; /* Trap number for hardware-generated signal
(unused on most architectures) */
union sigval si_value; /* Accompanying data from sigqueue() */
pid_t si_pid; /* Process ID of sending process */
uid_t si_uid; /* Real user ID of sender */
int si_errno; /* Error number (generally unused) */
void *si_addr; /* Address that generated signal
(hardware-generated signals only) */
int si_overrun; /* Overrun count (Linux 2.6, POSIX timers) */
int si_timerid; /* (Kernel-internal) Timer ID
(Linux 2.6, POSIX timers) */
long si_band; /* Band event (SIGPOLL/SIGIO) */
int si_fd; /* File descriptor (SIGPOLL/SIGIO) */
int si_status; /* Exit status or signal (SIGCHLD) */
clock_t si_utime; /* User CPU time (SIGCHLD) */
clock_t si_stime; /* System CPU time (SIGCHLD) */
} siginfo_t;
```

# 15 Interruption and Restarting of System Calls
