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

DVWA (Damn Vulnerable Web Application) is PHP/MySql web application to train security specialists to test vulnerabilities. For more information about DVWA please click **[here](http://www.dvwa.co.uk/)**.

1. Connect to Ubuntu VM using Azure Bastion

Open **Azure Portal**, select the Ubuntu Virtual Machine created previously (**DVWA**), click **Connect** and select **Bastion**. Insert the following credentials bellow and then click **Connect**

- username: Azuser1
- password: Azsecworkshop!

> Note: if the new tab doesn't open just check if your browser is not blocking **Pop-Ups**

![img14](/img/img14.png)

![img15](/img/img15.png)





### Step 2 | create Log Analytics workspace ###

[]

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
