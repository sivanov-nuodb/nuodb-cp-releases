---
title: "Connect to the Database"
description: ""
summary: ""
date: 2024-08-14T13:40:00+03:00
lastmod: 2024-08-14T13:40:00+03:00
draft: false
weight: 130
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  noindex: false # false (default) or true
---

This guide will use port forwarding to connect to the NuoDB database.

```sh
ADMIN_SVC=$(kubectl get svc -n nuodb-cp-system \
    -l 'cp.nuodb.com/organization=acme,cp.nuodb.com/project=messaging,!cp.nuodb.com/database' -oname | grep "clusterip")
DB_SVC=$(kubectl get svc -n nuodb-cp-system \
    -l "cp.nuodb.com/organization=acme,cp.nuodb.com/project=messaging,cp.nuodb.com/database" -oname)
kubectl port-forward -n nuodb-cp-system $ADMIN_SVC 48004 2>&1 >/dev/null &
kubectl port-forward -n nuodb-cp-system $DB_SVC 48006 2>&1 >/dev/null &
```

Connect to the NuoDB database via `nuosql` (requires [nuodb-client][8] package v20230228 or later).

```sh
CA_CERT="$(curl -s -u "system/admin:$PASS" $BASE_URL/databases/acme/messaging/demo | jq -r '.status.caPem')"
DB_URL="$(curl -s -u "system/admin:$PASS" $BASE_URL/databases/acme/messaging/demo | jq -r '.status.sqlEndpoint')"
nuosql "demo@${DB_URL}" --user dba --password secret --connection-property trustedCertificates="$CA_CERT"
```

[8]: https://github.com/nuodb/nuodb-client/releases