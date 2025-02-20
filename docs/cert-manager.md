# MinIO tenant with cert-manager [![Slack](https://slack.min.io/slack?type=svg)](https://slack.min.io)

This document explains how to deploy a MinIO tenant using certificates generated by [cert-manager](https://cert-manager.io/).

## Getting Started

### Prerequisites

- Kubernetes version `+v1.19`. While cert-manager supports [earlier K8s versions](https://cert-manager.io/docs/installation/supported-releases/), the MinIO Operator requires 1.19 or later.
- MinIO Operator installed
- `kubectl` access to your `k8s` cluster
- [cert-manager](https://cert-manager.io/docs/installation/) 1.7.X or later installed
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.7.2/cert-manager.yaml
```
- [kustomize](https://kustomize.io/) installed

### Deploy tenant

Use the example to deploy a `MinIO` tenant using tls certificates generated by `cert-manager`, go into base folder of
the operator project and run the following command.

```bash
kustomize build examples/kustomization/tenant-certmanager | kubectl apply -f -
```


This file request `cert-manager` to issue a new certificate based on the following internal domains.

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: tenant-certmanager-cert
  namespace: minio-tenant
spec:
  dnsNames:
    - "*.tenant-certmanager.svc.cluster.local"
    - "*.storage-certmanager.tenant-certmanager.svc.cluster.local"
    - "*.storage-certmanager-hl.tenant-certmanager.svc.cluster.local"
  secretName: tenant-certmanager-tls
  issuerRef:
    name: tenant-certmanager-issuer
```
Then it creates a new tenant including the new `tenant-certmanager-tls` secret in the `externalCertSecret` field.

```yaml
  ## Use certificates generated by cert-manager.
  externalCertSecret:
    - name: tenant-certmanager-tls
      type: cert-manager.io/v1
```