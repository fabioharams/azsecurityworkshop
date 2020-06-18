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

[include requirements]

### step 1 | create resource group ###

### step 2 | create vnet ###

### step 3 | create subnets ###

[Bastion Subnet]
[Servers]
[App Gateway Subnet]
[DVWA Subnet]

### step 4 | create Linux VM ###

[Ubuntu18]
 
## Start the lab ##

### Step 1 | Install DVWA on UBUNTU ###

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
