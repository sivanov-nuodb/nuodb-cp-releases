---
title: "Deploy your first Database"
description: ""
summary: ""
date: 2024-08-14T13:37:13+03:00
lastmod: 2024-08-14T13:37:13+03:00
draft: false
weight: 120
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

Once the Control Plane is deployed, projects and databases can now be created.

### Access and Authentication

This guide will use port forwarding and [cURL][9] to demonstrate how to create projects and databases through the REST service.

```sh
kubectl port-forward -n nuodb-cp-system svc/nuodb-cp-rest 8080 2>&1 >/dev/null &
```

To successfully authenticate with the REST API, get the *system/admin* user's password from the cluster:

```sh
PASS=$(kubectl get secret dbaas-user-system-admin -n nuodb-cp-system -o jsonpath='{.data.password}' | base64 -d)
BASE_URL="http://localhost:8080"
```

### Create Project

Create a new project *messaging* in organization *acme*:

```sh
curl -u "system/admin:$PASS" -X PUT -H 'Content-Type: application/json' \
    $BASE_URL/projects/acme/messaging \
    -d '{"sla": "dev", "tier": "n0.small"}'
```

>**Note**
> Creating project and database with `n0.small` service tier will require 3 vCPU and 5Gi RAM allocatable resources from your cluster. If your setup is resource constrained, consider using `n0.nano` service tier.

Wait for the project to become available.

```sh
while [ "$(curl -s -u "system/admin:$PASS" $BASE_URL/projects/acme/messaging | jq '.status.ready')" != "true" ]; do echo "Waiting ..."; sleep 5; done; echo "Domain is available"
```

### Create database

Create a new database *demo* in project *messaging*:

```sh
curl -u "system/admin:$PASS" -X PUT -H 'Content-Type: application/json' \
    $BASE_URL/databases/acme/messaging/demo \
    -d '{"dbaPassword": "secret"}'
```

Wait for the database to become available.

```sh
while [ "$(curl -s -u "system/admin:$PASS" $BASE_URL/databases/acme/messaging/demo | jq '.status.ready')" != "true" ]; do echo "Waiting ..."; sleep 5; done; echo "Database is available"
```

[9]: https://curl.se/
