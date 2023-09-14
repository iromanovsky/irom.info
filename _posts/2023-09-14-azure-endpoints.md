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
| **How routing works** | <p>Public Endpoints are available on public Internet IP addresses.</p><p>**From public Internet** anyone can reach them.</p><p>**From private VNet**, traffic is routed according to your routing tables (via Azure NAT or a Firewall).</p><p>On certain resources, such as storage, there is an option to prefer Microsoft routing, where the actual traffic remains within the Microsoft backbone.<p> | Direct routes for public IP ranges are advertised to your subnet and are visible in Effective Routes. These routes are specific to [resource types and regions](https://www.microsoft.com/download/details.aspx?id=56519).</p><p>Client traffic flows directly to the endpoint, bypassing the routing table attached to the subnet, but it remains within the Microsoft backbone. It appears as if NAT is not used, as the resources on service endpoints can see the client's original private IP from the VNet.<p> | <p>An IP interface is created in the VNet where you deploy your private endpoint.</p><p>All traffic to endpoints remains within the private VNet.</p> |
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
- **Resources for environments requiring the highest protection from data exfiltration:** This class of systems should be treated separately from the general class. To effectively protect from data exfiltration, all inbound and outbound traffic should be restricted to _tenant-specific_ (owned by you) destinations only. _This may come with the cost of losing a number of native cloud platform and 3rd party vendor capabilities, like Monitoring, Backup, Site Recovery, Patching automation, and others. Most of these services are multi-tenant and do not work without opening [wide IP ranges and wildcard URLs](https://learn.microsoft.com/en-us/azure/azure-monitor/app/ip-addresses) to destinations shared between multiple tenants._
- **Complying with specific regulations:** Private Endpoints indeed may enable some scenarios, regulatory controls, or industry certifications that were not easily achievable in the cloud before PE introduction. _Important to note that many regulatory standards do not take cloud infrastructure into account, but thanks to the ambiguity of their language, may allow pretty wide interpretations. Always refer to the original standard wording to find out if it really mandates you to use private endpoints (long before they were ever created)._

The benefits of Private Endpoints are **doubtful** in the below scenarios:

- **Blindly following generic recommendations:** Implementing Private Endpoints based on generic recommendations, "best practices," benchmarks, or secure scores without considering your specific usage scenario can lead to increased system complexity, fragility, and higher maintenance costs. Enforcing Private Endpoints without a valid reason might result in losing cloud capabilities without significant benefits.
- **Relying solely on network security:** In the [zero-trust security model](https://learn.microsoft.com/en-us/azure/security/fundamentals/zero-trust), it's assumed that breaches can occur at any point in the data flow. Trusting the network to provide security along the data path is not recommended. While selective traffic routing, private networks, and firewalls offer some protection, they may not be sufficient in hybrid environments spanning multiple data centers, clouds, and VPN devices. Instead, comprehensive security measures like end-to-end encryption, authentication, and endpoint protection should be employed.
- **Providing access to corporate resources:**  Private Endpoints may not be the most efficient choice for providing access to corporate resources over a VPN or a weak WAN link. VPNs and local WAN links can introduce bottlenecks when accessing the resources on private endpoints. Leveraging public endpoints can utilize the [Microsoft backbone](https://azure.microsoft.com/en-gb/explore/global-infrastructure/global-network/) (traffic managers, front door gateways, CDNs) for a [more efficient](https://learn.microsoft.com/en-us/microsoft-365/enterprise/microsoft-365-network-connectivity-principles?view=o365-worldwide#avoid-network-hairpins) data path.
- **Access across multiple Azure regions in a Hub-and-Spoke model:** Private endpoints for the same resource in multiple regions or VNets can introduce significant complexity, especially concerning name resolution. This complexity can outweigh the benefits in scenarios where resources need to be accessed across various Azure regions or during failover situations.
- **Cloud-native application design:** In cloud-native application design, where the goal is to leverage PaaS services extensively, implementing private endpoints for "grounding" interactions between PaaS components can add unnecessary complexity and costs. VNet integration options for PaaS services often come at premium price points.

### A note on data exfiltration

The most popular reason for enforcing private endpoints is "to prevent data exfiltration". While I agree that this could be a valid requirement, I need to show how challenging it is to implement it right.

While reducing exfiltration vectors may help prevent exfiltration attempts from unprofessional users, it requires much more effort to combat real professionals.

Relying solely on private endpoints for storage and key vaults and blocking outbound access to public endpoints may create a false sense of security.

There are other exfiltration vectors you need to consider as well:

- DNS: Very few systems can operate without using DNS. If you allow the resolution of public DNS domains, a knowledgeable attacker may gradually [export your data](https://www.infoblox.com/dns-security-resource-center/dns-security-issues-threats/dns-security-threats-data-exfiltration). You can use custom DNS servers to try to prevent this. Still, an attacker may use the platform's [magic IP 168.63.129.16](https://learn.microsoft.com/en-us/azure/virtual-network/what-is-ip-address-168-63-129-16) if it is not blocked.
- Platform services like [Azure AD](https://azureipranges.azurewebsites.net/), [AVD](https://learn.microsoft.com/en-us/azure/virtual-desktop/safe-url-list?tabs=azure), [Monitoring](https://learn.microsoft.com/en-us/azure/azure-monitor/app/ip-addresses) and [Backup](https://learn.microsoft.com/en-us/azure/backup/backup-support-matrix-mars-agent) are multi-tenant in nature and require opening wide ranges of destinations shared between customers.
- Productivity tools like [Microsoft 365](https://learn.microsoft.com/en-us/windows-365/enterprise/requirements-network?tabs=enterprise%2Cent#windows-365-service) and even using the [Azure Portal](https://learn.microsoft.com/en-us/azure/azure-portal/azure-portal-safelist-urls?tabs=public-cloud) require opening almost a half of the Internet.
- System updates, KMS activation, CRL endpoints, and time services can also be potentially exploited.

Mitigating all of these vectors becomes a highly complex task, limiting the capabilities of your workload in the direction of a disconnected server stored in a safe.

I hope this convinces you that effectively protecting cloud systems from data exfiltration by a professional adversary is challenging, and systems that genuinely require this level of protection should be treated separately from general workloads. Trying to implement the highest security level for all servers as a general rule may not be worth the effort and may result in missed opportunities.


# Recommendations for Endpoint Types by Azure Service

Considering that each company may have a unique environment and specific requirements, it is valuable to document the guidance regarding the use of endpoint types tailored to particular scenarios.

| Service | Recommendation |
|--|--|
| Storage Account | <p>**Public Endpoints** for publicly accessible storage, such as diagnostic data, Terraform state files, or exchanging information with external parties.<p/> <p>**Service Endpoints** for storage shared across VNets and/or regions.</p> <p>**Private Endpoint** for private data for apps in high-security zone; for file shares when migrating data from on-premise.</p> |
| Azure KeyVault | <p>_Note: The security of Key Vault is not significantly improved by using Private Endpoints since Key Vault access is protected with Azure AD authentication. Key Vault is not a practical target for data exfiltration, and Key Vault clients still require outbound access to AAD._</p> <p>**Public + Service Endpoints** for shared Key Vaults used by Automation / DevOps / Terraform / PaaS / Admins.</p> <p>**Private Endpoints** for app-specific KVs deployed on-demand for storing private data, like encryption keys.</p> |
| Azure SQL Database | <p>**Public Endpoints** for cloud-native PaaS apps not integrated into VNet.</p> <p>**Private Endpoints** for private databases when used from IaaS servers.</p> |
| ..and so on..|  |

# Discussion

You are very welcome to discuss this topic in [Comments](https://www.linkedin.com/feed/update/urn:li:share:7107987444888686593/).
