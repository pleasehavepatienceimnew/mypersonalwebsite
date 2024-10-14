---
layout: default
---
09/09/2024

# Wazuh SOC Automation Lab: Building an Automated SOC Environment

The objective of this lab is to configure a secure SOC environment behind a firewall and automate alert and ticket generation using open-source security tools.

## Lab Architecture
Below is the logical diagram of the SOC lab setup:
![Labdiagram](Diagrams/wazuhlabdiagram.jpg)

To achieve this, I used the following tools:
- **OPNsense Firewall**
- **Wazuh SIEM and XDR**
- **TheHive** (ticketing system)
- **Shuffle** (SOAR platform)

I hosted all my virtual machines (VMs) using Oracle VirtualBox on my computer, allowing me to learn and configure each component step-by-step.

## Firewall Configuration: OPNsense VM

I configured the OPNsense firewall to place all my VMs in a secure LAN network while connecting them to the internet through a WAN interface. All traffic from the VMs is routed through the firewall, ensuring control and visibility over the entire lab environment.

## SIEM & XDR: Wazuh Manager VM

The Wazuh Manager is responsible for collecting logs from the Windows 10 machine, generating alerts, and sending them to Shuffle for automated ticket creation. It serves as the primary SIEM tool in the lab, helping monitor and analyze activity.

## Log Collection: Windows 10 Host

On the Windows 10 machine, I installed Sysmon and the Wazuh Agent to capture logs and send them back to the Wazuh Manager. This host is also the target of simulated attacks to generate alerts for testing the automated workflows.

## Ticketing System: TheHive VM

TheHive is used to manage incident tickets. Once an alert is triggered by Wazuh, TheHive automatically creates a ticket for the analyst to investigate.

## SOAR Integration: Shuffle VM

Shuffle, installed via Docker and accessed through a web GUI, automates workflows. It enriches indicators of compromise (IOCs) using the VirusTotal API and facilitates seamless communication between Wazuh and TheHive.

## How the Lab Works

As demonstrated in the diagram, the purpose of this lab is to automate SOC workflows and enhance security incident response efficiency.

1. A simulated attack is performed on the Windows 10 host.
2. Wazuh Agent detects the malicious activity and forwards the logs to Wazuh Manager.
3. The Wazuh Manager generates a custom alert and sends it to Shuffle.
4. Shuffle enriches the IOCs using the VirusTotal API.
5. Shuffle then creates a ticket in TheHive and sends an email to the analyst with all relevant information and enriched IOCs.
6. The analyst receives the email, logs into TheHive and Wazuh, and begins investigating the incident further.

This lab helped me gain hands-on experience with configuring and managing a SOC environment, automating key processes, and integrating various open-source tools to streamline incident response.
