# LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”

##### Step 1 — Prepare a Web Server
- Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

<img width="1033" alt="Web server" src="https://user-images.githubusercontent.com/51254648/153749360-8a2a21d4-31b3-4703-925e-cdc643a2959c.png">

<img width="1025" alt="EBS vol" src="https://user-images.githubusercontent.com/51254648/153749364-40132b4a-fee0-422a-818c-a3d1bb647250.png">


- Attach all three volumes one by one to your Web Server EC2 instance
 
 <img width="1033" alt="EBS attached" src="https://user-images.githubusercontent.com/51254648/153749367-bd724491-a1e8-4dc8-8740-caca594b0eca.png">


- Use *lsblk* command to inspect what block devices are attached to the server.

<img width="745" alt="lsblk" src="https://user-images.githubusercontent.com/51254648/153749375-2d2cb396-7a8c-4316-97bb-87d27b032a1c.png">


- Use *df -h* command to see all mounts and free space on your server

<img width="382" alt="mount   space" src="https://user-images.githubusercontent.com/51254648/153749378-539598a7-7bde-4043-86c7-b808e03c4731.png">

- Use gdisk utility to create a single partition on each of the 3 disks
*sudo gdisk /dev/xvdf*

<img width="563" alt="partition xvdf" src="https://user-images.githubusercontent.com/51254648/153749383-12c0af19-61b0-47a6-aedf-3bef9bdcf900.png">

*sudo gdisk /dev/xvdg*

<img width="544" alt="Partition xvdg" src="https://user-images.githubusercontent.com/51254648/153749385-22a11ebc-3e44-42bc-9db8-946430a05edb.png">

*sudo gdisk /dev/xvdh*

<img width="546" alt="Partition xvdh" src="https://user-images.githubusercontent.com/51254648/153749387-9a2fd26f-9738-47ac-a42a-4b3ff69bf76a.png">


- Use *lsblk* utility to view the newly configured partition on each of the 3 disks.

<img width="316" alt="new partitions" src="https://user-images.githubusercontent.com/51254648/153749389-6146b0a2-b987-496f-95da-c7118219512e.png">


- Install lvm2 package using *sudo yum install lvm2*. 

<img width="832" alt="install lvm2" src="https://user-images.githubusercontent.com/51254648/153749390-d38f547d-0f04-4eb3-923d-1646343c7374.png">

- Run *sudo lvmdiskscan* command to check for available partitions.

<img width="350" alt="available partition" src="https://user-images.githubusercontent.com/51254648/153749391-b3c9049a-52a2-4cb2-a27e-27c844c2c5fa.png">


- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

<img width="405" alt="PV create" src="https://user-images.githubusercontent.com/51254648/153749392-67e92186-9fdd-408b-802f-32ec09b1168a.png">


- Verify that your Physical volume has been created successfully by running *sudo pvs*

<img width="298" alt="verify pv" src="https://user-images.githubusercontent.com/51254648/153749394-ed996f83-c838-40ec-a54d-6733f5571daf.png">


- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
*sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1*

- Verify that your VG has been created successfully by running *sudo vgs*

<img width="334" alt="verify vg" src="https://user-images.githubusercontent.com/51254648/153749396-5d6c0b34-2c69-4743-a1c5-4e4dae46f091.png">


- Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. 
*sudo lvcreate -n apps-lv -L 14G webdata-vg*
*sudo lvcreate -n logs-lv -L 14G webdata-vg*

<img width="631" alt="create vg" src="https://user-images.githubusercontent.com/51254648/153749397-e4c1355c-a740-49da-8448-fa468dda3100.png">

<img width="523" alt="create lv" src="https://user-images.githubusercontent.com/51254648/153749399-3e7ff1ae-961f-437e-995c-50bf710a59bf.png">


- Verify that your Logical Volume has been created successfully by running sudo lvs

<img width="638" alt="verify lvs" src="https://user-images.githubusercontent.com/51254648/153749400-8b55330e-7d99-404f-97c9-383a62386d64.png">


- Verify the entire setup
*sudo vgdisplay -v* #view complete setup - VG, PV, and LV

<img width="577" alt="verify setup 1" src="https://user-images.githubusercontent.com/51254648/153749401-b7cbb78a-911c-4f48-a50a-ef4d38a22d4a.png">


*sudo lsblk*

<img width="421" alt="verify setup 2" src="https://user-images.githubusercontent.com/51254648/153749402-fdf83119-f184-4ec5-bbcc-dc6ca0a2a915.png">


- Use mkfs.ext4 to format the logical volumes with ext4 filesystem
*sudo mkfs -t ext4 /dev/webdata-vg/apps-lv*
*sudo mkfs -t ext4 /dev/webdata-vg/logs-lv*

<img width="568" alt="ext4 filesystem" src="https://user-images.githubusercontent.com/51254648/153749404-fd3c7269-231a-4d4c-ae9d-d01c8943f12d.png">


- Create /var/www/html directory to store website files
*sudo mkdir -p /var/www/html*

- Create /home/recovery/logs to store backup of log data
*sudo mkdir -p /home/recovery/logs*

- Mount /var/www/html on apps-lv logical volume
*sudo mount /dev/webdata-vg/apps-lv /var/www/html/*

- Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
*sudo rsync -av /var/log/. /home/recovery/logs/*

- Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. 
*sudo mount /dev/webdata-vg/logs-lv /var/log*

- Restore log files back into /var/log directory
*sudo rsync -av /home/recovery/logs/. /var/log*

- Update /etc/fstab file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the /etc/fstab file
*sudo blkid*

<img width="941" alt="dev UUID" src="https://user-images.githubusercontent.com/51254648/153749407-5681b376-29ad-4010-bc9b-b0a35ca1ccf0.png">


*sudo vi /etc/fstab*

<img width="691" alt="edit fstab" src="https://user-images.githubusercontent.com/51254648/153749409-fe048af0-f582-4608-9940-fa33799287dc.png">


- Test the configuration and reload the daemon
 *sudo mount -a*
 *sudo systemctl daemon-reload*
 
 - Verify your setup by running df -h, 
 
 <img width="551" alt="verify setup" src="https://user-images.githubusercontent.com/51254648/153749410-8c0ab66e-0ccc-4d38-843f-67826a3d879f.png">

 
 ## Step 2 — Prepare the Database Server
 - Launch an EC2 instance that will serve as "db-server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

<img width="1027" alt="Db-server" src="https://user-images.githubusercontent.com/51254648/153749411-53546502-b0ad-42d5-8f97-5fb5edfc06a0.png">

<img width="1020" alt="create db vol" src="https://user-images.githubusercontent.com/51254648/153749413-f24338ad-8bc9-484d-908d-ef4b945f946a.png">


- Attach all three volumes one by one to your Web Server EC2 instance

<img width="1026" alt="attach db vol" src="https://user-images.githubusercontent.com/51254648/153749415-134c5a37-3860-4747-b493-549dc279f102.png">


- Use *lsblk* command to inspect what block devices are attached to the server.

<img width="314" alt="db lsblk" src="https://user-images.githubusercontent.com/51254648/153749418-f0f2c0f4-1821-403d-a168-cd81079580e4.png">


- Use *df -h* command to see all mounts and free space on your server

<img width="365" alt="db mnt   spc" src="https://user-images.githubusercontent.com/51254648/153749419-afb7396e-f37e-4a7d-b5f2-4e3eab3b24f9.png">


- Use gdisk utility to create a single partition on each of the 3 disks
*sudo gdisk /dev/xvdf*

<img width="593" alt="db partition xvdf" src="https://user-images.githubusercontent.com/51254648/153749420-2e54d249-8a64-4029-a460-fe9b323760ec.png">

*sudo gdisk /dev/xvdg*

<img width="557" alt="db partition xvdg" src="https://user-images.githubusercontent.com/51254648/153749421-5c5a106c-d29e-4442-80d4-7f832b7c545a.png">

*sudo gdisk /dev/xvdh*

<img width="549" alt="db partition xvdh" src="https://user-images.githubusercontent.com/51254648/153749423-713c119e-3a84-4fc1-b3e3-1f923a6b44f0.png">

- Use *lsblk* utility to view the newly configured partition on each of the 3 disks.

<img width="321" alt="db partitions" src="https://user-images.githubusercontent.com/51254648/153749425-69509af2-443d-4822-9801-f97b4066e217.png">


- Install lvm2 package using *sudo yum install lvm2*. 

<img width="790" alt="db install lvm2" src="https://user-images.githubusercontent.com/51254648/153749426-827b3a82-2327-437c-bab1-9783b25e5fa5.png">

- Run *sudo lvmdiskscan* command to check for available partitions.

<img width="350" alt="db available partition" src="https://user-images.githubusercontent.com/51254648/153749428-fddf4b11-eb59-4c71-8d3d-31bb403abe29.png">

- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

<img width="395" alt="db pv create" src="https://user-images.githubusercontent.com/51254648/153749429-ecc455cc-9162-494e-b6e8-34f86fa16963.png">

- Verify that your Physical volume has been created successfully by running *sudo pvs*

<img width="308" alt="db verify pvs" src="https://user-images.githubusercontent.com/51254648/153749430-027bf928-819c-46dd-8f80-b9118c6f8391.png">


- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG dbdata-vg
*sudo vgcreate dbdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1*

- Verify that your VG has been created successfully by running *sudo vgs*

<img width="329" alt="db verify vg" src="https://user-images.githubusercontent.com/51254648/153749431-f9ccfe28-ed9b-48d8-b088-6a279c7a8587.png">

- Use lvcreate utility to create 2 logical volumes. db-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. 
*sudo lvcreate -n db-lv -L 14G dbdata-vg*
*sudo lvcreate -n logs-lv -L 14G dbdata-vg*

<img width="507" alt="db create lvs" src="https://user-images.githubusercontent.com/51254648/153749435-c9c5aaa0-d391-4ecc-8487-0492d90f47ed.png">


- Verify that your Logical Volume has been created successfully by running sudo lvs

<img width="627" alt="db verify lvs" src="https://user-images.githubusercontent.com/51254648/153749436-52966c46-9694-445b-aa85-25b2b048d3c9.png">


- Verify the entire setup
*sudo vgdisplay -v* #view complete setup - VG, PV, and LV

<img width="583" alt="db verify setup" src="https://user-images.githubusercontent.com/51254648/153749437-1ed18a04-ff2d-4263-8970-bee018d2d56a.png">

*sudo lsblk*

<img width="417" alt="db verify setup 2" src="https://user-images.githubusercontent.com/51254648/153749439-7883929c-534f-4b97-bb31-3d34c2f3b4c0.png">


- Use mkfs.ext4 to format the logical volumes with ext4 filesystem
*sudo mkfs -t ext4 /dev/dbdata-vg/db-lv*
*sudo mkfs -t ext4 /dev/dbdata-vg/logs-lv*

<img width="556" alt="db ext4 filesystem" src="https://user-images.githubusercontent.com/51254648/153749440-4da44d52-4f5a-43a3-8fb8-d262d7a11149.png">


- Create /db directory to store database files
*sudo mkdir -p /db*

- Create /home/recovery/logs to store backup of log data
*sudo mkdir -p /home/recovery/logs*

- Mount /var/www/html on db-lv logical volume
*sudo mount /dev/dbdata-vg/db-lv /db*

- Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
*sudo rsync -av /var/log/. /home/recovery/logs/*

- Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. 
*sudo mount /dev/dbdata-vg/logs-lv /var/log*

- Restore log files back into /var/log directory
*sudo rsync -av /home/recovery/logs/. /var/log*

- Update /etc/fstab file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the /etc/fstab file
*sudo blkid*

<img width="819" alt="db blkid" src="https://user-images.githubusercontent.com/51254648/153749444-eb4444e5-d6f8-49d8-993a-cd59806f8f44.png">

*sudo vi /etc/fstab*

<img width="660" alt="db edit fstab" src="https://user-images.githubusercontent.com/51254648/153749448-41203aae-9af9-4441-a551-86937dba4d5e.png">

- Test the configuration and reload the daemon
 *sudo mount -a*
 *sudo systemctl daemon-reload*
 
 - Verify your setup by running df -h, 
 
 <img width="489" alt="db verify full setup" src="https://user-images.githubusercontent.com/51254648/153749451-db4a4462-4f4a-4d49-aff5-12dad90ab430.png">



##### Step 3 — Install WordPress on your Web Server EC2

- Update the repository
*sudo yum -y update*

- Install wget, Apache and it’s dependencies
*sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json*

- Start Apache
*sudo systemctl enable httpd*
*sudo systemctl start httpd*

- To install PHP and it’s depemdencies
*sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1*

- Restart Apache
sudo systemctl restart httpd

- Download wordpress and copy wordpress to var/www/html
  *mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
  sudo cp -R wordpress /var/www/html/*
  
  - Configure SELinux Policies
  sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
  
  ##### Step 4 — Install MySQL on your DB Server EC2
  *sudo yum update
sudo yum install mysql-server*

- restart the service and enable it so it will be running even after reboot:
*sudo systemctl restart mysqld
sudo systemctl enable mysqld* 
- Verify that the service is up and running by using sudo systemctl status mysqld

<img width="726" alt="mysql status" src="https://user-images.githubusercontent.com/51254648/153749452-1e9aeeed-ce8e-410e-aadf-eeac3545db00.png">

##### Step 5 — Configure DB to work with WordPress
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
 
 <img width="536" alt="create wordpress db" src="https://user-images.githubusercontent.com/51254648/153749453-ae752482-d607-4c7b-ba27-ea9f8edba52d.png">

##### Step 6 — Configure WordPress to connect to remote database.
 - open MySQL port 3306 on DB Server EC2

<img width="1131" alt="add port 80" src="https://user-images.githubusercontent.com/51254648/153749459-94848cf9-f97b-4935-9071-d5e7bc74dbfa.png">
 
 - Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client
*sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>*

 - Verify if you can successfully execute SHOW DATABASES; 
 
 <img width="733" alt="show db frm web server" src="https://user-images.githubusercontent.com/51254648/153749455-4f38b8a6-a623-41cd-8632-9d3479ce7887.png">

 
 - Change permissions and configuration so Apache could use WordPress
 
 <img width="706" alt="change config" src="https://user-images.githubusercontent.com/51254648/153749456-dd0bc72b-fa16-43ad-823d-9b7e03847e58.png">

 
 - Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/
 
 <img width="1271" alt="Screenshot 2022-02-12 at 19 18 47" src="https://user-images.githubusercontent.com/51254648/153750129-a395818b-6fc0-4dcd-b2d5-d881a812f7ca.png">
 

