# 2 Tier infrastructue setup through AWS
- a 2-tier infrastructure can be setup via AWS by creating a virtual private cloud (VPC), with corresponding subnets, security groups and network access lists (ACLs)
- also required is an internet gateway and routing tables - one public and one private
- each component has a different role within the infrastructure:
- VPC: a virtual network, which can contain different network resources/components
- Subnet: a subdivision of a larger network, can hold app server, database servers and bastion servers for example
- Security Group: a set of inbound and outbound rules for specific instances on a network
- Network ACL: a set of inbound and outbound rules that allow access to a subnet

## Steps to set up

![2 tier with AWS](AWS_2tier.png)
- begin by creating a VPC on AWS, for this example the IP is 10.205.0.0/16
- create an internet gateway and attach it to the created VPC
- create 3 subnets (all within the VPC): a public app subnet (10.205.1.0/24), a private database subnet (10.205.2.0/24) and a bastion subnet (10.205.3.0/24)
- 