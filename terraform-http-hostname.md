## Task 
- Using Terraform, create 3 EC2 instances and put them behind an Elastic Load Balancer. Make sure the after applying your plan, Terraform exports the public IP addresses of the 3 instances to a file called host-inventory. Get a .com.ng or any other domain name for yourself (be creative, this will be a domain you can keep using) and set it up with AWS Route53 within your terraform plan, then add an A record for subdomain terraform-test that points to your ELB IP address. Create an Ansible script that uses the host-inventory file Terraform created to install Apache, set timezone to Africa/Lagos and displays a simple HTML page that displays content to clearly identify on all 3 EC2 instances. Your project is complete when one visits terraform-test.yoursdmain.com and it shows the content from your instances, while rotating between the servers as your refresh to display their unique content.


## Steps 
- I logged into my vagrant machine, which had the private key of my keypair 
- I ran `sudo apt update` `sudo apt install -y net-tools` `sudo apt install -y git` `sudo apt install -y unzip` to update and install necessary packages. 
- I then installed terraform on my machine by running `$ wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg` `$ echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list` and `$ sudo apt update && sudo apt install terraform -y`. 
- To confirm the installation, I checked the version of terraform running by running `terraform -v` 
- I also installed AWS CLI for linux 64bit OS by running `curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"` `unzip awscliv2.zip` `sudo ./aws/install`  
- I then configure AWS CLI to hold my access key and secret access key by running `aws configure`, after which i entered the respective keys as prompted. 
- I also installed Ansible by running `sudo apt update` `sudo apt install software-properties-common` `sudo add-apt-repository --yes --update ppa:ansible/ansible` `sudo apt install ansible -y`. 
- I created a directory called `terraform` and in this directory, I created a `main.tf` and a `variable.tf` file, which contains my terraform structure and related variables. 
- main.tf
```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "~>4.16.0"
    }
  }
}

provider "aws" {
  region = "eu-west-2"
}

#creating my first instance
resource "aws_instance" "web1" {
  ami           = "ami-0d09654d0a20d3ae2"
  instance_type = "t2.micro"
  key_name = var.keypair
  vpc_security_group_ids = var.security_group
  subnet_id = var.subnet[0]
  associate_public_ip_address = true

  tags = {
    Name = "web1"
  }

  provisioner "local-exec" {
    command = "echo ubuntu@${self.public_ip} >> /home/vagrant/ansible/host-inventory"
  }
}

#creating my second instance
resource "aws_instance" "web2" {
  ami           = "ami-0d09654d0a20d3ae2"
  instance_type = "t2.micro"
  key_name = var.keypair
  vpc_security_group_ids = var.security_group
  subnet_id = var.subnet[1]
  associate_public_ip_address = true

  tags = {
    Name = "web2"
  }

  provisioner "local-exec" {
    command = "echo ubuntu@${self.public_ip} >> /home/vagrant/ansible/host-inventory"
  }
}

#creating my third instance
resource "aws_instance" "web3" {
  ami           = "ami-0d09654d0a20d3ae2"
  instance_type = "t2.micro"
  key_name = var.keypair
  vpc_security_group_ids = var.security_group
  subnet_id = var.subnet[2]
  associate_public_ip_address = true

  tags = {
    Name = "web3"
  }

  provisioner "local-exec" {
    command = "echo ubuntu@${self.public_ip} >> /home/vagrant/ansible/host-inventory"
  }
}

#creating my load balancer target group
resource "aws_lb_target_group" "tf-lb-tg" {
  name     = "tf-lb-tg"
  target_type = "instance"
  port     = 80
  protocol = "HTTP"
  vpc_id = "vpc-057be8ef7f8346de3"

  tags = {
    Name = "tf-lb-tg"
  }

  #creating its health check
  health_check {
    path                = "/"
    protocol            = "HTTP"
    matcher             = "200"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }
}

#attaching my target group to my instances
resource "aws_lb_target_group_attachment" "tf1" {
  target_group_arn = aws_lb_target_group.tf-lb-tg.arn
  target_id        = "${aws_instance.web1.id}"
  port             = 80
}

resource "aws_lb_target_group_attachment" "tf2" {
  target_group_arn = aws_lb_target_group.tf-lb-tg.arn
  target_id        = "${aws_instance.web2.id}"
  port             = 80
}

resource "aws_lb_target_group_attachment" "tf3" {
  target_group_arn = aws_lb_target_group.tf-lb-tg.arn
  target_id        = "${aws_instance.web3.id}"
  port             = 80
}

#attaching my target group to my load balancer
resource "aws_lb_listener" "tf-listener" {
  load_balancer_arn = aws_lb.tf-lb.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.tf-lb-tg.arn
  }

   tags = {
    Name = "tf-listener"
  }
}

#creating a load balancer 
resource "aws_lb" "tf-lb" {
  name               = "tf-lb"
  load_balancer_type = "application"
  internal           = false
  security_groups = var.security_group
  subnets            = [var.subnet[0], var.subnet[1], var.subnet[2]]
  ip_address_type = "ipv4"
  enable_deletion_protection = false

  tags = {
    Name = "tf-lb"
  }
}


locals {
  defaults = ["terraform-test", "zainabakinlawon.me"]
}

resource "aws_route53_zone" "zainabakinlawon" {
  name = local.defaults[1]

  tags = {
    Name = local.defaults[1]
  }
}

resource "aws_route53_record" "terraform-test" {
  zone_id = aws_route53_zone.zainabakinlawon.zone_id
  name    = "${local.defaults[0]}.${local.defaults[1]}"
  type    = "CNAME"
  ttl     = "5"
  records = ["${aws_lb.tf-lb.dns_name}"]
}
```


- variable.tf

```
variable "keypair" {
  type = string
  sensitive = true
}

variable "subnet" {
  type = list(string)
  default = [ "subnet-04e1c98ebc020002c", "subnet-08901d5a577f56c14", "subnet-0c8b3a197bb7f86df" ]
}

variable "security_group" {
    type = list(string)
    default = [ "sg-07f9193d97ec78c94" ]
}
```


- I created a directory called `ansible` which had the following files; 

- playbook.yml
```
- hosts: all
  become: true
  tasks:

  - name: update and upgrade the servers
    apt:
      update_cache: yes
      upgrade: yes

  - name: install apache2
    tags: apache, apache2, ubuntu
    apt:
      name:
        - apache2
      state: latest 

  - name: Start Apache Service
    service:
      name: apache2
      state: started

  - name: Enable The Apache2 Service
    service:
      name: apache2
      enabled: yes

  - name: set timezone to Africa/Lagos
    tags: time
    timezone: name=Africa/Lagos

  - name: Change Apache Configuration File State
    file:
      path: /etc/apache2/sites-available/000-default.conf
      state: absent

  - name: Setup Configuration file
    copy:
      content: |
        <VirtualHost *:80>
        ServerName zainabakinlawon.co.uk
        ServerAlias terraform-test.zainabakinlawon.co.uk
        ServerAdmin akinlawonjoyz@gmail.com
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
      dest: /etc/apache2/sites-available/000-default.conf
    register: mySite

  - name: print hostname on server
    tags: printf
    shell: echo "<h1>This is my server name $(hostname -f)</h1>" > /var/www/html/index.html

  - name: restart apache2
    service:
      name: apache2
      state: restarted

```

- ansible.cfg
```
[defaults]
inventory      = host-inventory
private_key_file = /home/vagrant/my-keypair.pem
ansible_user = ubuntu
host_key_checking = false

```


- 
