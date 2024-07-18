# KQL-SECURITY-QUERRIES
<center>
<a target="_blank" href="Microsoft" title="Microsoft"><img src="https://img.shields.io/static/v1?label=Product&message=Microsoft&color=red"></a>
<a target="_blank" href="Sentinel" title="Sentinel"><img src="https://img.shields.io/static/v1?label=SIEM&message=Sentinel&color=blue"></a>
<a target="_blank" href="Defender" title="Defender"><img src="https://img.shields.io/static/v1?label=AntiVirus&message=Defender&color=green"></a>
<a target="_blank" href="Author" title="cybereagle2001"><img src="https://img.shields.io/static/v1?label=Author&message=cybereagle2001&color=yellow"></a>

</center>    

![image](https://github.com/user-attachments/assets/6a3a1cb2-60ea-4173-a3c4-de2968078c23)


This repository, created by @cybereagle2001 (Oussama Ben Hadj Dahman), a cybersecurity expert and researcher, aims to centralize useful KQL (Kusto Query Language) queries. These queries are designed to assist cybersecurity professionals in their daily tasks, making their work more efficient and effective.

<center>

## Table of Contents

### 1. Microsoft Sentinel Queries
   - [Extracting High Severity Alerts from Microsoft Defender](#extracting-high-severity-alerts-from-microsoft-defender)
   - [Enhancing Investigation with Alert Links](#enhancing-investigation-with-alert-links)
   - [Extracting and Highlighting Affected Accounts, IPs, and Hosts](#extracting-and-highlighting-affected-accounts-ips-and-hosts)
   - [Identifying All Possible Threats on Devices and Servers](#identifying-all-possible-threats-on-devices-and-servers)
   - [Retrieving Alerts Related to Actual Incidents](#retrieving-alerts-related-to-actual-incidents)
   - [Visualizing Incidents by MITRE ATT&CK Tactics](#visualizing-incidents-by-mitre-attck-tactics)

### 2. Microsoft Security Queries
   - [Advanced Threat Hunt](#advanced-threat-hunt)
   - [Detailed Threat Hunt](#detailed-threat-hunt)

</center>

### Microsoft Sentinel Queries

Microsoft Sentinel is a renowned Security Information and Event Management (SIEM) solution in modern cybersecurity, offered as a service by Microsoft. It enables comprehensive data visualization and analytics through specific workbooks activated based on each connector. Additionally, it triggers relevant logs for in-depth monitoring and investigation.

#### Extracting High Severity Alerts from Microsoft Defender

This query retrieves alerts generated by Microsoft Defender with high severity:

```kql
SecurityAlert
| where ProviderName contains "MD" and AlertSeverity contains "high"
| project AlertName, AlertSeverity, Description, ProviderName, AlertType, Entities, Status, Techniques
```

#### Enhancing Investigation with Alert Links

To facilitate better investigations, we can add the column storing the alert link:

```kql
SecurityAlert
| where ProviderName contains "MD" and AlertSeverity contains "high"
| project AlertName, AlertSeverity, Description, ProviderName, AlertType, Entities, Status, Techniques, AlertLink
```

#### Extracting and Highlighting Affected Accounts, IPs, and Hosts

This query extracts all alerts generated by Windows Defender and highlights the affected accounts, IPs, and hostnames for each alert:

```kql
SecurityAlert
| where ProviderName contains "MD"
| extend EntitiesDynamicArray = parse_json(Entities)
| mv-expand EntitiesDynamicArray
| extend EntityType = tostring(parse_json(EntitiesDynamicArray).Type), EntityAddress = tostring(EntitiesDynamicArray.Address), EntityHostName = tostring(EntitiesDynamicArray.HostName), EntityAccountName = tostring(EntitiesDynamicArray.Name)
| extend HostName = iif(EntityType == 'host', EntityHostName, '')
| extend IPAddress = iif(EntityType == 'ip', EntityAddress, '')
| extend Account = iif(EntityType == 'account', EntityAccountName, '')
| where isnotempty(IPAddress) or isnotempty(Account) or isnotempty(HostName)
| summarize AccountList = make_set(Account), IPList = make_set(IPAddress), HostList = make_set(HostName) by TimeGenerated, AlertName, AlertSeverity, Description, AlertType, Status, Techniques
```

#### Identifying All Possible Threats on Devices and Servers

This comprehensive query helps identify all potential threats on devices and servers for mitigation:

```kql
SecurityAlert
| where ProviderName contains "MD" and Status != "Resolved"
| extend EntitiesDynamicArray = parse_json(Entities)
| mv-expand EntitiesDynamicArray
| extend EntityType = tostring(parse_json(EntitiesDynamicArray).Type), EntityAddress = tostring(EntitiesDynamicArray.Address), EntityHostName = tostring(EntitiesDynamicArray.HostName), EntityAccountName = tostring(EntitiesDynamicArray.Name)
| extend HostName = iif(EntityType == 'host', EntityHostName, '')
| extend IPAddress = iif(EntityType == 'ip', EntityAddress, '')
| extend Account = iif(EntityType == 'account', EntityAccountName, '')
| where isnotempty(IPAddress) or isnotempty(Account) or isnotempty(HostName)
| summarize AccountList = make_set(Account), HostList = make_set(HostName) by TimeGenerated, AlertName, AlertSeverity, Description, AlertType, Status, Techniques
```

#### Retrieving Alerts Related to Actual Incidents

To retrieve only alerts that are related to actual incidents or have been turned into incidents, use the following query:

```kql
SecurityAlert
| where ProviderName contains "MD" and Status != "Resolved" and IsIncident == True
| extend EntitiesDynamicArray = parse_json(Entities)
| mv-expand EntitiesDynamicArray
| extend EntityType = tostring(parse_json(EntitiesDynamicArray).Type), EntityAddress = tostring(EntitiesDynamicArray.Address), EntityHostName = tostring(EntitiesDynamicArray.HostName), EntityAccountName = tostring(EntitiesDynamicArray.Name)
| extend HostName = iif(EntityType == 'host', EntityHostName, '')
| extend IPAddress = iif(EntityType == 'ip', EntityAddress, '')
| extend Account = iif(EntityType == 'account', EntityAccountName, '')
| where isnotempty(IPAddress) or isnotempty(Account) or isnotempty(HostName)
| summarize AccountList = make_set(Account), HostList = make_set(HostName) by TimeGenerated, AlertName, AlertSeverity, Description, AlertType, Status, Techniques, AlertLink
```

#### Visualizing Incidents by MITRE ATT&CK Tactics

To visualize incidents generated in Microsoft Sentinel by MITRE ATT&CK tactics, use the following query. Note that the required data connector is Microsoft Sentinel Incidents, which is generated automatically if you create incidents in Sentinel.

```kql
SecurityIncident
| where TimeGenerated > ago(30d)
| summarize arg_min(TimeGenerated, *) by IncidentNumber
| extend Tactics = tostring(AdditionalData.tactics)
| where Tactics != "[]"
| mv-expand todynamic(Tactics)
| summarize Count = count() by tostring(Tactics)
| sort by Count
| render barchart with (title="Microsoft Sentinel incidents by MITRE ATT&CK tactic")
```

# Microsoft Security Querries
## Advanced Threat Hunt
In order to identify the infected devices by a specific CVE we can use the following querry :
```kql
DeviceTvmSoftwareVulnerabilities
| where CveId == "CVE-2024-6387"
| project DeviceId, DeviceName, OSPlatform, OSVersion, OSArchitecture, SoftwareVendor, CveMitigationStatus
```
### Detailed Threat Hunt

This KQL (Kusto Query Language) query is designed to identify devices that are vulnerable to a specific CVE (Common Vulnerabilities and Exposures) ID, based on their association with critical identities within a network. The query consists of three main parts: defining critical identities, identifying critical devices associated with these identities, and filtering for devices vulnerable to a specific CVE. Here’s a detailed breakdown:

#### 1. Defining Critical Identities

```kql
let CriticalIdentities = 
    ExposureGraphNodes
    | where set_has_element(Categories, "identity")
    | where isnotnull(NodeProperties.rawData.criticalityLevel) and NodeProperties.rawData.criticalityLevel.criticalityLevel < 4 
    | distinct NodeName;
```

- **ExposureGraphNodes**: This table contains information about various nodes in the exposure graph, such as users, devices, or other entities.
- **where set_has_element(Categories, "identity")**: Filters the nodes to include only those categorized as "identity".
- **where isnotnull(NodeProperties.rawData.criticalityLevel) and NodeProperties.rawData.criticalityLevel.criticalityLevel < 4**: Further filters the identities to include only those with a defined criticality level that is less than 4 (assuming a scale where lower numbers indicate higher criticality).
- **distinct NodeName**: Selects unique node names that meet the above criteria, resulting in a set of critical identities.

#### 2. Identifying Critical Devices

```kql
let CriticalDevices = 
    ExposureGraphEdges 
    | where EdgeLabel == @"can authenticate to"
    | join ExposureGraphNodes on $left.TargetNodeId==$right.NodeId
    | extend DName = tostring(NodeProperties.rawData.deviceName)
    | extend isLocalAdmin = EdgeProperties.rawData.userRightsOnDevice.isLocalAdmin
    | where SourceNodeName has_any (CriticalIdentities)
    | distinct DName;
```

- **ExposureGraphEdges**: This table contains information about relationships (edges) between nodes in the exposure graph.
- **where EdgeLabel == @"can authenticate to"**: Filters the edges to include only those where the relationship type is "can authenticate to", indicating that one node (identity) can authenticate to another node (device).
- **join ExposureGraphNodes on $left.TargetNodeId==$right.NodeId**: Joins the edges with the nodes to get additional properties of the target nodes (devices).
- **extend DName = tostring(NodeProperties.rawData.deviceName)**: Extracts the device name from the node properties.
- **extend isLocalAdmin = EdgeProperties.rawData.userRightsOnDevice.isLocalAdmin**: Extracts whether the identity has local admin rights on the device.
- **where SourceNodeName has_any (CriticalIdentities)**: Filters to include only edges where the source node name is one of the critical identities identified earlier.
- **distinct DName**: Selects unique device names associated with these critical identities.

#### 3. Filtering for Vulnerable Devices

```kql
DeviceTvmSoftwareVulnerabilities 
| where CveId == "CVE-2024-38021"
| where DeviceName has_any (CriticalDevices)
```

- **DeviceTvmSoftwareVulnerabilities**: This table contains information about software vulnerabilities on devices.
- **where CveId == "CVE-2024-38021"**: Filters the table to include only records related to the specific CVE ID "CVE-2024-38021".
- **where DeviceName has_any (CriticalDevices)**: Further filters to include only devices that are in the list of critical devices identified earlier.

#### 4. Final Querry
```kql
let CriticalIdentities =
ExposureGraphNodes
| where set_has_element(Categories, "identity")
| where isnotnull(NodeProperties.rawData.criticalityLevel) and
NodeProperties.rawData.criticalityLevel.criticalityLevel < 4 
| distinct NodeName;
let CriticalDevices =
ExposureGraphEdges 
| where EdgeLabel == @"can authenticate to"
| join ExposureGraphNodes on $left.TargetNodeId==$right.NodeId
| extend DName = tostring(NodeProperties.rawData.deviceName)
| extend isLocalAdmin = EdgeProperties.rawData.userRightsOnDevice.isLocalAdmin
| where SourceNodeName has_any (CriticalIdentities)
| distinct DName;
DeviceTvmSoftwareVulnerabilities 
| where CveId == "CVE-2024-38021"
| where DeviceName has_any (CriticalDevices)
```


