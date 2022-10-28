# reactWebsiteInAWSCloud
This is a tutorial on how to set up a basic React App on an AWS EC2 node, with DNS and SSL encryption.  All the code here can also be gained from an <npx create-react-app command>.  However, this README.md contains valuable information for creating a website on a vm in the cloud with proper network configurations.  If you've never used: DNS, Load Balancers, SSL Encryption, Firewalls, Virtual Private Networks (VPCs), HTTP/HTTPS Protocols, VMs, the Cloud or React.  This tutorial is an excellent guide to walk you through all those concepts in a pratical situation.
## Creating and AWS Account and the Cost
First thing's first, you'll need an aws account and yes, you will need to set up the billing section.  This whole process won't cost you more than $20, most of which will go to registering for dns, either way aws will ask you for a credit ccard which you will need to supply 
## Launching an EC2 instance
  Now that we've established an account for our cloud based server it is time to create said server.  The server we are going to create is an EC2 or Elastic Compute node which is a vm that runs in the cloud.  EC2 nodes are very practical because they automatically are created with a public ip address which will come in handy when we move on to the dns section.  First you will want to go to the aws search tab and look up EC2s *picture*.  From here you will want to click the *Launch instances* button *picture*.
  Name the node whatever you want, for the os we are using ubuntu 22.04 64-bt free tier in this tutorial, remember that for later.
  Next there is key pair section which is relevant if you want to be able to ssh to you ec2 node.  Simply create a key pair with all the default settings and it will auto download the .pem file to your local device.  Save it somewhere you'll remember on your device you won't be able to download it a second time.
  Now in *Network settings* your EC2 node will ask you to create a security group for your VPC (Virtual Private Cloud).  Watering this down your EC2 node is asking if you want some holes made in the firewall for your EC2 node, which we do.  We want to be able to:
  - ssh from anywhere (port 22 on subnet 0.0.0.0/0)
  - use http anywhere (port 80 on subnet 0.0.0.0/0)
  - use https anywhere (port 443 on subnet 0.0.0.0/0)
  These are common enough and there are checkmarks correlating to each in the config, mark them off.
  Now we are done, all the other configs can remain at the default settings.  Click *Launch Instance* and watch as your vm gets spun up.  You'll know it's ready after it passes both health checks *picture*.  You can see the node's terminal after by selecting the instance and then the *Connect* button and either Connecting directly in AWS with the *Connect* button in *Connect to instace* or sshing from your local terminal with your .pem file like this:
  *ssh -i "*something*.pem" ubuntu@*aws dns/public ip**
  In the *SSH client* under *Connect to instance* they directly give you the commands you need to use.
## Installing React and Node.js
## Creating a DNS
## Creating an SSL Certificate
## Creating a load balancer
## Keeping your React Server running as a Service
