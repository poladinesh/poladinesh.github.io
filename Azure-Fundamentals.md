## Azure Fundamentals

Cloud Concepts: 
High Availability: ability of a system to respond to users; expressed as a percentage Scalability: ability to system to grow/scale as users increase 
Elasticity: ability of a system to grow and shrink based on application demand Agility: ability to change rapidly based on changes to market or environment
Fault Tolerance: ability of a system to handle faults like power, networking or hardware failures
Disaster Recovery: ability of a system to recover from failure within a period of time and how much data is lost
Economies of Scale: The more you buy, the cheaper it is to own and operate per unit
CapEx: Money that is invested in assets(like computers) that return invest over time (cannot deduct from taxes most of the time) OpEx: Money that is spent every day on operating expenses (can deduct these expenses from gross revenues to make net profits); OpEx is money that can be immediately deducted from income to reduce your taxable expenses Consumption-based model: pay per minute, pay per hour or pay per execution Example of Azure Functions - gets charged only when called (serverless), other services charge using time based model (hourly, minute,)

IaaS:  Infrastructure such as VMs, Networking, LoadBalancers, Firewalls PaaS: Upload code packages, configuration  and have them run, without access to the hardware
SaaS: Typical software that you would install on your servers, Example:- SQL Server Software; access to configuration only

Public Cloud: computing services offered over public internet, anyone can signup Private Cloud: private offering of the cloud and available to your company. Not available for general public Hybrid Cloud: combination of public and private clouds

Core Azure Services:
 Azure Architectural Components:
Regions: Geographical location, 60+ regions with over 140 countries as of May 2020
Availability Zones: Availability Zones are physically separate datacenters within an Azure region (Not every region has support for Availability Zones) Availability Sets: allows workloads to be spread over multiple hosts, racks but still remain at the same data center(refers to 2 or more VMs deployed across different fault domains to avoid single point of failure)
Resource Groups: A way of organizing resources ARM: Azure Resource Manager enables you to interface with azure in a consistent way (a common API)

Single VM > Availability Sets > Availability Zones > Region Pairs

Core Azure Products: Compute: VMs, VM Sets, App Service, Functions, Azure Container Service, Azure Kubernetes Services Networking: Virtual Network, Loadbalancer, VPN Gateway, Application Gateway, Content Delivery Network
Storage: Azure Storage - Blob, File, Table, Queue , Managed Disk, Backup and Recover Storage Databases: Cosmos DB, Azure SQL DB, Azure DB for PostgreSQL, Azure DB Migration Service, Azure Synapse Analytics (formerly SQL Data Warehouse)
Azure Marketplace: One stop shop for all services/resources from different vendors including Microsoft

Azure Solutions: IoT, Big Data, AI, Serverless IoT: IoT Hub, IoT Central BigData: Azure Synapse Analytics, HDInsight(Hadoop products), Data Lake Analytics,  Azure Databricks(BigData)
AI: Azure Machine Learning Service, Studio Serverless: Azure Functions, Logic Apps, App Grid, Event Grid 
Azure Management Tools: Azure CLI, Powershell(SDK available), Azure Portal, Azure Cloud Shell Azure Advisor: Analyses usage and suggests recommendations 

 Understand Security, Privacy, Compliance & Trust:
Secure Network Connectivity:  Azure Firewall/ApplicationGateway(WAF/L7),  Azure DDoS Protection (Basic & Standard): protection policies on virtualnetwork, logging/alerting/telemetry, resource cost scale protection -> on standard plan
Network Security Group(NSG): a series of rule that can allow or deny traffic inbound/outbound, resides at subnet level Application Security Group(ASG):ASGs can group resource by type User Defined Route(UDR):  force traffic over a route(example: force traffic through a firewall or over a corporate network) traffic can be inbound/outbound/both
 Use NSG all the time DDoS as needed or after attacked Application Gateway with WAF Security through layers 
Identity Services:
Authentication: proof of who you are  Authorization: what level of access do you have 
Azure Active Directory: Microsofts IDaaS, preferred solution for identity management, supports SSO, can sync with your corporate AD, not LDAP supported, is different from AD
MFA: multi-factor authentication sms, authenticator app, phone call 
Secure Tools & Features: Azure Security: Physical Security vs Digital Security, Azure AD Layered Approach: Security Layers Data - virtual network endpoint Application - API Management, WAF Compute - Limit RDP Access, Windows Update & Patch Network - NSG, use of subnets, deny by default Perimeter - DDoS, firewalls Identity & Access - Azure AD
Physical - Door Locks and key cards
Azure Security Center: unified security management and advanced threat protection (free tier & standard tier available) Azure Key Vault: central, secure repository for secrets, certs and keys Azure Information Protection: Apply labels to emails and documents, used to protect docs from being viewed, print and forward/shared  Advanced Threat Protection: monitor and profile user behavior and activities, protects user identities and reduces the attack surfaces, identifies suspicious activities and advances attacks, investigate alerts and user activities  Azure Governance:  Azure Policy: implements standards for your org across Azure, create rules across all of your azure resources; evaluate compliance to those rules (built in policies available)
	Examples: require sql server 12.0, allowed storage account sku, allowed locations, allowed VM sku, apply tag and its default value, not allowed resources type
can create policies using JSON definition Policy Initiatives: group of policies - example: every rouse and resource group must have the five tags 
RBAC: create roles that represent the common tasks of the job; assign granular permission to the roles & roles to the people (built in roles available-Reader/Contributor/Owner) Locks: read only lock(only read the resource, no modifications can be done) and Delete lock (only modify resource, no one can delete the the resource, lock needs to be removed before deleted);use RBAC to restrict access to locks Azure Advisor: makes recommendations on high availability security, performance, cost Azure Blueprints: Azure Subscriptions templates with roles and policies already defined 

Azure Monitoring and Reporting: 
Azure monitor: gathers events(logs and metrics) from all sources within azure Azure service health: service issues with azure (part of azure monitor); general alerts across all of azure  Privacy Compliance and Data Protection standards:
Compliance: GDPR(General Data Protection Regulation for EU Citizens), ISO(Intonational Organization for Standardization), NIST CSF (National Institute of Standards and Technology Cyber Security Framework) - Microsoft Trust Center talks about NIST CSF
Microsoft Privacy Statement: Privacy.microsoft.com
Trust Center: A Portal that talks about Microsofts compliance to certain standards, security, GDPR, ISO, NIST etc  Service Trust Portal: same as trust center but describes things at a very low level being very specific like seeing documents around pen test reports, compliance manager, white papers, blueprints etc Compliance Manager: workflow based risk assessment tool to help manage regulatory compliance  Azure Government Services & Department of Defence(DoD): Isolated datacenter separate from azure public cloud; need to meet specific standards for US Government Azure Germany Services; Azure China Services

 Azure Pricing and Support: 
Subscriptions: Subscription is a billing unit; can be used to organize resources into completely distinct accounts Management Groups: Way of creating hierarchy of subscriptions to organize them into a group

Management groups —> Subscriptions —> Resource Groups --> Resources
 Planning and Management of Costs:

Purchase from Microsoft: Pay as you go: monthly billing on consumption based model Enterprise Agreement: negotiated minimum spend annual custom prices  Purchase from Microsoft Solution Provider such as HP or other third party  Azure Free Account: $200 credit for the first 30 days, 12 months of free services Free Services: ResourceGroups, VirtualNetworks(unto 50), LoadBalancer(basic), Azure Active Directory(basic), NetworkSecuritygroups, Free-tier web apps(upto 10)
 Pay per usage: Functions, Logic Apps, Storage(pay per GB), Outbound bandwidth, Cognitive Services API Pay for time(per second):   Stability in pricing: pay a fixed price per month, reserved instances (discounts for 1-yr or 3-yr commitment), multi-tenant or isolated environment
Hybrid licensing on reserved instances can lead upto 70% savings Predictable -> fixed pricing
 pay for bandwidth: (outbound bandwidth)first 5 GB is free, inbound is free  Pricing Calculator: not 100% accurate, export & share the estimate

TCO: Its the total cost of owning and operating a machine including up-front hardware costs, labor, electricity, internet access, real estate, security, cooling etc

Best practices for minimizing azure costs: Azure advisor - cost tab Auto shutdown on dev/qa resources Utilize cool/archive storage where possible Use Reserved instances, also use hybrid licensing configure alerts when billing exceeds an expected level Azure Policy to restrict access to certain expensive resources (restrict spinning up VMs more than 16 CPUs) AutoScaling resources (grow and shrink as needed) Downsize when resources over-provisioned
Enforce a Policy so that every resource must have billing tags or owner tags for easy identification of resources
 Azure cost management: A free tool inside azure to analyze spending

Support Options: 
Basic: Self-help support, Docs, Azure Advisor, Service Health dashboard & Health API Developer: business hours email access, unlimited tickers, Sev-C, 1 day response time(< 8 hours), general architectural guidance, $29/month Standard: 24X7 phone - email, unlimited tickets, Sev-C(8hr), Sev-B(4hr), Sev-A(1 hr), general architectural guidance, $100/month Professional: standard + onboarding and consultation, architectural guidance on best practice,  delivery manager, $1000/month Premier: professional + everything including Microsoft products, technical account manager, on demand training, contact us

Knowledge center available for everyone

SLAs: a financial agreement for what Microsoft advertise  
Downtime:
99.99% - 4 mins per month
99.95% - 21 mins per month
99.9%    - 44 mins per month 99%.      - 7.2 hours per month 

Example: VM VM’s with 2 or more instances deployed across 2 or more AZ - SLA -> 99.99% VM’s with 2 or more instances deployed in same availability set - SLA> 99.95 Single VM with premium storage - SLA -> 99.9%
 Composite SLA: SLA - service 1 X SLA service  2 (no redundancy) If there is redundancy, calculate the SLA for that using 1.0 (failure rate 1X failure rate 2) For example: storage has 9.99% = 1.0 (.001% X 0.001%) = 99.9999%

Service Lifecycle: Preview: they are for testing and not for production use
Private preview: requires registration Public preview: available to anyone

General Availability: available for anyone and can start using it for production purposes  Service Credits:
Monthly Uptime Percentage	Service Credit Percentage
< 99.9	10%
< 99	25%
< 95	100%


Other Important Answers:

2 VMs per month are Free (one Windows and 1 Linux)
On premise Ad to Azure AD sync happens using AD Connect software
MFA for Admins - Privileged Identity Management
Specify spending limits - to avoid incurring costs above certain level
After 200$ credit, all services are stopped until a paid account is created

correlate events from multiple resources into a central repository - Azure Log Analytics
All inbound data transfer is free of charge - express route & other  depends on the plan You can download and install the Azure CLI on Linux based operating systems. You can also install Powershell on Linux based operating systems You can club subscriptions into Management groups. This allows you to manage the compliance of resources across multiple subscriptions
you would be guaranteed an SLA of 99.9% for Azure paid services
You also get charged for the amount of read and write operations performed and for any data transfer activities
Azure powershell is compatible azure cli  You can download and install the Azure CLI on macOS operating systems. You can also install Powershell on macOS based operating systems
With the Azure Web App service, you can change the pricing tier to get more memory for the underlying applications
If you are using Azure Web Apps, which is the platform as a service in Azure, you can use the Standard and Premium App Service Plan to scale up the number of instances based on demand
No, once you create the resource, even after the service becomes generally available, your resource will still be available
For your subscription, you can go ahead on to the Budgets section and add a new budget alert -> for exceeding cost of current billing period
Yes, Azure Security Center can also be used to monitor your on-premises resources
You can't join Android devices; you can just register them to Azure AD
Yes, you can setup agents on your on-premises machines to send monitoring data to Azure Monitor
When you setup an alert, you can configure the alert to be sent onto users in an Azure AD Group.


Azure Cosmos DB is a completely serverless service. Here you don’t have to manage any aspect with regards to the service.

Yes, you can have multiple delete locks for a resource
Yes, this is possible. This can be done in Azure where you have multiple lock types.
Yes, Azure resources inherit locks from resource groups

No, at a time, an Azure subscription can only be linked to one Azure AD tenant.
Yes, you can change the Azure AD tenant linked to an Azure subscription. For your subscription, you can use the Change directory option to accomplish this. A tenant is also known as a directory.

Network Security Groups can only be associated with either a subnet or a virtual network interface
The Azure SQL data warehousing solution has high availability already built into the solution.
Yes, so you can use tools or in-built services on the virtual machine itself to encrypt the traffic. For example, if you are hosting Internet Information Services on a Windows machine, you can use the SSL feature on Internet Information Services to encrypt the traffic that flows out of the virtual machine.


 Azure Reserved VMs come under CAPex With Reserved Instances, you can actually save on costs by making a commitment to buying compute power upfront 
Azure Advisor does not have the facility to provide this sort of information. -> provide a list of machines protected by Azure backup
Yes, Azure Active Directory provides authentication services for resources hosted in both Azure and Microsoft Office 365.
Azure Active Directory is a managed service. You don’t need to host any domain controllers yourself to use Azure Active Directory.  To achieve high availability the service makes 3 copies of your data This depends on the type of replication method you chose for the storage account. Here we have to assume our answer based on the lowest redundancy option that is available for Azure storage accounts. And that is Locally-redundant storage. Here the data is only replicated three times within a particular physical location and NOT across data centers.

You have to rehydrate the data, convert it to either the Hot or Cool access tier and then access the object. 
The limit for the data is from 500 TB to 2 PB. And there is no limit on the number of files that can be stored in the storage account.

VPN: There is no cost associated with inbound data transfers.
Yes, there is a cost for outbound data transfers The Local network gateway resource is used to represent the VPN device in the on-premise infrastructure.

Yes, you can create a rule in the Azure Health service to see if there is a failure in any of the underlying Azure services.
Yes, you have the option of granting access to Identities stored across multiple identity stores.
It is not necessary that only Active Directory based users have access to resources hosted in Azure.

Microsoft guarantees at least 99.9% availability for both Active Directory Basic and Premium services.

In the event that Microsoft is at fault for not providing the required SLA for their services, they will provide Azure credits which can be redeemed.

Services in the public preview can be deployed to production-based environments. But this is not recommended , since there is no SLA for services in public preview


You can use Azure Activity Logs to see the various activities performed on Azure resources.

They would need to setup a subscription first. The billing for the resources would be tagged against the subscription.

