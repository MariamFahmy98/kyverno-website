---
title: "Verify VerticalPodAutoscaler Target"
category: Other
version: 
subject: VerticalPodAutoscaler
policyType: "validate"
description: >
    VerticalPodAutoscaler (VPA) is useful to automatically adjust the resources assigned to Pods. It requires defining a specific target resource by kind and name. There are no built-in validation checks by the VPA controller to ensure that the target resource exists or that the target kind is specified correctly. This policy contains two rules, the first of which verifies that the kind is specified exactly as Deployment, StatefulSet, ReplicaSet, or DaemonSet, which helps avoid typos. The second rule verifies that the target resource exists before allowing the VPA to be created.
---

## Policy Definition
<a href="https://github.com/kyverno/policies/raw/main//other/verify-vpa-target/verify-vpa-target.yaml" target="-blank">/other/verify-vpa-target/verify-vpa-target.yaml</a>

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-vpa-target
  annotations:
    policies.kyverno.io/title: Verify VerticalPodAutoscaler Target
    policies.kyverno.io/category: Other
    policies.kyverno.io/severity: medium
    kyverno.io/kyverno-version: 1.11.4
    kyverno.io/kubernetes-version: "1.27"
    policies.kyverno.io/subject: VerticalPodAutoscaler
    policies.kyverno.io/description: >-
      VerticalPodAutoscaler (VPA) is useful to automatically adjust the resources assigned to Pods.
      It requires defining a specific target resource by kind and name. There are no built-in
      validation checks by the VPA controller to ensure that the target resource exists or that the target
      kind is specified correctly. This policy contains two rules, the first of which verifies that the
      kind is specified exactly as Deployment, StatefulSet, ReplicaSet, or DaemonSet, which helps avoid typos.
      The second rule verifies that the target resource exists before allowing the VPA to be created.
spec:
  validationFailureAction: Audit
  background: false
  rules:
  - name: verify-kind-name
    match:
      any:
      - resources:
          kinds:
          - VerticalPodAutoscaler
          operations:
          - CREATE
    validate:
      message: >-
        The target kind must be specified exactly as Deployment, StatefulSet, ReplicaSet, or DaemonSet.
      pattern:
        spec:
          targetRef:
            kind: Deployment | StatefulSet | ReplicaSet | DaemonSet
  - name: check-targetref
    match:
      any:
      - resources:
          kinds:
          - VerticalPodAutoscaler
          operations:
          - CREATE
    preconditions:
      all:
      - key:
        - Deployment
        - StatefulSet
        - ReplicaSet
        - DaemonSet
        operator: AnyIn
        value: "{{ request.object.spec.targetRef.kind }}"
    context:
    # Builds a mapping of the target kind to the plural form of the resource to be used in the API call.
    - name: map
      variable:
        value:
          Deployment: deployments
          StatefulSet: statefulsets
          ReplicaSet: replicasets
          DaemonSet: daemonsets
    - name: targetkind
      variable:
        jmesPath: request.object.spec.targetRef.kind
    - name: targets
      apiCall:
        urlPath: "/apis/apps/v1/namespaces/{{ request.namespace }}/{{ map.{{targetkind}} }}"
        jmesPath: "items[].metadata.name"
    validate:
      message: >-
        The target {{ request.object.spec.targetRef.kind }} named
        {{ request.object.spec.targetRef.name }} does not exist in the
        {{ request.namespace }} namespace.
      deny:
        conditions:
          all:
          - key: "{{ request.object.spec.targetRef.name }}"
            operator: AnyNotIn
            value: "{{ targets }}"

```
