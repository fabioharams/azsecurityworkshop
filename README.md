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

8. Create a Windows Server 2016/2019 VM using the following parameters:

- Computer name: WS01
- Vnet: VNETCORP
- Subnet: Default (10.0.0.0/24)
- Public IP Address: None
- Configure Network Security Group (NSG): LABSEC
- Public Inbound Ports: None
- OS Disk Type: Standard SSD
- Username: Azuser1
- Password: Azsecworkshop!

> Note: The NSG LABSEC and Vnet/Subnet already exists and must be used to accomplish other labs.

9. Check DVWA

- Open Azure Portal and then select the WS01 VM. Click on **Connect** button, input the credentials used on Step 8 and click **Connect**. The Server Manager will appear. 

![img19](/img/img19.png)

- On the left side of **Server Manager** click on **Local Server**. Click on **IE Enhanced Security Configuration**. Change to **Off* for both Administrators and Users.

![img20](/img/img20.png)

![img21](/img/img21.png)

- Check the Private IP Address of DVWA VM

Open Azure Portal, click on DVWA virtual machine  and take note of Private IP Address. Probably the IP address will be **10.0.0.4** .

![img22](/img/img22.png)

- Access DVWA through VM01

Use VM01 to check if DVWA is up and running. Connect to VM01 using Azure Bastion, open **Internet Explorer** and then type **10.0.0.4** on URL. This will open the DVWA login screen.

> Note: Do not click on **Create / Reset Database** yet because you first need to setup permissions

![img23](/img/img23.png)

- Access DVWA using Azure Bastion

Connect on DVWA VM using Azure Bastion and type the following commands

> <code>cd /var/www/html</code>

> <code>sudo chmod 757 /var/www/html/hackable/uploads/</code>

> <code>sudo chmod 646 /var/www/html/external/phpids/0.6/lib/IDS/tmp/phpids_log.txt</code>

> <code>sudo chmod 757 /var/www/html/config</code>

Do not disconnect. You will continue on next step

- Adjust PHP

Open **vi** with **sudo** and edit the settings for pho file

> <code>sudo vi /etc/php/7.2/apache2/php.ini</code>

Find line 837 and change the parameter **allow_url_include = Off** to **allow_url_include = On**

Exit **vi** by pressing **ESC** button and type **:wq**

- Setup permission (MySQL)

Now you can access again the **DVWA** VM through **Azure Bastion**. Type the following commands to setup the required permission:

> <code>sudo mysql -uroot</code>

> <code>DROP USER 'root'@'localhost';</code>

> <code>CREATE USER 'root'@'localhost' IDENTIFIED BY 'p@ssw0rd';</code>

> <code>GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION;</code>

> <code>FLUSH PRIVILEGES;</code>

> <code>exit</code>

Now you are ready to return to **VM01** and create the database

- Create database

Open **Azure Portal** , select **VM01** and connect using **Azure Bastion**. Once you are connected then open **Internet Explorer** and access the URL **http://10.0.0.4**.

![img24](/img/img24.png)

Click **Create / Reset Database**. You will see that the database is created and will be redirected to login screen again. Logon again and the DVWA page will appear.

![img25](/img/img25.png)

At this moment we have our DVWA VM ready. Follow the next steps to prepare the monitoring. 


### Step 2 | Create Log Analytics workspace ###

All logs will be forwarded to Log Analytics and it's a requirement for Azure Sentinel, Network Watcher, etc. Follow the steps bellow to create your Log Analytics Workspace.

> 1. Create Workspace

Open Azure Portal, click New and type **Log Analytics Workspace** . Click **Create** and use these parameters:

> Note: Make sure to use the same **Resource Group** and **Region**

![img16](/img/img16.png)

![img17](/img/img17.png)

![img18](/img/img18.png)


### Step 3 | Deploy Application Gateway w/ Web Application Firewall(WAF) ###

Azure Application Gateway is a web traffic load balancer that enables you to manage traffic to your web applications.Also includes Web Application Firewall (WAF), a service that provides centralized protection of your web applications from common exploits and vulnerabilities.

> 1. Deploy Application Gateway w/ WAF

For this workshop you will deploy **Application Gateway w/ WAF V1** to detect attacks to DVWA VM. The reason to use Application Gateway V1 instead of V2 is about the possibility to restrict access to specific public IP address. Application Gateway will publish a Public IP Address but it's not so simple to restrict wich IP Address can access this environment. It's very useful if you want to test for a long time but don't want anyone from internet to access the DVWA (the credentials to access DVWA are simple). Using Application Gateway V1 it's possible to restrict this traffic using **Network Security Group (NSG)**. Of course it means that you need to change your NSG Rule every time your Public IP Address (from your ISP connection) change. 
If you don't need this control then you can create your Application Gateway w/ WAF V2. 

- Open **Azure Portal**, click **Create a resource** and type **Application Gateway** . Click **Create**

![img26](/img/img26.png)

- Use the following parameters for **Basic**
- - Resource Group: LABSECURITY
- - Application gateway name: APPGW
- - Region: East US
- - Tier: WAF
- - Instance count: 1
- - SKU size: Medium
- - Firewall status: Enabled
- - Firewall mode: Prevention
- - HTTP2: Disabled
- - Virtual network: VNETCORP
- - Subnet: AppGw (10.0.1.0/24)

![img27](/img/img27.png)

> Click **Next: Frontends**

- Use the following parameters for **Frontends**
- - Frontend IP address: Public
- - Public IP Address: click **Add new**
- - - Name: PUBIPDVWA
- - - Click **OK**

![img28](/img/img28.png)

> Click **Next: Backends**

- Use the following parameters for **Backends**

- - Click **Add a backend pool**
- - - Name: BACKENDDVWA
- - - Add backend pool without targets: No
- - - BackEnd Targets
- - - - Target type: Virtual Machine
- - - - Target: dvwa*** (10.0.0.4)
- - - - Click **Add**

![img29](/img/img29.png)

> Click **Next: Configuration**

> Click **Add a routing rule**

![img30](/img/img30.png)

- Use the following parameters for **Add a routing rule**
- - Rule name: RULEDVWA
- - Listener name: LISTENERDVWA
- - Frontend IP: select **Public**
- - Protocol: HTTP
- - Port: 80
- - Listener type: Basic
- - Error page url: No
- - Click **Backend targets**
- - - Target type: Backend pool
- - - Backend target: select **BACKENDDVWA**
- - - HTTP settings: click **Add new**
- - - - HTTP settings name: HTTPSETTINGSDVWA
- - - - Backend protocol: HTTP
- - - - Backend port: 80
- - - - Cookie-based affinity: Disable
- - - - Connection draining: Disable
- - - - Request time-out (seconds): 20
- - - - Override backend path: blank
- - - - Override with new host name: No
- - - - Click **Add**
- - - Click **Add**

![img31](/img/img31.png)

> Click **Next: Tags**

> Click **Next: Review + create**

> Click **Create**

Wait few minutes to finish the deployment (˜10min) and then click on **APPGW** (located on your Resource Group). You can see the public IP address assigned to App Gateway. Take note of this IP address and then access using **Internet Explorer** on **VM01**. This is just a test to make sure that the traffic to DVWA is handled by Application Gateway w/ WAF.

![img32](/img/img32.png)

> Note: This **Frontend public IP address** is fake. 

> 2. Restrict access to Application Gateway (optional)

As explained before if you want to restrict wich IP address from internet can access the DVWA then you need to configure the **Network Security Group**. If not just ignore this step.

- Open **Azure Portal** and then click **Create a resource**. Type **Network security group** and then click **Create**

![img33](/img/img33.png)

- Use the following parameters for **Create network security group**
- - Resource Group: LABSECURITY
- - Name: APPGWLABSECURITY
- - Region: East US
- - Click **Next: Tags**
- - Click **Next: Review + create**
- - Click **Create**

![img34](/img/img34.png)

Now you can open again the Resource Group **LABSECURITY** and click on **NSG** **APPLABSECURITY**

- Open NSG **APPGLABSECURITY** and use the following rules (only **Inbound Security rules**)

![img35](/img/img35.png)

- - Rule **AppGwProbe**

![img36](/img/img36.png)

- - Rule **AccessFromHome**

![img37](/img/img37.png)

- - Rule **AzureLoadBalancerProbe**

![img38](/img/img38.png)

- - Rule **YouShallNotPass**

![img39](/img/img39.png)


> Note: You must change your rule **AccessFromHome** (field **Source IP address**) and use your Public IP address that you are using. You can easily find this just openning **Google** and typing **what is my ip**. This is the IP Address that you will need to insert on **Source IP address** field.

![img40](/img/img40.png)

> Note: Now you have Application Gateway forwarding to DVWA VM and only allowing access from your Public IP. Next step you will forward logs from NSG and Application Gateway to Log Analytics.


### Step 4 | Configure Network Watcher ###



[enable region]
[configure NSG FLow logs - AppGw, NSG]
[diagnostic logs]
[Traffic analytics]

### Step 5 | Configure Security Center ###

[change converage]
[add security solution]


### Step 6 | Configure Azure Sentinel ###

[add workspace]
[enable data connector]
[save workbook]

### step 7 | Appgw ###