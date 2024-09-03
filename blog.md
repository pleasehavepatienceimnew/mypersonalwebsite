---
layout: default
---


### Wazuh SOC Lab

In this blog, I will walk you through on how I created a Wazuh SOC lab on a VM with OPNsense firewall and how you can do so too.

I used Oracle VirtualBox to host my VMs. The idea is that I wanted to host everything on my computer to learn how all the configurations are done step by step all by myself.

I then went on and got an image for OPNsense Firewall and installed it on a Virtual Machine. I added a NAT and internal network network adapters. The firewalll will use the internal network as the LAN and the NAT as WAN. This way the other VMs can have their traffic routed through the firewall and be able to connect to the internet.

After configuring the firewall to upstream correctly and that a seperate Kali VM can connect to the internet through the firewall as gateway. We now go on to setup our other VMs.

For this lab, we are setting up Wazuh SIEM and XDR and using to ingest logs from a Windows 10 VM that's acting as the victim host. From there, Wazuh will create an alert and send it to shuffle which will then create a ticket in TheHive that will be available for the analyst to pick up. An email will also be sent to the analyst to notify him and giving him initial details regarding the incident.

Unfortunately, I have not been able to include Active response from Wazuh.


## Firewall

I setup it up to have an IP address of 10.200.200.254 with a 24 subnet range. I then accessed the firewall GUI using the IP address in a browser. 

The firewall will have all traffic routed from the other VMs through it to to the internet.

## Wazuh

Using a new Ubuntu VM, I set the IP to be 10.200.200.26 and a 10.200.200.254 gateway with Google DNS. Afterwards, I install Wazuh Manager on the machine, and accessed the web gui using the IP 10.200.200.26.

Wazuh manager will collect the logs from the Windows 10 machine to send it to Shuffle.

## Windows 10 Host

I installed a Windows 10 VM machine to isntall the Wazuh agent that will collect the logs as I will simulate an attack on this machine. I set the IP address as 10.200.200.21.

This machine will be the target of a simulated attack so that we can create a simulated alert and ticket using the Shuffle workflow.

## TheHive

In another new Ubuntu VM, we set up the IP to be 10.200.200.27 and 10.200.200.254 gateway and Google DNS. Afterwards, I installed multiple dependencies that are needed for TheHive to work. I then  installed TheHive.

This part here took me a while to get it right and I had to create multiple VMs to get it work. Once it worked, I was able use the web gui using the IP 10.200.200.27:9000.

TheHive will simulate a ticket being created once an alert is triggered from Wazuh.

## Shuffle

This is the trickiest one. Originally I used the cloud Shuffle to create the workflow. This didn't work as I wasn't able to connect it to my VMs. So I had to install another Ubuntu VM instance where I learned how to use Docker to install Shuffle that is only available in docker.

The VM had the IP 10.200.200.30. To test Shuffle, I accessed the web GUI using 10.200.200.30:3443.

Shuffle will be used to create a workflow that will connect all of these VMs together.

### Problems I faced

This lab was meant to be done mainly in the cloud but I decided to build everything myself on VMs because I wanted to learn how to route all my traffic through the firewall and have it all work together in sync.

My first problem was not being able to connect my VMs to the internet. I had no experience in setting up firewalls, and this was a problem I had for more than a week. I needed to put the firewall's NAT gateway as the upstream gateway which is 10.0.2.2 respectively.

The second problem that I faced was setting up TheHive. The amount of dependencies needed is crazy. If you make one wrong step, the whole setup is ruined which will require a new VM to start all over again. Java was one of the most painfully staking experiences ever. Cassandra does not work sometimes with Java 11. So you need to install Java 8. Making that work was not the most fun part but eventually I got it to work and was able to connect to TheHive through the web GUI.

The last main problem was for me to work with Shuffle. Originally, Shuffle can be used on the cloud by just making an account. I first tried that but then very quickly I was stuck because I can't connect to my VMs from the cloud. After researching, I found out that you can only host shuffle using docker. I then intstalled Docker Desktop on my host computer. I was able to successfully connect my VMs to my computer, but I wasn't able to make Shuffle work.

In hindsight, I could have made it work. But I didn't know what the problem was. So I made a new Ubuntu VM and learned how to work with Docker on Linux to host Shuffle using Docker-compose. After that, I faced the same problem of not being able to make the workflow work. Turns out, on-premise Shuffle does not allow for integration using https but only http. When I changed that it worked and everything after went smoothly.