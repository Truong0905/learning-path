# 1 Core Dump Files
- Certain signals cause a process to create a core dump and terminate(Table 20-1, page 396)
- A core dump is a file containing a memory image of the process at the time it terminated.
- How to use core dump file
```bash
$ gdb myapp core
```

![[Pasted image 20250614095545.png]]

```bash
echo "/var/dumps/core_%e_%p_%t" > /proc/sys/kernel/core_pattern
```

# 2 Special Cases for Delivery, Disposition, and Handling
## 2.1 SIGKILL and SIGSTOP
- Both **signal**() and **sigaction**() return an error on attempts to change the disposition of these signals.
## 2.2 SIGCONT and stop signals
- the SIGCONT signal is used to continue a process previously stopped by one of the stop signals (SIGSTOP, SIGTSTP, SIGTTIN, and SIGTTOU)
- If a process is currently stopped, the arrival of a SIGCONT signal always causes the process to resume, even if the process is currently blocking or ignoring SIGCONT
- If the stopped process was blocking SIGCONT, and had established a handler for SIGCONT, then, after the process is resumed, **the handler is invoked only when SIGCONT is later unblocked**
- **Don’t change the disposition of ignored terminal-generated signals**
# 3 Interruptible and Uninterruptible Process Sleep States
- At various times, the kernel may put a process to sleep, and two sleep states are distinguished:
	- TASK_UNINTERRUPTIBLE: The process is waiting on certain special classes of event, such as the completion of a disk I/O
	- TASK_INTERRUPTIBLE : For example, it is waiting for terminal input, for data to be written to a currently empty pipe, or for the value of a System V semaphore to be increased
	- TASK_KILLABLE: This state is like TASK_UNINTERRUPTIBLE, but wakes the process if a fatal signal (i.e., one that would kill the process) is received

```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/sched.h>
#include <linux/wait.h>

static DECLARE_WAIT_QUEUE_HEAD(my_wait_queue);

int my_function(void) {
    // Set process state to TASK_INTERRUPTIBLE (can be interrupted by signals)
    set_current_state(TASK_INTERRUPTIBLE);
    
    // Wait for an event (this will block the process)
    schedule();
    
    printk(KERN_INFO "Process has been awakened!\n");
    return 0;
}
```

```c
int my_function(void) {
    // Set process state to TASK_UNINTERRUPTIBLE (cannot be interrupted by signals)
    set_current_state(TASK_UNINTERRUPTIBLE);
    
    // Wait for an event (process will NOT wake up due to signals)
    schedule();
    
    printk(KERN_INFO "Process has been awakened!\n");
    return 0;
}
```

# 4 Hardware-Generated Signals
- The correct way to deal with hardware-generated signals is either to accept their default action (process termination) or to write handlers that don’t perform a normal return. Other than returning normally, a handler can complete execution by calling \_exit() to terminate the process or by calling siglongjmp() (Section 21.2.1) to ensure that control passes to some point in the program other than the instruction that generated the signal.
# 5 Synchronous and Asynchronous Signal Generation
- Synchronous
	- The hardware-generated signals (SIGBUS, SIGFPE, SIGILL, SIGSEGV, and SIGEMT) described in Section 22.4 are generated as a consequence of executing a specific machine-language instruction that results in a hardware exception.
	- A process can use **raise**(), **kill**(), or **killpg**() to send a signal to itself
- ["] Note that synchronicity is an attribute of how a signal is generated, rather than of the signal itself. All signals may be generated synchronously (e.g., when a process sends itself a signal using kill()) or asynchronously (e.g., when the signal is sent by another process using kill())

# 6 Timing and Order of Signal Delivery
- the signal is delivered at one of the following times:
	- when the process is rescheduled after it earlier timed out (i.e., at the start of a time slice);
	- at completion of a system call (delivery of the signal may cause a blocking system call to complete prematurely).
# 7 Realtime Signals
- They have the following advantages over standard signals:
	- Realtime signals provide an increased range of signals that can be used for application-defined purposes. Only two standard signals are freely available for application-defined purposes: SIGUSR1 and SIGUSR2
	- Realtime signals are queued. If multiple instances of a realtime signal are sent  to a process, then the signal is delivered multiple times. By contrast, if we send  further instances of a standard signal that is already pending for a process, that signal is delivered only once
	- When sending a realtime signal, it is possible to specify data (an integer or pointer value) that accompanies the signal. The signal handler in the receiving process can retrieve this data
	- The order of delivery of different realtime signals is guaranteed. If multiple different realtime signals are pending, then the lowest-numbered signal is delivered first. In other words, signals are prioritized, with lower-numbered signals having higher priority. When multiple signals of the same type are queued, they are delivered—along with their accompanying data—in the order in which they were sent
## 7.1 Sending Realtime Signals
- The **sigqueue**() system call sends the realtime signal specified by sig to the process specified by pid.
```c
#define _POSIX_C_SOURCE 199309
#include <signal.h>
int sigqueue(pid_t pid, int sig, const union sigval value);
// Returns 0 on success, or –1 on error

union sigval {
int sival_int; /* Integer value for accompanying data */
void *sival_ptr; /* Pointer value for accompanying data */
};

```
- Unlike **kill**(), we can’t use **sigqueue**() to send a signal to an entire process group by specifying a negative value in **pid**.
- A call to **sigqueue**() may fail if the limit on the number of queued signals has been reached. In this case, errno is set to **EAGAIN**, indicating that we need to send the signal again (at some later time when some of the currently queued signals have been delivered)

## 7.2 Handling Realtime Signals
- Same like implementing handler of  sigaction()
# 8 Waiting for a Signal Using a Mask: **sigsuspend**()

- The **sigsuspend**() system call **replaces** the **process signal mask** **by** the signal set pointed to by **mask**, and **then suspends execution of the process until a signal is caught and its handler returns**. **Once the handler returns**, sigsuspend() **restores** the **process signal mask** to **the value it had prior to the call**
```c
#include <signal.h
int sigsuspend(const sigset_t *mask);

//(Normally) returns –1 with errno set to EINTR
```

This function is equivalent to automically performing these operations:
```c
sigprocmask(SIG_SETMASK, &mask, &prevMask); /* Assign new mask */
pause();
sigprocmask(SIG_SETMASK, &prevMask, NULL); /* Restore old mask */
```
With **mask** is mask in **sigsuspend**
# 9 Synchronously Waiting for a Signal
- The **sigwaitinfo**() system call suspends execution of the process until one of the signals in the signal set pointed to by **set** is delivered.
```c
#define _POSIX_C_SOURCE 199309
#include <signal.h>
int sigwaitinfo(const sigset_t *set, siginfo_t *info);
// Returns number of delivered signal on success, or –1 on error
```

-  If one of the signals in **set** is already pending at the time of the call, **sigwaitinfo**() returns immediately. The delivered signal is removed from the process’s list of pending signals, and the signal number is returned as the function result.
- If the **info** argument **is not** NULL, then it points to a **siginfo_t** structure that is initialized to contain the **same** information provided to a signal handler taking a **siginfo_t** argument


- The **sigtimedwait**() system call is a **variation** on **sigwaitinfo**(). The only difference is that sigtimedwait() allows us to specify a **time limit for waiting**.
```c
#define _POSIX_C_SOURCE 199309
#include <signal.h>

int sigtimedwait(const sigset_t *set, siginfo_t *info, const struct timespec *timeout);
// Returns number of delivered signal on success, or –1 on error or timeout (EAGAIN)

struct timespec {
time_t tv_sec; /* Seconds ('time_t' is an integer type) */
long tv_nsec; /* Nanoseconds */
};
```
- Specifying both fields of the structure as 0 causes an immediate timeout—that is, a poll to **check if any of the specified set of signals is pending**
- If the **timeout** argument is specified as **NULL**, then **sigtimedwait**() is exactly **equivalent** to **sigwaitinfo**().


# 10 Fetching Signals via a File Descriptor
```c
#include <sys/signalfd.h>
int signalfd(int fd, const sigset_t *mask, int flags);
// Returns file descriptor on success, or –1 on error
```
- The buffer given to read() must be large enough to hold at least one signalfd_siginfo structure, defined as follows in <sys/signalfd.h>:
```c
struct signalfd_siginfo {

uint32_t ssi_signo; /* Signal number */

int32_t ssi_errno; /* Error number (generally unused) */

int32_t ssi_code; /* Signal code */

uint32_t ssi_pid; /* Process ID of sending process */

uint32_t ssi_uid; /* Real user ID of sender */

int32_t ssi_fd; /* File descriptor (SIGPOLL/SIGIO) */

uint32_t ssi_tid; /* Kernel timer ID (POSIX timers) */

uint32_t ssi_band; /* Band event (SIGPOLL/SIGIO) */

uint32_t ssi_tid; /* (Kernel-internal) timer ID (POSIX timers) */

uint32_t ssi_overrun; /* Overrun count (POSIX timers) */ uint32_t ssi_trapno; /* Trap number */

int32_t ssi_status; /* Exit status or signal (SIGCHLD) */

int32_t ssi_int; /* Integer sent by sigqueue() */

uint64_t ssi_ptr; /* Pointer sent by sigqueue() */

uint64_t ssi_utime; /* User CPU time (SIGCHLD) */

uint64_t ssi_stime; /* System CPU time (SIGCHLD) */

uint64_t ssi_addr; /* Address that generated signal
					(hardware-generated signals only) */ 
};
```
