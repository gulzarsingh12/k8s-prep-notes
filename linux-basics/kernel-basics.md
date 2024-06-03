# Kernel

This is the interface between the user applications and system resources.i.e. cpu, memory and other hardware devices.

Kernel does following:
- Memory management
- Process management
- Device drivers
- System calls and security

#### monolithic
This measns it manages all the functions itself like memory management, cpu scheduling and several operations.

#### modular
Kernel's functionality can be extended using dynamically loaded modules.

#### Version
To get the kernel version run `uname`

`uname -r'  returns `4.15.0.72-generic` 

`uname -a` return more information.

## Memory Management

Memory is dividied into 2 spaces. **Kernel** space and **User** space.
- A process is run with unrestriced access to hardware in **Kernel space**. Only kernel kode, kernel extensions and device drivers run in kernel space.
- A processes is run with limited access in user space to cpu and memory. User applications are run in user space and if need anything which is part of kernel space, it uses system calls. for example to `cat /etc/os-release`. This will result in `open` system call. other examples of system calls are `close` `getpid` `closedir` etc.

## Harware
To understand working with hardware, just take an example of usb drive. When usb drive is connected, device drivers code executes (Kernel Space) and detect a stage change and generate the event. it is called **uevents**. Then uevents event is send to **udev** service (User Space). **udev** will create the the device node for usb drive under /dev filesystem.i.e. **/dev/sdb1**. Once the process is complete, newly attached device is visible in **/dev** filesystem.

### dmesg
When linux os boots, kernel logs are written to ring buffer called dmesg. (Ring buffer is like something in-memory buffer for writing kernel log which overrides the oldest log like in a ring as it is constant in size.)

When os is booting, syslog dameon is not yet started till systemd init starts it. So it needs to write the logs somewhere till then. it is the memory buffer aka ring buffer aka **dmesg**. On starup the messages displayed on console/screen are stored in dmesg.

Dmesg logs are kernel logs till syslog daemon is started but it contains very important information about the hardware. Various device drivers logs can be checked to see any issues during startup with any hardware device.

`dmesg | grep -i usb` will provide the information about usd device.

### udevadm
it is management tool for **udev**. so its like udev admin. 

`udevadm info --query=path --name=/dev/sda5` To query the udev database for harddisk informartion

`udevadm monitor` prints the **uevents** from kernel and **udev** events. once udev process the kernel uevents then it will sends the udev event. You can run this command to see which device is connected or removed to the system. for example, try to run it and then remove and attach usb mouse to see the events.

### lspci
To list the information about all pci devices like ethernet controller, graphics card, wireless adapters etc. pci stands for peripheral component interconnect.

### lsblk
To list information the block devices. it will list the physical disks and their partitions. major number tells the device type. like 8 is scsi disk, 1 is ram, 3 is printer etc. 

### lscpu
To list the information about cpu. 
- architecture - tell you if 64bit or 32 bit. 32 bit can have max 3GB, 64 bit can have 18EB.
- cpus - total no of virtuals cpu core. it can be counter as threads*cores*sockets
- threads - the hypervised core/threads or logical processor
- the actual cores on the socket
- the physical socket on the chip.

To count the cpus in above. For example if there is 1 socket having 4 cores and each core has 2 threads. then it is calculated as 1*4*2=8 cpus.

### lsmem
To get the information about memory.
`free -m` can also be used to get the information about memory

### lshw
To get the information about entire harware on the system. it can report firmaware, cpu version, bus speed etc.

## Boot Process
BIOS POST -> Boot Loader (GRUB2) -> Kernel Initialization -> Init Process (Systemd)

### bios post
post means power on self test. When system is started/restarted. It will run post test which will ensure that harware is connected and working. if post fails, then system might not come up.

### boot loader
after succesful post, bios loads the boot loader and executes it. boot loader resides in first sector of the hard disk. this is located in **/boot** filesystem. In almost all linux these days, GRUB2 (Grand unified boot loader) is used.

Boot loader will provide the user with boot screen and provided the options to boot. In case of dual boot, it will provide the option for windows boot manager. once the selection is made. it will load the linux kernel code. Usually kernel code is compressed to save space. it will decompress the kernel first and execute the kernel. once kernel code starts executing, it will handover the control to kernel.

### kernel initialization
kernel will do tasks like initialize harware, do memory management etc.

### init process
once kernel is completely operational, it will look for **init** process. which sets up user space and processes needed for user environment. init function will call systemd daemon. then systemd is reposnsible to bring the system to usable state. it will mount the fielsystem. systemd is universal standard these days. systemd will reduce the startup time by parallelizing the startup of services. to check the init system used `ls -l /sbin/init`.it should return `/lib/systemd/systemd` for systemd init system.


## RunLevels/Target
The term runlevels is used in the sysV init systems. but with systemd init system, it is called targets.

To compare
 5 -> graphical.target  (required gui display manger service)
 3 -> multiuser.target  (started on console)


### To get target
to check runlevel  run `runlevel`

To check the target run `systemctl get-default`. it will show like `graphical.target`

How it works is, it will look at file `/etc/systemd/system/default.target`

`ls -l /etc/systemd/system/default.target`
if its graphical.target (runlevel 5), then it will show symlink to graphical.target.. `/etc/systemd/system/default.target -> /lib/systemd/system/graphical.target`

**Note**  while testing i found that there was no `/etc/systemd/system/default.target` file found but when i ran `systemctl set-default multi-user.target`. You should see the file in /etc now.

### To set target
`systemctl set-default multiuser.target`
 it will create sysmlink for default to multiuser.target. this is equivalent to switch runlevel from 5 to 3.
