**04/11/2024**

- [l] Mastering_Embedded_Linux_Programming_ElectroVolt.ir_.pdf
# 1. Starting out
- Selecting the right operating system
	- Linux works well where the problem being solved justifies the complexity. It is especially good where connectivity, robustness, and complex user interfaces are required.
	- However it cannot solve every problem, so here are some things to consider before you jump in:
		- Linux requires a lot more resources. It needs at least a 32-bit processor, and lots more memory
		- The early parts of a project, board bring-up, require detailed knowledge of Linux and how it relates to your hardware. Likewise, when debugging and tuning your application, you will need to be able to interpret the results.
		- Linux can handle many real-time activities so long as you pay attention to certain details
	- The main players are:
		- The open source community
		- CPU architects—These are the organizations that design the CPUs we use.
		- SoC vendors (Atmel, Broadcom, Freescale, Intel, Qualcomm, TI, and many others)—They take the kernel and toolchain from the CPU architects and modify it to support their chips.
		- Board vendors and OEMs—these people take the reference designs from SoC vendors and build them in to specific products, for instance set-top boxes or cameras, or create more general purpose development boards, such as those from Avantech and Kontron.
		- => These form a chain, with your project usually at the end, which means that you do not have a free choice of components. You cannot simply take the latest kernel from kernel.org, except in a few rare cases, because it does not have support for the chip or board that you are using.

- Project lifecycle
	- Set up the development environment and create a working platform
	- You will have to make concerning the storage of programs and data, how to divide work between kernel device drivers and applications, and how to initialize the system.
	- Writing embedded applications
	- Debugging and optimizing performance
- The four elements of embedded Linux:
	- Toolchain: This consists of the compiler and other tools needed to create code for your target device. Everything else depends on the toolchain.
	- Bootloader: This is necessary to initialize the board and to load and boot the Linux kernel.
	- Kernel: This is the heart of the system, managing system resources and interfacing with hardware.
	- Root filesystem: This contains the libraries and programs that are run once the kernel has completed its initialization
- Licenses:
	- GPL: The GPL licenses has clauses which compel you to pass the rights to obtain and modify the software on to your end users. . The GPL goes further to say that you cannot incorporate GPL code into proprietary programs. Any attempt to do so would make the GPL apply to the whole.
	- So, what about libraries? If they are licensed with the GPL, any program linked with them becomes GPL also. However, most libraries are licensed under the Lesser General Public License (LGPL). If this is the case, you are allowed to link with them from a proprietary program.
# 2. Learning About Toolchains

## 2.1 A standard GNU toolchain consists of three main components

- **Binutils**: A set of binary utilities including the assembler, and the linker, ld. It is available at http://www.gnu.org/software/binutils/

- **GNU** Compiler Collection (GCC): These are the compilers for C and other languages which, depending on the version of GCC, include C++, Objective-C, Objective-C++, Java, Fortran, Ada, and Go. They all use a common back-end which produces assembler code which is fed to the GNU assembler. It is available at http://gcc.gnu.org/

- **C** library: A standardized API based on the POSIX specification which is the principle interface to the operating system kernel from applications. 

## 2.2 Types of toolchain
- For our purposes, there are two types of toolchain:
	- **Native**: This toolchain runs on the same type of system, sometimes the same actual system, as the programs it generates. This is the usual case for desktops and servers, and it is becoming popular on certain classes of embedded devices. The Raspberry Pi running Debian for ARM, for example, has self-hosted native compilers. 
	-  **Cross**: This toolchain runs on a different type of system than the target, allowing the development to be done on a fast desktop PC and then loaded onto the embedded target for testing.

## 2.3  CPU architectures
- The toolchain has to be built according to the capabilities of the target CPU, which includes: 
	-  CPU architecture: arm, mips, x86_64, and so on 
	-  Big- or little-endian operation: Some CPUs can operate in both modes, but the machine code is different for each
	-  Floating point support: Not all versions of embedded processors implement a hardware floating point unit, in which case, the toolchain can be configured to call a software floating point library instead 
	- ==Application Binary Interface== (**ABI**): The calling convention used for passing parameters between function calls
		- Extended Application Binary Interface (EABI)
		- Old Application Binary Interface (OABI)
		- EABIHF
## 2.4 GNU uses a prefix 
- to the tools to identify the various combinations that can be generated, consisting of a tuple of three or four components separated by dashes, as described here:
	- CPU: arm, mips, or x86_64.
	- Vendor: arm, mips, or x86_64.
	- kernel: For our purposes, it is always 'linux'.
	- Operating system: A name for the user space component, which might be gnu or uclibcgnu. The ABI may be appended here as well so, for ARM toolchains, you may see gnueabi, gnueabihf, uclibcgnueabi, or uclibcgnueabihf.
	=> Here is an example using a cross compiler:
		CROSS_COMPILER :=arm-linux-gnueabihf-

## 2.5 Choosing the C library
- Even if you are writing programs in another language, maybe Java or Python, the respective run-time support libraries will have to call the C library eventually:
![[Pasted image 20241106220957.png]]
-  Whenever the C library needs the services of the kernel it will use the kernel **system call** interface to transition between user space and kernel space
- There are several C libraries to choose from. The main options are as follows:
	- glibc: Available at http://www.gnu.org/software/libc. It is the standard GNU C library. It is big and, until recently, not very configurable, but it is the most complete implementation of the POSIX API.
	- eglibc: Available at http://www.eglibc.org/home. This is the embedded GLIBC. It was a series of patches to glibc which added configuration options and support for architectures not covered by glibc (specifically, the PowerPC e500). The split between eglibc and glibc was always rather artificial and, fortunately, the code base from eglibc has been merged back into glibc as of version 2.20, leaving us with one improved library. eglibc is no longer maintained.
	- uClibc: Available at http://www.uclibc.org. The 'u' is really a Greek 'mu' character, indicating that this is the micro controller C library. It was first developed to work with uClinux (Linux for CPUs without memory management units), but has since been adapted to be used with full Linux. There is a configuration utility which allows you to fine-tune its features to your needs. Even a full configuration is smaller than glibc but it is not as complete an implementation of the POSIX standards
	- musl libc: Available at http://www.musl-libc.org. It is a new C library designed for embedded systems.
- => So, which to choose?
	![[Pasted image 20241106222014.png]]


## 2.6 Finding a toolchain

### 2.6.1 Building a toolchain using crosstool-NG

#### 2.6.1.1 Installing crosstool-NG
- You will need to install the packages using the following command:
```
sudo apt-get install automake bison chrpath flex g++ git gperf gawk libexpat1-dev libncurses5-dev libsdl1.2-dev libtool python2.7-dev texinfo
```
  Link crosstool-NG : [Index of /download/crosstool-ng/](http://crosstool-ng.org/download/crosstool-ng/)
```
$ tar xf crosstool-ng-1.20.0.tar.bz2
$ cd crosstool-ng-1.20.0 
$ ./configure --enable-local
$ make
$ make install
```
#### 2.6.1.2 Selecting the toolchain
- Crosstool-NG can build many different combinations of toolchains. To make the initial configuration easier, it comes with a set of samples that cover many of the common use-cases. Use *./ct-ng list-samples* to generate the list
- a BeagleBone Black which has an ARM Cortex A8 core and a VFPv3 floating point unit, and that you want to use a current version of glibc. The closest sample is arm-cortex_a8-linux-gnueabi. You can see the default configuration by prefixing the name with show-:
```
./ct-ng show-arm-cortex_a8-linux-gnueabi
```

- To select this as the target configuration, you would type:
```
./ct-ng arm-cortex_a8-linux-gnueabi
```
- At this point, you can review the configuration and make changes using the configuration menu command menuconfig:

```
./ct-ng menuconfig
```


- There are a few configuration changes that I would recommend you make at this point:
	• In Paths and misc options, disable Render the toolchain read-only
	
	(CT_INSTALL_DIR_RO)
	
	• In Target options | Floating point, select hardware (FPU)
	
	(CT_ARCH_FLOAT_HW)
	
	• In C-library | extra config, add --enable-obsolete-rpc
	
	(CT_LIBC_GLIBC_EXTRA_CONFIG_ARRAY)

```
The first is necessary if you want to add libraries to the toolchain after it has been installed, which I will describe later in this chapter. The next is to select the optimum floating point implementation for a processor with a hardware floating point unit. The last forces the toolchain to be generated with an obsolete header file, rpc.h, which is still used by a number of packages (note that this is only a problem if you selected glibc). The names in parentheses are the configuration labels stored in the configuration file. When you have made the changes, exit menuconfig, and save the configuration as you do so	
```

- Toolchain ABIs that ARM has two variants, one which passes floating point parameters in integer registers and one that uses the VFP registers? The float configuration you have just chosen is of the latter type and so the ABI part of the tuple should read eabihf. There is a configuration parameter that does exactly what you want but it is not enabled by default, neither does it appear in the menu, at least not in this version of crosstool. Consequently, you will have to edit .config and add the line shown in bold as follows:
```
#

# arm other options #

CT_ARCH_ARM_MODE="arm" CT_ARCH_ARM_MODE_ARM=y # CT_ARCH_ARM_MODE_THUMB is not set # CT_ARCH_ARM_INTERWORKING is not set CT_ARCH_ARM_EABI_FORCE=y CT_ARCH_ARM_EABI=y CT_ARCH_ARM_TUPLE_USE_EABIHF=y
```
		
- Now you can use crosstool-NG to get, configure, and build the components according to your specification by typing the following command:
```
$ ./ct-ng build
```

	The build will take about half an hour, after which you will find your toolchain is present in ~/x-tools/arm-cortex_a8-linux-gnueabihf/.

#### 2.6.1.3 Finding out about your cross compiler
- To find the version
```
arm-cortex_a8-linux-gnueabihf-gcc --version
```
To find how it was configured, use -v:
```
arm-cortex_a8-linux-gnueabihf-gcc -v
```

- If you plan to create a new toolchain for each target, then it makes sense to set everything up at the beginning because it will reduce the risks of getting it wrong later on => Buildroot
- If, on the other hand, you want to build a toolchain that is generic and you are prepared to provide the correct settings when you build for a particular target, then you should make the base toolchain generic, which is the way the Yocto Project handles things

#### 2.6.1.4 The sysroot, library, and header files
- The toolchain sysroot is a directory which contains subdirectories for libraries, header files, and other configuration files. It can be set when the toolchain is configured through --with-sysroot= or it can be set on the command line, using --sysroot=
```
arm-cortex_a8-linux-gnueabihf-gcc -print-sysroot

=> /home/truonglv/x-tools/arm-cortex_a8-linux-gnueabihf/arm-cortex_a8-linux-gnueabihf/sysroot
```

#### 2.6.1.5 Linking with libraries: static and dynamic linking
- The library code can be linked in two different ways: statically, meaning that all the library functions your application calls and their dependencies are pulled from the library archive and bound into your executable; and dynamically, meaning that references to the library files and functions in those files are generated in the code but the actual linking is done dynamically at runtime
- **Static libraries** :
	- You tell gcc to link all libraries statically by adding -static to the command line: 
	```
	$ arm-cortex_a8-linux-gnueabihf-gcc -static helloworld.c -o helloworld-static
	```

	-  Creating a static library is as simple as creating an archive of object files using the **ar** command. If I had two source files named test1.c and test2.c and I want to create a static library named libtest.a, then I would do this:
		$ arm-cortex_a8-linux-gnueabihf-gcc -c test1.c 
		$ arm-cortex_a8-linux-gnueabihf-gcc -c test2.c
		$ arm-cortex_a8-linux-gnueabihf-ar rc libtest.a test1.o test2.o
	- Then I could link libtest into my helloworld program using:
		$ arm-cortex_a8-linux-gnueabihf-gcc helloworld.c -ltest -L../libs - I../libs -o helloworld

- **Shared libraries** :
	The object code for a shared library must be position-independent so that the runtime linker is free to locate it in memory at the next free address. To do this, add the **-fPIC** parameter to gcc, and then link it using the **-shared** option: 
		$ arm-cortex_a8-linux-gnueabihf-gcc -fPIC -c test1.c
		$ arm-cortex_a8-linux-gnueabihf-gcc -fPIC -c test2.c
		$ arm-cortex_a8-linux-gnueabihf-gcc -shared -o libtest.so test1.o test2.o
	To link an application with this library, you add -ltest, exactly as in the static case mentioned in the preceding paragraph but, this time, the code is not included in the executable, but there is a reference to the library that the runtime linker will have to resolve: 
		$ arm-cortex_a8-linux-gnueabihf-gcc helloworld.c -ltest -L../libs - I../libs -o helloworld
		
	The linker will look for libtest.so in the default search path: /lib and /usr/lib. If you want it to look for libraries in other directories as well, you can place a colon-separated list of paths in the shell variable LD_LIBRARY_PATH: 
	  export LD_LIBRARY_PATH=/opt/lib:/opt/usr/lib

# 3. All About Bootloaders

## 3.1 Building U-Boot

$ git clone git://git.denx.de/u-boot.git

- STEP 1: distclean : deletes all the previously compiled/generated object files. 
	make ARCH=arm CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf- distclean
- STEP 2 : apply board default configuration for uboot
	make ARCH=arm CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf- am335x_evm_defconfig 
- STEP 3 : run menuconfig, if you want to do any settings other than default configuration . 
	make CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf-  menuconfig
- STEP 4 : compile 
	make CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf- -j8 

The results of the compilation are:
	• u-boot: This is U-Boot in ELF object format, suitable for use with a debugger	
	• u-boot.map: This is the symbol table	
	• u-boot.bin: This is U-Boot in raw binary format, suitable for running on your device	
	• u-boot.img: This is u-boot.bin with a U-Boot header added, suitable for uploading to a running copy of U-Boot
	• u-boot.srec: This is U-Boot in Motorola srec format, suitable fortransferring over a serial connection

After that
- cp MLO u-boot.img /media/truonglv/BOOT/
- [!] Note: To enter u-boot mode, you need hold button S2 and press any key in keyboard

## 3.2 Using U-Boot

## 3.3 Environment variables
- For example **setenv foo bar** creates the variable **foo** with the value **bar**
- You can delete a variable by setting it to a null string, **setenv foo**
- You can print all the variables to the console using printenv, or a single variable using **printenv foo**
- Usually, it is possible to use the **saveenv** command to save the entire environment to permanent storage of some kind.
	- ["] If there is raw NAND or NOR flash, then an erase block is reserved for this purpose, often with another used for a redundant copy to guard against corruption. If there is eMMC or SD card storage it can be stored in a file in a partition of the disk. Other options include storing in a serial EEPROM connected via an I2C or SPI interface or non-volatile RAM.

## 3.4 Automating the boot with U-Boot scripts

- [?] **How to write Bootscripts**: The bootscript is an script that is automatically executed when the boot loader starts, and before  the OS auto boot process.
- [x] Step 1: First, you need the **u-boot-tools** installed in your Linux machine:
```markup
sudo apt install u-boot-tools
```
=> That package provide to us the tool **mkimage** to convert a text file (.src, .txt) file to a bootscript file for U-Boot.
- [x] Step 2: Now, create your custom script, in this case a simple script for load binary file:
```markup
nvim mycustomscript.scr
```
- [x]  Now we can convert the text file to bootscript with mkimage
```
mkimage -A arm -O linux -T kernel -C gzip -a 0x80008000 -e 0x80008000 -n 'Linux' -d uENV.txt uENV
```
- [ ] **Insert the SD card into your BeagleBone Black** and power it on.
- [ ] **Set the boot command in U-Boot**:
```
setenv bootcmd 'load mmc 0:1 0x80000000 boot.scr; source 0x80000000'
```
- [ ] **Save the environment** to make sure the changes persist:
```
 saveenv
```
After completing these steps, your BeagleBone Black should automatically run the U-Boot script each time it boots.

## 3.5 Porting U-Boot to a new board

- [?] Let's assume that your hardware department has created a new board called "Nova" that is based on the BeagleBone Black and that you need to port U-Boot to it. ( Please read the book again to know more details)

### 3.5.1 In U-boot folder
- U-boot configuration settings now is located in **Kconfig** files 
- Each board had a **Kconfig** file which contains minimal information derived from the old **boards.cfg** file.
- The main directories you will be dealing with are :
	- **arch**: Contains code specific to each supported architecture in directories arm, mips, powerpc, and so on. Within each architecture, there is a subdirectory for each member of the family, for example, in arch/arm/cpu, there are directories for the architecture variants, including amt926ejs, armv7, and armv8.
	- **board**: Contains code specific to a board. Where there are several boards from the same vendor, they can be collected together into a subdirectory, hence the support for the am335x evm board, on which the BeagelBone is based, is in board/ti/am335x.
	- **common**: Contains core functions including the command shells and the commands that can be called from them, each in a file named cmd_\[command name].c.
	- **doc**: Contains several README files describing various aspects of U-Boot. If you are wondering how to proceed with your U-Boot port, this is a good place to start.
	- **include**: In addition to many shared header files, this contains the very important subdirectory include/configs where you will find the majority of the board configuration settings. As the move to Kconfig progresses, the information will be moved out into Kconfig files but, at the time of writing, that process has only just begun
 
### 3.5.2 Kconfig and U-Boot:
 
- **Kconfig** extracts configuration information from Kconfig files and stores the total system configuration in a file named **.config**
- A U-Boot build can produce up to three binaries: a normal u-boot.bin, a **Secondary Program Loader (SPL)**, and a **Tertiary Program Loader (TPL)**, each with possibly different configuration options. Consequently, lines in **.config** and default configuration files can be prefixed with the codes shown in the following table to indicate which target they apply to:
![[Pasted image 20241207104514.png]]

- Now, for your new board, you need to create some new files:
	- 1. Each board has a default configuration stored in configs/[board name}\_defconfig. 
	```c
	CONFIG_SPL=y 
	CONFIG_SYS_EXTRA_OPTIONS="SERIAL1,CONS_INDEX=1,EMMC_BOOT"
	+S:CONFIG_ARM=y +S:CONFIG_TARGET_NOVA=y
	```
	- ["] On the first line, CONFIG_SPL=y causes the SPL binary, MLO, to be generated, CONFIG_ARM=y causes the contents of arch/arm/Kconfig to be included on line three. On line four, CONFIG_TARGET_NOVA=y selects your board. Note that lines three and four are prefixed by +S: so that they apply to both the SPL and normal binaries. You should also add a menu option to the ARM architecture Kconfig that allows people to select Nova as a target:
	```c
	CONFIG_SPL=y 
	config TARGET_NOVA 
	bool "Support Nova!"
	```

	- 2. Board-specific files: Each board has a subdirectory named board/\[board name] or board/\[vendor]/ \[board name] which should contain:
	
		- Kconfig: Contains configuration options for the board 
		-  MAINTAINERS: Contains a record of whether the board is currently maintained and, if so, by whom
		-  Makefile: Used to build the board-specific code
		-  README: Contains any useful information about this port of U-Boot, for example, which hardware variants are covered
		```
		$ mkdir board/nova
		```

	- Configuration header files: Each board has a header file in include/configs which contains the majority of the configuration. The file is named by the SYS_CONFIG_NAME identifier in the board's Kconfig. The format of this file is described in detail in the README file at the top level of the U-Boot source tree.

### 3.5.3 Building and testing
- To build for the Nova board, select the configuration you have just created:
$ make CROSS_COMPILE=arm-cortex_a8-linux-gnueabi- nova_defconfig
$ make CROSS_COMPILE=arm-cortex_a8-linux-gnueabi 
- Copy MLO and u-boot.img to the FAT partition of the micro-SD card you created earlier and boot the board



# 4. Porting and Configuring the Kernel

## 4.1 Building the kernel

### 4.1.1 Inside Linux kernel folder
The main directories of interest are:
- **arch**: This contains architecture-specific files. There is one subdirectory per architecture.
- **Documentation**: This contains kernel documentation. Always look here first if you want to find more information about an aspect of Linux.
- **drivers**: This contains device drivers, thousands of them. There is a subdirectory for each type of driver
- **fs**: This contains filesystem code
- **include**: This contains kernel header files, including those required when building the toolchain
- **init**: This contains the kernel start-up code
- **kernel**: This contains core functions, including scheduling, locking, timers, power management, and debug/trace code.
- **mm**: This contains memory management
- **net**: This contains network protocols.
- **scripts**: This contains many useful scripts including the device tree compiler, dtc,
- **tools**: This contains many useful tools, including the Linux performance counters tool, perf
### 4.1.2 Understanding kernel configuration

- The configuration mechanism is called **Kconfig**, and the build system that it integrates with is called **Kbuild**.
- Both are documented in **Documentation/kbuild/**.
- The configuration options are declared in a hierarchy of files named Kconfig using a syntax described in **Documentation/kbuild/kconfig-language.rst**. ( *Have to read it to understand how to write Kconfig file* )
- All configuration items are stored in **.config** file. The variable names stored in **.config** are prefixed with **CONFIG_** . For example:
```
CONFIG_DEVMEM=y
```
- [i] Therefore, DEVMEM will be enabled

- There are several other data types in addition to bool. Here is the list:
	-  **bool**: This is either y or not defined.
	-  **tristate**: This is used where a feature can be built as a kernel module or built into the main kernel image. The values are m for a module, y to be built in, and not defined if the feature is not enabled. 
	- **int**: This is an integer value written using decimal notation. 
	- **hex**: This is an unsigned integer value written using hexadecimal notation. 
	-  **string**: This is a string value.
-  There may be dependencies between items, expressed by the **depends on** phrase, as shown here:
```
config MTD_CMDLINE_PARTS
	tristate "Command line partition table parsing"
	depends on MTD
```
- [!] If CONFIG_MTD has not been enabled elsewhere, this menu option is not shown and so cannot be selected.

- There are also reverse dependencies: the **select** keyword enables other options if this one is enabled. The **Kconfig** file in **arch/$ARCH** has a large number of select statements that enable features specific to the architecture, as can be seen here for **arm**:
```
config ARM
	bool
default y
	select ARCH_HAS_ATOMIC64_DEC_IF_POSITIVE 
	select ARCH_HAS_ELF_RANDOMIZE 
[...]
```
- [I] With so many things to configure, it is unreasonable to start with a clean sheet each time you want to build a kernel so there are a set of known working configuration files in arch/$ARCH/configs, each containing suitable configuration values for a single SoC or a group of SoCs. You can select one with make \[configuration file name].

### 4.1.3 Using LOCALVERSION to identify your kernel
- You will find in the **General setup configuration** menu. Or you can define the local version in the .config file like this:
```
CONFIG_LOCALVERSION="-myversion-v1.0"
```
- Running make kernelversion produces the same output as before but now, if I run make kernelrelease, I see:
```
$ make kernelrelease 5.1.10-myversion-v1.0
```
- ["] I can now identify and track my custom kernel.

### 4.1.4 Kernel modules
- Desktop Linux distributions use them extensively so that **the correct device and kernel functions** can be **loaded** at **runtime** depending on the **hardware detected and features required**. Without them, every single driver and feature would have to be statically linked in to the kernel, making it unfeasibly large.

## 4.2 Compiling

- The kernel build system, **kbuild**, is a set of make scripts that **take the configuration** information **from** the **.config** file.
- **Work out the dependencies** and compile everything that is necessary to produce a kernel image containing all the **statically linked components**, possibly a **device tree binary** and possibly **one or more kernel modules**.
### 4.2.1  Compiling the kernel image

- ["] There is a small issue with building a uImage file for ARM with multi-platform support. This support allows a single kernel binary to run on multiple platforms and was introduced in Linux 3.7. The kernel selects the correct platform by reading the machine number or the device tree passed by the bootloader. The issue arises because the physical memory location may vary across platforms, meaning the kernel relocation address, typically 0x8000 bytes from the start of physical RAM, can differ. This relocation address is encoded into the uImage header by the mkimage command during kernel build. However, the process fails if multiple relocation addresses exist, making the uImage format incompatible with multi-platform images. Nonetheless, a uImage binary can still be created from a multi-platform build if the correct LOADADDR of the specific SoC is provided. This load address can be found in the mach-\[your SoC]/Makefile.boot under the value of zreladdr-y.


### 4.2.2 Compiling device trees
### 4.2.3 Compiling modules

- If you have configured some features to be built as modules, you can build them separately using the modules target:
```
make -j 4 ARCH=arm CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf- modules
```
- But The compiled modules have a .ko suffix and are generated in the same directory as the source code, meaning that they are scattered all around the kernel source tree.
- Therefore, We need one more command to gather them by *modules_install* . The default location is /lib/modules in your development system, which is almost certainly not what you want. So, we need this option *INSTALL_MOD_PATH:

```
make -j4 ARCH=arm CROSS_COMPILE=arm-cortex_a8-linux-gnueabihf- INSTALL_MOD_PATH=$HOME/rootfs modules_install
```


## 4.3 Porting Linux to a new board 

### 4.3.1 With a device tree
- The first thing to do is create a device tree for the board and modify it to describe the additional or changed hardware on the board
- We can build nova.dtb explicitly:
```
make ARCH=arm nova.dtb
```
### 4.3.2 Without a device tree

# 5. Building a Root Filesystem

## 5.1 What should be in the root filesystem?

- [!] In order to transition from kernel initialization to user space, the kernel has to mount a root filesystem and execute a program in that root filesystem. This can be via a ramdisk or by mounting a real filesystem on a block device. The code for all of this is in **init/main.c**, starting with the function **rest_init()** which creates the first thread with PID 1 and runs the code in **kernel_init()**

- To make a useful system, you need these components as a minimum:
	- **init**: The program that starts everything off, usually by **running a series of scripts**.
	- **shell**: Needed to **give you a command prompt** but, more importantly, **to run the shell scripts called by init and other programs**.
	- **daemons**: Various **server programs**, started by init.
	- **libraries**: Usually, the programs mentioned so far are linked with **shared libraries** which **must be present** in the **root** **filesystem**
	- **Configuration files**: The **configuration for init and other daemons** is stored in a series of ASCII text files, usually **in** the **/etc** directory
	- **Device nodes**: The special files that **give access** to various **device drivers**
	- **/proc** and **/sys**: Two **pseudo filesystems** that represent kernel data structures as a hierarchy of directories and files. Many programs and library functions read these files.
	- **kernel modules**: If you have configured some parts of your kernel to be modules, they will be here, usually in /lib/modules/\[kernel version]

## 5.2 Programs for the root filesystem

### 5.2.1 The init program
- The first program to be run and so has PID 1
- It runs as the root user and so has maximum access to system resources.
- Usually, it runs shell scripts which start daemons.
### 5.2.2 Shell
- We need a shell to run scripts and to give us a command-line prompt so that we can interact with the system
- An interactive shell is probably not necessary in a production device, but it is useful for development, debugging, and maintenance.
### 5.2.3 Utilities
- To make a shell useful, you need the utility programs that the Unix command-line is based on.
- Even for a basic root filesystem, there are approximately 50 utilities, which presents two problems. 
	- Firstly, tracking down the source code for each and cross compiling it would be quite a big job. 
	- Secondly, the resulting collection of programs would take up several tens of megabytes, which was a real problem in the early days of embedded Linux when a few megabytes was all you had
	=> BusyBox
## 5.3 Device nodes
- Most devices in Linux are represented by device nodes, in accordance with the Unix philosophy that everything is a file (except network interfaces, which are sockets).
- The conventional location for device nodes is the directory **/dev**. For example, a serial port may be represented by the device node /dev/ttyS0.
- Device nodes are created using the program **mknod** (short for make node):
```
$ mknod <name> <type> <major> <minor>
```
Where:
	- name: name of this device
	- type : **c** < character device > or **b** < block device >
	- major and minor: There is a list of standard major and minor numbers in the kernel source in Documentation/devices.txt.
## 5.4 The proc and sysfs filesystems
- **proc** and **sysfs** are two pseudo filesystems that give a window onto the inner workings of the kernel.
- proc: Its original purpose was to expose information about processes to user space, hence the name.
- sysfs: exports a very ordered hierarchy of files relating to devices and the way they are connected to each other
### 5.4.1 Mounting filesystems
- The **mount** command allows us to attach one filesystem to a directory within another, forming a hierarchy of filesystems
- The format of the mount command is as follows:
```
$ mount [-t vfstype] [-o options] device directory
```
Ex:
```
$ mount -t ext4 /dev/mmcblk0p1 /mnt
```

- [i] Looking at the example of mounting the **proc** filesystem, there is something odd: there is no device node, /dev/proc, since it is a pseudo filesystem, not a real one. But the mount command requires a device as a parameter. Consequently we have to give a string where the device should go, but it does not matter much what that string is. These two commands achieve exactly the same result:
```
mount -t proc proc /proc 
mount -t proc nodevice /proc
```

## 5.5 Kernel modules

- If you have kernel modules, they need to be installed into the root filesystem, using the kernel **make modules_install** target.This will copy them into the directory **/lib/modules/** together with the configuration files needed by the **modprobe** command
- Be aware that you have just created a dependency between the kernel and the root filesystem. If you update one, you will have to update the other.
## 5.6 Transfering the root filesystem to the target

- **ramdisk**: a filesystem image that is loaded into RAM by the bootloader. Ramdisks are easy to create and have no dependencies on mass storage drivers.
- **disk image**: a copy of the root filesystem formatted and ready to be loaded onto a mass storage device on the target. For example, it could be an image in ext4 format ready to be copied onto an SD card, or it could be in jffs2 format ready to be loaded into flash memory via the bootloader.
- **network filesystem**: the staging directory can be exported to the network via an NFS server and mounted by the target at boot-time.













