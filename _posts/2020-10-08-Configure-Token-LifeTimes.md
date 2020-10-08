---
layout: post
title: Configure Token LifeTimes for a service principal
---

This configuration is per tenant, service principal, or application.  
In preview only, so need to load the module before using it:  
```powershell
Import-Module AzureADPreview
Connect-AzureAD -Tenant {tenantId}

# list all the policies
Get-AzureADPolicy

# Create a policy so access token gets expired after 10 min
$policy = New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1,"AccessTokenLifetime":"00:10:00"}}') -DisplayName "TenMinutesTokenPolicy" -IsOrganizationDefault $false -Type "TokenLifetimePolicy"

# Get ID of the service principal
$sp = Get-AzureADServicePrincipal -Filter "DisplayName eq '<service principal display name>'"

# Assign policy to a service principal
Add-AzureADServicePrincipalPolicy -Id $sp.ObjectId -RefObjectId $policy.Id
```
  
A few issues when setting this up:
* Got lots of bad requests trying to get it expire in 2min time with no joy, this is because:  
https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-configurable-token-lifetimes#token-types  
Minimium time is 10 min as specified above.

* When assign the TokenLifetimePolicy to a service principal/application, it is important to be aware of this will affect all the access token issued for getting access to this resrouce (sp/app), not access token requested from this sp. i.e.: it will need to be assigned on the callee side. 

Quiz Time:
* I'd like to configure token life time, so when my app "myApp" is calling a db hosted in SQL Azure, the access token will be expired in 10 min instead of the ususal 60 min, what do I do?  

Option A:
$sp = Get-AzureADServicePrincipal -Filter "DisplayName eq 'my App'"  
Add-AzureADServicePrincipalPolicy -Id $sp.ObjectId -RefObjectId $policy.Id  

Option B:
$sp = Get-AzureADServicePrincipal -Filter "DisplayName eq 'Azure SQL Database'"  
Add-AzureADServicePrincipalPolicy -Id $sp.ObjectId -RefObjectId $policy.Id  

Answer: B  
