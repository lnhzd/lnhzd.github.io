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
  
Got lots of bad requests trying to get it expire in 2min time with no joy, this is because:  
https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-configurable-token-lifetimes#token-types  
Minimium time is 10 min as specified above.
