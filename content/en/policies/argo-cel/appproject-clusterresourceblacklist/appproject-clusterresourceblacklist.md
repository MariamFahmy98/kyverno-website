---
title: "Enforce AppProject with clusterResourceBlacklist in CEL expressions"
category: Argo in CEL
version: 1.11.0
subject: AppProject
policyType: "validate"
description: >
    An AppProject may optionally specify clusterResourceBlacklist which is a blacklisted group of cluster resources. This is often a good practice to ensure AppProjects do not allow more access than needed. This policy is a combination of two rules which enforce that all AppProjects specify clusterResourceBlacklist and that their group and kind have wildcards as values.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//argo-cel/appproject-clusterresourceblacklist/appproject-clusterresourceblacklist.yaml" target="-blank">/argo-cel/appproject-clusterresourceblacklist/appproject-clusterresourceblacklist.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: appproject-clusterresourceblacklist
  annotations:
    policies.kyverno.io/title: Enforce AppProject with clusterResourceBlacklist in CEL expressions
    policies.kyverno.io/category: Argo in CEL 
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.11.0
    policies.kyverno.io/minversion: 1.11.0
    kyverno.io/kubernetes-version: "1.26-1.27"
    policies.kyverno.io/subject: AppProject
    policies.kyverno.io/description: >-
      An AppProject may optionally specify clusterResourceBlacklist which is a blacklisted
      group of cluster resources. This is often a good practice to ensure AppProjects do
      not allow more access than needed. This policy is a combination of two rules which
      enforce that all AppProjects specify clusterResourceBlacklist and that their group
      and kind have wildcards as values.
spec:
  validationFailureAction: Audit
  background: true
  rules:
    - name: has-wildcard-and-validate-clusterresourceblacklist
      match:
        any:
        - resources:
            kinds:
              - AppProject
            operations:
            - CREATE
            - UPDATE
      validate:
        cel:
          expressions:
            - expression: "has(object.spec.clusterResourceBlacklist)"
              message: "AppProject must specify clusterResourceBlacklist."
            - expression: "object.spec.clusterResourceBlacklist.all(element, element.group.contains('*') && element.kind.contains('*'))"
              message: "Wildcards must be present in group and kind for clusterResourceBlacklist."


```
