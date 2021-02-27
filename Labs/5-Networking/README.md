# Lab 5 - AWS Networking

## Introduction

In this lesson, we are going to learn how to configure your AWS network. We are going to create :

- A VPC (Virtual Private Cloud)
- Private and Public Subnets
- Route Tables
- An Internet Gateway
- A NAT Gateway

VPC acts as network isolation for your computing, storage, and other resources you use. All
resources that belong to a VPC are logically isolated and launched inside an isolated virtual
network. In a VPC network, we have sub-networks (subnets).

Subnetting has many benefits. In a large network with no subnets, every server would see
broadcast packets from all the servers and users on the network, resulting in the switches having
to move all that traffic to the appropriate ports. It also allows for easier administration and
growth control. It helps in boosting security; for instance, it is more secure to create a private
subnet for the machines that we don't need to expose to the Internet.

Note: A machine in a private subnet, even without being public, can access to the Internet. This is
possible using the NAT gateway (Network Address Translation). When a private-addressed
machine sends or receive traffic, it will use the NAT gateway without even having a public IP
address.

So, each private subnet should have its own NAT gateway in order to allow external access. Even
if we don't need access to the Internet in most cases, using APT or YUM package managers to
install and upgrade your system packages, need Internet access.

## Internet Gateway vs. NAT Gateway

Let's try to understand the difference between these gateways. They both allow instances to have
external access (Internet), and this may be confusing to understand the real difference.

The first thing and the main difference is that the Internet Gateway (IGW) is attached or allocated
to a VPC while a NAT Gateway (NGW) is associated with a subnet in the VPC. A VPC can have
multiple subnets, therefore multiple IGW; however, it can not have more than 1 IGW.

There is also another difference: The IGW gives your VPC external access and the NAT gateway
routes to an IGW to access the internet. So even if a subnet has a NGW, it will be unable to reach
external hosts if the VPC does not have an IGW.

Another difference resides in the fact that IGW is nothing then a logical link to the Internet. It is
like your RJ45 cable, it allows your home to have external access, but there is no control.
However, the NGW is a machine; consider it like your home switch that you can control to
open/close ports, block web sites ..etc.

Historically, the NGW was not a managed service, you should create your own EC2 machine, but
now when you create a NAT Gateway, logically, AWS creates a managed VM for you. In all cases,
now the NGW is a managed service. To create a NGW, you should assign an Elastic IP address to
it. The routing mechanism of a NGW is done using Route Tables.


## Creating a VPC

To understand all of this, the best and the most common example I found is the following:

![1](./data/4-1.png)

_source: aws.amazon.com_

**Important Note** : The network we are building follows the same diagram but may uses different
IP addresses VPC, subnets, CIDRs..etc

Let's start by creating a new VPC. You can delete the default VPC and if you want to recreate it
later, use the "Create Default VPC" menu.

![2](./data/4-2.png)

Click on "Create VPC"

![3](./data/4-3.png)

Give the VPC a name. I name it "custom".

Provide the IPv4 CIDR block; I used `"10.0.0.0/16"` which means that I will have the ability to use all
the blocks between 10.0.0.0 and 10.0.255.255, which contains 65536 IP addresses.

I do not need a dedicated VPC; the default tenancy once is good for most use cases.

![4](./data/4-4.png)

Let's move to create two subnets; one public and one private subnet.

![5](./data/4-5.png)

The first public subnet has the name "public"; you can choose another name. I also assign it to
the AZ "eu-west-3a". If you are using another region, you can choose a different zone. Finally, for
the IP addresses block, it ranges from 10.0.0.0 to 10.0.0.255, which means 256 addresses, and it
is enough for most uses cases. Unless you are going to deploy more than 256 machines in this
subnet, you can choose another block.

Do the same thing to create another private subnet.

![6](./data/4-6.png)

We are using the CIDR 10.0.1.0/24, and this is a random choice, you can choose another CIDR, but
since the first subnet has the CIDR 10.0.0.0/24, why not choosing the next CIDR 10.0.1.0/24.

After creating two subnets, we can notice that both are using the same route table, and it is the
default route table created by AWS.

![7](./data/4-7.png)

You can click on it and rename it to "default", to avoid any confusion.

![8](./data/4-8.png)

Now, let's create a second route table, which will act as the private route table in our
infrastructure.

![9](./data/4-9.png)

The third route table will be the public one.

![10](./data/4-10.png)

Now, at this step, we have two subnets (1 private and 1 public), two route tables (+ the default
one), and we need an Internet Gateway.

![11](./data/4-11.png)

Create one, give a name tag, and attach it to the VPC.

![12](./data/4-12.png)

Let's associate the public subnet to the public route table.

![13](./data/4-13.png)

Click on "Edit subnet associations" and choose the public subnet from the list.

![14](./data/4-14.png)

Any EC2 instance in the public subnet should be able to send and receive traffic from the
Internet, and this is why the route table should allow the subnet to reach and receive traffic from
all hosts (0.0.0.0/0).

To make this happen, click on "Routes", then "Edit routes".

![15](./data/4-15.png)

Add "0.0.0.0/0" as destination and the Internet Gateway we created as the target.

![16](./data/4-16.png)

This means that if an EC2 machine in the public subnet wants to ping google, the VPC will check
its route table before and find that it can reach the Internet using the line we added to the route
table, which means that the traffic will flow through the Internet Gateway. The reverse traffic will
also flow through the Internet Gateway, then the router, and finally reach the EC2 machine.

![17](./data/4-17.png)

**Important Note** : The network we are building follows the same diagram but may uses different
IP addresses VPC, subnets, CIDRs..etc

Now, go to the private route table and associate the private subnet to it.

![18](./data/4-18.png)

Since the private subnet or the hosts in the private subnet need to access the Internet while
keeping them private (not reachable from the outside), we need to create a NAT Gateway.

![19](./data/4-19.png)

Create a new NAT Gateway, choose the public subnet, and create a new EIP (Elastic IP).

Note: When creating a NAT Gateway, keep in mind two things: It should be on the public subnet,
even if we are going to use it with the private subnet. It should have an EIP.

No go back to the route tables and edit the private subnet. We need to tell the VPC (more exactly
the private subnet route table), that any outgoing traffic requested by an instance in the private
subnet and going out to the Internet should transit through the NAT Gateway.

![20](./data/4-20.png)

Edit the routes. As a destination, use "0.0.0.0/0" and, as a target, use the NAT Gateway.

![21](./data/4-21.png)

So, any traffic outgoing traffic from the private subnet will check the route table and sees that it
should go through the NAT Gateway. The NAT Gateway will also check the route table of the
public subnet (because it is in the public subnet) and finds out that in order to send outgoing
traffic, it should use the Internet Gateway.

Private EC2 -> Router -> NAT Gateway -> Router -> Internet Gateway.

So, if you were asking this question:

If the NAT Gateway is used with the private subnet and not the public one, why is it in the public
subnet?

The answer is:

Exactly, the private subnet uses the NAT Gateway. The public subnet makes no use of the NAT
Gateway. However, since the NAT Gateway should have access to the Internet, it should be in the
public subnet.

![22](./data/4-22.png)

**Important Note** : The network we are building follows the same diagram but may uses different
IP addresses VPC, subnets, CIDRs..etc


Now, if you create an EC2 machine on the public subnet, it will be able to send and receive
outbound and inbound traffic. However, the machines in the private subnet will only be able to
send outbound traffic. In case we need to ssh into these machines, a solution would be creating a
bastion host in the public subnet and use it as an entry point to any machine in the private
subnet.

If you want to create a bastion host, you can also deactivate the SSH access to 0.0.0.0/0 in the
security group of public hosts and only allow the bastion IP address.

So, the bastion host can be a way to harden security, allowing SSH access to all machines (public
and private) only through it.

The second solution is creating a VPN.

## Using a VPN

Create a new subnet for the VPN machine. We will use a different zone here (eu-west-3c).

![23](./data/4-23.png)

Now, since the VPN should be accessible from our machines, or any other machine, we should
associate the public route table to the VPN subnet.

![24](./data/4-24.png)

To create a VPN, the most common choice is OpenVPN, let's create an EC2 machine in the VPN
subnet and install OpenVPN, then configure it.

You can choose the OpenVPN instance from the AWS Marketplace since we will have a ready-to-
use instance of OpenVPN.

I recommend using the free tier eligible AMI.

![25](./data/4-25.png)

In the instance configuration step, choose the "custom" VPC we created previously, use the "vpn"
subnet, and we don't need to assign a public IP at this step.

Public IPs are ephemeral, if we stop or shut down the instance, the IP address will be released,
and we will not be able to use it. So if you are using the VPN for a production setup, disable the
public IP, we are going to attach an EIP later. Note; As we have seen, EIP is a static IP address. It
will not change when we shutdown our VPN instance.

![26](./data/4-26.png)


If you are using the VPN to test, choose to auto-assign a public IP.

If you didn't choose to assign a public IP, when the instance is created, go "Elastic IPs" and create
a new one.

![27](./data/4-27.png)

Now, go to "Network Interfaces" and associate the EIP to the network interface of the VPN machine.

![28](./data/4-28.png)

Now, you will be able to SSH into the OpenVPN machine using:

```
ssh -i "aws-tutorial.pem" openvpnas@15.188.64.165:943/admin
```

If you did choose to assign a public IP, use the auto assigned IP.

Make sure to use "openvpnas" as a username and use the EIP with the key you chose previously
when creating the machine.

If you created the EC2 machine using the same OpenVPN AMI that I created, when you will ssh
into the machine, you would be asked to configure OpenVPN. Choose the default settings, except
the username and choose "vpnadmin" as a username or something different from "openvpn"
then set the password.

![29](./data/4-29.png)

At the end of the configuration, OpenVPN will give you the IP address to access the admin
dashboard.

If you want to reconfigure OpenVPN, use:

```
sudo /usr/local/openvpn_as/bin/ovpn-init
```

To download an OpenVPN client, visit the client page using the same address but without
"/admin".

![31](./data/4-31.png)

Click on your OS, and you will be able to read the official documentation. Download your
OpenVPN client, then go back to the same page and to download the "client.ovpn" file, click on
your profile:

![32](./data/4-31.png)

The file we downloaded helps in connecting your laptop to the VPN.

In Ubuntu, for example, install the package "openvpn" then:

```
openvpn --config client.ovpn
```

Now, to test if I can access EC2 private machine from my laptop, I created a machine on the
private subnet:

![33](./data/4-33.png)

Then using my laptop (after connecting to the VPN), I pinged the private IP of the machine:

So, if I can ping a machine with a private IP address, I'm sure that my OpenVPN settings work. You
can do the same to test.

```
ping 10.0.1.39


PING 10.0.1.39 (10.0.1.39) 56(84) bytes of data.
64 bytes from 10.0.1.39: icmp_seq=1 ttl=63 time=7.00 ms
64 bytes from 10.0.1.39: icmp_seq=2 ttl=63 time=10.8 ms
64 bytes from 10.0.1.39: icmp_seq=3 ttl=63 time=8.88 ms
64 bytes from 10.0.1.39: icmp_seq=4 ttl=63 time=10.7 ms
64 bytes from 10.0.1.39: icmp_seq=5 ttl=63 time=9.26 ms
64 bytes from 10.0.1.39: icmp_seq=6 ttl=63 time=8.14 ms
64 bytes from 10.0.1.39: icmp_seq=7 ttl=63 time=7.13 ms
64 bytes from 10.0.1.39: icmp_seq=8 ttl=63 time=9.38 ms
64 bytes from 10.0.1.39: icmp_seq=9 ttl=63 time=7.84 ms
64 bytes from 10.0.1.39: icmp_seq=10 ttl=63 time=9.87 ms
64 bytes from 10.0.1.39: icmp_seq=11 ttl=63 time=7.52 ms
64 bytes from 10.0.1.39: icmp_seq=12 ttl=63 time=10.4 ms
64 bytes from 10.0.1.39: icmp_seq=13 ttl=63 time=8.75 ms
```