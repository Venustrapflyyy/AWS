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

- I launched two instances in the privates subnets A and B. They did not have a public IP address, only a private IP address of `10.0.3.247` and `10.0.3.120` respectively. The security group for these instances allowed SSH traffic from the CIDR block of public subnet A (`10.0.0.0/24`), allowed all traffic from the security group of the public subnet and allowed all traffic from the security group of the NAT instance. 
. These private instances could not be accessed directly over the internet so I;
- launched an instance in public subnet A, to serve as a jumpbox, through which my private instances can be accessed. the security group of this instance allowed SSH and all traffic from the internet. 
- To SSH into my private instances, I first SSH-ed into my public instance using `ssh -i "my-keypair.pem" ubuntu@x.x.x.x`. using the private key on my vagrant machine to authenticate the public key on the instance. 
- I then created a file on the public instance with the name `my-keypair.pem`, containing the private key to authenticate the public key contained in the instance. The name of the private key file is `my-keypair.pem`. 
- To log into the private instances, I ran `ssh -i "my-keypair.pem" ubuntu@10.0.3.247` and `ssh -i "my-keypair.pem" ubuntu@10.0.3.120` separately, to log into the individual instances. 
- In the private instances, to confirm that I had internet access, I ran `curl http://www.dr-chuck.com/page1.htm` and got a response of 
```

```I updated my apt repository by running `sudo apt update`. 
- Then I installed Nginx by running `sudo apt install Nginx`.
- I also installed php.fpm by running `sudo apt install php.fpm`. 
- I ran `sudo systemctl enable Nginx` to allow my Nginx service to restart everytime I reboot my instance. 
- I ran `sudo systemctl enable php.fpm` to allow my php.8.1fpm service to restart everytime I reboot my instance. 
- I then ran `sudo nano /etc/nginx/sites-available/default` to edit my Nginx configuration file, which I made to contain 
```
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        root /var/www/html;
        index index.php index.html;
        server_name _;
        location / {
                try_files $uri $uri/ =404;
        }
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        }
}
``` 
- with every other line commented out. 
- I ran `sudo nginx -t` and got a response saying that my syntax test was successful. 
- Then I ran `cd  /var/www/html` after which I  created a php file by running `sudo nano index.php`, with its content; 
```
<?php
$hostname = gethostname();
echo $hostname;
?>
```
