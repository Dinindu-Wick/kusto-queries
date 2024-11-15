### Viewing Account Changes and Entity details

Account Change activity including creation, deletion, etc in AuditLogs table along with details of entities from Identity info table.

```AuditLogs
//Different Categories to Check ("RoleManagement", "UserManagement", "Authorization", "Policy", "PolicyManagement", "ApplicationManagement", "GroupManagement", "ProvisioningManagement") 
//|where Category == "ApplicationManagement"
//|where TimeGenerated between (()..()) //Time period for investigation
//|where OperationName has_any("Delete user","Add user", "Update user") // Enable this if you need to find deletion, addition or update operations for a user account
|where * has "username"  //Username(s) or account name(s)
|extend InitiatedByUPN = tostring(InitiatedBy.user.userPrincipalName), CreatedByIPAddress = tostring(InitiatedBy.user.ipAddress)
|extend InitiatedByDisplayName = tostring(InitiatedBy.user.displayName)
|extend InitiatedByServiceAppID = tostring(InitiatedBy.app.servicePrincipalId), InitiatedByServiceAppName = tostring(InitiatedBy.app.servicePrincipalName)
|extend CreatedByApp = tostring(InitiatedBy.app.displayName)
|extend targetUPN = tostring(TargetResources[0].userPrincipalName)
|extend targetDisplayName = tostring(TargetResources[0].displayName)
// |project TimeGenerated, InitiatedByUPN, InitiatedByDisplayName, CreatedByIPAddress, CreatedByApp, targetUPN, targetDisplayName, OperationName, Category, Identity, Result//, InitiatedByServiceAppID, InitiatedByServiceAppName
//################################  Joining AuditLogs with Identity Info table  #################################
//|join kind=leftouter IdentityInfo on $left.InitiatedByDisplayName == $right.AccountDisplayName //Join All records from the left table and only matching rows from the right table where $left tables InitiatedByDisplayName column match value in $right.AccountDisplayName
|join kind=leftouter IdentityInfo on $left.InitiatedByUPN == $right.AccountUPN //Join All records from the left table and only matching rows from the right table where $left tables InitiatedByUPN column match value in $right.AccountUPN
// |extend value1 = parse_json(column).value1 //Extract values in dict
|project TimeGenerated, InitiatedByUPN, InitiatedByDisplayName, CreatedByIPAddress, CreatedByApp, targetUPN, targetDisplayName, OperationName, Category, Identity, Result, AssignedRoles, AccountDisplayName, JobTitle
    //summarise by Initiated user and Operations and make a set of IP's relating to each of the activity
//|summarize count(), make_set(CreatedByIPAddress) by InitiatedByUPN, InitiatedByDisplayName, CreatedByApp, targetUPN, targetDisplayName, OperationName, Category, Identity, Result, tostring(AssignedRoles), AccountDisplayName, JobTitle
//###############################################################################################################
```
