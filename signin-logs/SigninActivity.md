# KQL's to investigate Sign-in activities

### SigninLogs Activity
```
//Check for success and failed sign-ins for all users IP addresses from outside of Australia for success and suspicious failures
let successCodes = dynamic([0]);
let user_names = dynamic(["",""]);
let ip_addresses = dynamic(["",""]);
let time_range = 3d;
union SigninLogs, AADManagedIdentitySignInLogs
|where TimeGenerated >= ago(time_range) //Change the duration for the search
//|where TimeGenerated between (()..()) //Provide a time period for the search
//|where IPAddress !hasprefix ("159.207.248") //Use this to exclude ip address range
//|where IPAddress has_any(ip_addresses)
|where UserPrincipalName has_any (user_names) //Provide a list of usernames for the search
|where ResultType in (successCodes) or ResultType !in (successCodes) //where only sign-ins that have a success or failure code or suspicious failure codes
// | where Location !="AU"  //Show only sign-ins from locations outside of Australia
|extend step1AuthMethod = strcat(tostring(parse_json(AuthenticationDetails)[0].authenticationMethod), " | " , tostring(parse_json(AuthenticationDetails)[0].authenticationStepResultDetail), " | ", tostring(parse_json(AuthenticationDetails)[0].succeeded)) //First factor (password) needs to be true to reach conditional access policy
|extend step2AuthMethod = strcat(tostring(parse_json(AuthenticationDetails)[1].authenticationMethod), " | " , tostring(parse_json(AuthenticationDetails)[1].authenticationStepResultDetail), " | ", tostring(parse_json(AuthenticationDetails)[1].succeeded))
//|extend DeviceDetail = strcat(tostring(parse_json(DeviceDetail.browser)), " | " , tostring(parse_json(DeviceDetail.operatingSystem)))
|extend LocationDetails = strcat(tostring(parse_json(LocationDetails_dynamic.city)), " | ", tostring(parse_json(LocationDetails_dynamic.countryOrRegion)))
| summarize 
    ['Count of successful signins'] = countif((ResultType in(successCodes))), //count number of successful sign-ins where success code match ResultType value
    ['Successful result codes'] = make_set_if(ResultType, (ResultType in(successCodes))), //make a set of the relevant success codes,
    ['Successful From Location'] = make_set_if(LocationDetails, (ResultType in(successCodes))), //make a list of locations with successful sign-ins,
    ['Success IP addresses'] = make_set_if(IPAddress, (ResultType in (successCodes))), //make a set of IP addresses for successful sign-ins
    ['Count of failed signins']=countif((ResultType !in(successCodes))),//count number of failed sign-ins where failure code match ResultType value
    ['Failed result codes'] = make_set_if(ResultType, (ResultType !in(successCodes))), //make a set of the relevant failure codes
    ['Failed From Location'] = make_set_if(LocationDetails, (ResultType !in(successCodes))), //make a list of locations with failed sign-ins,
    ['Failed IP addresses'] = make_set_if(IPAddress, (ResultType !in(successCodes))), //make a set of IP addresses for failed sign-ins
    ['ResultDescription'] = make_set_if(ResultDescription, (ResultType !in (successCodes)))
by TimeGenerated, UserPrincipalName, step1AuthMethod, step2AuthMethod, AppDisplayName //, DeviceDetail
```
Ref: https://www.kqlsearch.com/


### Non-Interactive User Sign-in activity

```
//Used for Device logins checks
let time_range = 7d;
let date_time1 = dynamic(["2023-09-05T08:16:31.950721Z"]);
let date_time2 = dynamic(["2023-09-06T08:17:30.00Z"]);
let user_names = dynamic(["userone"]);
let device_names = dynamic(["device"]);
AADNonInteractiveUserSignInLogs
//|where TimeGenerated between ((todatetime('{date_time1}')) .. (todatetime('{date_time2}')))
|where TimeGenerated >= ago(time_range) //Change the duration for the search
|where UserPrincipalName has_any(user_names) or UserDisplayName has_any (user_names)
|extend DeviceName = strcat(tostring(parse_json(DeviceDetail).displayName), " | " , tostring(parse_json(DeviceDetail).operatingSystem), " | ", tostring(parse_json(DeviceDetail).browser))
|where DeviceName has_any (device_names)
|summarize count() by Identity, UserPrincipalName, AuthenticationRequirement, IPAddress, ClientAppUsed, Location, AppDisplayName, UserAgent, UniqueTokenIdentifier, ResourceDisplayName, DeviceName, ResultType, ResultDescription
```

