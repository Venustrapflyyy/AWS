## Task

- Set up 2 EC2 instances on AWS(use the free tier instances) and deploy an Nginx web server on these instances(you are free to use Ansible). Set up an ALB (Application Load balancer) to route requests to your EC2 instance, making sure that each server displays its own Hostname or IP address. You can use any programming language of your choice to display this. 
##### Note 
- We should not be able to access the web servers through their respective IP addresses, access must be only via the load balancer. A logical network should be defined on the cloud for your servers. The EC2 instances must be launched in a private network and should not be assigned public IP addresses. 



## Steps 

- To carry out this project, I started by creating a VPC in the region London, with the CIDR block `20.0.0.0/16`. 
- Then I created a private and a public subnet each in two availability zones A and B. 
