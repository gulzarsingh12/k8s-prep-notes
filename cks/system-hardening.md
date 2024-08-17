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

update `/etc/ssh/sshd_config` with `PermitRootLogin no` and `PasswordAuthentication no`

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
- or check with `lsof -i :8888` to check process running on port.
- to get the pid of process use `pidof apache2` or `ps -ef | grep apache2`
- to find the path of execution
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

## Tracee (from aqua security)
tracee is an open source tool and uses eBPF (extended berkley package filter) technology which runs in kernel space without interfering with kernel source code or loading any modules. 
you can install it or run as docker container.

if container, need to mount
-  `/tmp/tracee` default workspace
-  `/lib/modules` kernel headers
-  `/usr/src` kernel headers
need previleged flag to be true

### Examples
- to see the syscalls for ls command `docker run --name tracee --rm --previleged --pid=host --v /tmp/tracee:/tmp/tracee --v /lib/modules/:/lib/modules/:ro --v /usr/src/:/usr/src/:ro aquasec/tracee:0.4.0 --trace com=ls`
- to see all the sys calls for a new process `docker run --name tracee --rm --previleged --pid=host --v /tmp/tracee:/tmp/tracee --v /lib/modules/:/lib/modules/:ro --v /usr/src/:/usr/src/:ro aquasec/tracee:0.4.0 --trace pid=new`
- to see syscalls for new containers `docker run --name tracee --rm --previleged --pid=host --v /tmp/tracee:/tmp/tracee --v /lib/modules/:/lib/modules/:ro --v /usr/src/:/usr/src/:ro aquasec/tracee:0.4.0 --trace container=new` ...  then run the container in separate window as `docker run ubuntu echo hi`

## Seccomp
seccomp is available on almost all linux os.  To check if seccomp is supported

`grep -i /boot/config-$(uname -r)` will print `CONFIG_SECCOMP=y`

run `docker run -it --rm docker/whalesay /bin/sh` and see pid. it should be 1. check the seccomp for pid at `grep Seccomp /proc/1/status` will print `Seccomp: 2`, which means secomp profile is appled in filtered mode. this is because docker has built in secomp profile. it has blocked 60+ calls from 300+ including ptrace which is used in dirty cow attack.

There r 3 modes.  **mode 0 - diabled, mode 1 - strict, mode 2 - filtered**

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

### seccomp in k8s
seccomp profile feature is introduced since kubernetes **1.22** but it is disabled by default and that means it is unconfined as default. but you can change it under securityContext section of the **pod** to below allowed values

if not defined, then it is unconfined as below.
````
securityContext:
    seccompProfile:
      type: Unconfined
````

````
securityContext:
    seccompProfile:
      type: RuntimeDefault
````
it means, it will use the container runtime seccomp profile.

````
securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/custom.json
````
it means, it will apply custom seccop profile which is stored on kubelet at `/var/lib/kubelet/seccomp/profiles/custom.json`. it has to be under `/var/lib/kubelet/seccomp`


in above case, it is explicitly enabled otherwise unconfined. 

Since **v1.27**, it can be enabled as **RuntimeDefault** for any container by passing `--seccomp-default` to kubelet runtime. To use it as unconfined, you have to explicitly do that.

To check it in docker, try as 
`docker run r.j3ss.co/amicontainerd amicontainerd`, it will show 64 calls blocked

to check in kubernetes,
`k run amicontainerd --image r.j3ss.co/amicontainerd -- amicontainerd`, it will print disabled, which means k8s by default is unconfined.


when running under seccompProfile as runtimedefault, it is possible that application may escalate the priveleges, to avoid that also set `allowPrevilegeEscalation: false` in container section of the security context.

if used audit profile, audit.json in seccompProfile localhost
````
{
    "defaultAction": "SCMP_ACT_LOG"
}
````
it will print all syscalls. you can check that in `grep syscall /var/log/syslog` file.
but it will print the number of the syscall , to see which call it is, `grep -w 35 /usr/include/asm/unistd_64.h`

another way is to run tracee in `--trace container==new`, it will print the syscall names with container names to identify it.

in exam, they can ask to copy a secomp profile from a location to all nodes and then apply it.

## AppArmor
seccomp is good for blocking syscalls but not cater to cases where you need to allow syscall conditionally. for example if you want to allow write to a file syscall but doesnt want to allow it to write in certain dir path like /proc. another example is that you dont want to allow remount of root filesystem. For those custom requirements app armor can be used.

To use app armor, you need to ensure app armor kernel module is loaded `cat /sys/module/apparmor/parameters/enabled` should be `y or Y`

it also create profiles like seccomp. these profiles are are loaded in kernel and are defined at `/sys/kernel/security/apparmor/profiles`

### profiles
````
#apparmor-deny-write

profile apparmor-deny-write flags(attached_disconnected) {
   file,
   # Deny all file writes
   deny /** w
}

#apparmor-deny-proc-write

profile apparmor-deny-proc-write flags(attached_disconnected) {
   file,
   # Deny all /proc file writes
   deny /proc/* w
}

#file in above means allow files


#apparmor-deny-remount-root

profile apparmor-deny-remount-root flags(attached_disconnected) {
   # Deny remount the read only root filesystem
   deny mount options(ro, remount) -> /
}
````

check loaded profiles by `aa-status`

3 modes - enforce, complain, unconfined

in complain mode, it will just complain and log warnings.

### create profiles
use app aror tools

`apt install apparmor-utils`

then run `aa-genprof /root/add_data.sh` to generate the profile.   note that add_data.sh is the actual command we want to run and get syscalls for that application run.

In a separate window, run `./add_data.sh`, in genprof window allow/deny whichever calls you want to allow or deny.
it will ask questions. answer like `Inherit` or `Allow` to allow syscall. it will provide severity too for each call.

check `aa-status` to see if above new profile is added.

### disable a profile
- To load a profile `apparmor_parser /etc/apparmor.d/root.add_data.sh`
- To disable it do the same above with **-R**
  ````
  apparmor_parser -R /etc/apparmor.d/root.add_data.sh
  ln -s /etc/apparmor.d/root.add_data.sh /etc/apparmor.d/disable/
  ````

## apparmor in kubernetes

below are requirements
- apparmor kernel module enabled
- appaemour profiles loaded in kernel
- container runtime should support apparmor profile (this is usally supported on all.i.e. docker, containerd crio etc.
- ensure profile are loaded `aa-status`

  add below annotation in pod under metadata `container.apparmor.security.beta.kubernetes.io: localhost/<profile-name>`

  for e.g. `container.apparmor.security.beta.kubernetes.io: localhost/apparmor-deny-write`

Above annotation was used from v1.4 to v1.29. Since v1.30, it is set as below
````
securityContext:
  appArmorProfile:
    type: <profile_type>
````
type is **RuntimeDefault, Localhost, Unconfined**

for locahost, use **localhostProfile** to set the profile and verify `kubectl exec hello-apparmor -- cat /proc/1/attr/current` should retrun profile name.

## linux capabilities
From linux kernel >2.2
there are previlege and unprevilege processes.  there are limited capabilities assigned even to previlege processes.
to check capibilities `getcap /usr/bin/ping` or `getcap 332` (pid) to see sys calls 

be default docker allows 12 syscalls only. hence when even run kubernetes pod with seccompProfile=unconfined, systime is not allowed. But to allow it, you need to add capability as below
````
securityContext:
   capabilties:
      add: ["SYS_TIME"]
````
so now, it will work.
