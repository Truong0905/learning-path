# 1. Platform data

- [i] If we don't use device tree, we can use platform data instead
## 1. 1 Device init
```C
static int __init device_X_platform_init(void)
```
- Step 1: Register platform devices
```C
int platform_device_register(struct platform_device * data);
```


# 2 Device driver

# 3 Linux device model
- It's a collection of various data structures, and helper functions that provide a uniflying and hierachical view of all the buses, devices, drivers  present on this system. ( **/sysfs** )
- Sysfs exposes underlying bus, device, and driver details and theri relationships in the linux device model
- Components of device module:
	- Device  : **struct device**
	- Device driver: **struct device_driver**
	- Bus: **struct bus_type**
	- Kobject: **struct kobject**
	- Ksets: **struct kset**
	- Kobject type: **struct kobj_type**
- 
obj-m := $(MODULE_NAME).o

$(MODULE_NAME)-objs += a.o b.o c.o


# 4 How To Design a driver

## 4.1 In driver init function

### 4.1.1 Preparing structures
- We need a struct that contain at least 3 elements:
```C
struct Xdriver_private_data
{
    int total_devices;
    dev_t device_base_num;
    struct class *class;
    struct device *device;
};
```








