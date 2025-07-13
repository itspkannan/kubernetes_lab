## 🚀 Local Kubernetes Cluster with K3d + Custom Registry + Calico CNI + NGINX 

This project bootstraps a local Kubernetes environment using [K3d](https://k3d.io/) with an example of using local file system for PVC.
  

## 📁 Project Structure

```bash
.
├── k3d-config.yaml               # K3d cluster definition (1 server, 2 agents + custom registry)
├── Makefile                      # CLI to manage cluster and app deployment
├── README.md                     # This file
```


## 📋 Cluster Details

* **Cluster Name**: `k3d-persistence_volume_cluster`
* **Config File**: `k3d-config.yaml`


## 🛠️ Makefile Commands

Run `make help` to see all available commands:

```bash
❯ make help

Available commands:
  calico.init     Initial Calico CNI
  calico.test     Initial Calico CNI pods to be active
  create.cluster  Create K3d cluster with custom registry
  delete.all      Cleanup everything
  delete.cluster  Delete the k3d cluster
  describe.cluster Describe the k3d cluster
  get.nodes       List Kubernetes nodes
  kubeconfig      Merge kubeconfig and switch context
  pvc.init        initialze the Persistence volume and Claim

```


This will:

* Create a K3d cluster
* Mount a local volume
* Create PV and PVC

![init](./init_cluster.gif)

## Summary of PVC, PV manifest configuration 


1. **Set `volumeName: test-pv`** in the `PersistentVolumeClaim` to explicitly bind it to the manually created `PersistentVolume`.
2. **Add `storageClassName: ""`** in the PVC to disable dynamic provisioning (which defaults to `local-path`).
3. **Create the `volumes/test/` folder on the host** before starting the K3d cluster to ensure the `hostPath` mapping works correctly.


### ✅ What the Pod Does

* The pod runs a simple `busybox` container that executes:

  ```sh
  echo Hello from PVC > /data/hello.txt && sleep 3600
  ```

* It writes the message `"Hello from PVC"` to `/data/hello.txt`, where `/data` is mounted from the PVC.

* The pod then sleeps for 1 hour to keep the container alive, allowing inspection.

1. **Check PVC Status**

   ```bash
   kubectl get pvc test-pvc
   ```

   - Should be `Bound`.

2. **Check PV Status**

   ```bash
   kubectl get pv test-pv
   ```

   - Should be `Bound`.

3. **Check Pod Status**

   ```bash
   kubectl get pod pvc-test-pod
   ```

   - Should be `Running`.

4. **Verify File Inside the Pod**

   ```bash
   kubectl exec pvc-test-pod -- cat /data/hello.txt
   ```

   - Output should be: `Hello from PVC`.

5. **Verify File on Host (K3d-mapped volume)**

   ```bash
   cat volumes/test/hello.txt
   ```

   - File should exist with correct content.

![verify](./pvc_pod_test.gif)

## 🧹 Delete Deployment and Cluster

```bash
❯ make delete.all           # Delete the full cluster
```
