
# 1. Scripted Partitioning & Formatting
##  1.1 Requirements 
![[Pasted image 20241111211242.png]]

- **sudo apt install bash binutils bison bzip2 coreutils diffutils findutils gawk gcc glibc grep gzip m4 make patch perl python sed tar texinfo xz**
	=> If you cannot install any packges, pls search in google
##  1.2 Creating LFS

- lfs.sh

```bash
#!/bin/bash

#host PC
export LFS=/mnt/lfs

#targeted board
export LFS_TGT=arm-lfs-linux-gnueabihf

#disk ( uinng sudo dmesg command to find)
export LFS_DISK=/dev/sdb

if ! grep -q "$LFS" /proc/mounts;then
	source setupdisk.sh "$LSF_DISK"
	sudo mount "${LFS_DISK}2" "${LFS}"
	sudo chown -v $USER "${LFS}"
fi

mkdir -pv $LFS/sources
mkdir -pv $LFS/tools

mkdir -pv $LFS/boot
mkdir -pv $LFS/etc
mkdir -pv $LFS/bin
mkdir -pv $LFS/lib
mkdir -pv $LFS/sbin
mkdir -pv $LFS/usr
mkdir -pv $LFS/var

echo $LFS_TGT | grep -qs '64';then 
	mkdir -pv $LFS/lib64
```

- setupdisk.sh 
```bash

LFS_DISK="$1"


# Step 1: Create partition table
# the meaning of the following commnand 
# o: create a new partition table
# n : create a new partition of that disk
# p: create primary patition type
# 1: ( patition number) number 1
# : where this partition should be start => here, it will start at beginning
# +1024M : size of this partition
# j : ignore warning
# a : bootable
# n: create a new partition
# ... : repeat again
# w : write to disk
# q : close the program
sudo fdisk "$LFS_DISK" << EOF
o
n
p
1

+1024M
y
a
n
p
2


y
p
w
q
EOF
# Step 2: Create new file system
sudo mkfs -t fat16 -F "${LFS_DISK}1"
sudo mkfs -t ext3 -F "${LFS_DISK}2" 
```
- versioncheck.sh
```bash
```

- download.sh
```bash

```