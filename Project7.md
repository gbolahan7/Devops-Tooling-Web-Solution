## STEP - 1 NFS SERVER SETUP

### We are going to setup new instance like previous projects. But with RHEL Operating systems with 4 instances namely NFS server, 3 web servers and database server. The Database server is setup on an Ubuntu server instance.

### Next, we configure logical volume management by creating 3 EBS volumes of 10GB each and attach the volumes to the 3 webservers respectively. 

### Create 3 partitions (xvdf, xvdg and xvdh) and corresponding physical and logical vols 
![partitions](/images/3partitions.PNG)

![phy vols](/images/physcialvols.PNG)

![log vols](/images/3logicalvols.PNG)

### Next, create 3 directories of (mnt) apps, logs and opt and then mount the already created 3 logical volumes

![mount](/images/mountpts.PNG)

## The NFS server is installed and configured to start on reboot
`sudo yum -y update`
`sudo yum install nfs-utils -y`
`sudo systemctl start nfs-server.service`
`sudo systemctl enable nfs-server.service`
`sudo systemctl status nfs-server.service`

### Permissions for the web servers to Read, Write and execute files on the NFS are created next:
`sudo chown -R nobody: /mnt/apps `
`sudo chown -R nobody: /mnt/logs `
`sudo chown -R nobody: /mnt/opt `

`sudo chmod -R 777 /mnt/apps`
`sudo chmod -R 777 /mnt/logs`
`sudo chmod -R 777 /mnt/opt`

`sudo systemctl restart nfs-server.service`

![nfs run](/images/nfs-runn.PNG)

## Configuration of access to NFS for clients within the same subnet by opening the exports file and updating the file
`sudo vi /etc/exports`

`/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)`
`/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)`
`/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)`

`sudo exportfs -arv`

### update the ports used by the NFS by opening it using security groups inbound rule so that the NFS server can be accessed from the client (TCP 111, UDP 111, UDP 2049)
`rpcinfo -p | grep nfs`

## STEP 2 - DB Server Configuration

### MySQL is installed on the DB server 
`sudo apt update`
`sudo apt install mysql-server`

### next, a database named 'tooling' is created and a user 'webaccess' created also. necessary permissions granted to the user on this database to be able to perform any operation only from the webserver subnet cidr

![nfs run](/images/dbcreate.PNG)


## STEP 3 - Web Servers Setup

## Launch a new instance and install NFS client
`sudo yum install nfs-utils nfs4-acl-tools -y`

### mount NFS server export for apps on /var/www/
`sudo mkdir /var/www`

`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`

## verify the NFS was successful mounted -> `df -h` 
## ensure changes will persist on webserver after reboot and add the line <NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0

`sudo vi/etc/fstab`

### - The Remi's repo, apache and php are all installed next. I am making use of RHEL OS 9 the installation is slightly different from 8:
`sudo dnf update -y`

`sudo dnf upgrade --refresh -y`

`sudo subscription-manager repos --enable codeready-builder-for-rhel-9-$(arch)-rpms`

`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm`

`sudo dnf install epel-release`

`sudo dnf update`

`sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm`

`sudo dnf module install php:remi-8.1`

`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`setsebool -P httpd_execmem 1`

### - Verification that the apache files and directories are webserver /var/www and the nfs server in mnt/apps by creating a new file in one server and confiming if it is present in other wweb servers.

### - The log folder for apache on webserver is mounted to nfs server export for logs 
`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd`

### - The tooling website code was forked from the git repository by installing git via the command line first and then cloning the repository

![clone](/images/repo-clone.PNG)

### - The html folder from the repo is copied into the the /var/www/html 

![html file](/images/htmlfile-copy.PNG)


### - The TCP port 80 on the web server is opened to allow connections from everywhere. The permissions on the /var/www/html folder is checked and also SELinux is disabled by setting SELINUX=disabled.

`sudo setenforce 0`

`sudo vi /etc/sysconfig/selinux`

### - The website configuration to connect to db is updated in /var/www/html/functions.php. Then tooling-db.sql script is added to the db 

`mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`

### - open the /etc/httpd/conf.d/welcome.conf and move to /etc/httpd/conf.d/welcome.backup. The Apache httpd service is then restarted.

### - The MySQL/Aurora port connection is opened on the db server EC2 instance.

### - Access the created database and create a new admin user to access the web authentication
![admin](/images/admin-toolweb.PNG)

### - The website can be accessed via http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php
![web](/images/tooling-webpage.PNG)

![web](/images/new-toolweb.PNG)


