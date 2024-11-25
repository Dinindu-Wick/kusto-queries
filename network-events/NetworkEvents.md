# KQL's for investigating Network Events

### Device Network Events

#### Looking for any successful connections to specific domains or IP addresses

```
let ip_addresses = dynamic([""]);
let domains = dynamic([""]);
DeviceNetworkEvents
|where TimeGenerated >= ago(30d)
//|where * has "domain" or * has "domain"
|where RemoteURL has_any domains or RemoteIP has_any ip_addresses
|where ActionType == "ConnectionSuccess"
|summarize count() by DeviceName, InitiatingProcessAccountUpn, InitiatingProcessAccountName, ActionType, InitiatingProcessParentFileName, InitiatingProcessVersionInfoInternalFileName, LocalIP, RemoteIP, RemotePort, RemoteUrl, InitiatingProcessCommandLine
```

```
//Add this line to find all the applications that had successful connections and their IPs/URLs
|summarize count(), make_set(DeviceName), make_set(InitiatingProcessAccountUpn), make_set(InitiatingProcessAccountName), make_set(ActionType), make_set(InitiatingProcessParentFileName), make_set(InitiatingProcessVersionInfoInternalFileName), make_set(LocalIP), make_set(RemoteIP), make_set(RemotePort), make_set(RemoteUrl), make_set(InitiatingProcessCommandLine) by InitiatingProcessFileName
```



