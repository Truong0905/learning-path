# Build folder Tree
uboot : am335x-boneblack.dtb ; boot.scr ; MLO ;  u-boot.img ; uImage
RFS: BuildRoot + Step 5 of build kernel


# 1 Build U-boot

**Source** : https://github.com/u-boot/u-boot/tree/v2021.07

- Step 1: Find package 
```
sudo find . -name "am335x_evm_defconfig"
```
- Step 2: Install packages that needed:
```
sudo apt-get install bison
sudo apt-get install jq
sudo apt-get install flex
```
- Step 3: distclean : deletes all the previously compiled/generated object files. 
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
```
- Step 4: apply board default configuration for uboot
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- am335x_evm_defconfig 
```
- Step 5: run menuconfig, if you want to do any settings other than default configuration .  
```
make CROSS_COMPILE=arm-linux-gnueabihf-  menuconfig
```
- Step 6:  compile
```
make CROSS_COMPILE=arm-linux-gnueabihf- -j4 
```

- [x] Result: MLO, u-boot.img
# 2 Build Kernel

**Source** : https://github.com/torvalds/linux/tree/v5.15
- Step 0: 
```
sudo apt-get install libgmp-dev
sudo apt-get install libmpc-dev
sudo apt-get install u-boot-tools
```

- Step 1: 
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
```
- Step 2: 
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- omap2plus_defconfig
```
- Step 2.1: 
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- prepare
```
- Step 3:
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```
- Step 4:
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4 modules
```
- Step 5: ( After building Buildroot)
```
make -j4 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=/home/truonglv/beagleBone/build/rfs modules_install
```
- [X] **After all , we will get :**
	+ **uImage** file at : linux/arch/arm/boot
	 + **dts** *folder*  at:  linux/arch/arm/boot 


# 3 BuildRoot

**Source** : https://github.com/buildroot/buildroot/tree/2023.02

- Step 1: Select the default configuration for the target:
```
make beaglebone_defconfig
```

- Step 2: Optional: modify the configuration
```
make menuconfig
```
- Step 3: Build:
```
make -j4
```
- Step 4: Clean
```
make clean
```

- [x] Result
```
./output/images/
+-- am335x-boneblack.dtb
+-- am335x-boneblack-wireless.dtb
+-- am335x-boneblue.dtb
+-- am335x-bonegreen.dtb
+-- am335x-bonegreen-wireless.dtb
+-- am335x-bone.dtb
+-- am335x-evm.dtb
+-- am335x-evmsk.dtb
+-- boot.vfat
+-- MLO
+-- rootfs.ext2
+-- rootfs.tar
+-- sdcard.img
+-- u-boot.img
+-- uEnv.txt
+-- zImage
```

# 4 Writing Boot Script

- Step 1: Writing uEnv.txt file
```
nvim uENV.txt
```

```
## U-Boot script

## Set environment variables
setenv loadaddr 0x82000000
setenv fdtaddr 0x88000000

## Load kernel and device tree
load mmc 0:1 ${loadaddr} uImage
load mmc 0:1 ${fdtaddr} am335x-boneblack.dtb

## Set boot arguments
setenv bootargs console=ttyO0,115200 root=/dev/mmcblk0p2 rootfstype=ext4 rw rootwait

## Boot the kernel
bootm ${loadaddr} - ${fdtaddr}
```

- Step 2: convert the text file to bootscript with mkimage
```
mkimage -A arm -T script -C none -n "U-Boot script" -d uENV.txt boot.scr
```

#  5. Setup ssh
On host PC, in folder build, at file :~/beagleBone/build/rfs/etc/ssh/sshd_config
![[Pasted image 20250101203916.png]]

# 6. Enable Usb 0 interface

- At step 3 of building kernel: Setup usb0 interface in BBB
```
  Device Drivers  ---> 
  [*] Network device support  --->
      USB Network Adapters  --->
      <M> Multi-purpose USB Networking Framework  
      <M>   CDC Ethernet support (smart devices such as cable modems) 
      <M>   Host for RNDIS and ActiveSync devices (EXPERIMENTAL)
      <M>   Simple USB Network Links (CDC Ethernet subset)
                       
 Device Drivers  --->   
 [*] USB support  --->  
     <M>   USB Gadget Support  --->
     <M>     Ethernet Gadget (with CDC Ethernet support)
     <M>     CDC Composite Device (Ethernet and ACM)
     <M>     Multifunction Composite Gadget (EXPERIMENTAL
            [*]       RNDIS + CDC Serial + Storage configuration
```


```
nvim S39module
```
-rwxrwxr-x

```
#!/bin/sh
#
# load the kernel modules at startup 
#

case "$1" in
  start)
        printf "Loading kernel modules : "
        /sbin/modprobe g_ether
        [ $? = 0 ] && echo "OK" || echo "FAIL"
        ;;
  stop)
        printf "Unloading kernel modules : "
        /sbin/rmmod g_ether
        [ $? = 0 ] && echo "OK" || echo "FAIL"
        ;;
  restart|reload)
        "$0" stop
        "$0" start
        ;;
  *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
esac

exit $?

```

- [!] Need to copy into folder : rfs/etc/init.d

- In file rfs/etc/network/interfaces
```
auto lo
iface lo inet loopback

auto usb0
iface usb0 inet static
	address 192.168.6.2
	netmask 255.255.255.0
	network 192.168.6.0
	gateway 192.168.6.1
```

- When BBB is already booted:
	- Host PC
```
ifconfig -a
```
		then: 
```
sudo ifconfig <interface> 192.168.6.1 up  
```
# 7. Add a new user
```
adduser truongbbb
passwd truongbbb

mkdir /home
mkdir /home/truongbbb
chown truongbbb:truongbbb /home/truongbbb
```
# 8 Display current directory 
```
vi ~/.bashrc
```

```
#!/bin/bash
PS1='\u@\h:\w\$ '
```

```
source ~/.bashrc
```

# 9. Share internet from VM to Beagble Bone Black
- Step 1: In BBB
```
vi /etc/resolv.conf
```

```text
nameserver 8.8.8.8
nameserver 8.8.4.4
```

```
sudo route add default gw 192.168.1.1 usb0
```

- Step 2: Host PC
```
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo iptables --table nat --append POSTROUTING --out-interface ens33 --jump MASQUERADE
sudo iptables --append FORWARD --in-interface enx665779c1f17f --jump ACCEPT
```
- [i] ens33 and enx9a4b26497585 were found by 
```
ifconfig -a
```
# 10. Share files ( NFS)

```
sudo apt install nfs-kernel-server
```

- Host PC:
	- Step 1:  sudo mkdir /srv/nfs/bbb
	- Step 2:  sudo nvim /etc/exports
		/srv/nfs/bbb 192.168.6.2( rw,sync,no_root_squash,no_subtree_check)
	 - Step 3: sudo exportfs -arv
	 - Step 4: sudo service nfs-kernel-server restart
	
- On BBB:
```
sudo mount 192.168.0.95:/srv/nfs/bbb /home/truongbbb/code
```

```
sudo umount /home/truongbbb/code
```


# 11. Setup sudo command

- Step 1: 
```
visudo /etc/sudoers 
```
- Step 2: allow all
# 12. Change the Permissions of  busybox

- Switch to the root user and set the setuid bit on the `busybox` executable:
```
chmod u+s /bin/busybox
```


# 13  Making SD boot default on BBB by erasing eMMC MBR

- Step 1
```
sudo dd if=/dev/mmcblk1 of=emmcboot.img bs=1M count=1
```
- Step 2:
```
sudo dd if=/dev/zero of=/dev/mmcblk1 bs=1M count=1
```
- How to recover:
```
sudo dd if=emmcboot.img of=/dev/mmcblk1 bs=1M count=1
```
