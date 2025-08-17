# 1 Overview
- A single process can contain multiple threads. All of these threads are independently executing the same program, and they all share the same global memory, including the initialized data, uninitialized data, and heap segments.
- Pthreads API will create **kernel-level threads**
![[Pasted image 20250707213329.png]]

- We have some reasons to choose thread instead of process: 
	- It is difficult to share information between processes. Since the parent and child don’t share memory. We must use some form of interprocess communication in order to exchange information between processes.
	-  Process creation with **fork**() is relatively expensive.  The need to duplicate various process attributes such as page tables and file descriptor tables means that a **fork**() call is still time-consuming
- Threads address both of these problems: 
	- Sharing information between threads is easy and fast. It is just a matter of copying data into shared (global or heap) variables
	- Thread creation is faster than process creation—typically, ten times faster or better( On Linux, threads are implemented using the **clone**() system call)
- Besides global memory, threads also share a number of other attributes (i.e., these attributes are global to a process, rather than specific to a thread) And Among the attributes that are distinct for each thread

# 2 Background Details of the Pthreads API

- **Pthreads data types**:
![[Pasted image 20250707214630.png]]

- Threads and **errno** :
	- each thread has its own **errno** value
	- On Linux, a thread-specific **errno** is achieved in a similar manner to most other UNIX implementations: **errno** is defined as a macro that expands into a function call returning a modifiable lvalue that is distinct for each thread
- Return value from Pthreads functions : All Pthreads functions return 0 on success or a positive value on failure
	- Because each reference to **errno** in a threaded program carries the overhead of a function call ( **errno** is actually a macro that call a function to return errro value) .Instead, we use an intermediate variable and employ our errExitEN() diagnostic function
```c
pthread_t *thread; int s;
s = pthread_create(&thread, NULL, func, &arg);
if (s != 0)
	errExitEN(s, "pthread_create");
```

- Compiling Pthreads programs: On Linux, programs that use the Pthreads API must be compiled with the **cc –pthread** option: 
	- The \_REENTRANT preprocessor macro is defined. This causes the declarations of a few reentrant functions to be exposed
	- The program is linked with the libpthread library (the equivalent of –lpthread)

# 3 Thread Creation

- **When a program is started, the resulting process consists of a single thread, called the initial or main thread**
- In this section, we look at how to create additional threads:
```c
#include <pthread.h>
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start)(void *), void *arg);
// Returns 0 on success, or a positive error number on error
```

- The new thread commences execution by calling the function identified by **start** with the argument **arg** (i.e., **start**(**arg**))
- If we need to pass multiple arguments to **start**, then **arg** can be specified as a pointer to a structure containing the arguments as separate fields
- The return value of **start** is likewise of type **void \***, and it can be employed in the same way as the **arg** argument. We can obtain the return value by calling function **pthread_join** 

- The **thread** argument points to a buffer of type **pthread_t** into which the unique identifier for this thread is copied before **pthread_create**() returns
- The **attr** argument is a pointer to a **pthread_attr_t** object that specifies various attributes for the new thread. If **attr** is specified as NULL, then the thread is created with various default attributes, and this is what we’ll do in most of the example programs in this book.

# 4 Thread Termination

- The execution of a thread terminates in one of the following ways:
	- The thread’s start function performs a return specifying a return value for the thread
	- The thread calls **pthread_exit**()
	- The thread is canceled using **pthread_cancel**()
	- Any of the threads calls **exit**(), or the main thread performs a return (in the main() function), which causes all threads in the process to terminate immediately.

- The **pthread_exit**() function terminates the calling thread, and specifies a return value that can be obtained in another thread by calling **pthread_join**():
```c
include <pthread.h>
void pthread_exit(void *retval);
```

- ["] Calling **pthread_exit**() is equivalent to performing a return in the thread’s start function, with the difference that **pthread_exit**() can be called from any function that has been called by the thread’s start function.

- The **retval** argument specifies the return value for the thread. The value pointed to by retval should not be located on the thread’s stack, since the contents of that stack become undefined on thread termination. The same statement applies to the value given to a return statement in the thread’s start function
- If the main thread calls **pthread_exit**() instead of calling **exit**() or performing a return, then the other threads continue to execute
# 5 Thread IDs
- Each thread within a process is uniquely identified by a thread ID

- This ID is returned to the caller of **pthread_create**(), and a thread can obtain its own ID using **pthread_self**()

```c
include <pthread.h>
pthread_t pthread_self(void);
// Returns the thread ID of the calling thread
```

- The **pthread_equal**() function allows us check whether two thread IDs are the same.
```c
include <pthread.h>
int pthread_equal(pthread_t t1, pthread_t t2);
// Returns nonzero value if t1 and t2 are equal, otherwise 0
```


# 6 Joining with a Terminated Thread

- The **pthread_join**() function waits for the thread identified by thread to terminate. (If that thread has already terminated, **pthread_join**() returns immediately.) This operation is termed **joining**
```c
include <pthread.h>
int pthread_join(pthread_t thread, void **retval);
// Returns 0 on success, or a positive error number on error
```

- If **retval** is a non-NULL pointer, then it receives a copy of the terminated thread’s return value—that is, the value that was specified when the thread performed a **return** or called **pthread_exit**().

# 7 Detaching a Thread
- Sometimes, we don’t care about the thread’s return status; we simply want the system to automatically clean up and remove the thread when it terminates. In this case, we can mark the thread as detached, by making a call to **pthread_detach**() specifying the thread’s identifier in thread
```c
#include <pthread.h> 
int pthread_detach(pthread_t thread);
// Returns 0 on success, or a positive error number on error
```
- As an example of the use of pthread_detach(), a thread can detach itself using the following call: **pthread_detach(pthread_self());**
- Once a thread has been detached, it is no longer possible to use **pthread_join**() to obtain its return status, and the thread can’t be made joinable again


# 8 Thread Attributes
-  these attributes include information such as the location and size of the thread’s stack, the thread’s scheduling policy and priority,...
![[Pasted image 20250708223408.png]]

# 9 Threads Versus Processes
