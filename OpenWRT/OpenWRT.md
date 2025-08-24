
# Overview

## 1 How Packages are Compiled
- All packages are compiled by **OpenWrt's own toolchain**, which is also handled like packages (see /toolchain and /tools in the source), so you will always have the same compilers/tools as everyone else as they are downloaded from the same sources at the same version.
- When you run `make` the OpenWrt build system will use your system's existing building infrastructure to compile the OpenWrt's toolchain first, and then use that to compile the packages. This also has the major benefit of not requiring the user to set up a cross-compiling toolchain that is rather annoying and relatively complex.
## 2 Package Feeds
- Not all packages you can install in OpenWrt are from OpenWrt project proper, in fact most packages are not.
- The packages from OpenWrt's main repository are maintained directly by core developers, and they are the important, even essential components of OpenWrt firmware or parts of the build system. “**package feeds**” are source repositories that **contain additional packages** maintained by **the community**, and each package has its own maintainer.

# Adding a new device
## I. Important files

### 1. /target/linux/<arch_name>/base-files/etc/…

- **…board.d/** scripts for defining device-specific default hardware, like leds and network interfaces.
- **…hotplug.d/** scripts for defining device-specific actions to be done automatically on hotplugging of devices
- **…init.d/** scripts for defining device-specific actions to be done automatically on boot
- **…uci-defaults/** files for defining device-specific uci configuration defaults
- **…diag.sh** defines what is the led to use for error codes for each board
- ["] _Note that some of these functions are now done in the DTS for the board._

### 2. /target/linux/<arch_name>/base-files/lib/…
These are its subfolders and files:
- **…<arch_name>.sh** human-readable full board name associated to script-safe board name
- **…preinit/** common <arch_name> preinit startup scripts
- **…upgrade/** common <arch_name> upgrade scripts
### 3. /target/linux/<arch_name>/base-files/sbin
- This folder contains files and folders that will be integrated in the firmware’s /sbin folder, usually common <arch_name> sbin scripts and tools.
### 4. /target/linux/<arch_name>/dts/

- Device tree source files, or dts for short.
- Certain architectures have the DTS directory deeper down. ARM devices, for example, typically have it located at `files-X.yy/arch/arm/boot/dts/`_
### 5. /target/linux/<arch_name>/image/
- Configuration needed to build device-specific flashable images.
### 6. /target/linux/<arch_name>/<board_name>/
- Board-specific configuration.
### 7. /target/linux/<arch_name>/modules.mk
- Arch-specific kernel module config file for menuconfig

## II. Adding files

- After edit the files above, you need to touch the makefiles
```bash
touch target/linux/*/Makefile
```




# OpenWrt as a Docker Image
### 1. Purpose
- ["] Docker OpenWrt is **merely for rootfs/userland testing** on the host machine

-   What OpenWrt-in-Docker is
	- A **rootfs-only containerized environment**.
	- It does not use your board’s U-Boot or kernel.
	- It runs on your **PC’s kernel** inside Docker, not on target kernel.

- Running OpenWrt in **Docker** is mainly useful to:
	- Check and play with the **root filesystem** (packages, configs, LuCI web UI, scripts, etc.).
	- Test **userland applications** (like `opkg`, networking tools, daemons).
	- Quickly try OpenWrt behavior without flashing a board.
- But it does **not test**:
	- Your board-specific **kernel** (device drivers, hardware support).
	- **Bootloader (U-Boot)** or partition layout.
	- Any **SoC-specific features** (like AM335x peripherals on the BBB).


# Folder structure

- `tools` :  contains various utilities required for building toolchain and packages (e.g. autoconf automake), or for image generation (e.g. mkimage, squashfs)
- `toolchain` : refers to the compiler, the c library, and common tools which will be used to build the firmware image
	- `toolchain_build_<arch>` (mean to `build_dir/toolchain...` )which is a temporary directory used for building the toolchain for a specific architecture,
	- `staging_dir_<arch>` (mean to `staging_dir/toolchain...` )where the resulting toolchain is installed.
- `build_dir` :  is used **to** **unpack** all the **source archives** and **to compile them in**.
	- `build_dir/host`, **for compiling** all the tools that run on the host computer
	- `build_dir/toolchain...` **for compiling** the cross-C compiler and C standard library components that will be used to build the packages.
	-  `build_dir/target...` **for compiling** the actual packages, and the Linux kernel, for the target system
- `staging_dir` is used to “install” all the compiled programs into, ready either for use in building further packages, or for preparing the firmware image.
	- `staging_dir/toolchain...` is a mini Linux root with its own `bin/`, `lib/`, etc that contains the cross C compiler used to build the rest of the firmware. **You can actually use that to compile simple C programs outside of OpenWrt that can be loaded onto the firmware.**
	- `staging_dir/target.../root-...` contains “installed” versions of each target package again arranged with `bin/`, `lib/`, this will become the actual root directory that with some tweaking will get zipped up into the firmware image, something like `root-ar71xx`
- `target` - refers to the embedded platform, this contains items which are specific to a specific embedded platform.
- `package` - is for exactly that - packages. In an OpenWrt firmware, almost everything is an ipk,
- `dl` - anything downloaded by the toolchain, target or package steps will be placed in this directory






















# Building OpenWrt ON OpenWrt
- ["] **you can build OpenWrt directly on a system that is already running OpenWrt** (the _target system_).

# Cross Compiling your application
- ["] Download source code and compile it with the toolchain generated by OpenWRT

Step 1: Add that directory to the `PATH` environment variable:
    - PATH=$PATH:(your toolchain/bin directory here)
    - export PATH
Step 2: - Set the `STAGING_DIR` environment variable to the above toolchain dir and export it:
    - STAGING_DIR=(your toolchain directory here)
    - export STAGING_DIR
Step 3: Pass the _host_ and _build_ to the build system of the package to trigger cross-compile
	- For GNU configure :  ./configure  --build= ..   --host=.. 
	- For GNU make:override the `CC` and `LD` environment variables (usually not needed if GNU configure was used)

Step 4: The compiled program will be somewhere inside the folder your run ./configure and make from, try doing `find -iname *program*`
Step 5:  - If compilation aborts due to missing header files or shared objects, you might need to override `CFLAGS` and `LDFLAGS` to point to the `staging_dir/target-_architecture__uClibc-_libcver_/usr/include` and `.../usr/lib` directories

Step 5: When compilation is finished, copy the resulting binary and related libraries to the target device to test it. It might be necessary to set `LD_LIBRARY_PATH` on the target to point the binary to libraries in non-standard locations.

 


# External toolchain
- ["]  Use OpenWrt as external toolchain
Step 1: Build toolchain
- You have to select `Package the OpenWrt-based Toolchain` in the main view of `make menuconfig`
Step 2: Build Firmware
- Use the utility script to setup your buildroot
```bash
./scripts/ext-toolchain.sh \
  --toolchain /path/to/your/toolchain.tar.xz \
  --config target/subtarget   (example ipq806x/generic)

```
- Use your buildroot like a normal one. 
Step 3 (Optional) Overwrite an existing config
- If you already have a `.config` file but want to switch it to the external toolchain, add:
```bash
./scripts/ext-toolchain.sh \
  --toolchain /path/to/your/toolchain.tar.xz \
  --overwrite-config \
  --config target/subtarget   (example ipq806x/generic)
```

Step 4: Edit your config with `make menuconfig` and run `make` to compile your image








# Using Image builder

- ["] The Image Builder (previously called the Image Generator) is a pre-compiled environment suitable for creating custom images without the need for compiling them from source. It downloads pre-compiled packages and integrates them in a single flashable image.

- Doing so is useful if:
	- you want to fit more packages in a small flash size
	- you want to follow development snapshots
	- your device  has 32MB or less RAM and opkg does not work properly
	- you want to mass-flash dozens of devices and you need a specific firmware setup
- 🖥 **Prerequisites**
	- Runs on **64‑bit Linux** (can be in a VM on other OSes)
	- Install required build tools (package lists provided for Arch, Fedora, Debian/Ubuntu/Mint)
- 📥 **Getting Started**
	-  **Download** the Image Builder archive from the same page as your device’s firmware.
	-  **Unpack** it and `cd` into its directory.
- ⚙️ **Customization Options**
	- You can tweak your build using `make image` with variables such as:
		- **PROFILE** → target device profile
		- **PACKAGES** → add/remove packages (`pkg1 -pkg2`)
		- **FILES** → include custom config files (e.g., pre‑set network settings)
		- **DISABLED_SERVICES** → stop certain services from starting
		- **BIN_DIR / EXTRA_IMAGE_NAME** → control output folder & file naming
	- Dependencies are resolved automatically by `opkg`

- 🛠 **Extra Capabilities**
	- Add **custom packages** or private repositories
	- Pre‑configure security (e.g., disable root password login)
	- Create or modify profiles
	- Remove unnecessary files from the firmware (advanced use)
	- Build a fully self‑contained Image Builder with all packages
# OpenWrt Feeds

- ["] In OpenWrt, a “feed” is a collection of [packages](https://openwrt.org/docs/guide-developer/packages "docs:guide-developer:packages") which share a common location. Feeds may reside on a remote server, in a version control system, on the local filesystem, or in any other location addressable by a single name (path/URL) over a protocol with a supported feed method.

- Feeds are **additional predefined package build recipes** for OpenWrt Buildroot.
- Each feed points to where those packages live — this could be a **remote Git repository**, a local folder, or another supported source.
- The feed contains the **instructions (Makefiles)** for building packages, not the compiled packages themselves. ( Here, “building” means the **process** of taking source code and package metadata (like an OpenWrt Makefile) and **turning it into a finished, installable binary file**.)
- OpenWrt uses a config file (`feeds.conf` or `feeds.conf.default`) to list which feeds to use and where to fetch them from.
- With the `scripts/feeds` tool, you **update** a feed (download its package definitions) and then **install** packages from it into your build environment.
## 1. Feed Configuration
(`feeds.conf` or `feeds.conf.default`
- The Makefile in a feed is _just a recipe_:
	- It describes **where to download the source code** of a package.
	- Lists **dependencies** needed for the package.
	- Defines the **steps/commands** to compile and package it.

- The methods below will be used in the feed configuration file

| Method       | Function                                                                                                                  |
| ------------ | ------------------------------------------------------------------------------------------------------------------------- |
| src-bzr      | Data is downloaded from the source path/URL using `bzr`                                                                   |
| src-cpy      | Data is copied from the source path. The path can be specified as either relative to OpenWrt repository root or absolute. |
| src-darcs    | Data is downloaded from the source path/URL using `darcs`                                                                 |
| src-git      | Data is downloaded from the source path/URL using `git` as a shallow (depth of 1) clone                                   |
| src-git-full | Data is downloaded from the source path/URL using `git` as a full clone                                                   |
| src-gitsvn   | Bidirectional operation between a Subversion repository and git                                                           |
| src-hg       | Data is downloaded from the source path/URL using `hg`                                                                    |
| src-link     | A symlink to the source path is created. The path must be absolute.                                                       |
| src-svn      | Data is downloaded from the source path/URL using `svn`                                                                   |
## 2. Working with Feeds
- feeds are retrieved and managed by the script `scripts/feeds` within the `openwrt/openwrt.git` repository.
- During the `update` step, the feeds are obtained from their sources listed within a [feed configuration](https://openwrt.org/docs/guide-developer/feeds#feed_configuration "docs:guide-developer:feeds") and then copied into the `feeds` directory.
- After a successful update, each package recipe within a feed is represented within a folder in `feeds/<feed_name>/<package_name>/`, but they are not currently incorporated into the OpenWrt build system as they are not contained within the `package/` directory.
- During the `install` step, packages from the feeds obtained during an `update` are then linked into the `package/feeds/<feed_name>` folder, where `<feed_name>` is replaced by the name of the feed in the feed configuration.

-  After this step, it is then possible to perform package specific make operations by utilizing their path within the `package/` folder. For example:
```bash
make package/feeds/<feed_name>/<package_name>
```

## 3. Feed Commands
### 3.1 Feed clean
The clean command removes the locally stored feed data, including the feed indexes and data for all packages in the feed (but not the symlinks created by the install command, which will be dangling until the feeds are re-downloaded by the update command). This is done by removing the `feeds` directory and all subdirectories.
### 3.2 Feed install
The install command installs the applicable packages and any packages on which the applicable packages depend (both direct dependencies and build dependencies). The installation process consists of creating a symbolic link from `package/feeds/$feed_name/$package_name` to `feeds/$feed_name/$package_name` so that the package will be included in the configuration process when the directory hierarchy under `packages` is searched
 
### 3.3 Feed List

The list command reads and displays the list of packages in each feed from the index file for the applicable feeds. The index file is stored in the `feeds` directory with the name of the feed suffixed with `.index`. The file is generated by the update command.

### 3.4 Search

The search command reads through the feed metadata and lists packages which match the given search criteria.

### Uninstall

The uninstall command does the opposite of the install command (although it does not address dependent packages in any way). It simply removes any symlinks to the package from the subdirectories of `package/feeds`.

### Update

When `scripts/feeds update` is invoked, each of the applicable feeds are downloaded from their source location into a subdirectory of `feeds` with the feed name. It then parses the package information from the feed into an index file used by the list and search commands.

|Command|Description|
|---|---|
|./scripts/feeds update packages luci|Checkout the packages and luci feeds|

Note that update also stores the configured location of the feed in `feeds/$feed_name.tmp/location` such that changes to the configuration can be detected and handled appropriately.

After retrieval the downloaded packages need to be _“installed”_. Only after installation will they be available in the configuration interface!


## Custom feeds
- ["] Ok, you've developed your package, and now you **want to use it via make menuconfig**, OR you are developing a package and you **want to test it in a build** before you try to get it included in OpenWrt.








# Commands

## 1 Build commands


```bash
cd openwrt; git worktree add ../openwrt-23.05 -b my_openwrt-23.05; cd ../openwrt-23.05
```

```bash
sudo docker build --rm -t openwrt_bbb:22_08_25 --file ./docker/Dockerfile  ./openwrt/openwrt-23.05/
```

```bash
sudo docker run --interactive --rm --tty --ulimit 'nofile=1024:262144' --volume "$(pwd)/openwrt/openwrt-23.05:/workdir" --workdir '/workdir' openwrt_bbb:22_08_25 /bin/bash
```

```bash
sudo docker tag openwrt_bbb:22_08_25 truonglv95/bbb:22_08_25
```


```bash
sudo docker push truonglv95/bbb:22_08_25
```

- Docker compose file.yaml
```yaml
services:
  openwrt_bbb:
    image: openwrt_bbb:22_08_25
    stdin_open: true
    tty: true
    restart: "no"
    ulimits:
      nofile:
        soft: 1024
        hard: 262144
    volumes:
      - ./openwrt/openwrt-23.05:/workdir
    working_dir: /workdir
    command: /bin/bash

```


## 2 compile commands

### 2.1 Update the feeds

```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

### 2.2 Build the firmware image and packages:
```bash
alias make='make -j10 V=sc'
make defconfig download clean world
```


## 3 Run
```bash
sudo umount /dev/sdc1 /dev/sdc2   # make sure partitions are not mounted
```

```bash
sudo dd if=openwrt-omap-generic-ti_am335x-bone-black-squashfs-sdcard.img of=/dev/sdc bs=4M status=progress conv=fsync
```

```bash
sync
```

```bash
sudo eject /dev/sdc
```


## 4 Peek inside your `.img` file

### 1 Decompress the image (if not already)

```bash
gunzip openwrt-omap-generic-ti_am335x-bone-black-squashfs-sdcard.img.gz
```

### 2 Check partition table inside the image
```bash
fdisk -l openwrt-omap-generic-ti_am335x-bone-black-squashfs-sdcard.img
```
### 3 Mount partitions (without flashing!)
```bash
sudo losetup -Pf --show openwrt-omap-generic-ti_am335x-bone-black-squashfs-sdcard.img
```

- It will print something like:
```bash
/dev/loop0
```
- Now partitions are accessible as:
	- `/dev/loop7p1` → Boot
	- `/dev/loop7p2` → Rootfs

### 4. Inspect contents
- Mount boot partition

```bash
mkdir -p /tmp/boot
sudo mount /dev/loop7p1 /tmp/boot
ls -l /tmp/boot
```
### 5. Cleanup
```bash
sudo umount /tmp/boot
sudo losetup -d /dev/loop7
```
### 6 Copy in your new U-Boot files
```bash
sudo cp path/to/u-boot-am335x_evm/MLO /tmp/boot/
sudo cp path/to/u-boot-am335x_evm/u-boot.img /tmp/boot/
sync
```

- [!] Always copy `MLO` first, because TI ROM bootloader looks for the first valid MLO in FAT
- [!] Ensure filenames match exactly (`MLO` in all caps, `u-boot.img` lowercase). 

- End then cleanup:
```bash
sudo umount /tmp/boot
sudo losetup -d /dev/loop0
```





# Adding a new package

```bash
export PATH=/workdir/staging_dir/toolchain-arm_cortex-a8+vfpv3_gcc-12.3.0_musl_eabi/bin:$PATH
```

```
arm-openwrt-linux-muslgnueabi-gcc main.c -o main
```


# Addition
## 1 **Build System** (Linux host):
```bash
# Install dependencies (Ubuntu)
sudo apt install build-essential ccache ecj fastjar file g++ gawk \
gettext git java-propose-classpath libelf-dev libncurses5-dev \
libncursesw5-dev libssl-dev python python2.7-dev python3 unzip wget \
python3-distutils python3-setuptools rsync subversion swig time \
xsltproc zlib1g-dev

# Clone source
git clone https://git.openwrt.org/openwrt/openwrt.git
cd openwrt
./scripts/feeds update -a
./scripts/feeds install -a
```

## 2 **Menuconfig**

```bash
make menuconfig  #  Target/Subtarget, packages
make -j$(nproc)  # Build firmware
```
## 3 **Workflow**

```mermaid
graph TD
  A[Select hardware platform] --> B[Porting OpenWRT]
  B --> C[Custom kernel/config]
  C --> D[Build firmware image]
  D --> E[Flash Device]
  E --> F[Debug with serial/UART]
  F --> G[Develop new package]
```

## 4   **Porting OpenWRT**

- Identify SoC (Broadcom, MediaTek, Qualcomm)
- Find UART pinout (console debug)
- Analyze flash layout (MTD partitions)
- Write Device Tree (DTS) if needed
- Create profile in target/linux/<arch\>/


-  **device definition**
```makefile
#target/linux/ath79/image/Makefile
define Device/my_router
  DEVICE_VENDOR := MyBrand
  DEVICE_MODEL := RouterX
  DEVICE_PACKAGES := kmod-usb2 kmod-ath9k
  SUPPORTED_DEVICES := routerx
  IMAGE_SIZE := 16000k
endef
TARGET_DEVICES += my_router
```
 ## 5 **Debugging**
- **Tools*:
	- Serial console (picocom/minicom)
	- `logread` + `dmesg`
	- `strace` for process
	- `tcpdump` for network
	- Kernel crash dump (kdump)
- **Patch kernel**:
```bash
# Create patch from source
quilt new 0001-my-driver.patch
quilt add drivers/mydriver.c
# Modify code → quilt refresh		
```
## 6 Develop package

```txt
mypackage/
├── Makefile        # Build rules
├── src/            # Source code
│   └── main.c
└── files/          # Config files
    └── myapp.conf
```

- Makefile template:
```makefile
include $(TOPDIR)/rules.mk
PKG_NAME:=mypackage
PKG_VERSION:=1.0
include $(INCLUDE_DIR)/package.mk

define Package/mypackage
  SECTION:=utils
  CATEGORY:=Utilities
  TITLE:=My Custom Package
  DEPENDS:=+libopenssl
endef

define Package/mypackage/install
  $(INSTALL_DIR) $(1)/usr/bin
  $(INSTALL_BIN) $(PKG_BUILD_DIR)/myapp $(1)/usr/bin/
endef

$(eval $(call BuildPackage,mypackage))
```


## 7 Communitiy
- **Forum**: [forum.openwrt.org](https://forum.openwrt.org/)
- **Mailing list**:
    - [openwrt-devel@lists.openwrt.org](https://mailto:openwrt-devel@lists.openwrt.org/) (Develop)
    - [openwrt-users@lists.openwrt.org](https://mailto:openwrt-users@lists.openwrt.org/) (user)
- **Real-time chat**: [IRC #openwrt trên Libera](https://web.libera.chat/#openwrt)




## 8 Introduction

![[Pasted image 20250817211001.png]]

![[Pasted image 20250817211108.png]]

- [Document][[https://openwrt.org/docs/start]]

![[Pasted image 20250817211616.png]]
![[Pasted image 20250817211559.png]]

![[Pasted image 20250817211735.png]]
![[Pasted image 20250817211854.png]]


## 9 Setup wireless
![[Pasted image 20250817212648.png]]


## 10 Setup PPPOE
![[Pasted image 20250817213013.png]]
![[Pasted image 20250817213131.png]]


## 11 Port forwarrding and Station IP on OpenWRT

 ![[Pasted image 20250817213350.png]]

![[Pasted image 20250817213531.png]]

## 12 Change OpenWRT's LAN IP address


![[Pasted image 20250817213711.png]]
![[Pasted image 20250817213739.png]]

















