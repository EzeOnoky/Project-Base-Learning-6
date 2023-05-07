# Project-Base-Learning-6
## WEB SOLUTION WITH WORDPRESS

### PROJECT TASK
Prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress.

### KNOWLEDGE BASED INFO
You are progressing in practicing to implement web solutions using different technologies. As a DevOps engineer you will most probably encounter PHP-based solutions since, even in 2021, it is the dominant web programming language used by more websites than any other programming language.

WORDPRESS is a content management system(CMS) which is majorly built on PHP programming language(Recall your project 2, you deployed a TO DO application with PHP)

In this project you will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system(CMS) written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS). For this project, we will use MySQL.

Project 6 consists of two parts:

*1 - Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.* On the Web server, we will install MySQL client, while on the Database Server, we will install MySQL server, We will make that the web server is able to connect to the Database Server

*2 - Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.*

![PBL6_1](https://user-images.githubusercontent.com/122687798/222914753-8825b4eb-b37e-46ff-a6a7-396ae48dcb26.jpg)

So why this architecture, basically this is deployed for security, u dont want your data base being impacted when the web server develops an issue.

Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.

Business Layer (BL): This is the backend program that implements business logic. Application or Webserver

Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS 
Server

# STEP 1 — PPREPARE THE WEB SERVER

#### Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

#### Attach all three volumes one by one to your Web Server EC2 instance

#### Open up the Linux terminal to begin configuration

#### Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.

#### Use *df -h* command to see all mounts and free space on your server

![PBL6_2](https://user-images.githubusercontent.com/122687798/222915193-284cd922-e024-445d-9464-49987e877b78.JPG)

#### Use gdisk utility to create a single partition on each of the 3 disks - xvdf, xvdh, xvdg (*Learn how to use gdisk & fdisk and logical volume management*) 
1st we want to create a partition on the physical disk -  xvdf, xvdh, xvdg, then we switch to logical volume management.

*sudo gdisk /dev/xvdf* - to log in to the PHYSICAL DISK VOLUME and create partitions

![PBL6_3](https://user-images.githubusercontent.com/122687798/222916376-85799228-7c1b-4093-86c3-a0fcc4b05646.JPG)

Above pix shows our partition was successful, so from physical disk xvdf, xvdg, xvdh, we have created partitions  xvdf1, xvdg1, xvdh1. Next on this Partitions, we are going to create physical volumes.

#### Install lvm2 package using the package manage yum *sudo yum install lvm2*. 

Run *sudo lvmdiskscan* command to check for available partitions.

![PBL6_4](https://user-images.githubusercontent.com/122687798/223020431-1290382f-c01a-43f6-bfb6-3720b03ac204.JPG)

Note: Previously, in Ubuntu we used apt command to install packages, in RedHat/CentOS a different package manager is used, so we shall use yum command instead.

#### Use *pvcreate* utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
Above means on that you cannot create the physical volume on a physical device, you have to create them on a partition which are xvdf1, xvdh1, xvdg1.

*sudo pvcreate /dev/xvdf1*
*sudo pvcreate /dev/xvdg1*
*sudo pvcreate /dev/xvdh1*

or use below to create ALL with one CMD line

*sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1*

#### Verify that your Physical volume has been created successfully by running *sudo pvs*

![PBL6_5](https://user-images.githubusercontent.com/122687798/223021964-74d84420-6bee-493f-b4dd-8f52277f8586.JPG)

#### Use *vgcreate* utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

Above means that you are going to add all 3 stand alone 10G capacity Physical Voulme(PV) into one logical volumes called Volume Group, this VG, we will call it webdata-vg, in essence, we can create or VG like database-vg, darey-vg etc, creating this VG is a perequisite for creation of a logical volume. You cannot create logical volume on a partitioned physical volume. Volume Group gives you the flexibility to expand your disk space. lets say you desire to expand your infrastructure, after the physical disk is added, disk partioning is done, the partitioned physical volume is add to the VG. After this, the logical volume can now be expanded.

*sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1*   Notice the 3 10g PV has been concernated to for the 29G VG.

![PBL6_6](https://user-images.githubusercontent.com/122687798/223023116-0d7d281c-9cf5-4908-a393-e067c414cb37.JPG)

the above has concernated all 3 PV into one VG. Now on this VG, we can now create our logical volume which u give to your servers.

#### Use *lvcreate* utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. 
*NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs* This Logical Volume is now the actual disk you give to your servers(DB & Web server)

*sudo lvcreate -n apps-lv -L 14G webdata-vg*
*sudo lvcreate -n logs-lv -L 14G webdata-vg*

#### Verify that your Logical Volume has been created successfully by running & also Verify the entire setup

*sudo lvs*
*sudo vgdisplay -v #view complete setup - VG, PV, and LV*
*sudo lsblk*

![PBL6_7](https://user-images.githubusercontent.com/122687798/223024164-895d9013-140f-44f1-9139-36e3f33aa3bf.JPG)

#### So lets say our Logical Volume(LV) apps-lv gets filled up and we need expansion, how do we proceed?.

If it is our traditional disk - xvdf, xvdh, xvdg, to expand, we have to remove the disk and add a larger capacity disk, but with Logical volume, this is not the case. With LV, we just add another physical disk, probably xvdi, xvdj etc, then you proceed to create a physcial volume with the added physical disk i.e *sudo pvcreate /dev/xvdi1*. After this, the new PV is added to the Volume Group - VG. this increases the size of the VG. With the extra size available, you can now allocate more capacity to the Logical Volume - LV

#### Use mkfs.ext4 to format the logical volumes with ext4 filesystem
So you have created a the logical volumes /dev/webdata-vg/apps-lv & /dev/webdata-vg/logs-lv, now you need to make it a file system.

*sudo mkfs -t ext4 /dev/webdata-vg/apps-lv*
*sudo mkfs -t ext4 /dev/webdata-vg/logs-lv*


Next is to create a mount point for our devices(logical volume)

#### Create /var/www/html directory to store website files

*sudo mkdir -p /var/www/html*

#### Create /home/recovery/logs to store backup of log data

*sudo mkdir -p /home/recovery/logs*

### NOTE : the -P in *sudo mkdir -p /home/recovery/logs* will auto create the folders recovery, if it does not exist already. Likewise for *sudo mkdir -p /var/www/html*....folder www is auto created if it does not exist already.


#### Mount /var/www/html on apps-lv logical volume

*sudo mount /dev/webdata-vg/apps-lv /var/www/html/*

Above is simply asking you to move this device - apps-lv into this directory - /var/www/html/

![PBL6_8](https://user-images.githubusercontent.com/122687798/223030435-7bc01c5d-461d-4262-8b37-40b30df95fe0.JPG)


#### 1t check content of /var/log/

sudo ls -l /var/log/

#### Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

*sudo rsync -av /var/log/. /home/recovery/logs/*

#### Mount /var/log on logs-lv logical volume. (REM ABOVE WARNING... 1st check the content of /var/log, ensure you backup the content before you run the MOUNT cmd. MOUNT cmd will always delete existing data on the directory, /var/log important)

*sudo mount /dev/webdata-vg/logs-lv /var/log*

#### Restore log files back into /var/log directory

*sudo rsync -av /home/recovery/logs/. /var/log*
![PBL 6_9A](https://user-images.githubusercontent.com/122687798/236599553-92efe306-6586-41ab-b97a-b31e34a99c7b.jpg)
![PBL6_9](https://user-images.githubusercontent.com/122687798/223034328-57676c8d-3230-4a35-8122-6d14a116a543.JPG)

#### Update /etc/fstab file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the /etc/fstab file;

*sudo blkid*

The UUID in above print out is what you will use to populate the FSTAB. NB - UUID is equivalent to the created LV

*sudo vi /etc/fstab*

Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.

#### Test the configuration and reload the daemon

*sudo mount -a*
*sudo systemctl daemon-reload*
 
#### So we have updated the FSTAB, Verify your setup by running *df -h* , output must look like this:

![PBL6_10](https://user-images.githubusercontent.com/122687798/223326282-789a007a-aed4-4390-86e7-8c715aded86a.JPG)

# STEP 2 — PREPARE THE DATABASE SERVER

Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

#### 1 - Create Volume
#### 2 - Attach it

![PBL6_11](https://user-images.githubusercontent.com/122687798/223327645-19420cfa-20df-4989-bc24-33db2202244e.JPG)

#### 3 - Connect to your EC2 Instance for the database server and execute below CMD and partition the created disk volume

#### Start with disk volume - xvdf
lsblk


[ec2-user@ip-172-31-92-248 ~]$

#### Now we move to the next disk volume - xvdg

![PBL6_12](https://user-images.githubusercontent.com/122687798/223331183-ea6b533a-9b8e-4f1c-8a99-5a50f8b8b52e.JPG)

#### Now we move to the next disk volume - xvdh

SO we have partitioning of our 3 disk volume successfully done. do *lsblk* to confirm this.

![PBL6_13](https://user-images.githubusercontent.com/122687798/223331961-c6649a8d-32dd-455d-8fcd-a9b6d34aab92.JPG)

#### 4 Install the LVM2 - Logical Volume Management

*sudo yum install lvm2 -y*

#### 5 Mark each of 3 disks partitions as physical volumes (PVs) to be used by LVM, use the CMD - pvcreate

*sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1*

#### 6 Now procced to create the Volume Group VG using the 3 PVs

*sudo vgcreate vg-database /dev/xvdf1 /dev/xvdg1 /dev/xvdh1*

#### 7 Now we create our Logical Volume LV, name it db-lv, give it a 20G size

 *sudo lvcreate -n db-lv -L 20G vg-database*

Recall for the Web server, we created Logical Volume : apps-lv, in the Database Server, we are creating db-lv and mount it to /db directory instead of /var/www/html/ as was done in the Web Server

#### 8 Create a directory in the root( / ), name it db, then Make the new LV a file system

*sudo mkdir /db*
*sudo mkfs.ext4 /dev/vg-database/db-lv*

#### 9 Now mount the LV, but b4 u do, ensure u check and confirm the directory /db is empty

*sudo ls -l /db*
*sudo mount /dev/vg-database/db-lv /db*  so u mounted the content of /dev/vg-database/db-lv into /db

![PBL6_14](https://user-images.githubusercontent.com/122687798/223337893-d68d105d-db3d-4d9e-bd8f-93c8ddcbbed6.JPG)
Our Mount was successful !

#### 10 Now make the mount permanent by putting it in the /FSTAB and doing a system reload

*sudo vi /etc/fstab*
*sudo mount -a*
*sudo systemctl daemon-reload*
*df -h*

Below is the final setup of Disk for the Web Server and The Database Server

![PBL6_16](https://user-images.githubusercontent.com/122687798/236682699-5842a0d5-5b3e-47ec-b94d-b5b2fdeb5d2f.JPG)


# STEP 3 — INSTALL WORDPRESS ON THE WEB SERVER EC2

#### Update the repository - this will take some time

*sudo yum -y update

#### Install wget, Apache and it’s dependencies

*sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json

*sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm*


#### Start Apache

*sudo systemctl enable httpd
*sudo systemctl start httpd

#### To install PHP and it’s depemdencies

sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1

#### Restart Apache
*sudo systemctl restart httpd

#### Download wordpress and copy wordpress to var/www/html

![PBL 6_10](https://user-images.githubusercontent.com/122687798/236604987-12b13202-f635-4a12-b4d4-f39839d029d3.jpg)




