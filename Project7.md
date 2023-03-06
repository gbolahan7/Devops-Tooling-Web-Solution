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
