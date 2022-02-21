---
title: "Azure functions auth in k8s"
weight: 1
tags: ["kubernetes","az-functions"]
date: 2022-02-11T14:02:56+01:00
---

On this [docker hub page](https://hub.docker.com/_/microsoft-azure-functions-base) you can find all functions containers. This will allow you to run your Azure Functions inside kubernetes.

But how do you set this up? What things are not included?

## Functions in Azure itself

[Azure functions](https://docs.microsoft.com/en-us/azure/azure-functions/) make it very easy to run your pieces of code in Azure.

As I currently see it: Microsoft divided 'core' functions from hosting functions. The GUI inside Azure allows you to:
* Allows you to swap between different slots (but that switching actually causes short downtimes on your site, good topic for another blog)
* Use secrets from a keyvault in your application settings
* Host your function with custom domains
* Use keys to protect your HTTP functions with an [api key](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-http-webhook-trigger?tabs=csharp#api-key-authorization)
  * App keys protect all HTTP functions
  * Function keys protect specific HTTP functions
  * Functions use an attribute (`HttpTrigger(AuthorizationLevel.Anonymous)`) to define which exact access right they need (could be anonymous)

## Support for keys

Where the last function actuall *is* supported in the container as partly supported:
* https://github.com/Azure/azure-functions-host/issues/4147#issuecomment-477431016
* https://martinbjorkstrom.com/posts/2020-02-12-testing-protected-azure-functions).
* https://docs.microsoft.com/en-us/azure/azure-functions/security-concepts#secret-repositories

But the question is: do you want it. Having calls between functions secured with keys is actually tedious work if you have multiple functions that call each other. You can also just secure the outside-in traffic by putting [a key on your ingress]({{< ref "ingress-api-key" >}}).

