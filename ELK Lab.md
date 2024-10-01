---
layout: default
---
01/10/2024

ELK SOC Lab

This SOC Lab will be follow a diagram that is attached below. I will be setting up an ELK stack SOC Lab on the cloud VM provider Vultr to simulate the implementation and configuration of SIEM in Virtual Private Cloud. I will also simulate a brute force attack using Mythic C2 and will create alerts and dashboards to detect successful brute force attacks and the C2 application.

This is the Lab logical diagram:
![Labdiagram](Diagrams/ELKlabdiagram.jpg)

## Elasticsearch Server Configuration



This SOC Lab will be follow a diagram that is attached below. I will be setting up an ELK stack SOC Lab on the cloud VM provider Vultr to simulate the implementation and configuration of SIEM in Virtual Private Cloud.

Diagram outlining how everything will be connected and how it will work.

I created a VPC 2.0 Network on Vultr. Region set as Sydney. Network address I used the IP 172.31.0.0 with a prefix of 24

Created a host amd selected Ubuntu 22.04 LTS image with 80GB option named ELK-SOCLab. 

Downloaded deb x86_64 version of elasticsearch

In my SSH session I used wget [copied link] to download the package and then installed the package.

Going to /etc/hosts and nano elasticsearch.yml and change the "local_host" line to the public IP of the server. Uncommented the http port line.

In Vultr to manage my Firewall settings. I made a new firewall group and allowed only my host IP.

I went back to my server settings and added the new firewall group and clicked update.
 
Installed the Kibana DEB x86_64.
  
Then I head to /etc/kibana/kibana.yml using nano to configure Kibana
 
Uncomment the server port and server host. And change server host to your VM public ip address
 
 Use the following commands to start Kibana
 
Started the elasticsearch-create-enrollment-token binary with --scope kibana command
 
I went to Vultr to setup another firewall to allow incoming TCP connections from my IP address.

After doing that, I allowed the port 5601 in the Ubuntu VM using ufw allow 5601 command

After connecting to my Kibana using http://[VM public IP]:5601 I copy and pasted my Enrollment Token then went to the directory /usr/share/kibana/bin and ran the kibana-verification-code.

I used the username elastic and the password given to us when setting up elasticsearch.

In the SSH session, in the same directory I run ./kibana-encryption-keys generate command

I copied the keys generated and ran ./kibana-keystore add xpack.encryptedSavedObjects.encryptionKey

It asked me what the value is and I paste the value I copied earlier. Repeated for the second value.

Then restarted Kibana.



## Windows Client Configuration



Now I will setup my Windows Server. 

I selected Shared CPU, Sydney Location, and Windows Standard Image. With the cheapest option for storage selected.

Disabled Auto-Backups and IPV6 and then my hostname. And press Deploy.

Now go to your windows and use the view console option to access the VM

Installed Sysmon on the Windows 10 host for more logging capabilities.

Added Custom Windows Event Logs integration on Elastic on my configured Agent Policy to gather Sysmon logs from the Windwows 10 server.

Added a firewall rule to allow TCP connections from port 9200 on the Vultr Firewal Group Policy



## Fleet Server


Created a new host called "ELKSOCLab-FleetServer" with Ubuntu 22.04 as OS image

Host ELKSOCLab-FleetServer was added as a Fleet Server in ELK under Fleet Server Policy

Installed Elastic Server Agent on the host for the server Policy

Created a Fleet Windows Policy called ELK-SOCLAB-omar

Installed the ELK agent on my windows client


## Elasticsearch Ingest Data


In Elasticsearch add Custom Windows Event Logs

Added integration policy "SOCLAB-WIN-Defender"

Added integration policy "SOCLAB-WIN-SYSMON"



## Ubuntu Server


Created a new host called "ELKSOCLAB-Linux-Omar" with Ubuntu 24.04 as OS image

Created a Fleet Windows Policy called ELK-SOCLAB-Linux-Omar

Installed the ELK agent on my Ubuntu Server



## Dashboards


Created two Dashboards on ELK

One containing the following:

-SSH Failed Authentication Map
-SSH Failed Activity Table
-SSH Successful Authentication Map
-SSH Successful Activity Table
-RDP Failed Authentication Map
-RDP Failed Activity Table
-RDP Successful Authentication Map
-RDP Successful Activity Table


Second Dashboard containing the following:

- Process created table (to show if one of the queried processes was created)
- Process Initiated Network Table(to show if a process initiated connection with external IPs)
- Microsoft Defender Disabled Table




## Mythic Server Configuration


In Vulter, I deployed  another host called "ELKSOCLAB-Mythic" to deploy Mythic C2 

Installed Mythic using the Ubuntu docker bash script



## Mythic Agent Setup


I created an diagram that's shown below

Created an Apollo Mythic agent with an http profile and configured it to enable all commands and to callback to the Mythic Server set up earlier.

Agent was name "svchost.exe"


## Kali Linux 


Downloaded Kali linux image for VirtualBox 

Once booted up, you can use it straight away

Updated and Upgraded repos

By using a weak and common password, we use crowbar from the kali Linux to brute force into the Windows Host and disabled Windows Defender.

By opening a python http server, we can Invoke a web request to download the C2 Mythic Agent.

I run the C2 Mythic and establish connection with the Mythic Server.

This is the Lab logical diagram:
![Labdiagram](Diagrams/Attack Diagram.jpg)

## osTicket Setup


Hosted a new server on Vultr in the VPC group with an IP 172.31.0.5 called "ELK-SOCLAB-osTicket"

Downloaded and installed XAMPP version 8.2.12 / PHP 8.2.12

Headed to C:\xampp and opened properties file using notepad. Changed apache_domainname to the server public IP 107.191.57.125

Created a firewall in Windows Defender to allow inbound port 80 and 443

After running the apache and mysql from xampp and pressing the admin button which opens the localhost website

I edited the user root@localhost to accept hosts from 107.191.57.125 and changed the password 

Headed to C:\xampp\myPHPadmin and edited config.inc.php. Changed localhost ipv4 address to 107.191.57.125

Repeated the above steps for the user pma@localhost

Download sefl-hosted osTicket from thier official website. Version v1.18.1

In C:\xampp\htdocs create a new folder called "osticket" 

After extracting osTicket, copied the two folder - Scripts and Uploads into the "osticket" directory

Created a new databse for our osTicket and gave our root@localhost permissions to it

Went in 107.191.57.125/osticket/upload to setup osTicket

Changed the file ost-sampleconfig in C:\xampp\htdocs\osticket\upload\include to ost-config

Used this command after finishing installation "icacls .\ost-config.php /reset" in the path C:\xampp\htdocs\osticket\upload\include using Admin powershell

Your osTicket URL:
http://107.191.57.125/osticket/upload/	

Your Staff Control Panel:
http://107.191.57.125/osticket/upload/scp



## osTicket + ELK Integration


Created API key with 172.31.0.3 VPC IP address

Added the API key as a connector as a webhook for using the 172.31.0.5 VPC IP Address for more security.

