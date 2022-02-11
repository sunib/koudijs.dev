---
title: "Azure functions auth in k8s"
weight: 1
header_menu: true
date: 2022-02-11T14:07:56+01:00
---

[Azure functions](https://docs.microsoft.com/en-us/azure/azure-functions/) make it very easy to run your pieces of code in Azure. The cool thing is that they also make it very easy to use your functions inside a k8s cluster.

But not all stuff is supported. The functionapp resource is giving you a 'wrapper' that:
* Allows you to swap between different slots
* Use secrets from a keyvault in your application settings
* Host your function with custom domains
* Use keys to protect your HTTP functions with an [api key](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-http-webhook-trigger?tabs=csharp#api-key-authorization)
  * App keys protect all HTTP functions
  * Function keys protect specific HTTP functions
  * Functions use an attribute (`HttpTrigger(AuthorizationLevel.Anonymous)`) to define which exact access right they need (could be anonymous)

So some smart guys actually docuemnted how you can get your keys into the containers: https://github.com/Azure/azure-functions-host/issues/4147#issuecomment-477431016, https://martinbjorkstrom.com/posts/2020-02-12-testing-protected-azure-functions

There is even support for k8s cluster!!! https://docs.microsoft.com/en-us/azure/azure-functions/security-concepts#secret-repositories

So here goes my first blogpost, bammmm

Let's say you just wanted to have a simple API-key for your nginx ingress. In the end the ingress is still creating 

The if structures are pretty [interesting in nginx](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#if). There might be more effecient ways to do this, but for now this will do.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: some-ingress
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/configuration-snippet: |
      if ($arg_api_key != 'yourVerySecretKey') {
        return 401 'Access denied!';
      }
      if ($http_x_api_key != 'yourVerySecretKey') {
        return 401 'Access denied!';
      }
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - koudijs.dev
    secretName: tls-koudijs-dev
  rules:
  - host: koudijs.dev
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: some-backend-service
            port:
              number: 80
```

Create a little example out of this that could run in minikube? TODO: I'm not sure using variables inside the regex

But hey how does this even work?

All ingresses are transformed into an Nginx config file. It's templated, and after some digging I found the [actual place](https://github.com/kubernetes/ingress-nginx/blob/a2a0e67fee9964796f56e3428cf6d1dc99ced261/rootfs/etc/nginx/template/nginx.tmpl#L1326-L1327) in the nginx proxy repo.

Also be warned that injecting other kinds of stuff could lead to [dangerous situations](https://github.com/kubernetes/ingress-nginx/issues/7837). Please be aware that you - or some sysadmin - could configure `allow-snippet-annotations: "false"`, which might be wise. But it would also break the trick described in this blog.

