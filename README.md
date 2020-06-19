# Security Workshop | Azure (under development)
This workshop contains instructions to test resources on Azure like:
- Application Gateway
- Web Application Firewall (WAF)
- Log Analytics
- Azure Security Center
- Azure Sentinel
- Network Watcher

To test this scenario a virtual machine running UBUNTU with DVWA (Damn Vulnerable Web Application) will be used to test vulnerabilities.

## Overview of the environment ##

[include diagram]

## Prepare the environment ##

> 1. Create a Resource Group

e.g.: LABSECURITY

You can use any public region because the features on this lab doesn't require an specific region.  

![img1](/img/img1.png)

> 2. Create VNET and Subnets

Create a VNET in the same region of Resource Group with the following settings bellow:

- Name: VNETCORP
- Region: e.g. EAST US
- IPv4 Address Space: 10.0.0.0/16
- Subnets:
- - default: 10.0.0.0/24
- - AppGw: 10.0.1.0/24
- - AzureBastionSubnet: 10.0.2.0/24
- DDoS Protection: Basic
- Firewall: Disabled
- Tags: None

Note: you can create Bastion Host (and the Subnet) during the creation of VNET. I recommend you to do this later because you can use the same steps to do in other VNETs. Feel free to do if you have more experience on Azure VNET

![img2](/img/img2.png)

> 3. Create Linux VM for DVWA

- Create a Ubuntu Server 18.04 LTS from Azure Portal

![img3](/img/img3.png)

![img4](/img/img4.png)

- user: Azuser1
- password: Azsecworkshop!

![img5](/img/img5.png)

![img6](/img/img6.png)

![img7](/img/img7.png)

![img8](/img/img8.png)

![img9](/img/img9.png)

![img10](/img/img10.png)

> 4. Enable Azure Bastion

Follow these steps to use Azure Bastion. This is importante because the VM was created without Public IP address.

- On Azure Portal click on **"Create a resource"** and then type **BASTION** . Click **"Create"**

![img11](/img/img11.png)

![img12](/img/img12.png)

![img13](/img/img13.png)

> Above you can find more information about how to deploy Azure Bastion. Just remember to use Microsoft Edge/Chrome and disable Pop-ups

Link: https://docs.microsoft.com/en-us/azure/bastion/bastion-create-host-portal

> If you want to automate all the steps to create this environment then you can use the **template** folder located [here](https://github.com/fabioharams/azsecurityworkshop/tree/master/template)

## Start the lab ##

> ### Step 1 | Install DVWA on UBUNTU ###

DVWA (Damn Vulnerable Web Application) is a PHP/MySql web application very popular to train security specialists against vulnerabilities. For more information about DVWA please click **[here](https://github.com/ethicalhack3r/DVWA)**.

1. Connect to Ubuntu VM using Azure Bastion

Open **Azure Portal**, select the Ubuntu Virtual Machine created previously (**DVWA**), click **Connect** and select **Bastion**. Insert the following credentials bellow and then click **Connect**

- username: Azuser1
- password: Azsecworkshop!

> Note1: if the new tab doesn't open just check if your browser is not blocking **Pop-Ups**
> 
> Note2: Attention - Linux is case sensitive for username

![img14](/img/img14.png)

![img15](/img/img15.png)

2. Update Ubuntu

It's recommended to update Ubuntu (or any Virtual Machine) after installation. Execute the following command to update

> <code> sudo apt update && sudo apt upgrade -y </code> 

3. Download MySQL, PHP and Apache

These packages are required to install DVWA. Just execute the follwing command. Press **Y** to confirm:

> <code>sudo apt install apache2 mysql-server php php-mysqli php-gd libapache2-mod-php git</code>

Return to **home** folder

> <code>cd ~</code>

4. Clone the DVWA repository:

> <code> git clone --recurse-submodules https://github.com/ethicalhack3r/DVWA.git</code>

5. Remove the default web page created by Apache

> <code>sudo rm /var/www/html/index.html</code>

6. Copy the downloaded files to a new folder and after that change to folder

> <code>sudo cp -r ~/DVWA/* /var/www/html/</code>

> <code>cd /var/www/html</code>

7. Copy the config file for DVWA

> <code>sudo cp config/config.inc.php.dist config/config.inc.php</code>

Done! now you can connect from other Virtual Machine on Azure (using Azure Bastion) and test if DVWA is up and running (the setup for DVWA require a browser). The DVWA virtual machine doesn't have a Public IP Address so you will need a VM with browser to access and finish the configuration (or adjust anything else you want on DVWA)

- Create a Windows Server 2016/2019 VM using the following parameters:
- - Computer name: WS01
- - Vnet: VNETCORP
- - Subnet: Default (10.0.0.0/24)
- - Public IP Address: None
- - Configure Network Security Group (NSG): LABSEC
- - Public Inbound Ports: None
- - OS Disk Type: Standard SSD
- - Username: Azuser1
- - Password: Azsecworkshop!

> Note: The NSG LABSEC and Vnet/Subnet already exists and must be used to accomplish other labs.


### Step 2 | Create Log Analytics workspace ###

All logs will be forwarded to Log Analytics and it's a requirement for Azure Sentinel, Network Watcher, etc. Follow he steps bellow to create your Log Analytics Workspace.

> 1. Create Workspace

Open Azure Portal, click New and type **Log Analytics Workspace** . Click **Create** and use these parameters:

![img16](/img/img16.png)

![img17](/img/img17.png)

![img18](/img/img18.png)


### Step 3 | create App Gateway ###

[wafv1]
[create NSG for AppGw Subnet]
[Prevention mode]
[test connectivity]

### Step 4 | Configure Security Center ###

[change converage]
[add security solution]

### Step 5 | Configure Network Watcher ###

[enable region]
[configure NSG FLow logs - AppGw, NSG]
[diagnostic logs]
[Traffic analytics]

### Step 6 | Configure Azure Sentinel ###

[add workspace]
[enable data connector]
[save workbook]
