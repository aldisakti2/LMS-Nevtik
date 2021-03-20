## LMS-Nevtik
Hi, this is first project of Nevtik for building the Learning Management System or LMS for Nevtik Organization itself.<br/>
And in this note i will explain step by step that i and my team do for making this LMS running.

## Specification

- Server Spesification<br/>
Hypervisor          : Proxmox VE<br/>
Operating System    : Ubuntu Server 20.04<br/>
RAM                 : 6 GB<br/>
HDD                 : 64 GB<br/>
Network Adapter     : VirtIO Adapter (Bridge)<br/>

- Software Spesification<br/>
LMS Software        : Moodle v3.10.2<br/>
Web Server          : OpenLiteSpeed 1.6.20<br/>
Database Server     : MariaDB<br/>


## Step By Step
### Install VM on Proxmox<br/>

### IP Addressing Server<br/>
In this step the lms server using DHCP Reservation IP and in the default configuration
of Ubuntu Server we no need to configure IP Addressing in this VM. And configuration of
IP DHCP Reservation are in the Router that connected to lms server.<br/>

### Download Moodle<br/>
We must downloading the moodle before we continue to configure other else. And for this step our team was making this LMS to be connected to the internet. So, our LMS Server can connect to the moodle download page.<br/><br/> 
For downloading moodle from CLI LMS Server we using ```wget``` command for download moodle source code
```bash
$ sudo wget https://download.moodle.org/download.php/direct/stable310/moodle-3.10.2.tgz
```
After that we extract the moodle source with ```tar -xvf``` command
```bash
$ sudo tar -xvf moodle-3.10.2.tgz
```
### Openlitespeed Installation<br/>
Actually there are many option to install Openlitespeed but we choose installation Openlitespeed using Openlitespeed repository. Because we using Openlitespeed repository for install all of the dependency of the Openlitespeed, so we must adding Openlitespeed repository to the Ubuntu repository with command ```wget```
```bash
# wget -O - http://rpms.litespeedtech.com/debian/enable_lst_debain_repo.sh | bash
```
Next, we install Openlitespeed web server package and lsphp73 package for php programming language in moodle.
```bash
$ sudo apt-get install openlitespeed lsphp73 lsphp73* -y
```
After installation package success, we must set admin password of Openlitespeed web config.
For the details step of installing Openlitespeed from repository you can visit <a href="https://openlitespeed.org/kb/install-ols-from-litespeed-repositories/">this link.</a>

