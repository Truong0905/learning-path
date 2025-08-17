# 1 Process ID and Parent Process ID
- The **getpid**() system call returns the process ID of the calling process
```c
#include <unistd.h>
pid_t getpid(void); 
// Always successfully returns process ID of caller
pid_t getppid(void); 
// Always successfully returns process ID of parent of caller
```

# 2 Virtual Memory Management
![[Pasted image 20250608142442.png]]

# 3 Performing a Nonlocal Goto: **setjmp**() and **longjmp**()
```c
#include <setjmp.h>
int setjmp(jmp_buf env);
// Returns 0 on initial call, nonzero on return via longjmp() 
void longjmp(jmp_buf env, int val);
```