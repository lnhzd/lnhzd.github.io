---
layout: post
title: Grant Admin Consents via API
---


service principals getting an access token:  
==============================================

POST https://login.microsoftonline.com/{tenantId}/oauth2/v2.0/token HTTP/1.1  
Host: login.microsoftonline.com  
Content-Type: application/x-www-form-urlencoded  

grant_type=client_credentials&client_id={clientId}&client_secret={clientSecret}&scope=https://graph.microsoft.com/.default  
   
   
Granting Admin Consent for clientOid calling targetResrouceSpOid: appPermission  
==============================================  

POST https://graph.microsoft.com/v1.0/servicePrincipals/{targetResourceSpOid}/appRoleAssignments HTTP/1.1  
Authorization: Bearer {access_token}  
Content-Type: application/json; charset=utf-8  
Host:graph.microsoft.com  
Expect: 100-continue  

```json
{
 "principalId": "{clientOid}",
 "resourceId": "{targetResourceSpOid}",
  "appRoleId": "{appRoleId}"
}
```

Granting Admin Consent for clientOid calling targetResrouceSpOid: user Permission  
==============================================
POST https://graph.microsoft.com/v1.0/oauth2PermissionGrants HTTP/1.1  
Authorization: Bearer {access_token}  
Content-Type: application/json  
Host: graph.microsoft.com  
Content-Length: 187  

```json
{
    "clientId":  "{clientOid}",
    "consentType":  "AllPrincipals",
    "resourceId":  "{targetResourceSpOid}",
    "scope":  "{scopeName}"
}
```
