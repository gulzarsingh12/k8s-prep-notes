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


### ssh hardening
- disbale root acess
- disbale password based auth

update `/etc/ssh/sshd_config` with `PermitRootLogin no' and `PasswordAuthentication no`

to first login with certificate use `ssh-keygen -t rsa` to generate key and then `ssh-copy-id john@node01` to copy the ssh public key to server.
