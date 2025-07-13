CLUSTER_NAME=persistence-volume-cluster
CONFIG_FILE=k3d-config.yaml
K3D_TEMPLATE=k3d-config-template.yaml
ROOT_DIR := $(realpath $(dir $(lastword $(MAKEFILE_LIST))))

.DEFAULT_GOAL := help

.PHONY: help create delete get.nodes describe.cluster restart kubeconfig \
        build.push.deploy delete.all

help:
	@echo "\n\033[1;33mAvailable commands:\033[0m" \
	    && awk -F ':.*## ' '/^[a-zA-Z0-9_.-]+:.*## / {printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2}' $(MAKEFILE_LIST) \
	    | sort && echo ""

.PHONY: generate.k3d.config
generate.k3d.config:
	@echo "[INFO] Generating config for $(CLUSTER_NAME)"
	@CLUSTER_NAME="$(CLUSTER_NAME)" ROOT_DIR="$(ROOT_DIR)" \
		envsubst < $(K3D_TEMPLATE) > $(CONFIG_FILE)
	@echo "[INFO] Config generated: $(CONFIG_FILE)"

ensure.volume:
	mkdir -p volumes/test

create.cluster: ensure.volume generate.k3d.config ## Create K3d cluster with custom registry
	@echo "[INFO] Started creation of cluster $(CLUSTER_NAME)"
	@k3d cluster create --config $(CONFIG_FILE)
	@rm k3d-config.yaml
	@echo "[INFO] Completed creation of cluster $(CLUSTER_NAME)"

calico.init: ## Initial Calico CNI 
	@echo "[INFO] Started initialization of Calico CNI"
	@kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/calico.yaml
	@echo "[INFO] Completed initialization of Calico CNI"

calico.test: ## Initial Calico CNI pods to be active
	@kubectl wait --for=condition=Ready pod --all -n kube-system --timeout=180s

delete.cluster: ## Delete the k3d cluster
	@k3d cluster delete $(CLUSTER_NAME)

get.nodes: ## List Kubernetes nodes
	@kubectl get nodes -o wide

describe.cluster: ## Describe the k3d cluster
	@k3d cluster list $(CLUSTER_NAME)

kubeconfig: ## Merge kubeconfig and switch context
	k3d kubeconfig merge $(CLUSTER_NAME) --switch-context

delete.all: delete.cluster  ## Cleanup everything
	@echo "[INFO] Cleaning all the resources created"
	@rm -fr volumes

pvc.init: ## initialze the Persistence volume and Claim
	@kubectl apply -f manifest/pv-pvc-test.yaml

deploy.test: ## Deploy a test pod to validate the volume mount 
	@echo "[INFO] Deploying a test pod to validate the external folder mapping into K8s mapping."
	@kubectl apply -f manifest/pod-test.yaml

verify:
	@echo "[INFO] Verifying PVC Status..."
	@kubectl get pvc test-pvc
	@echo ""
	@echo "[INFO] VVerifying PV Status..."
	@echo ""
	@kubectl get pv test-pv
	@echo ""
	@echo "[INFO] VVerifying Pod Status..."
	@kubectl get pod pvc-test-pod
	@echo ""
	@echo "[INFO] Checking file inside the pod:"
	-kubectl exec pvc-test-pod -- cat /data/hello.txt || echo "[ERROR] File not found in pod"

	@echo "[INFO] Checking file on host volume:"
	@if [ -f volumes/test/hello.txt ]; then \
		echo "[INFO] Host file found: "; \
		cat volumes/test/hello.txt; \
	else \
		echo "[ERROR] Host file not found: volumes/test/hello.txt"; \
	fi


init: create.cluster calico.init calico.test pvc.init

