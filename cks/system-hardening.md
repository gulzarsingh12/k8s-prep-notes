# Run command on ssh
- `ssh user@host 'command1' 'command2'`. you can pass multiple commands

# Reduce attack surface
- use POLP
- remove obsolete and unwanted packages
- remove unwanted kenrnel modules
- close unwanted open ports
- limit the access be removing root preveliges etc.

## limit node access
-  user private network
   - limit public network acces. use vpn
   - if vpn not possibnle use authorized networks (i.e. whitelist ips)
- limit users
  - developers dont need acces on prod but development env
  - also end users too dont need access to node
  - `id` `last` `who` commands. remove unwanted users from /etc/passwd, /etc/shadow, /etc/group etc. diable shell, add /bin/nologin option. `usermod -s /bin/nologin john`


## ssh hardening
- disbale root acess
- disbale password based auth

update `/etc/ssh/sshd_config` with `PermitRootLogin no' and `PasswordAuthentication no`

to first login with certificate use `ssh-keygen -t rsa` to generate key and then `ssh-copy-id john@node01` to copy the ssh public key to server.

## sudo
Add a user to sudo without enabling root login.

`visudo` to `/etc/sudoers` to add the user.

**Note** `useradd` didn't work for adding user for ssh. use `adduser` instead

## remove obsolete package
- check if any unwanted package?  like apache2.. remove it
- check if any unwanted service?  remove it 
````
systemctl list-units --type service
systemctl stop apache2
systemctl disable apache2
apt remove apache2
````

## remove kernel modules
- list all modes `lsmod`
- enable module by running `modeprobe pcspkr`
- disable module
  ````
  cat /etc/modeprobe.d/blacklist.conf
  blacklist sctp
  blacklist dccp
  ````
  disble module by adding entry as **blacklist**. can create any file with name ending in `.conf` under `/etc/modeprobe.d` dir.

## identify and disable ports
### identify
- check port description under  `/etc/services`
- or check port listening on `netstat -an | grep -W LISTEN`
- for k8s, check documentation to see which port should be open.

In lab, i see they disable/stop the service to remove the port from listening.

### disable
- you can use iptables to setup incoming and outgoing traffic
- you can also instal ufw firewall (uncomplicated firewall)

#### ufw
- `apt update && apt install ufw`
- `systemctl enable ufw && systemctl start ufw`
- `ufw status` should show status as **inactive**
- `ufw default allow outgoing` all outgoing allow by default
- `ufw default deny incoming` all incoming deny by default
- usecase is to only allow 22 and 80 from 172.16.238.5(jump/bastion) and 80 from 172.16.238.100/28 and deny all incoming for 8080
  - `ufw allow from 172.16.238.5 to any port 22 proto tcp`
  - `ufw allow from 172.16.238.5 to any port 80 proto tcp`
  - `ufw allow from 172.16.238.100/28 to any port 80 proto tcp`
  - `ufw deny 8080` although its not required as we deny all incoming
- to delete `ufw delete deny 8080` or `ufw delete 5` to delete with sequence no


# Syscalls
## Strace
`strace` is the utility provided by the os itself to trace sys calls. `which strace`

- to run on a command `strace touch /tmp/error.log`. but this is to run a command with strace itself
-  to run with a already running process. get pid of the process `pidof etcd` and then `strace -p 2345`, it will show all the sys calls that will be used now ondwards by the process.
-  to show all the sys calls made by a command `strace -c touch /tmp/error.log`

## tracee (from aqua security)
tracee is an open source tool and uses eBPF (extended berkley package filter) technology which runs in kernel space without interfering with kernel source code or loading any modules. 
you can install it or run as docker container.

if container, need to mount
-  `/tmp/tracee` default workspace
-  `/lib/modules` kernel headers
-  `/usr/src` kernel headers
need previleged flag to be true

### examples
- to see the syscalls for ls command `docker run --name tracee --rm --previleged --pid=host --v /tmp/tracee:/tmp/tracee --v /lib/modules/:/lib/modules/:ro --v /usr/src/:/usr/src/:ro aquasec/tracee:0.4.0 --trace com=ls`
- to see all the sys calls for a new process `docker run --name tracee --rm --previleged --pid=host --v /tmp/tracee:/tmp/tracee --v /lib/modules/:/lib/modules/:ro --v /usr/src/:/usr/src/:ro aquasec/tracee:0.4.0 --trace pid=new`
- to see syscalls for new containers `docker run --name tracee --rm --previleged --pid=host --v /tmp/tracee:/tmp/tracee --v /lib/modules/:/lib/modules/:ro --v /usr/src/:/usr/src/:ro aquasec/tracee:0.4.0 --trace container=new` ...  then run the container in separate window as `docker run ubuntu echo hi`

## seccomp
seccomp is available on almost all linux os.  To check if seccomp is supported

`grep -i /boot/config-$(uname -r)` will print `CONFIG_SECCOMP=y`

run `docker run -it --rm docker/whalesay /bin/sh` and see pid. it should be 1. check the seccomp for pid at `grep Seccomp /proc/1/status` will print `Seccomp: 2`, which means secomp profile is appled in filtered mode. this is because docker has built in secomp profile. it has blocked 60+ calls from 300+ including ptrace which is used in dirty cow attack.

There r 3 modes.  **0 - diabled, 1 - strict, 2 - filtered**

### format
````
{
   "defaultAction": "SCMP_ACT_ERRNO",
   "architectures": [
      "SCMP_ARCH_X86_64",
      "SCMP_ARCH_X86_64",
      "SCMP_ARCH_X86_64"
   ],
   "syscalls": {
      "names": [
         <syscall-1>,
         <syscall-1>,
         <syscall-1>
      ],
      "action": "SCMP_ACT_ALLOW"
   }
}
````

- in above, default action is errno, which means block all calls except in list which are allowed. this is like whitelist profile but this is hard to implement as we need to know all the calls made by application

- another way could be blacklist, wehere default action is allow and then list contains blocked syscalls. this is better for most of the apps but there is a risk as all calls are allowed by default.

docker secomp inbuilt profile nlocks 60+ calls including reboot, change time, create/delete modules, mount/umount etc.

We can create custom profile and use them. `docker run --it --rm --security-opt seccomp=/root/custom.json docker/whalesay /bin/sh`

we can also disable seccomp completely.  `docker run --it --rm --security-opt seccomp=unconfined docker/whalesay /bin/sh`

However, it will still not allow you to change date and time as docker still provide additional security gates even with seccmp diabled.


