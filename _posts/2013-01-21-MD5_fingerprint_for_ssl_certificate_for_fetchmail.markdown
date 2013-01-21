---
layout: post
title: "MD5 fingerprint for ssl certificate for fetchmail"
---

```bash
openssl s_client -connect pop.gmail.com:995 -showcerts | openssl x509 -fingerprint -noout -md5
```
