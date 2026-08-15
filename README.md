# RabbitMQ GitOps (Argo CD + Operator)

```
                 Git
                  │
          RabbitmqCluster.yaml
                  │
                  ▼
                ArgoCD
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

Git stores `RabbitmqCluster.yaml`. Argo CD applies it to Kubernetes. The RabbitMQ Cluster Operator reads that CR and creates the 3-node broker.

The operator always names pods `{cluster}-server-N`, so the three nodes appear as:

| Diagram | Kubernetes pod |
|---------|----------------|
| rabbitmq-0 | `rabbitmq-server-0` |
| rabbitmq-1 | `rabbitmq-server-1` |
| rabbitmq-2 | `rabbitmq-server-2` |

That `-server` suffix is fixed by the operator and cannot be removed.

## Layout

| Path | Role |
|------|------|
| `RabbitmqCluster.yaml` | 3-replica cluster (the Git source of truth) |
| `namespace.yaml` | `rabbitmq` namespace |
| `apps/rabbitmq-operator/` | RabbitMQ Cluster Operator v2.22.4 |
| `argocd/apps/rabbitmq-operator.yaml` | Argo CD app for the operator |
| `argocd/apps/rabbitmq-cluster.yaml` | Argo CD app for `RabbitmqCluster.yaml` |
| `argocd/root-app.yaml` | App-of-Apps bootstrap |

## Prerequisites

- Kubernetes cluster with **3 worker nodes** (anti-affinity places one broker pod per node)
- A default StorageClass
- Argo CD in the `argocd` namespace

If you have fewer than 3 nodes, change `requiredDuringSchedulingIgnoredDuringExecution` in `RabbitmqCluster.yaml` to `preferredDuringSchedulingIgnoredDuringExecution` or extra pods stay `Pending`.

## Deploy

```bash
kubectl apply -f argocd/root-app.yaml
```

Or apply the child apps directly:

```bash
kubectl apply -f argocd/apps/rabbitmq-operator.yaml
kubectl apply -f argocd/apps/rabbitmq-cluster.yaml
```

## Verify

```bash
kubectl get pods -n rabbitmq-system
kubectl get rabbitmqcluster -n rabbitmq
kubectl get pods -n rabbitmq -o wide
```

Expected broker pods: `rabbitmq-server-0`, `rabbitmq-server-1`, `rabbitmq-server-2`.

## Credentials and UI

```bash
kubectl get secret rabbitmq-default-user -n rabbitmq -o jsonpath='{.data.username}' | base64 -d
kubectl get secret rabbitmq-default-user -n rabbitmq -o jsonpath='{.data.password}' | base64 -d
kubectl port-forward -n rabbitmq svc/rabbitmq 15672:15672
```

AMQP is on port `5672` of Service `rabbitmq`.
