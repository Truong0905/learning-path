
# 1 Local.Conf

- Select your machine
- Decide where you will save download files (DL_DIR ), bin files ( SSTATE_DIR) and  scripts such as do_fecth, do_complile,... (TMP )
- Decide which packages you want to use ( rmp, debian, ...)
- And other fearture related to build, setup, .... 
# 2. bblayers.conf

* bblayers.conf is a configuration file used by the build systems
* In this file the set of layers are defined that should be included in a build.
* The bblayers.conf file specifies the location of each layer on the local file system.
- Each layer is represented by a meta-xxx folder
- In meta-xxx folder, we have recipes which are:
	-  A set of instructions for building packages
	* A recipe describes where you get source code
	* Which patches to apply
	* How to configure the source
	* How to compile it and so on

# 3. Add packages ( git, python, ...)

- In Local.Conf file, we can add thhe packages which we want to use in the board
- We must first search for this package name and determine which layer contains it. If this layer is not added yet, we need to add this layer first, and then we will be able to add this package in Local.Conf.
- Each package have a recipe to build 
# 4 Create layers

- We can create our own layer or download them from:
	 **BeagleBoard.org:** https://www.beagleboard.org/
	 **OpenEmbedded Layer Index:** https://layers.openembedded.org/
	 **GitHub :** [Repository search results · GitHub](https://github.com/search?q=BeagleBone+Black+layer&form=MG0AV3&type=repositories)
	 **Yocto Project Wiki:** https://wiki.yoctoproject.org/


# 5. Basic variables

![[Pasted image 20250212224411.png]]

# 6 Variable Assignment

- **These variables are used in file.bb**

# 7. Yocto build task

- There are some common build tasks:
![[Pasted image 20250213205326.png]]


## 7.1 do_fetch()


![[Pasted image 20250213210852.png]]

- Don't need mention do_fetch when we already add **SRC_URI** and **SRCREV** variables

## 7.2 do_unpack()


![[Pasted image 20250213211507.png]]

- The source folder here is S = "$(WORKDIR)/git" . Therefore, when we unpack , the sorce code will be in **git** folder 

## 7.3 do_patch()


![[Pasted image 20250213212632.png]]


## 7.4 do_configure()

## 7.5 do_compile()

![[Pasted image 20250217210508.png]]

![[Pasted image 20250217210543.png]]

![[Pasted image 20250217210629.png]]

## 7.6 do_install()


![[Pasted image 20250217210931.png]]



- [!] Note : bindir = /usr/bin

![[Pasted image 20250217211228.png]]

# 8 Apply patch

![[Pasted image 20250217211323.png]]



![[Pasted image 20250217211903.png]]

```bash
sudo apt install screen
```


- We add git for the folder that we want to change. and then we create a patch for it. 

# 9 How to Add Runtime Dependencies

- If we have a recipe A that build a package B. And package B depends on package C to run. So we need to specify C is in RDEPENDS variable

![[Pasted image 20250217215416.png]]


- We have an issue when removing RDEPENDS for this recipe
![[Pasted image 20250217215715.png]]

# 10. Demystifying **RPROVIDES**
- RPROVIDES provide a alias for a recipe. 
- We can see here. The error is that there is no RPROVIDES "provides" recipe. So we need to add it the recipe ( line 12)
![[Pasted image 20250217220807.png]]
![[Pasted image 20250217220419.png]]

- We can also use this alias in others recipe
- ![[Pasted image 20250217220750.png]]

# 11 Managing Package Conflicts with RCONFLICTS


![[Pasted image 20250217221546.png]]

![[Pasted image 20250217221700.png]]





- We can give a rule for the RCONFLICTS. (Here is the version >= 1.2 will be conflict)
- ![[Pasted image 20250217221838.png]]

# 12 Exploring Build Dependencies

![[Pasted image 20250217224134.png]]


![[Pasted image 20250217224217.png]]


![[Pasted image 20250217224411.png]]
![[Pasted image 20250217224530.png]]


- [!] The "add" recipe will not be installed in this case ( except that it is also add in local.cfg file)

- We can use PREFERRED_PROVIDER_xxx to preferre the recipe. ( here we have 2 lib libmath in **add** and **subtract** recipes. When using this one, the **subtract** will be preferred ) 
![[Pasted image 20250217232202.png]]


# 13 Mastering Package Version Selection with PREFERRED_VERSION


![[Pasted image 20250217234104.png]]

- We can also use wildcard 
![[Pasted image 20250217234404.png]]



# 14 Comprehensive Guide to use oe_runmake

![[Pasted image 20250217235956.png]]
https://www.youtube.com/watch?v=_7nQHSvqtbI&list=PLwqS94HTEwpQmgL1UsSwNk_2tQdzq3eVJ&index=31&ab_channel=Tech-A-Byte
![[Pasted image 20250219205459.png]]
![[Pasted image 20250219204811.png]]

![[Pasted image 20250219204844.png]]

![[Pasted image 20250219205012.png]]

![[Pasted image 20250219205302.png]]







# 15 Customizing Builds with EXTRA_OEMAKE

![[Pasted image 20250219205733.png]]


![[Pasted image 20250219210245.png]]


![[Pasted image 20250219210439.png]]

![[Pasted image 20250219210631.png]]

![[Pasted image 20250219210827.png]]


![[Pasted image 20250219211023.png]]



# 16 BBMASK | Recipe Control for Debugging

![[Pasted image 20250219211958.png]]

![[Pasted image 20250219212155.png]]

#  17 BBappend File | Modifying Recipe using bbappend File

![[Pasted image 20250219212346.png]]


![[Pasted image 20250219213913.png]]

![[Pasted image 20250219214639.png]]


![[Pasted image 20250219214741.png]]

![[Pasted image 20250219214854.png]]


# 18 FILESEXTRAPATHS in bbappend | Extending Resources Path
![[Pasted image 20250219215055.png]]

![[Pasted image 20250219231501.png]]


![[Pasted image 20250219231806.png]]

# 19 Bitbake 'clean,' 'cleansstate' & 'cleanall' Tasks

![[Pasted image 20250220205351.png]]

![[Pasted image 20250220205414.png]]


![[Pasted image 20250220205754.png]]

![[Pasted image 20250220210648.png]]

- Do_clean will remove all things to go back to the do_fetch task stage.

- Sstate Cache contains bin files. If we don't have any changes and we just do do_clean, yocto will look into Sstate Cache first to find the bin files and do any othert tasks. Therefore, we have to do do_cleansstate to remove all bin files in this folder. And then yocto will do all steps again.
- Do_cleanall will clean all things ( download, source, sstate cache)

- **Let take an example**


![[Pasted image 20250220211314.png]]

 ( source folder)
![[Pasted image 20250220211352.png]]

- Do_clean
![[Pasted image 20250220211521.png]]

- Everything in source has been gone
![[Pasted image 20250220211617.png]]

- Build again ( this time will not do any fetch and compile  becasue the bin file has been in sstate folder)
![[Pasted image 20250220211744.png]]
![[Pasted image 20250220212112.png]]
- Becasue we don't do any task. So we don't have source folder here ( source of this recipe is git folder)
![[Pasted image 20250220212308.png]]
- Now, we're looking into sstate folder
![[Pasted image 20250220212547.png]]
- It's still here
![[Pasted image 20250220212607.png]]

- Now we are going to do do_cleansstate
![[Pasted image 20250220212736.png]]

- Now we will build again
![[Pasted image 20250220212914.png]]
- Now the build system cannot find the bin file in the sstate cache folder. Thereforem build system will do some tasks again
![[Pasted image 20250220212923.png]]

- Take a look  in the download folder
![[Pasted image 20250220213145.png]]

- Now let do cleanall
![[Pasted image 20250220213300.png]]

- Back to download folder, we don'y see this package. Everything now is removed ( source, sstate, download)
![[Pasted image 20250220213343.png]]
- Let's build again and now fetch also appears
![[Pasted image 20250220213552.png]]
![[Pasted image 20250220213706.png]]


# 20 Integrating SystemD | Understanding Init Manager

![[Pasted image 20250220214143.png]]

![[Pasted image 20250220223935.png]]
![[Pasted image 20250220225954.png]]

![[Pasted image 20250220225529.png]]

![[Pasted image 20250220225859.png]]

![[Pasted image 20250220230200.png]]

![[Pasted image 20250220230257.png]]

# 21 Kernel Configuration | Menuconfig

![[Pasted image 20250221001215.png]]

# 22 Kernel Configuration with defconfig | savedefconfig

![[Pasted image 20250221001422.png]]
![[Pasted image 20250221001529.png]]

![[Pasted image 20250221001836.png]]

![[Pasted image 20250221001927.png]]

![[Pasted image 20250221002109.png]]

![[Pasted image 20250221002229.png]]


- Copy this path 
![[Pasted image 20250221002324.png]]


- And then copy defconfig to the recipe folder
![[Pasted image 20250221002426.png]]
![[Pasted image 20250221002534.png]]
- Then clean sstate of kernel recipe
![[Pasted image 20250221002726.png]]


#  23 Kernel Configuration using Config Fragments | diffconfig

![[Pasted image 20250221205746.png]]

- We use Config Framents to only save the changes that we made
- Firstly, open menuconfig and add  your changes
![[Pasted image 20250221210526.png]]

- Secondly, we use **diffconfig** to create fragment.cfg file
![[Pasted image 20250221210814.png]]- Now, we have fragment.cfg here
![[Pasted image 20250221210909.png]]
- And then, we have to copy this file to our recipe folder
![[Pasted image 20250221210954.png]]

- Finally, update file.bbappend
![[Pasted image 20250221211036.png]]

- At this time, we can build with new kernel changes
![[Pasted image 20250221211144.png]]


# 24 Kernel Development | Out of Tree Kernel Module
- **It is not inside the kernel image**
- Refer this link to know how to build a kernel module
![[Pasted image 20250221212903.png]]
- And then , we have to build again
![[Pasted image 20250221212952.png]]

- In BBB, we will have this module
![[Pasted image 20250221213042.png]]
![[Pasted image 20250221213139.png]]
![[Pasted image 20250221213152.png]]


# 25  Kernel Development | Device Driver Integration in Mainline Kernel

![[Pasted image 20250224214406.png]]![[Pasted image 20250224214434.png]]

- Step 1: 
![[Pasted image 20250224215202.png]]

- Please check this video: [[https://www.youtube.com/watch?v=IavTmJY5atg&list=PLwqS94HTEwpQmgL1UsSwNk_2tQdzq3eVJ&index=49&ab_channel=Tech-A-Byte]]

# 26  Classes & bbclass File

![[Pasted image 20250224221727.png]]

![[Pasted image 20250224222723.png]]


- Step 1: Create a **classes** folder inside the meta layer
![[Pasted image 20250224222846.png]]

![[Pasted image 20250224222948.png]]

![[Pasted image 20250224223010.png]]

- Step : Add this .bbclass file to the recipe that you want to inherit from this class
![[Pasted image 20250224223235.png]]

- Step 
![[Pasted image 20250224223344.png]]

![[Pasted image 20250224223411.png]]
 - We can inherit a class into another class
 ![[Pasted image 20250224224144.png]]

- We can alo add class in local.conf to apply for all packages
![[Pasted image 20250224224516.png]]

# 27 Search a Recipe Locally | bitbake-layers | show-recipes



![[Pasted image 20250224225042.png]]
This way only searches in  bblayers.conf file
![[Pasted image 20250224225514.png]]


- We can use **find** Command
# 28 Devtool
![[Pasted image 20250224225941.png]]

What are these three things???
- **add** means to create a new recipe
![[Pasted image 20250224230555.png]]
![[Pasted image 20250224230612.png]]
![[Pasted image 20250224230704.png]]

- **modify** means to modify the source of the recipe
- **upgrade** means to upgrade an existing recipe


Example:
![[Pasted image 20250224230929.png]]
![[Pasted image 20250224231003.png]]

![[Pasted image 20250224231059.png]]
![[Pasted image 20250224231140.png]]

![[Pasted image 20250224231235.png]]

- We don't have anything here. Becasue the recipe does not know how to do with these tasks
![[Pasted image 20250224231352.png]]
- We need to update those
![[Pasted image 20250224231527.png]]
- Now we can build it
![[Pasted image 20250224231615.png]]
![[Pasted image 20250224231714.png]]
#  29 Create Recipe with Devtool

![[Pasted image 20250225000305.png]]

- Step 1: Creating a new recipe
![[Pasted image 20250225001238.png]]

- But we will get an erro becasue we forgot to specify the branch
![[Pasted image 20250225001355.png]]
- So, we need to add an extra argument.
![[Pasted image 20250225001551.png]]

![[Pasted image 20250225001655.png]]

![[Pasted image 20250225001734.png]]

- Step 2: Build this recipe
![[Pasted image 20250225002003.png]]

# 30 Testing With Devtool | deploy-target & undeploy-target Commands

![[Pasted image 20250225002323.png]]
- Chúng giúp bạn triển khai và gỡ bỏ các tệp đầu ra của công thức (recipe) trên máy mục tiêu (target machine) để kiểm tra.
	- `deploy-target`: Lệnh này cho phép bạn triển khai các tệp đầu ra của công thức lên máy mục tiêu. Điều này rất hữu ích khi bạn muốn kiểm tra các thay đổi của mình trực tiếp trên phần cứng mà không cần phải xây dựng lại toàn bộ hình ảnh (image).

	- `undeploy-target`: Lệnh này cho phép bạn gỡ bỏ các tệp đầu ra của công thức đã triển khai trên máy mục tiêu. Điều này giúp bạn duy trì môi trường kiểm tra sạch sẽ và tránh xung đột khi triển khai các thay đổi mới.
![[Pasted image 20250225003125.png]]

# 31 Export Recipe to Meta Layer | devtool finish command
- After deploying and testing the recipe in the target board, we will finish it
![[Pasted image 20250225003805.png]]

![[Pasted image 20250225003854.png]]

- But if you build with only devtool, you won't get the package in image. So we need to use bitbake to build again
![[Pasted image 20250225004028.png]]
![[Pasted image 20250225004157.png]]

![[Pasted image 20250225004307.png]]
![[Pasted image 20250225004402.png]]


![[Pasted image 20250225004448.png]]

# 32  Patch with Devtool Modify | Make Changes in Source

![[Pasted image 20250225205855.png]]
![[Pasted image 20250225210054.png]]
- Step 1:
![[Pasted image 20250225210226.png]]

![[Pasted image 20250225210358.png]]


Step 2: 
![[Pasted image 20250225210452.png]]

Step 3:
![[Pasted image 20250225210528.png]]
Step 4:
![[Pasted image 20250225210702.png]]

Step 5: Test in board
![[Pasted image 20250225210814.png]]
Step 6: un-deploy it
![[Pasted image 20250225210843.png]]
![[Pasted image 20250225210911.png]]

- Step 7:

This is the layer that we want to modify source
![[Pasted image 20250225211005.png]]
Now, we should create a new layer to containt changes

![[Pasted image 20250225211232.png]]
![[Pasted image 20250225211916.png]]
So, we can finish ( but we have to commit this change first)
![[Pasted image 20250225211515.png]]
![[Pasted image 20250225211410.png]]

- Now  we have the patch
![[Pasted image 20250225211753.png]]

# 33 Upgrading Recipes using devtool upgrade & devtool rename

![[Pasted image 20250225213054.png]]

![[Pasted image 20250225213307.png]]

![[Pasted image 20250225214611.png]]
![[Pasted image 20250225214812.png]]

![[Pasted image 20250225214919.png]]

![[Pasted image 20250225215122.png]]
![[Pasted image 20250225215142.png]]
![[Pasted image 20250225215208.png]]
![[Pasted image 20250225215301.png]]

![[Pasted image 20250225215342.png]]

![[Pasted image 20250225215432.png]]
![[Pasted image 20250225215515.png]]

# 34 Create a New User | useradd class | FILES Variable
![[Pasted image 20250225220056.png]]

![[Pasted image 20250225220126.png]]
![[Pasted image 20250225220521.png]]
![[Pasted image 20250225220655.png]]

![[Pasted image 20250225221413.png]]

![[Pasted image 20250225221443.png]]

![[Pasted image 20250225221501.png]]


# 35 Setup Root Password | extrausers class| debug-tweaks | Access Over SSH


- We need to remove **debug-tweaks** first
![[Pasted image 20250225225059.png]]
![[Pasted image 20250225225631.png]]


#  36 Python3 Packages
![[Pasted image 20250225225821.png]]
 - Method 1: From in existing Layers
 ![[Pasted image 20250225230045.png]]
![[Pasted image 20250225230050.png]]
- Method 2: From meta-python
- We have to clone this layer first
![[Pasted image 20250225230351.png]]
![[Pasted image 20250225230543.png]]

- Method 3: 
![[Pasted image 20250225231145.png]]
![[Pasted image 20250225231519.png]]

![[Pasted image 20250225231708.png]]


![[Pasted image 20250225231911.png]]

![[Pasted image 20250225231820.png]]


![[Pasted image 20250225231938.png]]
![[Pasted image 20250225232011.png]]

![[Pasted image 20250225232040.png]]

![[Pasted image 20250225232057.png]]
![[Pasted image 20250225232108.png]]
![[Pasted image 20250225232118.png]]


# 37. Creating Kernel Module Recipe with Devtool

![[Pasted image 20250227213034.png]]
- **These steps below  explain how to creating Kernel Module Recipe with Devtool**

![[Pasted image 20250227213611.png]]

![[Pasted image 20250227213948.png]]

![[Pasted image 20250227214021.png]]

![[Pasted image 20250227214059.png]]
![[Pasted image 20250227214152.png]]

![[Pasted image 20250227214208.png]]
![[Pasted image 20250227214437.png]]
![[Pasted image 20250227214455.png]]
![[Pasted image 20250227214506.png]]
![[Pasted image 20250227214641.png]]
![[Pasted image 20250227214717.png]]
![[Pasted image 20250227214745.png]]
![[Pasted image 20250227214800.png]]
![[Pasted image 20250227214918.png]]
# 38 Creating Recipe for CMake Based Build

https://www.youtube.com/watch?v=5ERPwxeeGNA&list=PLwqS94HTEwpQmgL1UsSwNk_2tQdzq3eVJ&index=68&ab_channel=Tech-A-Byte

