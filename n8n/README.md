# n8n Installation in k8s step by step
## Install postgres and redis database
1. **Install PostgreSQL**
We will install it without changing the default values.
```bash
helm install postgres 
    --namespace postgres
    --create-namespace
    oci://registry-1.docker.io/bitnamicharts/postgresql
```
Note: 
* The server pod is deployed through statefulset. A PVC of size 8 Gi is created using default storage class.
* Two services of `ClusterIP` type are created, but only one of them is active. PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster
```
postgres-postgresql.postgres.svc.cluster.local
```
* A random password for `postgres` user is created and stored in `postgres-postgresql` secret. We can retrieve this password using the following command.
```bash
kubectl get secret 
    --namespace postgres postgres-postgresql 
    -o jsonpath="{.data.postgres-password}" | base64 -d
```
2. **Install Redis**
Simmilar to PostgreSQL, we will install redis with standalone architecture
```bash
helm install redis 
    --namespace redis
    --create-namespace
    --set architecture=standalone
    oci://registry-1.docker.io/bitnamicharts/redis
```
Note: Like PosgreSQL:
* redis host : `redis-master.redis.svc.cluster.local`
* Retrieve password by this command:
```bash
kubectl get secret 
    --namespace redis redis 
    -o jsonpath="{.data.redis-password}" | base64 -d
```
## Install n8n in k8s
1. Create secret password for store PostgreSQL password
```bash
kubectl create namespace n8n
kubectl create secret generic postgres-secret 
    --namespace n8n
    --from-literal=password=<postgres-password>
```
2. Using this values file: `custom-values.yaml`
```yaml
image:
  repository: n8nio/n8n
  pullPolicy: IfNotPresent
  tag: ""
imagePullSecrets: []
nameOverride:
fullnameOverride:
hostAliases: []
ingress:
  enabled: false
  annotations: {}
  className: ""
  hosts:
    - host: workflow.example.com
      paths: []
  tls:
    - hosts:
        - workflow.example.com
      secretName: host-domain-cert
main:
  config:
    db:
      type: postgresdb
      postgresdb:
        host: postgres-postgresql.postgres.svc.cluster.local
        user: postgres
  secret:
    n8n:
      encryption_key: "a-random-string"
  extraEnv: &extraEnv
    DB_POSTGRESDB_PASSWORD:
      valueFrom:
        secretKeyRef:
          name: postgres-secret
          key: password
          
worker:
  enabled: false

webhook:
  enabled: false

extraManifests: []
extraTemplateManifests: []

valkey:
  enabled: false
```
2. Install n8n with this values
```bash
helm install n8n 
    --namespace n8n
    --create-namespace
    --values custom-values.yaml
    oci://8gears.container-registry.com/library/n8n
```
### REFERENCES
https://artifacthub.io/packages/helm/bitnami/postgresql
https://artifacthub.io/packages/helm/bitnami/redis 
https://blog.devgenius.io/postgres-helm-installation-ad30e9056018
