# about-kind

## Get

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64
chmod +x ./kind
sudo mv kind /usr/local/bin/kind
```

### Quick Creation

```bash
kind create cluster --name my-k8s
kubectl cluster-info --context kind-my-k8s
```

### Change Context / Cluster

```bash
kubectl config get-contexts
kubectl config use-contexts kind-<cluster_name>
```


## Create Cluster
Create cluster exposing ports for nginx:

```bash
kind create cluster --config cluster.yaml
```

Create pods, service and Nginx Ingress Deployments

```bash
kubectl apply -f manifest.yaml
```

## Delete clusters

```bash
kind delete clusters kind-<cluster_name>
```
