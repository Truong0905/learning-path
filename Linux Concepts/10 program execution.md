
# 1 Executing a New Program: **execve**()

- ["] The **execve**() system call loads a new program into a process’s memory. During this operation, the old program is discarded, and the process’s stack, data, and heap are replaced by those of the new program. After executing various C library run-time startup code and program initialization code (e.g., C++ static constructors or C functions declared with the gcc constructor attribute described in Section 42.4), the new program commences execution at its main() function.
- ["]  The execve() system call loads a new program into a process’s memory. During this operation, the old program is discarded, and the process’s stack, data, and heap are replaced by those of the new program. After executing various C library run-time startup code and program initialization code (e.g., C++ static constructors or C functions declared with the gcc constructor attribute described in Section 42.4), the new program commences execution at its main() function.
- Various library functions, all with names beginning with **exec**, are layered on top of the **execve**() system call
```c
#include <unistd.h>

int execve(const char *pathname, char *const argv[], char *const envp[]);
// Never returns on success; returns –1 on error
```
- After an **execve**(), the process ID of the process remains the same, because the same process continues to exist. A few other process attributes also remain unchanged

# 2 The **exec**() Library Functions

The library functions described in this section provide alternative APIs for performing an **exec**(). All of these functions are layered on top of **execve**().

```c
#include <unistd.h>

int execle(const char *pathname, const char *arg, ... /* , (char *) NULL, char *const envp[] */ );
int execlp(const char *filename, const char *arg, ... /* , (char *) NULL */);
int execvp(const char *filename, char *const argv[]);
int execv(const char *pathname, char *const argv[]);
int execl(const char *pathname, const char *arg, .../* , (char *) NULL */);

// None of the above returns on success; all return –1 on error
```
- The **filename** is sought in the list of directories specified in the **PATH** environment variable. The PATH variable is not used if the filename contains a slash (/), in which case it is treated as a relative or absolute pathname.
![[Pasted image 20250705152600.png]]
- `l`: Arguments as a List (`arg1, arg2, ..., NULL`)- 
- `v`: Arguments as a array(`argv[]`)
- `p`: The **PATH** Environment Variable is used
- `e`: Passing the Caller’s Environment to the New Program (`envp[]`)
```c
execlp(argv[1], argv[1], "hello world", (char *) NULL); /* l and p/
```

```c
char *envVec[] = { "GREET=salut", "BYE=adieu", NULL };
execle(argv[1], filename, "hello world", (char *) NULL, envVec); /* l and e */
```

```c
char *args[] = {"ls", "-l", NULL};
execvp("ls", args); /* v and p */
```

 * Executing a File Referred to by a Descriptor: **fexecve**()

 - ["] Since version 2.3.2, glibc provides fexecve(), which behaves just like execve(), but specifies the file to be execed via the open file descriptor fd, rather than as a pathname
```c
#define _GNU_SOURCE #include <unistd.h>
int fexecve(int fd, char *const argv[], char *const envp[]);
// Doesn’t return on success; returns –1 on error
```



# 3 Interpreter Scripts
# 4 File Descriptors and **exec**()
# 5 Signals and **exec**()
- During an **exec**(), the text of the existing process is discarded. This text may include signal handlers established by the calling program. Because the handlers disappear, the kernel resets the dispositions of all handled signals to **SIG_DFL**. The dispositions of all other signals (i.e., those with dispositions of **SIG_IGN** or **SIG_DFL**) are left unchanged by an **exec**()

# 6 Executing a Shell Command: **system**()
- The **system**() function allows the calling program to execute an arbitrary shell command.

```c
#include <stdlib.h> 
int system(const char *command);
// See main text for a description of return value
```
- the **system**() function creates a child process that invokes a shell to execute command. Here is an example of a call to system():
```c
system("ls | wc");
```
- Avoid using system() in set-user-ID and set-group-ID programs

# 7 Effect of **exec**() and **fork**() on Process Attributes
![[Pasted image 20250706164306.png]]

![[Pasted image 20250706164329.png]]

....

