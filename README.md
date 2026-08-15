# RabbitMQ cluster manifest

Argo CD is **not** installed by this repo. It is an existing tool in your Kubernetes cluster.

```
                 Git
                  │
          RabbitmqCluster.yaml
                  │
                  ▼
         Argo CD (already exists)
                  │
                  ▼
             Kubernetes
                  │
          ┌───────┴────────┐
          ▼                ▼
     cert-manager   RabbitMQ Operator
                           │
                           ▼
              rabbitmq-server-0/1/2
```

Latest operator (v2.22.4) needs cert-manager for webhook and metrics certificates.

Sync order:

1. cert-manager v1.21.1 (wave 0)
2. RabbitMQ Operator v2.22.4 (wave 1)
3. `RabbitmqCluster` (wave 2)

## Argo CD app

- **Repo:** `https://github.com/BunathPheng/Rabbit_MQ_Mainifestfile.git`
- **Revision:** `HEAD` / `main`
- **Path:** `.`
- **Namespace:** `rabbitmq`

Enable these Sync Options:

- Auto-Create Namespace
- Server-Side Apply
- Skip Dry Run on Missing Resource
- Retry

Then **Refresh** and **Sync**. If Certificate/Issuer still fail, wait until cert-manager is Running and Sync again:

```bash
kubectl get pods -n cert-manager
kubectl get crd certificates.cert-manager.io issuers.cert-manager.io
kubectl get pods -n rabbitmq-system
kubectl get rabbitmqcluster -n rabbitmq
kubectl get pods -n rabbitmq -o wide
```

## Files

| Path | Role |
|------|------|
| `apps/cert-manager/` | cert-manager v1.21.1 |
| `apps/rabbitmq-operator/` | Cluster Operator v2.22.4 |
| `RabbitmqCluster.yaml` | 3-node RabbitMQ cluster |
| `namespace.yaml` | `rabbitmq` namespace |
