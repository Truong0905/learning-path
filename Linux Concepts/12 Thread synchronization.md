

# 1 Protecting Accesses to Shared Variables: Mutexes

- A mutex has two states: **locked** and **unlocked**
- When a thread locks a mutex, it becomes the owner of that mutex. Only the mutex owner can unlock the mutex.
- the terms **acquire** and **release** are sometimes used synonymously for lock and unlock
## 1.1 Statically Allocated Mutexes
- A mutex can either be allocated as a static variable or be created dynamically at run time
- A mutex is a variable of the type **pthread_mutex_t**
- Before it can be used, a mutex must always be initialized.
- For a statically allocated mutex, we can do this by assigning it the value **PTHREAD_MUTEX_INITIALIZER**, as in the following example:
```c
pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;
```
## 1.2 Locking and Unlocking a Mutex
- After initialization, a mutex is unlocked.
- If the mutex is currently locked by another thread, then **pthread_mutex_lock**() blocks until the mutex is unlocked, at which point it locks the mutex and returns
```c
#include <pthread.h>

int pthread_mutex_lock(pthread_mutex_t *mutex); 
int pthread_mutex_unlock(pthread_mutex_t *mutex);
// Both return 0 on success, or a positive error number on error
```
- **pthread_mutex_trylock**() and **pthread_mutex_timedlock**()
	- The **pthread_mutex_trylock**() function is the same as **pthread_mutex_lock**(), except that if the mutex is currently locked, **pthread_mutex_trylock**() fails, returning the error **EBUSY**
	- The **pthread_mutex_timedlock**() function is the same as **pthread_mutex_lock**(), except that the caller can specify an additional argument, **abstime**, that places a limit on the time that the thread will sleep while waiting to acquire the mutex. Can return **ETIMEDOUT** after **abstime**
## 1.3 Performance of Mutexes
## 1.4 Mutex Deadlocks
- When more than one thread is locking the same set of mutexes, deadlock situations can arise
- The simplest way to avoid such deadlocks is to define a mutex hierarchy. When threads can lock the same set of mutexes, they should always lock them in the same order
- An alternative strategy that is less frequently used is “try, and then back off.” In this strategy, a thread locks the first mutex using pthread_mutex_lock(), and then locks the remaining mutexes using pthread_mutex_trylock(). If any of the pthread_mutex_trylock() calls fails (with EBUSY), then the thread releases all mutexes, and then tries again, perhaps after a delay interval
## 1.5 Dynamically Initializing a Mutex
- The static initializer value PTHREAD_MUTEX_INITIALIZER can be used only for initializing a statically allocated mutex with default attributes. In all other cases, we must dynamically initialize the mutex using pthread_mutex_init()
```c
#include <pthread.h>

int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);
// Returns 0 on success, or a positive error number on error
```
- The **mutex** argument identifies the mutex to be initialized.
- The **attr** argument is a pointer to a **pthread_mutexattr_t** object that has previously been initialized to define the attributes for the mutex. If **attr** is specified as NULL, then the mutex is assigned various default attributes.
- When an automatically or dynamically allocated mutex is no longer required, it should be destroyed using **pthread_mutex_destroy**().((It is not necessary to call pthread_mutex_destroy() on a mutex that was statically initialized using PTHREAD_MUTEX_INITIALIZER.)
```c
#include <pthread.h> 
int pthread_mutex_destroy(pthread_mutex_t *mutex);

// Returns 0 on success, or a positive error number on error
```
- A mutex that has been destroyed with pthread_mutex_destroy() can subsequently be reinitialized by pthread_mutex_init()

# 2 Signaling Changes of State: Condition Variables
## 2.1 Statically Allocated Condition Variables
- As with mutexes, condition variables can be allocated statically or dynamically.
- A condition variable has the type **pthread_cond_t**
- For a statically allocated condition variable, this is done by assigning it the value PTHREAD_COND_INITIALIZER, as in the following example:
```c
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
```
## 2.2 Signaling and Waiting on Condition Variables

- The principal condition variable operations are **signal** and **wait**. The signal operation is a notification to one or more waiting threads that a shared variable’s state has changed. The wait operation is the means of blocking until such a notification is received.
```c
#include <pthread.h>

int pthread_cond_signal(pthread_cond_t *cond); 
int pthread_cond_broadcast(pthread_cond_t *cond); 
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);

// All return 0 on success, or a positive error number on error
```
- The **pthread_cond_signal**() and **pthread_cond_broadcast**() functions both signal the condition variable specified by **cond**. The **pthread_cond_wait**() function blocks a thread until the condition variable **cond** is signaled
- With **pthread_cond_signal**(), we are simply guaranteed that at least one of the blocked threads is woken up; with **pthread_cond_broadcast**(), all blocked threads are woken up
- The **pthread_cond_timedwait**() function is the same as **pthread_cond_wait**(), except that the **abstime** argument specifies an upper limit on the time that the thread will sleep while waiting for the condition variable to be signaled
```c
#include <pthread.h>

int pthread_cond_timedwait(pthread_cond_t *cond, pthread_mutex_t *mutex, const struct timespec *abstime);

// Returns 0 on success, or a positive error number on error
```

- **pthread_cond_wait**(), which performs the following steps:
	- unlock the mutex specified by **mutex**;
	- block **the calling thread**(this) until another thread signals the condition variable **cond**;
	- relock mutex. ( after all)
- **pthread_cond_wait** :  releasing the mutex and blocking on the condition variable are performed atomically
## 2.3 Dynamically Allocated Condition Variables
- The pthread_cond_init() function is used to dynamically initialize a condition variable

```c
#include <pthread.h>
int pthread_cond_init(pthread_cond_t *cond, const pthread_condattr_t *attr); 
// Returns 0 on success, or a positive error number on error
```
- The **cond** argument identifies the condition variable to be initialized
```c
#include <pthread.h>
int pthread_cond_destroy(pthread_cond_t *cond);
Returns 0 on success, or a positive error number on error
```


# 3 Thread Safety (and Reentrancy Revisited)

- A function is said to be thread-safe if it can safely be invoked by multiple threads at the same time
![[Pasted image 20250714211110.png]]

# 4 One-Time Initialization

- Sometimes, a threaded application needs to ensure that some initialization action occurs just once, regardless of how many threads are created

```c
#include <pthread.h>

int pthread_once(pthread_once_t *once_control, void (*init)(void));

// Returns 0 on success, or a positive error number on error

```
 - The **once_control** argument is a pointer to a variable that must be statically initialized with the value PTHREAD_ONCE_INIT:
```c
pthread_once_t once_var = PTHREAD_ONCE_INIT;
```
- The first call to **pthread_once**() that specifies a pointer to a particular **pthread_once_t** variable modifies the value of the variable pointed to by **once_control** so that subsequent calls to **pthread_once**() don’t invoke **init**
- One common use of **pthread_once**() is in conjunction with thread-specific data

# 5 Thread-Specific Data

- Thread-specific data is a technique for making an existing function thread-safe without changing its interface
- A function that uses thread-specific data may be slightly less efficient than a reentrant function, but allows us to leave the programs that call the function unchanged

## 5.1 Thread-Specific Data from the Library Function’s Perspective
- we need to consider things from the point of view of a library function that uses thread-specific data:
	- The function must allocate a separate block of storage for each thread that calls the function.
	- On each subsequent call from the same thread, the function needs to be able to obtain the address of the storage block that was allocated the first time this thread called the function. The Pthreads API provides functions to handle this task
	- Different (i.e., independent) functions may each need thread-specific data. Each function needs a method of identifying its thread-specific data (a key), as distinct from the thread-specific data used by other functions
	- The function has no direct control over what happens when the thread terminates
## 5.2 Overview of the Thread-Specific Data API

- The function creates a key, which is the means of differentiating the thread-specific data item used by this function from the thread-specific data items used by other functions: **pthread_key_create**() <   Creating a key needs to be done only once, when the first thread calls the function => Need  **pthread_once**() >
- The call to pthread_key_create() serves a second purpose: it allows the caller to specify the address of the programmer-defined destructor function that is used  to deallocate each of the storage blocks allocated for this key when this thread terminates.
- The function allocates a thread-specific data block for each thread from which it is called. This is done using **malloc**() (or a similar function).
- In order to save a pointer to the storage allocated in the previous step, the function employs two Pthreads functions: **pthread_setspecific**() and **pthread_getspecific**().


## 5.3 Details of the Thread-Specific Data API

- Calling **pthread_key_create**() creates a new thread-specific data key that is returned to the caller in the buffer pointed to by key.

```c
#include <pthread.h>

int pthread_key_create(pthread_key_t *key, void (*destructor)(void *));

// Returns 0 on success, or a positive error number on error
```

- Because the returned key is used by all threads in the process, **key** should point to a global variable
- The **destructor** argument points to a programmer-defined function of the following form:
```c
void dest(void *value) {
/* Release storage pointed to by 'value' */ 
}
```

- The **pthread_setspecific**() function requests the Pthreads API to save a copy of **value** in a data structure that associates it with the calling thread and with **key**, a key returned by a previous call to **pthread_key_create**()
- The **pthread_getspecific**() function performs the converse operation

```c
#include <pthread.h>

int pthread_setspecific(pthread_key_t key, const void *value);
// Returns 0 on success, or a positive error number on error
void *pthread_getspecific(pthread_key_t key);
// Returns pointer, or NULL if no thread-specific data isassociated with key
```

- The **value** argument given to **pthread_setspecific**() is normally a pointer to a block of memory that has previously been allocated by the caller. This pointer will be passed as the argument for the destructor function for this **key** when the thread terminates

## 5.4 Employing the Thread-Specific Data API
- For example, In a function (my_func ) we use **pthread_setspecific**() to save pointer. And then, We can use  **pthread_getspecific**() to get this poiter. The pointer here  can be  provided by **malloc**(). Therefore, we can use this pointer in outside of this function
# 6 Thread-Local Storage
- The main advantage of thread-local storage is that it is much simpler to use than thread-specific data. To create a thread-local variable, we simply include the **__thread** specifier in the declaration of a global or static variable :
```c
static __thread char buf[MAX_ERROR_LEN]
```
# 7 Canceling a Thread

- The **pthread_cancel**() function sends a cancellation request to the specified **thread**

```c
#include <pthread.h> 
int pthread_cancel(pthread_t thread);
// Returns 0 on success, or a positive error number on error
```

# 8 Cancellation State and Type

- The **pthread_setcancelstate**() and **pthread_setcanceltype**() functions set flags that allow a thread to control how it responds to a cancellation request:
```c
#include <pthread.h>
int pthread_setcancelstate(int state, int *oldstate); 
int pthread_setcanceltype(int type, int *oldtype);
// Both return 0 on success, or a positive error number on error
```
- **state** :
	- PTHREAD_CANCEL_DISABLE  : The thread is not cancelable
	- PTHREAD_CANCEL_ENABLE: The thread is cancelable
- The thread’s previous cancelability state is returned in the location pointed to by **oldstate**
- If a thread is cancelable (PTHREAD_CANCEL_ENABLE). And now **type** in **pthread_setcanceltype**():
	- **PTHREAD_CANCEL_ASYNCHRONOUS** : The thread may be canceled at any time
	- **PTHREAD_CANCEL_DEFERRED** : The cancellation remains pending until a cancellation point is reached
- The thread’s previous cancelability type is returned in the location pointed to by **oldtype**

- ["] When a thread calls **fork**(), the child **inherits** the calling thread’s cancelability type and state. When a thread calls **exec**(), the cancelability type and state of the main thread of the new program are **reset** to **PTHREAD_CANCEL_ENABLE** and **PTHREAD_CANCEL_DEFERRED**, respectively


# 9 Cancellation Points
- When cancelability is enabled and deferred, a cancellation request is acted upon only when a thread next reaches a **cancellation point**

![[Pasted image 20250715220019.png]]

# 10  Testing for Thread Cancellation

The purpose of **pthread_testcancel**() is simply to be a cancellation point
```c
#include <pthread.h>
void pthread_testcancel(void);
```

# 11 Cleanup Handlers
- ["] If a thread with a pending cancellation were simply terminated when it reached a cancellation point, then shared variables and Pthreads objects (e.g., mutexes) might be left in an inconsistent state, perhaps causing the remaining threads in the process to produce incorrect results, deadlock, or crash.
- To get around this problem, a thread can establish one or more **cleanup handlers**
```c
#include <pthread.h>
void pthread_cleanup_push(void (*routine)(void*), void *arg);
void pthread_cleanup_pop(int execute);
```

- The **pthread_cleanup_push**() function adds the function whose address is specified in **routine** to the top of the calling thread’s stack of cleanup handlers.
- The **arg** value given to **pthread_cleanup_push**() is passed as the argument of the cleanup handler when it is invoked.
- ["] Typically, a cleanup action is needed only if a thread is canceled during the execution of a particular section of code. If the thread reaches the end of that section without being canceled, then the cleanup action is no longer required.
-  Each call to **pthread_cleanup_push**() has an accompanying call to **pthread_cleanup_pop**(). This function removes the topmost function from the stack of cleanup handlers .
- If the **execute** argument is nonzero, the handler is also executed. This is convenient if we want to perform the cleanup action even if the thread was not canceled
- **pthread_cleanup_push**() must be paired with exactly one corresponding **pthread_cleanup_pop**() in the same lexical block. For example, it is not correct to write code such as the following:
```c
pthread_cleanup_push(func, arg);
...
if (cond) {
	pthread_cleanup_pop(0);
}
```

# 12 Asynchronous Cancelability
- As a general principle, an asynchronously cancelable thread can’t allocate any resources or acquire any mutexes, semaphores, or locks

# 13 Thread Stacks
- Each thread has its own stack whose size is fixed when the thread is created.
# 14  Threads and Signals
## 14.1 How the UNIX Signal Model Maps to Threads

- If any unhandled signal whose default action is  stop or terminate is delivered to any thread in a process, then all of the threads in the process are stopped or terminated
- Signal dispositions are process-wide; all threads in a process share the same disposition for each signal
- A signal may be directed to either the process as a whole or to a specific thread.
.... (page 863)
