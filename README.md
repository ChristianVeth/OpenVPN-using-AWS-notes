This guide will teach you how manually setup your own VPN using OpenVPN on an AWS server, as well my struggles along the way. Also, if you're interested in using your OpenVPN with online gaming, there are intructions on how to configure your security groups to allow for connection to the Apex Legends online gaming servers (these instructions should be able to be subsituted for any online games relatively easily).

1. First things first, you need to create your own Elastic IP address. This will make it so our IP Address stays the same at all times, even when shut down and restarted. Now, navigate to your AWS Dashboard and select Elastic IPs. Then click on allocate new address, select form your options of either an Amazon's pool of IPv4 addresses, or you can use one of the Global static IP addresses by using AWS's Global Accelerator. 

2. Now that you have your own Elastic IP, it's time to create a security group. Your VPN will need at least 3 open inbound ports. Navigate to Security Groups from your AWS Dashboard, then create a new security group. First we will need an SSH open to port 22 so that we can log into it and configure our settings. We will also need Custom TCP port 943, and lastly Custom UDP 1194. All of these should be configured with access from your public facing IP. Next we need to add 2 outbound rules. First add HTTP to port 80, with access to any IPv4, then add TCP to port 443 with access to any IPv4. Give your security group a name, and a description if you'd like, then press create security group to save!

3. Next it's time to create your server, navigate back to the AWS Dashboard and press on Instances, then Launch Instance. Give your server a name, such as My OpenVPN. Then type       OpenVPN and hit enter in the "Search our full catalog" field under Application and OS Images. Move over to the AWS Marketplace AMIs. Now, we're going to select Ubuntu 22.04 (the free version). Move down to Instance Type, and select t2.mico (you can upgrade later based on your needs, but for now we'll do a free version). You can either create a new key pair or select an already existing one you've created. Slide down to the Network Settings and choose Select existing security group, then choose the one we've just created. For configure storage, we will select the default free amount. Go ahead and hit Launch Instance!

4. Navigate to your ElasticIPs page, make sure you have the Elastic IP you've just created selected, then press on Actions > Associate Elastic IP address. Under Instance, select your newly created VPN server. Leave Private IP address blank, and then press Associate. Copy down your Elastic IP address so that you have it easily accessible.

5. Now it's time to SSH into your server! Open up the terminal and type in;                 

        | ssh -i /pathtokey/nameofkey.pem ubuntu@yourelasticip                                       

    Type yes in response to your request, and you're in!

6. As always, it's best practice to update and upgrade your machine. Type in;                          

        | sudo apt update                                                                                                       
        | sudo apt upgrade -y

7. Now, head over to https://as-portal.openvpn.com/get-access-server/ubuntu?source=default-signup and sign up for a free account (if you don't already have one). Then follow the instructions by adding the following commands in your terminal, but first we will need to use sudo  along with the commands we were given from openvpn's official 
instructions;                                                      

        | sudo apt update && apt -y install ca-certificates wget net-                                                                   

    After inputting this command, I received an error that access has been denied to port 80! Since you've follow my instructions so far, unfortunately so will you. But that's okay, because it's all a learning experience, and it's an easy fix! I forgot to have us add access to HTTP port 80 as an outbound rule (since this is our server reaching out to openvpn, as opposed to ssh into the server, which would be an inbound rule). We will need to add two outbound rules, HTTP to port 80, and Custom TCP to port 443 as well. and make sure to allow access to anywhere. These will both be need to allow accedss to your OpenVPN. Let's try that command again;                                                 

        | sudo apt update && apt -y install ca-certificates wget net-                                    

    No more access to port 80 denied this time, but it's still not working! Now I've got another error, this time, it's 13: permissions denied... so let's do some digging... apparently specifying sudo while downloading OpenVPN it's wanting us to become root. So, let's try again, but this time we're going to have to add a different first step to the command line;                                                                     

        | sudo -i                                                                                                                                                                                                                   
        | sudo apt update && apt -y install ca-certificates wget net-tools gnupg                                                               

    then select the service option, not user, and select OK. Hit enter.                                                                 

        | sudo wget https://as-repository.openvpn.net/as-repo-public.asc -qO /etc/apt/trusted.gpg.d/as-repository.asc                                                                    | 

    sudo echo "deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/as-repository.asc] http://as-repository.openvpn.net/as/debian jammy main">/etc/apt/sources.list.d/openvpn-as-repo.list                                                                   

        | sudo apt update && apt -y install openvpn-as                                                         

    When asked about Newer kernel available, press Ok. Which services should be restarted? systemd.logind.service.                                                                            
        
        Access Server Web UIs are available here;                                                                 
        Admin UI: https://x.x.x.x:943/admin                                                                             
        Client UI: https://x.x.x.x:943/                                                                                      
    
    To login please use the "openvpn" account with "xxxxxxxxx" password.                    
    Notate your Access Server Web UIs, but remember to change the private ip that openvpn has provide to reflect the public IP address of your Elastic IP.             Copy down, or change your password using;                                                             

        | sudo passwd openvpn                                                                                               

    then select your new password!

8. Head to your web browser and enter your Admin UI url! Use "openvpn" as the account name, and enter your password. 

9. Once your logged in, head over to Network settings, and change the IP address from your private AWS IP address to the Elastic IP address. Press Save Settings at the bottom of the page, then Update Running Server once your webpage shows Settings Changed. 

10. Now logout of the admin console, and go to the Client side url! Type in your username and password, then press on Yourself (user-locked profile) under Available Connection Profiles. This will download the certificate, which will authenticate your computer for connection to the VPN. Once the certificate has downloaded, you can logout of the Client UI. Then, download the recommended OpenVPN Connect software for your platform. Lastly, select to add vpn profile from your downloaded cert, enter the password, and BOOM your connected to your new VPN! If you're only interested in the VPN, and browsing the web, give it a try and see if it works. However, if you'd like to continue on to see if we can get our VPN's working with Apex Legends (or other online games), then go to the next step!

11. Time to try to connect to the Apex Legends servers! I'm playing on PC, and through steam. First step, open up steam, and make sure we can connect to their app. Steam connects just fine for me, I'm online  and ready to start up Apex. Moment of truth!

12. As expected with this sort of first time project, we run into a hurdle!           

![Alt text] (D:\Obsidian Vault\Image Links\apexports.png)

Now for the fun part, the troubleshooting! As I see it, there's one of 3 possibilities for the origin of our error; AWS server side, Apex Legends server side, or OpenVPN settings in the Admin UI. It's likely not OpenVPN settings, since I'd assume they've configured their default settings without any sort of firewalls or blacklists. That leaves just two more options... one of which would be virtually out of my control, so let's assume it's our AWS server and start with that!

13. The culprit for an issue with the AWS server settings would almost certainly stem from the security rules I've set for the AWS server. Googling your issue is always a good first    step, hopefully someone else has already into the same problem, and you can find a step by step on how to fix it. Unfortunately for me, google has turned up dry for fixes on this particular issue, so it's time for some old fashioned trial and error. Now given the issue I ran into earlier with having outbound access port 80 through HTTP denied, I've got a hunch of what may need to be done... the ports which the Apex servers use to communicate back and forth with your network will have to opened, otherwise the access will be denied, just like when trying to install openvpn. If you're like me (you probably are if you're reading this guide) and you've tinkered with your router settings and to improve your connection for online gaming, you'll likely be very familiar with this next step, because you'll be doing virtually the same things as when port forwarding to a games servers to open your NAT! Time to google what ports are needed for port forwarding with Apex Legends. If you're also playing from your PC, and on steam, you can use the info I'll be providing verbatim, if not you'll just need to google what ports need to open for your particular platform that you're using to play Apex. 

14. Head to either your security groups, or to your Instance and select the security tab to edit the rules. Googling some information about port forwarding to Apex shows us that Apex uses the same ports for inbound and outbound traffic. For PC on Steam's platform, there are quite a few ports that will need to be added in our rules. For both TCP and UDP, you'll be selecting Custom TCP/UDP, and allowing access to anywhere for both. If we knew the public facing IP's for Apex's servers, we could input those for the rules to tighten up our security, but unfortunately we do not. 
***Update*** Good news! I've found the public facing IP for the Apex servers! We can leave the Outbound rules as they are, however we should change the Inbound rules as it's not safe to have ports open to any IP address. In your inbound rules, add "127.0.0.53/32" to your source for each of the ports that have been opened for Apex Legends. This will now result in only the Apex Servers being able to communicate with these ports instead of any IPv4. Below are the ports need for PC on steam!                                                                                       
![[apexports.png]] 

15. You should now be able to connect to the Apex Legends online gaming servers! Keep in mind this should theoretically work with any connecting to any game, you will just need to substitute the new games ports respectively, and find out if they have different rules for inbound/outbound. 