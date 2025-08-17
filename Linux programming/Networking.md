### **1. Tổng quan về Socket**

- **Socket** là giao diện trừu tượng để giao tiếp giữa các tiến trình (trên cùng máy hoặc qua mạng).
    
- **Phân loại**:
    
    - **Stream sockets (TCP)**: Hướng kết nối, đảm bảo độ tin cậy.
        
    - **Datagram sockets (UDP)**: Không kết nối, tốc độ cao nhưng không đảm bảo dữ liệu.
        
    - **Raw sockets**: Truy cập trực tiếp vào tầng mạng (ví dụ: ICMP).
        

---

### **2. Các System Call Cơ Bản**

- **`socket()`**: Tạo socket.
    
    c
    
    Copy
    
    Download
    
    int fd = socket(domain, type, protocol); // Ví dụ: AF_INET, SOCK_STREAM, 0
    
- **`bind()`**: Gán địa chỉ (IP + port) vào socket.
    
    c
    
    Copy
    
    Download
    
    bind(fd, (struct sockaddr*)&addr, sizeof(addr));
    
- **`listen()`**: Chờ kết nối từ client (cho TCP server).
    
    c
    
    Copy
    
    Download
    
    listen(fd, backlog);
    
- **`accept()`**: Chấp nhận kết nối, trả về socket mới cho client.
    
    c
    
    Copy
    
    Download
    
    int client_fd = accept(fd, NULL, NULL);
    
- **`connect()`**: Kết nối đến server (cho TCP client).
    
    c
    
    Copy
    
    Download
    
    connect(fd, (struct sockaddr*)&server_addr, sizeof(server_addr));
    
- **`send()`/`recv()`**: Gửi/nhận dữ liệu (TCP).
    
- **`sendto()`/`recvfrom()`**: Gửi/nhận dữ liệu (UDP).
    

---

### **3. Địa Chỉ và Cấu Trúc Dữ Liệu**

- **`struct sockaddr_in`** (IPv4):
    
    c
    
    Copy
    
    Download
    
    struct sockaddr_in {
        sa_family_t sin_family; // AF_INET
        in_port_t sin_port;      // Port (network byte order)
        struct in_addr sin_addr; // IP (ví dụ: INADDR_ANY)
    };
    
- **`struct sockaddr_in6`** (IPv6).
    
- **`getaddrinfo()`**: Chuyển đổi địa chỉ dạng text (ví dụ: "google.com") sang dạng nhị phân.
    
- **Network Byte Order**: Sử dụng `htons()`, `ntohl()`, etc. để đảm bảo tính tương thích giữa các máy.
    

---

### **4. Thiết Kế Server**

- **Iterative Server**: Xử lý từng client một (phù hợp UDP).
    
- **Concurrent Server**: Xử lý nhiều client đồng thời (dùng `fork()`, thread, hoặc I/O multiplexing).
    
- **Mẫu TCP Server Cơ Bản**:
    
    1. `socket()` → `bind()` → `listen()` → `accept()` (vòng lặp).
        
    2. Dùng `fork()` hoặc threads để xử lý từng client.
        

---

### **5. I/O Multiplexing**

Quản lý nhiều socket cùng lúc với:

- **`select()`**: Giới hạn số file descriptor (FD_SETSIZE).
    
- **`poll()`**: Linh hoạt hơn, không giới hạn FD.
    
- **`epoll`** (Linux-specific): Hiệu suất cao cho số lượng kết nối lớn.
    

---

### **6. UDP và Xử Lý Lỗi**

- **UDP Server/Client**: Không cần `listen()`/`accept()`, dùng `sendto()`/`recvfrom()`.
    
- **Lỗi Phổ Biến**:
    
    - Quên kiểm tra giá trị trả về của system calls.
        
    - Giả định `send()`/`recv()` xử lý toàn bộ dữ liệu một lần.
        
    - Xử lý sai tín hiệu `SIGPIPE` (khi gửi đến socket đã đóng).
        

---

### **7. Advanced Topics**

- **Non-blocking Sockets**: Sử dụng `fcntl()` để thiết lập `O_NONBLOCK`.
    
- **Socket Options**: `setsockopt()` để cấu hình (ví dụ: `SO_REUSEADDR`).
    
- **Unix Domain Sockets**: Giao tiếp local hiệu suất cao (AF_UNIX).
    

---

### **8. Ví dụ Minh Họa**

**TCP Echo Server**:

c

Copy

Download

#include <sys/socket.h>
// ... (include headers)
int main() {
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr = {AF_INET, htons(8080), INADDR_ANY};
    bind(fd, (struct sockaddr*)&addr, sizeof(addr));
    listen(fd, 5);
    while (1) {
        int client_fd = accept(fd, NULL, NULL);
        char buf[1024];
        ssize_t n = read(client_fd, buf, sizeof(buf));
        write(client_fd, buf, n);
        close(client_fd);
    }
}

---

### **9. Tài Nguyên Trong TLPI**

- **Chương 56**: Sockets Introduction.
    
- **Chương 57-58**: TCP/UDP và IPv4/IPv6.
    
- **Chương 59-60**: Server Design, I/O Multiplexing.
    
- **Chương 61-63**: Advanced Socket Topics (Unix Domain Sockets, SCTP).
    

TLPI nhấn mạnh **portability** và **best practices**, kèm theo code mẫu chi tiết. Đây là tài liệu không thể thiếu cho lập trình hệ thống Linux!