### **Phần cứng cần thiết**

- **Bắt buộc**:
    
    - BeagleBone Black (BBB).
        
    - Cảm biến I2C (BMP280 – nhiệt độ/áp suất).
        
    - Màn hình OLED SPI (SSD1306).
        
- **Tùy chọn**:
    
    - Cảm biến SPI (VD: ADXL345 – gia tốc).
        

---

### **Lộ trình 5 tháng với Yocto**

#### **Tháng 1: Thiết lập Yocto Project cho BBB**

- **Công việc**:
    
    - Tạo custom Yocto layer cho BBB (dùng `meta-ti` làm base).
        
    - Viết recipe cho:
        
        - Thư viện C++ (Boost, Paho MQTT).
            
        - Tools debug (gdbserver, strace).
            
    - Tích hợp SPI/I2C kernel drivers qua Device Tree Overlay.
        
- **Kỹ năng**: Yocto layer management, bitbake syntax, kernel configuration.
    
- **Deliverables**:
    
    - Custom image boot được BBB, có sẵn `/dev/i2c-1`, `/dev/spidev1.0`.
        
    - GitHub repo: Yocto layer + documentation.
        

#### **Tháng 2: Kernel Driver cho I2C & SPI**

- **Công việc**:
    
    - **I2C Driver**:
        
        - Viết kernel module đọc data từ BMP280 (sử dụng `i2c-dev` interface).
            
        - Expose data qua sysfs (`/sys/bmp280/temperature`).
            
    - **SPI Driver**:
        
        - Viết kernel module cho SSD1306 (khởi tạo màn hình, vẽ pixel).
            
        - Tạo character device (`/dev/ssd1306`) để userspace điều khiển.
            
    - **Tích hợp vào Yocto**:
        
        - Thêm kernel module vào image qua `IMAGE_INSTALL_append`.
            
- **Kỹ năng**: Kernel programming, Yocto recipe, device tree.
    
- **Deliverables**:
    
    - Code kernel module + Yocto patch.
        
    - Demo đọc nhiệt độ và hiển thị lên OLED.
        

#### **Tháng 3: Ứng dụng C++ Đa luồng & IPC**

- **Công việc**:
    
    - **Viết ứng dụng C++**:
        
        - Luồng 1: Đọc data từ I2C (BMP280) và SPI (ADXL345 – nếu có).
            
        - Luồng 2: Xử lý data (filter noise, tính toán).
            
        - Luồng 3: Hiển thị lên OLED và gửi qua MQTT.
            
        - Dùng **ZeroMQ** (IPC) để giao tiếp giữa các luồng.
            
    - **Tích hợp vào Yocto**:
        
        - Tạo recipe cho ứng dụng, thêm vào image.
            
- **Kỹ năng**: C++ multithreading, ZeroMQ, Yocto SDK.
    
- **Deliverables**:
    
    - Ứng dụng chạy trên Yocto image, code + unit test.
        

#### **Tháng 4: Tối ưu hóa & Bảo mật**

- **Công việc**:
    
    - **Tối ưu Yocto Image**:
        
        - Giảm image size bằng cách loại bỏ package không cần thiết.
            
        - Tích hợp `busybox` thay thế coreutils.
            
    - **Security**:
        
        - Kích hoạt SELinux trong Yocto.
            
        - Mã hóa data MQTT bằng OpenSSL (viết recipe cho thư viện crypto).
            
    - **Real-Time**:
        
        - Patch kernel với PREEMPT_RT cho ứng dụng real-time.
            
- **Kỹ năng**: Yocto optimization, security hardening.
    
- **Deliverables**:
    
    - Báo cáo benchmark: Boot time, memory usage.
        

#### **Tháng 5: CI/CD & Triển khai**

- **Công việc**:
    
    - **CI/CD Pipeline**:
        
        - Tích hợp Jenkins/GitLab CI để auto-build Yocto image khi có code mới.
            
        - Viết script test tự động (ví dụ: pytest cho ứng dụng).
            
    - **Cloud Integration**:
        
        - Gửi data lên AWS IoT Core/ThingsBoard.
            
        - Visualize data bằng Grafana.
            
    - **Documentation**:
        
        - Viết user manual cho hệ thống.
            
- **Kỹ năng**: DevOps, cloud, technical writing.
    
- **Deliverables**:
    
    - Video demo end-to-end: Từ cảm biến → OLED → Cloud Dashboard.
        

---

### **Cách trình bày portfolio**

1. **GitHub Structure**:
    
    Copy
    
    /meta-custom-layer      # Yocto layer  
      /recipes-kernel       # Kernel modules  
      /recipes-app          # Ứng dụng C++  
    /hardware-setup         # Hướng dẫn kết nối phần cứng  
    /ci-cd                  # Jenkinsfile, test scripts  
    
2. **Blog Posts**:
    
    - _"Xây dựng Custom Linux OS cho BBB với Yocto: Từ Zero đến Hero"_.
        
    - _"Debug Race Condition trong Multi-threaded C++ trên Embedded Linux"_.
        
3. **LinkedIn Highlights**:
    
    - Post ảnh Yocto build process, video demo hệ thống.
        
    - Tag công ty như **Renesas**, **Toradex** (dùng Yocto nhiều).
        

---

### **Câu hỏi phỏng vấn dự đoán & Cách trả lời**

1. **"Tại sao chọn Yocto thay vì Buildroot?"**  
    → _"Yocto linh hoạt hơn cho dự án phức tạp, dễ maintain qua các phiên bản, và được ưa chuộng trong industry."_
    
2. **"Bạn xử lý dependency conflict trong Yocto thế nào?"**  
    → _"Dùng `bitbake-layers show-layers` để trace dependency, override recipe bằng `bbappend`."_
    
3. **"Làm sao debug kernel module crash trên Yocto image?"**  
    → _"Dùng `dmesg`, `klogd`, và compile kernel với debug symbol."_

### **Kỹ năng Debug & Cách Tích Hợp Vào Dự án**

#### **1. Debug Kernel Module**

- **Công cụ**:
    
    - `dmesg`/`journalctl`: Theo dõi kernel log thời gian thực.
        
    - `kgdb`: Remote debug kernel qua Ethernet/USB.
        
    - `kprobes`: Trace hàm kernel để phát hiện lỗi logic.
        
- **Tình huống trong dự án**:
    
    - **Lỗi không đọc được data từ BMP280 (I2C)**:
        
        - Dùng `i2cdetect` kiểm tra device address.
            
        - Chạy `logic analyzer` (Saleae) để xem tín hiệu SCL/SDA.
            
        - Thêm printk() trong kernel module để log quá trình đọc I2C.
            
    - **Kernel panic khi load module**:
        
        - Biên dịch kernel với CONFIG_DEBUG_INFO=y để có symbol.
            
        - Dùng `crash` utility phân tích core dump.
            

#### **2. Debug Ứng dụng Userspace (C++)**

- **Công cụ**:
    
    - **GDB**: Debug multi-thread app với `gdb-multiarch` + `gdbserver`.
        
    - **Valgrind**: Phát hiện memory leak, race condition (Helgrind).
        
    - **Strace**: Theo dõi system call và signal.
        
- **Tình huống trong dự án**:
    
    - **Data corruption khi giao tiếp giữa các luồng**:
        
        - Dùng `Helgrind` để phát hiện race condition.
            
        - Thêm mutex logging để trace lock/unlock sequence.
            
    - **Ứng dụng crash khi gửi MQTT**:
        
        - Chạy với `gdb` và đặt breakpoint tại hàm `paho_mqtt_publish()`.
            
        - Kiểm tra memory allocation bằng `Valgrind --tool=memcheck`.
            

#### **3. Debug Phần Cứng (SPI/I2C)**

- **Công cụ**:
    
    - **Logic Analyzer** (Saleae): Capture tín hiệu SPI/I2C để kiểm tra xung clock, data.
        
    - **Oscilloscope**: Đo điện áp chân GPIO, phát hiện noise.
        
    - **Các lệnh Linux**:
        
        - `i2c-tools` (i2cdump, i2cset): Tương tác trực tiếp với I2C device.
            
        - `spidev_test`: Kiểm tra SPI communication.
            
- **Tình huống trong dự án**:
    
    - **Màn hình OLED SPI không hiển thị**:
        
        - Dùng `spidev_test` để gửi test pattern.
            
        - Kiểm tra chân CS (Chip Select) bằng oscilloscope.
            
    - **Nhiệt độ đọc từ BMP280 sai**:
        
        - Capture I2C transaction bằng logic analyzer, so sánh với datasheet.
            
        - Kiểm tra calibration data trong sensor.
            

#### **4. Debug Yocto Build**

- **Công cụ**:
    
    - `bitbake -D`: Hiển thị debug log khi build.
        
    - `devshell`: Mở shell trong môi trường build của Yocto.
        
    - `QA Report`: Kiểm tra lỗi dependency, file conflict.
        
- **Tình huống trong dự án**:
    
    - **Build Yocto bị lỗi do recipe conflict**:
        
        - Chạy `bitbake -c cleanall <recipe>` để rebuild từ đầu.
            
        - Dùng `oe-pkgdata-util` để kiểm tra package dependencies.
            
    - **Image không boot được**:
        
        - Kiểm tra U-Boot log qua serial console.
            
        - Verify kernel command line args trong `boot.scr`.
            

---

### **Cách Trình Bày Kỹ Năng Debug Khi Phỏng Vấn**

1. **Kể Câu Chuyện Debug Thực Tế**:
    
    - _"Trong dự án, mình gặp lỗi ứng dụng crash ngẫu nhiên khi gửi MQTT. Mình đã dùng Valgrind để phát hiện memory leak do không giải phóng biến cấp struct, sau đó sửa bằng smart pointer trong C++."_
        
2. **Demo Video Debug**:
    
    - Quay màn hình terminal khi dùng GDB để fix race condition.
        
    - Hiển thị waveform SPI/I2C trên logic analyzer.
        
3. **Chuẩn Bị Sổ Tay Debug**:
    
    - Liệt kê các lệnh debug hay dùng (dmesg, strace, i2cdetect) và ví dụ.
        

---

### **Bổ Sung Vào Lộ Trình Dự Án**

#### **Tháng 1: Thiết Lập Công Cụ Debug**

- Cài đặt OpenOCD, GDB, logic analyzer driver trên PC.
    
- Viết script tự động capture kernel log khi boot.
    

#### **Tháng 3: Debug Ứng dụng Đa Luồng**

- Thêm unit test với Google Test, tích hợp Valgrind vào CI/CD.
    
- Viết test case gây race condition cố ý để demo cách debug.
    

#### **Tháng 5: Tổng Hợp Báo Cáo Debug**

- Tạo tài liệu "Debug Handbook" ghi lại tất cả lỗi đã gặp và cách khắc phục.

