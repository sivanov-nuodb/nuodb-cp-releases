---
title: "Deploy NuoDB Control Plane"
description: ""
summary: ""
date: 2024-08-14T13:27:07+03:00
lastmod: 2024-08-14T13:27:07+03:00
draft: false
weight: 110
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

This document describes how to provision NuoDB databases in multi-tenancy model by using NuoDB Control Plane (CP).
NuoDB Control Plane works with [Kubernetes][1] locally or in the cloud.
The steps in this guide can be followed regardless of the selected Kubernetes platform provider.

## Prerequisites

- A running [Kubernetes cluster][2]
- [kubectl][3] installed and able to access the cluster.
- [Helm 3.x][4] installed.

## Installing Dependencies

### Install Cert Manager

To enable [admission webhooks][7] in the NuoDB operator, [cert-manager](https://github.com/cert-manager/cert-manager) must be installed to automatically generate certificates for the webhook server.

Add the official Helm repositories.

```sh
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

Install Cert Manager Helm chart.

```sh
helm upgrade --install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --set installCRDs=true \
  --create-namespace
```

Wait for Cert Manager to become available.

```sh
kubectl -n cert-manager wait pod --all --for=condition=Ready
```

## Installing NuoDB Control Plane

The NuoDB Control Plane consists of [Custom Resource Definitions][5] and the following workloads:

- *NuoDB CP Operator*, which enforces the desired state of the NuoDB [custom resources][6].
- *NuoDB CP REST service*, that exposes a REST API allowing users to manipulate and inspect DBaaS entities.

By default the NuoDB CP will operate in a single namespace only which will be used for NuoDB CP and all databases created by it.
The databases are grouped into *projects*, which are themselves grouped into *organizations*.

Add the official Helm repositories.

```sh
helm repo add nuodb-cp https://nuodb.github.io/nuodb-cp-releases/charts
helm repo update
```

Install NuoDB CP Helm charts.

```sh
helm upgrade --install nuodb-cp-crd nuodb-cp/nuodb-cp-crd \
    --namespace nuodb-cp-system \
    --create-namespace

helm upgrade --install nuodb-cp-operator nuodb-cp/nuodb-cp-operator \
    --namespace nuodb-cp-system \
    --set cpOperator.webhooks.enabled=true \
    --set 'cpOperator.extraArgs[0]=--ingress-https-port=48006' # Enables connecting to databases with port-forwarding

helm upgrade --install nuodb-cp-rest nuodb-cp/nuodb-cp-rest \
    --namespace nuodb-cp-system \
    --set cpRest.authentication.enabled=true \
    --set cpRest.authentication.admin.create=true \
    --set cpRest.baseDomainName=dbaas.localtest.me # Enables connecting to databases with port-forwarding
```

Wait for NuoDB Control Plane to become available.

```sh
kubectl -n nuodb-cp-system -l app=nuodb-cp-operator wait pod --all --for=condition=Ready
kubectl -n nuodb-cp-system -l app=nuodb-cp-rest wait pod --all --for=condition=Ready
```

[1]: https://kubernetes.io/docs/home/
[2]: https://kubernetes.io/docs/concepts/overview/components/
[3]: https://kubernetes.io/docs/tasks/tools/
[4]: https://helm.sh/
[5]: https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions
[6]: https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#custom-resources
[7]: https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/
