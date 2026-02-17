# My-Two-Tier-Architecture



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
