# Web Solution With WordPress
## Three-tier Architecture
![project 6 architecture](https://user-images.githubusercontent.com/51024695/126815203-3c3a8c8e-383a-4e24-a3cf-8674a9d29a3c.PNG)
### Project 6 consists of two parts
#### 1.Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.
#### 2.Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.
#### Launch Ec2 instance choose redhat as OS, name it Web Server.
#### Add 3 EBS volumes and attach all three to the web server.
#### To see the attached volume and all mounts and free space
* lsblk
* df -h
![disk](https://user-images.githubusercontent.com/51024695/126822584-701c5f0e-2457-48fc-b65f-fc452a361b07.PNG)
#### A logical volume which will span accross all three volumes is what we will be using.
#### Logical volumes are not just created, you have to first create a partition, then create a physical disk. 
#### After creating a physical disk, you create a volume group and finally create the logical volume on the volume group.
#### Use gdisk to create partition on each of the three disk
* sudo gdisk /dev/xvdf
* sudo gdisk /dev/xvdg
* sudo gdisk /dev/xvdf
![xvdf_gdisk](https://user-images.githubusercontent.com/51024695/126824746-0c25ba48-e4b8-4b7c-909b-76948c1fe488.PNG)
![xvdg_gdisk](https://user-images.githubusercontent.com/51024695/126824680-d7181820-52f4-4f3b-9720-f2b9143e2d6b.PNG)
![xvdh_gdisk](https://user-images.githubusercontent.com/51024695/126824765-9b9dff14-8470-4c38-ae2f-a9029b32778b.PNG)
#### Use pvcreate to each of the 3 disk a physical volume
* sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdg1
![pvcreate](https://user-images.githubusercontent.com/51024695/126837695-f337b243-8903-49b0-82c8-e79d5da58777.PNG)
#### Use pvs to verify the physical volume creation is successful
* sudo pvs
![sudo pvs verify](https://user-images.githubusercontent.com/51024695/126838554-8efda31d-70e1-45fd-b983-639b3c295883.PNG)
#### Install lvm2 and lvmdiskscan to check for available partitions
* sudo yum install lvm2
* sudo lvmdiskscan
![install lvm2](https://user-images.githubusercontent.com/51024695/126838714-029f728a-c2f5-479b-89a4-1b8b35cce1bd.PNG)
#### Create a volume group named VG webdata with vgcreate the use sudo vgs to verify
* sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
![volume grup create and confirm](https://user-images.githubusercontent.com/51024695/126839455-7a530ca4-2d30-43b1-aa5f-195d38afb2a4.PNG)
#### Now use lvcreate to make 2 logical volumes named apps-lv and logs-lv
* sudo lvcreate -n apps-lv -L 14G webdata-vg
* sudo lvcreate -n logs-lv -L 14G webdata-vg
![creating logical volume apps and logs](https://user-images.githubusercontent.com/51024695/126841807-47360c4e-3ce0-4ac1-a364-032ae724046a.PNG)
#### View and confirm the logical volume usind lvs
* sudo lvs
![lvm](https://user-images.githubusercontent.com/51024695/126842001-24ea2ac2-5b13-47c0-87ae-26bcdcf2d4e6.PNG)
#### Use vgdisplay to view complete setup
* sudo vgdisplay -v
![sudo vgdisplay](https://user-images.githubusercontent.com/51024695/126842488-afb35543-d9f1-4c2f-9293-585674544c4b.PNG)
#### Format the logical volumes with ext4 filesystem
* sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
* sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
![ext2 filesystem on apps](https://user-images.githubusercontent.com/51024695/126843081-20e80f71-bc52-4f1c-bd17-73756b16fc03.PNG)
![ext2 filesystem on logs](https://user-images.githubusercontent.com/51024695/126843093-b6387333-d5f1-43c0-9a1d-f2fab29df646.PNG)
#### Create /var/www/html directory to store website files 
* sudo mkdir -p /var/www/html
#### Create /home/recovery/logs for recovery purposes
* sudo mkdir -p /home/recovery/logs
#### Mount /var/www/html on apps-lv
* sudo mount /dev/webdata-vg/apps-lv /var/www/html/
#### When you mount a directory in Linux, its contents are deleted. The /var/log contains essential files that can make the system malfunction if altered or deleted.
#### It was for this reason the /home/recovery/logs was created. Use rysnc to back up /var/logs to /home/recovery/logs before mounting,
* sudo rsync -av /var/log/. /home/recovery/logs/
#### Time to mount /var/log to logs logical volume we created earlier.
* sudo mount /dev/webdata-vg/logs-lv /var/log
#### Use rysnc again to restore log files back into /var/log directory.
* sudo rsync -av /home/recovery/logs/log/. /var/log
#### For the mounts to remain after a system restaart we have to update the /etc/fstab but first we must get the block id information.
* sudo blkid
![blkid](https://user-images.githubusercontent.com/51024695/126845915-ffca16ee-7315-4c17-adda-5d7322f6d03d.PNG)
#### Use vi editor to update the fstab, use mount -a to check the fstab update is succesful then restart daemon
* sudo vi /etc/fstab
![fstab config and relaod](https://user-images.githubusercontent.com/51024695/126848015-015fc2b5-cd03-4de3-9ce3-a8f43ee02966.PNG)
#### Verify your setup by running df -h
* df -h
![df-h after fstab](https://user-images.githubusercontent.com/51024695/126848092-dbb11210-2707-4985-b10f-88913d25bcb3.PNG)
## Prepare the Database Server
#### Launch a second RedHat EC2 instance that will have a role - DB Server
#### Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.
#### Update the DB server
* sudo yum update
![yum update DB](https://user-images.githubusercontent.com/51024695/126848787-5285e1ba-79ed-4524-84a5-8e7f208dc0fa.PNG)
#### After creating logical volume and mounting it to db directory, a df -h command should give you the output below
![df -h DB](https://user-images.githubusercontent.com/51024695/126849197-692df9c5-fd2f-4b7e-890b-7778cfd04989.PNG)
## Install Wordpress on your Web Server EC2
* sudo yum -y update
#### Install wget, Apache and it’s dependencies
* sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
#### Start Apache with the following commands
* sudo systemctl enable httpd 
* sudo systemctl start httpd
#### Time to install PHP and its dependencies
* sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
* sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
* sudo yum module list php
* sudo yum module reset php
* sudo yum module enable php:remi-7.4
* sudo yum install php php-opcache php-gd php-curl php-mysqlnd
* sudo systemctl start php-fpm
* sudo systemctl enable php-fpm
* setsebool -P httpd_execmem 1
#### Restart Apache
* sudo systemctl restart httpd
#### Download wordpress and copy wordpress to var/www/html. To do this, create the wordpress directory and download from there.
* mkdir wordpress
* cd   wordpress
* sudo wget http://wordpress.org/latest.tar.gz
* sudo tar xzvf latest.tar.gz
* sudo rm -rf latest.tar.gz
* cp wordpress/wp-config-sample.php wordpress/wp-config.php
* cp -R wordpress /var/www/html
### Configure SElinux Policies. 
#### SELinux is the security implementation which enhances system security and in the event of security breach, it stops that from spreading in entire system.
* sudo chown -R apache:apache /var/www/html/wordpress
* sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
* sudo setsebool -P httpd_can_network_connect=1
### Install MySQL on your DB Server EC2
* sudo yum update 
* sudo yum install mysql-server
#### Start, Enable and check status of mysqld
* sudo systemctl restart mysqld
* sudo systemctl enable mysqld 
* sudo systemctl status mysqld
![status mysqld DB](https://user-images.githubusercontent.com/51024695/126864270-734bb402-7967-444f-9710-f442eebb4f04.PNG)
### Configure DB to work with wordpress
#### To do this switch to mysql, create a database named wordpress, create a user named myuser with password 'mypass'
* Sudo mysql
* sudo mysql
* CREATE DATABASE wordpress;
* CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
* GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
* FLUSH PRIVILEGES;
* SHOW DATABASES;
* exit
![DB CREATIONAND USERS](https://user-images.githubusercontent.com/51024695/126865348-00d34e3e-a92c-4a22-8758-387af1f9c749.PNG)
#### Confirm sql user have been created successfully
* select user, host from mysql.user
![db user](https://user-images.githubusercontent.com/51024695/126865417-19185a80-75f9-4237-bf48-9faef1349cb4.PNG)
#### Connect to mysql on the DB server from the web server
* sudo mysql -u myuser -p -h < DB server local IP address>
![connect to sql from webserver](https://user-images.githubusercontent.com/51024695/126865536-25e9f013-6ff8-4258-80c0-a761bb979f4c.PNG)
#### Use show databases to view database from the webserver
* show databases;
![database from webserver](https://user-images.githubusercontent.com/51024695/126865588-1a1dcd13-259d-4956-8b89-eb2fb4df084c.PNG)
#### Open your browser and type the public IP of the webserver
![wordpress success page](https://user-images.githubusercontent.com/51024695/126865634-3e482401-647c-4be4-8cee-2c49e1d486bb.PNG
![wordpress on public IP](https://user-images.githubusercontent.com/51024695/126865639-ad0454cd-eb7d-491d-961f-cd6949dd62c2.PNG)
![filling out on server public ip](https://user-images.githubusercontent.com/51024695/126865655-5a2f1d77-935e-47a7-9b1a-99b3ef2d62e1.PNG)











