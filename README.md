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

## Step 1 — Prepare a Web Server

#### Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.

#### Attach all three volumes one by one to your Web Server EC2 instance

#### Open up the Linux terminal to begin configuration

#### Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.

#### Use *df -h* command to see all mounts and free space on your server

![PBL6_2](https://user-images.githubusercontent.com/122687798/222915193-284cd922-e024-445d-9464-49987e877b78.JPG)

#### Use gdisk utility to create a single partition on each of the 3 disks - xvdf, xvdh, xvdg (*Learn how to use gdisk & fdisk and logical volume management*) 
1st we want to create a partition on the physical disk -  xvdf, xvdh, xvdg, the we switch to logical volume management.

*sudo gdisk /dev/xvdf* - to log in to the PHYSICAL DISK VOLUME

![PBL6_3](https://user-images.githubusercontent.com/122687798/222916376-85799228-7c1b-4093-86c3-a0fcc4b05646.JPG)

Above pix shows our partition was successful, next on this Partitions, you are going to create physical volumes

#### Install lvm2 package using the package manage yum *sudo yum install lvm2*. 

Run *sudo lvmdiskscan* command to check for available partitions.

![PBL6_4](https://user-images.githubusercontent.com/122687798/223020431-1290382f-c01a-43f6-bfb6-3720b03ac204.JPG)

Note: Previously, in Ubuntu we used apt command to install packages, in RedHat/CentOS a different package manager is used, so we shall use yum command instead.

#### Use *pvcreate* utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
Above means on that you cant create the physical volume on a physical device, you have to create them on a partition which are xvdf1, xvdh1, xvdg1.

*sudo pvcreate /dev/xvdf1*
*sudo pvcreate /dev/xvdg1*
*sudo pvcreate /dev/xvdh1*

or use below to create ALL with one CMD line

*sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1*

#### Verify that your Physical volume has been created successfully by running *sudo pvs*

![PBL6_5](https://user-images.githubusercontent.com/122687798/223021964-74d84420-6bee-493f-b4dd-8f52277f8586.JPG)

#### Use *vgcreate* utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
Above means that you are going to add all 3 stand alone 10G capacity Physical Voulme(PV) into one logical volumes called Volume Group, this VG, we will call it webdata-vg, in essence, we can create or VG like database-vg, darey-vg etc

*sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1*

![PBL6_6](https://user-images.githubusercontent.com/122687798/223023116-0d7d281c-9cf5-4908-a393-e067c414cb37.JPG)

the above has concernated all 3 PV into one VG. Now on this VG, we can now create our logical volume which u give to your servers.

#### Use *lvcreate* utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. 
*NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs*

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
So you have created a device /dev/webdata-vg/apps-lv & /dev/webdata-vg/logs-lv, so you need to add a file system to it.

*sudo mkfs -t ext4 /dev/webdata-vg/apps-lv*
*sudo mkfs -t ext4 /dev/webdata-vg/logs-lv*

Next is to create a mount point for our devices

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

[ec2-user@ip-172-31-80-191 ~]$ sudo ls -l /var/log/
total 536
drwx------. 2 root   root       23 Mar  4 15:07 audit
-rw-rw----. 1 root   utmp        0 Jan 27 05:38 btmp
-rw-r--r--. 1 root   root     1356 Mar  6 05:36 choose_repo.log
drwxr-x---. 2 chrony chrony      6 Aug 17  2021 chrony
-rw-r--r--. 1 root   root   247508 Mar  6 04:26 cloud-init.log
-rw-r-----. 1 root   adm      6589 Mar  6 04:26 cloud-init-output.log
-rw-------. 1 root   root     3172 Mar  6 06:01 cron
-rw-r--r--. 1 root   root     9365 Mar  6 05:36 dnf.librepo.log
-rw-r--r--. 1 root   root    21582 Mar  6 05:36 dnf.log
-rw-r--r--. 1 root   root     2162 Mar  6 05:36 dnf.rpm.log
-rw-r--r--. 1 root   root      720 Mar  6 05:36 hawkey.log
drwx------. 2 root   root        6 Dec 13 17:42 insights-client
-rw-rw-r--. 1 root   utmp   292292 Mar  6 04:27 lastlog
-rw-------. 1 root   root        0 Jan 27 05:38 maillog
-rw-------. 1 root   root   206346 Mar  6 06:03 messages
drwx------. 2 root   root        6 Jan 27 05:38 private
lrwxrwxrwx. 1 root   root       39 Jan 27 05:38 README -> ../../usr/share/doc/systemd/README.logs
drwxr-xr-x. 2 root   root       43 Mar  4 15:30 rhsm
-rw-------. 1 root   root    13240 Mar  6 05:45 secure
-rw-------. 1 root   root        0 Jan 27 05:38 spooler
drwxr-x---. 2 sssd   sssd        6 Jun  3  2022 sssd
-rw-------. 1 root   root        0 Jan 27 05:38 tallylog
drwxr-xr-x. 2 root   root       23 Mar  4 15:07 tuned
-rw-rw-r--. 1 root   utmp     6912 Mar  6 04:27 wtmp
[ec2-user@ip-172-31-80-191 ~]$


#### Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

*sudo rsync -av /var/log/. /home/recovery/logs/*

#### Mount /var/log on logs-lv logical volume. (REM ABOVE WARNING... 1st check the content of /var/log, ensure you backup the content before you run the MOUNT cmd. MOUNT cmd will always delete existing data on the directory, /var/log important)

*sudo mount /dev/webdata-vg/logs-lv /var/log*

#### Restore log files back into /var/log directory

*sudo rsync -av /home/recovery/logs/. /var/log*

![PBL6_9](https://user-images.githubusercontent.com/122687798/223034328-57676c8d-3230-4a35-8122-6d14a116a543.JPG)

#### Update /etc/fstab file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the /etc/fstab file;

*sudo blkid*





