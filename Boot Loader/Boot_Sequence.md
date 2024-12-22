- [1.Boot Sequence](#1boot-sequence)
	- [1.1 Rom boot loader](#11-rom-boot-loader)
	- [1.2 Secondary Program loader-SPL (or Memory Loader-MLO)](#12-secondary-program-loader-spl-or-memory-loader-mlo)
	- [1.3 U-boot](#13-u-boot)
	- [Note: Boot Sources of the AM335x SOC](#note-boot-sources-of-the-am335x-soc)
- [2.  Linux boot sequence discussion](#2--linux-boot-sequence-discussion)
	- [2.1 ROM and SPL](#21-rom-and-spl)
	- [2.2 U-Boot](#22-u-boot)
	- [2.3 Boot strap loader](#23-boot-strap-loader)
- [3. Booting BBB from SD card](#3-booting-bbb-from-sd-card)
	- [3.1 Prepare for SD card](#31-prepare-for-sd-card)
	- [3.2 Writing uENV.txt](#32-writing-uenvtxt)
	- [3.3 Booting BBB over serial port](#33-booting-bbb-over-serial-port)
	- [3.4 Booting BBB over TFTP](#34-booting-bbb-over-tftp)
		- [3.4.1 TFTP booting Required Setup](#341-tftp-booting-required-setup)
		- [3.4.2 What we need to know](#342-what-we-need-to-know)
		- [3.4.3 Preparing TFTP host](#343-preparing-tftp-host)
- [4 Compilation command](#4-compilation-command)
	- [4.1  Cross tool-chain installation and settings for linux host](#41-cross-tool-chain-installation-and-settings-for-linux-host)
	- [4.2 U-boot Compilation ( MLO, u-boot.img )](#42-u-boot-compilation--mlo-u-bootimg-)
	- [4.3 Linux Kernel  Compilation](#43-linux-kernel--compilation)
	- [4.4 Busybox](#44-busybox)
	- [4.5  Testing boot images and busybox](#45--testing-boot-images-and-busybox)
- [5. Buildroot](#5-buildroot)



# 1.Boot Sequence
![alt text](./Image/image.png)

## 1.1 Rom boot loader
![alt text](./Image/image-1.png)
![alt text](./Image/image-6.png)
![alt text](./Image/image-8.png)



## 1.2 Secondary Program loader-SPL (or Memory Loader-MLO)

![alt text](./Image/image-2.png)
![alt text](./Image/image-7.png)

## 1.3 U-boot
![alt text](./Image/image-3.png)
![alt text](./Image/image-9.png)
![alt text](./Image/image-10.png)
![alt text](./Image/image-11.png)

<blockquote>
<h5>
<ul>
It initialize the DDR memory registers to use the DDR memory, because, MLOs job is actually to load the third stage boot loader such as the U-Boot into the external DDR memory
</ul>
</h5>
</blockquote>

![alt text](./Image/image-13.png)
![alt text](./Image/image-12.png)


**Why AM335x RBL cannot load the U-Boot directly to DDR?**
[RBL](./Text/RBL%20_cannot_load_the_Uboot.txt)

## Note: Boot Sources of the AM335x SOC

  1) NAND Flash

  2) NOR Flash (eXecute In place, XIP)

  3) USB

  4) eMMC

  5) SD card

  6) Ethernet

  7) UART

  8) SPI

![alt text](./Image/image-5.png)
![alt text](./Image/image-4.png)

# 2.  Linux boot sequence discussion

## 2.1 ROM and SPL
![alt text](./Image/image-15.png)
![alt text](./Image/image-16.png)
![alt text](./Image/image-17.png)
## 2.2 U-Boot
![alt text](./Image/image-14.png)
![alt text](./Image/image-18.png)
![alt text](./Image/image-19.png)
![alt text](./Image/image-20.png)

## 2.3 Boot strap loader

- Step 1: U boot hands off the control to the head.s file in the boot strap loader of the Linux.
	- Source file "bootm.c " of the u-boot source code
	- [Link u-boot](https://ftp.denx.de/pub/u-boot/) : arc/arm/lib/bootm.c 
		- In this file, we have a funtion **boot_jump_linux** (header_image, flag)
- Step 2 : Then, the head.s calls miscellaneous.c (misc.c) file. That also belongs to the bootstrap loader to uncompress the compressed image.
	- [Link linux kenerl](https://github.com/beagleboard/linux/tree/v5.10.168-ti-rt-r79)
	- head.S : linux/arch/arm/boot/compressed/head.S
		- In this file , at label : **start**  =>  this is a place where the control comes from U-boot to the bootstrap loader of the Linux kernel and it does lot of assembly related activities.
	- After that what happens is- the head.s file of the bootstrap loader calls the misc.c file in order to decompress the kernel (decompress_kernel -> function in misc.c file) < linux/arch/arm/boot/compressed/misc.s >
- Step 3: Then the control comes to another head.s file of the Linux kernel.
	- After that what happens is, the next step- the control after that will be passed from head.s of the bootstrap loader to another head.s of the Linux kernel. <  linux/arch/arm/kernel/head.S  >
- Step 4+ 5: And from head.s file of the Linux kernel, the control comes to the main.c file of the Linux kernel.   <  linux/arch/arm/kernel/head-common.c>  <  linux/init/main.c> (**start_kernel** function in main.c file will be called from head-common.c file)
	-  The head.s belongs to the Linux kernel architecture specific code, mostly it does CPU specific initializations of the SOC ( ex:  ARM cortex- A8 is the CPU). It's particularly interested in searching of CPU type like whether it belongs to ARM 9 or ARM 10 or ARM cortex A8, etc.
	- And once it comes to know about the architecture type, it majorly initializes the MMU and creates the initial page table entries and then enables the MMU of the processor for virtual memory support, before giving control to the main.c file of the Linux kernel which is the generic one.
- Step 6:  From main.c file, the first application of the Linux kernel that is init is launched



![alt text](./Image/image-21.png)


![[./image/Pasted image 20241028232717.png]]


# 3. Booting BBB from SD card
![[Pasted image 20241102103551.png]]
## 3.1 Prepare for SD card
- Step 1: Unmount SD card and delete  all current partitions
	![[Pasted image 20241102101431.png]]

- Step 2: Create BOOT partition : At least 1 GB and this is FAT16
	![[Pasted image 20241102101719.png]]
	- Step 3: Create ROOTFS partition ( ext3)
	![[Pasted image 20241102102044.png]]
	- Step 4: Apply all these partition ( Take a long time)
	![[Pasted image 20241102102237.png]]

- Step 5: Set boot flag for BOOT partition

## 3.2 Writing uENV.txt
- Some tips:
	- To enter Uboot mode, press reset ( in the board) and press the SPACE key ( keyboard) repeatedly.
	- Typing  "help" On the U-boot command prompt the U-boot will list out all the commands which is supported by this version of the U-boot.
	- If you want to know how a command work, you just type:  **help <command_name>**  
 - U-boot environment variables:
	 - Similar to linux environment variables, U-boot also has a set to standards as well as user defined enviromental variables which can be used to override or change behavior of U-boot
	- Type ***printenv  <variable_name>*** on the U-boot command prompt to see the value of this env variable
		- Note that, If you only type **printenv** => you will see all  environment variables
	- Type **setenv  <variable_name>  <value_for_this_variable>**  => to create a new env variable
	- Type **run <variable_name>** to run all commands stored in this variable
- What we need to setup in uENV.txt
	-  Step 1:Load  the linux kernel image to the DDR memory from MMC 0 ( SD card) interface by using this command:
		- load mmc 0:1 0x82000000 uImage
		- (type help load to know about "load" command)
	- Step 2: Load dtb image to the DDR memory from MMC ( SD card)  0 interface by using this command:
		- load mmc 0:1 0x88000000 uImage am335x-boneblack.dtb
	- Step 3: Send **bootargs** to the linux kernerl  from  u-boot: 
		- setenv bootargs  console=ttyO0,115200
	- Step 4: Boot from memory:
		- bootm 0x82000000 - 0x88000000
	- Until this step, The Linux kernel still has no idea where exactly the FS that is root file system is present, whether it is present on the USB or whether it is present on the, uh, network or ext3 file system. Now you have to edit this boot arguments to include the root fs type. ( **bootargs**)
		(Root FS is in the partition 2 of SD card: /dev/mmcblk0p2  )
		- setenv bootargs  console=ttyO0,115200  root=/dev/mmcblk0p2 rw
	![[Pasted image 20241102210901.png]]
	**=>  Our job is now to automate all these steps using the uENV.txt**
	- Minicom supports several protocols for communication between BBB and host PC. Type CTR+ A, then type S on BBB to display the protocols (Here we are using it to copy the uEnv.txt file from host pc to BBB)
	- After copying the uEnv file from host PC to BBB, we need to import it into BBB's env (because actually we only copy data from host PC (file) to BBB (at a certain address in binary form)) with the command :
		- Env -t < memory address > < size in bytes >
			- The above two parameters will be displayed when the above copy process ends.
	- Note that when creating the uEnv.txt file, there needs to be an empty line at the end

	![RFS](./Text/About_RFS.txt)

## 3.3 Booting BBB over serial port
- Press S2: SPI0 -> MMC0 ->USB0 -> UART0
![[Pasted image 20241102212108.png]]
=> Don't connect SD card , 

![[Pasted image 20241102211444.png]]![[Pasted image 20241102211458.png]]
![[Pasted image 20241102211541.png]]
![[Pasted image 20241102211957.png]]
![[Pasted image 20241102212009.png]]
![[Pasted image 20241102212022.png]]
![[Pasted image 20241102212050.png]]


- All steps:  ctrl + a -> s
	-  Load file SPL ( secondary program loader) < tự động thực thi SPL luôn và chuyển qua bước tiếp theo>
	- Load file u-boot < làm nhanh ko bị timeout >
	- Load file image bằng câu lệnh "loadx  <địa chỉ load>" < 0x82000000> ( hoặc loady,.. Tuỳ vào recommend ) sau đó ctr +a -> s -> chọn file image để load xuống địa chỉ đã chỉ định
	- Load devices tree binary => loadx 0x88000000
	- Load initramfs => loadx 0x88080000
	- setenv bootargs console=ttyo0,115200  root=/dev/ram0 rw initrd= 0x88080000
	- bootm 0x82000000 0x88080000 0x88000000

- NOTE:   
If you are facing issues with uboot boot after downloading it through XMODEM, then please refer to these threads where TI Software team suggests to use YMODEM protocol to download the uboot image instead of XMODEM. 

[https://e2e.ti.com/support/arm/sitara_arm/f/791/t/646278?AM3358-UART-boot-mode](https://e2e.ti.com/support/arm/sitara_arm/f/791/t/646278?AM3358-UART-boot-mode)

[http://processors.wiki.ti.com/index.php/AM335x_U-Boot_User%27s_Guide#Boot_Over_UART](http://processors.wiki.ti.com/index.php/AM335x_U-Boot_User%27s_Guide#Boot_Over_UART)


## 3.4 Booting BBB over TFTP

### 3.4.1 TFTP booting Required Setup
	1. Power the board using either connecting to PC via USB cable or using DC adapter.
	2. Connect Ethernet port of the BBB hardware to PC's Ethernet port using Ethernet cable.
	3. We will also use SD card for this experiment.

- First we boot  the board via SD card with these images (SPL, u-boot.img, uEnv.txt) up to u-boot
-  We transfer these images ( xx.dtb , uImage, initramfs) using TFTB from host to board over Ethernet ( We need to create a folder: var/lib/tftpboot to save these files)

### 3.4.2 What we need to know

![[Pasted image 20241102213754.png]]
![[Pasted image 20241102213821.png]]
![[Pasted image 20241102213848.png]]
![[Pasted image 20241102214017.png]]


### 3.4.3 Preparing TFTP host

Setting up a TFTP server on Ubuntu host 

- **Step 1:**  First on your Ubuntu host run the below command using your terminal program.
	$ sudo apt-get update
	$ sudo apt-get isntall
	$ sudo apt upgrade
- **Step 2:** install TFTP server : This command installs the tftpd deamon. **tftpd** is a server for the Trivial File Transfer Protocol.
    $ sudo apt-get install tftpd-hpa
- **Step 3 :** Create/Open the file “tftpd-hpa” in the below directory
	$ sudo vim /etc/default/tftpd-hpa
	 and put the below entry in to this file  and save it
- **Step 4:** Add the Following Entries: Inside the file, add the following entries:
```
TFTP_USERNAME="tftp"  
TFTP_DIRECTORY="/var/lib/tftpboot"  
TFTP_ADDRESS=":69"  
TFTP_OPTIONS="--create --secure"
```
- **Step 5:**   Create a folder /var/lib/tftpboot and execute below commands:
	$ sudo mkdir -p /var/lib/tftpboot
	$ sudo chown tftp:tftp /var/lib/tftpboot
	$ sudo chmod -R 777 /var/lib/tftpboot
- **Step 6:**Now we can start the TFTP daemon.
	$ sudo systemctl start tftpd-hpa

uENV.txt:
```
console=ttyO0,115200n8
ipaddr=192.168.7.2
serverip=192.168.1.61
absolutepath=/var/lib/tftpboot/
rootpath=/srv/nfs/bbb,nolock,wsize=1024,rsize=1024 rootwait rootdelay=5
loadtftp=echo Booting from network ...;tftpboot ${loadaddr} ${absolutepath}uImage; tftpboot ${fdtaddr} ${absolutepath}am335x-boneblack.dtb
netargs=setenv bootargs console=${console} root=/dev/nfs rw nfsroot=${serverip}:${rootpath} 
uenvcmd=setenv autoload no; run loadtftp; run netargs; bootm ${loadaddr} - ${fdtaddr}
```

# 4 Compilation command
## 4.1  Cross tool-chain installation and settings for linux host

- Step  1 : Download arm cross toolchain for your Host machine
	 -  [Link croos toolchain](https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/): gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz
- Step 2 :  Export  path of the cross compilation toolchain. (you have to include this path in the environmental variable called PATH.)
	 $ nvim /home/truonglv/.bashrc
```
		export PATH=$PATH:/home/truonglv/Downloads/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin.
```

- Step 3: Run command : **source / home/truonglv/.bashrc**
##  4.2 U-boot Compilation ( MLO, u-boot.img )

[Link U boot source code](https://openbeagle.org/beagleboard/u-boot/-/tree/v2022.04-bbb.io-am335x-am57xx)
https://github.com/RobertCNelson/u-boot

sudo find . -name "am335x_evm_defconfig" => kiểm tra có package này ko nếu ko có thì ko compile đc 
```
Các lệnh cần chạy đê có thể đáp ứng quá trình compile
Cập nhật danh sách gói:
sudo apt-get update

Cài đặt bison:
sudo apt-get install bison

Cài đặt jq:
sudo apt-get install jq

Cài đặt flex:
sudo apt-get install flex

Giải thích nguyên nhân lỗi:
Lỗi “bash: json: not found”: Lỗi này xảy ra khi hệ thống không tìm thấy chương trình json. Điều này có thể do jq chưa được cài đặt. jq là một công cụ dòng lệnh để xử lý JSON, và cài đặt nó sẽ giải quyết vấn đề này.
Lỗi “bison: not found”: Lỗi này cho thấy bison chưa được cài đặt. bison là một công cụ phân tích cú pháp, cần thiết cho quá trình biên dịch.
Lỗi “flex: not found”: Lỗi này cho thấy flex chưa được cài đặt. flex là một công cụ tạo bộ phân tích từ vựng, cũng cần thiết cho quá trình biên dịch.
```

 Configuring and generating SPL,MLO,U-boot images: (We have 2 methods)
 - Method 1: run script: build-am335x.sh ( before run this script we need change permission of this file by running this command: sudo chmod +x build-am335x.sh ) 
 - Method 2: Excute command manually

This is method 2:
**In the folder source code u-boot, which we already downloaded, we do the following steps:**

- STEP 1: distclean : deletes all the previously compiled/generated object files. 
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
- STEP 2 : apply board default configuration for uboot
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- am335x_evm_defconfig 
- STEP 3 : run menuconfig, if you want to do any settings other than default configuration . 
	make CROSS_COMPILE=arm-linux-gnueabihf-  menuconfig
- STEP 4 : compile 
	make CROSS_COMPILE=arm-linux-gnueabihf- -j8 

- [!]  *After all, we will get these file: **MLO, u-boot.img***
## 4.3 Linux Kernel  Compilation

[ 5.10.168-ti-rt-r76](**https://github.com/beagleboard/linux/tree/v5.10.168-ti-rt-r76**.)

```
Trong quá trình complie có thể gặp lỗi khi thiêu các package sau:
Cài đặt thư viện GMP:
sudo apt-get install libgmp-dev
Cài đặt thư viện MPC:
sudo apt-get install libmpc-dev
Cài đặt công cụ U-Boot:
sudo apt-get install u-boot-tools
```
- STEP 1:
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
- STEP 2:
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bb.org_defconfig 
	( For 4.11, use omap2plus_defconfig)

	(STEP2.1 : dùng cho build kernel module ( tự build )
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- prepare )

- STEP 3:
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig

- STEP 4:
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- uImage dtbs LOADADDR=0x80008000 -j8

- STEP 5:
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4 modules

- STEP 6:  you have to install all those modules ( at step 5 ) into the root file system.
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=/<**PATH_OF_RFS**> modules_install
	( **PATH_OF_RFS**  will be created at   4.4 step 4)
	
- [!] **After all , we will get :**
	+ **uImage** file at : linux/arch/arm/boot
	 + **dts** folder  at:  linux/arch/arm/boot 

## 4.4 Busybox
[Link](https://busybox.net/downloads/)  : tar.bz2 ( can download a newest version)

```
Busybox is nothing but a software tool, that enables you to create your customized root file system for your embedded Linux products.
```

- STEP 1: download busybox 

- STEP 2 : Apply default configuration
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- defconfig

- STEP 3 : change default settings if you want 
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig

- STEP 4 : generate the busy box binary and minimal file system 
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- CONFIG_PREFIX=<**PATH_OF_RFS**> install
## 4.5  Testing boot images and busybox

- Step 1: Create a folder to hold all bin files
- Step 2: Copy all bins file
	- Save the U-Boot binary :
		cp <u-boot_directory>/u-boot.img .
		cp <u-boot_directory>MLO .
		cp <u-boot_directory>/spl/u-boot-spl.bin .
	
	- Save the Linux kernel image
		cp <Linux_kerel_dir>/arch/arm/boot/uImage .
		cp <Linux_kerel_dir>/arch/arm/boot/dts/am335x-boneblack.dtb .
- Step 3:  Plug in SD card 
	- Step 3.1 : Copy  MLO and u-boot.img into BOOT section of SD card
	- Step 3.2 : Create uEnv.txt
```
console=ttyO0,115200n8
ipaddr=192.168.7.2
serverip=192.168.1.61
absolutepath=/var/lib/tftpboot/
rootpath=/srv/nfs/bbb,nolock,wsize=1024,rsize=1024 rootwait rootdelay=5
loadtftp=echo Booting from network ...;tftpboot ${loadaddr} ${absolutepath}uImage; tftpboot ${fdtaddr} ${absolutepath}am335x-boneblack.dtb
netargs=setenv bootargs console=${console} root=/dev/nfs rw nfsroot=${serverip}:${rootpath} 
uenvcmd=setenv autoload no; run loadtftp; run netargs; bootm ${loadaddr} - ${fdtaddr}
```
- where: 
	- **rootpath** and **absolutepath** are allowcated in host PC
	- **rsize** and **wsize** of BOOT partition
	- **rootdelay** in seconds
	- **absolutepath** contain : uImage and am335x-boneblack.dtb
	- **rootpath** contain : copy everything PATH_OF_RFS to here
- Step 4: Settings for the NFS access
	$ sudo nvim /etc/exports 
```
<rootpath> <BBB_IP = ipaddr >( rw,sync,no_root_squash,no_subtree_check)
```
	 
	 - After that, i will run these commands:
		$ sudo exportfs -a
		$ sudo exportfs -rv
		$ sudo service nfs-kernel-server restart

# 5. Buildroot

[Link BuildRoot](https://buildroot.org/downloads/)


We can generate all things in Section 4 by doing only Buildroot.

