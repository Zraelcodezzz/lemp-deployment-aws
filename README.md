# lemp-deployment-aws
This repository contains a step by step guide for deploying a LEMP stack (Linux, eNginx, MySQL, PHP) stack on Amazon Web Services (AWS)

REQUIREMENTS

EC2 instance

GitBash(Optional)

**STEP O : EC2 INSTANCE CONNECTION**

 Firstly,we SSH into our vitual machine

          ssh -i 'keyfile.pem' ubuntu@'instanceIP'

   ![image](https://github.com/user-attachments/assets/843e0994-5718-4642-872d-f4984f4ce26e)
   
**STEP 1 : NGINX SERVER INSTALLATION AND CONFIGURATION**

i. Being a new session on this instance,let's update our server service index using the command

       sudo apt update

   ![image](https://github.com/user-attachments/assets/242c42ae-8e66-4b22-a855-eca4b424d14c)     

 ii. Afterwards we go ahead to install Nginx    

       sudo apt install nginx

  ![image](https://github.com/user-attachments/assets/dc8de3cc-438d-4d55-b764-7821f6aa02a6)

iii.  Now let's verify Nginx status 

          sudo systemctl status nginx
          
![image](https://github.com/user-attachments/assets/a5141133-0037-477f-b4b0-729d06cd7db2)

ensure it's green and running.

iv.  Next we dash to our ec2 console and configure our ec2 to open connection via port 80  

![image](https://github.com/user-attachments/assets/bf9cbe01-8ac7-45b2-aae2-b62aaccd6752)

now let's use the curl command to access the port locally

          curl http://localhost:80
          
![image](https://github.com/user-attachments/assets/bcdc665d-510e-4b80-ab2a-60396dea374f)

that's a confirmation that our server is responsive to the curl command via terminal

then let's be sure it's sure it's responsive over the internet via our browser

                 http://'IP address':80

![image](https://github.com/user-attachments/assets/5243f449-5833-4f32-8cbf-b8a400c2cba6)

**STEP 2 :MYSQL DOCUMENTAION**

i.  Use the command

                sudo apt install mysql-server -y

  ![image](https://github.com/user-attachments/assets/952cd898-fc3e-4e6c-8f57-e44608396a01)

ii.  Next we log into mysql as a user root using sudo

              sudo mysql
  Before we run a script let's set a password for our database

  ![image](https://github.com/user-attachments/assets/7873aea8-0064-492e-abda-cf4f1b33242f)

  NOTE:Be careful with spacing as the mysql syntax is sensitive with spaces.

  Then exit aftr confirmation.

iii.   Now let's start our script using the command

             sudo mysql_secure_installation

  enter password as requested.
iv.   We're then asked for password validation( this is optional but let's see it out)

![image](https://github.com/user-attachments/assets/485af770-a433-4efd-bdf1-c5606c89875a)

Follow as shown.

v. Let's try to login using our newly set password.

            sudo mysql -p

 ![image](https://github.com/user-attachments/assets/42f74e3c-7973-49d9-ad32-1ad8231f97ee)



 ** STEP 3: PHP INSTALLATION**
Unlike in LAMP stack where apache enables the PHP in each request,Nginx operates differently by using an external program to process PHP which as  a mode of interaction between PHP and itself.This difference impacts better perfomance in PHP based websites.So in our installation we're going to be getting things configured by getting the additional programs and modules on board. Let's go!

 i. Firstly let's install these packages.

             sudo apt install php-fpm php-mysql

  ![image](https://github.com/user-attachments/assets/5e5d00ee-b46c-4736-b5c3-117f5b9c6efb)

**STEP 4: CONFIGURING NGINX TO USE PHP PROCESSOR**

   Just like we create virtual hosts  in Apache,we'll create server blocks in Nginx to key in our configuration details and host more than one domain on a single server.By default Nginx serve documents out of /var/www/html which is fine for single sites but for hosting muliple sites on this server,it's better to create a sub-directory rather than modifying tjhat default directory.We don't like stress right?
   
   Now let's cook!

   i. We'll create a new web directory in our domain 

               sudo mkdir /var/www/projectLEMP

  ii. Then we assign ownership of our directory with our identity as the $USER 

             sudo chown -R $USER:$USER /var/www/projectLEMP

 iii.   Next we open a new configuration file in Nginx's sites-available directory using your preferred text-editor

         sudo vim /etc/nginx/sites-available/projectLEMP
         
Then we paste in this configurationinto our file.

       server {
    listen 80;
    server_name projectLEMP wwww.projectLEMP;
    root /var/www/projectLEMP;
    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    # Pass PHP scripts to FastCGI server
    
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    }

    # Deny access to .htaccess files
    location ~ /\.ht {
        deny all;
    }
    }

  ![image](https://github.com/user-attachments/assets/fe579353-6b21-415c-8ef6-dafc8a97910f)

 listen 80;: Your server will listen on port 80 for HTTP traffic.
 
root /var/www/projectLEMP;: Specifies the root directory for your project files.

index index.html index.htm index.php;: Specifies the default index files.

server_name directive defines which domain names or IP addresses Nginx will respond to.

The location blocks define how different kinds of requests are handled by Nginx. These blocks specify URL patterns and tell Nginx what to do when those patterns are matched. Below are the common location blocks used in your configuration.


iv. Let's link access the config file from our sites-enabled directory

      sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

v. Now we unsure our configurations is valid  using 

      sudo nginx -t

  ![image](https://github.com/user-attachments/assets/f7016383-81cc-425b-aeaa-e3234c0d6755)

  if it's not okay ,go back to the file then check and correct errors.

vi.   Next we need to disable the default nginx host that is configured to listern on port 80

           sudo unlink /etc/nginx/sites-enabled/default

  afterwards we reload our server to update the changes

           sudo unlink /etc/nginx/sites-enabled/default


vii.   Our site is up and running but /var/www/projectLEMP is empty.Let's go ahead and create an index.html file to test out our new server block.


           sudo echo 'Hello LEMP from hostname' $(TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` && curl -H "X-aws-ec2-metadata-token: $TOKEN" -s 
          http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` && curl -H "X- 
          aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html

![image](https://github.com/user-attachments/assets/883c21db-01c4-40d6-b913-39715daf4ea7)

![image](https://github.com/user-attachments/assets/7f250d27-ab72-4d85-9251-b88c83d2c809)

Now let's try it out with our public DNS name

![image](https://github.com/user-attachments/assets/43e8914b-397b-4bdf-b726-84cd4d27f2cc)


**STEP 5: PHP TEST WITH NGINX**

i. Let's create a php test file in our webroot

       vim /var/www/projectLEMP/info.php

 and input

         <?php
          phpinfo();

![image](https://github.com/user-attachments/assets/193282d5-4cfc-4f59-bb36-ecdde9260d91)

ii. Now let's acess this specific page on our browser.

  ![image](https://github.com/user-attachments/assets/3d8521d3-8928-486c-8a3e-abcfa85b08aa)

iii. We then remove the php file as it includes sensitive information.We can get it back up whenever needed using the mentioned steps.

      sudo rm /var/projectLEMP/info.php
      
As expected...
   ![image](https://github.com/user-attachments/assets/eebde94f-a5a8-49c4-ab2b-e1ac4ab2d30d)


  **STEP 6: DATA RETRIEVAL FROM MYSQL WITH PHP**

  Lastly on this project we're going to create a test database(DB) with simple "To do" list so our Nginx website will be able to request data from the database and pass it out on the site.

  We'll need to create a new user with the mysql_native_password authentication in order to correct to the MySQL database from PHP reaoson is MySQl PHP library mysqlnd doesn't support caching_sha2_authentication which is the default authentication method for MySQL 8.
  
i. To start with,connect to the MySQL console using the root account:

         sudo mysql -p
ii. Then

          mysql> CREATE DATABASE 'example_database';

iii. Let's grant privileges to our user on the database

    CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
    
iv.  Next we need to give user permission over there example_database we created.

            GRANT ALL ON example_database.* TO 'example_user'@'%';


now exit the shell 

  ![image](https://github.com/user-attachments/assets/7105a090-2188-40ac-9e9d-eaf41a19aa21)


v.    Let's test if the user has proper permissions by logging into the MySQL console again

         mysql -u example_user -p
       

![image](https://github.com/user-attachments/assets/7140e631-4898-4827-8cc4-08e1971f2390)

vi. Now let's access our database

             SHOW DATABASES;
             
We get this table output

  ![image](https://github.com/user-attachments/assets/5ea80041-c558-45e0-804c-ff9758a38e49)

vii. Now let's create a test table named todo_list by running the following commands

            CREATE TABLE example_database.todo_list (
            item_id INT AUTO_INCREMENT,
            content VARCHAR(255),
            PRIMARY KEY(item_id)
            );
  vii. Let's try to insert something in our table

              INSERT INTO example_database.todo_list (content) VALUES ("Let's not break this table");
              
  viii. To confirm that our data input was saved.

               SELECT * FROM example_database.todo_list;

   ![image](https://github.com/user-attachments/assets/f837a821-9e33-43c0-86e9-68777b7fe9e9)

   exit the mysql console afterwards.

   ix.  To end with,let's create a PHP script that will connect to mysql and query for content.We'll be doing this in our web root directory and Yes!I'll be using vim

                  vim /var/www/projectLEMP/todo_list.php

   Then our script...


                                 
         <?php
          $user = "example_user";
          $password = "PassWord.1"; // Updated password
          $database = "example_database";
          $table = "todo_list";

          try {
          $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
           $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

           echo "<h2>TODO</h2><ol>";

           $stmt = $db->query("SELECT content FROM $table");

          foreach ($stmt as $row) {
           echo "<li>" . htmlspecialchars($row['content']) . "</li>";
           }

           echo "</ol>";
            } catch (PDOException $e) {
             echo "Error!: " . $e->getMessage() . "<br/>";
            die();
             }
             ?>






   ![image](https://github.com/user-attachments/assets/7a21082e-e776-4126-ac99-96167f0b85e5)

   x. We dash to our browser to see the content


  ![image](https://github.com/user-attachments/assets/faec594c-1199-4c42-83f1-df66661248ee)


  Cheers!


   

  
           
   

      




           


  



             

 
          




  
  

                



       


          
     


   
        
