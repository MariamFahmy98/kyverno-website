---
type: "docs"
title: Restrict Service Account
linkTitle: Restrict Service Account
weight: 30
description: >
    Restrict Pod resources to use a known service account can be useful to ensure workload identity security. Change this policy to match your namespace, image and serviceAccountName
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/restrict-service-account.yaml" target="-blank">/other/restrict-service-account.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: restrict-service-account
  annotations:
    policies.kyverno.io/title: Restrict Service Account
    policies.kyverno.io/category: Sample
    policies.kyverno.io/description: >-
      Restrict Pod resources to use a known service account can be useful to
      ensure workload identity security. Change this policy to match your
      namespace, image and serviceAccountName
spec:
  rules:
  # this rule requires that workload Controller resources have a service
  # account explicitly defined. Otherwise the pod will be created with
  # default service account.
  - name: require-service-account
    match:
      resources:
        kinds:
        - Pod
        namespaces:
        - staging
    validate:
      message: "Service account required for Controllers"
      pattern:
        spec:
          serviceAccountName: "?*"
          containers:
          # in a real-world scenario this would be your app's image
          - =(image): "alpine"
  - name: validate-service-account
    match:
      resources:
        kinds:
        - Pod
        namespaces:
        - staging
    validate:
      message: "Invalid service account"
      pattern:
        spec:
          =(serviceAccountName): "gcs-viewer"
          containers:
          # in a real-world scenario this would be your app's image
          - =(image): "alpine"

```