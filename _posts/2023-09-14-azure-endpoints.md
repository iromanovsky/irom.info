---
#layout: post-argon
title: Choosing between Public, Service, and Private Endpoints in Azure
date:   2023-09-14 08:00:00 +0100
author: Igor
categories: Blog
tags: [azure, devops, caf]
#permalink: /post1/
slug: azure-endpoints
excerpt_separator: <!--more-->
redirect_from: [/post/azure-endpoints]
#published: false
---
<div style="text-align: center">

![private-endpoint](https://github.com/iromanovsky/irom.info/assets/15823576/972101cf-3dc8-4ba5-9a68-17857926f631)

</div>

When deciding between Public, Service, or Private endpoints in Azure, it's essential to make an informed choice rather than blindly following generic recommendations.

Tools like Azure Secure Score, various benchmarks, and CAF documentation may seem convincing, but uncritical adherence can result in less resilient solutions and increased maintenance costs.

In this post, I will explain the differences between Public, Service, and Private endpoints and share the pros and cons of their use in common scenarios.

<!--more-->

<div style="padding-left: 50%">

> &mdash; We have a hole in security!<br/>
> &mdash; Thank God, at least something is in security...
> <p>Internet meme</p>

</div>

# Endpoints comparison
    
To get started, let's compare how each type of endpoint works.

| | Public Endpoints  | Service Endpoints | Private Endpoints |
| - | - | - | - |
| **Availability** | On all PaaS services | On [limited number of PaaS services](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview) | On [on most of PaaS Services](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-overview#private-link-resource) |
| **How routing works** | <p>Public Endpoints are available on public Internet IP addresses.</p><p>**From public Internet** anyone can reach them.</p><p>**From private VNet**, traffic is routed according to your routing tables (via Azure NAT or a Firewall).</p><p>On certain resources, such as storage, there is an option to prefer Microsoft routing, where the actual traffic remains within the Microsoft backbone.<p> | Direct routes for public IP ranges are advertised to your subnet and are visible in Effective Routes. These routes are specific [resource types and regions](https://www.microsoft.com/download/details.aspx?id=56519)</p><p>Client traffic flows directly to the endpoint, bypassing the routing table attached to the subnet, but it remains within the Microsoft backbone. It appears as if NAT is not used, as the resources on service endpoints can see the client's original private IP from the VNet.<p> | <p>An IP interface is created in the VNet where you deploy your private endpoint.</p><p>All traffic to endpoints remains within the private VNet.</p> |
| **Network security measures** | <p>**On destination:** Resource ACL</p><p>**On client:** NSG, firewalls</p> | <p>**On destination:** Resource ACL which can use public and private IPs, and specific VNet subnets</p><p>**On client:** [Service Endpoints policies](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoint-policies-overview) (only available for selected services), NSG</p> | <p>**On destination:** NSG (when [Private Endpoint Network Policies](https://learn.microsoft.com/en-us/azure/private-link/disable-private-endpoint-network-policy?tabs=network-policy-portal) are enabled)</p><p>**On client:** NSG, firewalls</p> |
| **Management burden** | None | <p>Need to disable Public Access on Resource ACL</p><p>Need to enable Service Endpoints on each subnet where they are used</p> | <p>Need to disable Public Access on Resource ACL</p><p>Need to deploy PE to VNets, for each service (blob, file) on each resource (storage, key vault)</p><p>Consuming private IP addresses, especially if you deploy PEs in dedicated subnets (+5 reserved IP addresses wasted)</p><p>Requires changes in [name resolution](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns) (from changing HOSTS file on each client up to implementing sophisticated DNS integration)</p><p>High complexity for name resolution when need to deploy multiple PEs of the same resource</p> |

# General recommendations

Below are scenarios where using each type of endpoint is recommended.

## Public Endpoints

Use Public endpoints for:

- Public resources, like web hosting _[app service, storage]_
- Resources for cloud-born applications, like SaaS, PaaS, and apps that are not integrated with your private network _[web apps, functions, storage]_
- Shared internal resources, which should be available across networks, clouds, and various client devices _[storage, monitoring, backup, site recovery]_
- Resources used from other clouds

## Service Endpoints

Use service endpoints for:

- Services requiring the highest possible bandwidth and least possible latency _[storage, databases]_
- Shared services consumed from multiple VNets and regions _[central file share, Cloud Witness for clusters]_
- Services requiring less administrative effort than managing private endpoints

## Private Endpoints

Use private endpoints for:

- **Resources that need to be accessed from on-premises over a private network:** Requires establishing hybrid connectivity and DNS integration.
- **Resources that are only required to be accessed from one VNet and not exposed outside of their system** (see [private data store pattern](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/data-considerations), like for internal databases and storage).
- **Resources for environments requiring the highest protection from data exfiltration:** This class of systems should be treated separately from the general class. To effectively protect from data exfiltration, all inbound and outbound traffic should be restricted to _tenant-specific_ (owned by you) destinations only. _This may come with the cost of losing a number of native platform and vendor capabilities, like Monitoring, Backup, Site Recovery, Patching automation, and others. Most of these services are multi-tenant and do not work without opening [wide IP ranges and wildcard URLs](https://learn.microsoft.com/en-us/azure/azure-monitor/app/ip-addresses) to destinations shared between multiple tenants._
- **Complying with specific regulations:** Private Endpoints indeed may enable some scenarios, regulatory controls or industry certifications that were not easily achievable in the cloud before PE introduction. _Important to note that many regulatory standards do not take cloud infrastructure into account, but thanks to the ambiguity of their language, may allow pretty wide interpretations. Always refer to the original standard wording to find out if it really mandates you to use private endpoints (long before they were ever created)._

The benefits of Private Endpoints are **doubtful** in the below scenarios:

- **Blindly Following Generic Recommendations:** Implementing Private Endpoints based on generic recommendations, "best practices," benchmarks, or secure scores without considering your specific usage scenario can lead to increased system complexity, fragility, and higher maintenance costs. Enforcing Private Endpoints without a valid reason might result in losing cloud capabilities without significant benefits.
- **Relying Solely on Network Security:** In the [zero-trust security model](https://learn.microsoft.com/en-us/azure/security/fundamentals/zero-trust), it's assumed that breaches can occur at any point in the data flow. Trusting the network to provide security along the data path is not recommended. While selective traffic routing, private networks, and firewalls offer some protection, they may not be sufficient in hybrid environments spanning multiple data centers, clouds, and VPN devices. Instead, comprehensive security measures like end-to-end encryption, authentication, and endpoint protection should be employed.
- **Providing Access Over VPN to Corporate Resources:** When granting access to corporate resources over a VPN, private endpoints may not be the most efficient choice. VPNs can introduce bottlenecks when accessing private endpoints. Leveraging public endpoints can utilize the Microsoft backbone for a more efficient data path.
- **Access Across Multiple Azure Regions in Hub-and-Spoke Models:** Private endpoints for the same resource in multiple regions or VNets can introduce significant complexity, especially concerning name resolution. This complexity can outweigh the benefits in scenarios where resources need to be accessed across various Azure regions or during failover situations.
- **Cloud-Native Application Design:** In cloud-native application design, where the goal is to leverage PaaS services extensively, implementing private endpoints for "grounding" interactions between PaaS components can add unnecessary complexity and costs. VNet integration options for PaaS services often come at premium price points.

# Recommendations for Endpoint Types by Azure Service
Considering that each company may have a unique environment and specific requirements, it is valuable to document the guidance regarding the use of endpoint types tailored to particular scenarios.

| Service | Recommendation |
|--|--|
| Storage Account | <p>**Public Endpoint** for publicly accessible storage, such as diagnostic data, Terraform state files, or exchanging information with external parties.<p/><p>**Service Endpoint** for storage shared across VNets and/or regions</p><p>**Private Endpoint**: for private data for apps in high-security zone; for file shares when migrating data from on-premise |
| Azure KeyVault | <p>_Note: The security of Key Vault is not significantly improved by using Private Endpoints since Key Vault access is protected with Azure AD authentication. Key Vault is not a practical target for data exfiltration, and Key Vault clients still require outbound access to AAD._</p><p>**Public + Service Endpoint** for shared Key Vaults used by Automation / DevOps / Terraform / PaaS / Admins </p> <p>**Private Endpoint** for app-specific KVs deployed on-demand for storing private data like encryption keys</p> |
| Azure SQL Database | <p>**Public Endpoints Endpoints** for cloud-native PaaS apps not integrated into VNet</p> <p>**Private Endpoints** for private database when used from IaaS servers</p>
| ..and so on..|  |

# Discussion

You are very welcome to discuss this topic in [Comments](https://www.linkedin.com/feed/update/urn:li:share:7107987444888686593/).
