---
title: "Disallow Default Namespace"
category: Multi-Tenancy
version: 1.6.0
subject: Pod
policyType: "validate"
description: >
    Kubernetes Namespaces are an optional feature that provide a way to segment and isolate cluster resources across multiple applications and users. As a best practice, workloads should be isolated with Namespaces. Namespaces should be required and the default (empty) Namespace should not be used. This policy validates that Pods specify a Namespace name other than `default`. Rule auto-generation is disabled here due to Pod controllers need to specify the `namespace` field under the top-level `metadata` object and not at the Pod template level.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//best-practices/disallow-default-namespace/disallow-default-namespace.yaml" target="-blank">/best-practices/disallow-default-namespace/disallow-default-namespace.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-default-namespace
  annotations:
    pod-policies.kyverno.io/autogen-controllers: none
    policies.kyverno.io/title: Disallow Default Namespace
    policies.kyverno.io/minversion: 1.6.0
    policies.kyverno.io/category: Multi-Tenancy
    policies.kyverno.io/severity: medium
    policies.kyverno.io/subject: Pod
    policies.kyverno.io/description: >-
      Kubernetes Namespaces are an optional feature that provide a way to segment and
      isolate cluster resources across multiple applications and users. As a best
      practice, workloads should be isolated with Namespaces. Namespaces should be required
      and the default (empty) Namespace should not be used. This policy validates that Pods
      specify a Namespace name other than `default`. Rule auto-generation is disabled here
      due to Pod controllers need to specify the `namespace` field under the top-level `metadata`
      object and not at the Pod template level.
spec:
  validationFailureAction: audit
  background: true
  rules:
  - name: validate-namespace
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "Using 'default' namespace is not allowed."
      pattern:
        metadata:
          namespace: "!default"
  - name: validate-podcontroller-namespace
    match:
      any:
      - resources:
          kinds:
          - DaemonSet
          - Deployment
          - Job
          - StatefulSet
    validate:
      message: "Using 'default' namespace is not allowed for pod controllers."
      pattern:
        metadata:
          namespace: "!default"
```