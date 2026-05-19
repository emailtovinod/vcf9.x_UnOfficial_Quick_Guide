  
**VMware Cloud Foundation 9.x**

Unofficial Quick Read

VCF 9.0 Architecture Reset  •  VCF 9.1 Refinement Layer

Covering: Management Services • VCF Automation • VKS • Networking • Storage • DC-DR • Private AI

Edition: May 2026  |  Covers VCF 9.0 GA (Jun 2025\) through VCF 9.1 GA (May 2026\)

Sources: Broadcom TechDocs • VMware Blogs • Tom Fojta’s Blog • William Lam • vStellar • Gibson Virtualization • vrealize.it • Adrian Heißler • Official Release Notes

# **1\. Executive Understanding: What VCF 9.x Really Changes**

VCF 9.x is not merely a versioned upgrade of VCF 5.x. It is Broadcom's redesigned private-cloud operating model where operations, lifecycle, licensing, identity, automation, Kubernetes consumption, networking, cost, and security posture are pulled into a unified platform architecture. Broadcom positions VCF 9.0 as a single unified platform for traditional, modern, and AI applications with consistent operations and governance across data centres, edge, and managed cloud infrastructure.

## **1.1 The Three Transformations**

| Transformation | What Changes | Old Model | New Model |
| :---- | :---- | :---- | :---- |
| Operations Transformation | Central management console shifts | SDDC Manager was the primary Day-2 console | VCF Operations becomes the operational heart |
| Consumption Transformation | How tenants provision resources | Ticket-based provisioning, Aria Automation per-tenant | VCF Automation \+ All Apps Org \= self-service private cloud |
| Resilience Transformation | DR and cyber recovery are first-class architecture | VMware SRM \+ vSphere Replication as add-ons | vSAN DP, VMware Live Recovery, on-prem ransomware clean room built in |

## **1.2 VCF 9.0 vs VCF 9.1 Positioning**

* **VCF 9.0 (GA June 2025):** The architectural reset. Introduced the Fleet model, VCF Operations as primary console, VCF Automation Modern Cloud Interface, NSX VPC/TGW networking model, and vSAN ESA Data Protection.

* **VCF 9.1 (GA May 2026):** The optimization and hardening layer. Over 2,000 VCF 9 deployments at GA. Introduced VCF Management Services (containerised common runtime), massive VKS scale improvements, NVMe Memory Tiering, native object storage (tech preview), Continuous Compliance Enforcement, and on-premises ransomware recovery.

| Key Metric:  Broadcom survey of 44 VCF 9 customers (March 2026): 51% reduction in infrastructure management time, 46% reduction in monitoring time, 47% reduction in required capacity versus previous projections, 39% faster mean time to repair. |
| :---- |

## **1.3 CTO-Level Positioning Statement**

| How to Pitch VCF 9.x:  "A governed private-cloud operating model where infrastructure teams become cloud providers, application teams consume VM/Kubernetes/AI/network/storage services through self-service, and the CTO organisation gets fleet-level control over lifecycle, security, cost, identity, and resilience." |
| :---- |

# **2\. VCF 9.x Mental Model: Hierarchy and Constructs**

Every VCF 9.x design decision flows from understanding the hierarchy of constructs. Think of the stack in this order:

| Layer | Construct | Description | Scope |
| :---- | :---- | :---- | :---- |
| 1 – Top | VCF Private Cloud | The entire estate under a single governance model | Global |
| 2 | VCF Fleet | One or more VCF instances managed by a single set of fleet-level components (VCF Operations \+ VCF Automation) | Enterprise / Region |
| 3 | VCF Instance | Classic VCF unit: management domain, SDDC Manager, vCenter, NSX, workload domains | Data centre |
| 4 | Workload Domain | Logical grouping of vSphere clusters with shared NSX overlay and lifecycle boundary | Team / App class / Tenant |
| 5 | vSphere Cluster | Compute cluster with vSAN ESA or external storage, hosting Supervisor or plain VMs | Cluster |
| 6 | Supervisor / VKS / VM Service | Optional Kubernetes control plane enabling VKS clusters, namespaces, VM Service, VPCs | Cluster / Zone |
| 7 – Base | Application Workloads | VMs, containers, VKS-managed pods, AI inference jobs | Per namespace / project |

## **2.1 Fleet-Level vs Instance-Level Components**

| Component | Level | Role in VCF 9.x |
| :---- | :---- | :---- |
| VCF Operations | Fleet-level | Primary operational console: lifecycle, health, cost, certificates, logs, security, licensing |
| VCF Automation | Fleet-level | Self-service consumption layer: VM/VKS/network/volume/image services across vCenters |
| VCF Management Services | Fleet \+ Instance (9.1) | Shared containerised runtime hosting vIDB, Software Depot, Fleet Lifecycle Manager, Log Management |
| SDDC Manager | Instance-level | Still exists per instance; handles instance-scoped orchestration; many actions now surface via VCF Operations |
| vCenter | Instance/Domain-level | vSphere management, Supervisor, cluster, host, and network pool operations |
| NSX Manager | Instance/Domain-level | NSX networking, T0/T1, VPC, TGW, vDefend, edge clusters |

## **2.2 VCF Organisation Construct (Automation)**

In VCF Automation, an Organisation is the top-level tenancy boundary. The provider admin creates organisations to represent tenants, business units, or lines of business. Each organisation operates within its own secure, isolated boundary and consumes capacity through a Region Quota, then subdivides resources into Projects and Namespaces.

| Analogy:  Think of the provider team as building owners who have constructed the power grid, cooling, and network backbone. Organisation admins receive the keys and decide how to allocate space, set access controls, and define shared services for their application teams. |
| :---- |

# **3\. Core Platform Components in VCF 9.x**

## **3.1 VCF Installer**

The VCF Installer is the starting point for all VCF deployments. VCF 9.1 significantly improved the installer workflow, consolidating planning decisions and introducing new deployment paths.

### **VCF 9.1 Installer Deployment Paths**

| Path | Use Case |
| :---- | :---- |
| Deploy a new VCF fleet | Greenfield: fresh VCF instance from scratch including all management components |
| Deploy a new VCF Instance | Add a second VCF instance to an already-deployed fleet |
| Deploy deferred components | Install management components that were skipped during initial bring-up |
| Deploy VCF Management Services | Add the 9.1 Management Services runtime to an existing VCF environment (key for brownfield 9.1 adoption) |
| Converge existing vCenter | Take a vSphere 8.0 Update 3+ environment and fold it into a VCF management domain |
| Upgrade VCF 5.x to VCF 9.0 | Supported in-place upgrade path from previous release |
| Import vSphere 9.0 environment | Absorb an existing vSphere 9.0 deployment into an existing VCF 9.0 instance |

### **VCF 9.1 Installer: New Plan Section**

VCF 9.1 introduces a Plan section in the installer wizard that consolidates key architecture decisions early in the process rather than scattering them across the workflow:

* Deployment Paths: Select what you are deploying before entering details

* Existing Components: Option to reuse existing infrastructure rather than always deploying fresh

* Size Options: Single consolidated page for all component sizing (Simple or HA model, then per-component breakdown of vCPUs, RAM, disk)

* Network Options: Management VLAN design, IP addressing, and network connectivity model

## **3.2 VCF Operations**

VCF Operations is the operational heart of VCF 9.x. It supersedes the older model where SDDC Manager was the primary Day-2 console. In VCF 9.x, cloud administrators are expected to operate the entire private cloud through VCF Operations for fleet-level concerns.

| VCF Operations Capability | Detail |
| :---- | :---- |
| Fleet Management | Single sign-on, centralised identity, password and certificate management, configuration management, tag management |
| Lifecycle Management | Manage both management components (VCF Operations, VCF Automation) and workload components (vCenter, ESX); online/offline depot; binary download and application |
| Health and Diagnostics | Integrated diagnostic dashboards, root-cause analysis, cross-stack health visibility |
| Log Management (9.1) | Native log architecture built on OpenSearch; replaces standalone vRLI cluster under Management Services |
| Security Operations | Authentication audit, user activity, advisories, infrastructure security controls, TLS 1.3 enforcement |
| Cost and Capacity | Cost visibility, capacity planning, chargeback; What-If analysis for Memory Tiering and capacity optimisation |
| Certificate Management | Certificate replacement/renewal, CSR workflows, auto-renewal with Microsoft CA or OpenSSL CA |
| Password Management | Centralised credential rotation across vCenter, NSX Manager, ESX hosts |
| Licensing Management | Central licence file management; 9.1 introduces local licence server integration |
| Continuous Compliance (9.1) | Automated assessment and remediation against PCI-DSS and Security Baseline benchmarks; requires Advanced Cyber Compliance add-on |

## **3.3 VCF Management Services (9.1 — Critical New Architecture)**

VCF Management Services is the headline architectural change in VCF 9.1. It replaces the model where VCF Operations and related components ran as independent standalone VMs.

### **What Is the Management Services Cluster?**

VCF Management Services introduces a common runtime — a managed Kubernetes cluster — that hosts core platform services as containers rather than standalone appliance VMs. This cluster is mandatory for all VCF 9.1 deployments.

| Network Requirements:  A minimum of 12 contiguous IP addresses for cluster worker nodes, plus 4 FQDNs for cluster services. |
| :---- |

### **Services Hosted in the Management Services Cluster (9.1)**

| Service | What It Replaces | Notes |
| :---- | :---- | :---- |
| VMware Identity Broker (vIDB) | Standalone vIDB cluster (VCF 9.0) | After upgrade to 9.1, vIDB merges into the cluster; old vIDB cluster is shut down |
| Software Depot | Multiple product-specific repos (NSX, vCenter, SDDC Manager) | Centralised depot covering all component updates; uses OAuth tokens for secure management in online and disconnected (dark site) mode |
| VCF Lifecycle Manager | Separate Lifecycle Manager VM | Old Lifecycle Manager VM is decommissioned during upgrade; fleet lifecycle and SDDC lifecycle now run here |
| SaltStack Configuration Management | Not present in 9.0 | VMware’s declarative desired-state config management tool; deployed by default in 9.1 |
| VCF Operations for Logs | Dedicated vRLI cluster | Now runs under the services cluster; native log architecture built on OpenSearch |

The VCF Services Runtime Cluster serves as both the host for Management Services and the foundation for VCF Automation deployments. This means VCF Automation also benefits from the unified runtime, simplifying its lifecycle alongside management services.

| Architecture Implication:  With VCF 9.1, the management domain is no longer merely a landing zone for vCenter and NSX. It is the active control plane for the entire fleet's operational stack, hosting containerised services that govern lifecycle, identity, logging, and configuration at scale. |
| :---- |

### **VCF 9.1 Management Services Deployment Models**

Broadcom defines multiple Management Services deployment models based on the availability and scale requirements of each VCF instance. The HA model deploys three worker nodes for the services cluster, while the Simple model uses a single node. Production deployments should always use the HA model.

## **3.4 Day-2 Operations: What Moved Where**

One of the most important operational changes in VCF 9.x is which console is used for which Day-2 action. Architects must redesign runbooks accordingly.

| Day-2 Action | Old (VCF 5.x) | New (VCF 9.x) | Console |
| :---- | :---- | :---- | :---- |
| Workload domain deployment | SDDC Manager | Moved | VCF Operations |
| Cluster creation and expansion | SDDC Manager | Moved | vCenter |
| Host commissioning/decommissioning | SDDC Manager | Moved | vCenter |
| Network pool operations | SDDC Manager | Moved | vCenter |
| NSX Edge cluster deployment | SDDC Manager | Moved | vCenter (network connectivity workflows) |
| Backups | SDDC Manager | Moved | VCF Operations |
| DNS / NTP settings | SDDC Manager | Moved | VCF Operations |
| CA configuration | SDDC Manager | Moved | VCF Operations |
| Certificate management | SDDC Manager | Moved | VCF Operations |
| Password management | SDDC Manager | Moved | VCF Operations |
| Lifecycle management | SDDC Manager \+ vSphere LCM | Moved | VCF Operations |
| Licensing | Component-level keys | Moved | VCF Operations (9.0 licence file; 9.1 local licence server) |

# **4\. Licensing, Identity, and SSO**

## **4.1 Licensing Model Evolution**

| Version | Licensing Model | Key Points |
| :---- | :---- | :---- |
| VCF 5.x and earlier | Component-level long licence keys | Separate keys for each component; manual management; fragmented visibility |
| VCF 9.0 | Single secure licence file managed from VCF Operations | Supports VCF cores, vSAN TiBs, Private AI Foundation, vSphere Foundation cores, VCF Edge cores; centralised management |
| VCF 9.1 | Local licence server \+ multi-licence flexibility | Supports multiple primary licences and licences from multiple Site IDs within a single vCenter; eliminates need for separate vCenters for different licence types; automated usage reporting (online); 4-step offline registration (dark site) |

## **4.2 VCF Identity and SSO**

VCF 9.0 introduces a more unified SSO model. Admins configure VCF Single Sign-On after deployment through a VCF Operations workflow. The VMware Identity Broker (vIDB) is the SSO hub, and in VCF 9.1 it is absorbed into the Management Services Cluster.

### **Supported Identity Providers (VCF 9.x)**

* Microsoft Entra ID (formerly Azure AD)

* Okta

* PingFederate

* Generic SAML 2.0

* OAuth2 / OIDC connected providers

### **VCF SSO Scope and Design**

VCF Single Sign-On covers: VCF Operations, VCF Automation, vCenter (all instances in fleet), NSX Manager instances, and Management Services. Each VCF instance maintains its own vCenter SSO domain, but fleet-level SSO federation ensures a single identity experience for operators.

| Design Decision:  After Supervisor is deployed, changing the load balancer choice (NSX LB vs Avi) is not straightforward. This decision must be made before Supervisor enablement and locked in early in the deployment plan. (Tom Fojta’s Blog) |
| :---- |

## **4.3 Security Operations**

| Security Capability | VCF 9.0 | VCF 9.1 Addition |
| :---- | :---- | :---- |
| TLS Profile | TLS 1.3 default, TLS 1.2 fallback | No change; TLS 1.3 remains default |
| Certificate Automation | CSR workflows, auto-renewal with Microsoft CA / OpenSSL CA | Certificates managed via Management Services |
| SecOps Dashboard | Authentication audit, user activity, advisories | Enhanced with forensic audit trails |
| Continuous Compliance | Not present | Added: PCI-DSS and Security Baseline benchmarks; requires Advanced Cyber Compliance add-on (Advanced Service) |
| Live Patching | Limited scope | Added: Live Patching for TPM-enabled ESX hosts; eliminates unplanned downtime from patching |
| Legacy Surface Removal | Removed CIM/SLP, Update Manager baselines, IWA in vCenter, vSphere Trust Authority | Continued hardening |
| vMotion Encryption | CPU-based | Added: Intel QAT hardware offload; up to 70% CPU savings during migrations |

# **5\. VCF Automation and Organisation Models**

## **5.1 Overview**

VCF Automation (VCFA) is the self-service consumption layer of VCF 9.x. It exposes a public-cloud-like interface for private cloud services, accessible via UI, CLI, and Kubernetes-style declarative APIs. It enables IT to expose governed, self-service resources to application teams with full lifecycle management.

VCF Automation 9.x introduces a Modern Cloud Interface that aggregates services across VCF environments and vCenters into a common endpoint, replacing the older Aria Automation per-vCenter model.

## **5.2 Three Organisation Types**

VCF 9.0 introduces three distinct organisation (tenant) types in VCF Automation. Understanding which to use — and when — is a key architectural decision.

| Organisation Type | Description | Dependency | Primary Use Case |
| :---- | :---- | :---- | :---- |
| All Apps Organisation | The strategic default. Full access to the Modern Cloud Interface, VKS, VM Service, VPC networking, namespaces, extensible services, and Kubernetes IaaS APIs. | Requires vSphere Supervisor enabled on the workload domain | Greenfield private clouds; teams consuming Kubernetes, VMs, AI, and extensible services together |
| VM Apps Organisation | Classic tenant model (rebranded from Aria Automation 8.x). Familiar experience for VM-centric blueprints and catalogs. Designated as legacy/classic. | No Supervisor required; vCenter integration only; enable via feature flag “VM Apps Organisation Creation” | Teams migrating from Aria Automation 8.x; pure VM-centric automation without Kubernetes |
| Provider App (system) | The provider-level management plane. Platform admins use this to create organisations, define regions, assign quotas, configure provider gateways, and manage global policies. | Built-in; always present; accessible at /provider URL | Platform team managing infrastructure; never exposed to tenants |

| Strategic Direction:  Broadcom’s stated strategic direction is All Apps Organisation. VM Apps Organisation will likely enter a deprecation lifecycle in future releases. For greenfield VCF 9.x deployments, All Apps should be the default choice. Teams with existing Aria Automation 8.x tenants will see them appear as VM Apps (Classic Tenant) upon upgrade. |
| :---- |

## **5.3 All Apps Organisation: Configuration Model**

Setting up an All Apps Organisation is a structured provider-then-tenant workflow:

### **Provider Steps**

* Create a Region: maps to a vCenter \+ NSX Manager pair; defines the compute and network boundary

* Create a Zone: represents clusters that provide compute for Supervisors (in VCF 9.0, one cluster per zone)

* Enable Supervisor on the workload domain cluster

* Create the Organisation: creates the tenant boundary and enables isolated access

* Assign Region Quota: defines how much CPU, memory, and storage the organisation can consume from the region; each region can map only one Supervisor per region

* Configure Provider Gateway: import the T0/VRF gateway from NSX; assign IP Spaces for the organisation

* Configure Regional Networking: link the organisation to the provider gateway and Edge cluster

### **Org Admin Steps (After Handover)**

* Create VPCs and Subnets (Public, Private-VPC, or Private-TGW)

* Configure Transit Gateway if multi-VPC routing is needed

* Configure NAT, external IPs, and load balancer resources

* Set up identity and user access (RBAC)

* Create Namespaces / Projects for application teams

* Publish catalog items, blueprints, and policies for self-service consumption

## **5.4 VCF Automation Services Portfolio**

| Service Category | Services Available | Notes |
| :---- | :---- | :---- |
| Core IaaS | VM Service, VKS, Network, Volume, VM Image | Accessible via UI, CLI (kubectl), or Kubernetes declarative API |
| Extensible Services | Harbor Image Registry, Contour, cert-manager, Istio, ExternalDNS, Data Services Manager, Secret Store | Optional services deployed through VCF Automation service catalog |
| 9.1 New Services | Container Service (CaaS), Native Object Storage (S3 – tech preview), SQL Server DBaaS, Live Application Stack Blueprints, Tanzu Marketplace | Container Service runs directly on ESX without full K8s cluster overhead |

## **5.5 Policy-as-Code and Governance Framework**

VCF Automation supports YAML-based policy-as-code for IaaS resources, based on native Kubernetes Validating Admission Policy. Policies can govern VMs and VKS clusters across organisations and vSphere namespaces.

| Governance Area | What to Define |
| :---- | :---- |
| Quotas | CPU, memory, storage, region quota, namespace/project quota; separate limits per organisation |
| Placement Policies | Which zones, clusters, domains, or policies workloads can use; fault domain affinity |
| Day-2 Actions | Who can resize, snapshot, power on/off, delete, or reconfigure; governed by role |
| Lease Policies | How long dev/test resources can exist before automatic cleanup |
| Image Policy | Approved templates, images, content libraries; project-level content libraries (9.1) |
| Network Policy | VPCs, NAT, external IPs, public/private subnets; namespace delegation (9.1) |
| Security Policy | vDefend, microsegmentation, firewall, identity/RBAC, self-service lateral security (9.1) |
| Terraform Integration | Enhanced Terraform provider capabilities (9.1); VCF SDK; API-first operations |

## **5.6 VCF 9.1 Automation Enhancements Summary**

* Container-as-a-Service (CaaS): three distinct runtime options – VM Service, Container Service, VKS – each optimised for different workload patterns

* Fast Deploy for VMs and VKS: linked clone technology reduces VKS cluster deployment from 37 minutes to 11 minutes (69% improvement)

* Regional Harbor: centralised container image registry with replication rules across regions

* Namespace Capture: serialises live environments into reusable YAML blueprints

* Project-level Content Libraries: organisation-scoped image management

* Namespace Delegation: developers can self-provision namespaces with inherited quotas, identity, registry, and ingress

* Fully Allocated Region Quotas: precise resource allocation across organisations

* Enhanced Terraform Provider: richer infrastructure-as-code capabilities for VCF-native resources

* App Stack Formation: version and redeploy entire application topologies as code (Live Application Stack Blueprints)

# **6\. VKS — vSphere Kubernetes Service**

## **6.1 VKS Architecture Overview**

VMware vSphere Kubernetes Service (VKS) is the integrated Kubernetes platform within VCF 9.x. It is managed through vSphere Supervisor and VCF Automation, providing a lifecycle-managed, enterprise Kubernetes service.

VKS deployment workflow is highly automated across multiple orchestration phases: topology generation, control plane node provisioning, worker node provisioning via VMService, Kubernetes component installation, and transition to worker node availability. Understanding these phases is critical for troubleshooting deployment issues and validating infrastructure readiness.

## **6.2 VKS Scale Improvements: VCF 9.0 vs 9.1**

| Metric | VCF 9.0 | VCF 9.1 | Improvement |
| :---- | :---- | :---- | :---- |
| Clusters per Supervisor | Up to 250 | Up to 500 | 2x increase |
| Hosts per fleet | \~2,500 | Up to 5,000 | 2x fleet capacity |
| Parallel upgrade throughput | 64 clusters simultaneously | 256 clusters simultaneously | 4x faster cluster upgrades |
| VKS cluster deployment time | \~37 minutes | \~11 minutes | 69% faster (via linked clone) |
| Kubernetes versions (updates/year) | Coupled to VCF release | 3 VKS updates per year (decoupled) | Faster Kubernetes cadence |

| Enterprise Significance:  A single VCF instance can now plausibly host a very large-scale container platform without the multi-supervisor sharding workarounds required in earlier releases. These numbers materially change the conversation for large enterprises evaluating VKS for enterprise Kubernetes platform strategy. |
| :---- |

## **6.3 Three Container Runtime Options (VCF 9.1)**

VCF 9.1 clearly delineates three distinct runtime options for application teams, eliminating the previous ambiguity:

| Runtime | Description | Expertise Required | Best For |
| :---- | :---- | :---- | :---- |
| VM Service | Traditional VM provisioning through VCF Automation; Fast Deploy via linked clones in 9.1 | Low (infrastructure-centric) | Traditional workloads, VDI, dev environments, rapid VM provisioning |
| Container Service (CaaS) | Serverless-like container runtime executing directly on ESX without K8s cluster overhead; full lifecycle management via UI (no kubectl required) | Very Low (no Kubernetes knowledge needed) | Application teams needing containers without K8s complexity; gentle on-ramp to Kubernetes |
| VKS (vSphere Kubernetes Service) | Full managed Kubernetes clusters with enterprise features; Conformant Kubernetes \+ cloud-native ISV integrations | Medium (Kubernetes-familiar teams) | Microservices, AI inference, stateful applications, multi-tenant isolation, full K8s feature set |

| Design Decision:  The CaaS Container Service provides a YAML-to-VKS upgrade path: when an application architecture evolves, the UI generates consistent YAML for a smooth transition from simple container deployments to full VKS clusters. This means teams can start with CaaS and graduate to VKS without re-architecting. |
| :---- |

## **6.4 VKS Networking Architecture**

VKS clusters deployed in tenant VPC networks must have connectivity to the Supervisor API endpoint. This communication is proxied via the VCF Automation Namespace Proxy service — tenants do not have direct access to Supervisor networks.

* Supervisor control plane nodes communicate over the VM management network with vCenter (one IP per node plus floating IP)

* Supervisor workloads use two VPCs: kube-system VPC and supervisor-services VPC

* VPC External IP Block must be routable across management networks (for vCenter, NSX, Avi, VCFA communication); treat as management network, not tenant-accessible

* Private Transit Gateway IP Block is for subnets routable across VPCs (default 172.20.0/16 unless it clashes)

* Multi-Network Support (VCF 9.1): cluster nodes can be deployed with multiple vNICs to isolate application, storage, and management traffic

## **6.5 VKS Security: Avi \+ vDefend Integration**

Broadcom integrates Avi Load Balancer and vDefend deeply with VKS for enterprise-grade security and load balancing:

* Zero-Touch Deployment: Avi and vDefend are automatically configured across all VKS clusters through VCF Supervisor workflows — no separate installation required

* Self-Service Lateral Security (9.1): distributed micro-segmentation in application-team hands, governed by central guardrails

* Automated Load Balancing (9.1): Avi LB managed through VCF Automation; eliminates hardware appliance requirements

* Distributed IDS/IPS for Kubernetes: network detection and response for container workloads

* 9 Tbps threat inspection performance claimed for distributed inference environments

| Critical Warning:  The load balancer choice (NSX LB vs Avi) must be decided before Supervisor deployment. After Supervisor is deployed, changing the load balancer is not straightforward. Lock this decision in during initial architecture planning. |
| :---- |

# **7\. Networking Architecture: VPC, TGW, and Provider Gateway**

## **7.1 Networking Model Overview**

VCF 9.x introduces a strong cloud-consumption networking model based on Virtual Private Clouds (VPCs) and Transit Gateways (TGWs). This replaces the older model where tenants directly requested NSX T1 gateways. The new model separates provider networking from tenant networking cleanly.

## **7.2 Core Networking Constructs**

| Construct | Description | Who Manages |
| :---- | :---- | :---- |
| Provider Gateway | Imported T0/VRF gateway from NSX; connects tenant VPCs to physical underlay; defines the north-south routing boundary for the region | Provider admin |
| IP Space | Logical container of IP prefixes assigned to an organisation; controls IP allocation for subnets and NAT IPs | Provider admin assigns; org admin consumes |
| Transit Gateway (TGW) | Aggregates VPC traffic and connects VPCs to external networks; supports HA (active-active or active-standby) | Provider admin creates; org admin attaches VPCs |
| VPC (Virtual Private Cloud) | Isolated self-service network domain with custom IP addressing, routing, and security policies; VPC Gateway handles north-south and east-west routing | Org admin creates and manages |
| VPC Gateway | Handles routing between VPC subnets (east-west) and to/from Transit Gateway (north-south) | Automatic per VPC |
| External IP Block | Routable IP ranges used for load balancer VIPs, NAT IPs, and externally accessible services | Provider admin configures; org admin consumes |

## **7.3 VPC Subnet Access Modes**

| Subnet Mode | Purpose | Routing Boundary | Use Case |
| :---- | :---- | :---- | :---- |
| Public | Workloads are directly reachable in the data centre network; subnet is advertised externally | Data centre network | Internet-facing services, shared services accessible without NAT |
| Private-VPC | Isolated inside the VPC; not routable outside; NAT can expose individual services | VPC boundary | Application servers, databases, workloads requiring strict isolation with selective exposure |
| Private-TGW | Routed through Transit Gateway; reachable by other VPCs attached to the same TGW | Transit Gateway | Shared services consumed across multiple tenant VPCs; cross-VPC communication |

## **7.4 Centralised TGW (CTGW) vs Distributed TGW (DTGW)**

| Model | Implementation | Features | Best Fit |
| :---- | :---- | :---- | :---- |
| Centralised TGW (CTGW) | Uses NSX Edge node cluster and Tier-0 router; dedicated Edge VMs handle routing | Full NSX feature set: NAT, DHCP, LB, Gateway Firewall, DFW, richer services; supports internet/WAN connectivity | Enterprise production; internet/WAN-connected tenants; advanced networking services; financial services |
| Distributed TGW (DTGW) | Uses distributed routing on ESX hosts; sometimes called “edgeless”; can connect northbound directly to VLAN | Smaller footprint; no dedicated Edge VMs; limited feature set compared to CTGW | VLAN-heavy environments; edge/remote sites with constrained hardware; lower-complexity use cases |

| BFSI Design Guidance:  For regulated financial services environments, use Centralised TGW (CTGW) as the default. Reserve DTGW for branch/edge scenarios. Use dedicated Provider Gateways (VRFs) for high-isolation tenants or sensitive zones; use shared Provider Gateways for general-purpose dev/test. |
| :---- |

## **7.5 Provider and Tenant Networking Separation**

A VCF Automation Region typically maps to a workload domain and NSX Manager domain. The provider imports T0/VRF gateways as Provider Gateways, assigns IP Spaces, and configures regional networking. Tenant organisations then consume VPCs, subnets, Transit Gateways, NAT, and external IPs within their governed boundary.

| Layer | Responsibility | Managed By |
| :---- | :---- | :---- |
| Provider Layer | Physical underlay, T0/VRFs, Edge clusters, IP Spaces, Provider Gateways, external connectivity | Platform/Provider team |
| Regional Layer | Region definition, zone mapping, Supervisor configuration, TGW creation, quota assignment | Platform/Provider team |
| Organisation Layer | VPCs, subnets, NAT, external IPs, load balancer resources, firewall policies within VPC | Org Admin |
| Project/Namespace Layer | Application-level networking, service mesh, ingress, network policies within namespaces | Application team (governed by org policies) |

## **7.6 EVPN/VXLAN and Physical Fabric Integration (VCF 9.1)**

VCF 9.1 adds standards-based EVPN with Arista, Cisco, and SONiC integration, delivering a consistent overlay fabric across the three dominant data centre networking stacks. This is significant for:

* Multi-vendor underlay environments: one operating model regardless of switch vendor

* Open-source fabric adoption: SONiC support legitimises open networking for VCF underlay

* Estate absorption: shortens the time to onboard new sites or absorb acquired estates

# **8\. Private AI and Infrastructure Efficiency (VCF 9.1)**

## **8.1 Why Private AI on VCF**

Broadcom's Private Cloud Outlook 2026 report reveals that more than 56% of organisations are running or planning production inferencing in a private cloud. Public cloud use for production inference fell 15 percentage points year-over-year. Key drivers are data sovereignty, IP protection, cost predictability, and regulatory requirements (especially relevant for BFSI clients subject to DORA, RBI, and similar frameworks).

## **8.2 NVMe Memory Tiering**

Memory Tiering is one of the most impactful cost-reduction capabilities in VCF 9.1. It extends effective memory pools by using NVMe SSDs as a secondary memory tier:

| Aspect | VCF 9.0 | VCF 9.1 Enhancement |
| :---- | :---- | :---- |
| Core Concept | Hot pages in DRAM; cold pages offloaded to NVMe | Same model with major performance improvements |
| Database Performance Gain | Baseline | Up to 16% improvement in HammerDB benchmarks |
| CPU Overhead | \~5-10% | 12% CPU reduction in VMmark benchmarks |
| NVMe Tier Redundancy | Hardware RAID only (Tri-Mode controller or Intel VROC) | Native software mirroring added; no additional controller hardware needed |
| VM Profile Restrictions | Security VMs, low-latency VMs, FT VMs could not power on with Memory Tiering | All restrictions removed; all VM profiles supported |
| Nested Virtualisation | Not supported with Memory Tiering | Fully supported; nested VMs participate in tiering |
| TCO Claim | Baseline | Up to 40% lower server costs; higher VM density |
| Observability | Basic | Dedicated Memory Tiering dashboard in VCF Operations; What-If analysis for pre-purchase modelling |

## **8.3 vSAN Global Deduplication and Compression**

VCF 9.1 extends vSAN storage efficiency for AI data pipelines:

* Global deduplication operates cluster-wide (not just per disk group), eliminating redundant data across the entire cluster

* Enhanced compression operates continuously in the background without impacting application behaviour

* Both capabilities support encrypted environments

* Claimed TCO reduction: up to 39% lower storage TCO for AI data pipelines

## **8.4 GPU and Accelerator Support**

| Capability | Detail |
| :---- | :---- |
| AMD GPU Support | Enhanced DirectPath I/O for latest AMD Instinct GPUs (including MI350); near-bare-metal accelerator performance |
| NVIDIA GPU Support | ConnectX-7 NICs and BlueField-3 with Enhanced DirectPath I/O for high-speed AI model training and data transfer |
| AI vMotion | GPU-enabled workloads can migrate between GPUs with zero downtime |
| Topology-aware Scheduling | NUMA-aware and GPU-topology-aware placement for AI workloads |
| DirectPath I/O | PCIe passthrough for near-bare-metal GPU performance within VMs |

## **8.5 Private AI Observability**

VCF 9.1 adds telemetry specifically for AI workloads in VCF Operations:

* GPU Metrics: utilisation, memory pressure, and model-level visibility on the same console as the rest of the estate

* AI Model Metrics: token usage, agent activity, MCP server inventory, model behaviour tracking

* Private AI Model Dashboard: consolidated view for MLOps teams without needing separate monitoring stacks

## **8.6 vSphere Elastic Provisioning**

VCF 9.1 introduces vSphere Elastic Provisioning to simplify how hosts are brought online and assigned to workloads:

* Parallel imaging of hosts: eliminates sequential provisioning bottlenecks

* Automated discovery: hosts auto-enrol into the fleet once they boot

* Consistent configuration: SaltStack desired-state configuration applied automatically via Management Services

* Zero-touch edge provisioning: bare metal can boot, connect to vCenter, receive configuration, and integrate into VCF without manual setup at remote sites

# **9\. Storage Architecture**

## **9.1 vSAN ESA as the Default Building Block**

vSAN Express Storage Architecture (ESA) is the default storage model for VCF 9.x clusters. It is designed for NVMe-based storage tiers and provides significantly better performance, compression, and snapshot capabilities compared to vSAN OSA.

| Storage Model | Description | Best Fit |
| :---- | :---- | :---- |
| vSAN ESA HCI Cluster | Converged compute and storage on NVMe; default VCF 9.x building block; supports Data Protection, global dedup/compression, Memory Tiering | Default production clusters; most new VCF deployments |
| vSAN Storage Cluster (Disaggregated) | Separates storage nodes from compute nodes; storage nodes provide vSAN capacity consumed by compute-only nodes | Independent scaling of compute and storage; storage-intensive workloads on dedicated nodes |
| vSAN Stretched Cluster | Synchronous replication across two sites with a witness at a third; site-aware placement; eliminates metro storage array dependency | Metro availability within \~5ms RTT; local site failure tolerance without invoking regional DR |
| External VMFS / NFS | Traditional FC, iSCSI, or NFS storage arrays; NFS used for VCF Automation Supervisor storage classes | Legacy integration; specific performance requirements; enterprise array investment reuse |
| vSAN Storage Cluster Stretched Topology | VCF 9.0 introduced vSphere cluster \+ vSAN storage cluster combination for stretched arrangements; reduces hardware dependency | Availability requiring site-awareness without full dual-site HCI |

## **9.2 vSAN Data Protection**

vSAN Data Protection (introduced in VCF 9.0) provides native snapshot and replication capabilities without requiring third-party backup software for VM-level protection:

| Capability | Detail |
| :---- | :---- |
| Snapshot Retention | Up to 200 snapshots per VM; VM-level protection granularity |
| Local Datastore Snapshots | Point-in-time recovery without replication; for short-term recovery within the cluster |
| Remote Datastore Snapshots | Replicate snapshots to a remote vSAN ESA datastore; supports asynchronous replication |
| vSAN-to-vSAN Replication | Introduced in VCF 9.0; lower-cost asynchronous replication between vSAN ESA datastores; simplifies recovery vs traditional array-based approaches |
| Host-based VM Replication | RPO as low as 1 minute for mission-critical applications |
| Recovery | VM-level protection and recovery workflows; supports DR orchestration through VMware Live Recovery |

## **9.3 vSAN for Recovery (VCF 9.1)**

VCF 9.1 introduces vSAN for Recovery, which unifies disaster recovery and ransomware recovery under a single native capability:

* Efficient replication of workloads using native snapshot capabilities

* Deep snapshot chains support both DR and cyber recovery workflows

* Integrated replication for fast recovery while maintaining operational simplicity

* Works alongside VMware Live Recovery / VCF Protection and Recovery for orchestrated failover

# **10\. Deployment Topologies**

## **10.1 Appliance Deployment Models**

### **Simple vs High Availability**

| Model | NSX Manager | VCF Operations | VCF Automation | VCF Mgmt Services (9.1) | Use Case |
| :---- | :---- | :---- | :---- | :---- | :---- |
| Simple | 1 node | 1 node | 1 node | Single-node cluster | Lab, PoC, evaluation, constrained environments |
| High Availability | 3 nodes | 3 nodes | 3 nodes | 3-node cluster | Production; recommended for all enterprise and BFSI deployments |

| Production Rule:  HA model is mandatory for production. The reason is not only failure tolerance but also reduced disruption during lifecycle, patching, and upgrade operations. Never deploy production VCF in Simple mode. |
| :---- |

## **10.2 Minimum Component Count (VCF 9.1 Simple Model)**

For reference, a minimal VCF 9.1 Simple deployment includes:

* vCenter Server

* SDDC Manager

* NSX Manager

* VCF Operations Manager

* VCF Management Services Cluster (new in 9.1; replaces standalone Fleet Management Appliance and vIDB cluster)

* VCF Automation

* VKS Appliance (if Supervisor is selected)

## **10.3 Infrastructure Blueprints (Broadcom TechDocs)**

| Topology | Description | Recommended For |
| :---- | :---- | :---- |
| Single-site minimal footprint | One VCF instance, Simple model, minimal host count; no HA for management components | Lab, PoC, small edge, constrained budget environments |
| Single-site HA | One VCF instance, HA model for all management components; standard production starting point | Standard production; most enterprise deployments should start here |
| Multiple sites, single region | Two or more VCF instances in the same metro/region; shared fleet-level management; fault domain design | Large campus or DC-pair design; local fault-domain protection within a region |
| Multiple sites, single region \+ additional region | Primary metro VCF fleet plus a secondary region for DR or edge; cross-region fleet management | Enterprise with primary metro plus regional DR; sovereign data boundary requirements |
| Multiple regions | VCF fleet spanning national or global geography; sovereign zoning; strict DR posture | National/global enterprise; regulatory data residency requirements; active-active across regions |
| Fleet with DR design | VCF management plane itself is protected with replication and recovery; management-plane DR is an explicit architecture concern | Enterprises where the control plane SLA is as important as the application SLA |

## **10.4 Workload Domain Topologies**

| Pattern | Description | When to Choose |
| :---- | :---- | :---- |
| Consolidated management \+ workloads | Management domain also hosts some organisation workloads; shared initial infrastructure | Lab, PoC, small environments; not for production regulated workloads |
| Shared workload domain (multi-tenant) | Multiple organisations or teams share one workload domain; isolation via VCF Automation \+ NSX VPC | Enterprise shared private cloud; cost-efficient multi-tenancy with NSX/VPC segmentation |
| Dedicated workload domain per tenant or app class | Each major tenant, compliance boundary, or workload class gets a dedicated WLD with its own vCenter, NSX, and clusters | BFSI/regulated environments; strict isolation; noisy-neighbour avoidance; audit boundary separation |

| BFSI Design Pattern:  For banking and financial services: use dedicated workload domains for production systems of record (core banking, payment processing, trading); use a shared workload domain for internal tools, dev/test, and analytics. Apply vDefend micro-segmentation across all domains regardless of sharing model. |
| :---- |

# **11\. DC – DR Strategy for VCF 9.x**

## **11.1 Layered Resilience Model**

VCF 9.x enables a four-layer resilience model. Each layer addresses a different failure scope, and an enterprise architecture should combine all four rather than treating them as alternatives.

| Layer | Scope | Mechanism | VCF 9.x Capability |
| :---- | :---- | :---- | :---- |
| Layer 1: Local HA | Host or component failure within a site | vSphere HA, DRS, HA appliance models, redundant management networks | HA deployment model; vSAN policy-based redundancy; NSX/VCF Operations HA |
| Layer 2: Metro / Stretched Availability | Site failure within metro (\< 5ms RTT) | vSAN stretched clusters; site-aware cluster topology | vSAN stretched clusters; vSphere cluster \+ vSAN storage cluster stretched topology (VCF 9.0) |
| Layer 3: Regional DR | Data centre failure or regional disaster | Asynchronous replication; orchestrated failover | vSAN Data Protection \+ vSAN-to-vSAN replication; VMware Live Recovery / VCF Protection and Recovery; RPO as low as 1 minute |
| Layer 4: Cyber / Ransomware Recovery | Active compromise, encryption, or data corruption | Isolated clean room; immutable recovery points; validation before cutover | On-premises VCF isolated clean room (VVS); CrowdStrike Falcon integration; forensic audit trails; vSAN for Recovery (VCF 9.1) |

## **11.2 Layer 1: Local HA — Minimum Production Requirements**

* HA appliance model for all management components (NSX, VCF Operations, VCF Automation, Management Services)

* Redundant management networks (management VLAN \+ second VLAN as per VCF 9.1 installer network options)

* Minimum 4 ESX hosts per cluster for vSAN ESA data availability (3-host minimum technically possible; 4+ for maintenance without degraded protection)

* vSphere HA and DRS enabled on all clusters

* Redundant ToR switches and uplinks; appropriate bonding on hosts

* Storage policy enforcing FTT=1 (RAID-1 or RAID-5 depending on host count) as minimum

## **11.3 Layer 2: Metro Availability**

Use vSAN stretched clusters for workloads requiring local-site failure tolerance without invoking a full DR failover:

* Maximum \~5ms RTT between sites (synchronous replication constraint)

* Witness at a third location (can be lightweight/virtual)

* vSphere DRS site affinity rules ensure workloads run at the preferred site

* VCF 9.0 supports site-aware stretched topology using vSphere clusters \+ vSAN storage clusters, reducing hardware dependency vs older dual-site HCI models

| Warning:  Stretched cluster is an availability mechanism, not a complete DR or cyber recovery strategy. It does not replace immutable recovery points, ransomware recovery, or isolated clean rooms. It also does not protect against data corruption that replicates synchronously to both sites. |
| :---- |

## **11.4 Layer 3: Regional DR**

For site-to-site DR, use VMware Live Recovery (VCF Protection and Recovery):

* Supports DR orchestration with runbooks, test failover, and failback

* Enhanced vSphere Replication with RPO as low as 1 minute for mission-critical applications

* vSAN Data Protection integration: local datastore snapshots, remote datastore snapshots, vSAN-to-vSAN replication

* Management plane protection: VCF Operations, VCF Automation, and Management Services should also be replicated (see management-plane DR order below)

## **11.5 Layer 4: Cyber Recovery / Ransomware Recovery**

VCF 9.0 adds support for cyber recovery to an on-premises VCF isolated clean room as a VMware Validated Solution (VVS). VCF 9.1 deepens this with native ransomware recovery:

| Capability | VCF 9.0 | VCF 9.1 Enhancement |
| :---- | :---- | :---- |
| On-Premises Clean Room | On-premises VCF isolated clean room as VVS | Integrated as a first-class platform feature; no longer just a VVS add-on |
| Recovery Orchestration | VMware Live Recovery with manual clean room validation | Integrated validation tools within the platform |
| Endpoint Security | vDefend for network-layer protection | CrowdStrike Falcon Endpoint Security integration for recovery workflow |
| Forensic Capability | Audit trails for authentication events | Centralized historical findings; standardised log architecture; forensic data for incident investigations |
| Data Sovereignty | Avoid cloud-based recovery to prevent cross-border data movement | On-premises avoids massive bandwidth fees during crisis restoration |

The Ransomware Recovery VVS design places the recovery instance in a separate, isolated workload domain. Business-critical workloads are replicated to this isolated domain. In the event of an attack, workloads are recovered to the isolated environment for forensic analysis and validation before being returned to production.

## **11.6 Management Plane DR: Recovery Order**

Do not design DR only for application VMs. The VCF management plane itself needs a recovery design. Architect a documented recovery order:

| Step | Component | Why This Order |
| :---- | :---- | :---- |
| 1 | Physical network / underlay / routing / DNS / NTP | Everything depends on IP reachability and name resolution |
| 2 | Identity provider / VCF Identity Broker / SSO dependencies | All management plane logins require identity before anything else |
| 3 | vCenter and NSX management components | Platform operations require vCenter before SDDC Manager or VCF Operations can function |
| 4 | SDDC Manager / VCF Instance management | Instance-level orchestration resumes after vCenter and NSX are available |
| 5 | VCF Operations / Management Services (9.1) | Fleet-level operational visibility and lifecycle management restored |
| 6 | VCF Automation | Self-service consumption layer restored for tenant operations |
| 7 | VKS / Supervisor / tenant services | Kubernetes platform services restored; VKS clusters can be failed over |
| 8 | Application workloads | Business workloads are the last to recover; management plane must be stable first |
| 9 | Observability, backup, security, and cost tools | Non-critical supporting services restored last |

# **12\. Security Architecture**

## **12.1 vDefend: NSX-based Security Platform**

VMware vDefend is the security component of the NSX platform in VCF 9.x, providing network-based protection for VM and containerised workloads:

| Capability | Description |
| :---- | :---- |
| VPC-aware Micro-segmentation | Distributed Firewall (DFW) policies apply at the vNIC level; east-west traffic control independent of VLAN topology |
| Lateral Movement Prevention | NSX-based distributed IDS/IPS detects and blocks lateral movement; critical for ransomware containment |
| Distributed IDS/IPS for Kubernetes | Network detection and response integrated with VKS workloads in VCF 9.1 |
| Gateway Firewall | North-south traffic inspection at T0/T1 boundary; stateful inspection |
| Network Detection and Response | Threat analytics across network flows; complements endpoint tools like CrowdStrike |
| Self-Service Lateral Security (9.1) | Application teams can apply micro-segmentation within their VPCs, governed by platform policy guardrails; no ticket needed |

## **12.2 Continuous Compliance Enforcement (VCF 9.1)**

VCF 9.1 introduces Continuous Compliance Enforcement as part of the Advanced Cyber Compliance (ACC) add-on service:

* Automated compliance assessments across the entire VCF fleet

* Supported benchmarks at launch: PCI-DSS and VCF Security Baseline

* Built-in drift detection: immediate notification when configuration deviates from the benchmark

* Centralized remediation workflows: fix compliance issues directly from the VCF Operations UI

* Audit trail integration: forensic-grade logs for incident investigation and regulatory evidence

* Transforms compliance from a quarterly audit exercise into a runtime guarantee

| BFSI Relevance:  Continuous Compliance Enforcement directly addresses DORA (Digital Operational Resilience Act), PCI-DSS, and similar regulatory requirements by providing continuous automated evidence of compliance posture rather than point-in-time audit snapshots. |
| :---- |

## **12.3 Security Design Checklist**

| Area | Key Decisions |
| :---- | :---- |
| Identity | VCF SSO with Entra ID / Okta / SAML; MFA enforcement; service account governance |
| Certificate Management | Microsoft CA or OpenSSL CA integration; auto-renewal; TLS 1.3 profile enforcement |
| Network Security | vDefend DFW policies; dedicated vs shared Provider Gateways per tenant; gateway firewall rules |
| Kubernetes Security | Avi LB \+ vDefend for VKS; policy-as-code via Validating Admission Policy; namespace RBAC |
| Endpoint Security | CrowdStrike integration for ransomware recovery workflow (9.1); host-based protection strategy |
| Compliance | Continuous Compliance Enforcement (ACC add-on); PCI-DSS and Security Baseline benchmarks |
| Patch Management | Live Patching for TPM-enabled hosts (9.1); fleet lifecycle management via VCF Operations |
| Audit and Forensics | Centralised log management on OpenSearch (Management Services); forensic audit trails; event correlation |

# **13\. Configuration Options Architects Must Decide Early**

The following decisions must be locked before or during initial deployment. Changing these after deployment ranges from disruptive to unsupported.

| Area | Key Decisions | Timing |
| :---- | :---- | :---- |
| Deployment Model | Simple vs HA; production must be HA; HA cannot be easily changed post-deployment | Day 0 |
| Fleet Scope | Single VCF instance vs multiple VCF instances under one fleet; determines fleet lifecycle complexity | Day 0 |
| Management Domain Sizing | Dedicated management cluster sizing; vSAN ESA vs external storage; HA placement rules | Day 0 |
| Load Balancer | NSX LB vs Avi; MUST be decided before Supervisor deployment; cannot change easily after | Day 0 / Before Supervisor |
| Identity and SSO | VCF SSO configuration; external IdP (Entra ID / Okta / SAML); Identity Broker design | Day 0-1 |
| Licensing | VCF 9.0 single licence file vs VCF 9.1 local licence server; connected vs offline (dark site) mode | Day 0 |
| VCF Automation Org Model | VM Apps vs All Apps (strategic default should be All Apps); tenant boundary design | Day 1 |
| Supervisor Configuration | Zones (one cluster per zone in 9.0); Supervisor VPC networking; IP Blocks; Edge cluster selection | Day 1 |
| Networking Model | Shared vs dedicated Provider Gateway per organisation; CTGW vs DTGW; IP Space design; NAT model | Day 0-1 |
| Storage Policy | vSAN ESA storage profiles; FTT; RAID policy; snapshot retention rules; replication targets | Day 1 |
| Workload Domain Model | Shared vs dedicated WLD per tenant/app class; NSX instance scope | Day 1 |
| DR Strategy | vSAN DP scope; VLR/Protection and Recovery targets; ransomware recovery isolation design | Day 1-2 |
| Observability | VCF Operations scope; log retention in OpenSearch; Prometheus integration; VKS visibility | Day 1-2 |
| Security Posture | vDefend policies; TLS profile; CA integration; Continuous Compliance scope (if ACC add-on) | Day 1-2 |

# **14\. Brownfield Adoption Strategy**

## **14.1 Supported Migration Pathways**

| Pathway | Description | Key Considerations |
| :---- | :---- | :---- |
| New VCF 9.x Greenfield | Deploy fresh VCF 9.x fleet from scratch | Simplest; use VCF Installer Plan section for architecture decisions; full VCF 9.1 feature set from day one |
| Converge vSphere 8.0 U3+ to VCF Management Domain | Take an existing vCenter and convert it into a VCF Management Domain | vSphere 8.0 Update 3 minimum; network and storage must meet VCF requirements; SDDC Manager is deployed into the existing environment |
| Upgrade VCF 5.x to VCF 9.0 | In-place upgrade of existing VCF 5.x instance | Plan Aria Automation upgrade as separate workstream if custom integrations exist; test upgrade in a lab first |
| Import vSphere 9.0 to existing VCF 9.0 | Absorb an existing vSphere 9.0 environment as a workload domain in an already-running VCF 9.0 instance | vSphere 9.0 required; NSX must be present or deployed during import |
| Deploy VCF Management Services to existing VCF | VCF 9.1 installer can add Management Services to an existing VCF deployment | New in VCF 9.1; enables 9.1 architecture features without full rebuild |

## **14.2 Brownfield Discovery Checklist**

| Area | What to Assess |
| :---- | :---- |
| vCenter Topology | Number of vCenters, SSO domains, linked mode groups, version; identify any non-standard configurations |
| NSX Estate | NSX version, transport zones, T0/T1 design, Edge clusters, microsegmentation policies, DFW rule complexity |
| Aria / VCF Operations | Existing Aria Operations, Logs, Networks, Automation; custom management packs; integration dependencies |
| Storage | vSAN ESA vs OSA (OSA is not directly supported in VCF 9.x new deployments); external arrays; stretched clusters |
| Identity | AD/LDAP configuration, SAML/OIDC, MFA, service accounts; SSO domain structure |
| Certificates | Internal CA, external CA, certificate expiration, automation readiness; wildcard vs per-component certs |
| Workload Domains | Current clusters, workload placement, lifecycle boundaries; running vs decommission candidates |
| Automation Estate | Existing Aria Automation blueprints, integrations (ServiceNow, Terraform), custom event topics; migration complexity |
| Kubernetes Estate | Existing TKG/TKGm/VKS/OpenShift; container registry, ingress controllers, service mesh, persistent storage |
| DR and Backup | SRM/VLR, vSphere Replication, backup tools (Veeam/Cohesity/Commvault), cyber recovery, immutability status |
| Licensing | Current licence keys per component; mapping to VCF 9.x licence file (cores, TiBs, Edge); ELA scope |

| Aria Automation Upgrade Warning:  If your existing environment has Aria Automation with custom integrations, plan the upgrade as an independent workstream. Complex custom integrations may require refactoring before or after upgrade to VCF Automation 9.x. A direct lift-and-shift of complex Aria Automation tenants to All Apps Organisations is not always straightforward. |
| :---- |

# **15\. Recommended Study Path**

| Phase | Topic Area | Key Concepts to Master | Primary Sources |
| :---- | :---- | :---- | :---- |
| Phase 1 | Platform Foundation | VCF 9.x hierarchy (fleet, instance, domain); VCF Installer 9.1 (paths, Plan section, sizing); mental model shift from component stack to fleet-managed private cloud | Broadcom TechDocs Design, VMware Blogs deployment pathways, vStellar home lab series |
| Phase 2 | Management Services and Operations | VCF Operations capabilities; VCF Management Services cluster (what services it hosts, networking requirements); Management Services deployment models; Day-2 operation console mapping; lifecycle management flow | Gibson Virtualization VCF 9.1 blog, TechDocs VCF Management Services Models, VMware Blog 9.0 Day-2 Operations |
| Phase 3 | VCF Automation and Org Models | All Apps vs VM Apps vs Provider App; Region, Zone, Quota model; VPC networking for Supervisor; provider handover to org admin; policy-as-code; 9.1 CaaS and Fast Deploy | VMware Blog All Apps Org Configurations, Tom Fojta Networking Deep Dive, vrcloud24x7 Org Deep Dive, Adrian Heißler Configuration Guide |
| Phase 4 | VKS and Containers | VKS architecture and deployment phases; VKS 9.1 scale numbers; three runtime options (VM/CaaS/VKS); Avi \+ vDefend integration; multi-network support | VMware Blog VKS 9.1, William Lam VKS lab, VCF HOL What’s New VKS |
| Phase 5 | Networking Architecture | VPC, TGW, CTGW vs DTGW; Provider Gateway and IP Spaces; subnet access modes; EVPN/SONiC underlay; Supervisor VPC networking topology (from Tom Fojta) | Tom Fojta Networking Deep Dive, VMware Blog VPC Architecture, Broadcom TechDocs VPC in NSX |
| Phase 6 | Storage, Data Protection, Private AI | vSAN ESA; vSAN Data Protection (snapshots, replication); vSAN for Recovery (9.1); Memory Tiering (9.0 vs 9.1); GPU support; global dedup/compression | VMware Blog vSAN 9.0, Advanced Memory Tiering VCF 9.1, vSAN Data Protection blog, VCF 9.1 Announcement |
| Phase 7 | DC-DR and Security | Four-layer resilience model; management-plane DR order; ransomware recovery VVS; Continuous Compliance Enforcement; vDefend; cyber clean room design | TechDocs On-Prem Ransomware Recovery VVS, TechDocs Site Protection VVS, VMware Blog Live Recovery VCF 9.0, VCF 9.1 security blog |
| Phase 8 | Enterprise Design Patterns | Build reference architectures for: single-site HA, dual-site metro, active/passive DR, multi-region VCF fleet, shared tenant, dedicated regulated tenant, Private AI-ready VCF, edge VCF | Broadcom TechDocs Blueprints, William Lam VCF 9.1 Design Blueprints and Fleet Latency Diagrams (ports.broadcom.com) |

# **16\. CTO-Grade Architecture Recommendation**

## **16.1 Recommended Baseline Architecture for Enterprise / BFSI**

| Layer | Recommendation | Rationale |
| :---- | :---- | :---- |
| Fleet Design | One production VCF fleet per major operational boundary; avoid uncontrolled fleet sprawl | Fleet-level components span the entire estate; too many fleets increase management overhead |
| Management Domain | Dedicated, HA, protected by management-plane DR; minimum 4 hosts, vSAN ESA | Management domain is the control plane; treat its resilience with the same rigour as production workloads |
| VCF Operations | HA deployment; make it the sole authoritative console for fleet operations | Standardising on VCF Operations ensures all lifecycle and security actions are tracked and governed |
| Management Services (9.1) | Deploy with HA cluster model; plan 12+ contiguous IPs and 4 FQDNs during design | Mandatory in 9.1; hosts critical services including Identity Broker, Software Depot, and Lifecycle Manager |
| VCF Automation | HA deployment; enable All Apps Org as the strategic consumption model | All Apps provides the full self-service surface; aligns with Broadcom’s product roadmap direction |
| Tenancy Model | Shared WLD for general workloads; dedicated WLD for regulated/high-risk tenants | Balance of resource efficiency (shared) with compliance isolation (dedicated) |
| Networking | CTGW for feature-rich production tenants; DTGW selectively for VLAN/edge use cases; dedicated Provider Gateway (VRF) per regulated tenant | CTGW provides full NSX feature set required for regulated environments |
| Load Balancer | Avi for VKS and Supervisor; decide before Supervisor deployment | Deep integration with VKS; cannot change easily after Supervisor enablement |
| Storage | vSAN ESA where possible; storage clusters for disaggregated scaling; external storage only where justified | vSAN ESA delivers best integration with VCF lifecycle, Data Protection, and Memory Tiering |
| DR Architecture | Combine: HA (Layer 1\) \+ stretched availability (Layer 2, metro) \+ regional DR via VLR (Layer 3\) \+ cyber clean room (Layer 4\) | No single mechanism addresses all failure modes; defence-in-depth is required |
| Security | VCF SSO \+ Entra ID/Okta; cert automation; TLS 1.3; vDefend microsegmentation; Continuous Compliance (ACC add-on); CrowdStrike for ransomware recovery | Zero-trust posture across identity, network, and compliance |
| Private AI | Memory Tiering on compute clusters running mixed AI/non-AI; GPU DirectPath I/O for inference nodes; GPU Metrics dashboard in VCF Operations | Reduces infrastructure cost while maintaining performance for AI workloads |
| Operating Model | Platform team owns fleet; Org Admins own tenant/project governance; App teams consume services via VCF Automation UI/CLI/API | Role clarity reduces operational friction and enables self-service without sacrificing governance |

## **16.2 Operating Model: Three-Tier Responsibility**

| Team | Scope of Responsibility | Primary Console | Key Actions |
| :---- | :---- | :---- | :---- |
| Platform / Provider Team | Fleet lifecycle, licensing, global security, Management Services, underlay, shared infrastructure | VCF Operations (fleet view) | Lifecycle management; certificate automation; fleet health; quota assignment; provider gateway configuration |
| Org Admin (per tenant) | Organisation resources: VPCs, namespaces, projects, users, policies, catalog, quotas | VCF Automation (org UI) | Create VPCs; manage users and roles; publish catalog items; enforce org-level governance; day-2 on org resources |
| Application / Dev Team | Application workloads within governed namespaces and projects | VCF Automation (tenant UI), kubectl, VCF CLI | Self-service VM and VKS provisioning; Day-2 on their resources; image management; namespace consumption |

## **16.3 Key Takeaway**

| Three Transformations in One:  VCF 9.x is best understood as three transformations happening simultaneously:1. OPERATIONS TRANSFORMATION — VCF Operations becomes the central console for fleet, lifecycle, security, logs, health, cost, certificates, passwords, and licensing.2. CONSUMPTION TRANSFORMATION — VCF Automation and All Apps Organisation make VCF a self-service private cloud for VM, Kubernetes, network, volume, image, CaaS, and extensible services.3. RESILIENCE TRANSFORMATION — vSAN Data Protection, vSAN-to-vSAN replication, vSAN for Recovery, VMware Live Recovery, on-premises ransomware recovery, and Continuous Compliance Enforcement make DR and cyber recovery first-class architecture concerns.For CTO-level study, focus less on individual screens and more on the new operating model: Fleet → Operations → Automation → Organisation → VPC/Namespace → Application. |
| :---- |

# **17\. Quick Reference: VCF 9.x at a Glance**

## **17.1 What Is New in VCF 9.1 vs 9.0**

| Capability Area | VCF 9.0 | VCF 9.1 Addition |
| :---- | :---- | :---- |
| Management Architecture | Standalone Fleet Management Appliance \+ standalone vIDB cluster | VCF Management Services Cluster (containerised runtime) replaces both; mandatory |
| Software Depot | Per-product repos (NSX, vCenter, SDDC Manager) | Centralised Software Depot under Management Services; OAuth token auth |
| Lifecycle Manager | Separate VM | Absorbed into Management Services Cluster; old VM decommissioned on upgrade |
| Log Management | Separate vRLI cluster | Native log management on OpenSearch under Management Services |
| Configuration Management | Not present | SaltStack deployed by default; declarative desired-state config |
| Licensing | Single licence file managed from VCF Operations | Local Licence Server; multi-licence and multi-Site-ID support; automated usage reporting |
| VKS Scale | 250 clusters/Supervisor; 2,500 hosts/fleet | 500 clusters/Supervisor; 5,000 hosts/fleet; 4x faster upgrades |
| Container Runtime | VKS only | Three options: VM Service, Container Service (CaaS), VKS |
| VKS Provisioning Speed | \~37 min/cluster | \~11 min/cluster (linked clones) |
| Memory Tiering | Basic NVMe tiering; limited VM profiles supported | 16% perf gain; software mirroring; all VM profiles supported; nested VM support; What-If analysis |
| Storage Efficiency | vSAN Data Protection | Global dedup+compression; vSAN for Recovery (DR \+ ransomware) |
| Compliance | Manual assessment | Continuous Compliance Enforcement (PCI-DSS \+ Security Baseline); ACC add-on |
| Ransomware Recovery | On-premises clean room as VVS | Native integration; CrowdStrike Falcon support; on-prem sovereignty |
| GPU Support | NVIDIA DirectPath I/O | AMD Instinct MI350 added; AI vMotion (zero-downtime GPU migration) |
| Networking | EVPN foundation | EVPN with Arista, Cisco, SONiC; unified standards-based fabric |
| Object Storage | Not present | S3-compatible native object storage (tech preview) |
| Installer | Scatter planning across deployment | Consolidated Plan section; new deployment paths including VCF Mgmt Services deployment |

## **17.2 VCF 9.x Vendor Efficiency Claims (Internal Broadcom Estimates, April 2026\)**

| Metric | Claimed Improvement | Mechanism |
| :---- | :---- | :---- |
| Server Cost Reduction | Up to 40% | NVMe Memory Tiering increases VM density without hardware refresh |
| Storage TCO Reduction | Up to 39% | vSAN global deduplication and enhanced compression |
| Kubernetes Operational Cost | Up to 46% | VKS scale improvements, automated fleet operations |
| Cluster Upgrade Speed | 4x faster | 256 clusters in parallel (up from 64\) |
| VKS Deployment Speed | 69% faster | Linked clone technology (37 min to 11 min) |
| Infrastructure Management Time | 51% reduction | VCF Operations unified console (survey of 44 customers, March 2026\) |
| Monitoring Time | 46% reduction | Converged metrics, logs, and flows in single console |
| Capacity Requirements | 47% reduction | Advanced visibility and optimisation |
| Mean Time to Repair | 39% faster | Integrated diagnostic dashboards and root-cause analysis |
| vMotion CPU Overhead | Up to 70% reduction | Intel QAT hardware offload for encrypted vMotion |
| Fleet Management Capacity | 2x (to 5,000 hosts) | Automated fleet operations at scale |

| Note:  All figures are Broadcom internal estimates or survey results as of April/March 2026\. Real-world results will vary based on workload type, hardware configuration, and deployment complexity. Validate TCO claims with customer-specific sizing before presenting to C-suite. |
| :---- |

