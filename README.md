# RabbitMQ GitOps (Argo CD + Operator)

Git holds the desired state. Argo CD syncs it to Kubernetes. The RabbitMQ Cluster Operator creates and manages a 3-node broker.

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
  rabbitmq-server-0  rabbitmq-server-1  rabbitmq-server-2
```

The operator names pods `{cluster}-server-N`. With `metadata.name: rabbitmq` that is `rabbitmq-server-0`, `rabbitmq-server-1`, and `rabbitmq-server-2`.

## Layout

| Path | Role |
|------|------|
| `apps/rabbitmq-operator/` | Installs RabbitMQ Cluster Operator v2.22.4 |
| `apps/rabbitmq-cluster/RabbitmqCluster.yaml` | 3-replica `RabbitmqCluster` custom resource |
| `argocd/apps/rabbitmq-operator.yaml` | Argo CD app for the operator |
| `argocd/apps/rabbitmq-cluster.yaml` | Argo CD app for the cluster |
| `argocd/root-app.yaml` | App-of-Apps bootstrap |

## Prerequisites

- Kubernetes cluster with **3 worker nodes** (pod anti-affinity requires one RabbitMQ pod per node)
- A default StorageClass for PersistentVolumeClaims
- Argo CD installed in the `argocd` namespace

If you have fewer than 3 nodes, change `requiredDuringSchedulingIgnoredDuringExecution` in `RabbitmqCluster.yaml` to `preferredDuringSchedulingIgnoredDuringExecution` or the extra pods will stay `Pending`.

## Deploy

Apply the root app once. Argo CD then creates the operator app, then the cluster app.

```bash
kubectl apply -f argocd/root-app.yaml
```

Or apply the two child apps directly:

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

Expected broker pods:

```
rabbitmq-server-0
rabbitmq-server-1
rabbitmq-server-2
```

## Credentials and UI

The operator creates a default-user Secret:

```bash
kubectl get secret rabbitmq-default-user -n rabbitmq -o jsonpath='{.data.username}' | base64 -d
kubectl get secret rabbitmq-default-user -n rabbitmq -o jsonpath='{.data.password}' | base64 -d
```

Management UI (in-cluster service `rabbitmq` on port `15672`):

```bash
kubectl port-forward -n rabbitmq svc/rabbitmq 15672:15672
```

AMQP is on port `5672` of the same service.

## Cluster settings

- 3 replicas so quorum queues keep a majority if one node fails
- Persistent storage so messages survive pod restarts
- `pause_minority` partition handling
- Default queue type `quorum`
- Anti-affinity so two broker pods do not land on the same node
- Cluster app uses `prune: false` so Argo CD does not delete broker data on a sync
