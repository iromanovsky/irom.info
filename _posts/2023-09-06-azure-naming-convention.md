---
#layout: post-argon
title: Naming Convention for Azure Resources
date: 2023-09-06 08:00:00 +0100
last_modified_at: 2023-10-02 14:30:00 +0100
author: Igor
categories: [Azure, Foundations]
tags: [azure, devops, caf, naming-convention]
#permalink: /post1/
slug: azure-naming-convention
excerpt_separator: <!--more-->
#redirect_from: [/lorem, /post/lorem]
image: https://github.com/iromanovsky/irom.info/assets/15823576/b6ce746d-b7e8-48c1-b663-ac49cb298e07
description: >-
    Sharing the naming convention I've been using for the biggest Azure iInfrastructure projects during my career at Microsoft and EPAM since 2015.
#published: false
---
<div align="center">

![change-my-mind-naming](https://github.com/iromanovsky/irom.info/assets/15823576/b6ce746d-b7e8-48c1-b663-ac49cb298e07)

</div>

In this post, I'm sharing the naming convention I've been using for the biggest Azure infrastructure projects during my career at Microsoft and EPAM. Since 2015, it has proven to work well in enterprise-scale environments long before Microsoft introduced CAF and the concept of Landing Zones.

I've noticed that many customers tend to take CAF's guidance on naming conventions too literally, which is far from perfect when you move beyond test lab scales.

It's probably too late for you to change the naming convention of Azure resources, as the cloud adoption curve has been past its peak for years. So, now could be a good time to share my old "trade secret".

<!--more-->

## Naming principles
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
- Names may use both UPPERCASE and lowercase characters to support better readability and [accessibility](https://simple.wikipedia.org/wiki/Accessibility), where allowed by the resource type. Although some resources (like storage accounts) and other cloud providers (like AWS) have more restrictive rules on naming, they should not set the bar for the readability of all other Azure resources
- Names of globally unique resources like storage accounts should have short suffix derived from the company name, like '**iro**' (as abbreviation of my name) to avoid overlapping with resource names of other customers
- No need to spend time to define conventions for all possible resource types in Azure, using the same principles you can expand your convention when needed

## Naming sections

Below are the sections of the resource names in the order they should be used.
|Naming section | Description | Examples |
|--|--|--|
| System | To divide big company-wide systems, service lines, or business units from each other.<br/><br/>**Order position 1**, the highest unit of division. This could be a company-wide system name, like SAP, AVD, SharedServices; or an alias of a business unit if they prefer to manage multiple applications in a single group |SAP, AVD, Security, SharedSvc, PlatMgmt |
| Environment | To separate different environments of the same system. <br/><br/>**Order position 2**, since systems are often divided into different environments. Usually, the components inside the environment may share the same components, infrastructure, and security measures. |SBX, DEV, QA, NPR, PRD|
| Region | To accommodate expansions of the system to other regions without naming conflicts. <br/><br/>**Order position 3**, since the same environment often spans across multiple regions. Even if you never plan to expand your system to other regions, this may come in the future as a common architecture pattern for cloud high availability and disaster recovery. |EUW, EUN|
| Component | Optional name of system's component, such as application, workload, or service that the resource is a part of. <br/><br/>Used to distinguish subsystems within the same System. <br/><br/>**Order position 4**, since inside the same environment and region the system might have various logical parts. By dividing a system into logical components that share the same lifecycle and purpose, you leverage the intended use of _Resource Groups_ concept. | Network, ADDC, ADFS, NVA_Int, NVA_Ext, FrontEnd, BackEnd, App1_DB <br/><br/>_**Note** the optional instance indicator that could be included into this segment as a digit or text like DB, see description of virtual Instance section below_ |
|Resource type| Always at the end of the resource name, with some rare exceptions. <br/><br/>**Order position 5**, contrary to the popular example of the naming convention in CAF, resource type should be at the end of the name (suffix) to support the whole concept or name section ordering from big taxons to small. Putting the resource type at the beginning of the names (prefix) breaks this whole concept and leads to a naming standard that is not convenient to use on a big scale. | vnet, nsg, log, …|

### Virtual naming sections

The sections below are virtual -- they are constructed based on the sections defined above for the purpose of grouping and automation.

|Naming section | Description | Examples |
|--|--|--|
| Landing Zone | Constructed as 'System-Environment'. <br/> <br/>Usually, System-Environment forms a unit of separation of duties, security, and concerns, which corresponds to a model of Landing Zone, and is a good candidate for a dedicated subscription. | Security-PRD, SharedSvc-QA|
| Instance | **Optional** instance indicator for a specific resource, to differentiate it from other resources in the same logical array, like disks, virtual machine instances of the same service, and cluster nodes. <br/><br/>Many organizations used to use _padding_ for the instance count, no matter if the resource could even have multiple instances and how many of them are possible. <br/><br/>It is recommended to be wise and use padding only when necessary, for example, if in your organization you have only 6 domain controllers, no need to pad their instance counts into 4 digits, like DC0001, DC0002; padding with 2 digits like DC01, DC02 is enough for this purpose. | `NVAPEUWFW01`<br/>`NVAPEUWFW02`<br/> <small>instance count is padded as XX for 2 firewall instances, it supports upgrade and migration scenarios when you need room for more instances in the name</small> <br/><br/>`PlatConn-PRD-EUW-Hub-vnet-afw` <br/>`PlatCon-PRD-EUW-Hub-vnet-bas` <br/> <small>a vnet can have only one firewall and one bastion resource attached, so it does not make sense to include any instance count in the name of the resource</small> <br/><br/>`NVAPEUWFW01-D_LUN01-md` <br/>`NVAPEUWFW01-D_LUN02-md` <br/>`NVAPEUWFW01-D_Log1-md`<br/>`NVAPEUWFW01-D_Log2-md` <small>these are examples of multiple managed disk instances attached to the same vm with name NVAPEUWFW01</small> | 

<details>
<summary>[expand to see] A note on padding from CAF</summary>

>  <p>Padding improves readability and sorting of assets when those assets are managed in a configuration management database (CMDB), IT Asset Management tool, or traditional accounting tools. When the deployed asset is managed centrally as part of a larger inventory or portfolio of IT assets, the padding approach aligns with interfaces those systems use to manage inventory naming.</p>
>  <p>Unfortunately, the traditional asset padding approach can prove problematic in infrastructure-as-code approaches that might iterate through assets based on a non-padded number. This approach is common during deployment or automated configuration management tasks. Those scripts would have to routinely strip the padding and convert the padded number to a real number, which slows script development and run time.</p>
> <p>Choose an approach that's suitable for your organization. Before choosing a numbering scheme, with or without padding, evaluate what will affect long-term operations more: CMDB and asset management solutions or code-based inventory management. Then, consistently follow the padding option that best fits your operational needs.</p>
--- [CAF: Define your naming convention](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming)
</details><br/>

### Resource type abbreviations

The table below illustrates abbreviations for the most used resources. When expanding existing naming convention for new resource types, it's a good practice to use well-known abbreviations from [CAF: Abbreviation examples for Azure resources](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations).

| Resource type | Abbreviation |
|--|--|
|Resource Group | rg |
|Virtual Network | vnet|
|Network Security Group| nsg |
|Network Interface | nic|
|Managed Disk | md|
| .. | .. |

### Region abbreviations

Contrary to CAF which provides examples for region abbreviation as `westus`, `eastus2`, `westeu`; it's better to follow the logic of natural grouping by regions when alphabetically sorting the resources, so West and North Europe should be abbreviated as EUW and EUW, East and West US should be abbreviated USW and USE respectively.

| Region | Abbreviation |
|--|--|
|West Europe | EUW |
|North Europe | EUN |
|East US| USE |
|East US 2 | USE2 |
|Central US | USC |
| .. | .. |

_Note: The list of regions is constantly expanding, it will be challenging to accommodate all possible regions into 3 characters and keep it readable, so it's a good idea to have region abbreviation flexible._

### Naming restrictions

Please be aware that most Azure resources have restrictions in terms of length and characters, and may require to be globally unique across all Azure customers. 

Please refer to [Naming rules and restrictions for Azure resources](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-name-rules) for details and updates.

## Naming convention table

The table below describes common resource naming conventions with comments explaining cases _when naming does not exactly follow the common pattern_:

> \<System\>-\<Environment\>-\<Region\>[-Component]\-<res_type\> 

The table is supposed to be a living document, and be updated as new resource types are introduced into the environment.

| Resource Type | Scope | Format + Examples + Comments |
|:--|--|--|
| **Governance** |||
|Management Group | Tenant | **\<GroupName\>-mg** <br/><small>Display Name for MG can be different from MG actual resource name and is easy to change.<br/><br/>Even though management groups are rarely handled together with other resource types, it still makes sense for them to have a name suffix of (-mg) since it helps to distinguish -MGs from -Subs when only using names in diagrams and communications.</small><br/><br/> &bull; `Platform-mg` <br /> &bull; `SAP-PRD-mg` |
| Subscription | Account/Enterprise Agreement | **\<System\>\[-Env\]\[-Component\]-sub**<br/><small>Note that subscription names are easy to change when needed, however, it is still a good practice to keep subscription names consistent and aligned. Subscriptions might still benefit from having a resource type suffix (-sub), to help to distinguish them from management groups. </small> <br/><br/>&bull; `SAP-PRD-sub` <small>Note the omitted region segment since subscription by definition may contain resources from multiple regions</small><br/>&bull; `AVD-PRD-sub` <small>For subscriptions dedicated to a single system, the name of the subscription before (-sub) can form the Landing Zone prefix for all the resources inside.</small><br />&bull; `BUnit1-PRD-sub` <small>For subscriptions hosting multiple systems inside, the sub name without type extension (-sub) can still be used as a common prefix for all shared components, like networking.</small><br />&bull; `PlatformConnectivity-sub` <small>For platform related subscriptions, such as Connectivity, which can be shared among different environments, 'environment' section can be skipped if there is no plan to deploy non-production resources. </small> |
| Resource Group | Subscription | **\<System\>-\<Env\>\[-Component\]-rg** <br/><br/>&bull; `SAP-PRD-EUW-App1_DB-rg`<br>&bull; `PlatCon-PRD-EUW-Network-rg`<br>&bull; `PlatIdty-PRD-EUW-ADDC-rg`<br>&bull; `SharedSvc-PRD-EUW-Network-rg` |
| Policy Definition | Management Group or Subscription | **\<PolicyName\>-pol** <br/><small>Display Name for a Policy Definition is easy to change. Actual Policy definition resource names can even use a GUID pattern to ensure uniqueness, but you might wat to hve them stable to preserve policy assignments during policy definition updates.</small><br/><br/>&bull; `Custom-Enable-Storage-Diagnostics-pol` <br/>&bull; `Custom-Enable-NSG-Flow_Logs-pol` |
| Policy Set (Initiative) | Management Group or Subscription | **\<InitiativeName\>-ini** <br/><small>Display Name for a Policy Initiative is easy to change. Same as with policy definitions, you might wat to have them stable to preserve policy assignments during policy set updates.</small><br/><br/>&bull; `Custom-Required-Deploy-ini` <br/>&bull; `Custom-Optional-Monitor-ini` |
| **Networking** |||
| Virtual Network | Resource group | **\<System\>-\<Env\>-\<Region\>[-Component]-vnet** <br/><br/>&bull; `PlatCon-PRD-EUW-Ext-vnet`<br>&bull; `PlatCon-PRD-EUW-Hub-vnet`<br/><br/>&bull; `SAP-PRD-EUW-vnet` <small>Note that vnet name may omit the component segment (like Hub or Ext) if SAP-PRD Landing Zone is designed to have only one vnet.</small> |
| Subnet | Virtual network| **\<App\>[-Component]** <br/><small>Note that subnet names are just attributes of a vnet, there is no need to make them globally unique or specify CIDR inside the subnet name. Keeping subnet names short and simple helps when managing and diagnosing networking on Azure portal since shorter subnet names are displayed fully in the GUI and not truncated with ... in the most interesting place. </small><br/><br/>&bull; `NVA_Hub-Mgmt`<br>&bull; `AD-DC`<br>&bull; `App1-FE` <br/>&bull; `App1-BE`<br /><small>Using hyphen (-) to destingush different components of the same app</small><br/><br />&bull; `GatewaySubnet` <br/>&bull; `AzureBastionSubnet` <br/>&bull; `AzureFirewallSubnet` <br/><small>These subnet names are predifined. There is no need to have Subnet as a name suffix for non-predefined subnets.</small> |
| Azure Firewall |Resource group |**\<parent_vnet_name\>-afw** <br/><br/>&bull;  `PlatConn-PRD-EUW-Hub-vnet-afw`|
|Azure Bastion | Resource group| **\<parent_vnet_name\>-bas** <br/><br/>&bull;  `PlatCon-PRD-EUW-Hub-vnet-bas` |
| Virtual network gateway| Resource group | **\<parent_vnet_name\>-vnge** <br/><small>for ER gateway</small><br/>&bull; `PlatCon-PRD-EUW-Hub-vnet-vnge`<br /><br />**\<parent_vnet_name\>-vngv** <br /> <small>for VPN gateway</small><br/>&bull; `PlatCon-PRD-EUW-Hub-vnet-vngv` |
| Gateway connection | Resource group | **\<parent_vng_name\>[-Component]-con** <br/><small>for ER connection</small><br/>&bull;  `PlatCon-PRD-EUW-Hub-vnet-vnge-C_EquinixAMS-con`<br/><br/>**\<parent_vng_name\>[-Component]-con** <br/><small>for VPN S2S connection</small><br/>&bull;  `PlatCon-PRD-EUW-Hub-vnet-vngv-S_TSystemsFRA-con` |
|Local network gateway (VPN)| Resource group| **\<parent_vng_name\>[-Component]--lng** <br/><br/>&bull;  `PlatCon-PRD-EUW-Hub-vnet-vngv-L_TSystemsFRA-lng` |
| Route Table | Resource Group | **\<parent_vnet_name\>[-Component]-rt** <br/><small>for route tables deployed in the same resource group as vnet</small><br/>&bull; `PlatCon-PRD-EUW-Hub-vnet-R_Default-rt`<br/>&bull; `PlatCon-PRD-EUW-Hub-vnet-R_GatewaySubnet-rt` <br/><br/> **\<System\>-\<Env\>-\<Region\>[-Component]-rt** <br/><small>for route tables on app level, like deployed inside NVA resource group</small><br/>&bull; `SAP-PRD-EUW-App1_DB-rt`<br/>&bull; `SAP-PRD-EUW-App1_FE-rt` |
| Route | Route Table | \<PeeredVnetName\>[_#] <br/><small>for routes pointing to peered vnets, use names of target vnets as route names</small><br/>&bull; `PlatIdty-PRD-EUW-vnet`<br/><br />&bull; `SharedSvc-PRD-EUW-vnet_0`<br/>&bull; `SharedSvc-PRD-EUW-vnet_1` <br/><small>with optional instance numbers when the target vnet has multiple address space</small><br/><br /> **[RouteName]** <br/><small>this format is used for other custom routes</small><br/>&bull; `LAN_10`<br/>&bull; `LAN_192`<br/>&bull; `Default` |
| Network Security Group | Resource group | **\<parent_vnet_name\>[-Component]-nsg** <br/><small>for NSGs deployed in the same resource group as vnet</small><br/>&bull; `SAP-PRD-EUW-vnet-nsg`<br />&bull; `PlatCon-PRD-EUW-Hub-vnet-N_Diag-nsg` <br />&bull; `PlatCon-PRD-EUW-Hub-vnet-N_Default-nsg` <br/><br/> \<System\>-\<Env\>-\<Region\>[-Component]-nsg <br/><small>for NSGs on app level, when deployed inside the app's own resource group</small><br/>&bull; `PlatIdty-PRD-EUW-ADDC-nsg` |
| Load Balancer | Resource group| **\<System\>-\<Env\>-\<Region\>[-Component]-lbe** <br/><small>for External Load balancer</small><br/>&bull; `PlatCon-PRD-EUW-NVA_Ext-lbe` <br/><br/> **\<System\>-\<Env\>-\<Region\>[-Component]-lbi** <br/> <small>for Internal Load balancer</small><br/>&bull; `PlatCon-PRD-EUW-NVA_Ext-lbi` |
|Load Balancer configuration| Load Balancer| <small>While names of objects in Load Balancer configuration do not have to be unique outside the Load Balancer, the config itself might get quite complex and would benefit from using descriptive and well-thought names.</small> <br /><br />**\<FE_Name\>-fe** <br/><small>for Load balancer FrontEnd</small><br/>&bull; `FrontEnd1-fe` <br/><br/> **\<Probe_Name\>-probe** <br/><small>for Load balancer Probe</small><br/>&bull; `SSH-probe` <br/><br/> **\<Rule_Name\>-rule** <br/><small>for Load balancer Rule</small><br/>&bull; `HA-rule` <br/><br/> **\<BE_Name\>-be** <br/><small>for Load balancer BackEnd</small><br/>&bull; `BackEnd1-be` |
|Public IP| Resource group|**\<parent_name\>[-PIP_Name]-pip** <br/><small>Below are examples of various situations when PIPs are used</small><br/><br/>&bull; `PlatCon-PRD-EUW-Hub-vnet-bas-pip` <small>single PIP attached to a bastion</small> <br/>&bull; `PlatCon-PRD-EUW-Hub-vnet-vngv-IP2-pip` <small>multiple PIPs attached to a VPN gateway</small> <br/>&bull; `PlatCon-PRD-EUW-NVA-N_Ext-lbe-pip` <small>signle PIP attached to a load balancer</small> <br/>&bull; `App1-PRD-EUW-FrontEnd-lbe-IP24-pip` <small>multiple PIPs attached to a load balancer</small> <br/>&bull; `ADPEUWDC01-nic-pip` <small>single PIP attached to a VM with a single interface</small> <br/>&bull; `NVAEUWROS1-N_Mgmt-nic-pip` <small>single PIP attached to a VM with multiple interfaces</small> |
|DNS Label|Global|**\<FreeText\>-iro** <br/><small>Should be unique accross all customers in the region, it might need to include a company differentiator to distinguish from other customers</small><br/>&bull; `NVAEUWROS1-iro` |
| **Virtual Machines** |||
|Virtual Machine|Resource group <br/><small>but actually you might want to have it unique within your network</small>|<small>Names of VM resources on Azure should match OS hostnames running on those VMs for consistency. To [quote](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming) Microsoft: _<q>Keeping Azure VM names shorter than the naming restrictions of the OS helps create consistency, improve communication when discussing resources, and reduce confusion when you're working in the Azure portal while being signed in to the VM itself.</q>_<br/><br/> OS hostnames, in their turn, should follow existing corporate naming standards and comply with restrictions of operating systems, like using reduced character set (A-Z, 0-9, and '-' only), and length up to 15 characters (to be compatible with both [NETBIOS](https://learn.microsoft.com/en-us/troubleshoot/windows-server/identity/naming-conventions-for-computer-domain-site-ou#netbios-computer-names) and [DNS](https://learn.microsoft.com/en-us/troubleshoot/windows-server/identity/naming-conventions-for-computer-domain-site-ou#dns-host-names)). <br/><br/>However, if you have a luxuary of starting from a scratch, here is my suggestion: </small><br/><br/> **\<SYS\>\<ENV\>\<REG\>\<ROLE\>\<##[#]\>**<br/><small>where: <br/> - SYS: system abbreviation, 3-4 characters, like SAP, AD, NVA<br> - ENV: environment, 1-2 chars like S,D,Q,P<br>- REG: region, 2-3 chars like EUW, EUN<br />- ROLE: abbreviation of role of server in the system<br>- ## - 2-3 digits of intance counter</small><br/><br/>&bull; `ADPEUWDC01` <small>AD Prod EU_West DC 01</small> <br>&bull; `NVAPEUWFW01` <small>NVA Prod EU_West Firewall 01</small> <br>&bull; `SAPPEUWBWA01` <small>SAP Prod EU_West BW Application 01</small> <br/><br/><small>Note: Hostnames use only UPPERCASE for a reason, this helps visual accesibility when working with FQDNs, where domain names are traditionally lowercase, you get FQDN in the format `HOSTNAME.domain.name`</small>|
|NIC |Resource group|**\<parent_vm_name\>[-Nic_Name]-nic** <br/><br/>&bull; `ADPEUWDC01-nic`<br/>&bull; `NVAPEUWFW01-N_Int-nic`<br/>&bull; `NVAPEUWFW01_N_Ext-nic`|
|Managed Disk|Resource group|**\<parent_vm_name\>[-Disk_Name]-md** <br/><br/>&bull; `ADPEUWDC01-D_OS-md`<br>&bull;`IAUDEUWBT001-D_LUN1-md`|
|Availability Set |Resource group | **\<System\>-\<Env\>-\<Region\>[-Component]-as** <br/><br/>&bull; `PlatIdty-PRD-EUW-ADDC-as`<br/>&bull; `PlatCon-PRD-EUW-NVA-as`<br />&bull; `SAP-PRD-EUW-BWA-as` |
| **Storage** |||
|Storage account|Global|<small>This type of extremely popular resource has the strictest restrictions on naming: needs to be globally unique across all customers and regions of Azure Cloud, and use only 24 alphanumeric lowercase characters (a-z, 0-9). This makes storage account naming the toughest to make it human-readable. If nothing changes in Azure platform, at some point in time it could be required to use GUID-like storage account names and use different ways (like tags and friendly name aliases) to identify storage accounts.<br /><br />Below is the proposed approach to keep storage account names as human-accessible as possible.</small><br /><br />**\<sys\>\<env\>\<reg\>[comp]irosa**<br><small>To comply with SA restrictions, shorter naming segments (like for hostnames) should be used: <br/>- sys: abbreviation of system, like shsvc, pcon <br/>- env: environment, like pr, np, qa <br/>- reg: region, like euw, eun <br/>- comp: component abbreviation, like files, diag <br/>- iro: company abbreviation to reduce conflicts with other customers <br/>- sa: storage account resurce type</small> <br/><br/>&bull; `shsvcpreuwfilesirosa` <small>Shared_Services Prod EU_West Files </small> <br/>&bull; `shsvcpreuwdiagirosa` <small>Shared_Services Prod EU_West VM_Diagnistics </small> <br/>&bull; `sappreuwbwdatairosa` <small>SAP Prod EU_West BW_Data </small> |
|Recovery Service Vault|Resource group|**\<System\>-\<Env\>-\<Region\>[-Component]-rsv** <br> <small>alphanumerics and hyphens only</small> <br/>&bull; `PlatIdty-PRD-EUW-rsv`<br/>&bull; `SAP-NPR-EUW-BackupLRS-rsv`<br/>&bull; `SAP-PRD-EUW-BackupGRS-rsv`|
| **Key Vault** |||
| Key Vault |Global|**\<System\>-\<Env\>-\<Region\>[-Component]-irokv**<br><small>Like storage accounts, name is up to 24 characters, but you can use hyphens and alphanumerics in both lower and UPPER registers, which helps with reading accessibility. <br/><br />Might need the same solution as for storage accounts using GUID-like names with using other ways to provide human-friendly identification.</small><br /><br/>&bull; `PlatIdty-PRD-EUW-irokv`<br/>&bull; `ShSvc-PRD-EUW-Sec1-irokv`<br/>&bull; `IA-PRD-EUW-Enc-irokv`<br/>&bull; `SAP-PRD-EUW-01-irokv`<br/> <small>In specific cases naming parts can be truncated more to accommodate KV naming restrictions</small> |
|Key Vault objects |Key Vault|**\<Key_Name\>-key** <br/><small>for keys</small><br/>&bull; `DefaultDiskEncryption-key` <br/><br/> **\<Secret_Name\>-secret** <br/><small>for secrets</small><br/>&bull; `DomainJoinPass-secret`|
| **Common and Management resources** |||
|Log Analytics workspace |Resource group|**\<System\>-\<Env\>-\<Region\>[-Component]-log** <br/><br/>&bull; `PlatMgmt-PRD-EUW-Operations-log` <br/>&bull; `PlatMgmt-PRD-EUW-Security-log`|
|Automation Account |Resource group|**\<System\>-\<Env\>-\<Region\>[-Component]-aa** <br/><br/>&bull; `PlatMgmt-PRD-EUW-Mgmt-aa`|
|Private Endpoint |Resource group|**<parent_name>[-service]-pe** <br/><br/>&bull; `shsvcpreuwfilesirosa-file-pe`|
|**PaaS services** |||
|Azure Cache for Redis|Global|**\<system\>-\<env\>-\<region\>[-component]-iroredis** <br/><small>up to 63 chars, lowercase letters, numbers, and hyphens</small><br/>&bull; `sap-prd-euw-bwcache-iroredis`|
|Azure SQL Database|Global|**\<system\>\<env\>\<region\>[component]-irosql** <br/><small>up to 63 chars, lowercase letters, numbers, and hyphens</small><br/>&bull; `sap-prd-euw-bwdata-irosql`|
| .. and so on .. | ... | ...|

## Comparison with CAF convention
### What happens if CAF convention is followed literally

You probably know this picture? I see it cited all the time, like a bible. The issue with this picture is that it is _an example, not a bible_. Its intended purpose is to show the important naming parts, not to propose the exact naming convention.

<img src="https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/_images/ready/resource-naming.png" alt="Components of an Azure resource name" />

This is what happens when customers take this picture literally and turn it into their naming convention:

- Resource type is used as a prefix
- Resources on Azure Portal are sorted by resource type by default
- Parent-child relationships between resources are not visible, however you see mixed resources from different systems and environments together, which may lead to confusion and operating mistakes

<details>
<summary>[expand to see] the pictures...</summary>
<div style="max-width:80%;">

![naming-caf1](https://github.com/iromanovsky/irom.info/assets/15823576/84c7179d-b4b2-4920-9e8f-8ee5336e1720)
![naming-caf2](https://github.com/iromanovsky/irom.info/assets/15823576/ad63911b-990d-4704-bbf7-2bbe63ea066f)

</div>
</details><br/>

### How the proposed convention looks in real life

- Names constructed to reflect relations between resources, the resource type is used as a suffix 
- You can easily search resources and see related 'child' resources 
- You can still sort by resource type if needed
- The names that Azure generates automatically (and you cannot change them) look pretty similar

I can't share screenshots from customers' environments due to NDA restrictions and will update this post with screenshots from my lab environment when I get it rebuilt. Meanwhile, here is a list of typical resources of a small Landing Zone environment using the suggested naming convention, in YAML format: 

<details>
<summary>[expand to see] sample-landing-zone.yaml...</summary>

<!--
{% gist d2ff5edba7e48cc3044433a4cef7d3c4 sample-landing-zone.yaml %}
-->

```yaml
PlatCon-PRD-sub: # platform connectivity subscription
  - PlatCon-PRD-EUW-Network-rg:
      - PlatCon-PRD-EUW-Hub-vnet
      - PlatCon-PRD-EUW-Hub-vnet-afw # firewall
      - PlatCon-PRD-EUW-Hub-vnet-afw-pip
      - PlatCon-PRD-EUW-Hub-vnet-afw-Pol_1-afwp
      - PlatCon-PRD-EUW-Hub-vnet-afw-Pol_2-afwp
      - PlatCon-PRD-EUW-Hub-vnet-agw # app gateway
      - PlatCon-PRD-EUW-Hub-vnet-agw-mid
      - PlatCon-PRD-EUW-Hub-vnet-agw-pip
      - PlatCon-PRD-EUW-Hub-vnet-bas # bastion
      - PlatCon-PRD-EUW-Hub-vnet-bas-nsg
      - PlatCon-PRD-EUW-Hub-vnet-bas-pip
      - PlatCon-PRD-EUW-Hub-vnet-N_Diag-nsg # nsg
      - PlatCon-PRD-EUW-Hub-vnet-N_Default-nsg
      - PlatCon-PRD-EUW-Hub-vnet-R_GatewaySubnet-rt # route table
      - PlatCon-PRD-EUW-Hub-vnet-R_Default-rt
      - PlatCon-PRD-EUW-Hub-vnet-vngv # vpn gateway
      - PlatCon-PRD-EUW-Hub-vnet-vngv-IP1-pip
      - PlatCon-PRD-EUW-Hub-vnet-vngv-IP2-pip
      - PlatCon-PRD-EUW-Hub-vnet-vngv-S_Home-lng
      - PlatCon-PRD-EUW-nw # network watcher
      - pconpeuwhubirosa # storage
  - PlatCon-PRD-EUW-NVA-rg:
      - PlatCon-PRD-EUW-NVA-I_CHR_6.46.5-img
      - PlatCon-PRD-EUW-NVA-I_CHR_7.2.1-img
      - PlatCon-PRD-EUW-NVA-N_Ext-asg
      - PlatCon-PRD-EUW-NVA-N_Ext-lbe
      - PlatCon-PRD-EUW-NVA-N_Ext-lbe-pip
      - PlatCon-PRD-EUW-NVA-N_Int-asg
      - PlatCon-PRD-EUW-NVA-N_Int-lbi
      - PlatCon-PRD-EUW-NVA-N_Mgmt-asg
      - PlatCon-PRD-EUW-NVA-nsg
      - PlatCon-PRD-EUW-NVA-ROS-as
      - pconpeuwnvairosa # storage
      - NVAPEUWROS1 #NVA VM1
      - NVAPEUWROS1-D_OS-md
      - NVAPEUWROS1-N_Ext-nic
      - NVAPEUWROS1-N_Int-nic
      - NVAPEUWROS1-N_Mgmt-nic
      - NVAPEUWROS1-N_Mgmt-nic-pip
      - NVAPEUWROS2 #NVA VM2
      - NVAPEUWROS2-D_OS-md
      - NVAPEUWROS2-N_Ext-nic
      - NVAPEUWROS2-N_Int-nic
      - NVAPEUWROS2-N_Mgmt-nic
      - NVAPEUWROS2-N_Mgmt-nic-pip      

PlatMgmt-PRD-sub: # platform management subscription 
  - PlatMgmt-PRD-EUW-Network-rg:
      - PlatMgmt-PRD-EUW-vnet
      - PlatMgmt-PRD-EUW-vnet-nsg
      - PlatMgmt-PRD-EUW-vnet-rt
      - PlatMgmt-PRD-EUW-nw
  - PlatMgmt-PRD-EUW-Ops-rg:
      - PlatMgmt-PRD-EUW-Ops-aa
      - PlatMgmt-PRD-EUW-Ops-log

PlatIdty-PRD-sub: # platform identity subscription
  - PlatIdty-PRD-EUW-Network-rg:
      - PlatIdty-PRD-EUW-vnet
      - PlatIdty-PRD-EUW-vnet-nsg
      - PlatIdty-PRD-EUW-vnet-rt
      - PlatIdty-PRD-EUW-nw
  - PlatIdty-PRD-EUW-ADDC-rg:
      - ADPEUWDC01 #DC VM1
      - ADPEUWDC01-D_OS-md
      - ADPEUWDC01-D_LUN1-md
      - ADPEUWDC01-nic
      - ADPEUWDC02 #DC VM2
      - ADPEUWDC02-D_OS-md
      - ADPEUWDC02-D_LUN1-md
      - ADPEUWDC02-nic
      - PlatIdty-PRD-EUW-ADDC-as # availability set
      - PlatIdty-PRD-EUW-ADDC-nsg
      - pidpeuwdiagirosa # diagnostics storage

SharedSvc-PRD-sub: # shared services subscription
  - SharedSvc-PRD-EUW-Network-rg:
      - SharedSvc-PRD-EUW-AKS-vnet
      - SharedSvc-PRD-EUW-AKS-vnet-Cluster2_Main-nsg
      - SharedSvc-PRD-EUW-AKS-vnet-Cluster2_Virtual-nsg
      - SharedSvc-PRD-EUW-Apps-vnet
      - SharedSvc-PRD-EUW-Apps-vnet-nsg
      - SharedSvc-PRD-EUW-Apps-vnet-rt
      - SharedSvc-PRD-EUW-nw
  - SharedSvc-PRD-EUW-AKS_Cluster1-rg: []
  - SharedSvc-PRD-EUW-AKS_Cluster2-rg: []
  - SharedSvc-PRD-EUW-App1_FE-rg: []
  - SharedSvc-PRD-EUW-App1_BE-rg: []
```

</details><br/>

<!--
<script src="https://gist.github.com/iromanovsky/d2ff5edba7e48cc3044433a4cef7d3c4.js"></script>
-->

## Links

- This is a Microsoft article from golden times (2016) that got lost, but not forgotten: [Recommended naming conventions for Azure resources](https://github.com/MicrosoftDocs/azure-docs/blob/350b597783e508a3987a0e3116e12fee325bc9f3/articles/guidance/guidance-naming-conventions.md)

## Change my mind

The cover picture in this post is not a complete joke. Even though I've been using and refining this convention for so many years and customers, I'm eager to hear your opinion and change my mind if you propose justified improvements. 

I have not decided on the exact comment implementation for this blog yet, so in the meantime, you are very welcome to discuss this topic in [LinkedIn comments](https://www.linkedin.com/feed/update/urn:li:share:7105125783165095936/).
