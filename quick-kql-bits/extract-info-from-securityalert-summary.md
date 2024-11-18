# KQL bits to quickly extract important information neccesary for investigations

### Extract all Entity information from SecurityAlert tables
```
//Extract entity details from SecurityAlert
| mv-expand parse_json(Entities)
| extend Type = tostring(Entities.Type)
| extend AccountName = tostring(Entities.Name)
| extend Host = tostring(Entities.HostName)
| extend Address = tostring(Entities.Address)
| summarize count(), ['Type'] = make_set_if(Type, true), ['Account Name'] = make_set_if(AccountName, true), ['Host Name'] = make_set_if(Host, true), ['IP Address'] = make_set_if(Address, true) by SystemAlertId, AlertName
```