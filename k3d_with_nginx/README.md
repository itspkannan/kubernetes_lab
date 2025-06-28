
# 🚀 Local Kubernetes Cluster with K3d + NGINX

This project bootstraps a local Kubernetes environment using [K3d](https://k3d.io/) and deploys a simple NGINX service via manifests. It's designed for learning and quick experimentation.


## 📁 Project Structure

```bash
.
├── k3d-config.yaml               # K3d cluster definition (1 server, 2 agents + custom registry)
├── Makefile                      # CLI to manage cluster and app deployment
├── README.md                     # This file
└── nginx-manifest/               # Kubernetes manifests
    ├── namespace.yaml
    ├── deployment.yaml
    └── service.yaml
```

## 📋 Cluster Details

- **Cluster Name**: `k3d-nginx-cluster`
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
  deploy.view      View nginx pods, svc
  describe.cluster Describe the K3d cluster
  get.nodes        Get Kubernetes nodes
  help             Help message
  kubeconfig       Merge K3d kubeconfig into local ~/.kube/config
  restart          Restart cluster (delete + create)
  test.nginx       Test NGINX via curl

```

## Create Cluster

```bash
❯ make create.network start.registry create calico.init calico.test
docker network create k3d-calico-cni-cluster
a52a3e2287a32cff00765dfa2f831edae9c880e5588494850cc00ebdd01c81dd
docker compose up -d
[+] Running 1/1
 ✔ Container calico-cni-cluster-registry  Started                                                                                                                                                                                               0.1s
k3d cluster create --config k3d-config.yaml
INFO[0000] Using config file k3d-config.yaml (k3d.io/v1alpha5#simple)
INFO[0000] portmapping '8080:80' targets the loadbalancer: defaulting to [servers:*:proxy agents:*:proxy]
INFO[0000] Prep: Network
INFO[0000] Re-using existing network 'k3d-calico-cni-cluster' (a52a3e2287a32cff00765dfa2f831edae9c880e5588494850cc00ebdd01c81dd)
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
INFO[0004] Starting node 'k3d-calico-cni-cluster-agent-0'
INFO[0004] Starting node 'k3d-calico-cni-cluster-agent-1'
INFO[0016] Starting helpers...
INFO[0016] Starting node 'k3d-calico-cni-cluster-serverlb'
INFO[0022] Injecting records for hostAliases (incl. host.k3d.internal) and for 6 network members into CoreDNS configmap...
INFO[0024] Cluster 'calico-cni-cluster' created successfully!
INFO[0024] You can now use it like this:
kubectl cluster-info
poddisruptionbudget.policy/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
serviceaccount/calico-node created
serviceaccount/calico-cni-plugin created
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpfilters.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrole.rbac.authorization.k8s.io/calico-cni-plugin created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-cni-plugin created
daemonset.apps/calico-node created
deployment.apps/calico-kube-controllers created
kubectl wait --for=condition=Ready pod --all -n kube-system --timeout=180s
pod/calico-kube-controllers-798f56bb9d-ll4l6 condition met
pod/calico-node-7z8jn condition met
pod/calico-node-ldz69 condition met
pod/calico-node-ps82t condition met
pod/coredns-ccb96694c-tdfkv condition met
pod/local-path-provisioner-5cf85fd84d-lqvhj condition met
pod/metrics-server-5985cbc9d7-lwjwg condition met
```

## 🚀 NGINX deployment

### Deploy and verify nginx
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
kubectl delete -f nginx-manifest/service.yaml --ignore-not-found
service "nginx-service" deleted
kubectl delete -f nginx-manifest/deployment.yaml --ignore-not-found
deployment.apps "nginx-deployment" deleted
kubectl delete -f nginx-manifest/namespace.yaml
namespace "nginx-app" deleted

❯ make delete
k3d cluster delete k3d-nginx-cluster
INFO[0000] Deleting cluster 'k3d-nginx-cluster'
INFO[0001] Deleting cluster network 'k3d-k3d-nginx-cluster'
INFO[0001] Deleting 1 attached volumes...
INFO[0001] Removing cluster details from default kubeconfig...
INFO[0001] Removing standalone kubeconfig file (if there is one)...
INFO[0001] Successfully deleted cluster k3d-nginx-cluster!
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

