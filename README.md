# 🚀 Learning Kubernetes with K3d

This repository is a personal workshop to learn Kubernetes locally using [K3d](https://k3d.io/), a lightweight wrapper to run K3s in Docker. Each subproject demonstrates a different aspect of Kubernetes operations in a local environment.

## 📁 Project Structure

```bash
.
├── k3d_with_ngnix/              # Deploy NGINX in a multi-node K3d cluster with Ingress access
├── k3d_with_calico_cni/         # Integrate Calico as a custom CNI in a K3d cluster
├── k3d_with_custom_registry/    # Use a local Docker registry with K3d
```

## 📦 Prerequisites

* Docker
* [k3d](https://k3d.io/#installation)
* kubectl
* make

## 📚 Projects Overview

### `k3d_with_ngnix`

> **Goal**: Create a K3d cluster with 1 server and 2 agents, deploy an NGINX service with an Ingress, and test it via `curl`.

* 🧱 Creates a K3d cluster using a `k3d-config.yaml`
* 📦 Deploys a basic NGINX Deployment, Service, and Ingress
* 🌐 Binds port `8080` on host to access NGINX via `localhost:8080`


### `k3d_with_calico_cni`

> **Goal**: Replace default K3d networking with [Calico CNI](https://docs.tigera.io/calico/latest/introduction/).

* Sets up Calico as the networking layer
* Demonstrates Calico pod networking and policy control (To be implemented)

### `k3d_with_custom_registry`

> **Goal**: Attach a custom local Docker registry to your K3d cluster.

* Automatically creates a registry container
* Links the registry with the K3d cluster
* Demonstrates pushing and pulling custom images


## 🧠 Learnings

* Efficient Kubernetes cluster bootstrapping with K3d
* Local development patterns with custom registries and volume mounts
* Managing CNIs like Calico
* Using Makefiles for repeatable infrastructure actions


## 📌 License

MIT – feel free to fork, modify, and extend.
