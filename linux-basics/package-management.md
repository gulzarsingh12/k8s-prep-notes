# Package Types

There are 2 types package managers mostly used:
* RPM based.i.e. red hat, fedora, centos etc.
* Debian based.i.e. debian, ubuntu, linux mint etc.

## RPM

RPM stands for redhat package manager. In rpm based installation, there are below package managers used
* rpm
* yum
* dnf

### rpm
rpm is the base package manager comes with system. Its very basic package manager and doesnt provide feature like yum. 
To install any package, all the dependencies have to be installed manually. Installation file (.rpm) also have to be 
provided explicitly.

`/var/lib/rpm` - rpm database where information about all the installed packages is stored. any changes since installation, 
what version is installed. everything is stored here.

#### commands
* `rpm -ivh telnet.rpm`  -i for install, -v for verbose
* `rpm -e telnet.rpm`  -e for uninstall
* `rpm -Uvh telnet.rpm` -U to upgrade
* `rpm -q telnet.rpm` to query the information about installed package from rpm database (/var/lib/rpm).
* `rpm -vF <path to file>` to verify that package is installed from trusted source.

### yum
it is the rpm based advance package manager which is mostly used and has features which are not in rpm. 
* software repository.  all the repo links/information to download the packages are maintained in `/etc/yum.repos.d`. files inside this are `.repo` source can be local/remote(http/ftp)
* automatic dependency resolution

As you can see no .rpm file provided but .repo file to download the package. while this is not possible with rpm.

However yum still depends on rpm package manager and calls it internally.

For licensed redhat official repo, `/etc/yum.repos.d/redhat.repo`
To use other repo like nginx `/etc/yum.repos.d/nginx.repo`

For e.g. `yum  install httpd` how it works

it will first run the transaction check to see if package is available in .repo files in `/etc/yum.repos.d/`. it will also 
resolve and try to check if any dependencies to install if not already available in the system?

Then it will print transaction summary to see the details of package that will be installed. upon confirmarion, package will be 
installed

#### commands
* `yum repolist` to list all the repos.
* `yum provides scp` to check which package will provide the scp. for e.g. /usr/bin/scp will be provided by openssh_client package
* `yum install httpd` to install the package
* `yum remove httpd` to uninstall the package
* `yum update httpd` to update the package
* `yum update` to update all the packages in the system

## Debian
dpkg is similar to rpm. its extension is .deb

### dpkg
It is also basic package manager for debian based system. same issues as rpm. no software repository and automatic dependency resolution. same way as rpm, `.deb` file has to be provided here too.

`/var/lib/dpkg/status` - pdkg database where information about all the installed packages is stored.

#### commands
* `dpkg -i telnet.deb` -i to install
* `dpkg -r telnet.deb` -r to remove
* `dpkg -l telnet.deb` -l to list
* `dpkg -s telnet.deb` -s to show status
* `dpkg -p <path to file>` -p to verify the package

### apt
it is the debian based advance package manager which is mostly used and has features which are not in dpkg. 
* software repository.  all the repo links/information to download the packages are maintained in `/etc/apt/sources.list` or `/etc/apt/sources.list.d` inside this are `.list` files. source can be local/remote(http/ftp)
* automatic dependency resolution

As you can see no .deb file provided but .repo file to download the package. while this is not possible with dpkg.

However apt still depends on dpkg package manager and calls it internally.

#### commands
* `apt update` to update/refresh the software repository with latest package information. it is good time to run it after system install or adding new source in the list
* To add more repo `apt edit-sources`. it will open the `/etc/apt/sources.list` file to add/update/remove repos. you can also do that by directly opening the `/etc/apt/sources.list` in vi editor.
* `apt upgrade` to upgrade all the packages.
* `apt install telnet` to install
* `apt remove telnet` to remove
* `apt search telnet` to search or `apt list | grep telnet`

### apt-get
this is older package manager than apt. apt is better than apt-get. it has better ui. it will show relevant information. it will show progress bar too. search command will show relevant information related to the searched package in color highlighted.
you can't search using apt-get but have to use another tool `apt-cache search telnet` but it will show lot of irrelevant information too.

commands to install remove are similar to apt.

### rpm vs debian database
* rpm stores the database at `/var/lib/rpm` while dpkg stores the database at `/var/lib/dpkg/status`
* yum stores downloaded packages at `/var/cache/yum` while apt stored the downloaded packages at `/var/cache/apt/archives` but it maintains the information of the packages at `/var/lib/apt/lists/`
* software repository sources are stored at `/etc/yum.repos.d` for yum while at `/etc/apt/sources.list` or `/etc/apt/sources.list.d` for apt.

