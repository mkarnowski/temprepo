---
title: Cisco ACI with Avi Vantage Deployment Guide
layout: layout182
---

## Overview

### Cisco ACI

Cisco Application Centric Infrastructure is a software defined networking solution offered by Cisco for data centers and clouds, which helps in increasing operational efficiencies, delivering network automation, and improving security for any combination of on-premises data centers, private, and public clouds.

<a href="img/img1.jpg"> <img class="aligncenter" src="img/img1.jpg" alt="img1"></a>

The important building blocks of Cisco ACI are Nexus 9000 hardware and APIC.

The APIC provides centralized policy automation and management for ACI fabric. The Controller helps in common policy and management framework across physical, virtual, and cloud infrastructure.

ACI is based on open architecture (open APIs and standards) which integrates Layer 4-Layer 7 (L4-L7) services in the network. ACI solution offers robust implementation of multi-tenant security, quality of service (QoS), and high availability.

The following is a list of most used terminologies in ACI:

<table class="table table table-bordered table-hover"> <tbody>

<tr>
    <td><b>ACI fabric</b></td>
    <td>A Virtual Extensible LAN (VXLAN) overlay configured by APIC on leaf or spine switches to provide end-to-end connectivity for clients or servers.</td>
</tr>

<tr>
    <td><b>Bridge domains</b></td>
    <td>A bridge domain is a Layer 2 segment analogous to VLANs in a traditional network. </td>
</tr>

<tr>
    <td><b>Endpoint groups (EPGs)</b></td>
    <td>Endpoint groups are associated with endpoints in the network.
        The endpoints are identified by their domain connectivity (virtual, physical, or outside) and their connectivity method. For instance, virtual machine port groups (VLAN, VXLAN), physical interfaces or VLANs including virtual port channels, external VLANs, external subnets. </td>
</tr>

<tr>
    <td><b>Contracts</b></td>
    <td>These are directional access lists between the provider and consumer EPGs. They comprise of one or more filters (ACEs) to identify and allow traffic between EPGs. By default, communication between EPGs is blocked and it requires a contract to allow the traffic. <br>

   <b>Note</b>: Traffic within EPGs is allowed by default and so no contract is required.</td>
</tr>

<tr>
    <td><b>Application network profiles</b></td>
    <td>These are containers that group one or more EPGs together with their associated connectivity policies. </td>
</tr>

<tr>
    <td><b>L4-L7 service graph templates</b></td>
    <td>This is a generic representation of expected traffic flow in the network. These templates are reusable and can be used in multiple contracts.</td>
</tr>

<tr>
    <td><b>L4-L7 device</b></td>
    <td>
These are of two types:
<ol>
<li>Logical device &ndash; Represents a cluster of two devices that operate in active/standby mode. This is a logical representation of the physical/virtual device (load balancer) along with logical interfaces which implies the connectivity. </li>
<li>Concrete device &ndash; Represents a service device, such as, a virtual load balancer. In case of Avi Vantage, these are actual Service Engine VMs.</li>
</ol>
</td>
</tr>

<tr>
    <td><b>Tenants</b></td>
    <td>These are network wide administrative containers, which are like logical containers for application policies.</td>
</tr>

</tbody>
</table>


### Avi Vantage

The Avi Vantage Platform provides enterprise-grade distributed ADC and iWAF(Intelligent Web Application Firewall) solutions for on-premise and public cloud infrastructure. Avi Vantage also provides inbuilt analytics that enhances the end-user application experience as well as ease of operationalizing for network administrators.

Avi Vantage is a complete software solution which runs on commodity x86 servers or as a virtual machine and is entirely enabled by REST APIs.

The product components include:

* **Avi Controller (control plane):** Central policy and management plane that analyzes the real-time telemetry collected from Avi Service Engines and presents it in visual, actionable dashboards for administrators using an intuitive user interface built on RESTful APIs.

* **Avi Service Engines (data plane):** Distributed load balancers with iWAF that are deployed closest to the applications across multiple cloud infrastructures. The Avi Service Engines collect and send real-time application telemetry to the Avi Controller.

The architecture for Avi Vantage is Controller led, which de-couples the control plane and data plane. This architecture makes it possible to automate L4-L7 using Avi Controller and for ACI to provide L2-L3 network automation for Service Engines.

Below is the architectural representation and integration of Avi Vantage with ACI.

<a href="img/img2.png"> <img class="aligncenter" src="img/img2.png" alt="img2"></a>

### Software Requirements

 The following table lists the minimum software requirements for Avi Vantage integration with ACI:

<table class="table table table-bordered table-hover"><tbody>

<tr>
    <th>Component</th>
    <th>Version</th>
</tr>

<tr>
    <td>Avi Controller</td>
    <td>17.2.10 or later</td>
</tr>

<tr>
    <td>Cisco APIC</td>
    <td>1.03f or later</td>
</tr>

<tr>
    <td>VMware vCenter</td>
    <td>5.1, 5.5, 6.0, 6.5, or 6.7</td>
</tr>

</tbody>
</table>


## Integration Options for Avi Vantage in ACI Fabric

This section discusses integrating Avi Vantage with ACI in different hosting infrastructures.

**VMware**

1. **Service manager mode with REST API**<br>
    This is a hybrid integration mode in which ACI handles network automation for L4-L7 devices and the Avi Controller handles configuring the L4-L7 services.

2. **Network policy mode with write access vCenter cloud**<br>
    This is a traditional mode where Avi Vantage is not integrated with ACI. ACI is used only to provide access between the client network and virtual service network. Avi Vantage integrates with vCenter in write access mode.

3. **Network policy mode with no-access/read access vCenter cloud** <br>
    This is a traditional mode where Avi Vantage is not integrated with ACI. ACI is used only to provide access between the client network and virtual service network. Avi Vantage will be deployed as an external L3out on VMware infrastructure with no access cloud (no integration with vCenter) or read access (read only access to vcenter) and will perform the BGP peering with ACI.

**Cisco CSP 2100/ Bare Metal Servers**

1. **Network policy mode**<br>
    This is a traditional mode where Avi Vantage is not integrated with ACI. Avi Vantage will be deployed as an  external L3Out on Cisco CSP and will perform the BGP peering with ACI.


### Service Manager Mode via REST API

The service manager mode with REST API provides complete automation and flexibility to insert L4-L7 services with ease in ACI Fabric.

The main advantage of this mode is the end-to-end automation using Cisco ACI and Avi Vantage.
Below is the configuration workflow for Cisco ACI and Avi Vantage integration in service manager mode with REST API.

<a href="img/img23.jpg"> <img class="aligncenter" src="img/img23.jpg" alt="img23"></a>

The following three sections explain the detailed configuration workflow for this integration:

#### Day Zero Configuration

**Note:** This is a one time setup which requires vCenter deployment. Avi Controller should be configured with vCenter and APIC credentials under the cloud connector page.

1. Navigate to **Infrastructure** > **Cloud** and create a new cloud or use the default cloud based on your requirement.

    <a href="img/img24.png"> <img class="aligncenter" src="img/img24.png" alt="img24"></a>

    **Note:** The SE Tunnel Mode is enabled by default and is specifically useful in POCs. We highly recommend disabling this for any production deployment.

2. Click on **Next** and select the Data Center. If the virtual service network is not the directly connected network, then select the checkbox for **Prefer Static Routes vs Directly Connected Network** and use static routes for VIP’s network resolution.

    <a href="img/img25.png"> <img class="aligncenter" src="img/img25.png" alt="img25"></a>

3. Click on **Next** and select the management network for SE interfaces. For static address management instead of DHCP server, add the static address pool.

    <a href="img/img26.png"> <img class="aligncenter" src="img/img26.png" alt="img26"></a>

    **Note:** The networks displayed in the above screenshot are the port-groups which got imported from the vCenter. This list is only used for management network selection.

4. Avi Controller creates the L4-L7 device under the tenant which is mentioned in the cloud connector page. This L4-L7 device can be exported to other tenants for service graph creation in other tenants.

    <a href="img/img27.png"> <img class="aligncenter" src="img/img27.png" alt="img27"></a>

    Starting with Avi Vantage release 17.2.10, Avi Controller by default creates two automated service graphs, *AviLayer2Graph* and *AviLayer3Graph*. <br>

    ***AviLayer2Graph*** &ndash; This graph can be used for any east-west app load balancing or any two-arm mode deployments. The graph will have connectors with L2 adjacency type, so that the interfaces can be a part of the bridge domains of east-west app. <br>

    ***AviLayer3Graph*** &ndash; This graph can be used for any north-south app load balancing or one-arm mode deployment, where the traffic is sourced from external L3Out branch networks. The graph will have connectors with L3 adjacency type, so that the interfaces can be associated with L3Out external EPGs.<br>

    You can choose one of these automated service graphs or create a manual service graph to include other L4-L7 devices such as firewall along with load balancer in service graph. <br>

    Create a L4-L7 service graph manually with one node cluster using the L4-L7 device which was created in the earlier step. Navigate to **Tenant** > **L4-L7 Service Graph** and choose **Create a New Graph** with a single node cluster.

    <a href="img/img28.png"> <img class="aligncenter" src="img/img28.png" alt="img28"></a>

    **Note:** Use "Avi" as the naming convention for the node in the service graph. These keywords are case sensitive.

     After creating the service graph and associating it with contracts and EPGs, proceed with the virtual service provisioning. For more details on associating contracts and EPGs, refer to [East-West]({%vpath%}/cisco-aci-deployment-guide/#avi-vantage-deployment-for-east---west-traffic) and [North-South]({%vpath%}/cisco-aci-deployment-guide/#avi-vantage-deployment-for-north-south-traffic) deployment sections based on the use case.

#### Network Provisioning in Avi Controller

 When used in conjunction with ACI, the Avi Controller can only assign static IP addresses to the Service Engine interfaces. The bridge domains which are created in APIC for a particular tenant gets imported as network entities from APIC to the Avi Controller.

Define a pool of IP addresses for each bridge domain. This pool range is used by the Avi Controller to assign IP addresses to the interfaces of the SE that connect to these bridge domains.

Navigate to **Infrastructure** > **Networks** and select the cloud created. Edit the bridge domain networks that were imported from the ACI to add the IP address pool.

 <a href="img/img29.png"> <img class="aligncenter" src="img/img29.png" alt="img29"></a>

Repeat the steps for other bridge domains.


#### Virtual Service Provisioning

1. After completing the Day-0 configuration, APIC instantiates service graphs based on the contracts to which the graph templates have been associated.

2. The deployed service graph instance from APIC is imported to the Avi Controller automatically.

3.  Create the virtual service. For the virtual service name, click on the drop-down and select the deployed service graph instance which was imported to Avi Controller and for pools select the EPGs configured for the servers.

    <a href="img/img30.png"> <img class="aligncenter" src="img/img30.png" alt="img30"></a>

4. Creating virtual service will trigger the Service Engine creation in vCenter and will add the SEs to APIC as L4-L7 concrete device. Once the Service Engines are deployed a new virtual service will be hosted on the same Service Engine depending on the Service Engine group properties.

5. Device or interface mapping and network stitching will be done automatically by APIC and no user intervention is required.

6. After the SEs are created, the virtual service will be ready to accept the traffic. The Avi SE will be deployed in GoTo mode (routed mode) and Avi SEs will perform the source NAT, by default.

    <a href="img/img31.png"> <img class="aligncenter" src="img/img31.png" alt="img31"></a>

    **Note:** Prior to Avi Vantage release 18.1.2, each virtual service would need a contract with the associated service graph. For instance, creating 10 virtual services would require 10 contracts associated with the service graph. So, to create a virtual service, you need to create a service graph template and associate it with all the contracts. <br>
    Starting with Avi Vantage release 18.1.2, multiple virtual services can share the same service graph in ACI.

    <a href="img/img32.jpg"> <img class="aligncenter" src="img/img32.jpg" alt="img32"></a>


The REST API communication workflow is as follows:

   1. Avi Vantage uses Rest API to get tenant details for creating a logical device.
   2. Once the tenant is chosen, Avi Vantage creates a L4-L7 device in the specific tenant.
   3. Automated *AviLayer2* and *AviLayer3* service graphs are created by default and if required manual service graph can be created in APIC along with the contract assignment for the EPGs. The APIC will create a deployed service graph instance which will be provided to Avi Vantage. This instance will be used for virtual service creation.
   4. APIC will sync the configured EPGs to the Avi Controller and vCenter.
   5. After creating the VIP, Avi Controller will create the SEs and register it with APIC as a concrete device.
   6. APIC will map this device to logical device context and map the interfaces between the logical interfaces and SE vNICs.
   7. APIC will interact with the VMM domain and create the dynamic port groups which will be mapped to the SE interfaces of VMware domain.

**Note:** This is the most recommended mode for any deployment as this allows service graph template customization by adding firewall, IDS, etc., along with Avi Vantage for the traffic flow.

### Network Policy Mode with Avi Vantage on Write Access VMware Cloud

In this mode, Avi Vantage will not be integrated with APIC. Rather the APIC will be integrated with VMware and the VMware infrastructure is used to configure the interfaces and port groups.

<a href="img/img33.jpg"> <img class="aligncenter" src="img/img33.jpg" alt="img33"></a>

As seen above, the integration is with vCenter in write access mode. Given below is the configuration workflow for this mode.

<a href="img/img34.jpg"> <img class="aligncenter" src="img/img34.jpg" alt="img34"></a>

To deploy Avi Vantage in vCenter with write access mode, refer to [Installing Avi Vantage for VMware vCenter.]({%vpath%}/installing-avi-vantage-for-vmware-vcenter/)

This is a traditional deployment where ACI provides access (contracts) between the clients and virtual service. ACI will not provide any L2-L3 automation in this case.

#### Configuring APIC contracts for Avi Vantage

   After deploying Avi Vantage in vCenter write access mode, you need to create contracts to allow communication between the clients and virtual service networks.

   The contracts can be configured in ACI for the following deployment modes:

   **Avi Vantage Deployed in Two-Arm Mode**<br>
     In this mode, the clients and servers are in different networks. Avi Vantage will host virtual services in a different network. You need to create a contract to allow communication between the client EPG and virtual servers EPG. There is no contract required between Avi Vantage and the server EPG, if Avi Vantage does not have an interface in the server EPG network.

   **Avi Vantage Deployed in One Arm Mode**<br>
    In this mode, the clients, servers, and Avi load balancer are in the same network. There is no need for any contract, as all intra communication within an EPG is allowed by default. However, if you have Avi Vantage only in the client network without any interface in the server network, then you need a contract between the client EPGs and server EPGs.

### Network Policy Mode with Avi Vantage on No Access or Read Access VMware Cloud

In this deployment, BGP is used for peering with ACI fabric and to exchange the virtual service routes. This deployment is mostly applicable for setups with no vCenter deployed or have no vCenter integration with Avi Controller.

In vCenter no-access cloud, the Avi Controller has no access to vCenter resources. In vCenter read access cloud, the Avi Controller will integrate with vCenter, but the Controller can only read the vCenter resources and not write or create any Service Engines or resources. In both cases, you need to manually deploy Service Engines.

For more information on VMware no-access or read access, please refer to [Deploying in Read/No access mode]({%vpath%}/installing-avi-vantage-for-vmware-vcenter/#deploying-in-read--no-access-mode).

To support BGP peering between the Avi Service Engines hosted on vCenter and ACI fabric, ensure that there is no VMM domain integration between the vSwitch used for Service Engines and the ESXi host ports that are assigned to the Service Engines. The SEs should be connected to the ACI fabric and configured as external routed device in APIC. Then, APIC will considered it as a L3Out device.


#### BGP Peering

For configuring BGP L3Out on Cisco APIC, please refer to the APIC configuration section in [Avi Vantage Deployment for North-South Traffic.]({%vpath%}/cisco-aci-deployment-guide/#avi-vantage-deployment-for-north-south-traffic)

To configure BGP on Avi Controller, navigate to **Infrastructure** > **Routing**, and select the cloud in which BGP peering is configured. Under **BGP Peering**, enter the peer IP, AS numbers, and other parameters.

<a href="img/img35.png"> <img class="aligncenter" src="img/img35.png" alt="img35"></a>

 For complete information on BGP on Avi Controller, refer to [BGP Support for Scaling Virtual Services.]({%vpath%}/bgp-support-for-virtual-services/)

#### Virtual Service Provisioning

After configuring BGP peering, proceed with creating virtual services.

In Avi Controller, click on creating virtual service (Advanced) option. Enter all virtual service parameters, such as, the name, IP address, pool, etc. Click on **Next** and navigate to the **Advanced** tab. Enable **Advertise VIP via BGP** option.

<a href="img/img36.png"> <img class="aligncenter" src="img/img36.png" alt="img36"></a>

<a href="img/img37.png"> <img class="aligncenter" src="img/img37.png" alt="img37"></a>

After the virtual service is created, the Avi Service Engine will start publishing the routes to ACI fabric.

You can check the BGP peering status and advertised routes in APIC. Alternatively, to check BGP peering and neighbour status on Avi Vantage, login to the SE CLI and run the commands shown below under quagga router. Refer to [How to Access and Use Quagga Shell using Avi CLI]({%vpath%}/how-to-access-and-use-quagga-shell/) for reference.


<pre class="command-line language-bash" dataprompt="$" data-output="2-99"> <code>

10-128-3-198> show ip bgp
BGP table version is 0, local router ID is 2.103.143.46
Status codes: s suppressed, d damped, h history, * valid, > best, = multipath,
              i internal, r RIB-failure, S Stale, R Removed
Origin codes: i - IGP, e - EGP, ? - incomplete

Network          Next Hop            Metric LocPrf Weight Path
*> 15.15.15.5/32    0.0.0.0                  0         32768 i
*> 16.16.16.0/24    17.17.17.1               0             0 500 670 ?

Total number of prefixes 2
</code></pre>

<pre class="command-line language-bash" dataprompt="$" data-output="2-99"> <code>
10-128-3-198> show ip bgp summary
BGP router identifier 2.103.143.46, local AS number 600
RIB entries 3, using 336 bytes of memory
Peers 1, using 4568 bytes of memory

Neighbor        V         AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
17.17.17.1      4   500   19369   19373        0    0    0 01w6d10h        1

Total number of neighbors 1
</code></pre>

You can verify the virtual service routes in ACI fabric. The sample below is the output of virtual service route learned by a border leaf in ACI fabric.

<pre class="command-line language-bash" dataprompt="$" data-output="2-99"> 
<code>
leaf-1# show bgp ipv4 unicast vrf VMware-NO-Access-Demo:vmware-no-vrf
BGP routing table information for VRF VMware-NO-Access-Demo:vmware-no-vrf, address family IPv4 Unicast
BGP table version is 12, local router ID is 4.4.4.4
Status: s-suppressed, x-deleted, S-stale, d-dampened, h-history, *-valid, >-best
Path type: i-internal, e-external, c-confed, l-local, a-aggregate, r-redist, I-injected
Origin codes: i - IGP, e - EGP, ? - incomplete, | - multipath, & - backup

Network            Next Hop            Metric     LocPrf     Weight Path
*>r4.4.4.4/32         0.0.0.0                  0        100      32768 ?
*>e15.15.15.5/32      17.17.17.2               0         0       500  600
*>r16.16.16.0/24      0.0.0.0                  0        100      32768 ?
*>r17.17.17.0/24      0.0.0.0                  0        100      32768 ?

</code>
</pre>

### Network Policy Mode with Avi Vantage in Write Access VMware Cloud with BGP L3_Out

In this deployment, BGP is used for peering with ACI fabric and to exchange the virtual service routes. This deployment is mostly applicable for setups where BGP auto scaling is required for virtual service scaling on Avi Service Engines.

In vCenter write access mode, the Avi Controller is configured with vCenter cloud connector. The Controller has write access permissions to vCenter and handles complete automation involved in creating Service Engines and placing them in the network. The Controller also scales the Service Engines based on the configured threshold.

For more information, refer to [Installing Avi Vantage for VMware vCenter]({%vpath%}/installing-avi-vantage-for-vmware-vcenter/#deploying-in-write-access-mode).

The BGP peering and virtual service configuration remains the same as mentioned in the [Network Policy Mode with Avi Vantage on No Access or Read Access VMware Cloud]({%vpath%}/cisco-aci-deployment-guide/#network-policy-mode-with-avi-vantage-on-no-access-or-read-access-vmware-cloud) section.

Both APIC managed virtual distributed switch and non-APIC virtual distributed switch are supported. For APIC managed switch, you need to create a static port group with static VLAN in vCenter for the Service Engine Data vNICs. Also, add the static VLAN pool in the VMM domain VLAN pool in APIC. Use the SVI interface with static VLAN as L3out in the ACI fabric.



## Service Manager Mode via REST API Integration Use Cases

The figure below represents the complete application traffic flow in ACI fabric along with Avi Vantage load balancers:

<a href="img/img3.png"> <img class="aligncenter" src="img/img3.png" alt="img3"></a>

This figure depicts the application traffic flow which includes the north-south traffic flow from clients to virtual services and also the east-west traffic flow for internal application communication.

This section discusses the following two use cases for Avi Vantage deployment in ACI fabric:

1. [Avi Vantage deployment for east-west traffic]({%vpath%}/cisco-aci-deployment-guide/#avi-vantage-deployment-for-east-west-traffic)
2. [Avi Vantage deployment for north-south traffic]({%vpath%}/cisco-aci-deployment-guide/#avi-vantage-deployment-for-north-south-traffic)

In both these cases, Avi Vantage will be deployed in the GoTo mode and a service graph must be created as mentioned in the [Service manager mode with REST API]({%vpath%}/cisco-aci-deployment-guide/#service-manager-mode-via-rest-api) section. The service graphs will be same in both the deployments. You can even use a single service graph template for east-west traffic as well as for the north-south traffic deployment.

### Avi Vantage Deployment for East - West Traffic

Assuming that the service graph has been created as mentioned in [Service manager mode with REST API]({%vpath%}/cisco-aci-deployment-guide/#service-manager-mode-via-rest-api), this section explains the configuration for Cisco ACI EPG and contracts for east-west traffic.

The east-west traffic would generally be server-to-server traffic and would mostly be from one VM to another. So, bridge domains with VMM attachments for EPGs are used.

<a href="img/img4.png"> <img class="aligncenter" src="img/img4.png" alt="img4"></a>

The most common deployment for any east-west traffic is a 3-tier architecture, as represented above and the naming convention is used for different objects in the configuration steps further.

#### APIC Configuration

The following is an example to configure ACI in east-west traffic. For more details, refer to ACI fundamentals at [Cisco Application Centric Infrastructure Fundamentals.](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/1-x/aci-fundamentals/b_ACI-Fundamentals.html)

1. Under **Tenants**, navigate to the configured tenant. For creating an isolated network for this traffic, navigate to **Networking** > **VRF** > **Create VRF**.

2. Navigate to **Bridge Domains** and create a bridge domain with the name *BD1* for the web subnet. Select the VRF which was created earlier and add the subnet for which the ACI will create SVI interfaces used as gateway for servers.

    <a href="img/img5.jpg"> <img class="aligncenter" src="img/img5.jpg" alt="img5"></a>

    Follow the exact steps to create *BD2* and *BD3* bridge domains for app and database subnets.

3. After creating the bridge domains, navigate to **Application Profile** and create an application profile *3-Tier-App*. <br>

4. Navigate to *3-Tier-App* > **Application EPGs** and create EPGs for web, app, and database.

    <a href="img/img6.jpg"> <img class="aligncenter" src="img/img6.jpg" alt="img6"></a>

    **Note:** Ensure that you select the checkbox for **Associate to VM Domain Profiles** and click on **Next** to add your VMM domain to this EPG, which communicates with vCenter and creates the port-groups.

5. Navigate to **Security Policy** > **Contracts** and click on **Create Contract**. Enter the *Contract Name* and add the contract subject with the filter and service graph.

    <a href="img/img7.png"> <img class="aligncenter" src="img/img7.png" alt="img7"></a>

6. Create two contracts, one for WEB-APP traffic load balancing and another for APP-DB traffic load balancing. Add a filter and service graph to both the contracts.

7. After creating the contracts, associate the contract with the EPGs.

8. Navigate to **Application Profile *3-Tier-APP*** > Application EPGs and select WEB-EPG. Navigate to **Contracts**, add consumed contract, and select the contract which was created earlier for WEB-APP communication.

    <a href="img/img8.png"> <img class="aligncenter" src="img/img8.png" alt="img8"></a>

    Follow the exact steps for other EPGs and associate contracts accordingly.

For WEB-APP traffic load balancing, WEB-EPG would be the consumed contract and APP-EPG would be the provider contract. Similarly, for APP-DB traffic flow, APP-EPG would be the consumed contract and DB-EPG would be the provider contract.

In terms of ACI, the consumed contract would be analogous to client traffic and provider contract would be analogous to server contract. In this case, the WEB-EPG is like the client EPG which consumes resources from APP-EPG which is like the server EPG providing resources.

#### Avi Vantage Configuration

After the contracts are associated with the EPGs, you should see the deployed graph instance. Refer to the *Virtual Service Provisioning* section under [VMware Service Manager Mode via REST API]({%vpath%}/cisco-aci-deployment-guide/#service-manager-mode-via-rest-api) to configure the virtual service. The Service Engines will be mapped with the logical device under L4-L7 devices, as shown in the screenshot below.

<a href="img/img9.png"> <img class="aligncenter" src="img/img9.png" alt="img9"></a>

As shown, the cluster instances also get mapped to the Service Engine's interfaces.

**Note:** The automatic port-group mapping on vCenter for Avi SEs would usually take about 2-3 minutes. During this time, it is expected for the pool members to be down. After the port-groups are assigned, you should see that the pool member status will be up and green.

### Avi Vantage Deployment for North-South Traffic

Assuming that the service graph has been created as mentioned in the [Service manager with REST API mode]({%vpath%}/cisco-aci-deployment-guide/#service-manager-mode-via-rest-api), this section explains the configuration for Cisco ACI EPG and contracts for north-south traffic.

The north-south traffic is between the clients and servers. The clients could be directly connected to the fabric or to the external WAN using L3Out on ACI. The servers are virtual services in this deployment.

<a href="img/img10.png"> <img class="aligncenter" src="img/img10.png" alt="img10"></a>

#### APIC Configuration

The following is an example to configure ACI in north-south traffic. For more details, refer to ACI fundamentals at [Cisco Application Centric Infrastructure Fundamentals.](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/1-x/aci-fundamentals/b_ACI-Fundamentals.html)

Most of the configuration steps mentioned for east-west traffic is applicable for north-south traffic deployments. The one section differing is the L3Out client network. The exact same naming conventions are used in this deployment as well.

* In cases, where the clients are not directly connected to the ACI and reaches the ACI fabric using the WAN link from branches, you need to configure L3out on ACI for clients to access the servers behind the ACI.

* For L3Outs, you can use dynamic routing protocol or static routing depending on the WAN connectivity.

* The VS-APP bridge domain is the virtual service network which are virtual services network hosted on Service Engines.

1. Configure MP-BGP in ACI fabric and attach the AAEP policy for the physical domain connectivity. Refer to the Cisco documentation at the links below for complete configuration details.

    * Configuring MP-BGP in ACI &ndash; [Fabric Setup for External Network Peering](https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/guide-c07-733236.html#_Toc406851689)
    * Attaching an external device to ACI &ndash; [Creating Domains, Attach Entity Profiles, and VLANs to Deploy an EPG on a Specific Port](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/kb/b_KB_Creating_AEP_Physical_Domains_VLANS_to_Deploy_an_EPG_on_a_Specific_Port.html#concept_F3C24AA0EA114A9784C4D722190E8C2D)


2. To configure the external routed network, navigate to **Networking** > **External Routed Network** > **Create Routed Outside**. Provide a name for L3Out and add a node profile and interface profile. Add external networks (also referred as external EPGs) which can be 0.0.0.0/0 to accept all routes from the external router. Add the needed subnets, if you want to allow specific routes.

    <a href="img/img11.jpg"> <img class="aligncenter" src="img/img11.jpg" alt="img11"></a>

    Add this to the VRF in which the VS-App bridge domain is created.

    <a href="img/img12.png"> <img class="aligncenter" src="img/img12.png" alt="img12"></a>

3. For VS-App bridge domain, associate the L3Out. If you are publishing this subnet routes to the external router, ensure that the subnets under the bridge domains are set to public.

   <a href="img/img13.jpg"> <img class="aligncenter" src="img/img13.jpg" alt="img13"></a>

4. Once the external connectivity is established, you can check whether the routes are populated under **Inventory** > **Pod** > **Node** > **Protocols**.

5. Create a contract along with the service graph, similar to the east-west contract. The change is with adding external networks to access virtual service. Ensure that the contract between the external network and the VS-APP bridge domain is present along with the service graph.

    For the contract created with service graph, assign the consumed contract role to the external network and the provider contract role to VS-APP EPG, so that the communication is allowed.

    <a href="img/img14.png"> <img class="aligncenter" src="img/img14.png" alt="img14"></a>

#### Avi Vantage Configuration

After the contracts are associated with the EPGs, you should see the deployed graph instance. Refer to the *Virtual Service Provisioning* section under [VMware Service Manager Mode via REST API]({%vpath%}/cisco-aci-deployment-guide/#service-manager-mode-via-rest-api) to configure the virtual service.
 The Service Engines will be mapped to the logical device under L4-L7 devices.

Now the external clients should be able to access the virtual service which is hosted on Avi Service Engines.

## Avi Vantage Deployment for APIC Multi Tenancy

This is a specific use case for the service provider or managed hosting profiles, who would like to deploy Avi Vantage for each APIC tenant, in which each tenant represents a user/customer managed by the provider/hosting profile.

In this deployment, Avi Vantage will be integrated in APIC common tenant and VRFs are used in Avi Vantage to isolate the traffic.

The following are the two types of Avi Vantage deployment modes for multi-tenancy:

1. **Tenant Context Mode** &ndash; Service Engine is deployed for each tenant and each tenant also has a specific VRF for isolating traffic.

    <a href="img/img15.png"> <img class="aligncenter" src="img/img15.png" alt="img15"></a>

2. **Provider Context Mode** &ndash; Service Engine is shared across multiple tenants and VRFs on these Service Engines are used to isolate the traffic.

    <a href="img/img16.png"> <img class="aligncenter" src="img/img16.png" alt="img16"></a>


Tenants can be deployed within a provider context or a tenant context. For tenant settings, navigate to **Administration** > **Settings** > **Tenant Settings**.

<a href="img/img17.png"> <img class="aligncenter" src="img/img17.png" alt="img17"></a>

For both modes, the APIC configuration would remain the same. The following are the sample steps for deploying Avi Vantage in multi tenancy in ACI fabric:

1. Integrate Avi Vantage with APIC in common tenant and export the L4-L7 device to the required tenant.

    <a href="img/img18.png"> <img class="aligncenter" src="img/img18.png" alt="img18"></a>

2. After the integration, export the L4-L7 device in APIC common tenant to the tenant of interest in APIC. In APIC, navigate to **Common Tenant** > **Services** > **L4-L7** > **Devices**. Here you will notice a L4-L7 device created by the Avi Controller. Select the device, left click, and select the option **Export L4-L7 Device**.

    <a href="img/img19.png"> <img class="aligncenter" src="img/img19.png" alt="img19"></a>

3. After the device is exported to the tenant of interest, create the service graph as explained in [Service manager mode with REST API]({%vpath%}/cisco-aci-deployment-guide/#service-manager-mode-via-rest-api) section.

4. VRFs in Avi Vantage are used to isolate the traffic from one tenant to another tenant. To associate the networks with VRFs, create the VRFs. In Avi Controller admin tenant, navigate to **Infrastructure** > **Routing** > **VRF Context** and create a new VRF context.

    <a href="img/img20.png"> <img class="aligncenter" src="img/img20.png" alt="img20"></a>

5. Navigate to **Infrastructure** > **Networks**, to see the bridge domains that are synced from the APIC in which the L4-L7 device is exported.

    <a href="img/img21.png"> <img class="aligncenter" src="img/img21.png" alt="img21"></a>

6. Move the networks to the appropriate VRFs and configure a static pool for IP address assignment for Service Engines.

7. Navigate to **Infrastructure** > **Networks** and select the cloud created. Edit the imported bridge domain networks, to add the IP address pool and select the VRF context.

    <a href="img/img22.png"> <img class="aligncenter" src="img/img22.png" alt="img22"></a>

    Repeat the steps for other bridge domains.



## Monitoring

 Avi Controller provides real time analytics dashboards which provide application load balancing and security analytics in a single frame.

 <a href="img/img38.png"> <img class="aligncenter" src="img/img38.png" alt="img38"></a>

 Avi Vantage virtual service real time metrics provides details relating to transactions per second, delay, response times, etc.

<a href="img/img39.png"> <img class="aligncenter" src="img/img39.png" alt="img39"></a>

Avi Vantage logs provide detailed view of each connection, as in the displayed case, where the client/virtual service/server end-to-end communication is displayed, which is used for troubleshooting.

<a href="img/img40.png"> <img class="aligncenter" src="img/img40.png" alt="img40"></a>

Avi Vantage WAF analytics provides information on real time web security attacks on the virtual service.

The Avi Controller offers the following monitoring capabilities for Cisco ACI fabric:

* Monitors load balancer (SE) and application server health.
* Provides real-time application analytics.
* Protects applications against L4-L7 DDoS attacks.
* Monitors APIC EPG membership to automatically add or remove application instances from pools.
* Performs load balancer auto scaling based on real-time performance metrics (CPU, memory, bandwidth, connections, latency, etc).
* Provides point-and-click simplicity for iWAF policies with central control.
* Provides granular security insights on traffic flows and rule matches to enable precise policies using iWAF.

## Troubleshooting

The following are a few commonly noticed issues in Avi Vantage integration with ACI and the recommended solution:

1. Deployed service graph instance is not getting created in ACI for the created service graph or the Avi Controller is not getting imported with the deployed service graph instance from ACI.<br>

    *Solution:* The possible cause for this issue could be the naming used for cluster node in the service graph. Ensure that the cluster node is named as “Avi” in the service graph.

2. LIf to CIf invalid mapping errors and Cdev config errors in ACI. <br>

    *Solution:* These are temporary error seen as it takes about 2-3 minutes for the dynamic port group mapping in vCenter and another 2 minutes for the Service Engine to spin up. If you are still noticing this error, check the communication between the ACI and the vCenter.

   For complete details on integration issues and verification steps for the L4-L7 error messages, refer to [Troubleshooting Cisco Application Centric Infrastructure](https://www.cisco.com/c/dam/en/us/td/docs/switches/datacenter/aci/apic/sw/1-x/Troubleshooting_ACI/Cisco_TroubleshootingApplicationCentricInfrastructure.pdf).

3.  Dynamic port-groups are not getting created on vCenter and SEs are not able to communicate with pools and clients. <br>

    *Solution:* This situation can occur if the VMM domain association is not done for the client/server EPGs. After creating VMM, only the dynamic port-groups associations are created by the APIC on vCenter. These need to be assigned to the clients/servers in vCenter. Ensure that the VMM domains are assigned to the EPGs, as well.

4. Pool servers or virtual services are down<br>

    *Solution:* This could happen if no static address pool allocation is configured for SE interfaces. The virtual service network needs the static address pool and pool network allocation, so that the SE interfaces will get assigned to the network address.

5. Verify communication issues between the ACI and Avi Vantage. <br>
Use the `apic_agent` log file output to determine any communication issues, as shown in the output below.

    <pre class="command-line language-bash" dataprompt="$" data-output="2-99"> <code>
    admin@Demo-Controller-17:/$ more /opt/avi/log/apic_agent.log

    [2017-11-02 08:55:55,712] INFO [apic_agent._get_apic_tenants:2573] [u'common', u'MTenant_Base1', u'CiscoScale', u'Demo', u'ApicScale', u'MTenant_2', u'vCenter-with-Avi', u'infra', u'Pavan', u'MTenant_Base', u'MTenant_1', u'ApicL3', u'miska', u'Development', u'Training', u'PJ', u'Abhinav_A1', u'jason', u'Pavan-unmanaged', u'mgmt', u'UI_Test_Tenant', u'ApicVipShare', u'DemoCommon', u'TME_India']
    [2017-11-02 08:55:56,988] WARNING [apic_agent._apic_refresh_helper:1262] LdevIf subscriptionRefresh failed status 403 rsp {"totalCount":"1","imdata":[{"error":{"attributes":{"code":"403","text":"Need a valid webtoken cookie (named APIC-Cookie) or a signed request with signature in the cookie APIC-Request-Signature for all REST API requests"}}}]}


    [2017-11-02 08:56:15,928] INFO [apic_agent.Update:288] Updating APIC configuration
    [2017-11-02 08:56:15,929] INFO [apic_agent.Update:289] uuid: "<APICCONFIG>"
    obj_type: APICCONFIGURATION
    resource {
      cloud {
        uuid: "cloud-a5f36abc-82b2-449f-bde2-f7c53f826c1d"
        name: "Default-Cloud"
        vtype: CLOUD_VCENTER
        vcenter_configuration {
          username: "root"
          password: "vmware"
          vcenter_url: "10.128.3.200"
          privilege: WRITE_ACCESS
          datacenter: "Apic"
          management_network: "network-11-cloud-a5f36abc-82b2-449f-bde2-f7c53f826c1d"
          management_ip_subnet {
            ip_addr {
              addr: "10.128.3.0"
              type: V4
            }
            mask: 24
          }
        }
    </code> </pre>

    Even after following the above suggested steps if there are still some issue with integration then please contact Avi support for further troubleshooting at  avinetworks-support@avinetworks.com

## Resources

1. [Avi Knowledge Base](https://kb.avinetworks.com/)
2. [Cisco ACI Reference Guide](https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/white-paper-c11-737909.html)
3. [Avi Vantage with Cisco ACI Solution Brief](http://info.avinetworks.com/solution-brief-cisco-aci-integration)
4. [Cisco ACI white paper](https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/white-paper-c11-737361.html)
5. [Cisco ACI Service Graph deployment white paper](https://www.cisco.com/c/en/us/solutions/collateral/data-center-virtualization/application-centric-infrastructure/white-paper-c11-734298.html)
6. [Avi Vantage with Cisco ACI Demo Video](https://www.youtube.com/watch?v=xfQ_McCikJQ&t=2s)
