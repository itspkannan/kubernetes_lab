CLUSTER_NAME=custom-registry-cluster
CONFIG_FILE=k3d-config.yaml
DEFAULT_REG=localhost:5000
IMAGE_NAME=$(DEFAULT_REG)/nginx:latest

.DEFAULT_GOAL := help

.PHONY: help create delete get.nodes describe.cluster restart kubeconfig \
        build.push.deploy delete.all test.nginx start.registry stop.registry deploy.nginx

help:
	@echo "\n\033[1;33mAvailable commands:\033[0m" \
	    && awk -F ':.*## ' '/^[a-zA-Z0-9_.-]+:.*## / {printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2}' $(MAKEFILE_LIST) \
	    | sort && echo ""

create.network: ## Create a K3d network
	docker network create k3d-$(CLUSTER_NAME)

start.registry: ## Start Docker registry using Compose
	docker compose up -d

stop.registry: ## Stop Docker registry
	docker compose down

create: ## Create K3d cluster with custom registry
	k3d cluster create --config $(CONFIG_FILE)

delete.network: ## Delete a K3d network
	docker network rm k3d-$(CLUSTER_NAME)

delete: ## Delete the k3d cluster
	k3d cluster delete $(CLUSTER_NAME)
	$(MAKE) stop.registry
	$(MAKE) delete.network

get.nodes: ## List Kubernetes nodes
	kubectl get nodes -o wide

describe.cluster: ## Describe the k3d cluster
	k3d cluster list $(CLUSTER_NAME)

restart: delete create ## Recreate the cluster

kubeconfig: ## Merge kubeconfig and switch context
	k3d kubeconfig merge $(CLUSTER_NAME) --switch-context

build.push: ## Build, push nginx image
	docker pull nginx:latest
	docker tag nginx:latest $(IMAGE_NAME)
	docker push $(IMAGE_NAME)

deploy.nginx: create.namespace ## Deploy NGINX using manifests
	kubectl apply -f ngnix-manifest/deployment.yaml
	kubectl apply -f ngnix-manifest/service.yaml

delete.all: delete.ngnix delete.namespace ## Cleanup everything

ngnix: ## Delete ngnix deployment/service
	kubectl delete -f manifests/service.yaml --ignore-not-found
	kubectl delete -f manifests/deployment.yaml --ignore-not-found

create.namespace: ## Create Kubernetes namespace
	kubectl apply -f ngnix-manifest/namespace.yaml

delete.namespace: ## Delete nginx namespace
	kubectl delete -f manifests/namespace.yaml --ignore-not-found

deploy.view: ## View ngnix pods, svc
	kubectl get pods -n nginx-app
	kubectl get svc -n nginx-app

test.nginx: ## Test nginx availability
	curl --fail http://localhost:8080 || echo "NGINX not reachable"
