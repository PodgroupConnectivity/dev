Development Environment
=========

A collection of tools to make working with the numerous software components as painless as possible.

If there's a task that annoys you when working with the code, open an issue to suggest an improvement. Or, better, send a pull request to get your automation included in the project.

Installation guide
=========

Changed instructions for successful building of OpenBTS in Ubuntu Server 12.04 LTS with USPR 2901 or Ettus B210

- [Development Environment](#development-environment)
  - [Install Base Operating System](#install-base-operating-system)
  - [Update Git](#update-git)
- [Downloading](#downloading)
  - [Tools](#tools)
  - [Components](#components)
- [Selecting a Branch or Tag](#selecting-a-branch-or-tag)
- [Compiling and installing previous dependencies](#compiling-and-installing-previous-dependencies)
- [Building](#building)
- [Installing](#installing)
- [Exploring](#exploring)
  - [OpenBTS Command Line Interface](#openbts-command-line-interface)
  - [Subscriber Registry Database](#subscriber-registry-database)

### Disclaimer

_These instructions are admittedly focused around a very specific collection of hardware and software, namely those that are used in the products that Range Networks ships. We are working to change this so instructions are provided for a wide variety of Linux distributions and architectures as well as for SDR hardware from different vendors._

_[Click here](https://wush.net/trac/rangepublic/wiki) to view more complete documentation regarding hardware setups._

# Development Environment

Use this section to set up a development environment that is identical to those used internally at Range Networks. We are working to support other development environments setups in the future but this one is guaranteed to work.

## Install Base Operating System

_Note: The AMD64 architecture is now building correctly on all packages but has not yet undergone functional testing. If you are interested in trying it out, please use a 64-bit Ubuntu flavor and then select the "master" branch when building below._

If you're already running Ubuntu Desktop or Server 32-bit 12.04, you can skip this section.

1. download Ubuntu Server 32-bit 12.04.5 LTS [from here](http://releases.ubuntu.com/12.04/ubuntu-12.04.5-server-i386.iso)
1. boot the .iso on a fresh machine (VMWare virtual machines are also often used)
1. select "English" for language
1. press F4
1. select "Install a minimal system" from install type
1. select "Install Ubuntu Server" from main menu
1. select your language
1. select your country
1. use keyboard layout auto-detection or select the country and layout
1. enter a hostname
1. enter "openbts" for "full name for the new user"
1. enter "openbts" for "username for your account"
1. enter and confirm a password for the openbts account
1. select "no" to "Encrypt your home directory?"
1. select "Guided - Use Entire Disk" for "Partitioning method"
1. select/confirm the disk you'd like to use
1. select "yes" to "Write the changes to disks?"
1. enter nothing for "HTTP Proxy" and select continue
1. select "No automatic updates"
1. toggle OpenSSH on in "Software selection"
1. select "continue"
1. select "yes" to "Install the GRUB boot loader on a hard disk"
1. select "continue" to "Finish the installation"

## Update Git

The OpenBTS project utilizes several new features in Git. To make sure your client is compatible (version >=1.8.2), perform the following.

```
$ sudo apt-get install software-properties-common python-software-properties
$ sudo add-apt-repository ppa:git-core/ppa
(press enter to continue)
$ sudo apt-get update
$ sudo apt-get install git
```

Once you have git installed, most of the remaining installation process is automated via scripts.

# Downloading

Several software components are needed to actually create a usable mobile network. To efficiently manage them, we've prepared some scripts to automate the clone, pull, branch and build operations.

## Tools

From the command line in your fresh development environment, execute the following to download the most recent set of tools:

```
$ git clone https://github.com/RangeNetworks/dev.git
```

Before proceeding, these tools require that you are using a modern version of Git (>1.8.2). Check now if you are:

```
$ git --version
git version 1.9.1
```

## Components

This development scripts assume that you have SSH keys set up for GitHub. If you do not, please [[follow these instructions|https://help.github.com/articles/generating-ssh-keys]] to set them up before proceeding.

Now, to download all of the components simply run the ```clone.sh``` script. 

```
$ cd dev
$ ./clone.sh
```

# Selecting a Branch or Tag

Before building, you should choose which branch or tag you'd like to compile using ```switchto.sh```.

```
$ ./switchto.sh 5.0
```

Now change the downloading link for libcoredump for the one below (attached the whole line):
`./libcoredumper/build.sh: Line 28`
```
sayAndDo wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/google-coredumper/coredumper-1.2.1.tar.gz
```

# Compiling and installing previous dependencies
Compiling and installing some dependencies
```
cd liba53
make
sudo make install
sudo ldconfig
cd ..
```

Installing zmq
Adding repos:
```
sudo apt-get install software-properties-common python-software-properties
sudo add-apt-repository ppa:chris-lea/zeromq
sudo apt-get update
```
Installing components:
```
sudo apt-get install libzmq3
sudo apt-get install libzmq3-dev
sudo apt-get install python-zmq 
```

# Building

The ```build.sh``` script will automatically install any build dependencies (building them manually when required). After dependencies are taken care of, each component is compiled into an installable package.

```
$ ./build.sh B210
```

Compiled packages are now in a new directory named ```BUILDS/sometimestamp```.

# Installing

Use dpkg to install the fresh packages (this will complain about dependencies):

```
$ sudo dpkg -i BUILDS/timestamp/*.deb
```

To have Aptitude resolve the dependencies, execute the following:

```
$ sudo apt-get -f install
```

When prompted about overwriting existing configuration files, use your own judgment. It is recommended to overwrite all files to make sure things work out of the box. However, overwriting ```/etc/network/interfaces``` will set your system to a static IP instead of whatever you had configured previously.

# Running

Each component has an Upstart service definition for Ubuntu. To start all the required services, execute the following:

```
$ sudo start sipauthserve
$ sudo start smqueue
$ sudo start openbts
$ sudo start asterisk
```

Conversely, to stop them:

```
$ sudo stop sipauthserve
$ sudo stop smqueue
$ sudo stop openbts
$ sudo stop asterisk
```

# Exploring

## OpenBTS Command Line Interface

```
$ cd /OpenBTS
$ ./OpenBTSCLI
OpenBTS> help       (list all commands available)
OpenBTS> audit      (check if your configuration is correct)
OpenBTS> config     (list all parameters)
OpenBTS> config XYZ (list all parameters that contain XYZ)
OpenBTS> devconfig  (change developer and factory parameters)
OpenBTS> trxconfig  (view the factory radio calibration) [so far only on Range Networks equipment]
OpenBTS> chans      (view the currently active channels)
OpenBTS> tmsis      (view all IMSIs that have interacted with the system)
OpenBTS> trans      (view all completed transactions like calls and sms)
OpenBTS> quit
```

## Subscriber Registry Database

```
$ sudo sqlite3 /var/lib/asterisk/sqlite3dir/sqlite3.db
sqlite> .tables
DIALDATA_TABLE RRLP SIP_BUDDIES rates
sqlite> select * from sip_buddies;
sqlite> select * from dialdata_table;
sqlite> .quit
```[javier@archy] ~/wkspc/podgroup/repos/dev/dev.wiki (master)$ cd ..
[javier@archy] ~/wkspc/podgroup/repos/dev (master)$ rm -rf dev.wiki/
[javier@archy] ~/wkspc/podgroup/repos/dev (master)$ cd ..
[javier@archy] ~/wkspc/podgroup/repos $ git clone https://github.com/RangeNetworks/dev.wiki.git
Clonando en 'dev.wiki'...
remote: Counting objects: 88, done.
remote: Total 88 (delta 0), reused 0 (delta 0), pack-reused 88
Desempaquetando objetos: 100% (88/88), listo.
[javier@archy] ~/wkspc/podgroup/repos $ cd dev.wiki/
[javier@archy] ~/wkspc/podgroup/repos/dev.wiki (master)$ ls
Home.md
[javier@archy] ~/wkspc/podgroup/repos/dev.wiki (master)$ cat Home.md 
- [Development Environment](#development-environment)
  - [Install Base Operating System](#install-base-operating-system)
  - [Update Git](#update-git)
- [Downloading](#downloading)
  - [Tools](#tools)
  - [Components](#components)
- [Selecting a Branch or Tag](#selecting-a-branch-or-tag)
- [Building](#building)
- [Installing](#installing)
- [Exploring](#exploring)
  - [OpenBTS Command Line Interface](#openbts-command-line-interface)
  - [Subscriber Registry Database](#subscriber-registry-database)

### Disclaimer

_These instructions are admittedly focused around a very specific collection of hardware and software, namely those that are used in the products that Range Networks ships. We are working to change this so instructions are provided for a wide variety of Linux distributions and architectures as well as for SDR hardware from different vendors._

_[Click here](https://wush.net/trac/rangepublic/wiki) to view more complete documentation regarding hardware setups._

# Development Environment

Use this section to setup a development environment that is identical to those used internally at Range Networks. We are working to support other development environments setups in the future but this one is guaranteed to work.

## Install Base Operating System

_Note: The AMD64 architecture is now building correctly on all packages but has not yet undergone functional testing. If you are interested in trying it out, please use a 64-bit Ubuntu flavor and then select the "master" branch when building below._

If you're already running Ubuntu Desktop or Server 32-bit 12.04, you can skip this section.

1. download Ubuntu Server 32-bit 12.04.4 LTS [from here](http://sunsite.rediris.es/mirror/ubuntu-releases/precise/ubuntu-12.04.4-server-i386.iso)
1. boot the .iso on a fresh machine (VMWare virtual machines are also often used)
1. select "English" for language
1. press F4
1. select "Install a minimal system" from install type
1. select "Install Ubuntu Server" from main menu
1. select your language
1. select your country
1. use keyboard layout auto-detection or select the country and layout
1. enter a hostname
1. enter "openbts" for "full name for the new user"
1. enter "openbts" for "username for your account"
1. enter and confirm a password for the openbts account
1. select "no" to "Encrypt your home directory?"
1. select "Guided - Use Entire Disk" for "Partitioning method"
1. select/confirm the disk you'd like to use
1. select "yes" to "Write the changes to disks?"
1. enter nothing for "HTTP Proxy" and select continue
1. select "No automatic updates"
1. toggle OpenSSH on in "Software selection"
1. select "continue"
1. select "yes" to "Install the GRUB boot loader on a hard disk"
1. select "continue" to "Finish the installation"

## Update Git

The OpenBTS project utilizes several new features in Git. To make sure your client is compatible (e.g. newer than 1.8.2), perform the following.

```
$ sudo apt-get install software-properties-common python-software-properties
$ sudo add-apt-repository ppa:git-core/ppa
(press enter to continue)
$ sudo apt-get update
$ sudo apt-get install git
```

Once you have git installed, most of the remaining installation process is automated via scripts.

# Downloading

Several software components are needed to actually create a usable mobile network. To efficiently manage them, we've prepared some scripts to automate the clone, pull, branch and build operations.

## Tools

From the command line in your fresh development environment, execute the following to download the most recent set of tools:

```
$ git clone https://github.com/RangeNetworks/dev.git
```

Before proceeding, these tools require that you are using a modern version of Git (>1.8.2). Check now if you are:

```
$ git --version
git version 1.9.1
```

## Components

This development scripts assume that you have SSH keys set up for GitHub. If you do not, please [[follow these instructions|https://help.github.com/articles/generating-ssh-keys]] to set them up before proceeding.

Now, to download all of the components simply run the ```clone.sh``` script. 

```
$ cd dev
$ ./clone.sh
```

# Selecting a Branch or Tag

Before building, you should choose which branch or tag you'd like to compile using ```switchto.sh```.

```
$ ./switchto.sh master
(or)
$ ./switchto.sh 4.0
(or)
$ ./switchto.sh v4.0.0
```

# Building

The ```build.sh``` script will automatically install any build dependencies (building them manually when required). After dependencies are taken care of, each component is compiled into an installable package.

```
$ ./build.sh <radio-type>
```

Compiled packages are now in a new directory named ```BUILDS/sometimestamp```.

# Installing

Use dpkg to install the fresh packages (this will complain about dependencies):

```
$ sudo dpkg -i BUILDS/timestamp/*.deb
```

To have Aptitude resolve the dependencies, execute the following:

```
$ sudo apt-get -f install
```

When prompted about overwriting existing configuration files, use your own judgment. It is recommended to overwrite all files to make sure things work out of the box. However, overwriting ```/etc/network/interfaces``` will set your system to a static IP instead of whatever you had configured previously.

# Running

Each component has an Upstart service definition for Ubuntu. To start all the required services, execute the following:

```
$ sudo start sipauthserve
$ sudo start smqueue
$ sudo start openbts
$ sudo start asterisk
```

Conversely, to stop them:

```
$ sudo stop sipauthserve
$ sudo stop smqueue
$ sudo stop openbts
$ sudo stop asterisk
```

# Exploring

## OpenBTS Command Line Interface

```
$ cd /OpenBTS
$ ./OpenBTSCLI
OpenBTS> help       (list all commands available)
OpenBTS> audit      (check if your configuration is correct)
OpenBTS> config     (list all parameters)
OpenBTS> config XYZ (list all parameters that contain XYZ)
OpenBTS> devconfig  (change developer and factory parameters)
OpenBTS> trxconfig  (view the factory radio calibration) [so far only on Range Networks equipment]
OpenBTS> chans      (view the currently active channels)
OpenBTS> tmsis      (view all IMSIs that have interacted with the system)
OpenBTS> trans      (view all completed transactions like calls and sms)
OpenBTS> quit
```

## Subscriber Registry Database

```
$ sudo sqlite3 /var/lib/asterisk/sqlite3dir/sqlite3.db
sqlite> .tables
DIALDATA_TABLE RRLP SIP_BUDDIES rates
sqlite> select * from sip_buddies;
sqlite> select * from dialdata_table;
sqlite> .quit
