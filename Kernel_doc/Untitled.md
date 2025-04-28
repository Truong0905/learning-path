### 1. **Elixir Bootlin**

**Website**: [https://elixir.bootlin.com/linux/latest/source](https://elixir.bootlin.com/linux/latest/source)

- **Tính năng chính**:
    
    - Tra cứu mã nguồn Linux kernel trực tuyến với khả năng **cross-reference** (tham chiếu chéo).
        
    - Hỗ trợ tìm kiếm struct, hàm, macro, file header, và các thành phần khác.
        
    - Tương thích với nhiều phiên bản kernel (từ rất cũ đến mới nhất).
        
    - Hiển thị code với syntax highlighting và liên kết đến các định nghĩa liên quan.
        
- **Ví dụ**:  
    Bạn muốn tra cứu struct `task_struct`, chỉ cần nhập từ khóa vào ô tìm kiếm và xem định nghĩa cùng nơi nó được sử dụng.
    

---

### 2. **Linux Kernel Documentation**

**Website**: [https://docs.kernel.org/](https://docs.kernel.org/)

- **Tính năng chính**:
    
    - Tài liệu chính thức của Linux kernel, bao gồm:
        
        - API cho core subsystems (vd: scheduler, memory management).
            
        - Hướng dẫn phát triển driver.
            
        - Giải thích chi tiết về các cấu trúc dữ liệu (struct).
            
    - Có phần **"Kernel API"** dành riêng cho lập trình viên.
        

---

### 3. **Linux man-pages**

**Website**: [https://man7.org/linux/man-pages/](https://man7.org/linux/man-pages/)

- **Tính năng chính**:
    
    - Tập trung vào system calls (vd: `open()`, `read()`, `ioctl()`) và các hàm thư viện.
        
    - Một số trang man (vd: `man 2 syscalls`) mô tả chi tiết cách giao tiếp giữa user-space và kernel-space.
        

---

### 4. **Kernel Newbies**

**Website**: [https://kernelnewbies.org/](https://kernelnewbies.org/)

- **Phù hợp cho người mới**:
    
    - Giải thích các khái niệm kernel cơ bản.
        
    - Danh sách các struct và API phổ biến (ví dụ: linked lists trong kernel).
        

---

### 5. **LXR (Linux Cross-Reference)**

**Website**: [https://lxr.missinglinkelectronics.com/linux](https://lxr.missinglinkelectronics.com/linux)

- Công cụ tương tự Elixir Bootlin nhưng ít được cập nhật thường xuyên.
    

---

### Gợi ý sử dụng:

- **Nhanh nhất**: Dùng **Elixir Bootlin** để tra cứu trực tiếp mã nguồn.
    
- **Tìm tài liệu chi tiết**: Đọc **Kernel Documentation** hoặc **man-pages**.
    
- **Học từ ví dụ**: GitHub hoặc các repo chứa mã nguồn driver mẫu (vd: [Linux Device Drivers](https://lwn.net/Kernel/LDD3/)).