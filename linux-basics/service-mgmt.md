# SystemD
This is the init system for linux OS now. SystemD, once started will take care starting all the services of the system required to function. This depends on the selected target/runlevel though.

Based on the default target, it will will read all the targets inside the default target. So if default.target has reference to multi-user.target then it start all the service defined the multi-user.target or targets defined further in it. for e.g. network.target etc.

All the services are defined under `/etc/systemd/system/project-mercury.service`. This `.service` file is called **service unit** file

## Sections
Directives defined in below sections are as in below service unit file.

````
[Unit]
Description=Python Django for project mercury
Documentation=http://wiki.caleston-dev.ca/mercury
After=postgresql.service

[Service]
ExecStart=/bin/bash /usr/bin/project-mercury.sh
User=project_mercury
Restart=on-failure
RestartSec=10

[Install]
WantedBy graphical.target
````

### Service
In the service unit file, under this section, we define the basic minimum derivative `ExecStart` which is script to run. This is minimum required field to run any service.

````
[Service]
ExecStart=/bin/bash /usr/bin/project-mercury.sh
````

To run this service run `systemctl start project-mercury.service`. You can check the status using `systemctl status project-mercury.service`

To stop `systemctl stop project-mercury.service`

#### User
If not specifiec, service will start as root. Set below to set the user to start with
````
[Service]
User=project_mercury
````

#### Restart
* If service startup fails, we want it to `Restart` on failure. 
* To specify the time interval after which the service should be restarted (in secs)
````
[Service]
Restart=on-failure
RestartSec=10
````

### Install
To enable the above during boot, it has to specify the target/runlevel which you are going to use during the boot.

Specify the `WantedBy` directive to set **target**/runlevel

````
[Install]
WantedBy graphical.target
````

### Unit
To define the dependencies on which it depends and wait for that.
You can also define any documentation or description/information about this service.

````
[Unit]
Description=Python Django for project mercury
Documentation=http://wiki.caleston-dev.ca/mercury
After=postgresql.service
````

## Commands
### daemon-reload
`systemctl daemon-reload`
Once service unit file is saved, this command is run to reload the configuration changes.

Please note that this command will reload the configuration changes in **systemd**. it could be for all the services or systemd changes too.

### reload
`systemctl reload project-mercury.service`
When you change something in service unit file, you can reload the changes without restareting the service. So it only for the changes to that particular service.

Note that, the service should support this. It means that if you change something in service unit and then service should be able to reflect those changes in service without restart otherwise yuo need to restart the service anyway.

### edit
`systemctl edit project-mercury.service --full` 
To make changes in the service unit. 

Benefit of this is that it will make the changes affective immediately and the you dont need to run `daemon-reload`

### other commands
* `systemctl start docker` to start the service
* `systemctl stop docker` to stop the service
* `systemctl restart docker` to restart the service
* `systemctl enable docker` to enable the service. To start the service at system boot
* `systemctl disable docker` to disable the service at boot
* `systemctl disable docker` to check the status of service.e.g, Active (running), Inactive (stopped), Failed (Error/Crashed/Timeout)
* `systemctl get-default` to get the default target
* `systemctl set-default multi-user.target` to set the target.
* `systemctl list-units --all` To list all the service units with more details
* `systemctl list-units` To list the service units only.

## JournalCtl
This is a command line tool to get the logs information about service units.

* `journalctl` will print all the logs of all the services from oldest entry to latest
* `journalctl -b` to see all the logs from current boot.
* `journalctl -u project-mercury.service` to see the logs of specific service. 

`journalctl`

