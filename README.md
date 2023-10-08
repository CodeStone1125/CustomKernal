# linux-kernel
Modified version Kernel. The homework 1 for NYCU OS AUT2023

### Requirement
1. Change kernel suffix
2.  Implement ```sys_hello``` ,```sys_revstr```

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
wget -P ~/ https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.19.12.tar.xz.
```

### Change Directory
```
tar -xvf ~/linux-5.19.12.tar.xz -C ~/
sudo reboot
cd linux-5.19.12
```

### Create system folder 
```
mkdir hello
mkdir revstr
```

### Implement ```sys_hello```
- Note: Implementation for ```sys_revstr``` is the same procedure.
```
gedit helloworld/helloworld.c
```
-Them code it
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

### Implement ```sys_revstr```
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
- Save and Exit
### Edit Config File
```
sudo nano .config
```
- Set CONFIG_SYSTEM_TRUSTED_KEYS to ""
- Set CONFIG_SYSTEM_REVOCATION_KEYS to ""

### Compile Linux Kernel (takes time)
```
sudo make -j$(nproc)
```
#### Secret Command (skip this)
```
sleep 5 && echo start > ~/start.txt && sudo make -j$(nproc) > ~/logs.txt 2> ~/errors.txt && echo success > ~/success.txt || echo fail > ~/fail.txt & disown
```
- Turns out we can use `screen` window manager instead of using the so-called Secret Command. `screen` can be installed using `sudo apt install screen`.

### Install Linux Kernel Modules
```
sudo make modules_install -j$(nproc)
```

### Install the Linux Kernel
```
sudo make install -j$(nproc)
```

### Update GRUB Config
```
sudo update-initramfs -c -k 5.19.0-rc3
sudo update-grub
```

### Reboot Device
```
sudo reboot
```

### After Installation
```
ajmi@burner:~$ uname -mrs
Linux 5.19.0-rc3 x86_64
```

## System Calls
### retint System Call (451)
Return number from a system call
#### retint.c
```c
#include <stdio.h>
#include <linux/kernel.h>
#include <sys/syscall.h>
#include <unistd.h>

int main()
{
    long int n = syscall(451);
    printf("System Call sys_retint Returned: %ld\n", n);
    return 0;
}
```
#### Output
```
System Call sys_retint Returned: 2022
```

### swpnum System Call (452)
Swap numbers using a system call
#### swpnum.c
```c
#include <stdio.h>
#include <linux/kernel.h>
#include <sys/syscall.h>
#include <unistd.h>

int main()
{
    int a = 1;
    int b = 2;
    printf("Before Swapping: a = %d, b = %d\n", a, b);
    syscall(452, &a, &b);
    printf("After Swapping: a = %d, b = %d\n", a, b);
    return 0;
}
```
#### Output
```
Before Swapping: a = 1, b = 2
After Swapping: a = 2, b = 1
```

### revstr System Call (453)
Reverse string using a system call
#### revstr.c
```c
#include <stdio.h>
#include <linux/kernel.h>
#include <sys/syscall.h>
#include <unistd.h>

int main()
{
    char s[10] = "abcdefg";
    printf("Before Reversing: %s\n", s);
    syscall(453, s, 10);
    printf("After Reversing: %s\n", s);
    return 0;
}
```
#### Output
```
Before Reversing: abcdefg
After Reversing: gfedcba
```

### cpyarr System Call (454)
Copy array using a system call
#### cpyarr.c
```c
#include <stdio.h>
#include <linux/kernel.h>
#include <sys/syscall.h>
#include <unistd.h>

int main()
{
    int a[7] = {1, 2, 3, 4, 5, 6, 7};
    int b[7];
    printf("Initial Array: ");
    for (int i=0; i<7; i++)
        printf("%d ", a[i]);
    printf("\n");
    syscall(454, a, b, 7);
    printf("Copied Array: ");
    for (int i=0; i<7; i++)
        printf("%d ", b[i]);
    printf("\n");
    return 0;
}
```
#### Output
```
Initial Array: 1 2 3 4 5 6 7 
Copied Array: 1 2 3 4 5 6 7 
```

### swpsrt System Call (455)
Swap members of structure using a system call
#### swpsrt.c
```c
#include <stdio.h>
#include <linux/kernel.h>
#include <sys/syscall.h>
#include <unistd.h>

struct swap_srt {
    int a;
    int b;
};

int main()
{
    struct swap_srt s;
    s.a = 1;
    s.b = 2;
    printf("Before Swapping: s.a = %d, s.b = %d\n", s.a, s.b);
    syscall(455, &s);
    printf("After Swapping: s.a = %d, s.b = %d\n", s.a, s.b);
    return 0;
}
```
#### Output
```
Before Swapping: s.a = 1, s.b = 2
After Swapping: s.a = 2, s.b = 1
```

### revsrt System Call (456)
Reverse array in structure using a system call
#### revsrt.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <linux/kernel.h>
#include <sys/syscall.h>
#include <unistd.h>

struct reverse_srt {
    int *arr;
    int n;
};

int main()
{
    struct reverse_srt s;
    s.n = 7;
    s.arr = malloc(s.n * sizeof(int));
    for (int i=0; i<s.n; i++)
        s.arr[i] = i+1;
    printf("Before Reversing: ");
    for (int i=0; i<7; i++)
        printf("%d ", s.arr[i]);
    printf("\n");
    syscall(456, &s);
    printf("After Reversing: ");
    for (int i=0; i<7; i++)
        printf("%d ", s.arr[i]);
    printf("\n");
    return 0;
}
```
#### Output
```
Before Reversing: 1 2 3 4 5 6 7 
After Reversing: 7 6 5 4 3 2 1 
```
