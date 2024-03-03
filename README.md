# Kernel Module Development

## What is Kernel Module?
Kernel modules are codes that `can be added or removed from the kernel later`. Kernel modules developed in C are actually not much different from an ordinary program. We can see installed modules in kernel with `lsmod` command.

## Why Should Use Module?
Hardware is handled by kernel drivers in Linux. Generally, these modules locate under the `/lib/modules` directory and install when system booting. While these are ready-made modules in the system, we can also add modules ourselves. The functionality of the kernel can be increased with modules without rebooting the system. If we consider a hardware driver, the driver and the kernel will provide the connection between the system and the hardware. But, for this process we will need to add an extension to the kernel. Any tampering with the kernel requires recompiling the kernel and rebooting the system. Restarting is undesirable for computers that need to stay on all the time, such as server systems. Modules are used to prevent such situations.

## How to Install Kernel Module?
Module tools such as 

* **insmod**
* **modprobe**
* **rmmod**
* **lsmod**

are used for operations to be performed with kernel modules. 

### insmod and modprobe
`insmod` or `modprobe` is used to add a new module to the kernel. `Insmod installs a single module` in the kernel, while `modprobe automatically loads other modules dependent` on the relevant module.

Some modules depend on other modules being loaded, and if the dependent module is not loaded, the insmod command will fail. In this case, you must first install the dependent modules one by one or modprobe can be used.

```bash
$ insmod /home/murat/new_module.ko
```

The command will load the module at the location you specified.

```bash
$ modprobe new_module
```

In both cases, additional options can be `specified during installation`. It's like specifying the desktop resolution to the video card driver.

The module's path along with its file extension `must be specified in the insmod command`. However, there is no such situation in the modprobe command, just specifying the module name is sufficient.

### rmmod
The `rmmod` command is used to remove kernel modules.

```bash
$ rmmod my_module
```

The point to be considered here is that the module you want to delete can be used by other modules. If the module is not used, it will be deleted. If it is used, the modules it depends on are listed or it gives an error saying this module is being used. If you want to remove the module along with dependent modules, you should use the `modprobe -r` command.

## How Kernel Module Develop?
Unlike programs, modules do not have a *main()* function. Instead of the main() function there is a pair of functions `init_module` and `cleanup_module`:

The `init_module()` function is `called when the module is first loaded` into the kernel. When you want to `remove the module, the cleanup_module() function is called` to undo what init_module() did. If the module is written in a dynamic structure, new variables are registered during this run. However, if your module is in a static structure, the variables are registered during `boot`.

### Example

```c
#include <linux/module.h> // included for all kernel modules
#include <linux/kernel.h> // included for KERN_INFO
#include <linux/init.h> // included for __init and __exit macros

MODULE_AUTHOR("Murat CEZAN");
MODULE_DESCRIPTION("Example Hello World module");

static int __init hello_init(void)
{
    printk(KERN_INFO "Hello world!\n");
    return 0; // Non-zero return means that the module couldn't be loaded.
}

static void __exit hello_cleanup(void)
{
    printk(KERN_INFO "Cleaning up module.\n");
}

module_init(hello_init);
module_exit(hello_cleanup);
```

The `linux/module.h` header file is `used in all kernel modules`. The `linux/kernel.h` header file is `used to use printk()`. The printk() function is used to print to the screen at the kernel level. It acts like the printf() function in the C library, but it does not output directly to the screen at the kernel level. The printed values can be seen in the `/var/log/syslog` file or in the output of the dmesg command.

Code to be placed in the `Makefile` file:
```makefile
obj-m += new_module.o
all:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

Let's compile our module with the `make` command.

## How to Compile?
First of all, we place the two files we created in a directory. Then we compile our module with the `make` command. We can access information about the compiled module with the modinfo command.

After these operations, we place our module into the kernel. To do this, you must be a root user. We add our module to the kernel with the `sudo insmod new_module.ko` command.

```bash
$ make
$ sudo insmod my_modul.ko // içeri aktarma.
$ lsmod | grep my_modul //kontrol.
$ sudo rmmod my_modul // modulun silinmesi.
```
