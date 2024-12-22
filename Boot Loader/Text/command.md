

# 2. Compilation Command

## 2.1 Cross tool-chain installation and settings for linux host

- STEP 1 : Download arm cross toolchain for your Host machine
  - [Link croos toolchain](https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/): gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz

- STEP 2 :  Export  path of the cross compilation toolchain. (you have to include this path in the environmental variable called PATH.)
    <blockquote>
    <h5>Ex</h5>
    <ul>
    <li>nvim /home/truonglv/.bashrc</li>
    <li>export PATH=$PATH:/home/truonglv/Downloads/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin.</li>
    </ul>
    </blockquote>
- STEP 3: Run command : **source / home/truonglv/.bashrc**

- STEP 1: distclean : deletes all the previously compiled/generated object files.
    **make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean**
- STEP 2 : apply board default configuration for uboot

    **make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- am335x_evm_defconfig**


- STEP 3 : run menuconfig, if you want to do any settings other than default configuration .

    **make CROSS_COMPILE=arm-linux-gnueabihf-  menuconfig**


- STEP 4 : compile

    **make CROSS_COMPILE=arm-linux-gnueabihf- -j4**  // -j4(4 core machine) will instructs the make tool to spawn 4 threads
    **make CROSS_COMPILE=arm-linux-gnueabihf- -j8**  // -j8(8 core machine) will instructs the make tool to spawn 8 threads


## 2.2 U-boot Compilation
<blockquote>
<h5>sudo find . -name "am335x_evm_defconfig" => Check this package is avaiable or not. If not, we cannot complie </h5>

<h5> If you get any issues, run these commands below </h5>
<ul>
<li>  update </li>
sudo apt-get update
<li> Install bison: </li>
sudo apt-get install bison

<li> Install jq:</li>
sudo apt-get install jq

<li> Install flex: </li>
sudo apt-get install flex
</ul>
</blockquote>