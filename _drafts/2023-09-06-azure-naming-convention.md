---
#layout: post-argon
title:  Naming Convention for Azure Resources
date:   2023-09-06 08:00:00 +0100
author: Igor
categories: Blog
tags: [azure, devops, caf, naming-convention]
#permalink: /post1/
slug: azure-naming-convention
excerpt_separator: <!--more-->
#redirect_from: [/lorem, /post/lorem]
#published: false
---
<div align="center">
<img src="https://github.com/iromanovsky/irom.info/assets/15823576/d891e42e-8134-4a38-b53e-3bdf71ac7ce1" width="50%" height="50%" align="center">
</div>

<!--
![change-my-mind-naming](https://github.com/iromanovsky/irom.info/assets/15823576/d891e42e-8134-4a38-b53e-3bdf71ac7ce1)
-->

In this post, I'm sharing the naming convention I've been using for the biggest Azure infrastructure projects during my career at Microsoft and EPAM. Since 2015, it has proven to work well in enterprise-scale environments long before Microsoft introduced CAF and the concept of Landing Zones.

I've noticed that many customers tend to take CAF's guidance on naming conventions too literally, which is far from perfect when you move beyond test lab scales.

It's probably too late for you to change the naming convention of Azure resources, as the cloud adoption curve has been past its peak for years. So, now could be a good time to share my old "trade secret."

# Naming principles
<!--
$$
\underbrace{SAP}_{\text{< System >}} - \underbrace{PRD}_{\text{< Env >}} - \underbrace{EUW}_{\text{< Region >}} - \underbrace{App1\_DB}_{\text{[ Component ]}} - \underbrace{rg}_{\text{< res_type >}}
$$
-->
- Generic naming pattern is:

<div align="center">

![tex2img_equation](https://github.com/iromanovsky/irom.info/assets/15823576/b8947a35-790b-4fd9-aadc-b5ecac858e5a)

</div>

- Names are divided into sections. The sections could be \<Mandatory\> or [Optional]
- In most cases, a hyphen (-) is used to separate naming sections, and an underscore (_) can be used to divide parts within the same section. Other symbols are less suitable as separators, because of worse compatibility with URL and DNS standards, along with existing Azure resource [naming restrictions](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-name-rules).
- Sections are ordered to have bigger ['taxons'](https://simple.wikipedia.org/wiki/Taxon) first.<br/><br/>
_System > Environment > Region > Component > Instance > Resource_Type_
<br/><br/>
This helps to navigate, search, and keep the structure when resources are sorted alphabetically
- Names of logically 'child' resources are appended to the 'parent’s' name. This helps to have 'child' resources grouped with their parents when sorted alphabetically.<br/><br/>
For example, virtual network gateways are attached to a virtual network, so they are considered 'child' resources and displayed together with their parent when resources are sorted alphabetically or searched:
  - `PlatCon-PRD-EUW-Hub-vnet` (for virtual network)
  - `PlatCon-PRD-EUW-Hub-vnet-vnge` (for express route gateway)
  - `PlatCon-PRD-EUW-Hub-vnet-vngv` (for VPN gateway) 
  - `PlatCon-PRD-EUW-Hub-vnet-vngv-T_Systems-con` (for VPN gateway connection)
- It should be possible to skip some sections where they are not applicable like there is no region for some kinds of resources
- There could be exclusions from the generic pattern (for instance, VM names are better to specify the same as OS hostname to avoid many hidden stones; there could be names that cannot include some characters, like storage accounts)
- Names may use both uppercase and lowercase characters to support better readability and [accessibility](https://simple.wikipedia.org/wiki/Accessibility), where allowed by the resource type. Although some resources have more restrictions on names, like storage accounts, they should not set the bar for the readability of all other resources
- Names of globally unique resources like storage accounts should have short suffix derived from the company name, like '**iro**' (as abbreviation of my name) to avoid overlapping with resource names of other customers
- No need to spend time to define conventions for all possible resource types in Azure, using the same principles you can expand your convention when needed

# Naming sections

Below are the sections of the resource names in the order they should be used.
|Naming section | Description | Examples |
|--|--|--|
| System | To divide big company-wide systems, service lines, or business units from each other.<br/><br/>**Order position 1**, the highest unit of division. This could be a company-wide system name, like SAP, AVD, SharedServices; or an alias of a business unit if they prefer to manage multiple applications in a single group |SAP, AVD, Security, SharedSvc, PlatMgmt |
| Environment | To separate different environments of the same system. <br/><br/>**Order position 2**, since systems are often divided into different environments. Usually, the components inside the environment may share the same components, infrastructure, and security measures. |SBX, DEV, QA, NPR, PRD|
| Region | To accommodate expansions of the system to other regions without naming conflicts. <br/><br/>**Order position 3**, since the same environment often spans across multiple regions. Even if you never plan to expand your system to other regions, this may come in the future as a common architecture pattern for cloud high availability and disaster recovery. |EUW, EUN|
| Component | Optional name of system's component, such as application, workload, or service that the resource is a part of. <br/><br/>Used to distinguish subsystems within the same System. <br/><br/>**Order position 4**, since inside the same environment and region the system might have various logical parts. By dividing a system into logical components that share the same lifecycle and purpose, you leverage the intended use of _Resource Groups_ concept. | Network, ADDC, ADFS, NVA_Int, NVA_Ext, FrontEnd, BackEnd, App1_DB <br/><br/>_**Note** the optional instance indicator that could be included into this segment as a digit or text like DB, see description of virtual Instance section below_ |
|Resource type| Always at the end of the resource name, with some rare exceptions. <br/><br/>**Order position 5**, contrary to the popular example of the naming convention in CAF, resource type should be at the end of the name (suffix) to support the whole concept or name section ordering from big taxons to small. Putting the resource type at the beginning of the names (prefix) breaks this whole concept and leads to a naming standard that is not convenient to use on a big scale. | vnet, nsg, log, …|

Virtual naming sections:

The sections below are virtual -- they are constructed based on the sections defined above for the purpose of grouping and automation.

|Naming section | Description | Examples |
|--|--|--|
| Landing Zone | Constructed as 'System-Environment'. <br/> <br/>Usually, System-Environment forms a unit of separation of duties, security, and concerns, which corresponds to a model of Landing Zone, and is a good candidate for a dedicated subscription. | Security-PRD, SharedSvc-QA|
| Instance | **Optional** instance indicator for a specific resource, to differentiate it from other resources in the same logical array, like disks, virtual machine instances of the same service, and cluster nodes. <br/><br/>Many organizations used to use _padding_ for the instance count, no matter if the resource could even have multiple instances and how many of them are possible.  <br/><br/>It is recommended to be wise and use padding only when necessary, for example, if in your organization you have only 6 domain controllers, no need to pad their instance counts into 4 digits, like DC0001, DC0002; padding with 2 digits like DC01, DC02 is enough for this purpose. | `NVAPEUWFW01`<br/>`NVAPEUWFW02`<br/> <small>instance count is padded as XX for 2 firewall instances, it supports upgrade and migration scenarios when you need room for more instances in the name</small> <br/><br/>`PlatConn-PRD-EUW-Hub-vnet-afw` <br/>`PlatCon-PRD-EUW-Hub-vnet-bas` <br/> <small>a vnet can have only one firewall and one bastion resource attached, so it does not make sense to include any instance count in the name of the resource</small> <br/><br/>`NVAPEUWFW01-D_LUN01-md` <br/>`NVAPEUWFW01-D_LUN02-md` <br/>`NVAPEUWFW01-D_Log1-md`<br/>`NVAPEUWFW01-D_Log2-md` <small>these are examples of multiple managed disk instances attached to the same vm with name NVAPEUWFW01</small> | 

<details>
<summary>Note on padding from CAF</summary>

> [CAF: Define your naming convention](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming)
<br/><br/>Padding improves readability and sorting of assets when those assets are managed in a configuration management database (CMDB), IT Asset Management tool, or traditional accounting tools. When the deployed asset is managed centrally as part of a larger inventory or portfolio of IT assets, the padding approach aligns with interfaces those systems use to manage inventory naming.
<br/><br/>Unfortunately, the traditional asset padding approach can prove problematic in infrastructure-as-code approaches that might iterate through assets based on a non-padded number. This approach is common during deployment or automated configuration management tasks. Those scripts would have to routinely strip the padding and convert the padded number to a real number, which slows script development and run time.
<br/><br/>Choose an approach that's suitable for your organization. Before choosing a numbering scheme, with or without padding, evaluate what will affect long-term operations more: CMDB and asset management solutions or code-based inventory management. Then, consistently follow the padding option that best fits your operational needs.

</details>

## Resource type abbreviations
The table below illustrates abbreviations for the most used resources. When expanding existing naming convention for new resource types, it's a good practice to use well-known abbreviations from [CAF: Abbreviation examples for Azure resources](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations).

| Resource type | Abbreviation |
|--|--|
|Resource Group | rg |
|Virtual Network | vnet|
|Network Security Group| nsg |
|Network Interface | nic|
|Managed Disk | md|
| .. | .. |

## Region abbreviations
Contrary to CAF which provides examples for region abbreviation as westus, eastus2, westeu, usva, ustx; it's better to follow the logic of natural grouping by regions when alphabetically sorting the resources, so West and North Europe should be abbreviated as EUW and EUW, East and West US  should be abbreviated USW and USE respectively.

| Region | Abbreviation |
|--|--|
|West Europe | EUW |
|North Europe | EUN |
|East US| USE |
|East US 2 | USE2 |
|Central US | USC |
| .. | .. |

_Note: The list of regions is constantly expanding, it will be challenging to accommodate all possible regions into 3 characters and keep it readable, so it's a good idea to have region abbreviation flexible._

## Naming restrictions
Please be aware that most Azure resources have restrictions in terms of length and characters, and may require to be globally unique across all Azure customers. 

Please refer to [Naming rules and restrictions for Azure resources](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-name-rules) for details and updates.

# Naming convention table
The table below describes common resource naming conventions with comments explaining cases _when naming does not exactly follow the common pattern_:

> \<System\>-\<Environment\>-\<Region\>[-Component]\-<res_type\> 

The table is supposed to be a living document as new resource types are introduced into the environment.

| Resource Type |  Scope | Format + Examples + Comments |
|--|--|--|
| **Governance** |
|Management Group | Tenant | \<GroupName\>-mg <br/><small>Display Name for MG can be different from MG actual resource name and is easy to change.</small><br/><br/><small>Even though management groups are rarely handled together with other resource types, it still makes sense for them to have a name suffix of (-mg) since it helps to easier distinguish -MGs from -Subs when only using names in diagrams and communications.</small><br/><br/>  `Platform-mg`<br/>`SAP-PRD-mg`  |
| Subscription | Account/Enterprise Agreement | \<System\>-\<Env\>[-Component]-sub<br/><small>Note that subscription names are easy to change when needed, however, it is still a good practice to keep subscription names consistent and aligned.</small> <br/><br/><small>For subscriptions dedicated to a single system, the name of the subscription before (-sub) can form the Landing Zone prefix for all the resources inside. For subscriptions hosting multiple systems inside, the sub name without type extension (-sub) can still be used as a common prefix for all shared components, like networking. <br/><br/>Subscribtions might still benefit from having  a resource type suffix (-sub), to help to distinguish them from management groups. </small> <br/><br/> `SAP-PRD-sub`<br><small>Note the omitted region segment since subscription by definition may contain resources from multiple regions</small><br/><br/>`PlatCon-PRD-sub`<br/>`Platform-Connectivity-sub` <br/> <small>For platform related subscriptions, such as Connectivity, which can be shared among different environments, 'environment' section can be skipped if there is no plan to deploy non-production resources. </small>  |
| Resource Group | Subscription |  \<System\>-\<Env\>[-Component]-rg <br/><br/> `SAP-PRD-EUW-App1_DB-rg`<br>  `PlatCon-PRD-EUW-Network-rg`<br> `PlatIdty-PRD-EUW-ADDC-rg`<br>`SharedSvc-PRD-EUW-Network-rg`|
| Policy Definition | Management Group or Subscription | \<PolicyName\>-pol <br/><small>Display Name for a Policy Definition is easy to change. Actual Policy definition resource names can even use a GUID pattern to ensure uniqueness.</small><br/><br/> `Custom-Enable-Storage-Diagnostics-pol` <br/>`Custom-Enable-NSG-Flow_Logs-pol` |
| Policy Initiative | Management Group or Subscription | \<InitiativeName\>-ini <br/><small>Display Name for a Policy Initiative is easy to change.</small><br/><br/> `Custom-Required-Deploy-ini` <br/>`Custom-Optional-Monitor-ini` |
| **Networking** |
| Virtual Network | Resource group | \<System\>-\<Env\>-\<Region\>[-Component]-vnet <br/><br/> `PlatCon-PRD-EUW-Ext-vnet`<br>`PlatCon-PRD-EUW-Hub-vnet`<br/><br/>`SharedSvc-PRD-EUW-vnet` <br/><small>Note that vnet name may omit the component segment (like Hub or Ext) if SharedSvc-PRD Landing Zone is designed to have only one vnet.</small> |
| Subnet | Virtual network| \<App\>[-AppComponent] <br/><small>Note that subnet names are just attributes of a vnet, there is no need to make them globally unique or specify CIDR inside the subnet name. Keeping subnet names short and simple helps when managing or diagnosing networking on Azure portal, since subnet names are displayed fully in the GUI and not truncated with ... in the most interesting place. </small><br/><br/> `NVA_Hub-Mgmt`<br>`ADDC`<br>`IA-DevDefault` <br/><br/>`GatewaySubnet` <br/>`AzureBastionSubnet` <br/>`AzureFirewallSubnet` <br/><small>These subnet names are predifined. There is no need to have Subnet as a name suffix for non-predefined subnets.</small> |
| Azure Firewall |Resource group |<vnet_name>-afw <br/><br/> `PlatConn-PRD-EUW-Hub-vnet-afw`|
|Azure Bastion | Resource group| <vnet_name>-bas <br/><br/> `PlatCon-PRD-EUW-Hub-vnet-bas`|
| Virtual network gateway| Resource group | <vnet_name>-vngv <br/><small>for VPN gateway</small><br/><br/> `PlatCon-PRD-EUW-Hub-vnet-vngv`<br/><br/><vnet_name>-vnge <br/><small>for ER gateway</small><br/><br/> `PlatCon-PRD-EUW-Hub-vnet-vnge` |
| Gateway connection | Resource group | <vng_name>[-Component]-con <br/><small>for ER connection</small><br/><br/> `PlatCon-PRD-EUW-Hub-vnet-vnge-C_EquinixAMS-con`<br/><br/><vng_name>[-Component]-con <br/><small>for VPN S2S connection</small><br/><br/> `PlatCon-PRD-EUW-Hub-vnet-vngv-S_TSystemsFRA-con` |
|Local network gateway (VPN)| Resource group| <vng_name>[-Component]-lng <br/><br/> `PlatCon-PRD-EUW-Hub-vnet-vngv-L_TSystemsFRA-lng`|
| Route Table | Resource Group | <vnet_name>[-Component]-rt <br/><small>For route tables deployed in the same resource group as vnet</small><br/> `PlatCon-PRD-EUW-Hub-vnet-R_Default-rt`<br/>`PlatCon-PRD-EUW-Hub-vnet-R_GatewaySubnet-rt` <br/><br/>  \<System\>-\<Env\>-\<Region\>[-Component]-rt <br/><small>For route tables on app level, like deployed inside NVA resource group</small><br/> `SharedSvc-PRD-EUW-App1_DB-rt`<br/>`SharedSvc-PRD-EUW-App1_FE-rt` |
| Route | Route Table | \<PeeredVnetName\>[_#] <br/><small>for routes to peered vnets we use names of target vnets as route names with optional instance numbers when the target vnet has multiple address spaces</small><br/> `PlatIdty-PRD-EUW-vnet`<br/>`SharedSvc-PRD-EUW-vnet_0`<br/>`SharedSvc-PRD-EUW-vnet_1` <br/><br/> [RouteName] <br/><small>this format is used for other custom routes</small><br/> `LAN_10`<br/>`LAN_192`<br/>`Default`||
| Network Security Group | Resource group | \<vnet_name\>-nsg <br/><small>For NSGs deployed in the same resource group as vnet</small><br/> `PlatCon-PRD-EUW-Hub-vnet-nsg` <br/><br/>  \<System\>-\<Env\>-\<Region\>[-Component]-nsg <br/><small>For NSGs on app level, like deployed inside NVA resource group</small><br/> `PlatIdty-PRD-EUW-ADDC-nsg`  |
| Load Balancer | Resource group| \<System\>-\<Env\>-\<Region\>[-Component]-lbe <br/><small>for External Load balancer</small><br/> `PlatCon-PRD-EUW-NVA_Ext-lbe` <br/><br/> \<System\>-\<Env\>-\<Region\>[-Component]-lbi <br/><small>for Internal Load balancer</small><br/> `PlatCon-PRD-EUW-NVA_Ext-lbi` |
|Load Balancer attributes| Load Balancer| \<Name\>-fe <br/><small>for Load balancer FrontEnd</small><br/> `FrontEnd1-fe` <br/><br/> \<Name\>-probe <br/><small>for Load balancer Probe</small><br/> `SSH-probe` <br/><br/> \<Name\>-rule <br/><small>for Load balancer Rule</small><br/> `HA-rule` <br/><br/> \<Name\>-be <br/><small>for Load balancer BackEnd</small><br/> `BackEnd1-be` | 
|Public IP| Resource group|\<parent_name\>[-PipName]-pip <br/><small>Below are examples of various situations when PIPs are used</small><br/><br/> `PlatCon-PRD-EUW-Hub-vnet-bas-pip` <small>single PIP attached to a bastion</small> <br/> `PlatCon-PRD-EUW-Hub-vnet-vngv-IP2-pip` <small>multiple PIPs attached to a VPN gateway</small> <br/> `PlatCon-PRD-EUW-NVA-N_Ext-lbe-pip` <small>signle PIP attached to a load balancer</small> <br/> `App1-PRD-EUW-FrontEnd-lbe-IP24-pip` <small>multiple PIPs attached to a load balancer</small> <br/> `ADPEUWDC01-nic-pip` <small>single PIP attached to a VM with a single interface</small> <br/> `NVAEUWROS1-N_Mgmt-nic-pip` <small>single PIP attached to a VM with multiple interfaces</small> |
|DNS Label|Global|\<FreeText\>-iro <br/><small>Keep unique but short, include region and company indicator to distinguish from other customers</small><br/> `NVAEUWROS1-iro` |
| **Virtual Machines** |
|Virtual Machine|Resource group <br/><small>but actually you might want to have it unique within your network</small>|<small>Names of VM resources on Azure should match OS hostnames running on those VMs for consistency. To [quote](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming) Microsoft: _<q>Keeping Azure VM names shorter than the naming restrictions of the OS helps create consistency, improve communication when discussing resources, and reduce confusion when you're working in the Azure portal while being signed in to the VM itself.</q>_<br/><br/> OS hostnames, in their turn, should follow existing corporate naming standards and comply with restrictions of operating systems, like using reduced character set (A-Z, 0-9, and '-' only), and length up to 15 characters (to be compatible with both [NETBIOS](https://learn.microsoft.com/en-us/troubleshoot/windows-server/identity/naming-conventions-for-computer-domain-site-ou#netbios-computer-names) and [DNS](https://learn.microsoft.com/en-us/troubleshoot/windows-server/identity/naming-conventions-for-computer-domain-site-ou#dns-host-names)). <br/><br/>However, if you have a luxuary of starting from a scratch, here is my suggestion: </small><br/><br/> \<SYSTEM\>\<ENV\>\<REGION\>\<ROLE\>\<##[#]\><br/><small>where: <br/> - SYSTEM - 3-4 characters<br> - ENV - one of S,D,Q,P<br> - ROLE - role of server in system, 2 characters<br>- ## - 2-3 digits of intance counter</small><br/><br/> `ADPEUWDC01` <small>AD Prod EU_West DC 01</small> <br>`NVAPEUWFW01` <small>NVA Prod EU_West Firewall 01</small> <br>`SAPPEUWBWA01` <small>SAP Prod EU_West BW Application 01</small> <br/><br/><small>Note the last trick: hostnames use only UPPERCASE for a reason, this helps visual accesibility when working with FQDNs, where domain names are traditionally lowercase, you get FQDN in the format `HOSTNAME.domain.name`</small>|
|NIC |Resource group|\<parent_vm_name\>[-NicName]-nic <br/><br/> `ADPEUWDC01-nic`<br/>`NVAPEUWFW01-N_Int-nic`<br/>`NVAPEUWFW01_N_Ext-nic`|
|Managed Disk|Resource group|\<parent_vm_name\>[-DiskName]-md <br/><br/> `ADPEUWDC01-D_OS-md`<br>`IAUDEUWBT001-D_LUN1-md`|
|Availability Set |Resource group | \<System\>-\<Env\>-\<Region\>[-Component]-as <br/><br/> `PlatIdty-PRD-EUW-ADDC-as`<br/>`IA-PRD-EUW-Bots-as` <br/> `PlatCon-PRD-EUW-NVA-as` |
| **Storage** |
|Storage account|Global|\<sys\>\<env\>\<reg\>[comp]irosa<br><small>To comply with SA restriction of 24 alphanumeric characters (A-Z, 0-9), shorter naming segments should be used: <br/>- sys: brief abbreviation of system like  shsvc, pcon <br/>- env: brief environment like pr, np, qa <br/>- reg: brief region like euw, eun <br/>- comp: brief component name like files, diag <br/>- iro: brief company abbreviation to reduce conflicts with other customers <br/>- sa: storage account resurce type</small> <br/><br/> `shsvcpreuwfilesirosa` <br/> <small>Shared_Services Prod EU_West Files </small> <br/> `shsvcpreuwdiagirosa` <br/> <small>Shared_Services Prod EU_West VM_Diagnistics </small> <br/> `sappreuwbwdatairosa` <br/> <small>SAP Prod EU_West BW_Data </small> |
|Recovery Service Vault|Resource group|\<System\>-\<Env\>-\<Region\>[-Component]-rsv <br> <small>alphanumerics and hyphens only</small> <br/><br/> `PlatIdty-PRD-EUW-rsv`<br/>`IA-NPR-EUW-BackupLRS-rsv`<br/>`IA-PRD-EUW-BackupGRS-rsv`|
| **Key Vault** |
| Key Vault |Global|\<System\>-\<Env\>-\<Region\>[-Component]-irokv<br><small>like storage accounts, name is up to 24 characters, but you can use hyphens and alphanumerics in both lower and UPPER registers, which helps with reading accessibility</small> <br/><br/> `PlatIdty-PRD-EUW-irokv`<br/>`ShSvc-PRD-EUW-Sec1-irokv`<br/>`IA-PRD-EUW-Enc-irokv`<br/>`SAP-PRD-EUW-01-irokv`<br/> <small>In specific cases naming parts can be truncated more to accommodate KV naming restrictions</small> |
|Key Vault objects |Key Vault|\<Name\>-key <br/><small>for keys</small><br/> `DefaultDiskEncryption-key` <br/><br/> \<Name\>-secret <br/><small>for secrets</small><br/> `DomainJoinPass-secret`|
| **Common and Management resources** |
|Log Analytics workspace |Resource group|\<System\>-\<Env\>-\<Region\>[-Component]-log <br/><br/> `PlatMgmt-PRD-EUW-Operations-log` <br/> `PlatMgmt-PRD-EUW-Security-log`|
|Automation Account |Resource group|\<System\>-\<Env\>-\<Region\>[-Component]-aa <br/><br/> `PlatMgmt-PRD-EUW-Mgmt-aa`||
|Private Endpoint |Resource group|<parent_name>[-service]-pe <br/><br/> `shsvcpreuwfilesirosa-file-pe`|
|**PaaS services** |
|Azure Cache for Redis|Global|\<system\>-\<env\>-\<region\>[-component]-iroredis <br/><small>up to 63 chars, lowercase letters, numbers, and hyphens</small><br/> `sap-prd-euw-bwcache-iroredis`|
|Azure SQL Database|Global|\<system\>\<env\>\<region\>[component]-irosql <br/><small>up to 63 chars, lowercase letters, numbers, and hyphens</small><br/> `sap-prd-euw-bwdata-irosql`|
| .. and so on .. | ... | ...|

# Comparison with CAF convention
## What happens if CAF convention is followed literally

You probably know this picture? I see it cited all the time, like a bible. The issue with this picture is that it is _an example, not a bible_. Its intended purpose is to show the important naming parts, not to propose the exact naming convention.

<img src="https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/_images/ready/resource-naming.png"  alt="Components of an Azure resource name" />

This is what happens when customers take this picture literally and turn it into their naming convention:

- Resource type is used as a prefix
- Resources on Azure Portal are sorted by resource type by default
- Parent-child relationships between resources are not visible, however you see resources from different systems and environments together, which may lead to confusion and operating mistakes

<details>
<summary>Expand to see the pictures...</summary>
<div style="max-width:50%;">
  
![naming-caf1](https://github.com/iromanovsky/irom.info/assets/15823576/84c7179d-b4b2-4920-9e8f-8ee5336e1720)
![naming-caf2](https://github.com/iromanovsky/irom.info/assets/15823576/ad63911b-990d-4704-bbf7-2bbe63ea066f)

</div>
</details>

## How the proposed convention looks in real life
- Names constructed to reflect relations between resources, the resource type is used as a suffix 
- You can easily search resources and see related 'child' resources 
- You can still sort by resource type if needed
- The names that Azure generates automatically (and you cannot change them) look pretty similar

I can't share screenshots from customers' environments due to NDA restrictions and will update this post with screenshots from my lab environment when I get it rebuilt. Meanwhile, here is a list of typical resources of a small Landing Zone environment using the suggested naming convention, in YAML format: 

# Discussion

The cover picture in this post is not a complete joke. Even though I've been using and refining this convention for so many years and customers, I'm eager to hear your opinion and change my mind if you propose justified improvements. 

I have not decided on the exact comment implementation for this blog yet, so in the meantime, you are very welcome to discuss this topic in LinkedIn comments:
