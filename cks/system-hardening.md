# Run command on ssh
- `ssh user@host 'command1' 'command2'`. you can pass multiple commands

# reduce attack surface
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


