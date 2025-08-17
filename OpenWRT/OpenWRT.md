# 1 **Build System** (Linux host):
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

# 2 **Menuconfig**

```bash
make menuconfig  #  Target/Subtarget, packages
make -j$(nproc)  # Build firmware
```
# 3 **Workflow**

```mermaid
graph TD
  A[Select hardware platform] --> B[Porting OpenWRT]
  B --> C[Custom kernel/config]
  C --> D[Build firmware image]
  D --> E[Flash Device]
  E --> F[Debug with serial/UART]
  F --> G[Develop new package]
```

# 4   **Porting OpenWRT**

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
 # 5 **Debugging**
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
# 6 Develop package

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


# 7 Communitiy
- **Forum**: [forum.openwrt.org](https://forum.openwrt.org/)
- **Mailing list**:
    - [openwrt-devel@lists.openwrt.org](https://mailto:openwrt-devel@lists.openwrt.org/) (Develop)
    - [openwrt-users@lists.openwrt.org](https://mailto:openwrt-users@lists.openwrt.org/) (user)
- **Real-time chat**: [IRC #openwrt trên Libera](https://web.libera.chat/#openwrt)




# 8 Introduction

![[Pasted image 20250817211001.png]]

![[Pasted image 20250817211108.png]]

- [Document][[https://openwrt.org/docs/start]]

![[Pasted image 20250817211616.png]]
![[Pasted image 20250817211559.png]]

![[Pasted image 20250817211735.png]]
![[Pasted image 20250817211854.png]]


# 9 Setup wireless
![[Pasted image 20250817212648.png]]


# 10 Setup PPPOE
![[Pasted image 20250817213013.png]]
![[Pasted image 20250817213131.png]]


# 11 Port forwarrding and Station IP on OpenWRT

 ![[Pasted image 20250817213350.png]]

![[Pasted image 20250817213531.png]]

# 12 Change OpenWRT's LAN IP address


![[Pasted image 20250817213711.png]]
![[Pasted image 20250817213739.png]]

















