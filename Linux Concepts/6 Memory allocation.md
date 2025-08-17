# 1 Adjusting the Program Break: **brk**() and **sbrk**()
```c
#include <unistd.h> 
int brk(void *end_data_segment); 
// Returns 0 on success, or –1 on error 
void *sbrk(intptr_t increment); 
// Returns previous program break on success, or (void *) –1 on error
```
The **brk**() system call sets the program break to the location specified by **end_data_segment**. Since virtual memory is allocated in units of pages, **end_data_segment** is effectively rounded up to the next page boundary.