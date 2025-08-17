# **Mục tiêu chung**:

1. Viết một **block‐device driver** trong kernel, giả lập một ổ đĩa ảo (ví dụ dung lượng 16 MB) để có thể cài filesystem (ext4, btrfs…) lên trên đó.

2. Viết một **C++ daemon user‐space** để làm “server lưu trữ” nhận/đáp ứng các yêu cầu đọc/ghi khối từ driver qua mạng (TCP/IP).

3. Kết hợp caching, multithreading, và thiết kế giao thức riêng (có thể dùng JSON/TLV) để tối ưu hiệu năng.

4. Cuối cùng, bạn sẽ mount filesystem (ext4) trên ổ block device ảo này, dữ liệu thực tế được lưu trên server – giống như một “mini-cloud” của riêng bạn.

🔍 Lý do “xịn” của project
- **Kernel‐space & Block Device Driver**
    
    - Việc viết một block device driver (không chỉ char device) có độ phức tạp cao hơn: bạn phải đăng ký gendisk, request_queue, tự implement hàm read/write ở mức block I/O (BIO), tạo thiết bị `/dev/vbd0` trong `/dev`.
        
    - Khi driver hoàn thiện, bạn có thể format fs (ext4, xfs) lên ngay trên thiết bị ảo này — tức là bạn sẽ hiểu sâu cách Linux vận hành block layer, buffer cache, I/O scheduling, và interaction giữa user‐space filesystem với driver.
        
- **C++ Modern User‐Space Daemon**
    
    - C++17/C++20, multithreading (std::thread, std::mutex, condition_variable), smart pointers, RAII, logging, configuration (JSON/YAML).
        
    - Networking TCP socket (POSIX) hoặc Boost.Asio (nếu muốn nâng cao).
        
    - Thiết kế giao thức ít nhất bao gồm xác định định dạng packet: header (4-byte length, 1-byte op‐code), payload JSON/TLV để mô tả sector number, data size… Mỗi request read/write block driver sẽ được gửi qua TCP đến server.
        
- **Mô phỏng “Mini-Storage‐as‐a‐Service”**
    
    - Khi driver nhận I/O, nó sẽ đẩy gói tin sang server, chờ server phản hồi dữ liệu, rồi trả về cho filesystem.
        
    - Bạn có thể mount ext4 lên `/mnt/vbd0`, thao tác như ổ cứng thật (dd, mkfs.ext4, cp file…).
        
    - Có thể thêm tính năng caching dữ liệu recent trong driver: giảm số lần request qua mạng, thêm LRU cache, eviction policy…
        
- **Khả năng mở rộng**
    
    - Thử thêm TLS (OpenSSL) để client/drvier mã hóa traffic.
        
    - Mở rộng server thành cụm: replica, failover, journaling.
        
    - Thay TCP bằng RDMA (nếu muốn nghiên cứu tối ưu hàng không).
        
    - Viết script tự động test performance (fio, dd, ioperf) để benchmark.
        
- **“Điểm cộng” cho CV**
    
    - Bạn có thể khoe “Đã viết custom Linux block device driver, mount filesystem ext4, dữ liệu thực lưu trên server C++ remote.”
        
    - Hầu hết các công ty nhúng/IoT/edge storage không bắt bạn implement hết, nhưng đi sâu vào block layer cho thấy độ hiểu biết rất chuyên sâu.

🗂️ Chi tiết các giai đoạn
#  **Giai đoạn 1: Xây dựng “Server Lưu Trữ” C++ Cơ bản (User‐Space)**

**Mục tiêu**:

- Làm quen với C++ hiện đại (C++17/20), multithreading, networking cơ bản (POSIX socket).
    
- Tạo một **daemon C++** lắng nghe trên một port (ví dụ 9000), chờ các request “READ_SECTOR” hoặc “WRITE_SECTOR”, làm file backing store (một file lớn trên disk local) để lưu dữ liệu khối.
**Tasks**:
1. Thiết kế giao thức đơn giản
	- Mỗi gói tin (frame) do driver gửi lên sẽ có header 8 byte:
	    
	    - `uint32_t total_length` (network‐byte‐order) = độ dài header (8) + payload
	        
	    - `uint8_t opcode` (1=READ, 2=WRITE)
	        
	    - `uint8_t reserved[3]` (dự phòng)
	        
	    - `uint32_t block_index` (chỉ số khối cần đọc/ghi, 0‐based)
	        
	- Nếu là WRITE, payload tiếp theo (`total_length - 8`) = 512 byte data block.
	    
	- Nếu là READ, payload không có thêm dữ liệu; server sẽ trả về gói tin với opcode=3 (READ_RESPONSE) và payload 512 byte data.
2. Xây dựng C++ Server
	- Tạo class `StorageServer` (đa luồng):
		```Cpp
		class StorageServer {
		public:
		    StorageServer(uint16_t port, const std::string& backingFilePath, size_t totalBlocks);
		    void run();               // bind(), listen(), accept() loop
		    ~StorageServer();
		private:
		    void handleClient(int clientSock);
		    std::string backingPath_;
		    int64_t               backingFd_;     // mở file backing với O_RDWR|O_CREAT
		    size_t                totalBlocks_;   // vd: 16*1024*1024/512 = 32768 blocks
		    std::mutex            fileMutex_;     // bảo vệ read/write file
		};
		```
	- Trong `handleClient()`:
	
	- Đọc header 8 byte → xác định opcode, block_index.
	    
	- Nếu opcode==READ:
	    
	    - lock fileMutex_
	        
	    - `pread(backingFd_, buffer, 512, block_index * 512)`
	        
	    - unlock
	        
	    - Tạo gói `total_length=8+512`, opcode=3 (READ_RESPONSE), cùng block_index, rồi send() 512 byte data.
	        
	- Nếu opcode==WRITE:
	    
	    - Đọc tiếp 512 byte từ socket, lock, `pwrite(backingFd_, buffer, 512, block_index * 512)`, unlock
	        
	    - Gửi ACK (opcode=4, no payload).
3. Implement Logging, Configuration
	- Dùng thư viện [spdlog](https://github.com/gabime/spdlog) (header‐only) hoặc tự viết logger đơn giản vào file `server.log`.
	- Đọc config (port, backing file path, total size) từ file JSON (dùng [nlohmann/json](https://github.com/nlohmann/json)).
4. Test Cơ bản
	- Viết một “client test” rất đơn giản (C++ hoặc Python) để kết nối tới server, gửi yêu cầu WRITE block 0, rồi READ block 0 → so sánh dữ liệu.
	- Dùng `fallocate` tạo file backing 16 MB ban đầu (hoặc để server tự tạo).
**Kết quả Giai đoạn 1**:

- Bạn có một server C++ đa luồng, đáp ứng read/write khối dữ liệu.
- Thành thạo multithreading, file I/O (pread/pwrite), basic POSIX socket, JSON config.


# **Giai đoạn 2: Viết Kernel Module Block Device Driver (Kernel-Space)**

## **Mục tiêu**:
- Viết một kernel module “virtual block device” (tạo /dev/vbd0) dung lượng tạm tính 16 MB (32768 blocks × 512 byte).
- Driver không lưu dữ liệu nội bộ: mỗi request I/O (read/write) sẽ được “đẩy” qua mạng tới server của Giai đoạn 1, chứ không phải vào RAM.
- Tìm hiểu sâu block layer Linux (gendisk, request queue, struct bio).
## **Tasks**
### Chuẩn bị môi trường build
#### Cài kernel headers:
```bash
sudo apt-get update
sudo apt-get install -y build-essential linux-headers-$(uname -r)
```
#### Tạo thư mục `kernel_module/` với `Makefile`:
```Makefile
obj-m += vbd.o

KDIR := /lib/modules/$(shell uname -r)/build
PWD  := $(shell pwd)

all:
	$(MAKE) -C $(KDIR) M=$(PWD) modules

clean:
	$(MAKE) -C $(KDIR) M=$(PWD) clean

```
### Thiết kế driver
#### Trong `vbd.c`, include các header:
```C
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/fs.h>
#include <linux/genhd.h>
#include <linux/blkdev.h>
#include <linux/hdreg.h>
#include <linux/vmalloc.h>
#include <linux/errno.h>
#include <linux/bio.h>
#include <linux/socket.h>
#include <linux/net.h>
#include <linux/in.h>
```
#### Thông số thiết bị:
```c
#define VBD_SECTOR_SIZE     512
#define VBD_SECTOR_COUNT    32768   // 32768 × 512 = 16 MB
#define VBD_DISK_NAME       "vbd0"
#define VBD_MAJOR           0       // 0 → dynamic
#define BACKUP_SERVER_IP    "192.168.1.100"
#define BACKUP_SERVER_PORT  9000
```
#### Biến toàn cục:
```c
static int vbd_major = VBD_MAJOR;
static struct vbd_dev {
    unsigned long size;            // total bytes = SECTOR_COUNT × SECTOR_SIZE
    spinlock_t  lock;              // bảo vệ request_queue
    struct request_queue *queue;   // request queue
    struct gendisk *gd;            // gendisk struct
    struct socket *sock;           // socket TCP đến storage server
} Device;
```
#### **Hàm kết nối server** (được gọi trong init_module):
- Tạo socket kernel (sock_create_kern(AF_INET, SOCK_STREAM, 0, &Device.sock)),
	
- Thiết lập địa chỉ server (struct sockaddr_in),
	
- kernel_connect(Device.sock, (struct sockaddr *)&addr, sizeof(addr), 0).
	
#### Hàm vbd_transfer(struct vbd_dev *dev, sector_t sector, unsigned long nsect, char *buffer, int write):
- Tạo gói tin TCP như đã định nghĩa (opcode=READ hoặc WRITE, kèm block_index=sector, payload 512 byte nếu write).
- Gửi gói tin qua `kernel_sendmsg()`, chờ server trả data (với READ).
- Lưu/pull data vào `buffer`.

#### Hàm vbd_request(struct request_queue \*q):
```c
static void vbd_request(struct request_queue *q) {
    struct request *req;
    while ((req = blk_fetch_request(q)) != NULL) {
        struct bio_vec bv;
        struct req_iterator iter;
        sector_t sector = blk_rq_pos(req);
        unsigned int sectors = blk_rq_sectors(req);
        rq_for_each_segment(bv, req, iter) {
            char *buffer = page_address(bv.bv_page) + bv.bv_offset;
            if (rq_data_dir(req) == READ) {
                vbd_transfer(&Device, sector, 1, buffer, 0);
            } else { // WRITE
                vbd_transfer(&Device, sector, 1, buffer, 1);
            }
            sector++;
        }
        __blk_end_request_all(req, 0);
    }
}
```
#### **Đăng ký request queue**:
```c
spin_lock_init(&Device.lock);
Device.queue = blk_init_queue(vbd_request, &Device.lock);
Device.size = VBD_SECTOR_COUNT * VBD_SECTOR_SIZE;
Device.gd = alloc_disk(1);
Device.gd->major = vbd_major;
Device.gd->first_minor = 0;
Device.gd->fops = &vbd_ops;    // struct block_device_operations (open, release, ioctl…nếu cần)
Device.gd->queue = Device.queue;
snprintf(Device.gd->disk_name, 32, VBD_DISK_NAME);
set_capacity(Device.gd, VBD_SECTOR_COUNT);
add_disk(Device.gd);
```
#### **Hàm cleanup_module**:
```c
del_gendisk(Device.gd);
blk_cleanup_queue(Device.queue);
put_disk(Device.gd);
if (Device.sock) {
    sock_release(Device.sock);
}
```
### Biên dịch & Load module
```bash
cd kernel_module
make
sudo insmod vbd.ko
```
### Test cơ bản
- Chạy song song StorageServer (phase 1) và load `vbd.ko`.
- Trên máy Linux, gõ:
```bash
sudo mkfs.ext4 /dev/vbd0
sudo mount /dev/vbd0 /mnt
echo "Hello World" > /mnt/hello.txt
cat /mnt/hello.txt
```
- Quan sát server log: mỗi read/write block tương ứng hiển thị request.
### Xử lý lỗi & tối ưu
- Thêm timeout khi connect hoặc sendmsg / recvmsg (tránh trường hợp server treo → driver “khoá” toàn bộ I/O).
- Cân nhắc queue depth (số request chờ trong queue), queue scheduling policy.
- Xử lý singal (khi unmount, driver phải hoàn thành hết request đang chờ).

### **Kết quả Giai đoạn 2**:
- Bạn đã viết thành công một block device driver “ảo” với I/O đẩy qua mạng.
- Hiểu sâu về block layer (gendisk, request_queue, struct bio), cách tương tác giữa kernel‐space và dịch vụ user‐space.