## Task

- Set up 2 EC2 instances on AWS(use the free tier instances) and deploy an Nginx web server on these instances(you are free to use Ansible). Set up an ALB (Application Load balancer) to route requests to your EC2 instance, making sure that each server displays its own Hostname or IP address. You can use any programming language of your choice to display this. 
##### Note 
- We should not be able to access the web servers through their respective IP addresses, access must be only via the load balancer. A logical network should be defined on the cloud for your servers. The EC2 instances must be launched in a private network and should not be assigned public IP addresses. 



## Steps 

- To carry out this project, I started by creating a VPC in the region London, with the CIDR block `10.0.0.0/16`. 
- Then I created a private and a public subnet each in two availability zones A and B, where public subnet A, private subnet A, public subnet B and private subnet B have CIDR blocks of `10.0.0.0/24, 10.0.1.0/16, 10.0.2.0/16 and 10.0.3.0/16` respectively. 
- I then created an Internet gateway which I attached to this VPC  
- I created a public route table that was routed to the internet gateway in addition to being defaultly routed to the local network. Then I attached this route table to public subnets A and B. 
- I launched an instance in public subnet A, which has a security group that allows `http, https and all ICMP/IPv4` from both private instances A and B's CIDR blocks (`10.0.1.0/16` and `10.0.3.0/16`). This instance performs the role of network address translation for the instances in my private subnets, allowing them access to the internet.
- I also created a private route table that is defaultly routed to the local network and to the internet via the NAT instance. Then I attached this route table to private subnets A and B. 
![private route table]()
- 
