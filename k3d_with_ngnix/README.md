
# 🚀 Local Kubernetes Cluster with K3d + NGINX

This project bootstraps a local Kubernetes environment using [K3d](https://k3d.io/) and deploys a simple NGINX service via manifests. It's designed for learning and quick experimentation.


## 📁 Project Structure

```bash
.
├── k3d-config.yaml               # K3d cluster definition (1 server, 2 agents + custom registry)
├── Makefile                      # CLI to manage cluster and app deployment
├── README.md                     # This file
└── ngnix-manifest/               # Kubernetes manifests
    ├── namespace.yaml
    ├── deployment.yaml
    └── service.yaml
```

## 📋 Cluster Details

- **Cluster Name**: `k3d-ngnix-cluster`
- **Config File**: `k3d-config.yaml`
- **Ports**: Exposes `localhost:8080 → service:80`


## 🛠️ Makefile Commands

Run `make help` to see all commands:


```bash
❯ make help


Available commands:
  create           Create K3d cluster from config file
  create.namespace Create Kubernetes namespace
  delete           Delete the K3d cluster
  delete.namespace Delete Kubernetes namespace
  delete.nginx     Delete NGINX resources
  deploy.nginx     Deploy NGINX using manifests
  deploy.view      View ngnix pods, svc
  describe.cluster Describe the K3d cluster
  get.nodes        Get Kubernetes nodes
  help             Help message
  kubeconfig       Merge K3d kubeconfig into local ~/.kube/config
  restart          Restart cluster (delete + create)
  test.nginx       Test NGINX via curl

```

## Create Cluster

```bash
❯ make create
k3d cluster create --config k3d-config.yaml
INFO[0000] Using config file k3d-config.yaml (k3d.io/v1alpha5#simple)
INFO[0000] portmapping '8080:80' targets the loadbalancer: defaulting to [servers:*:proxy agents:*:proxy]
INFO[0000] Prep: Network
INFO[0000] Created network 'k3d-k3d-ngnix-cluster'
INFO[0000] Created image volume k3d-k3d-ngnix-cluster-images
INFO[0000] Starting new tools node...
INFO[0000] Starting node 'k3d-k3d-ngnix-cluster-tools'
INFO[0001] Creating node 'k3d-k3d-ngnix-cluster-server-0'
INFO[0001] Creating node 'k3d-k3d-ngnix-cluster-agent-0'
INFO[0001] Creating node 'k3d-k3d-ngnix-cluster-agent-1'
INFO[0001] Creating LoadBalancer 'k3d-k3d-ngnix-cluster-serverlb'
INFO[0001] Using the k3d-tools node to gather environment information
INFO[0001] Starting new tools node...
INFO[0001] Starting node 'k3d-k3d-ngnix-cluster-tools'
INFO[0002] Starting cluster 'k3d-ngnix-cluster'
INFO[0002] Starting servers...
INFO[0002] Starting node 'k3d-k3d-ngnix-cluster-server-0'
INFO[0004] Starting agents...
INFO[0005] Starting node 'k3d-k3d-ngnix-cluster-agent-0'
INFO[0005] Starting node 'k3d-k3d-ngnix-cluster-agent-1'
INFO[0016] Starting helpers...
INFO[0017] Starting node 'k3d-k3d-ngnix-cluster-serverlb'
INFO[0023] Injecting records for hostAliases (incl. host.k3d.internal) and for 5 network members into CoreDNS configmap...
INFO[0025] Cluster 'k3d-ngnix-cluster' created successfully!
INFO[0025] You can now use it like this:
kubectl cluster-info

```

## 🚀 NGINX deployment

### Deploy and verify ngnix
```bash
❯ make deploy.nginx
kubectl apply -f ngnix-manifest/namespace.yaml
namespace/nginx-app created
kubectl apply -f ngnix-manifest/deployment.yaml
deployment.apps/nginx-deployment created
kubectl apply -f ngnix-manifest/service.yaml
service/nginx-service created

❯ make deploy.view
kubectl get pods -n nginx-app
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-54b9c68f67-rc2fn   1/1     Running   0          16s
nginx-deployment-54b9c68f67-xk2c9   1/1     Running   0          16s
kubectl get svc -n nginx-app
NAME            TYPE           CLUSTER-IP    EXTERNAL-IP                        PORT(S)        AGE
nginx-service   LoadBalancer   10.43.13.44   172.19.0.3,172.19.0.4,172.19.0.5   80:30285/TCP   17s

❯ make test.nginx
curl --fail http://localhost:8080 || echo "NGINX not reachable"
....
....
```

You should see the default NGINX page content via:

### 🧹 Delete deployment and Cluster

```bash
make delete.nginx
❯ make delete.nginx
kubectl delete -f ngnix-manifest/service.yaml --ignore-not-found
service "nginx-service" deleted
kubectl delete -f ngnix-manifest/deployment.yaml --ignore-not-found
deployment.apps "nginx-deployment" deleted
kubectl delete -f ngnix-manifest/namespace.yaml
namespace "nginx-app" deleted

❯ make delete
k3d cluster delete k3d-ngnix-cluster
INFO[0000] Deleting cluster 'k3d-ngnix-cluster'
INFO[0001] Deleting cluster network 'k3d-k3d-ngnix-cluster'
INFO[0001] Deleting 1 attached volumes...
INFO[0001] Removing cluster details from default kubeconfig...
INFO[0001] Removing standalone kubeconfig file (if there is one)...
INFO[0001] Successfully deleted cluster k3d-ngnix-cluster!
```


## 🧠 Kubectl command 

Use these `kubectl` commands to explore further:

```bash
kubectl get nodes -o wide
kubectl get deployments
kubectl get pods
kubectl get svc
kubectl describe svc nginx-service
```

