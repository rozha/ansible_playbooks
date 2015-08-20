## ec2\_proftpd.yml
This playbook performs the following actions:

* ensure an 'ftpd' security group exists (TCP 20-22, 25000-27000)
* create a t1.micro instance
* update the system using a package manager
* install, configure, and run ProFTPD and fail2ban

Tested with the following AMIs:

* ami-98aa1cf0 Ubuntu Server 14.04 LTS (PV), SSD Volume Type, 64 bit
* ami-246ed34c Amazon Linux AMI 2014.09.1 (PV), 64 bit
* ami-1643ff7e Red Hat Enterprise Linux 6.5 (PV), 64 bit

To use:

* edit **vars/ec2\_proftpd**
 * **image**: AMI to create your new instance from
 * **instance\_type**: type of the instance
 * **keypair**: key pair to use
 * **region**: region to create the new instance in
 * **vpc\_id**: id of the VPC to run the instance in
 * **zone**: AWS availability zone
 * **ssh\_user**: user name to use to log in to the instance via SSH

* invoke the following commands:
 * export AWS\_ACCESS\_KEY=**YOURIAMACCESSKEY**
 * export AWS\_SECRET\_KEY=**YOURIAMSECRETKEY**
 * ansible-playbook ec2\_proftpd.yml
