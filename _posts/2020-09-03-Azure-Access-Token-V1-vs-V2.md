---
layout: post
title: Microsoft Identity Platform vs Azure AD
---

AAD refers to v1 endpoint, while MIP refers to v2 endpoint.
AAD and MIP exist side by side, for each tenant, you’ll have one discovery endpoint for each of them, e.g.:

AAD:  
https://login.microsoftonline.com/cbbd5fc3-8924-44f4-aa61-e1683f47d182/.well-known/openid-configuration  
[Formatted](https://raw.githubusercontent.com/lnhzd/lnhzd.github.io/master/resources/2020-09-03_discovery_v1)

MIP:  
https://login.microsoftonline.com/cbbd5fc3-8924-44f4-aa61-e1683f47d182/v2.0/.well-known/openid-configuration  
[Formatted](https://raw.githubusercontent.com/lnhzd/lnhzd.github.io/master/resources/2020-09-03_discovery_v2)  
 
#### Access Token, v1 or v2?
It is the value of “accesstokenacceptedversion” under Manifest of App Registration determines if a v1 or v2 token will be issued – by default this will be V1 – “accesstokenacceptedversion”: null.

#### Caller or Callee?
Accesstokenacceptedversion determines the version of access token issued for an API you need to get access to (callee).
If accesstokenacceptedversion is set to 2 for an Azure App named App A with AppId (c3520820-bfc1-4e44-b0a2-cbe57eb28068) – then all the accessToken request asking for access of App A, either through a V1 endpoint, or a V2 endpoint, will BOTH get an v2 access token.

Therefore, when requesting an access token for calling app - c3520820-bfc1-4e44-b0a2-cbe57eb28068 – you’ll get the same V2 access token from both token requests below.

* v1:

POST https://login.microsoftonline.com/cbbd5fc3-8924-44f4-aa61-e1683f47d182/oauth2/token HTTP/1.1  
Host: login.microsoftonline.com  
Content-Type: application/x-www-form-urlencoded  

grant_type=password&client_id=11691347-c9a6-446b-aec1-04cba2dd36dc&client_secret=7Ck.B5vKOE[Jb]3kA.n1qy@QSdTVykox&resource=api://c3520820-bfc1-4e44-b0a2-cbe57eb28068&username=atlasuser@rdt-online.com&password=********

* v2:

POST https://login.microsoftonline.com/cbbd5fc3-8924-44f4-aa61-e1683f47d182/oauth2/v2.0/token HTTP/1.1  
Host: login.microsoftonline.com  
Content-Type: application/x-www-form-urlencoded  

grant_type=password&client_id=11691347-c9a6-446b-aec1-04cba2dd36dc&client_secret=7Ck.B5vKOE[Jb]3kA.n1qy@QSdTVykox&scope=api://c3520820-bfc1-4e44-b0a2-cbe57eb28068/user_impersonation%20Offline_access&username=atlasuser@rdt-online.com&password=********


#### Securing an API:
When adding an authentication scheme to secure the API for incoming traffic, we will need to configure an authority (TokenUrl) your API is secured with, such authority will have to match the “accesstokenacceptedversion” of the Azure AD app you are secured with.  
https://login.microsoftonline.com/cbbd5fc3-8924-44f4-aa61-e1683f47d182/oauth2/token  
https://login.microsoftonline.com/cbbd5fc3-8924-44f4-aa61-e1683f47d182/oauth2/v2.0/token  

on app’s start-up, depends on the authority configured, the app will go to the corresponding discovery endpoint to fetch the OIDC metadata doc, as a result of this, for any requests that calls into the API with an access token, the claims from the access token needs to match whatever exposed from the discovery endpoint – it is quite obvious that only the configured version of access token will be issued hence the meta doc from the discovery endpoint will need to match it or an 401 will be returned. 

#### a comparison of common used claims between v1 v2

| accessTokenAcceptedVersion   (callee) | iss (claim)               | aud (claim)                          | token endpoint | param    | resp |
|---------------------------------------|---------------------------|--------------------------------------|----------------|----------|------|
| 2                                     | login.microsoftonline.com | appId (Guid)                         | v2.0           | scope    | 200  |
| 2                                     | login.microsoftonline.com |                                      | v2.0           | resource | 400  |
| 2                                     | login.microsoftonline.com | 00000002-0000-0000-c000-000000000000 | v1.0           | scope    | 200  |
| 2                                     | login.microsoftonline.com | appId (Guid)                         | v1.0           | resource | 200  |
| 1                                     | sts.windows.net           | api://appid*                         | v2.0           | scope    | 200  |
| 1                                     | sts.windows.net           |                                      | v2.0           | resource | 400  |
| 1                                     | sts.windows.net           | 00000002-0000-0000-c000-000000000000 | v1.0           | scope    | 200  |
| 1                                     | sts.windows.net           | api://appid*                         | v1.0           | resource | 200  |


00000002-0000-0000-c000-000000000000 is the app Id for AAD graph API - scope param is not recognoized by v1 endpoint, so v1 assumes you'd like an access token for getting access to graph api by default.

