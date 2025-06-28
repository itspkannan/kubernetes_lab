## 🚀 Local Kubernetes Cluster with K3d + Custom Registry + Calico CNI + NGINX 

This project bootstraps a local Kubernetes environment using [K3d](https://k3d.io/), runs a **local Docker registry**, and deploys a simple NGINX service using manifests. The NGINX image is pulled from your custom registry, simulating private image usage in real-world Kubernetes deployments.

This K3D setup uses Calico as the CNI by disabling Flannel.
  

## 📁 Project Structure

```bash
.
├── docker-compose.yaml             # Docker registry container
├── k3d-config.yaml               # K3d cluster definition (1 server, 2 agents + custom registry)
├── Makefile                      # CLI to manage cluster and app deployment
├── README.md                     # This file
└── nginx-manifest/               # Kubernetes manifests
    ├── namespace.yaml
    ├── deployment.yaml
    └── service.yaml
```


## 📋 Cluster Details

* **Cluster Name**: `k3d-calico-cni-cluster`
* **Registry Name**: `calico-cni-cluster-registry` (mapped to `localhost:5000`)
* **Config File**: `k3d-config.yaml`
* **Ports**:

  * Exposes `localhost:8080 → service:80`
  * Exposes local Docker registry on `localhost:5000`


## 🛠️ Makefile Commands

Run `make help` to see all available commands:

```bash
❯ make help

Available commands:
  build.push      Build, push nginx image
  calico.init     Initial Calico CNI
  create          Create K3d cluster with custom registry
  create.namespace Create Kubernetes namespace
  create.network  Create a K3d network
  delete          Delete the k3d cluster
  delete.all      Cleanup everything
  delete.namespace Delete nginx namespace
  delete.network  Delete a K3d network
  deploy.nginx    Deploy NGINX using manifests
  deploy.view     View nginx pods, svc
  describe.cluster Describe the k3d cluster
  get.nodes       List Kubernetes nodes
  kubeconfig      Merge kubeconfig and switch context
  nginx           Delete nginx deployment/service
  restart         Recreate the cluster
  start.registry  Start Docker registry using Compose
  stop.registry   Stop Docker registry
  test.nginx      Test nginx availability
```

## 🧰 Create Cluster with Local Registry

```bash
❯ make start.registry create
docker compose up -d
[+] Running 2/2
 ✔ Network k3d_with_custom_registry_default  Created                                                                                                                                                                                            0.0s
 ✔ Container registry                        Started                                                                                                                                                                                            0.1s
k3d cluster create --config k3d-config.yaml
INFO[0000] Using config file k3d-config.yaml (k3d.io/v1alpha5#simple)
INFO[0000] portmapping '8080:80' lacks a nodefilter, but there's more than one node: defaulting to [servers:*:proxy agents:*:proxy]
INFO[0000] Prep: Network
INFO[0000] Created network 'k3d-calico-cni-cluster'
INFO[0000] Created image volume k3d-calico-cni-cluster-images
INFO[0000] Starting new tools node...
INFO[0000] Starting node 'k3d-calico-cni-cluster-tools'
INFO[0001] Creating node 'k3d-calico-cni-cluster-server-0'
INFO[0001] Creating node 'k3d-calico-cni-cluster-agent-0'
INFO[0001] Creating node 'k3d-calico-cni-cluster-agent-1'
INFO[0001] Creating LoadBalancer 'k3d-calico-cni-cluster-serverlb'
INFO[0001] Using the k3d-tools node to gather environment information
INFO[0001] Starting new tools node...
INFO[0001] Starting node 'k3d-calico-cni-cluster-tools'
INFO[0002] Starting cluster 'calico-cni-cluster'
INFO[0002] Starting servers...
INFO[0002] Starting node 'k3d-calico-cni-cluster-server-0'
INFO[0004] Starting agents...
INFO[0004] Starting node 'k3d-calico-cni-cluster-agent-1'
INFO[0004] Starting node 'k3d-calico-cni-cluster-agent-0'
INFO[0016] Starting helpers...
INFO[0016] Starting node 'k3d-calico-cni-cluster-serverlb'
INFO[0023] Injecting records for hostAliases (incl. host.k3d.internal) and for 5 network members into CoreDNS configmap...

INFO[0025] Cluster 'calico-cni-cluster' created successfully!
INFO[0025] You can now use it like this:
kubectl cluster-info
```

This will:

* Create a K3d cluster
* Start a local Docker registry named `k3d-registry`
* Wire the registry to the cluster's containerd runtime


## 🐳 Build & Push Custom NGINX Image

```bash
❯ make build.push
❯ make build.push
docker pull nginx:latest
latest: Pulling from library/nginx
Digest: sha256:dc53c8f25a10f9109190ed5b59bda2d707a3bde0e45857ce9e1efaa32ff9cbc1
Status: Image is up to date for nginx:latest
docker.io/library/nginx:latest

What's next:
    View a summary of image vulnerabilities and recommendations → docker scout quickview nginx:latest
docker tag nginx:latest localhost:5000/nginx:latest
docker push localhost:5000/nginx:latest
The push refers to repository [localhost:5000/nginx]
150c79212ae2: Pushed
9f93380e082e: Pushed
c4b50375dda1: Pushed
66d61f488355: Pushed
0445bbc338d6: Pushed
5bce65060a94: Pushed
6edfb9bfff29: Pushed
latest: digest: sha256:199d5bb3851dedec36e55044b594ede90c4e914a93abacbd76055a647eb3da59 size: 1778
```

This will:

1. Pull the official `nginx:latest`
2. Tag and push it to your local registry at `localhost:5000/nginx:latest`


## 🚀 NGINX Deployment



```bash
❯ make deploy.nginx
kubectl apply -f nginx-manifest/namespace.yaml
namespace/nginx-app created
kubectl apply -f nginx-manifest/deployment.yaml
deployment.apps/nginx-deployment created
kubectl apply -f nginx-manifest/service.yaml
service/nginx-service created

❯ make deploy.view
kubectl get pods -n nginx-app
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5cd97745bd-5dtwt   1/1     Running   0          3s
nginx-deployment-5cd97745bd-9k4vk   1/1     Running   0          3s
kubectl get svc -n nginx-app
NAME            TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-service   LoadBalancer   10.43.77.125   <pending>     80:31318/TCP   3s
```


You should see:

* 2 running NGINX pods
* LoadBalancer service exposing port 80



### Test the NGINX App

```bash
❯ make test.nginx
curl --fail http://localhost:8080 || echo "NGINX not reachable"
```

You should see the default NGINX welcome page.

Add hostname `registry` to `/etc/hosts` and then push the image with tag `registry:5000/nginx:latest` and also ensure that the image name is also updated in deployment.


## 🧹 Delete Deployment and Cluster

```bash
❯ make delete.nginx     # Delete NGINX deployment, service, and namespace
❯ make delete           # Delete the full cluster
```
