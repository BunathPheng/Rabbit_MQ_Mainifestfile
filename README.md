# RabbitMQ cluster manifest

Argo CD is **not** installed by this repo. It is an existing tool in your Kubernetes cluster.

This repo only stores the desired RabbitMQ cluster. Your existing Argo CD watches Git and syncs `RabbitmqCluster.yaml`.

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
                  ▼
        RabbitMQ Operator
                  │
                  ▼
       Creates/manages RabbitMQ
          ┌───────┼───────┐
          ▼       ▼       ▼
       rabbitmq-0 rabbitmq-1 rabbitmq-2
```

## What each part is

| Piece | Where it lives | This repo? |
|-------|----------------|------------|
| Argo CD | Already installed in your cluster | No |
| `RabbitmqCluster.yaml` | This Git repo | Yes |
| RabbitMQ Operator | Already installed, or install once from `apps/rabbitmq-operator` | Optional |
| `rabbitmq-server-0/1/2` | Created by the operator | No (runtime) |

The operator names pods `rabbitmq-server-0`, `rabbitmq-server-1`, `rabbitmq-server-2`. Those are the three nodes in the diagram.

## Connect existing Argo CD to this repo

In Argo CD (UI or CLI), create an Application that points at Git. Do **not** keep Argo CD Application YAML in this repo.

- **Repo:** `https://github.com/BunathPheng/Rabbit_MQ_Mainifestfile.git`
- **Revision:** `main`
- **Path:** `.`
- **Destination namespace:** `rabbitmq`

That is the only Argo CD setup. Argo CD then applies `RabbitmqCluster.yaml` on every Git change.

## Files

| Path | Role |
|------|------|
| `RabbitmqCluster.yaml` | 3-node RabbitMQ cluster |
| `namespace.yaml` | `rabbitmq` namespace |
| `apps/rabbitmq-operator/` | Operator install, only if it is not already on the cluster |

## Verify

```bash
kubectl get rabbitmqcluster -n rabbitmq
kubectl get pods -n rabbitmq -o wide
```
