# External Secrets Integration

In order to use `external-secrets` Operator (ESO) to fetch secrets from Google Secret Manager at runtime, we can create a ClusterSecretStore
that defines the configuration for accessing GSM **Google Secret Manager (GSM)**. An external secrets resource can then reference this secret store to fetch secrets from 

### 1. ClusterSecretStore

Defines the provider configuration for accessing GSM. It defines how kubernetes connects to the external secret manager store (e.g. GSM).Below is a sample manifest for a ClusterSecretStore

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: gsm-cluster-store
spec:
  provider:
    gcpsm:
      projectID: boreal-mode-295611
      auth:
        workloadIdentity:
          clusterLocation: europe-west1
          clusterName: boreal-mode-295611-gke-test
          serviceAccountRef:
            name: eso-gcp-sa
            namespace: external-secrets
```

Fetches username and password secrets from GSM and creates a Kubernetes native Secret with the same name. This secret can
can then be mounted to a kubernetes deployment object or in a pod. Below is a sample manifest for ExternalSecret.
```yaml
### 1. ExternalSecret
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: external-db-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: gsm-cluster-store
    kind: ClusterSecretStore
  target:
    name: external-db-credentials
    creationPolicy: Owner
  data:
    - secretKey: username
      remoteRef:
        key: test/db-credentials
        version: latest
    - secretKey: password
      remoteRef:
        key: test/db-credentials
        version: latest
```

