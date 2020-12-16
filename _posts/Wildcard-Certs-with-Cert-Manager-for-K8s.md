---
layout: post
title: Generate and use Wild card certs for K8s Apps using Cert-Manager with Istio Gateway
date: 2020-07-13 00:23
category: k8s
author: 
tags: 
 - k8s
 - Cert-Manager
 - wildcard Cert
 - Istio
 - Istio Gateway
summary: Wild Card Certificates with Cert-Manager on K8s and Istio Gateway
---
I love the simplicity that [Cert-Manager](https://cert-manager.io/docs/) brings to the certificate management for k8s. 
- Install Cert-Manager
- Create a Issuer ( ClusterIssuer Resource)
- Create any Certificate using the Issuer. 

Just Following the above steps, gets cert-manager to do all the negotiations on your behalf with letEncrypt API and creates a valid certificate which can then be used by the IngressGateways. 

The need for me was to extend this ease of use to wilecard certs as well.  I saw this Amazing Blog which when followed verbatim ,  got me up and running in just a few mins. I wanted to quickly put my own example using those instructions.  

I had a need to create wild card certs to be used for the apps and sys domain for Tanzu Application Service for K8s. I went with the thought of sing cert-manager instead ofusing the certbot or acme commands. 

