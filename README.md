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
| RabbitMQ Operator | Installed from `apps/rabbitmq-operator` on the first Argo CD sync | Yes |
| `rabbitmq-server-0/1/2` | Created by the operator | No (runtime) |

The operator names pods `rabbitmq-server-0`, `rabbitmq-server-1`, `rabbitmq-server-2`. Those are the three nodes in the diagram.

## Connect existing Argo CD to this repo

In Argo CD (UI or CLI), create an Application that points at Git. Do **not** keep Argo CD Application YAML in this repo.

- **Repo:** `https://github.com/BunathPheng/Rabbit_MQ_Mainifestfile.git`
- **Revision:** `main`
- **Path:** `.`
- **Destination namespace:** `rabbitmq`

In Sync Options, also enable **Skip Dry Run on Missing Resource**. The first sync installs the operator CRD, then `RabbitmqCluster`.

If the first sync still fails on `RabbitmqCluster`, wait until the operator is Running and click **Sync** again:

```bash
kubectl get crd rabbitmqclusters.rabbitmq.com
kubectl get pods -n rabbitmq-system
```

## Files

| Path | Role |
|------|------|
| `RabbitmqCluster.yaml` | 3-node RabbitMQ cluster |
| `namespace.yaml` | `rabbitmq` namespace |
| `apps/rabbitmq-operator/` | RabbitMQ Cluster Operator + `RabbitmqCluster` CRD |

## Verify

```bash
kubectl get rabbitmqcluster -n rabbitmq
kubectl get pods -n rabbitmq -o wide
```
