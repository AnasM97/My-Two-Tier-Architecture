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

## 





## Commands Used








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
