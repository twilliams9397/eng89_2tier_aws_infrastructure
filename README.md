# 2 Tier infrastructue setup through AWS
- a 2-tier infrastructure can be setup via AWS by creating a virtual private cloud (VPC), with corresponding subnets, security groups and network access lists (ACLs)
- also required is an internet gateway and routing tables - one public and one private
- each component has a different role within the infrastructure:
- VPC: a virtual network, which can contain different network resources/components
- Subnet: a subdivision of a larger network, can hold app server, database servers and bastion servers for example
- Security Group: a set of inbound and outbound rules for specific instances on a network. Security groups are a type of STATEFUL firewall, which means that it remembers an outgoing connection and automatically allows the response from that address.
- Network ACL: a set of inbound and outbound rules that allow access to a subnet. This is an example of a STATELESS firewall, where all ingoing and outgoing connections must be explicitly allowed as the deffault is to deny any connection.

## Steps to set up

![2 tier with AWS](AWS_2tier.png)
- begin by creating a VPC on AWS, for this example the IP is 10.205.0.0/16
- create an internet gateway and attach it to the created VPC
- create 3 subnets (all within the VPC): a public app subnet (10.205.1.0/24), a private database subnet (10.205.2.0/24) and a bastion subnet (10.205.3.0/24)
- create a public route table (add route 0.0.0.0/0 for all traffic) for the internet gateway, and associate it to the app and bastion subnets

### Security Groups

App:
- **inbound**: HTTP (port 80) and HTTPS (port 443) from anywhere (ensure there is a rule for IPv4 and IPv6), port 3000 from anywhere and SSH (port 22) from your personal IP

Database:
- **inbound**: Mongodb port 27017 from the app security group and SSH (port 22) from the bastion security group (must add this after bastion SG is created)

Bastion:
- **inbound**: SSH (port 22) from your personal IP

### Network ACLs

App (public subnet):
- **inbound**: HTTP (port 80) and HTTPS (port 443) from everywhere, SSH (port 22) from personal IP and port range 1024-65535 from everywhere
- **outbound**: HTTP (port 80) and HTTPS (port 443) to everywhere, Mongodb port 27017 to the database (private) IP and port range 1024-65535 to everywhere

Database (private subnet):
- **inbound**: SSH (port 22) from the bastion subnet IP (10.205.3.0/24), Mongodb port 27017 from the database subnet (10.205.2.0/24), HTTP (port 80), HTTPS (port 443) and port range 1024-65535 from everywhere
- **outbound**: port ranfe 1024-65535 to the app subnet (10.205.1.0/24) and bastion subnet (10.205.3.0/24) and HTTP (port 80) and HTTPS (port 443) to anywhere

Bastion:
- **inbound**: SSH (port 22) from personal IP and port range 1024-65535 from the database subnet (10.205.2.0/24)
- **outbound**: SSH (port 22) to the databse subnet (10.205.2.0/24) and port range 1024-65535 to personal IP

### ec2 Instances
- all created within the previously created VPC and with the same steps as https://github.com/twilliams9397/eng89_cloud_computing_aws/blob/main/README.md except the below settings for IPs and security groups:
- app: use the public subnet, enable the public IP, apply the app security group (created from Amazon Machine Image which already had the necessary installs and reverse proxy setup - see above README for details)
- databse: use the private subnet, disable the public IP, apply the database security group (created from Amazon Machine Image which already had the necessary installs - see above README for details)
- bastion: use the bastion subnet, enable the public IP, apply the bastion security group

### Deploying the Web App
- using the `scp` command, copy the .pem key into the bastion instance: `scp -i eng89_devops.pem eng89_devops.pem ubuntu@<bastion IP>:~`
- using the `scp` command, copy the app folder into the app instance e.g. `scp -i eng89_devops.pem -r /Users/Tom1/Documents/Sparta/Vagrant/Dev_Env/eng89_dev_env/app ubuntu@<app IP>:~/app/`
- once the key is correctly copied into the bastion instance, the database instance can be accessed via SSH from within the bastion instance
- run `sudo systemctl status mongod` to ensure the databse is running
- back in the app instance, ensure the `DB_HOST=mongodb://<IP>/27017` is the correct IP for the private database and ensure the reverse proxy in `/etc/nginx/sites-available/default` has the correct IP - public app IP
- navigate to the app folder and run `node seeds/seed.js` and then `node app.js`
- the web app should now be running on your app's public IP address