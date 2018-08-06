# AWS-EC2-Getting-Started
This is a quick guide to get started with an AWS EC2 instance using an AWS TC2, Node.js, Nginx, npm, as well as two npm packages express, and pm2. 
## Technologies Used
-AWS TC2 instance: <br>
Ubuntu Server 16.04 LTS (HVM), SSD Volume Type - ami-a4dc46d 
-node.js 
https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions
-nginx 
https://nginx.org/en/docs/ 
https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04 
-npm 
https://www.npmjs.com/package/npm
-express
https://www.npmjs.com/package/express
-pm2 
https://www.npmjs.com/package/pm2
## Provisioning an EC2
-Go to your AWS console and launch a new instance. 
-Select "Ubuntu Server 16.04 LTS (HVM), SSD Volume Type - ami-a4dc46db" as your Amazon Machine Image (AMI)
-Select t2.micro as an instance type 
-Under "Configure Security Group", add HTTP and HTTPS rules. Set "Source" for all rules to "Anywhere".
-Click "Review and Launch", then click "Launch". Then create a new key pair and save it somewhere you can easily access it. You will use this to ssh into your VM. Then click "Luanch Instances". 
*Now your instance is running*
## Accessing your instance remotely
You will need to use ssh to get into your instance. If you are on a Mac, you should be good to go. If you are on Windows you will need to install ssh. Here is a link for a guide to do that: https://winscp.net/eng/docs/guide_windows_openssh_server
-cd into the file you saved your key pair in. 
-Use the chmod command to make sure your private key file isn't publicly viewable. For example, if the name of your private key file is my-key-pair.pem, use the following command: 
```chmod 400 my-key-pair.pem```
Now use the following SSH command to connect to the instance:
```
ssh -i /my-key-pair.pem ubuntu@public_dns_name
```
You will be asked "Are you sure you want to continue connecting (yes/no)?", enter "yes".
*You are now within your AWS EC2*
You will want to check for and install any updates. You can do that with this command in your EC2:
```
sudo apt-get update && sudo apt-get upgrade -y
```
## Installing Node
Install node and npm using these commands: 
```
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs
```
Check node and npm verions with these commands: 
```
node --version
```
```
npm --version
```
## Installing Nginx
Run this command to install nginx: 
```
sudo apt-get install nginx -y
```
Check status of Nginx and start it using the following commands: 
To check the status of nginx: 
```
sudo systemctl status nginx  
```
To start nginx: 
```
sudo systemctl start nginx 
```
Make sure that Nginx will run on system startup by using command below: 
```
sudo systemctl enable nginx
```
*Nginx should now be running*
## Setting up Nginx as a reverse proxy for express
Here is a link to their actual documentation:
https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/
Get the private ip address as well as the public URL of your instance by visiting your EC2 under the "Instances" tab on AWS. 
Remove and add default file to sites-available by using following commands: 
```
sudo rm /etc/nginx/sites-available/default
sudo vi /etc/nginx/sites-available/default
```
This will open the file in Vim. Now add this code to that file: 
```
server {
    listen 80;
    server_name [url for your ec2 instance];
    location / {
        proxy_pass http://[your private ip]:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
     }
}
```
***Replace [url for your ec2 instance] with your ec2 domain name and [your private ip] with the private ip address associated with your EC2***
Test the configuration of Nginx using the command below: 
```
sudo nginx -t
```
Then reload Nginx if OK is displayed with this command:
```
sudo /etc/init.d/nginx reload
```
*Nginx is now configured*
## Create a Simple Node/Express server
Inside of you EC2, run these commands to create a directory called "app", initialize npm, and install express:
*Just leave everything the way it is when you initilize npm by hitting enter for each question.*
```
mkdir app
cd app
npm init
npm i express
```
Create and open a new server.js file in VIM using this command:
```
vi server.js
```
Here is a basic express server that you can enter into this file:
```
const express = require('express');
const app = express();

app.get('/', function(req, res){
   res.send("Hello World!");
});

app.listen(8080, 'private_ip_address');
```
***Replace private_ip_address with your private ip address.***
Use the following command to run your server: 
```
node server.js
```
*Visit your EC2's domain, you should see "Hello World" now. *
## Installing pm2 and runing your Express server with it
Install pm2 with the following command: 
```
sudo npm install pm2 -g
```
Run your application using pm2 to make sure your application runs automatically when the server restarts with this command:
```
pm2 start server.js
```
***Now you have set up a reverse proxy for your Express server using Nginx. pm2 will ensure that your server continues to run even if it restarts***
***Don't forget to stop your EC2 instance when you are't using it.***
You can do this by visiting your AWS account's "Instance's" tab. Here you can see all of your instances. Right click on and instance and select "Instance State" then "stop" to stop the instance. You can start an instance again by right clicking on it and selecting "Instance State" then "Start".
## You should now be good to go!
