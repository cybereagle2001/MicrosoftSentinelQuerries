# MicrosoftSentinelQuerries
Sentinel is a well known SIEM in the modern security and the most famous SIEM as a service by Microsoft. Based on each connector you can create it will give you tha ability to activate specific workbooks for data visualisation and statics but also will active a specific kind of Logs.
The most important 

This request will only extract alerts generated by Microsoft Defender with High Severity
```
 SecurityAlert
| where ProviderName contains "MD" and AlertSeverity contains "high"
| project AlertName,AlertSeverity,Description,ProviderName,AlertType,Entities,Status,Techniques

```
For better Investigation we need to have more details and information about the alert generated to do so we can add the column storing the Link of the alert :
```
 SecurityAlert
| where ProviderName contains "MD" and AlertSeverity contains "high"
| project AlertName,AlertSeverity,Description,ProviderName,AlertType,Entities,Status,Techniques,AlertLink
```

In order to extract all the alerts generated by Windows Defender and Also highliting for each alert the affected account, IP, HostList
```
SecurityAlert
// choosing only alerts created by Defender
| where ProviderName contains "MD"
//extend Entities
| extend EntitiesDynamicArray = parse_json(Entities) 
| mv-expand EntitiesDynamicArray
// Parsing relevant entity column extract hostname and IP address
| extend EntityType = tostring(parse_json(EntitiesDynamicArray).Type), EntityAddress = tostring(EntitiesDynamicArray.Address), EntityHostName = tostring(EntitiesDynamicArray.HostName), EntityAccountName = tostring(EntitiesDynamicArray.Name)
| extend HostName = iif(EntityType == 'host', EntityHostName, '')
| extend IPAddress = iif(EntityType == 'ip', EntityAddress, '')
| extend Account = iif(EntityType == 'account', EntityAccountName, '')
| where isnotempty(IPAddress) or isnotempty(Account) or isnotempty(HostName)
// Aggregating by Time and Alert to consolidate all entities per Alert
| summarize AccountList=make_set(Account), IPList = make_set(IPAddress), HostList = make_set(HostName) by TimeGenerated, AlertName,AlertSeverity,Description,AlertType,Status,Techniques
```

this is my magical QKL querry allowing me to identify all possible threats on devices and servers to mitigate :
```
SecurityAlert
// choosing only alerts created by Defender
| where ProviderName contains "MD" and Status != "Resolved"
//extend Entities
| extend EntitiesDynamicArray = parse_json(Entities)
| mv-expand EntitiesDynamicArray
// Parsing relevant entity column extract hostname and IP address
| extend EntityType = tostring(parse_json(EntitiesDynamicArray).Type), EntityAddress = tostring(EntitiesDynamicArray.Address), EntityHostName = tostring(EntitiesDynamicArray.HostName), EntityAccountName = tostring(EntitiesDynamicArray.Name)
| extend HostName = iif(EntityType == 'host', EntityHostName, '')
| extend IPAddress = iif(EntityType == 'ip', EntityAddress, '')
| extend Account = iif(EntityType == 'account', EntityAccountName, '')
| where isnotempty(IPAddress) or isnotempty(Account) or isnotempty(HostName)
// Aggregating by Time and Alert to consolidate all entities per Alert
| summarize AccountList=make_set(Account), HostList = make_set(HostName) by TimeGenerated, AlertName,AlertSeverity,Description,AlertType,Status,Techniques
```
