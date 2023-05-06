# WEB STACK IMPLEMENTATION (LEMP STACK) IN AWS
---

## What is a Technology stack?
---
A stack is an arrangement of “things” kept in order one over the other. A technology stack is a set of technologies that are stacked together to build any application. Popularly known as a technology infrastructure or solutions stack, technology stack has become essential for building easy-to-maintain, scalable web applications.

Technology stack determines the type of applications you can build, the level of customizations you can perform, and the resources you need to develop your application.

For example, a web tech stack typically looks like:

![web stack](https://github.com/Tolu4realluv/dareyio-pbl/blob/main/Project-1/web%20stack.png)

There are acronymns for individual technologies used together for a specific technology product. some examples are…
* LAMP (Linux, Apache, MySQL, PHP or Python, or Perl)
* LEMP (Linux, Nginx, MySQL, PHP or Python, or Perl)
* MERN (MongoDB, ExpressJS, ReactJS, NodeJS)
* MEAN (MongoDB, ExpressJS, AngularJS, NodeJS

And for this project, we are implemenitng LEMP stack solution.

---
## Connectint to the EC2 server through SSH
> NB: For this project, i already created an EC2 server on AWS, i just need to connect to it to start the implementation.


```ssh -i "mykey.pem" ubuntu@ec2-54-204-72-76.compute-1.amazonaws.com ```

![](https://github.com/Tolu4realluv/dareyio-pbl/blob/main/Project-2/ubuntu.JPG)

Then Update my software packages by running:

```sudo apt update ```

---

## INSTALLING THE NGINX WEB SERVER
NGINX is open-source web server software used for reverse proxy, load balancing, and caching. It provides HTTPS server capabilities and is mainly designed for maximum performance and stability. It also functions as a proxy server for email communications protocols, such as IMAP, POP3, and SMTP.

In order to display web pages to our site visitors, we are going to employ Nginx, a high-performance web server. We’ll use the apt package manager to install this package:

```sudo apt install nginx ```

To verify Nginx is running:

```sudo netstat -tulpn | grep nginx ```

![](https://github.com/Tolu4realluv/dareyio-pbl/blob/main/Project-2/Nginx.JPG)

---

## INSTALLING MYSQL

Now that we have a web server up and running, we need to install a Database Management System (DBMS) to be able to store and manage data for your site in a relational database.
MySQL is a relational database management system (RDBMS) developed by Oracle that is based on structured query language (SQL).  MySQL is a popular relational database management system used within PHP environments, so we will use it in our project.

#### To install MYSQL using ubuntu's package manager we run the following commands

```sudo apt install mysql-server```

when prompted, confirm installation by typing y

connect to the database:

```sudo mysql```

It’s recommended that we run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to our database system. Before running the script we will set a password for the root user, using mysql_native_password as default authentication method. We’re defining this user’s password as PassWord.2:

``` ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1'; ```

Then we Exit MYSQL shell:

``` exit ```

Then Start the interactive script by running:

``` sudo mysql_secure_installation ```

This asked if we'll like to set-up VALIDATE PASSWORD COMPONENT which we say NO

we can now log-in to MYSQL with the root password and run some queries:

``` sudo mysql -p ```

After successfully installing and testing mysql:

![](https://github.com/Tolu4realluv/dareyio-pbl/blob/main/Project-2/mysql.JPG)

---

## Installing PHP

We have Nginx installed to serve our content and MySQL installed to store and manage our data. Now you can install PHP to process code and generate dynamic content for the web server.

While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. we’ll need to install php-fpm, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, we’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.

To install these packages at once, we run:

``` sudo apt install php php-fpm php-mysql ```

we enter 'Y' for any prompt

To check PHP is properly installed:

``` php -v ```

![](https://github.com/Tolu4realluv/dareyio-pbl/blob/main/Project-2/php.JPG)

---

## CONFIGURING NGINX TO USE PHP PROCESSOR

When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. In this guide, we will use projectLEMP as an example domain name.

On Ubuntu 20.04, Nginx has one server block enabled by default and is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become difficult to manage if we are hosting multiple sites. Instead of modifying /var/www/html, we’ll create a directory structure within /var/www for the our_domain website, leaving /var/www/html in place as the default directory to be served if a client request does not match any other sites.

we Create the root web directory for our_domain as follows:

``` sudo mkdir /var/www/projectLEMP ```

Next, we assign ownership of the directory with the $USER environment variable, which will reference your current system user:

we know our current username by using the command;

``` whoami ```

```sudo chown -R ubuntu:ubuntu /var/www/projectLEMP ```

Then, we open a new configuration file in Nginx’s sites-available directory using our preferred command-line editor. Here, we’ll use nano:

```sudo nano /etc/nginx/sites-available/projectLEMP ```

This will create a new blank file. then we Paste in the following bare-bones configuration:
```
#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}

```

Here’s what each of these directives and location blocks do:

* listen — Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP
* root — Defines the document root where the files served by this website are stored
* index — Defines in which order Nginx will prioritize index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.
* server_name — Defines which domain names and/or IP addresses this server block should respond for. Point this directive to your server’s domain name or public IP address.
* location / — The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.
* location ~ \.php$ — This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.
* location ~ /\.ht — The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root ,they will not be served to visitors.

After completion, we save and close the file. by typing CTRL+X and then y and ENTER to confirm.

we then activate our configuration by linking to the config file from Nginx’s sites-enabled directory:

```sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/ ```

This will tell Nginx to use the configuration next time it is reloaded. we test our configuration for syntax errors by typing:

```sudo nginx -t ```

and we have:

![](https://github.com/Tolu4realluv/dareyio-pbl/blob/main/Project-2/syntax%20ok.JPG)

We also need to disable default Nginx host that is currently configured to listen on port 80, for this we run:

```sudo unlink /etc/nginx/sites-enabled/default ```

We then reload Nginx to apply the changes:

```sudo systemctl reload nginx ```

our new website is now active, but the web root /var/www/projectLEMP is still empty. We created an index.html file in that location so that we can test that our new server block works as expected:

```
sudo echo 'Hello LEMP from Toluwase' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html

```

We check this by typing our EC2 public ip address in a browser

```http://<Public-IP-Address>:80 ```

![](https://github.com/Tolu4realluv/dareyio-pbl/blob/main/Project-2/web.JPG)

## TESTING PHP WITH NGINX

At this point, our LEMP stack is completely installed and fully operational. we test it to validate that Nginx can correctly hand .php files off to our PHP processor.
we do this by creating a test PHP file in our document root. we opened a new file called info.php within our document root in our text editor:\

```sudo nano /var/www/projectLEMP/info.php ```

We then paste the following lines into the new file. This is valid PHP code that returns information about our server:

```
<?php
phpinfo(); 

```

We access this page in our web browser by visiting the domain name or public IP address we’ve set up in our Nginx configuration file, followed by /info.php:

```http://`server_domain_or_IP`/info.php ```

![](https://github.com/Tolu4realluv/dareyio-pbl/blob/main/Project-2/info%20page.JPG)

This shows that our NGINX can now handle PHP files

---

## RETRIEVING DATA FROM MYSQL DATABASE WITH PHP

In this step we will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it. We’ll need to create a new user with the mysql_native_password authentication method in order to be able to connect to the MySQL database from PHP.

We will create a database named makay_database and a user named makay.

Connecting to the msql root user:

``` sudo mysql -p ```

Creating a new Database with the command:

```CREATE DATABASE `makay_database`; ```

We then create new user and grant him full priviledge with the Database we just created:

```CREATE USER 'makay'@'%' IDENTIFIED WITH mysql_native_password BY 'password'; ```

```GRANT ALL ON makay_database.* TO 'makay'@'%'; ```


This gives the makay user full privileges over the makay_database database, while preventing this user from creating or modifying other databases on our server.

We then exit the mysql shell:

```exit ```

And then test connection to our new Database:

```mysql -u makay -p ``` 

And then check to see the databases

```show databases; ```

![](https://github.com/Tolu4realluv/dareyio-pbl/blob/main/Project-2/database.JPG)


Next, we’ll create a test table named kay_list. From the MySQL console, we run the following statement:

```
CREATE TABLE makay_database.kay_list (
    item_id INT AUTO_INCREMENT,
    content VARCHAR(255),
    PRIMARY KEY(item_id) );

```

Then we Insert a few rows of content in the test table. we repeat the command a few times, using different VALUES:

```INSERT INTO makay_database.kay_list (content) VALUES ("My first important item"); ```

```INSERT INTO makay_database.kay_list (content) VALUES ("My second important item"); ```

```INSERT INTO makay_database.kay_list (content) VALUES ("My third important item"); ```

```INSERT INTO makay_database.kay_list (content) VALUES ("my number four to-do item"); ```

```INSERT INTO makay_database.kay_list (content) VALUES ("My number five to-do item"); ```


To confirm that the data was successfully saved to our table, we run:

```SELECT * FROM makay_database.kay_list; ```

![](https://github.com/Tolu4realluv/dareyio-pbl/blob/main/Project-2/kaylist.JPG)

We can now exit mysql shell

```exit ```

Now we can create a PHP script that will connect to MySQL and query for our content. We create a new PHP file in our custom web root directory using our preferred editor. We’ll use nano for that:

```nano /var/www/projectLEMP/kay_list.php ```

And then we paste the config script that connects to the MySQL database and queries for the content of the kay_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception:

```
<?php
$user = "makay";
$password = "password";
$database = "makay_database";
$table = "kay_list";

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

We save and close the file by typiny CTRL X, Y and Enter

We can now access this page in our web browser by visiting the domain name or public IP address configured for our website, followed by /kay_list.php:

![](https://github.com/Tolu4realluv/dareyio-pbl/blob/main/Project-2/todo.JPG)

This means our PHP environment is ready to connect and interact with our MySQL server

This Shows everything is working perfectly and we have successfully deployed and configured a LEMP stack website in AWS Cloud.

