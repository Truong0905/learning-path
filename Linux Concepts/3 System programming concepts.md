# 1 System Calls

- Some general points:
	- A system call changes the processor state from user mode to kernel mode, so that the CPU can access protected kernel memory.
	- The set of system calls is fixed. Each system call is identified by a unique number. (This numbering scheme is not normally visible to programs, which identify system calls by name.)
	- Each system call may have a set of arguments that specify information to be transferred from user space (i.e., the process’s virtual address space) to kernel space and vice versa
	![[Pasted image 20250608085826.png]]

# 2 Handling Errors from System Calls and Library Functions
- When a system call fails, it sets the global integer variable **errno** to a positive value that identifies the specific error. (Including the header file provides a dec laration of *errno*)
# 3 Common Functions and Header Files
## 3.1  common header file
![[Pasted image 20250608090436.png]]
## 3.2 Portability Issues
- Feature Test Macros
	  -  \_POSIX_SOURCE
		If defined (with any value), expose definitions conforming to POSIX.1-1990 and ISO C (1990). This macro is superseded by \_POSIX_C_SOURCE.
	-  \_POSIX_C_SOURCE
