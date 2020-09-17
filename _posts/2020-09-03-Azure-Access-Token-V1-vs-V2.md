---
layout: post
title: Azure Access Token
---

A placeholder to talk about Azure Access Token v1 vs v2
-
-
-
-

| accessTokenAcceptedVersion   (callee) | iss                       | aud                                  | token endpoint | param    | resp |
|---------------------------------------|---------------------------|--------------------------------------|----------------|----------|------|
| 2                                     | login.microsoftonline.com | appId (Guid)                         | v2.0           | scope    | 200  |
| 2                                     | login.microsoftonline.com |                                      | v2.0           | resource | 400  |
| 2                                     | login.microsoftonline.com | 00000002-0000-0000-c000-000000000000 | v1.0           | scope    | 200  |
| 2                                     | login.microsoftonline.com | appId (Guid)                         | v1.0           | resource | 200  |
| 1                                     | sts.windows.net           | api://appid*                         | v2.0           | scope    | 200  |
| 1                                     | sts.windows.net           |                                      | v2.0           | resource | 400  |
| 1                                     | sts.windows.net           | 00000002-0000-0000-c000-000000000000 | v1.0           | scope    | 200  |
| 1                                     | sts.windows.net           | api://appid*                         | v1.0           | resource | 200  |
