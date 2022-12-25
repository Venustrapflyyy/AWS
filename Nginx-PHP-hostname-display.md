 ## Task 
 - Set up a stateless application on three EC2 instances running behind an elastic load balancer running Nginx web server, the application should display the hostname of the server serving content at that time. Point a subdomain to that ELB. 

## Steps 
- To start with, I created three EC2 instances in the same availability zone and installed Nginx web server on them by running `sudo apt update` then `sudo apt install nginx`. 
- I then created an application load balancer, creating a target group called 'my-tg' which contained the three EC2 instances as its targets. 
- Both instances and load balancer used the same security group which allowed SSH, HTTP and HTTPS traffic inbound and all traffic outbound, to and from the internet. 
- I installed php and php-fpm on the instances by running `sudo apt-get install php php-fpm`. 
- When I entered any of the public IP addresses of my instances, I got the Nginx default home page. 
- I then edit the NGINX configuration file by running `sudo nano /etc/nginx/sites-available/default`, which I made to contain 
```
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        index index.php index.html;

        server_name _;

        location / {
                try_files $uri $uri/ =404;

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php8.1-fpm.sock; 
```
- with every other line commented out. 
- I then replaced the `index.nginx-debian.html` and `index.html` files in` /var/www/html` with and `index.php` file which contained; 
```
<?php
$hostname = gethostname();
echo $hostname;
?>
```
- This made my server display its hostname whenever its public IP address is entered into a browser. 
- 
