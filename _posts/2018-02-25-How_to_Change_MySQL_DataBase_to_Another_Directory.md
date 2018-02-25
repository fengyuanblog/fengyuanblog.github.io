---
layout: post
post: true
title: How to Change MySQL DataBase to Another Directory
date: 2018-02-25
category: DataBase
tag: MySQL
author: Feng Yuan
---

* content
{: toc}




## Modifying the Configuration File<br/>

1. **_Using Symbolic Links._** Create a symlink from the default mysql folder `\var\lib\mysql` to the destination data folder. [Method Link](https://www.digitalocean.com/community/tutorials/how-to-change-a-mysql-data-directory-to-a-new-location-using-a-symlink)<br/>


2. **_Using Direct Directory Moving._** Unlike making a symlink, this method changes the configuration file `my.cnf` and points the directory to the destination folder. [Method Link](https://www.digitalocean.com/community/tutorials/how-to-move-a-mysql-data-directory-to-a-new-location-on-ubuntu-16-04)<br/>


## Using Partition Mounting<br/>

1. **_Mount A Whole Partition._** The first method is directly set the mounting point of the database partition to the default MySQL folder. Doing this, the database partition becomes "invisible". It can only be accessed through its mounting point, which is the default MySQL folder.<br/>


2. **_Bind Mounting A Single Directory._** Another method is to "bind" mount the destination directory to the MySQL default directory. This is a better method since the partition itself can be mounted to some mount point and becomes "visible" to others. [Method Link](https://askubuntu.com/questions/137424/how-do-i-move-the-mysql-data-directory)<br/>

>I just tried another way, that some may find useful:<br/>
>
>copy with permissions intact:<br/>
>`rsync -avzh /var/lib/mysql /path/to/new/place`<br/>
>
>back up (in case something goes wrong):<br/>
>`mv /var/lib/mysql /var/lib/_mysql`<br/>
>
>create a new empty directory in place of old:<br/>
>`mkdir /var/lib/mysql`<br/>
>
>bind mount the new location to the old:<br/>
>`mount -B /path/to/new/place /var/lib/mysql`<br/>
>
>Hope that helps someone, because symlinking didn't work for me, and this was the simplest way<br/>

## Permission Issues<br/>

1. **_Ordinary Linux Permission Issues._** Make sure the owner/group/others permission settings match the required permissions by mysql service. However, this is just the very fundamental setting. Meeting this requirement, you can stil possibly get "Permission Denied" error when starting mysql service process.<br/>


2. **_AppArmor Permissions._** MySQL service is by default added in the AppArmor monitoring list. There is a set of permissions on the mysqld process. The default settings allow read and write operations on the default MySQL database folder which is normally the `\var\lib\mysql` folder. Therefore, we can either change the directory in the `usr.sbin.mysqld` file in the AppArmor settings or add an directory alias in the `alias` file (which is preferred).<br/>


3. **_SELinux Permissions._** Some Linux systems may have enabled the SELinux option. SELinux can not co-exist with AppArmor. So, if you have AppArmor as default, never bother this SELinux thing. But, if you do have SELinux enabled, make sure you check the permission settings of the target directory using the `ls -Z` option. Further readings on SELinux is [here](https://en.wikipedia.org/wiki/Security-Enhanced_Linux).<br/>


4. **_ACL Permissions._** This is what I omitted and bothered me a whole day after I followed every step in the aforementioned tutorials. ACL stands for Access Control Lists and is by default enabled. When you notice a `+` sign in the permission lists of your files or directories, that means you have ACL permission on thouse files or directories. ACL permissions are almost invisible when you call `ls -al` command. You have to invoke `getfacl` command to show them. ACL added a second layer of permissions on your files or folders. Especially, ACL is set **by default** on the mount point of your external hard drives. This is why sometimes you can successfully change the directory on your system hard drive but you always receive the "Permission Denied" error when changing the database directory onto an externally mounted hard drive. Read more about ACL [here](https://help.ubuntu.com/community/FilePermissionsACLs).<br/>


5. **_Parent Directories._** You should not only check the permission settings on your target directory but also all its parent directories. This guarantees that the mysqld can successfully access down to the target directory through all the above layers.<br/>
