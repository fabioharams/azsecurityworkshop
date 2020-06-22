# Security Workshop | Azure (under development)
This workshop contains instructions to test resources on Azure like:
- Application Gateway
  - Build secure, scalable, and highly available web front ends in Azure
- Web Application Firewall (WAF)
  - A cloud-native web application firewall (WAF) service that provides powerful protection for web apps
- Log Analytics
  - Full observatility into your applications, infrastructure, and network
- Azure Security Center
  - Unify security management and enable advanced threat protection across hybrid cloud workloads
- Azure Sentinel
  - Put cloud-native SIEM and intelligent security analytics to work to help rptect your enterprise
- Network Watcher
  - Network performance monitoring and diagnostics solution

To test this scenario a virtual machine running UBUNTU with DVWA (Damn Vulnerable Web Application) will be used to detect vulnerabilities.

## WARNING: The purpose of this lab is just to test detection and prevention on Application Gateway with WAF. Don't do this on any other resource instead of this lab ##

## Overview of the environment ##

[include diagram]

## Prepare the environment ##

> 1. Create a Resource Group

e.g.: LABSECURITY

You can use any public region because the features on this lab doesn't require an specific region.  

![img1](/img/img1.png)

___

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

___
   

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

___

> 4. Enable Azure Bastion

Follow these steps to use Azure Bastion. This is importante because the VM was created without Public IP address.

- On Azure Portal click on **"Create a resource"** and then type **BASTION** . Click **"Create"**

![img11](/img/img11.png)

![img12](/img/img12.png)

![img13](/img/img13.png)

> Above you can find more information about how to deploy Azure Bastion. Just remember to use Microsoft Edge/Chrome and disable Pop-ups

Link: https://docs.microsoft.com/en-us/azure/bastion/bastion-create-host-portal

> If you want to automate all the steps to create this environment then you can use the **template** folder located [here](https://github.com/fabioharams/azsecurityworkshop/tree/master/template)

___


## Start the lab ##

> ### Step 1 | Install DVWA on UBUNTU ###

DVWA (Damn Vulnerable Web Application) is a PHP/MySql web application very popular to train security specialists against vulnerabilities. For more information about DVWA please click **[here](https://github.com/ethicalhack3r/DVWA)**.

___

1. Connect to Ubuntu VM using Azure Bastion

Open **Azure Portal**, select the Ubuntu Virtual Machine created previously (**DVWA**), click **Connect** and select **Bastion**. Insert the following credentials bellow and then click **Connect**

- username: Azuser1
- password: Azsecworkshop!

> Note1: if the new tab doesn't open just check if your browser is not blocking **Pop-Ups**
> 
> Note2: Attention - Linux is case sensitive for username

![img14](/img/img14.png)

![img15](/img/img15.png)

___

2. Update Ubuntu

It's recommended to update Ubuntu (or any Virtual Machine) after installation. Execute the following command to update

> <code> sudo apt update && sudo apt upgrade -y </code> 

___

3. Download MySQL, PHP and Apache

These packages are required to install DVWA. Just execute the follwing command. Press **Y** to confirm:

> <code>sudo apt install apache2 mysql-server php php-mysqli php-gd libapache2-mod-php git</code>

Return to **home** folder

> <code>cd ~</code>

___

4. Clone the DVWA repository:

> <code> git clone --recurse-submodules https://github.com/ethicalhack3r/DVWA.git</code>

___

5. Remove the default web page created by Apache

> <code>sudo rm /var/www/html/index.html</code>

___

6. Copy the downloaded files to a new folder and after that change to folder

> <code>sudo cp -r ~/DVWA/* /var/www/html/</code>

> <code>cd /var/www/html</code>

___

7. Copy the config file for DVWA

> <code>sudo cp config/config.inc.php.dist config/config.inc.php</code>

Done! now you can connect from other Virtual Machine on Azure (using Azure Bastion) and test if DVWA is up and running (the setup for DVWA require a browser). The DVWA virtual machine doesn't have a Public IP Address so you will need a VM with browser to access and finish the configuration (or adjust anything else you want on DVWA)

___

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

___

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

___

### Step 2 | Create Log Analytics workspace ###

All logs will be forwarded to Log Analytics and it's a requirement for Azure Sentinel, Network Watcher, etc. Follow the steps bellow to create your Log Analytics Workspace.

___

> 1. Create Workspace

Open Azure Portal, click New and type **Log Analytics Workspace** . Click **Create** and use these parameters:

> Note: Make sure to use the same **Resource Group** and **Region**

![img16](/img/img16.png)

![img17](/img/img17.png)

![img18](/img/img18.png)

___

### Step 3 | Deploy Application Gateway w/ Web Application Firewall(WAF) ###

Azure Application Gateway is a web traffic load balancer that enables you to manage traffic to your web applications.Also includes Web Application Firewall (WAF), a service that provides centralized protection of your web applications from common exploits and vulnerabilities.

___

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

___

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

___

### Step 4 | Forward Logs ###

**Network Watcher** is a network performance monitoring and diagnostics solution on Azure. We will use this solution to forward NSG Logs and Diagnostic logs to Log Analytics workspace.

Firts create a storage account. This will be used to retain logs.

- On **Azure Portal** click **Create a resource** and type **Storage account**. Click **Create**

![img43](/img/img43.png)

- Use the following parameters for **Basics**
- - Resource Group: LABSECURITY
- - Storage account name: storagelabsecurity
- - - Note: you can use other name here, just remeber to take note
- - Location: East US
- - Performance: Standard
- - Account kind: StorageV2 (general purpose v2)
- - Replication: Locally-redundant storage (LRS)
- - Access tier (default): Hot
- Click **Next: Networking**

![img44](/img/img44.png)

- Use the following parameters for **Networking**
- - Connectivity method: Public endpoint (all networks)
- - Routing preference: Microsoft networking routing (default)
- Click **Next: Data protection**

![img45](/img/img45.png)

- Use the following parameters for **Data protection**
- - Blob soft delete: Disabled
- - File share soft delete: Disabled
- - Versioning: Disabled
- Click **Next: Advanced**

![img46](/img/img46.png)

- Use the following parameters for **Advanced**
- - Secure transfer required: Enabled
- - Blob public access: Disabled
- - Minimum TLS version: Version 1.0
- - Large file shares: Disabled
- - Hierarchical namespace: Disabled
- Click **Review + Create**
- Click **Create**

![img47](/img/img47.png)

![img48](/img/img48.png)

___

> 1. Enable Network Watcher on your region

- Open **Azure Portal** and type **Network Watcher** on **Search** bar. Press **Enter**

- On **Region** click to expand. Check if **East US** is enabled. If not click on "..." and the click **Enable network watcher** 

![img41](/img/img41.png)

> Note: if you cannot enable Network watcher then just follow this [documentation](https://docs.microsoft.com/en-us/azure/network-watcher/network-watcher-create) to manually register.

___

> 2. Forward NSG flow logs

- Locate the menu **Logs** and then click on **NSG flow logs**. Select the Resource Group **LABSECURITY**. The NSG **APPGLABSECURITY** will shown.

![img42](/img/img42.png)

- Click on **APPGLABSECURITY** NSG and use the following parameters:
- - Status: ON
- - Flow Logs version: Version 2
- - Storage Account: select **storagelabsecurity** or any other name that you had choosen before
- - Retention: 30
- - Traffic Analytics status: On
- - Traffic Analytics processing interval: Every 1 hour
- - - Note: you can change later to "every 10min" but for the first ingestion is recommended to wait at least few hours before making any change
- - Log Analytics workspace: select **WORKSPACESECURITY01**
- Click **Save**

![img49](/img/img49.png)

___

> 3. Forward Diagnostics logs

- Locate the menu **Logs** and then click on **Diagnostic logs**. Select the Resource Group **LABSECURITY**.

![img50](/img/img50.png)

- Click on **APPGW** and then on **+ Add diagnostic setting**

![img51](/img/img51.png)

- Use the following parameters:
- - Diagnostic settings name: APPGWDIAG
- - Check all 3 checkboxes for **log**
- - also check **AllMetrics** for **metric**
- - on **destination details** select **Send to Log Analytics**. Make sure that **WORKSPACESECURITY01** . Click **Save**

![img52](/img/img52.png)

- Enable Diagnostic logs for the rest of the resources

Repeat the steps for all resources. Use the same **Storage Account** and **Log Analytics Workspace**. You can use any name for **Diagnostics settings name** you want. 
After configuring all resources you will have something like this:

![img53](/img/img53.png)

- (Optional) Enable Traffic Analytics

If you have time just wait few hours and click on **Traffic Analytics** option on **Logs**. This dashboard show all the traffic to your public resources on Azure. Also you can check malicious flow to your resources on Azure, etc.

![img54](/img/img54.png)

![img55](/img/img55.png)

___

### Step 5 | Configure Security Center ###

Security Center can monitor both Azure and on-premises resources. First it's necessary to onboard the Azure Subscription to Standard,

___

> 1. Onboard Azure Subscription

Follow the steps bellow to enable **Standard** Tier. By default any Azure subscription is **Free**. 

Link: https://docs.microsoft.com/en-us/azure/security-center/security-center-get-started

___

> 2. Add Azure Application Gateway WAF source

On **Azure Security Center** click on **Security Solution** (located on **RESOURCE SECURITY HYGIENE**). Click on **ADD** button on **Azure Application Gateway WAF**. After that click on **Create**

![img56](/img/img56.png)

___

> 3. Enable data collection on Log Analytics workspace

- Click on **Pricing & settings** and then click on your workspace **WORKSPACESECURITY01**

![img57](/img/img57.png)

- Click on **Standard** and click **Save**

![img58](/img/img58.png)

- On the left side click on **Data collection** and select **All Events** . Click **Save**

![img59](/img/img59.png)

___


### Step 6 | Configure Azure Sentinel ###

Now you can connect **Log Analytics Workspace** to **Sentinel**. Follow the steps bellow:

> 1. Open **Azure Portal** and type **Sentinel** on **Search** bar.Click on **Azure Sentinel**.

> 2. On **Azure Sentinel workspaces** click on **+Add** button, select **WORKSPACESECURITY01** and click again on **Add Azure Sentinel** button.

![img60](/img/img60.png)

> 3. The Azure Sentinel dashboard will appear

![img61](/img/img61.png)

> 4. On the left side click on **Data connectors** (Configuration panel). Select **Azure Security Center** and then click on **Open connector page** (right side).

![img62](/img/img62.png)

> 5. Click on **Connect**.

![img63](/img/img63.png)

> 6. This step may not be required if you had previously configured **Diagnostic Logs** for **Application Gatewa**. Inf not just follow here: on the left side click on **Data connectors** (Configuration panel). Select **Microsoft web application firewall(WAF)** and then click on **Open connector page** (right side).

![img64](/img/img64.png)

> 7. On **Azure Sentinel** click on **Workbooks** (located at the left side | Threat management).  On **Templates** click on **Microsoft Web Application Firewall (WAF) - firewall events** and then click on **Save** (rigth side)

![img65](/img/img65.png)

> 8. A pop-up will appear to **Save workbook to...** and you can choose the same region.

> After saving you can click on **View saved workbook** on the right side. 

![img67](/img/img67.png)

> 9. Repeat the steps to add other 2 workbooks missing:

- Microsoft Web Application Firewall (WAF) - gateway access events  
- Microsoft Web Application Firewall (WAF) - overview  
  
  

### Step 7 | Test attacks ###

## Simulate attacks ##
### Warning: Don't do this on any other resource instead of this lab ###


- Vulnerability: Command injection  

- - Example 1
  
><code>127.0.0.1; ls -al</code>  

- - Example 2

><code>system("cd /var/yp && make &> /dev/null");</code>


- Vulnerability: SQL Injection

- - Example 1

><code>%’ or 1=’1</code>

- - Example 2

><code>SELECT * FROM members WHERE username = 'admin'--' AND password = 'password'</code>

- - Example 3

><code>SELECT /*!32302 1/0, */ 1 FROM tablename</code>

- - Example 4

><code>SELECT @@hostname;</code>

- - Example 5

><code>SELECT grantee, privilege_type, is_grantable FROM information_schema.user_privileges WHERE privilege_type = ‘SUPER’;SELECT host, user FROM mysql.user WHERE Super_priv = ‘Y’; # priv</code>


- Vulnerability: Cross-Site Scripting
><code>
<script>alert(“you have been hacked”)</script> </code>


### Step 8 | Detect attacks ###

- List all actions blocked by WAF:
><code>
search *  
| where (action_s == "Blocked")
</code>

- Matched/Blocked requests by IP
><code>
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| summarize count() by clientIp_s, bin(TimeGenerated, 1m)
| render timechart
</code>


## More documents and links about this topic ###



- https://docs.microsoft.com/en-us/azure/application-gateway/log-analytics
- - Official documentation about query logs from Application Gateway with WAF
- https://francescomolfese.it/en/2018/07/azure-application-gateway-come-monitorarlo-con-log-analytics/
- - MVP Francesco Molfese developed a good guide about how to integrate App Gateway WAF with Log Analytics.
- https://roykim.ca/2018/09/06/penetration-testing-your-web-app-with-azure-application-gateway-waf-part-3-log-analytics/
- - MVP Roy Kim developed a good post about how to query logs from Application Gateway with WAF
- https://owasp.org/www-community/attacks/
- - OWASP Fpundatiuon link about attacks
- https://github.com/ethicalhack3r/DVWA
- - GitHub REPO
- https://davidsr.me/index.php/2018/06/13/azure-waf-to-protect-your-web-application/
- - David Sanchez developed a guide to test some vulnerabilities on Application gateway with WAF

