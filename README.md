# Learning Management System | Nevtik E-Learning
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
### OpenLiteSpeed Installation<br/>
Actually there are many option to install Openlitespeed but we choose installation Openlitespeed using Openlitespeed repository. Because we using Openlitespeed repository for install all of the dependency of the Openlitespeed, so we must adding Openlitespeed repository to the Ubuntu repository with command ```wget```
```bash
# wget -O - http://rpms.litespeedtech.com/debian/enable_lst_debain_repo.sh | bash
```
*For the details step of installing Openlitespeed from repository you can visit <a href="https://openlitespeed.org/kb/install-ols-from-litespeed-repositories/">this link.</a>

Next, we install Openlitespeed web server package and lsphp73 package for php programming language in moodle.
```bash
$ sudo apt-get install openlitespeed lsphp73 lsphp73* -y
```

After installation package success, we must set admin password of Openlitespeed web config. we changing admin password with running file ```/usr/local/lsws/admin/misc/admpass.sh``` and we following the instructions that appear to changing default password with new password.<br/>
And then we testing our Openlitespeed web admin control. We testing using our browser to access url ```http://<IP_Address of LMS Server>:7080```.

### OpenLiteSpeed Configuration<br/>
Firstly, we create directory for moodle source code and moodledata. And then we moving moodle source code to that directory also creating directory ```moodledata```. We also changing the ownership and access right of those directory.
```bash
$ sudo mkdir /usr/local/lsws/moodle
$ sudo mv moodle /usr/local/lsws/moodle/moodle
$ sudo mkdir /usr/local/lsws/moodle/moodledata
$ sudo chown www-data:www-data -R /usr/local/lsws/moodle/moodle*
$ sudo chmod 777 -R /usr/local/lsws/moodle/moodledata
$ sudo chmod 755 -R /usr/local/lsws/moodle/moodle
```
Next, we creating Virtualhost and Listener in the OpenLiteSpeed web server. For this step we must configure using OpenLiteSpeed web control. For how we make it you can look at this <a href="https://idroutes.blogspot.com/2020/06/konfigurasi-moodle-di-openlitespeed.html">link</a>.<br/>
We do a compiling PHP, but actually this compiling PHP are not recommended from OpenLiteSpeed official and we just curious about that feature on OpenLiteSpeed. For detailing about how we do compiling PHP you can look at this <a href="https://openlitespeed.org/kb/build-custom-php-for-openlitespeed/">link</a>.<br/>
*So, because compiling PHP are not really important you can skip that step.<br/>

### Database Configuration<br/>
Before we installing Moodle, we create database for moodle. For Moodle database we do some improvement that we will showing about those improvement at later.<br/>
Firstly, we installing and after that we running command ```mysql_secure_installation``` for do hardening mariadb.
```bash
$ sudo apt-get install mariadb-server-10.3 mariadb-client-10.3 -y
$ sudo mysql_secure_installation
```
In the *mysql_secure_installation* we just following the instructions. We configure MariaDB to supporting *utf8mb4*, we editing at file ```/etc/mysql/mariadb.conf.d/50-server.cnf```. We adding the configuration like this:
```bash
[mysqld]
default_storage_engine = innodb //For setting MariaDB to using innodb as database engine
innodb_file_per_table = 1 //it is we adding because we using format Barracuda
innodb_file_format = Barracuda //For using format Barracuda in MariaDB
innodb_large_prefix = 1 //For activating feature large prefix inside MariaDB
```

After that we starting to create database for Moodle like this:
```bash
$ sudo mysql -u root -p
Enter password:
MariaDB [(none)]> CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
MariaDB [(none)]> CREATE USER 'admin'@'localhost' IDENTIFIED BY 'nevtikskillsjuara_2021';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON moodle.* TO 'admin'@'localhost';
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> exit
$ sudo systemctl restart mariadb.service
```
We noticed in some conditions our MariaDB cannot running appropriately. And after we looked for the problem, we found that it happened because of the *apparmor* which is the default feature of Ubuntu. So we must disabling MariaDB from apparmor protection. And after that the MariaDB can run appropriately.
```bash
$ sudo ln -s /etc/apparmor.d/usr.sbin.mysqld /etc/apparmor.d/disable/
$ sudo apparmor_parser -R /etc/apparmor.d/usr.sbin.mysqld 
$ reboot
$ #Verify
$ sudo aa-status
```
*Note: For make sure if MariaDB will auto running when restarting VM you can enable mysql.service with systemctl enabl mysql.service


### Moodle Installation<br/>
For this installation, we install Moodle by accessing the Moodle by LMS's IP Address. Actually at this step we get some problem because were not using Private IP Address for installing Moodle but using Public IP Address for installing Moodle. And the problem is we get notice from Moodle if the problem are about *install hijacked*, so for solving that we must changing some code of PHP in the source code Moodle the were using. For detailing you can read at this <a href="https://docs.moodle.org/310/en/error/admin/installhijacked"> link </a>. And for the rest we did the Moodle installation just like what often appears in tutorials on the internet<br/>
*Note: If you get error when installing just make sure you read the red notice that appear and then search :)<br/>

### Improvement<br/>

1. OpenLiteSpeed Tuning
   - Enable lscache => https://openlitespeed.org/kb/openlitespeed-cache-module/
   - Configure SSL Connections => https://openlitespeed.org/kb/lets-encrypt-ssl-on-openlitespeed/?seq_no=2
   - Activate HTTP/2 and HTTP/3
2. Moodle
   - Using Moove Theme => https://moodle.org/plugins/theme_moove
   - Using https connection for Moodle:
     Access ```/usr/local/lsws/moodle/moodle/config.php``` and then change ```http://<Domain LMS>``` to be ```https://<Domain LMS>```
   - Making image on login page to be responsive:
     Access ```Site Administration > Appereance > Themes > Moove > Advanced > Raw SCSS``` and then added CSS code like this:
     ```bash
     #page-login-index.moove-login .logo img {
       height: 100%;
       width: 100%;
       max-height: 230px;
       max-width: 658px;
     }
     ```
     And then click 'Save', oh yeah don't forget to adding logo to the Moove theme.
3. MariaDB
   - Improvement in the ```/etc/mysql/mariadb.conf.d/50-server.cnf``` like this:
     ```bash
      [mysqld]

      datadir=/var/lib/mysql
      socket=/var/lib/mysql/mysql.sock
      symbolic-links=0
      
      query_cache_size=16M
      query_cache_type=1
      query_cache_limit=256K
      query_cache_min_res_unit=2K
      
      innodb_buffer_pool_instances=4
      innodb_buffer_pool_size=5G
      skip-name-resolve
      
      tmp_table_size= 512M
      max_heap_table_size= 512M
      
      key_buffer_size         = 32M
      max_allowed_packet      = 512M
      thread_stack            = 192K
      thread_cache_size       = 2
      max_connections         = 1000

      character-set-server = utf8mb4
      collation-server = utf8mb4_unicode_ci
      skip-character-set-client-handshake
      ```
      *Note: Some value can change depending on performance requirements<br/>
      *Reference Links: https://severalnines.com/database-blog/database-performance-tuning-mariadb
    - Improvement in the ```/etc/mysql/mariadb.conf.d/50-mysqld_safe.cnf``` like this: 
      ```bash
      [mysqld_safe]
      log-error=/var/log/mariadb/mariadb.log
      pid-file=/var/run/mariadb/mariadb.pid
      ```
    - Improvement in the ```mariadb systemd```, we changing MEMLOCK from limited to be infinity
4. PHP
   - We changing file_uploads, max_post_file, and memory_limits on ```/usr/local/lsws/lsphp/lsphp73/etc/php/litespeed/php.ini```
     *Note: All can be change dpeending on performance requirements
5. Crontab
   - So, we using ```crontab``` to scheduling some commands like for renewing SSL, clear cache, and automatic restart openlitespeed.
      ```bash
   0  0 1 */3 * certbot renew //For renewing SSL Certificate
   0  */3 * * * sync; echo 3 | sudo tee /proc/sys/vm/drop_caches //For clearing caches
   0  */24  * * * sudo systemctl restart lsws.service //For auto restarting openlitespeed
   0  */0-2 *  *  *  php /usr/local/lsws/moodle/moodle/admin/cli/cron.php
   ```
   *Note: you can change the scheduler with using command ```crontab -e```<br/>
   
   - Cron rules for some task on Moodle itself, for detail information about that you can visit ```Administration\Site Administration\Server\Task\Scheduled Tasks```<br/>
     Reference Links: https://docs.moodle.org/310/en/Scheduled_tasks#Format_for_scheduling_tasks<br/>
   
For more informations about this LMS, you can call admin of Nevtik E-Learning<br/>
Link of E-Learning Nevtik: https://academy.nevtik.org
