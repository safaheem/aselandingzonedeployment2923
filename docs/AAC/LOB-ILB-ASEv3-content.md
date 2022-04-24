> The H1 title is the same as the title metadata. Don't enter it here, but as the **name** value in the corresponding YAML file.

_Brief introduction goes here._ [**Deploy this solution**.](#deploy-the-solution)

![alt text.](./media/folder_name/architecture-diagram.png)

_Download a [Visio file](https://arch-center.azureedge.net/architecture.vsdx) that contains this architecture diagram. This file must be uploaded to `https://arch-center.azureedge.net/`_

## Architecture

Include visio diagram

### Components

The solution uses the following Azure services:

- **[App Service Environment v3 (ASEv3)](https://docs.microsoft.com/en-us/azure/app-service/environment/overview)** is a single tenant  service for customers that require high scale, network isolation and security, and/or high memory utilization. Apps are hosted in [App Service plans](https://docs.microsoft.com/en-us/azure/app-service/overview-hosting-plans) created in ASEv3 with options of using different tiers within an Isolated v2 Service Plan. Compared to earlier version of ASE numerous improvements have been made including, but not limited to, network dependency, scale time, and the removal of the stamp fee. This reference architecture uses an internal App Service Environment v3. 
  
 - **[Azure Private DNS Zones](https://docs.microsoft.com/en-us/azure/dns/private-dns-privatednszone)** allow users to manage and resolve domain names within a virtual network without needing to implement a custom DNS solution. A Private Azure DNS zone can be aligned to one or more virtual networks through [virtual network links](https://docs.microsoft.com/en-us/azure/dns/private-dns-virtual-network-links). Due to the internal nature of the ASEv3 this reference architecture uses, a private DNS zone is required to resolve the domain names of applications hosted on the App Service Environment.

- **[Application Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)** is a feature of Azure Monitor that helps Developers detect anomalies, diagnose issues, and understand usage patterns with extensible application performance management and monitoring for live web apps. A variety of platforms including .NET, Node.js, Java, and Python are supported for apps that are hosted in Azure, on-prem, hybrid, or other public clouds. Application Insights is included as part of this reference architecture to monitor behaviors of the deployed application.

- **[Log Analytics](https://docs.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-overview)** is a feature of Azure Monitor that allows users to edit and run log queries with data in Azure Monitor Logs, optionally from within the Azure portal. Developers can run simple queries for a set of records or use Log Analytics to perform advanced analysis and visualize the results. Log Analytics is configured as part of this reference architecture to aggregate all the monitoring logs for additional analysis and reporting.

- **[Azure Virtual Machine](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/overview)** is an on-demand, scalable computing resource that can be used to host a number of different workloads. In this reference architecture, virtual machines are used to provide a management jumpbox server, as well as a host for the DevOps Agent / GitHub Runner. 

- **[Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/general/basic-concepts)** is a cloud service to securely store and access secrets ranging from API keys and passwords to certificates and cryptographic keys. While this reference architecture does not store secrets in the Key Vault as part of the infrastructure deployment of this reference architecture, the Key Vault is deployed to facilitate secret management for future code deployments. 

- **[Azure Bastion](https://docs.microsoft.com/en-us/azure/bastion/bastion-overview)** is a Platform-as-a-Service service provisioned within the developer's virtual network which provides secure RDP/SSH connectivity to the developer's virtual machines over TLS from the Azure portal. With Azure Bastion, virtual machines no longer require a public IP address to connect via RDP/SSH. This reference architecture uses Azure Bastion to access the DevOps Agent / GitHub Runner server or the management jumpbox server. 


## Recommendations

The following recommendations apply for most scenarios. Follow these recommendations unless you have a specific requirement that overrides them.

_Include considerations for deploying or configuring the elements of this architecture._

- Review the reference implementation resources at [LOB-ILB-ASEv3](../../reference-implementations/LOB-ILB-ASEv3/) to better understand the specifics of this implementation.
- It is recommended that you clone this repo and modify the reference implementation resources to suit your requirements and your organization's specific landing zone guidelines.
- Ensure that the service principal used to deploy the solution has the required permissions to create the resource types listed above.
- Consider the CI/CD service you will use for deploying the reference implementation. As this reference implementation is an internal ASE, a self-hosted agent is needed to execute the deployment pipelines.  As such the choice is to use either a DevOps Agent or a GitHub Runner. Refer to the [user guide](../README.md) on specific configuration values required for each.
- Consider the region(s) to which you intend deploying this reference implementation, and consult the [ASEv3 Regions list](https://docs.microsoft.com/en-us/azure/app-service/environment/overview#regions) to ensure the selected region(s) are enabled for deployment.

## Scalability considerations

_Identify and address scalability concerns relevant to the architecture in this scenario._

- Based on your scalability requirements, you may want to adjust the number of workers, and the size of the worker nodes, in the default App Service plan created by this reference implementation. These settings can be changed by cloning the repo and changing the `numberOfWorkers` and `workerPool` values in the relevant deployment files. For a Bicep deployment, for example, these values area available in the [ase.bicep](../../reference-implementations/LOB-ILB-ASEv3/bicep/ase.bicep) file.
- While this reference implementation does not implement a geo distributed scaling architecture, this capability is available as per [Geo Distributed Scale with App Service Environments](https://docs.microsoft.com/en-us/azure/app-service/environment/app-service-app-service-environment-geo-distributed-scale)
- For other scalability topics, see the [performance efficiency checklist](https://docs.microsoft.com/en-us/azure/architecture/framework/scalability/performance-efficiency) available in the Azure Architecture Center.

## Availability considerations

_Identify and address availability concerns relevant to the architecture in this scenario._

- Consider your requirements for zone redundancy in this reference implementation. ASEv3 supports zone redundancy by spreading instances to all three zones in the target region. This can only be set at the time of ASE creation, and may not be available in all regions. See [Availability zone support for App Service Environment](https://docs.microsoft.com/en-us/azure/app-service/environment/overview-zone-redundancy) for more detail. This reference implementation does implement  zone redundancy, but this can be changed by cloning this repo and setting the `zoneRedundant` property to `false`.
- For additional considerations concerning availability, see the [availability checklist](https://docs.microsoft.com/en-us/azure/architecture/framework/resiliency/reliability-patterns) in the Azure Architecture Center.

## Manageability considerations

_Identify and address manageability concerns relevant to the architecture in this scenario._

- Consider the [App Service deployment best practices](https://docs.microsoft.com/en-us/azure/app-service/deploy-best-practices) to ensure robust CI/CD processes that will help ease the manageability tasks associated with deployment of changes and new functionality to an App Service running on an ASE.

## Security considerations

_Identify and address security concerns relevant to the architecture in this scenario._

- Since this reference implementation deploys an ASE into a virtual network (referred to as an internal ASE), all applications deployed to the ASE are inherently network-isolated at the scope of the virtual network.
- Applications deployed to the same ASE therefore share the same network and can see each other. Principles of [Zero Trust security](https://docs.microsoft.com/en-us/azure/security/fundamentals/zero-trust) should be considered to ensure that each App Service deployed to the ASE enforces its own authentication and authorization requirements.
- When configuring networking rules for the ASE virtual network, consider that all App Services deployed to the ASE will be affected by the same rules. Further restrictions can be applied to the individual App Services, where necessary.
- For additional considerations concerning security of App Services, see [App Service security recommendations](https://docs.microsoft.com/en-us/azure/app-service/security-recommendations)

## Cost Considerations
[Reserved instance](https://docs.microsoft.com/en-us/azure/app-service/overview-hosting-plans)

## Deploy this scenario

A deployment for the reference architecture that implements these recommendations and considerations is available on [GitHub](https://github.com/Azure/appservice-landing-zone-accelerator/tree/main/reference-implementations/LOB-ILB-ASEv3).



## Next steps

* [Security in Azure App Service](/azure/app-service/overview-security)
* [Networking for App Service](/azure/app-service/networking-features)

## Related resources


* [High availability enterprise deployment using App Services Environment](docs/reference-architectures/enterprise-integration/ase-high-availability-deployment.yml)
* [Enterprise deployment using App Services Environment](docs/reference-architectures/enterprise-integration/ase-standard-deployment.yml)