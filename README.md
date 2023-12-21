# linux-kernel
Modified version Kernel. The homework 1 for NYCU OS AUT2023

### Requirement
1. Change kernel suffix
2. Implement ```sys_hello``` ,```sys_revstr```

## Installation
### Before Installation
```
uname -mrs
Linux 5.19.12 x86_64
```

### Install Requirements
```
sudo apt update && sudo apt upgrade
sudo apt install gcc build-essential libncurses-dev bison flex libssl-dev libelf-dev dwarves -y 
```

### Clone Linux Kernel
```
wget -P ~/ https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.19.12.tar.xz
```

### Change Directory
```
tar -xvf ~/linux-5.19.12.tar.xz -C ~/
sudo reboot
cd linux-5.19.12
```

### Create system folder 
```
mkdir helloworld
```

### Implement ```sys_hello```
- Note: Implementation for ```sys_revstr``` is the same procedure.
```
gedit helloworld/helloworld.c
```
- Them code it
```
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/syscalls.h>

SYSCALL_DEFINE0(hello)
{
    printk(KERN_INFO "Hello, world!\n");
    printk(KERN_INFO "XXXXXXXXXX\n");
    return 0;
}
EXPORT_SYMBOL(__x64_sys_hello);
```
- Make makefile
```
gedit helloworld/Makefile
```
- Save and Exit
- add
```
obj-y := helloworld.o
```
- Save and Exit

### Edit Makefile
```
sudo gedit Makefile
```
- Set extraversion to "any_name_you_like" (For change suffix)
- Search core -y (should be second one)
- After block/ insert helloworld/
- Save and exit

### Edit Syscall.h
Because ```syscalls.h``` file contains the system call interfaces used for communication between the kernel and user-space programs.
```
gedit include/linux//syscalls.h
```
- Go to the bottom before endif and insert
```
asmlinkage long sys_hello(void);
```
- Save and exit

### Edit Syscall_64.tbl
- The syscall_64.tbl file defines the mapping between system call numbers and their corresponding functions in the kernel
```
gedit arch/x86/entry/syscalls/syscall_64.tbl
```
- Go to number 450th entry insert the 451th one
```
451    common    helloworld    sys_hello
```
- Save and exit

### Generate a kernel configuration
- The make localmodconfig command in the Linux kernel build process is used to generate a kernel configuration (.config file) tailored to the currently loaded or active kernel modules on the system.
```
sudo make localmodconfig
```
- All choose ```n```
```
gedit .config
```
- Remove any between "" on CONFIG_SYSTEM_TRUSTED_KEYS & CONFIG_SYSTEM_REVOCATON_KEYS

### Compile Linux Kernel (takes time)

- Check number of cpus
```
nproc
```
- Then compile kernal
- 4 is $(nproc)
```
sudo make -j4
sudo make modules_install -j4 # Install Linux Kernel Modules
sudo make install -j4 # Install the Linux Kernel
sudo update-grub # Update GRUB Config
```
- reboot into grud and select advanced options and this kernel

### After Installation
```
~$ uname -mrs
Linux 5.19.12-suffix x86_64
```
### Test system call(sys_hello)
```
gedit test.c
```
- Then code
```
#include <assert.h>
#include <unistd.h>
#include <sys/syscall.h>

/*
 * You must copy the __NR_hello marco from
 * <your-kernel-build-dir>/arch/x86/include/generated/uapi/asam/unistd_64.h
 * In this example, the value of __NR_hello is 548
 */
#define __NR_hello 548

int main(int argc, char *argv[]) {
    int ret = syscall(__NR_hello);
    assert(ret == 0);

    return 0;
}
```
- Then compile C file
```
gcc test.c
```
- Run a.out
```
./a.out
```
- Them check the information in kernal
```
dmesg
```
![image](https://github.com/CodeStone1125/CustomKernal/assets/72511296/f203b22a-e055-4ed8-81c7-00a41dc61204)

### System call
### ```sys_revstr```
```
#include <linux/kernel.h>
#include <linux/syscalls.h>
#include <linux/uaccess.h> // Required for copy_from_user

SYSCALL_DEFINE2(revstr, char *, str, int, n)

{
    char *s;
    int x;
    char t;
    long res;
    printk(KERN_INFO "The original string:%s!\n",str);
    s = kmalloc_array(n, sizeof(char), GFP_KERNEL);
    res = strncpy_from_user(s, str, n);
    if (res < 0) return -EFAULT;
    x = strnlen_user(str, n) - 1;
    for (int i=0; i<x/2; i++) {
        t = *(s + i);
        *(s + i) = *(s + x-1 - i);
        *(s + x-1 - i) = t;
    }
    res = copy_to_user(str, s, n);
    printk(KERN_INFO "The reverse string:%s!\n",s);
    if (res < 0) return -EFAULT;
    kfree(s);
    return 0;
}
EXPORT_SYMBOL(__x64_sys_revstr);
```
