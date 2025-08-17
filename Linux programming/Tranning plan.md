
###  0 
- **Linux System Programming**: Process/thread, socket, IPC, systemd.
    
- **Kernel & Driver Development**: Viết kernel module, device tree, debug kernel panic.
    
- **Build Systems**: Yocto, Buildroot (tạo custom Linux image cho Raspberry Pi/BeagleBone).
    
- **Tools**: GDB, Git,
- **Platform tuyển dụng**:
    
    - Upwork, Toptal (dự án freelance).
        
    - LinkedIn (tìm remote job tại công ty Mỹ/Châu Âu).
        
    - RemoteOK, WeWorkRemotely (chuyên về remote).
        
- **Kỹ năng cần nhấn mạnh**:
    
    - Thành thạo Yocto/Buildroot, kinh nghiệm debug kernel.
        
    - Hiểu biết về real-time Linux (PREEMPT_RT) hoặc security (SELinux, secure boot).

### **1. Processes (Tiến trình)**

- **Chương 24 - 28**:
    
    - **Chương 24**: Giới thiệu về processes, `fork()`, `exit()`, `wait()`.
        
    - **Chương 25**: Chi tiết về `fork()`, `vfork()`, và COW (Copy-On-Write).
        
    - **Chương 26**: Thực thi chương trình mới với `exec()`.
        
    - **Chương 27**: Zombie processes và cách xử lý.
        
    - **Chương 28**: Tương tác giữa processes qua **signals** (ví dụ: `SIGCHLD`, `SIGTERM`).
        

**Ví dụ embedded**: Dùng `fork()` + `exec()` để khởi chạy một tiến trình con quản lý sensor.

---

### **2. Threads (Đa luồng)**

- **Chương 29 - 33** (Phần "Threads"):
    
    - **Chương 29**: Giới thiệu về threads (pthreads), so sánh với processes.
        
    - **Chương 30**: Tạo và hủy threads (`pthread_create()`, `pthread_join()`).
        
    - **Chương 31**: Đồng bộ hóa với **mutexes** (`pthread_mutex_lock()`, `pthread_mutex_unlock()`).
        
    - **Chương 32**: Đồng bộ hóa với **condition variables** (`pthread_cond_wait()`).
        
    - **Chương 33**: Thread-safe functions và TLS (Thread-Local Storage).
        

**Ví dụ embedded**: Tạo thread đọc cảm biến và thread ghi log song song.

---

### **3. IPC (Giao tiếp giữa các tiến trình)**

- **Phần IV (Interprocess Communication) - Chương 42-55**:
    
    - **Chương 43-44**: **Pipes** và **FIFOs** (giao tiếp một chiều).
        
    - **Chương 45-47**: **System V IPC** (Message Queues, Semaphores, Shared Memory).
        
    - **Chương 48-50**: **POSIX IPC** (Message Queues, Semaphores, Shared Memory - hiện đại hơn System V).
        
    - **Chương 51-55**: **Sockets** (TCP/UDP, giao tiếp qua mạng).
        

**Ví dụ embedded**:

- Dùng **shared memory** để trao đổi data tốc độ cao giữa 2 processes.
    
- Dùng **Unix domain sockets** cho IPC local trên embedded Linux.
    

---

### **4. Signal Handling (Xử lý tín hiệu)**

- **Chương 20-23**:
    
    - **Chương 20**: Tổng quan về signals.
        
    - **Chương 21**: Gửi signals (`kill()`, `sigqueue()`).
        
    - **Chương 22**: Bắt signals với `sigaction()`.
        
    - **Chương 23**: Signal masks và xử lý đồng bộ.
        

**Ví dụ embedded**: Bắt signal `SIGINT` để dừng ứng dụng cleanly.

---

### **5. File I/O và Memory Mapping**

- **Chương 4-5, 49**:
    
    - **Chương 4**: File I/O cơ bản (`open()`, `read()`, `write()`).
        
    - **Chương 49**: Memory-mapped I/O (`mmap()`) - hữu ích cho truy cập phần cứng trực tiếp (ví dụ: GPIO trên embedded).
        

**Ví dụ embedded**: Map memory để điều khiển GPIO qua `/dev/mem`.

---

### **6. Debugging và Timing**

- **Chương 52**: Dùng `strace` để trace system calls.
    
- **Chương 23**: Dùng `clock_gettime()` cho timing chính xác (phù hợp real-time tasks).
    

---

### **7. Những Chương Thực Tế Cho Embedded Linux**

1. **Multi-threading + Mutexes**: Chương 29-33.
    
2. **Shared Memory IPC**: Chương 48 (POSIX shared memory) hoặc 54 (System V shared memory).
    
3. **Pipes/FIFOs**: Chương 44 - Phù hợp cho streaming data (ví dụ: truyền data cảm biến).
    
4. **Signals**: Chương 20-23 - Xử lý sự kiện phần cứng (ví dụ: ngắt từ GPIO).
    

---

### **8. Ví dụ Code Trong TLPI**

- **Processes**: Xem code ví dụ tại `proc/` trong [TLPI GitHub](https://github.com/bradfa/tlpi-dist).
    
- **Threads**: Thư mục `threads/` (ví dụ: `threads/simple_thread.c`).
    
- **IPC**: Thư mục `ipc/` (ví dụ: `ipc/pipes/pipe_ls_wc.c`).
    

---

### **Cách Đọc Hiệu Quả**

1. **Tập trung vào phần ứng dụng**: Đọc lý thuyết + chạy code ví dụ trên board embedded (Raspberry Pi/BeagleBone).
    
2. **Thử modify code**: Ví dụ:
    
    - Biến code single-thread thành multi-thread.
        
    - Thay pipes bằng shared memory để tối ưu tốc độ.
        
3. **Kết hợp man pages**: Gõ `man 2 <syscall>` để xem chi tiết (ví dụ: `man 2 pthread_create`).




### **3. Chương nào có thể bỏ qua tạm thời?**

Một số chương ít dùng hoặc chuyên biệt, bạn có thể lướt qua hoặc đọc sau:

- **`Chương 15–19`**: File attributes, extended attributes (trừ khi làm việc với metadata).
    
- **`Chương 35–36`**: Terminal I/O (nếu không phát triển ứng dụng tương tác shell).
    
- **`Chương 53–55`**: Virtual memory, process resources (nếu không debug performance).
    
- **`Chương 64–65`**: Alternative I/O models, SCTP (chỉ cần nếu làm network programming).
    

---

### **4. Chương nào không nên bỏ qua?**

Dù đọc chọn lọc, hãy đảm bảo bạn nắm vững:

- **`Chương 2–5`**: File I/O (vì mọi thứ trong Linux đều là file).
    
- **`Chương 6–8`**: Process management (`fork`, `exec`).
    
- **`Chương 20–24`**: Signal handling (quan trọng cho ứng dụng real-world).
    
- **`Chương 56–63`**: Networking (nếu làm hệ thống phân tán).