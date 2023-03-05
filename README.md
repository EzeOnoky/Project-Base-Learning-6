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

#### Install lvm2 package using *sudo yum install lvm2*. Run *sudo lvmdiskscan* command to check for available partitions.

Note: Previously, in Ubuntu we used apt command to install packages, in RedHat/CentOS a different package manager is used, so we shall use yum command instead.




