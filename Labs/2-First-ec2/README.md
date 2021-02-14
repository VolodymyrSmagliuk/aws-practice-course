# Lab 2 - Create your First EC2 Virtual

# Machine

## What is a Virtual Machine?

Let's say you have a computer with a good hardware configuration: (e.g., 16GB DDR3, SSD drives,
and a very good ). You want to use this machine as a server, so you connect it to the Internet, you
already have a static IP address, and you will host your customer small business' websites using
the LAMP stack (Linux, Apache, MySQL, PHP).

This server, with its actual configuration (16GB DDR3 memory ..etc.), could probably host
hundreds if not thousands of small websites; you can actually use it for many customers.
However, each customer would love to have an isolated environment with his own LAMP (Linux operating system, 
Apache web server, MySQL database and PHP) stack, files, and configurations. 
This is when virtualization helps in the pooling of physical resources.

Virtualization is a way to partitioning one physical server into several virtual servers, or machines.
These machines are called guests and will share the same hardware as the host server.

Every virtual machine acts like a "normal" machine; it can be accessed separately, it can have an
IP address ..etc.

Using a hypervisor, a system engineer could create multiple machines using a single physical one.


- VMware Workstation
- VMware Player
- VirtualBox
- Parallels Desktop for Mac, 
- QEM, 
- Xen, 
- Oracle VM Server for SPARC, 
- Oracle VM Server for x86,
- Microsoft Hyper-V
- VMware ESX/ESXi

All of the above are examples of hypervisor technologies used in the IT industry. Virtualization
saves money and resources and makes software installation and system administration easier..
and if you care about the environment, it can be considered an eco-friendly technology.

## Virtualization in AWS

AWS has data centers around the globe, working together to deliver a global cloud computing
service. The data centers are no more than physical machines. To create VMs, Amazon uses Xen
virtualization technology.

To quote the AWS Security Whitepaper, Amazon EC2 currently utilizes a highly customized version
of the Xen hypervisor, taking advantage of paravirtualization (in the case of Linux guests).

