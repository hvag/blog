---
layout: page
title: Security and Governance
permalink: /security/
published: true
comments: true
---

Let's start collecting items for a possible security policy

1. All users have unique credentials

2. Data at rest is encrypted

3. Data in flight is encrypted

4. Devices with public facing interfaces utilize two-factor authentication

5. System events will be logged


## <a name='kubernetes'></a>Kubernetes

### Secrets

  * On the API server, the secret data is store as plaintext in ETCD
  * The secret data is encoded as base64 - This is **NOT ENCRYPTION**
