# Linux Security Concepts

## Access Controls
This makes use of password based authentication to determine who can access the system

## PAM
Suppose I want to login to the linux system, I can easily open the ssh and connect it with the system using my user name password for authentication.
Just think about it, how this ssh can authenticate the user? PAM (Pluggable Authentication Module) is the revolutionary step in this. Any app can defer the authenctication to the PAM and PAM will do the authentication for you.

Suppose you develop an app, you want to authenticate with the system. How would you do that? One way is that you get the user credentials and then validate it with `/etc/passwd` and `/etc/shadow` to ensure user is authenticated. However, you are doing this manually. there can be any issue and every app have to do this authentication. what about different authentication methods like fingerprint, biometric verification etc. How to do that and everything in the world in the app itself?
Solution is PAM. App can let PAM to authenticate the user for it. This is very useful specially for application who are used systems. Pam related configuration can be defined for the apps at `/etc/pam.d` and `/ets/security`
PAM modules can also limit the system resources like cpu and memory to ensure that one user is not using all the system resources. There are many modules in PAM.

### without PAM
APP1   --->   Auth Activity --->   `/etc/passwd` `/etc/shadow`

### with PAM
APP1   --->   PAM --->   `/etc/passwd` `/etc/shadow`

### Network Security
Usually external firewalls are deployed to secure the network of the system. But on the system itself, iptables and firewalld can be used to secure the communication.

### SSH Hardening
To secure the ssh access to the system.

### SELinux
SELinux makes use of security policies to isolate the application from each other to protect the linux system.

# User Accounts

* Users are the system users who use system.
* Groups can used to define the specific set of the users who use the system in same way. For example, developers use the system for development.
* When a user is created, it is created in `/etc/passwd` and `/etc/shadow`
* When group is created, it is created `/etc/group`

## User
A user can be break down into below
* Username
* UID
* GID
* Home Directory
* Default Shell

To list this information use the `id` command. e.g. `id michael` will show the username, uid,gid, groups for the specified user. Hence when we need to use this for other than logged in user.

If `id` command is used without username then it will display of informatio for the current login user.

### Account Types

#### user account.
A regular user account.e.g bob, michal

#### super user account
A super user account. This account have unrestricted access to the system. It has (**UID=0**) e.g. root

#### System user account
These are users like for software and services which are deplyed when OS is installed. These users are created to not run as super user for the softwares. e.g. ssh, mail. **UID** is usually <100 or between 500-1000. They dont usually have dedicated home dir. if they do, its **not created under** `/home`

#### Service accounts
These are similar to system accounts. These are accounts created when some service is installed on the system. e.g. when nginx is installed, a user with name nginx is created to be used by service installed.


### Commands

* `id` To see the information about current user
* `who` To see information of currently logged in user. like when logged in, ip address from where logged in etc.
* `last` to show all logged in users. It also show adate and time when system was rebooted?
* `su -` to switch user as root. 

### /etc/sudoers
To run any command as root user without switching user is `sudo <command>`. It will prompt the logged user for its password.
For any command to work with `sudo`, a user must be listed in `/etc/sudoers`

Great benefit come with this is that we can restrict the root user's access to the shell. we can define   `root:x:0:0:root:/root:/usr/sbin/nologin` in `/etc/passwd`
With this, no user can login as root user but can run commands as sudo.

#### Syntax of the file
`user/group host=(user:group) command`
e.g. `bob ALL=(ALL:ALL) ALL` bob is allowed to run all commands as all users and all groups on all hosts

NOPASSWD can be used when you don't to get prompted for password on use of sudo command
`bob ALL=(ALL:ALL) NOPASSWD:ALL`


`USER_SPACE  HOST=RunAs(User:Group)  COMMANDS`
or
`USER_SPACE  HOST=RunAs(User:Group)  NOPASSWD:COMMANDS`

- USER_SPACE : this is the first column of the syntax which can contain username, %group or User_Alias
- HOST       : this will contain hostname detail on which the sudo will perform the assigned task
- RunAs      : This optional clause controls the target user (and group) sudo will run the Command as.
- COMMAND    : Give the list of command(s) which you want the user to execute

Aliases in sudoers file
- User_Alias
- Host_Alias
- Cmnd_Alias

`#includedir /etc/sudoers.d` this is not a comment but actual configuration in sudoers file. It means to define sudoers configuration files in `/etc/sudoers.d` dir. Note that there is no space between **#** directive and **includedir**

**Note:** You can edit this file with `visudo` only.

##### Examples
* With aliases
  ````
  User_Alias  ALIAS_NAME = bob, michal
  Host_Alias  APP_SERVERS = web, db
  Cmnd_Alias  CROND_SERVICE = /usr/bin/systemctl restart crond.service, /usr/bin/systemctl status crond.service

  ALIAS_NAME APP_SERVERS=(ALL:ALL) CROND_SERVICE
  ````
* With group
  `%sudo ALL=(ALL:ALL) ALL` sudo group is allowed all commands at all hosts as all users and groups
* With host
  `deepak domainserver=(ALL) /usr/bin/systemctl restart crond.service`
  Here user deepak is only allowed to run the specified command on domainserver only.

  This is useful for ops when they have to configure this on 1000s of machines. so this configuration is deployed on 1000s of server but deepak is domain administrator and only should be given admin acess to domain server not all the servers.

## Access Control Files

These are created in a way to not edit directly but using commands.

### /etc/passed
`bob:x:1001:1001::/home/bob:/usr/sbin/bash`

* USERNAME.e.g. bob
* PASSWORD.e.g. x. Password is not saved here. It is not stored here but in `/etc/shadow` file
* UID.e.g. 1001. it is required field as well
* GID.e.g. 1001. group id is same as user id if not specified. it is required field as well
* GECOS:
* HOMEDIR.e.g /home/bob
* SHELL.e.g. /bin/bash

### /etc/shadow
To store the hashed password of the user.
`bob:fdsfqeedfdsf:18188:0:99999:7:::`

Content of this file are hashed.

* USERNAME.e.g. bob
* PASSWORD. e.g. fdsfqeedfdsf. enrypted password. if astrik(*) or empty field, then it means it is never set.
* LASTCHANGE timestamp since 1970
* MINAGE 
* MAXAGE 
* WARN.e.g. 7. no of days before password expires when the used should be warned.
* INACTIVE.  no of days user can be inactive before disabling the user after expired passwor. if empty then it means not enforced.
* EXPDATE.  when account will be expired, if empty nnever expire

### /etc/group
Stores basic information about groups

`developer:x:1001:bob,sara`

* NAME.e.g. develper
* PASSWORD.e.g. not set here but in shadow file
* GID.e.g. 1001
* MEMBERS.e.g. bob,sara

## Commands

### useradd
to create new user. `useradd bob`

it will add `bob:x:1001:1001::/home/bob:/bin/sh` to /etc/passwd
and `bob:!:18341:0:99999:7:::` to /etc/shadow

`useradd -u 1009 -g 1009 -d /home/robert -s /bin/bash -c "Mercury Project member" bob`

* -c custom comments
* -d home dir
* -e expiry date
* -g specific gid
* -u specific uid
* -s specify shell
* -G multiple secondry groups

Checl with `id bob`

### userdel
to delete user  `userdel bob`

### passwd
To set the passwd `passwd bob`

To change the own password `passwd`

### groupadd
`groupadd -g 1001 developer`

### groupdel
to delete group `groupdel developer`

## Files permissions

first char in `ls -l` is file type.

### File types
* `d` directory
* `-` regular file
* `b` block device
* `c` character device
* `s` socket
* `l` symlink
* `p` pipe

### permission types

Permissions can be given to
* `u` owner/user permissions
* `g` group permissions
* `o` other users/group which is not the owner/group.

#### types

`-rwx rwx rwx`

`   u   g   o `

##### file/dir permissions

| Bit | Purpose | Octal Value |
| --  | ------- | ----------- |
r|Read|4
w|Write|2
x|Execute|1
-|No Permission|0

Suppose a dir have following permissions 

`ls -ld /home/bob/random_dir`

`d--xrwxrwx 1 bob bob 89  `

if u see above, read and write is no permission and but execute permission. what is execute permission mean for dir if no read and write permission.

it means if user tries to do `ls -l /home/bob/random_dir` it will show permission denied but if user execute a command like `cd /home/bob/random_dir`

But if you notice the group has full permissions but still not working. Actually permissions are checked sequentially and here owner permissions will be consider and no group permissions are checked further.

But if some user of this group tried to list, it will work.

##### octal value

Below are some examples

| octal | Permission |
| ----- | ---------- |
777|all permissions to ugo
755|all to u, r & x to group and others
555| r&x to all ugo
644| r&w to u, read only to g and o

##### possible octal combinations
| octal | count | read | write | execute |
| ----- | ----- | ---- | ----- | ------- |
7|4+2+1|y|y|y
6|4+2+0|y|y|n
5|4+0+1|y|n|y
4|4+0+0|y|n|n
3|0+2+1|n|y|y
2|0+2+0|n|y|n
1|0+0+1|n|n|y
0|0+0+0|n|n|n

### chown/chgrp
To change the owner of the file

* `chown bob:developer test.txt` will change the owner and group of the file to bob and developer.
* `chown bob test.txt` change only the owner of the file to bob
* `chgrp developer test.txt` change only the group of the file to developer
