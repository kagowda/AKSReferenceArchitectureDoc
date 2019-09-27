
In collaboration with field, AKS product team is penning a AKS reference architecture. As first draft, we will write reference architecture doc for below scenario. 

**Customer Scenario & Requirements**: 

ContosoFinancial, a large Financial Services organization, provides banking and online services to its customers around the world. ContosoFinancial has 3 physical locations around the world (1 North America, 1 Western Europe, 1 Singapore) with each physical location connected to Azure via Express Route. They are also divided into 5 major line of business, each with their own cost center. 
ContosoFinancial is a mature Azure customer with the majority of their existing workloads spanning their on-prem datacenters and Cloud, mostly leveraging IaaS VMs today. They are looking to take the next step in their Azure journey and achieve a greater level of agility and flexibility through the adoption of Microservices and Containers. They also recognize that there is huge momentum in the industry towards containers and Kubernetes as the container orchestrator. They want to take advantage of that momentum while at the same time look to see if they can reduce some of the management and operations overhead that currently exists with their IaaS VMs today. This is the main reason that ContosoFinancial is looking at Azure Kubernetes Service (AKS) as it aligns to their container strategy vision and is also a managed service which will help them reduce some of their existing technical debt around management and operations.  
Here are the key requirements that ContosoFinancial is looking for: 

- *Reliability and Availability*
	- Global Presence – Their customer base exists around the world so they will need presence in multiple countries/locations. 
	- High Availability – The customer workloads need to be highly available so that their customers are able to access their services such as online banking at anytime. 

- *Monitoring & Logging* 
	- The customer is currently using Log Analytics for their IaaS VM workloads and would like to continue to use that going forward for AKS if possible. 
	- ContosoFinancial uses DataDog today on-prem and would like to continue to use that with AKS so they have a single pane of glass. If we are recommending an alternative, help them understand how that migration is going to work. 
	- ContosoFinancial would like to setup alerts so that key systems and components are monitored and reported upon when not functioning as expected.Key metrics across cluster and containers? 
	- Audit Reporting – In order to meet their regulatory needs, ContosoFinancial needs an audit of all requests made to AKS
  
- *Security & Identity*:
	- Identity – ContosoFinancial is an existing O365 user today so they rely heavily on Azure AD for RBAC. 
	- SIEM Integration – Security information needs to flow into QRadar, ContosoFinancial’s existing SIEM tool. 
	- On-prem Connectivity – Although ContosoFinancial has moved a large majority of their workloads to the Cloud, there is still connectivity back to on-prem required. Connectivity back to on-prem requires getting IPs whitelisted to access certain services, as well as cert-based AuthN. 
	- Secrets Mgmt – ConstoFinancial uses Azure Key Vault today for storing secrets, certs and keys for disk encryption for example. What is the recommended way to expose sensitive information to the running containers? 
	- Certificate Authority (CA) – The desire is to leverage ContosoFinancials own CA when possible. 
	- Share Clusters – ContosoFinancial wants to share a cluster out to multiple parties so need to be able to carve the resources up into logical buckets and assign RBAC permissions (RBAC & Namespace Setup). 
	- Container Scanning – ContosoFinancial needs to be able to scan images as part of their DevOps pipeline along with scanning images at runtime for potential threats. 
- *Network* 
	- Existing Azure Network – ConstosoFinancial has adopted a hub and spoke VNET architecture aligned to Virtual Data Center (VDC).  
	- On-prem to Cloud Connectivity with Express Route (ER) – Address space 10.0.0.0/16 is currently used on-premises and extended out to Azure through ER. ContosFinancial would like to use Azure CNI, but foresee a problem with running out of IP space. 
	
- *Container Registry* – Currently use Artifactory today for storing container images. 
	-  ContosoFinancial is currently partial to Artifactory as their container image repository as it provides the ability to have a single store that can be segmented via AuthZ using RBAC. 
	
- *Cost Mgmt* – How does chargeback to the 5 different lines of business work? 

- *Package Management* – Looking for a process or tool to be able to easily deploy services to multiple environments. 

- *No Public IPs* – ContosoFinancial does not want any public endpoints such as control plane endpoints or ssh access to any of the nodes. The only exception to this requirement is Public IPs for public facing workloads such as mobile app API backends. 

- *Blue/Green Deployments* – ContosoFinancial wants to know the recommended practices with respect to AKS around doing blue/green deployment. 


**Division of Work**

- *PM Team + Kevin * :       Governance + Security Setup  – This is all about implementing security controls and setting up the guardrails for the organization which will be inherited by any AKS cluster that gets created. Think of this as the Enterprise Control Plane (ECP). 
		- Azure Security Center vs Guardicore 
		- Azure Policy + AKS (OPA & REGO)??? 
		
- *Pm Team +  Dave Strebel*  -    Cluster Pre-Provisioning – In addition to making sure the proper security controls are in place, which decisions need to be made and technologies in place for the cluster to be provisioned correctly. 
		- Service Principal Creation 
		- Advanced Networking Setup (Egress Rules with F/W) 
		- Subnet Sizing, Services Subnet for ILBs??? 
		- # Nodepools 
		- Ingress Setup – One for the whole cluster or one per namespace or … (needs to be considered when deciding on subnet sizing and whether or not there is a dedicated subnet for services) 
		
		
- *PM Team + Dave Strebel*  - Cluster Provisioning – What decisions need to be made at cluster creation time as they cannot be changed after the fact 
		- North/South Traffic (Private IP, SLB + Egress, Authorized IP) 
		- Azure CNI 
		- Azure AD Integration 
		- Monitoring Add-on 
		- Cluster Autoscaling 
		- Capacity planning
		
- *PM team +  Allen* -  Post-Provisioning Infra-centric – Now that the cluster is up and running, how do I grant access and share it out to multiple lines of businesses or application teams. 
		- RBAC 
		- Namespaces 
		- WAF 
		- F/W 
		- Network Policy w/Calico for East/West 
		- GitOps approach maybe using Flux connected to Repo with Namespace (RBAC and NetworkPolicy stuff to do post-provisioning for Bootstrapping) 
		- Maybe a RBAC and Namespace Operator??? 
		- Container Runtime Monitoring 
		- Availability & Reliability

- *PM team + Tommy* -  Application Planning & Deployment - Sample app - End to end showcase
	Image storage 
	CI/CD pipeline - 
		- Deploy Sample App - Deploy a sample microservice so that all of the requirements can be showcased. 
				- Setup CI/CD pipeline. 
		
-	*PM team +Tommy* - Blue/Green App Deployment – What does a blue/green deployment of the app along with the cluster look like. 
			- Azure DevOps 
			- Terraform vs ARM vs Azure CLI 
			- Helm Charts 


- *PM team + Israel + Allen* -  Post-Provisioning App-Centric – The core infra needs are implemented in the cluster, what about application workload specific requirements such as whether or not a service mesh is needed. Or which ingress controller to use. 
		- Ingress / Egress
		- Container capacity & density
		- Service Mesh 
		- App & container Telemetry 
		- Scale - Horizontal and Vertical
		
- *Kevin + Dave - Cost Governance* – What needs to get put into place to be able to monitor cost and chargeback to the correct lines of business. 
		- Azure Monitor vs Kubecost vs Cloud Custodian 

