## Project 
- Set up a LAMP stack on an AWS instance running Ubuntu 22.04, configuring a domain name and securing this to allow SSL traffic. 

## Steps 
- To begin, I logged into my AWS console as an admin user, after which I launched an EC2 instance in the eu-west-2b region. During the launch, I created a key pair named 'example-keypair', which I downloaded onto my computer. 
- I created a security group which allowed ports 22, 80, 443, 3306, and 3389 from the internet '0.0.0.0/0' 
- From a vagrant machine on my laptop, I copied the downloaded key pair from my downloads folder to the folder that contained my vagrant file (/vagrant), then I further copied it to `/home/vagrant`, after which I entered this command;
`ssh -i "LAMP-lab-server-keypair.pem" ubuntu@ec2-18-133-230-100.eu-west-2.compute.amazonaws.com` to SSH into my AWS instance. 
- Once I logged into the instance, I ran `sudo apt update`, `sudo apt-get update` and `sudo apt upgrade` to update the package cache, and upgrade packages to the new version. 
- I ran `sudo apt install apache2 -y` to install apache. 
- On my browser, I typed in `http://18.133.230.100` in got the default apache page. 
- To install SQL, I ran `sudo apt install mysql-server -y`, after which I ran this `sudo mysql_secure_installation` command to sequre the installation and followed the prompt. 
- I created the 
