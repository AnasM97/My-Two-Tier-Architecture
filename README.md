# My-Two-Tier-Architecture

## Creating My VPC

- Went to AWS and created VPC
- Changed the IPv4 CIDR to 	10.0.0.0/16 so as to have large range of private addresses for many subnets and resources to be created without running out of IP addresses.
  - /16 = First 16 bits are fixed, rest used for hosts ->
  - /16 holds 256^2 = 65536 total available IP Addresses.

## Creating Subnets
- Created a Public Subnet with Availability Zone(AZ) = eu-west-2a
- Set the IPv4 CIDR to 10.0.1.0/24 which gives the public subnet 251/65536 in terms of IP Addresses
- AWS reserves 5 IP Addresses(First 4 and the Last IP) for :
  - Network Address: 10.0.1.0
  - VPC Router 10.0.1.1
  - DNS : 10.0.1.2
  - Future Use : 10.0.1.3
  - Broadcast Address 10.0.1.255

Creating an Internet Gateway
- Created an IGW and the connected it to the VPC

## Route Table for Public Subnet
- Found the route table attached to my VPC and renamed it to PUBLIC-RT
- Next I added an internet route and its target was the IGW I created
- Had to associate my public subnet with the route table and now anything in my public subnet can reach the internet

## Launch Web Server in Public Subnet
- Created an EC2 Instance with my AMI being Ubuntu
- Instance type = t3.micro
- In the network settings I changed the VPC and set the subnet to my Public Subnet
- Set the source type for my ssh (security group) to my IP
  - I should have used the Public IP of the instance instead of my personal IP address
- Created a new Security group for HTTP , port 80 called Web-SG
  -  Made the source type anywhere (0.0.0.0/0) so that users can access my server from the internet
- SSH into my instance from VSCode
  - Also forgot to go to my ssh directory instead I SSH from my current directory
- Installed nginx and initially tried to go onto the website from my public IP Addresses link from my instance
  - This led to a problem where I was not seeing the nginx html file so I had to check if it was running
  - I then realised that the link was https://public-ip instead of http

## Creating security group for DB Server
- Created a new SG and linked to my VPC
- I changed the inbound rules so that only instances with security group Web-SG
- The type was MySQL/Aurora , port 3306

## Launching DB Server
- Launched 2nd EC2 Instance with same AMI,Key pair and instance type
- Connected to VPC and made subnet the Private Subnet
- Made SG = DB-SG
- Disabled auto-assign public ip

# Entering into DB-Server from Web-Server and installing MySQL
- I could not ssh directly into DB-Server since it was in a private subnet
- So I had get into the DB-Server from the Web-Server
- Instead of ssh using the Public IP, I had to use the Private IP of the DB-Server.
- Initial Attempt to Enter into Private Server.
  - SSH directly into Public server but failed to SSH into Private Server.
  - My key pair was not recognised since it was storede in my local machine and not in my public server.
- Second Attempt
  - Used Agent Forwading to forward my SSH key from my laptop through the public jump server so the private server could authenticate without the public server storing the key.
  - I tried to proxy jump directly from my local machine into the DB Server with my Web Server acting as a jump host, relaying my SSH connection.
  - SSH tried to authenticate to the Web Server but the server rejected the key I provided.
- Final Attempt
  - I started an SSH Agent and loaded my private key into it
  - Forwarded my local SSH agent to the public server and enabled the public server to authenticate to the private server using my laptop’s key.
  -  The authentication request was forwarded back to my laptop’s SSH agent and then the private server was authenticated using my local key without being copied or temporarily stored in the public server.
- Attempted to install MySQL but my private server had no access to the public server.
- I had 2 choices between a NAT Gateway(paid) or to use an Elastic IP then release it afterwards so I went for the Elastic IP.
- After creating the Elastic IP and connecting to my Private Server Instance, I was able to update the server and install mysql
- Next I secured mysql by :
  - Setting a root password
  - Removing anonymous users
  - Disabling remote root login
  - Removing test database
  - Ultimately reloading the priviledge tables so the above effects are taken immediately
- By default, MySQL only listens on 127.0.0.1, so it accepts connections from the local machine only.
- Changing the bind-address to 0.0.0.0 allows it accept connections from everywhere but since it is in a private server it will only allow connections from my public web server.
- I then restarted mysql which applied the changes.

## Creating a databse and connecting to Web Server
- First I had to log into mysql as root user where I can create my MYSQL database on my private server.
- I created the database and named it devopsapp.
- I initially made my first user to connect from any host(%) but I accidently changed that to only connect from my private server which is why later I could not connect my database from my Public Web Server
- The inital password also did not fit the requirements I set earlier which was only a minimum of 8 characters
- Afterwards, I created a user who could only connect from my private database however this time my password fit the requirements.
- Consquently from the above actions, I had to drop the user and then create another user again who could access from any host.
- Even after this my web server still could not connect so I went back to the Private Server and checked the firewall only for its status to be inactive
- I had to change yet again, the user and made 2 users who can access the database one from the private server and the public server which has a level of risk but that level is much lower compared to allowing connection from any hosts.
- Now I can connect to my database from both my private and public server

## Storing Data on Web Server
- Made a new directory and went changed directory to myapp
- Created a virtual environment to install my python and keep my application code separate from the Python Ubuntu needs.
- Downloaded Flask, which is a webframework for Python, and MySQL Connector, which allows my Python program to interact with the with MySQL but specifically in this case my Database, for Python packages from Python Packages Index(PyPi).
- Installed both on my Python 3 Environment
- Created a file app.py in the nano text editor which is where my application code is stored.
- Attempted to run flask but I did not have port 5000 on my SG for the Public-Server
- 

## Commands Used
- ssh i - keypair.pem ubuntu@web-server-public-ip
- sudo apt update
- sudo apt upgrade -y 
- sudo apt install nginx -y
- sudo systemctl start nginx
- sudo systemctl enable nginx
- sudo systtmectl status nginx 
- ssh i - keypair.pem ubuntu@db-server-private-ip#
- ssh -A -J ubuntu@public-server-ip ubuntu@private-server-ip
- eval "$(ssh-agent -s)"
- ssh-add my-key.pem
- ssh -A ubuntu@public-server-ip
- ssh ubuntu@private-server-ip
- sudo apt install mysql-server -y
- sudo systemctl start mysql
- sudo systemctl enable mysql
- sudo mysql_secure_installation
- sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
- sudo systemctl restart mysql
- sudo mysql -u root -p
  - REATE DATABASE devopsapp;
  - CREATE USER 'appuser'@'%' IDENTIFIED BY 'password123'; -> Password does not meet MYSQL requirements
  - CREATE USER 'appuser'@'private-server-ip' IDENTIFIED BY 'Str0ng!Passw0rd2026';
  - RANT ALL PRIVILEGES ON devopsapp.* TO 'appuser'@'privat-server-ip';
  - FLUSH PRIVILEGES;
  - EXIT;
- New SQL
  - DROP USER 'appuser'@'10.0.0.81';
  - CREATE USER 'appuser'@'%' IDENTIFIED BY 'Str0ng!Passw0rd2026';
  - GRANT ALL PRIVILEGES ON devopsapp.* TO 'appuser'@'%';
  - FLUSH PRIVILEGES; -> Applies Changes 
  - EXIT;
- sudo apt install mysql-client -y
- mysql -h private-server-ip -u appuser -p
- FINAL SQL
  - CREATE USER 'appuser'@'Private-Server-IP' IDENTIFIED BY 'Str0ng!Passw0rd2026';
  - GRANT ALL PRIVILEGES ON devopsapp.* TO 'appuser'@'Private-Server-IP';
  - CREATE USER 'appuser'@'Public-Server-IP' IDENTIFIED BY 'Str0ng!Passw0rd2026';
  - GRANT ALL PRIVILEGES ON devopsapp.* TO 'appuser'@'Public-Server-IP';
  - SELECT user, host FROM mysql.user WHERE user='appuser'; -
      - Shows 2 appusers and 2 hosts(Private+Public)
- sudo ufw allow from private-server-ip to any port 3306 proto tcp
- sudo ufw allow from public-server-ip to any port 3306 proto tcp
- sudo ufw enable
- mysql -u appuser -p -h private-server-ip -> From the DB Server
- mysql -u appuser -p -h public-server-ip -> From the Web Server
- mkdir myapp
- cd my app
- sudo apt install python3-venv python3-full -y
- python3 -m venv venv
- source venv/bin/activate




### Keywords I Learnt
- Virtual Private Cloud(VPC) : An isolated private network inside a public cloud provider.
  - You can launch servers,databases and services with your own IP ranges,routing rules and security controls
 
- Subnets : A smaller network inside the VPC that groups resources and controls how they can communicate and whether they are reachable from the internet
  - Public Subnet : Where users from the internet can access, usually a web server.
  - Private Subnet : Typically where application code can be where there is no access given to users from the internet.
  - Database Subnet : This is for databases and absolutely should never be given public access
  - (Only using public and private for this 2-tier)
 
- Internet Gateways(IGW) : A VPC component that allows resources in public subnets to send and recieve traffic from the internet
  - Without an IGW, the webserver cannot be accessed from the browser.

- Route Tables : A set of rules in a VPC that tell network traffic where to go next
  - Network Traffic : The flow of data being sent and received between computers over a network.

- Network Address Translation Gateweay(NAT): A service that lets private subnet resources access the internet without allowing the internet to initiate connections back to them.

- Security Group : A virtual firewall attached to a resource that controls which traffic is allowed in and out based on IP, port, and protocol.

- Amazon Machine Image(AMI) : An AMI contains the operating system, application server, and applications for your instance.

- ProxyJump (ssh -J) : A way to connect to a target server through an intermediate server (jump host) in a single command.

- Agent forwarding (ssh -A) : Allows you to use your local keys on a remote server without copying the private key to that server.

- Elastic IP: A static public IPv4 address provided by AWS that you can associate with an EC2 instance.
