---
title: "Containerless scripts"
weight: 1
draft: true
date: 2022-02-11T14:07:56+01:00
---

Applying contarizing on a simple bash scripts might be a bit over the top.

* https://suraj.io/post/use-configmap-for-scripts/
* https://stackoverflow.com/a/63906620/619465

What to add to this? It's actually pretty good already 🙂
```
  script.sh: |
{{ (.Files.Get "scripts/script.sh") | indent 4 }}
```

```
{{ (.Files.Glob "scripts/*.sh").AsConfig | indent 2 }}
```


https://yaml-multiline.info/

Note again the difference in multiline and between double quotes, at some day I will learn

How could this work:
```
  source:
    helm:
      fileParameters:
      - name: config
        path: configs/external-dns.yaml
```