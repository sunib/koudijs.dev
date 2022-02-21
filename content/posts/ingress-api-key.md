---
title: "Ingress API key"
weight: 1
tags: ["kubernetes"]
date: 2022-02-15T12:08:00+01:00
---

Let's say you just wanted to have a simple API-key for your nginx ingress. The if structures in [nginx configs](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#if) are an interesting way to do this. There might be more effecient ways to do this, but for now this will do.

## Example

`example-ingress.yaml`:

```yaml
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

Running it can off course be done by:

```
kubectl apply -f example-ingress.yaml
```

## How does it work?

The trick here is the `nginx.ingress.kubernetes.io/configuration-snippet` annotation. All ingresses are transformed into one big Nginx config file. It's templated, and after some digging I found the [actual place](https://github.com/kubernetes/ingress-nginx/blob/a2a0e67fee9964796f56e3428cf6d1dc99ced261/rootfs/etc/nginx/template/nginx.tmpl#L1326-L1327) in the nginx proxy repo. 

## Warning

Please be warned that injecting other kinds of stuff could lead to [dangerous situations](https://github.com/kubernetes/ingress-nginx/issues/7837). Please be aware that you - or some sysadmin - could configure `allow-snippet-annotations: "false"`, which might be wise. But it would also break the trick described in this blog!
