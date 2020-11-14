---
layout: post
title: httpClientFactory
---

There are already tons of good articles talking about benefit of using the httpClientFactory from .net core 2.1. 
The key fact is, the correct way to use new http client factory is to create a new instance of httpClient for every single call. 
Under the hood, the creating and disposal of delegating handler pipeline, including the primary http client handler (connection), is managed by http client factory so we don't need to worry about it anymore.  

This note is to outline two common mistakes using the new http client factory which has NOT been well documented by Microsoft.  

A few facts:  
We may have more than one named httpClient in a service, e.g.: when our service is consuming both Service A and Service B.  
For every named httpClient:  
- there can be only 1 **active** handler pipeline at a time.  
- there can be >= 0 expired handler pipeline at a time. Expired handler pipeline will only be disposed when no instance of httpClient is referencing it.They are still kept in the cache so if there are still any request being executed they are not affected.
- Everytime a new httpClient instance is created, it will be created with the **active** hanlder pipeline.  
- Every configured million seconds, a new active handler pipeline is created (along with a new scope), and the existing one is moved to the expired handler pipeline cache.

The idea of this setting is:   
1) to address socket exhaustion issue: to make sure there is only one primary http client handler for calling a service at a time. strictly speaking there will be more than one (expired ones), but they will be soon disposed after finishing executing the last http request.
2) to respect DNS changes: the connection can be refereshed every configured million seconds (via setHandlerLifetime)  

So based on above we can understand why it is important to ALWAYS create a new instance of httpClient via httpClientFactory for every single call:
Creating a new httpClient means we give it a chance to make use of the newly created active handler pipeline, so DNS changes can be respected.

Two common mistakes:

1) Mistake A: register parent component of typed httpClient as singleton.  
The two most populate way for creating httpClient via httpClientFactory is "Named" vs "Typed".  
It is quite obvious how to make sure a new instance of httpClient is created every call via "Named" approach:
```cs
        var client = _httpClientFactory.CreateClient("MyCustomAPI");
        return client.GetStringAsync("/ping");
 ```
 
 While it is less intuitive how the same should happen via "Typed" route:
 Typed httpClient is regisered as transient under the hood, by httpClientFactory.  
 We need to make absolutely sure every component resolving typed http client needs to be registered as transient as well.  
 
 2) Mistake B: Register delegating handlers as singlton.  
 Delegate handlers should always be registered as transient, as mentioned the factory will create a new set of pipeline every configured million seconds, if one of the handler along the chain is a singlton that may have some adverse effects constructing the new chain:  
https://github.com/dotnet/runtime/blob/76904319b41a1dd0823daaaaae6e56769ed19ed3/src/libraries/Microsoft.Extensions.Http/src/HttpMessageHandlerBuilder.cs#L93  
a single instance of delegating handler can only have one inner handler, registering it as singleton means  
1) the implementation cannot be shared across multiple named httpClients in the same service.  
2) It stops the factory instantiating a new set of handler pipeline when the previous active one gets expired.  


