---
#layout: post-argon
title: Name Resolution for Azure Private Endpoints
date: 2023-09-20 08:00:00 +0100
author: Igor
categories: [Azure, Networking]
tags: [azure, devops, caf]
#permalink: /post1/
slug: azure-pe-name-resolution
excerpt_separator: <!--more-->
redirect_from: [/post/azure-pe-name-resolution]
image: https://github.com/iromanovsky/irom.info/assets/15823576/123e3713-1b0b-449f-a7fe-7e4ca0c09802
#published: false
---
<div style="text-align: center">

![mermaid-diagram-2023-09-20-100614](https://github.com/iromanovsky/irom.info/assets/15823576/f863c57f-5902-449f-9a3e-73426f16ea65)

</div>

In my [previous post](2023-09-14-azure-endpoints.md), I explored the use cases and constraints of Private Endpoints.

In this post, I will delve into the topic of name resolution for private endpoints and why it is critical. We'll explore the available approaches and my generally recommended solution.

Tech nerd content warning, open with caution!

<!--more-->

## Why you need to care about name resolution

When you deploy a private endpoint for your resource, you're essentially creating a private network interface with an IP address on your VNet dedicated to a specific service, like a storage blob or an SQL database. It's important to note that a single resource connected via a private link can provide multiple services, such as blob, file, table, and more, and each of these services should have its dedicated Private Endpoint.

For instance, your PE might have the IP address `10.18.0.4`, granting access to blobs via this address: `https://irom.blob.core.windows.net/`.

However, it's not advisable to directly use this IP address in your applications. Since most services are protected with SSL encryption, the client needs to validate the resource name. Microsoft still utilizes its wildcard certificate for this purpose, which is issued to `Common Name (CN) *.blob.core.windows.net` by `Microsoft RSA TLS CA 02`.

As a result, attempting to connect via `https://10.18.0.4/` will result in name validation failure. To avoid this, it's essential to use the original URL, ensuring that your queries go through the private endpoint."

```
nslookup irom.blob.core.windows.net
..
irom.blob.core.windows.net canonical name = blob.ams06prdstr05a.store.core.windows.net.
..
Name: blob.ams06prdstr05a.store.core.windows.net
Address: 52.239.141.196
```
If you block the public access on your resource, your query will fail, otherwise, it will be served by the public endpoint, which is probably not what you wanted in the beginning _[did you know that deploying a private endpoint does not block public access for some resources?]_.

Now it's clear that you need to find a way to substitute the public IP for the name `irom.blob.core.windows.net` with your private endpoint IP `10.18.0.4`

> Within this document, you'll encounter numbered headers denoting different approaches for name resolution with private endpoints. These approaches can be combined to create more intricate solutions, and we've assigned numbers to facilitate cross-referencing.

Lets agree on the terms from the beginning:

<dl>
<dt><strong>Domain (FQDN)</strong></dt>
<dd>Refers to a specific name within a DNS hierarchy. For example, "privatelink.blob.core.windows.net" is a domain within the broader hierarchy of the top-level domain "blob.core.windows.net."</dd>
<dt><strong>Namespace</strong></dt>
<dd>Used in forwarding, it encompasses everything within a particular domain, including its subdomains. In FQDN <em>irom.privatelink.blob.core.windows.net</em>, if <code>blob.core.windows.net</code> represents the namespace, then <code>irom.privatelink</code> is one of its subdomains.</dd>
<dt><strong>DNS Zone</strong></dt>
<dd>Employed in DNS servers to store data related to a namespace. For instance the zone <em>privatelink.blob.core.windows.net</em> contains records for private endpoints.</dd>
<dt><strong>Private Endpoint</strong></dt>
<dd>Denotes a specific network interface deployed in a VNet to connect with a resource via a private link.</dd>
<dt><strong>privatelink</strong></dt>
<dd>Refers to namespaces in the format "*.privatelink.[service]". For instance, <code>privatelink.blob.core.windows.net</code> represents privatelink namespace for blob service on storage account.</dd>
</dl>

## 0. Hosts file

This is a quick and dirty, but still valid solution. You simply insert a record into your client's hosts file::

```
10.18.0.4 irom.blob.core.windows.net
```

Pros:
- ~~Quick~~
- Simplicity. It doesn't rely on external name resolution mechanisms.

Cons:
- ~~Dirty~~
- Does not scale well. When you must access the same resources from multiple clients, individual adjustments are necessary for each one.
- Fragile. When numerous resources on Private Endpoints (PEs) need periodic updates, this method can lead to errors and inconsistencies.

Overall, this approach remains suitable for general development/testing environments, and for [private data access pattern](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/data-considerations) when used carefully.
    
## 1. Azure Private DNS Zones

When you need to serve the same private endpoints to multiple clients and VNets, Azure Private DNS Zones may help you:

1. For each resource (like storage) and service (like blob) you create a zone like `privatelink.blob.core.windows.net` (refer to the full list of zones [here](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns#azure-services-dns-zone-configuration)).
2. You [link](https://learn.microsoft.com/en-us/azure/dns/private-dns-virtual-network-links) this zone to all the VNets where you need to use your private endpoints. 
3. When creating private endpoint resources, register your PE names to the DNS zones: `irom A record to 10.18.0.4`. When manually creating private endpoints in the Azure portal, it can do this for you automatically, however, you'll need to manage it yourself when automating the process, or you can employ [policies for automation](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/private-link-and-dns-integration-at-scale).
4. When clients within the VNets resolve names for Private Endpoint-enabled resources, they initiate an additional DNS query for _resourcename_ in **privatelink**.service.domain namespace. 
   - If the Private DNS Zone successfully resolves this query, the client receives the private IP address associated with the endpoint.. 
   - If **privatelink** name like irom.privatelink.blob.core.windows.net is resolved externally, the client gets the Public Endpoint IP. This could be less desirable but may serve as a failover solution in specific situations.

> <p>Why the zones for private endpoints are named like privatelink.blob.core.windows.net?</p><p>Because the parent namespace like  blob.core.windows.net contains the names of all Azure storage accounts, including those of other customers. Creating a private DNS zone named blob.core.windows.net would mean that names not registered in this zone, including resources without private endpoints, would fail to resolve and establish connections. This could be a scenario for disaster, be careful to not disrupt the name resolution of existing resources on Public Endpoints.</p>

Pros:
- Centralized and automated private endpoint name resolution can be implemented at the scale of your landing zone.

Cons:
- Requires automation to maintain consistency and manage the entire lifecycle of names in the Private DNS zone.
- Not applicable if you use custom DNS servers for your VNet (unless you configure more advanced scenarios, as described below).
- Does not support scenarios where multiple private endpoints for the same resource name are deployed in different VNets or regions.

## 2. Custom DNS servers

Many enterprise customers already operate their DNS systems for name resolution within their networks, with VNets configured to use custom DNS servers. In such cases, it becomes necessary to integrate the resolution of private endpoint names into the organization's DNS solution.

### 2.1 Hosting private link zones on custom DNS servers

One approach is to host private link zones like `privatelink.blob.core.windows.net` on your organization's DNS servers. If your DNS is managed by Active Directory domain controllers, you can benefit from DNS zone replication for centralized updates and enhanced availability.

Pros:
- Integration of private endpoint name resolution into your custom DNS solution, extending support to on-premise clients.

Cons:
- Requires separate management or privatelink records lifecycle, losing benefits of Azure Private DNS Zones

### 2.2 Forwarding privatelink namespaces from custom DNS to Azure DNS

This scenario is the most common, where VNets and on-premises clients are configured to use custom DNS servers. However, these custom servers are set up to forward **privatelink** namespaces to Azure DNS.

Pros:
- Combines the advantages of approaches (1) and (2.1): privatelink name resolution is seamlessly integrated into your custom DNS system, while the management of records still remains on the Azure side.

Cons:
- Introduces increased complexity.
- Complexity can further escalate when your custom DNS servers are distributed across various regions, both on-premises and in Azure. This is a common setup, especially for Active Directory. You'll need to plan for regional redundancy to ensure robust name resolution from all locations, leveraging the nearest Azure region.

Challenges:
- For forwarding to function correctly, the DNS server must be able to reach Azure DNS:
   - For servers hosted in Azure, you need to forward queries to the [Azure Magic IP 168.63.129.16](https://learn.microsoft.com/en-us/azure/virtual-network/what-is-ip-address-168-63-129-16).
   - For on-premises servers, you can either use existing servers hosted in Azure as forwarders or deploy [Azure Private Resolver](https://learn.microsoft.com/en-us/azure/dns/private-resolver-architecture) within your VNet to facilitate this forwarding process.

> Note: When configuring conditional forwarding from on-premises, it is crucial to forward **privatelink**.service.domain namespaces instead of forwarding the whole  **service**.domain namespaces (contrary to [CAF article](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns#azure-services-dns-zone-configuration)). If forwarding fails, it would result in the loss of resolution for **privatelink** resources only, as opposed to losing access to all resources of the service, which might also employ public endpoints. This targeted forwarding approach helps maintain selective control.

## 3. Advanced scenarios

### 3.1 Azure Firewall

For Azure Firewall FQDN rules to function correctly, all clients behind the firewall [should use](https://learn.microsoft.com/en-us/azure/firewall/dns-details#clients-not-configured-to-use-the-firewall-dns-proxy) the firewall as their DNS server. As per Microsoft:

> If a client computer is configured to use a DNS server that isn't the firewall DNS proxy, the results can be unpredictable.

Azure Firewall, in its turn, should also be configured to resolve privatelink namespaces and use the company’s custom DNS solution (if it is present).

This means:
- If you only need to resolve privatelink namespaces from Azure Firewall, link Azure Private DNS zones, similar to approach (1), to AzureFirewallSubnet.
- If you require both privatelink namespaces and the company’s custom DNS solution, [configure the firewall](https://learn.microsoft.com/en-us/azure/firewall/dns-settings) to use custom DNS servers, as described in (approach 2.2).

### 3.2 Secure Hub with Azure Firewall on Azure VWAN

In this setup, linking Azure Private DNS zones to AzureFirewallSubnet isn't possible because this subnet doesn't exist. The sole approach to resolve privatelink namespaces is configuring Azure Firewall to use custom DNS servers, like in (approach 2.2).

You can find an in-depth discussion of this solution in the article [Guide to Private Link and DNS in Azure Virtual WAN](https://learn.microsoft.com/en-us/azure/architecture/guide/networking/private-link-virtual-wan-dns-guide).

### 3.3 Private Endpoints of the same resource in multiple VNets

There are scenarios where deploying multiple private endpoints for the same resource across various VNets or regions becomes necessary. For instance, it ensures better bandwidth and lower latency by placing dedicated private endpoints closer to clients across different locations. However, this leads to a situation where the same private endpoint resource name must resolve to different IP addresses depending on the client's location:

```
10.18.0.4 irom.blob.core.windows.net #if the client is in West Europe.
10.19.0.4 irom.blob.core.windows.net #if the client is in North Europe.
...
# ..and so on for other regions..
```

While I do not have a perfect solution for this, here are some workaround options:

#### 3.3.1 Private data store pattern

If your resource is accessed exclusively by a limited number of clients, such as in the [private data store pattern](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/data-considerations), the simplest solution might be to employ the traditional hosts file method, as mentioned in (approach 0).

#### 3.3.2 Multiple private DNS Zones for the same namespace

Private DNS Zones are global resources that can be linked to VNets in any region, but they cannot determine which IP to return to clients based on their location.

To work around this limitation, consider the following approach:

1. Create a pinpoint Private DNS Zone for _each VNet_ and each _private endpoint_ for the _same resource_, but place them in _different Resource Groups_.

> <p>The pinpoint DNS entry is a zone created for a single host only.</p> <p>For example pinpoint DNS zones are created for individual "hosts":<br/>myglobalstore.privatelink.blob.core.windows.net<br/>myglobalsql.privatelink.database.windows.net</p><p>And this is a general zone that may contain records for multiple "hosts": <br/>privatelink.blob.core.windows.net</p>

2. Link pinpoint zones to their respective VNets and/or regions.

| VNet | Region | DNS Zone | Resource Group | Target Resource |
| -- | - | -- | -- | -- |
| Hub-WE-vnet | West Europe | myglobalstore.privatelink.blob.core.windows.net | Hub-WE-Endpoints-rg | myglobalstore |
| Hub-NE-vnet | North Europe | myglobalstore.privatelink.blob.core.windows.net | Hub-NE-Endpoints-rg | myglobalstore |
| Hub-WE-vnet | West Europe | myglobalsql.privatelink.database.windows.net | Hub-WE-Endpoints-rg | myglobalsql |
| Hub-NE-vnet | North Europe | myglobalsql.privatelink.database.windows.net | Hub-NE-Endpoints-rg | myglobalsql |

This approach allows you to control DNS resolution at a finer level for resources accessed from different locations.

Pros:
- This approach enables the resolution of the same privatelink name to different IPs depending on the client VNet, optimizing traffic routing.

Cons:
- Managing the lifecycle of numerous pinpoint DNS zones, their VNet links and private endpoint resources can be challenging at scale, considering management, automation, and [limitations](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/azure-subscription-service-limits#azure-dns-limits).
- Implementing this approach in coordination with global, multi-regional forwarding of privatelink namespaces from on-premises DNS servers to Azure (as in approach 2.2) may pose challenges.

#### 3.3.3 Policy-based DNS name resolution

Policy-based DNS name resolution can be implemented using third-party DNS solutions like DNS Query Resolution Policies, available on Windows Server from 2016 onwards. With this method, you can create multiple copies of DNS zones with identical names, such as privatelink.blob.core.windows.net, and these copies can provide different replies based on the client's location. 

For more information, you can refer to the following resources:

- [DNS Policies Overview](https://learn.microsoft.com/en-us/windows-server/networking/dns/deploy/dns-policies-overview)
- [Use DNS Policy for Geo-Location Based Traffic Management with Primary Servers](https://learn.microsoft.com/en-us/windows-server/networking/dns/deploy/primary-geo-location)

Pros: 
- In addition to the benefits of 3.3.2, this approach offers more fine-grained control of responses through resolution policies. These policies can consider client locations, not only within Azure but also on-premises, thus facilitating efficient traffic routing.

Cons:
- Similar to 3.3.2, this method introduces complexity, but it transfers this complexity from Azure to your third-party solution.

## Conclusion

Based on my experience, I recommend the following approach:

1. Deploy Private DNS zones for each privatelink namespace used in your organization (approach 1).

2. Configure your AD Domain Controllers for conditional forwarding of privatelink namespaces to Azure DNS (approach 2.2).

3. If you use Azure Firewall, configure the Firewall DNS proxy to use AD DCs deployed in Azure as DNS servers (approach 3.1).

4. If you use Azure Firewall, you have no choice but to use it as a DNS server for your clients. If not using Azure Firewall, use AD DCs.

5. For scenarios where you need Private Endpoints of the same resource in multiple VNets, consider approach 3.3.3, which involves Policy-based DNS name resolution for _specific pinpoint DNS zones_.

In the future, I plan to create a pretty [Mermaid diagram](2023-09-04-diagrams-as-code.md) to illustrate these recommendations further.

## Links

- [Choosing between Public, Service, and Private Endpoints in Azure](2023-09-14-azure-endpoints.md)

## Discussion

Help me decide which picture fits better as the post cover, the one on the top or the one on the bottom below.

This post represents a structured compilation of information I've never had on a single page before. Your feedback is highly appreciated, and I'd also love to hear your thoughts on how to further enhance this content or explore better approaches. Please feel free to leave your [comments](https://www.linkedin.com/feed/update/urn:li:share:7110258252159819776/).

<div style="text-align: center">

![Bike Fall Meme (1)_627](https://github.com/iromanovsky/irom.info/assets/15823576/cc425d24-4a68-421d-9ae1-15594beb84da)

</div>
