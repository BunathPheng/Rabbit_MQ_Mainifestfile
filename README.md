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

If pods stay `Pending` with `FailedScheduling`, this cluster has only one untainted worker. The manifest now prefers spreading pods and tolerates control-plane taints so `rabbitmq-server-0/1/2` can start.

The PVC message `the object has been modified` is a retryable bind race. It should clear after the pods can schedule.

## Files

| Path | Role |
|------|------|
| `apps/cert-manager/` | cert-manager v1.21.1 |
| `apps/rabbitmq-operator/` | Cluster Operator v2.22.4 |
| `RabbitmqCluster.yaml` | 3-node RabbitMQ cluster |
| `ingress.yaml` | Management UI Ingress (`https://rabbitmq.smartops.space`) |
| `namespace.yaml` | `rabbitmq` namespace |

## Access the management UI

The operator Service is `ClusterIP`, so it is not reachable from outside the cluster by itself. `ingress.yaml` exposes the UI on port `15672`.

1. You need an Ingress controller (this manifest uses `ingressClassName: nginx`).
2. Point DNS `rabbitmq.smartops.space` at the Ingress IP.
3. cert-manager issues a Let's Encrypt certificate after DNS is live.

```bash
kubectl get ingress -n rabbitmq
kubectl get certificate -n rabbitmq
kubectl get secret rabbitmq-default-user -n rabbitmq -o jsonpath='{.data.username}' | base64 -d
kubectl get secret rabbitmq-default-user -n rabbitmq -o jsonpath='{.data.password}' | base64 -d
```

Open `https://rabbitmq.smartops.space` and log in with those credentials.

AMQP (`5672`) is TCP, not HTTP. Ingress does not expose it. Use port-forward or a TCP LoadBalancer for producers/consumers:

```bash
kubectl port-forward -n rabbitmq svc/rabbitmq 5672:5672
```
