# Files

## Type of files

Everything in linux is a file. There are 3 types of files in linux.

* Regular file
* Directory
* Special files

Regular file is any file. it could be some text, script file or image or any binary file.
Directory stores other files and dirs.

### Special file

Below are type of special files
* character files
* block files

#### Character files
These files are the files under /dev which allow the devices to communicate serially. For example mouse, keyboard etc.

#### Block files
These files are the files under /dev too but it reads and write data to block device in blocks/chunks.for example hard disks.

#### Links
There are 2 types of links.i.e hard and soft

**hard** links means if you delete the link it will delete the actual file. 

**soft** links are like windows shortcuts which means deleting soft link will delete the reference to the file but actual file 
will not be deleted. soft links can also be called symlinks or symbolic links.

#### Socket
For bidrectional client/server style interprocess communication between two processes on the system. there are multiple benefits that you
dont need to setup any networking for this. multiple connections as client can be made to same socket. you can also call it as unix 
domain socket.

#### Named pipes
It is also used for inter-process communication. but It is unilateral and can be in one direction at a time only. it allow one process to 
pass input to another process. for reciving anything from the other end, we need to open have aother pipe. so for the same thing socket need only one sock file.

### Check type of file

#### file
To check the type of file, you can use `file` command.

`file /home/bob` will return it as directory

`file xyz.sock` as socket file

#### ls
by using `ls` command

it will return the 1st char as representation of the file.

- **d** - directory
- **-** - regular file
- **c** - character file
- **b** - block file
- **s** - socket file
- **p** - pipe
- **l** - link

## Filesystem hierarchy

````
                                   /
/bin /boot /dev /etc /home /lib /media /mnt /opt /tmp /usr /var /sbin /srv

       /usr
/usr/bin  /usr/lib
````

### /bin
This dir contains the binaries to be run by all users. This is accesible to all users. e.g. ls, cp, mkdir, mv etc.

### /dev
This dir contains special files for devices.character or block devices. e.g. hard disk, mouse, keyboard etc.

there are other important special files used in the system for various usecases. 

#### /dev/null
This is the null device to trash empty anything. Anything written to null device will disappear.

#### /dev/zero
This is the special file to generate zero. This is used in some cases where you need to fill the space ith empty file. 
`dd if=/dev/zero of=Filename bs=Block_Size count=sets_of_blocksize`

- of=Filename is where you can indicate the path and name of the file you want to create,
- bs=Block_size is where you will enter the size of each block such as 1MB.
- count=sets_of_blocksize is where you have to specify the count of the block size. such as if you want to create a file worth 1 gigabyte, you will use a 1Mb block and set the count to 1024.
  
`dd if=/dev/zero of=Dummy bs=1MB count=100` it will create a dummy file of 100MB. 

usecases could be 
- creating dummy file as above
- formatting disk with zeros.

#### /dev/random
This is to generate the random no for many usecases. it can used for cryptpgraphic purpose. it will use PRNG to get seed. which further depends on the entropy pool. There is a problem with `random` that it can block the process requesting random nos if there is not enough data in entry pool. This can happen more frequently on startup of the system as entropy pool gets filled eventually after lot of environmental noise. More the **noise**, more the pool will have data. Noise is created by triggering many events in the system. For example, user clicks on **keyboard**, **mouse**, something is opened on internet and generate network events. All these events are generated at random time and make it **unpredictable** to guess what happens in the system, hence more **high quality** data to generate random nos which are difficult to crack.

##### urandom
As **random** can be blocking depending on the how much entropy pool is filled, which can be empty or low on boot as there might not be enough noise. **urandom** is n on blocking. it never blocks even if entropy pool is low. However, it may provide low quality data for encryption and make it vunerable as it can be predictable. So to use **urandom** or **random** depends on the usecase.

where quality of the random number matters and you are generating keys for cryptographic purpose then use the **random** where process can't be blocked due to PRNG's entropy pool then it use **urandom**

### /etc
This dir contains the core configuration files for the system.e.g. password, network files etc.

### /usr
As per internet usr stands for unix system resources. but it container binaries, libraries and source of system programs.

- `/usr/bin` contains basic user commands
- `/usr/sbin` contains additional commands for the administrator
- `/usr/lib` contains the system libraries for the binaries in /usr/bin

e.g. thunderbird mail, firefox, vi editor etc are in /usr dir

### /home
this contains home dir of various users. In very old implementation, /usr directory was used for home directory of users as there was not enough space on device having (/ root partition) and saparate external device was mounted to /usr. But now this is not a problem.

### /lib
This contains the libraries used by binaries in /bin and /sbin directory. Simlarily /usr/lib is used for binaries in /usr/bin and /usr/sbin

### /sbin
It is similar to /bin except that binaries are only accible by sudo users. s stands for sudo in sbin.

### /tmp
To create any files temporarily when system is running. This dir is wiped on the startup

### /var
it is variable dir and is writable dir for any system operartions. For example /usr is considered read only and if anything to write by programs running in it, like logs, caches etc should be in /var.

### /boot
This is the dir containing boot loader files. boot loader programs and kernel images are stored here. It is recommended to be in the first sector of the hard disk.

### /proc
This contains the information about currently running processes. This is used by tools to get runtime information about the system. for example `/proc/cpuinfo` to get cpu information `/proc/meminfo` to get memory information. `/proc/{pid}/mem` can be used to get the memmory information about a process. we can various other informarion like diskstats, crypto etc.

### /opt
Any user created software should go here. Thats why its called optional.

Good practice is to keep the software in /opt and link the binaries in /bin so that all user can access it.

### /root
This is the root user's home directory. Root user home dir can't be /home/root but /root. Dont confuse it with / (root) dir.

### /media
Whenever a usb drive or external disk is connected, it is automatically mounted in /media dir. e.g. usb, sd, dvd etc.

### /mnt
This is same as /mnt but it is manually mount by administrators when required. For example, nfs drive is mounted from network to access data.

### /srv
This is for service data. For example if you run http webseervice then its good practice to store website data in /srv.

## Other useful info

- `df -hP` can be used to see all the devices mounted on the system include blk and external devices.
- It is also debatable (based on the internet articles I read), that `/bin` and `/lib` are no more required and can be moved to /usr/bin and /usr/lib respectively but it is tried and rolled back many times in many distors.
- /dev/random is better choice for formatting the disk then /dev/zero because it will make it difficult to recover the data.

   
