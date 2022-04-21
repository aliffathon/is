---
layout: post
title:  "Draft: Everythink"
date:   2022-04-19 11:00:07 +0700
categories: everythink
---

## Install Rancher on Docker Single Node

- [https://rancher.com/docs/rancher/v2.5/en/installation/other-installation-methods/single-node-docker/](https://rancher.com/docs/rancher/v2.5/en/installation/other-installation-methods/single-node-docker/)

```bash
# opsi a: default rancher-generated self-signed certificate
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  rancher/rancher:latest

# opsi b: menggunakan self-signed certificate yg sudah ada
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
  -v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
  -v /<CERT_DIRECTORY>/<CA_CERTS.pem>:/etc/rancher/ssl/cacerts.pem \
  --privileged \
  rancher/rancher:latest

# opsi c: menggunakan cert yg sudah di signed online (berbayar)
# opsi --no-cacerts mencegah rancher generate ca baru
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
  -v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
  --privileged \
  rancher/rancher:latest \
  --no-cacerts

# opsi d: letsencrypt, otomatis, perlu domain aktif
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  rancher/rancher:latest \
  --acme-domain <YOUR.DNS.NAME>
```