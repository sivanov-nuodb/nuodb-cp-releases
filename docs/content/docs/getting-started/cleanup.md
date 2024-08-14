---
title: "Remove resources"
description: ""
summary: ""
date: 2024-08-14T13:44:13+03:00
lastmod: 2024-08-14T13:44:13+03:00
draft: false
weight: 140
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

- Delete all custom resources that have been created in `nuodb-cp-system` namespace.

```sh
kubectl config set-context --current --namespace=nuodb-cp-system
kubectl get databases.cp.nuodb.com -o name | xargs kubectl delete
kubectl get domains.cp.nuodb.com -o name | xargs kubectl delete
kubectl get servicetiers.cp.nuodb.com -o name | xargs kubectl delete
kubectl get helmfeatures.cp.nuodb.com -o name | xargs kubectl delete
kubectl get databasequotas.cp.nuodb.com -o name | xargs -r kubectl delete
kubectl get secrets -o name --selector=cp.nuodb.com/organization | xargs -r kubectl delete
kubectl get pvc -o name --selector=group=nuodb | xargs -r kubectl delete
```

- Cleanup the installed resources in the following order:

```sh
helm uninstall nuodb-cp-rest --namespace nuodb-cp-system
helm uninstall nuodb-cp-operator --namespace nuodb-cp-system
helm uninstall nuodb-cp-crd --namespace nuodb-cp-system
helm uninstall cert-manager --namespace cert-manager
```

- Delete the provisioned namespace:

```sh
kubectl delete namespace nuodb-cp-system
kubectl delete namespace cert-manager
```
