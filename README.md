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

#### Security Groups

App:
- **inbound**: HTTP (port 80) and HTTPS (port 443) from anywhere (ensure there is a rule for IPv4 and IPv6), port 3000 from anywhere and SSH (port 22) from your personal IP

Database:
- **inbound**: Mongodb port 27017 from the app security group and SSH (port 22) from the bastion security group (must add this after bastion SG is created)

Bastion:
- **inbound**: SSH (port 22) from your personal IP

#### Network ACLs

App (public subnet):
- **inbound**: HTTP (port 80) and HTTPS (port 443) from everywhere, SSH (port 22) from personal IP and port range 1024-65535 from everywhere
- **outbound**: 