# reactWebsiteInAWSCloud
This is a tutorial on how to set up a basic **React App** on an **AWS EC2** node, with **DNS** and **SSL** encryption.  All the code here can also be gained from an ```npx create-react-app``` command.  However, this README.md contains valuable information for creating a website on a **VM** in the **Cloud** with proper network configurations.  If you've never used: 
- DNS
- Load Balancers
-  SSL Encryption
- Firewalls
-  Virtual Private Networks (VPCs)
- HTTP/HTTPS Protocols
-  VMs
- Cloud based Architechure
- React/Node.js/npm
This tutorial is an excellent guide to walk you through all those concepts in a pratical situation.
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
  Now we have an server created with all the proper access and network configs it's time to install React and Node.js.  Node.js is tempermental in aws vms so you can't install it the typical way, you need to install the Node.js binary directly.  This git repo has directions for how to install the bianary code for serveral linux os https://github.com/nodesource/distributions, the one we are interested in is Node.js v16.x for Ubuntu, as at this time it has good support.  Run these 2 commands from the README.md 
  - curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash - &&\
  - sudo apt-get install -y nodejs
  After that you need to install npm (Node.js package manager) with:
  - npm install
  And it doesn't hurt to run an apt-update and apt-upgrade after for good measure:
  - sudo apt-get update && apt-get upgrade -y
  Node run an init:
  - npm init
  And create your first react app with:
  - npx create-react-app *app name*
  Cd into the new app folder you've just created and you should be able to see you react app running with a start command:
  - npm start
  To see it run on the front end we first need to redirect the react code to forward to port 80 instead of 443.  This command should do the trick:
  - sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 3000
 Now if you run *npm start* and take the public ip from your EC2 node (you can find it when you look at the instance's summary dashboard), put it in your browser you should see your web server's running *picture*
## Creating a DNS
  Congratulations!!! You have a web server up, now it's time to add DNS to it.  DNS or Domain Name System is basically a way to tie a word name like *www.mywebsite.com* to your server so users can go to it rather than having to remember an ip adress like *54.124.124.4*.  There are several DNS providers out there and almost all of them you have to pay for, at least if you want you website to end in *.com*.  The one we are going to be using is in AWS itself called *Route 53*.  Simply query for it in your AWS search bar.  Once your there look in the side bar menu for *Registered domains*, the next dashboard should have a prompt to register for a domain.  It will ask you to choose a domain name, if the name is already taken Route 53 will run a quick check and inform you.  Once you've landed on a name add it to your cart and pay for it.  After you've purchased your domain name you need to give it time to propogate out, wait 10-15 min, or until your domain moves from *Pending requests* to "Registered domains*.
  Now you have a registered domain, but you need to tie that back to your EC2 node.  For that we need to go to our *Hosted zones* which are also located in *Route 53*. When you registered your domain it should have autogenerated a hosted czone for you with the same name as your website.  If you open your hosted zone you should see it contains 2 records: a NS and an SOA.  The NS ties you to your DNS and the SOA is required by default on your hosted zone.  In order for us to tie our hosted zone to our EC2 node we need to create an *A record* this ties an IPV4 address to a domain name.  It should be straight forward simply create a record of type A and tie the *public IP* of your EC2 node in as the value.  It should look like this *picture*.  After you've submitted your record you can test to see that dns is working but typing the name in the brower.  If it looks the same as when you type in the public ip directly you are golden *french hand*.
## Creating an SSL Certificate
  Another Huge Win! You have a website with DNS! Now it's time to make it secure and that starts with HTTPS and certificates.  You can create your own certs but in order for them to be trusted you have to it vetted by a reputable cert authoritiy.  The easiest way to do this in AWS is to go through ACM (Amazon's Certificate Manager) which you can query for in AWS's search bar.  Once there you should see a side bar tab to *Request certificate*, select that option and from there you're going to want to request a public certificate.  Make sure to give it the same name as your dns and request *DNS validation*.  Afterwords it should be pending for validation, to truly validate your cert for DNS a *CNAME record* need to be added to your *Hosted Zone*.  As both *ACM* and *Route 53* are in AWS there is a feature to *Create record in Route 53* in the certficate if you select it *picture*.  After the record is create wait a couple minutes for validation, ACM will update with an *Issued* status to should it's been linked to your Domain *picture*.
## Creating a load balancer
  Almost done, you have website up and cert made but you can't use https still! Why!? Well simply its because AWS EC2 nodes are not capable of handling certs typically, the infrastructure just isn't there.  The work around for this is a load balancer, AWS load balancers are equipped to handle certificates as well as other nice features such as redirects which we will also employ for a bit of polish :).  To create a load balancer go to the EC2 section of AWS it should be listed in the side bar menu.  Once there select *Create Load Balancer*, you're going to want to create an *Application Load Balancer* so select that one. For *Basic configuration* name it something related to your cert or EC2 node like *website0lb* and we want it to be *Internet-facing* for *IPv4*. For the *network mapping* section we need revisit your EC2 node and find 2 pieces of info: the VPC ID and the zone your EC2 node sits in. You can get the *VPC ID* and  *Availability Zone* from the *Networking* section of you *EC2 instance summary*.  You can add more zones to your load balance but make sure the one your EC2 sits in is part of it. Next in the *Security groups* section you can pull the security group id on your EC2 Node from the *Security* section of the summary.  For *Listeners and routing* make sure to have 2 listeners: one for HTTP (80) and one for HTTPS (443). 
  At this time we are going to do something a little fancy, we are going to:
  - port forward all our HTTP traffic to HTTPS (it's a nice touch for security :french hand)
  - tie our EC2 node to the HTTPS listener via a target group 
  A target group is a nice bridge tool AWS has between Load Balancers and EC2 and yes it is necessary.  To create one while in your Load Balancer menu select the *Create target group* hypertext in the HTTPS listener section *picture* a new tab will open, don't worry about loosing anything.  For our target group our *target type* is *instances* and the name is whatever you want try to make it something related like *website-tg*.  Make sure to switch HTTPS to HTTP, it'll come in handy for port forwarding later.  You can double check the VPC ID but it should be the same as the load balancer and all other default info should be correct at this time, select *Next*.  The next step is to tie your EC2 Node to this target group, it will be listed with it's instance id under *Available instances*, now click *Create target group*
  Now back in our Load Balancer hit the refresh button and our target group should appear add it to the HTTPS and HTTP Listeners.  Next in the *Secure listener settings* add your certin the *Select a certificate* box all the other default settings should be correct, hit *Create load balancer*.
  At this point we have a lovely node balancer with partial ties to our EC2 node but to complete the config we need to reconfigure our *A record* in our *Hosted Zone*.  A load balancer removes your website's dependance on an IP address and moves it to the load balance and our hosted zone needs to reflect that.  To reconfigure the *A record* go into our hosted zone and select the *A record* and edit it.  There should be a radio button that says *Alias* under *Record type*, slide it to the on position.  In the *Route traffic to* section look for the *Application Load Balancer* option, select it, *Application Zone* of your EC2 instance and name of your load balancer and then save.  While you wait for that to propogate go back to your load balancer so we can reconfigure HTTP to redirect to HTTPS.  Select your load balancer and go to the listeners section.  Select your HTTP listener and go to edit it.  Remove the *Forward to* section and when the *Default actions* section prompts you to *Add action* instead of selecting *Forward* click *Redirect*.   In your Redirect the *Protocol* should HTTPS and the *Port* should be 443, click *Save changes*.  
  To test that your load balancer and redirect configurations have be set correctly.  Simply type your website name into the browser (with HTTP if you wish) and your website should appear under HTTPS with locked icon that says your connection is secure *picture*.
## Keeping your React Server running as a Service
  Now you have your website up, but the question now is how to keep it running once the terminal closes.  To keep a process running without user interaction, the process needs to be backrounded, and to keep it running even with the prospect of the process crashing as web servers tend to do, we need to turn it into a service.  There are several softwares to do so but we are going to use *systemd*.  To create a service in systemd we need to create a service file like this:
  - sudo vim /lib/systemd/system/*websitename*.service
  The contents of which should like this, simply switch out the path of your web server:
  - *[Unit]
After=network.target
 
[Service]
Type=simple
User=root
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/npm start
Restart=on-failure
 
[Install]
WantedBy=multi-user.target*
  Now you need reload *systemd*:
  - systemctl daemon-reload
  And start and enable our new service:
  - systemctl start *websitename*
  - systemctl enable *websitename*
  You can check the service is running in systemd by looking at it's status:
  - systemctl status *websitename*
  Confirm it's working checking on your website in the browser
