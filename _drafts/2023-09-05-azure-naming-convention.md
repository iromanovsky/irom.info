---
#layout: post-argon
title:  Naming Convention for Azure Resources
date:   2023-09-05 08:00:00 +0100
author: Igor
categories: Blog
tags: [azure, devops, caf, naming-convention]
#permalink: /post1/
slug: azure-naming-convention
excerpt_separator: <!--more-->
#redirect_from: [/lorem, /post/lorem]
#published: false
---


# Picture

![change-my-mind-naming](https://github.com/iromanovsky/irom.info/assets/15823576/d891e42e-8134-4a38-b53e-3bdf71ac7ce1)


# Naming principles
<!--
$$
\underbrace{SAP}_{\text{< System >}} - \underbrace{PRD}_{\text{< Env >}} - \underbrace{EUW}_{\text{< Region >}} - \underbrace{App1\_DB}_{\text{[ Component ]}} - \underbrace{rg}_{\text{< res_type >}}
$$
-->

![tex2img_equation](https://github.com/iromanovsky/irom.info/assets/15823576/b8947a35-790b-4fd9-aadc-b5ecac858e5a)

- Names are divided into sections. The sections could be \<Mandatory\> or [Optional]
- In most cases, dash (-) is used to separate naming sections and underscore (_) can be used to divide words within the same section
- Sections are ordered to have bigger ['taxons'](https://simple.wikipedia.org/wiki/Taxon) first.<br/><br/>
_System > Environment > Region > Component > Instance > Resource_Type_
<br/><br/>
This helps to navigate, search, and keep the structure when resources are sorted alphabetically
- Names of logically 'child' resources are appended to the 'parent’s' name. This helps to have child resources grouped with their parents when sorted alphabetically.<br/><br/>
For example, virtual network gateways are attached to a virtual network, so they are considered child resources, and displayed together with their parent:
  - `PlatCon-PRD-EUW-Hub-vnet` (for virtual network)
  - `PlatCon-PRD-EUW-Hub-vnet-vnge` (for express route gateway)
  - `PlatCon-PRD-EUW-Hub-vnet-vngv` (for VPN gateway)
- Generic naming pattern is: <br/><br/>
<System>-<ENV>-<REGION>[-Component]-<resource_type>
   - Simple Example: System-PRD-EUW-vnet
   - Complex Example: PlatCon-PRD-EUW-Hub-vnet-vngv-T_Systems-con
- It should be possible to skip some sections where they are not applicable like there is no region for some kinds of resources
- There could be exclusions from generic pattern  (for instance, vm names are better to specify the same as OS hostname to avoid many hidden stones; there could be names that cannot include some characters, like storage accounts)
- Names may use both uppercase and lowercase characters to support better readability and [accessibility](https://simple.wikipedia.org/wiki/Accessibility), where allowed by the resource type. Although some resources have stricter limitations on names, like storage accounts, they should not set the bar for the readability of all other resources
- Names of globally unique resources like storage accounts should have short suffix derived from the company name, like '**epm**' (to abbreviate EPAM) to avoid overlapping resource names with other customers
- No sense to define conventions for all possible objects, use the same principles as above

# Naming sections

Below are the sections of the resource names in the order they should be used.
|Naming section | Description | Examples |
|--|--|--|
| System | To divide big company-wide systems or service lines from each other.<br/><br/>**Order priority 1**, the highest unit of division. This could be a company-wide system name, like SAP, AVD, SharedServices; or an alias of a business unit if they prefer to manage multiple applications in a single group |Security, SharedSvcs, PlatCon, PlatMgmt, PlatIdty|
| Environment | To separate different environments of the same system <br/><br/>**Order priority 2** since systems are often divided into different environments. Usually the components inside the environment share the same access and security measures. |SBX, DEV, QA, NPR, PRD|
| Region | To accommodate expansions of the system to other regions without naming conflict <br/><br/>**Order priority 3**, since the environment often spans across multiple regions for high availability |EUW, EUN|
| Component | Optional name of system's component, such as application, workload, or service that the resource is a part of. <br/><br/>Used to distinguish subsystems within the same System. <br/><br/>**Order priority 4**, since inside the same environment and region the system might have various logical parts. By dividing a system into logical components that share the same lifecycle and purpose, you leverage the intended use of _Resource Groups_ concept  | Network, ADDC, ADFS, NVA_Int, NVA_Ext, FrontEnd, BackEnd, App1_DB … <br/><br/>_**Note** the optional instance number that could be included into the component segment, see description of virtual Instance section below_ |
|Resource type| Always at the end of the object name ('suffix' pattern), with some rare exceptions, this greatly helps when scripting <br/><br/>**Order priority 5**, contrary to the popular example of naming convention in CAF, resource type should be at the end of the name (suffix) to support the whole concept or name section ordering from big taxons to small. Putting the resource type at the beginning of the names (prefix) breaks this whole concept and leads to naming standard which is not convenient to use in the big scale | vnet, nsg, log, …|


Virtual naming sections:

The sections below are virtual -- they are constructed based on the sections defined above for the purpose of grouping and automation.

|Naming section | Description | Examples |
|--|--|--|
| Landing Zone | Constructed as 'System-Environment'. <br/> <br/>Usually, System + Environment forms a unit of separation of duties, security, and  concerns, which corresponds to a model of Landing Zone, and a good candidate for a dedicated subscription. | Security-PRD, SharedSvcs-QA|
| Instance | **Optional** instance count for a specific resource, to differentiate it from other resources that have the same naming convention. Examples, 01, 001.<br/><br/>In the current model it is suggested to put the instance number as optional part of the Component name section, see example. <br/><br/>Many organisations used to use _padding_ for the instance count. It is recommended to be wise and use padding only when necessary, for example, if in your organisation you have only 6 domain controllers, no need to pad their instance counts in 4 digits, like DC0001, DC0002; padding with 2 digits like DC01, DC02 is enough for this purpose. | `NVAPEUWFW01`<br/>`NVAPEUWFW02`<br/>  (instance count is padded as XX for 2 firewall instances, it supports upgrade and migration scenarios when you need room for more instances in the name) <br/><br/>`PlatConn-PRD-EUW-Hub-vnet-afw` <br/>`PlatCon-PRD-EUW-Hub-vnet-bas` <br/> a vnet can have only one firewall and bastion resource attached, so it does not make sense to include any instance count in the name of the resource || 

<details>
<summary>Note from [CAF](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming)</summary>

Padding improves readability and sorting of assets when those assets are managed in a configuration management database (CMDB), IT Asset Management tool, or traditional accounting tools. When the deployed asset is managed centrally as part of a larger inventory or portfolio of IT assets, the padding approach aligns with interfaces those systems use to manage inventory naming.

Unfortunately, the traditional asset padding approach can prove problematic in infrastructure-as-code approaches that might iterate through assets based on a non-padded number. This approach is common during deployment or automated configuration management tasks. Those scripts would have to routinely strip the padding and convert the padded number to a real number, which slows script development and run time.

Choose an approach that's suitable for your organization. Before choosing a numbering scheme, with or without padding, evaluate what will affect long-term operations more: CMDB and asset management solutions or code-based inventory management. Then, consistently follow the padding option that best fits your operational needs.

</details>

## Resource type abbreviations
The table below illustrates abbreviations for mostly used resources. When expanding existing naming convention for new resource types, please refer to [CAF: Abbreviation examples for Azure resources](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations).

| Resource type | Abbreviation |
|--|--|
|Resource Group | rg |
|Virtual Network | vnet|
|Network Security Group| nsg |
|Network Interface | nic|
| .. | .. |

You will find abbreviations for already-defined naming conventions in the section below.

## Region abbreviations
Contrary to CAF that provides examples for region abbreviation as: westus, eastus2, westeu, usva, ustx; we suggest to follow the logic supporting natural grouping by regions when sorting the objects, so West and North Europe should be abbreviated as EUW and EUW, East and West US  should be abbreviated USW and USE respecively.

| Region | Abbreviation |
|--|--|
|West Europe | EUW |
|North Europe | EUN |
|East US| USE |
|East US 2 | USE2 |
| .. | .. |

## Naming Restrictions
Please be aware that most of Azure resources have restrictions in terms of length, characters and may require to be globally unique across all Azure customers. 

Please refer to [Naming rules and restrictions for Azure resources](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-name-rules) for details and updates.

# Naming convention table
The table below describes common resource naming convention and can be updated when new resource types are introduced

| Resource Type |  Scope | Format + Example + Comment |
|--|--|--|
|Governance|-|-|
|Management Group | Tenant | <GroupName>-mg <br/><br/> `Platform-mg` <br/><br/> <small>Display Name for MG can be different and is easy to change. </small> |
| Policy Definition  | Management Group or Subscription | <PolicyName> <br/><br/> `Custom-Required-Deploy` <br/><br/> <small>Display name for Policy is easy to change. Actual Policy definition names can use a GUID pattern to ensure uniqueness</small> |
| Subscription | Account/Enterprise Agreement | <System>-[ENV][-##]-sub <br/><br/> `System1-Prod-sub`<br>`System2-Nonprod-sub`<br/>`Platform-Connectivity-sub` <br/><br/> <small>Name for Sub is easy to change, but still it follows the 'taxons' hierarchy principle. For non LZ related subscriptions, such as Platform, which can be common among different environments, 'environment' piece is skipped in this example </small>|
| Resource Group | Subscription | <System>-<ENV>-<REGION>-[Component]-rg <br/><br/> `PlatCon-PRD-EUW-Network-rg`<br> `PlatIdty-PRD-EUW-ADDC-rg`<br>`SharedSvcs-PRD-EUW-Network-rg`|
|Networking |-|-|-|
| Virtual Network | Resource group | <System>-<ENV>-<REGION>-[Component]-vnet <br/><br/> `PlatCon-PRD-EUW-Ext-vnet`<br>`PlatCon-PRD-EUW-Hub-vnet`<br>`SharedSvcs-PRD-EUW-vnet`|
| Azure Firewall |Resource group |<vnet_name>-afw <br/><br/> `PlatConn-PRD-EUW-Hub-vnet-afw`|
|Azure Bastion | Resource group| <vnet_name>-bas <br/><br/> `PlatCon-PRD-EUW-Hub-vnet-bas`|
| Virtual network gateway - VPN| Virtual network | <vnet_name>-vngv <br/><br/> `PlatCon-PRD-EUW-Hub-vnet-vngv` |
| Virtual network gateway - ER | Virtual network | <vnet_name>-vnge <br/><br/> `PlatCon-PRD-EUW-Hub-vnet-vnge` |
|ER connection | Resource group | <vng_name>-[Component]-con <br/><br/> `PlatCon-PRD-EUW-Hub-vnet-vnge-C_EquinixAMS-con` |
|Site to site connection | Resource group |<vng_name>[-Component]-con <br/><br/> `PlatCon-PRD-EUW-Hub-vnet-vngv-S_TSystemsFRA-con`|
|Local network gateway (VPN)| Resource group| <vng_name>[-Component]-lng <br/><br/> `PlatCon-PRD-EUW-Hub-vnet-vngv-L_TSystemsFRA-lng`|
| Subnet | Virtual network| <App>[-AppComponent] <br/><br/> `NVA_Hub-Mgmt`<br>`ADDC`<br>`IA-DevDefault`|
| Route Table - vnet level | Resource Group | <vnet_name>[-Component]-rt <br/><br/> `PlatCon-PRD-EUW-Hub-vnet-R_Default-rt`<br/>`PlatCon-PRD-EUW-Hub-vnet-R_GatewaySubnet-rt` |
| Route Table - app level | Resource Group | <System>-<ENV>-<REGION>[-Component]-rt <br/><br/> `SharedSvcs-PRD-EUW-App1_DB-rt`<br/>`SharedSvcs-PRD-EUW-App1_FE-rt` |
| Route - for vnet peering | Route Table | <PeeredVnetName>[_#] <br/><br/> `PlatIdty-PRD-EUW-vnet`<br/>`SharedSvcs-PRD-EUW-vnet_0`<br/>`SharedSvcs-PRD-EUW-vnet_1`||
| Route - custom | Route Table | [RouteName] <br/><br/> `LAN_10`<br/>`LAN_192`<br/>`Default` |
| NSG - vnet level| Resource group| <vnet_name>-nsg <br/><br/> `PlatCon-PRD-EUW-Hub-vnet-nsg` |
| NSG - app level | Resource group| <System>-<ENV>-<REGION>[-Component]-nsg <br/><br/> `PlatIdty-PRD-EUW-ADDC-nsg` |
| Load Balancer - external| Resource group|<System>-<ENV>-<REGION>[-Component]-lbe <br/><br/> `PlatCon-PRD-EUW-NVA_Ext-lbe`|
| Load Balancer - internal| Resource group|<System>-<ENV>-<REGION>[-Component]-lbi <br/><br/> `PlatCon-PRD-EUW-NVA_Ext-lbi`|
|Load Balancer FrontEnd| LoadBalancer| <Name>-fe <br/><br/> `FrontEnd1-fe`|
|Load Balancer HealthProbe| LoadBalancer|<Name>-probe <br/><br/> `SSH-probe` |
|Load Balancer RuleName|LoadBalancer|<Name>-rule <br/><br/> `HA-rule`|
|Load Balancer BackEndPool| LoadBalancer|<Name>-be <br/><br/> `BackEnd1-be` |
|Public IP| Resource group|<parent_name>-[PipName]-pip <br/><br/> `ADPEUWDC01-nic-pip` |
|DNS Label|Global|<FreeText> <br/><br/> `NVAPEUWFW01-epm`<br/><br/> <small>Keep unique but short, include region and epm indicator to distinguish from other customers</small> |
| Virtual Machines |-|-|-|
|Virtual Machine|Resource group|<SYSTEM><ENV><REGION><ROLE><##[#]><br/><br/>SYSTEM - 3-4 characters<br>ENV - on of S,D,Q,P<br>ROLE - role of server in system, 2 characters<br>## - 2-3 digits <br/><br/> `ADPEUWDC01`<br>`IAUDEUWBT001`|
|NIC |Resource group|<vm_name>[-NicName]-<nic> <br/><br/> `ADPEUWDC01-nic`<br/>`NVAPEUWFW01-N_Int-nic`<br/>`NVAPEUWFW01_N_Ext-nic`|
|Managed Disk|Resource group|<vm_name>[-DiskName]-<md> <br/><br/> `ADPEUWDC01-D_OS-md`<br>`IAUDEUWBT001-D_LUN1-md`|
|Availability Set |Resource group |<System>-<ENV>-<REGION>[-Component]-as <br/><br/> `PlatIdty-PRD-EUW-ADDC-as`<br/>`IA-PRD-EUW-Bots-as`|
|Storage | - |-|-|
|Azure Storage account - general use|Global|<short_system_alias><short_env_alias><region><component>epmsa<br>up to 24 characters and digits only <br/><br/> `shsvcsprewfilesepmsa`|
|Azure Storage account - diagnostic logs|Global|short_system_alias><short_env_alias><region>diagepmsa<br>up to 24 characters and digits only <br/><br/>`shsvcsprewdiagepmsa`|
|Recovery Service Vault|Resource group|<System>-<ENV>-<REGION>[-Component]-rsv<br>alphanumerics and hyphens only <br/><br/> `PlatIdty-PRD-EUW-rsv`<br/>`IA-NPR-EUW-BackupLRS-rsv`<br/>`IA-PRD-EUW-BackupGRS-rsv`|
|Azure KeyVault|-|-|-|
|KeyVault|Global|<System>-<ENV>-<REGION>[-Component]-epmkv<br>up to 24 characters, alphanumerics and hyphens only <br/><br/> `PlatIdty-PRD-EUW-epmkv`<br/>`IA-PRD-EUW-Secr-epmkv`<br/>`IA-PRD-EUW-Enc-epmkv`<br/>`Proj1-PRD-EUW-01-epmkv`|<small>In specific cases naming parts can be truncated to accommodate KV naming restrictions</small>|
|KeyVault Key|Key Vault|<Name>-key <br/><br/> `DefaultDiskEncryption-key`|
|KeyVault Secret|Key vault|<Name>-secret <br/><br/> `DomainJoinPass-secret`||
|Common and Management Objects|-|-|-|
|Log Analytics workspace |Resource group|<System>-<ENV>-<REGION>[-Component]-la <br/><br/> `PlatMgmt-PRD-EUW-Central-la`|
|Automation Account |Resource group|<System>-<ENV>-<REGION>[-Component]-aa <br/><br/> `PlatMgmt-PRD-EUW-Mgmt-aa`||
|Private Endpoint |Resource group|<parent_name>[-service]-pe <br/><br/> `shsvcsprewfilesepmsa-file-pe`|
|PaaS services |-|-|-|
|Azure Cache for Redis|Global|<short_system_alias><short_env_alias><short_region>[component]epmredis <br/><br/> `iadveworchepmredis`|
|Azure SQL Database|Global|<short_system_alias><short_env_alias><short_region>[component]epmsqldb <br/><br/> `iadvewaspcepmsqldb`|

# Example vs Good naming conventions
## Example convention, followed literally
<IMG  src="https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/_images/ready/resource-naming.png"  alt="Components of an Azure resource name"/>

- Object type as prefix (as in enterprise scale example)
- Objects are always ordered by object type
- It is not easy to see relationships between objects, and order/sort/group objects related to each other

![naming1.png](/.attachments/naming1-12745c6e-2731-488d-b49e-cbba73b23310.png)
![naming2.png](/.attachments/naming2-3c55bd5f-0abe-4cbf-bb68-b42f5aa19ad9.png)

## Proposed naming convention
- Names constructed to reflect relations between objects, object type as suffix. 
- You can easily search objects and see related objects. You can still sort object by type if needed.
- Similar approach is used when Azure auto-generates the names

![naming3.png](/.attachments/naming3-bfd79c05-7971-48eb-bb71-ae4e6be52fb6.png)
![naming4.png](/.attachments/naming4-0ff0b738-b420-468b-bed8-05201576093a.png)
