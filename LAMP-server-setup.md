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
- To install SQL, I ran `sudo apt install mysql-server -y`, after which I ran this `sudo mysql_secure_installation` command to sequre the installation and followed the prompt including creating password for my root admin. 
- I created a new user called zainab with the command `CREATE USER 'zainab'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';`, then granted this user all permissions on all databases using `GRANT ALL PRIVILEGES ON *.* TO 'sammy'@'localhost' WITH GRANT OPTION;`. 
- I ran the `FLUSH PRIVILEGES;` command to free up any memory that the server cached as a result of the preceding CREATE USER and GRANT statements. 
- I then ran `sudo apt install php libapache2-mod-php php-mysql` to install PHP and its dependencies. 
- I logged into MySQL as zainab by running `mysql -u zainab -p`, then I created a database called "example_database" with `CREATE DATABASE example_database;`. I thereafter created a table in the database with 
```
CREATE TABLE example_database.todo_list (
	item_id INT AUTO_INCREMENT,
	content VARCHAR(255),
	PRIMARY KEY(item_id)
);
```
- Then I inserted a few rows of content inside the table by running `INSERT INTO example_database.todo_list (content) VALUES ("My first important item");`, `INSERT INTO example_database.todo_list (content) VALUES ("My second important item");` etc. To see the conent of my table, I ran `SELECT * FROM example_database.todo_list;`, after which I exited the MySQL console. 
- I also created a PHP script by running `nano /var/www/html/todo_list.php`, this contained 
```
<?php
$user = "zainab";
$password = "******";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>"; 
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
```
- When I entered `http://my_ip_address/todo_list.php` in my browser, I got ![todo list screenshot](https://github.com/Venustrapflyyy/AWS/blob/main/todo.png)
