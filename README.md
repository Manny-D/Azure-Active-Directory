# Creating an Azure Active Directory 

<p align="center">
<img src="https://github.com/Manny-D/Azure-Active-Directory/assets/99146530/02ccd842-ae28-4192-88f6-fc81884550a9"/>
</p>

<br>

## Description 
In this project, I will setup a simulated Active Directory environment where I will create the following:

- A Domain Controller (DC) VM with a static IP to manage user credentials (Active Directory).
- A Client VM that joins the domain and uses the DC for DNS.

The DC will act as a central point for monitoring network traffic. Clients will route their internet traffic through the DC, allowing administrators to identify suspicious activity in logs.

Additionally, a PowerShell script will generate 1,000 user accounts, showcasing efficient credential management within the Active Directory domain.

<br>

## Environments and Technologies

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop (RDP)
- Active Directory Domain Services
- PowerShell

<br>

## Operating Systems

- Windows Server 2022
- Windows 10 (21H2)

<br>

## Project Diagram

<p align="center">
<img src="https://i.imgur.com/BcRNcBi.png" height="50%" width="50%" alt="9"/><br />
</p>

<br>

## Step 1: Setup

When in Azure, create a <b>Resource Group</b>, then create 2 Virtual Machines (VMs). <br>
One will be the DC while the other will be the Client.

### Create the Domain Controller VM
Fill in the <b>Virtual machine name</b> field, then assign it to the <b>Resource group</b> created above. 

<img src="https://i.imgur.com/uYfHMQG.png" height="70%" width="70%" alt="9"/><br />

For the image, select <b>Windows Server 2022 Datacenter: Azure Edition - x64 Gen2</b>. <br>
<b>Note</b>: The recommended vcpu size is 2.

<img src="https://i.imgur.com/FNoA7m0.png" height="70%" width="70%" alt="9"/><br />

Create the Administrator Login credentials for the DC and be sure to take note of them. 

Click Next until you get to the <b>Networking</b> tab and create / name your <b>Virtual Network</b>. This will be important when creating the Client VM. <br>

Check the box under Licensing then click the <b>Review and create</b> button to create the DC VM.

<img src="https://i.imgur.com/NxXFK16.png" height="70%" width="70%" alt="9"/><br />

<br>

### Create the Client VM <br>
The steps are the same as the DC creation above, however, for the Client VM Image, choose a <b>Windows 10</b> one.

<img src="https://i.imgur.com/2PvUCJN.png" height="70%" width="70%" alt="9"/><br />

Create login credentials for this VM and be sure to take note of them.

Click Next until you get to the <b>Networking</b> tab and select the same <b>Virtual network</b> as the Domain Controller. 

Click the <b>Review and create</b> button to create the Client VM.

<br>

### Setting the Domain Controller's network interface card (NIC) IP address to static
Domain controllers require static IPs, unlike dynamic ones, to ensure a consistent location (IP address) for client machines to find them for authentication and resource discovery.

Within the DC, click <b>Networking</b> -> Next -> click on <b>Network Interface</b>

<img src="https://i.imgur.com/6W2WZTA.png" height="60%" width="60%" alt="9"/><br />

Next click <b>IP configurations</b> -> click <b>IP configuration</b> 

<img src="https://i.imgur.com/0R53K7r.png" height="60%" width="60%" alt="9"/><br />

Scroll to <b>Allocation</b> and change the radio button from <b>Dynamic</b> to <b>Static</b> -> click Save

<img src="https://i.imgur.com/JAGBZtk.png" height="60%" width="60%" alt="9"/><br />

<br>

### Enabling ICMP on the Domain Controller (DC)

ICMP allows us to see if the DC is online and reachable on the network and can also be used for Active Directory replication health checks in some configurations.


With the Client VM credentials created before, login to the Client VM using its IP address via Remote Desktop Connection. 

<img src="https://i.imgur.com/28TrmKg.png" height="55%" width="55%" alt="9"/><br />

Next, open a Command Prompt and ping the Domain Controller with the static IP address. <br>
ex. <b>ping (DC IP) -t</b> to continuously ping it. 
<b>Note</b>: It will time out till we update settings in the next steps. 

```commandline
ping 10.0.0.4 -t
```

<img src="https://i.imgur.com/1zyrIUN.png" height="80%" width="80%" alt="9"/><br />

<br>

To enable ICMPv4, login to the Domain Controller VM -> <b>Windows Defender Firewall with Advanced Security</b> 

<img src="https://i.imgur.com/bYkAEwk.png" height="85%" width="85%" alt="9"/><br />

Click on <b>Inbound Rules</b> -> sort by <b>Protocol</b> -> look for the 2 listed as: <b>Core Networking Diagnostics - ICMP Echo Request(ICMPv4-In)</b>

<img src="https://i.imgur.com/EydkpVV.png" height="80%" width="80%" alt="9"/><br />

Then right-click on each one -> tick Enable -> Save

<img src="https://i.imgur.com/ENb2KyF.png" height="55%" width="55%" alt="9"/><br />

In the Client VM, check on the pings in the command prompt. The Domain Controller should now be responding. 

<img src="https://i.imgur.com/2YNRrzi.png" height="80%" width="80%" alt="9"/><br />

<br>

## Step 2: Installing Active Directory

In the Domain Controller, open <b>Server Manager</b> -> click on <b>Add roles and features</b>

<img src="https://i.imgur.com/0BcdJpW.png" height="80%" width="80%" alt="9"/><br />

Click Next -> till you reach <b>Server Roles</b> -> tick the <b>Active Directory Domain Services</b> box -> <b>Add Features</b>

<img src="https://i.imgur.com/K5oTmkD.png" height="80%" width="80%" alt="9"/><br />

Click Next -> till you reach <b>Confirmation</b> -> click Install <br>
<b>Note</b>: Installation times will vary. <br>

Once <b>Configuration required. Installation succeeded on (Your DC name here)</b> appears, you can click Close.

At the top-right corner of the <b>Server Manager</b> window, click on the flag / yellow triangle with a <b>!</b> -> click <b>Promote the server to a domain controller</b>

<img src="https://i.imgur.com/D8p1wU9.png" height="40%" width="40%" alt="9"/><br />

In the <b>Configuration Wizard</b> pop-up, tick the <b>Add a new forest</b> radio button -> enter a <b>Root domain name</b> -> Next

<img src="https://i.imgur.com/BefHqfW.png" height="80%" width="80%" alt="9"/><br />

Create a <b>Directory Services Restory Mode (DSRM) password</b> (required but not relevant in this lab) -> Next

<img src="https://i.imgur.com/TYXfTrJ.png" height="80%" width="80%" alt="9"/><br />

<br>
The NETBIOS domain will be created (will take some time) -> Next -> till you reach <b>Prerequisites Check</b> (will take some time) -> <b>Install</b> <br>
<br>
Once the installion is completd, the VM will reboot -> once rebooted, login to the Domain Controller using the domain name and the username. 
<b>Note</b> See example below. <br>

<img src="https://i.imgur.com/nT5uFiT.png" height="55%" width="55%" alt="9"/><br />

<br>

