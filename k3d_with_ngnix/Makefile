CLUSTER_NAME=k3d-ngnix-cluster
CONFIG_FILE=k3d-config.yaml
.DEFAULT_GOAL := help

.PHONY: create delete get-nodes describe-cluster restart kubeconfig deploy.nginx delete.nginx test.nginx


help:  ## Help message
	@echo ""
	@echo "\033[1;33mAvailable commands:\033[0m" && \
	awk -F ':.*?## ' '/^[a-zA-Z0-9_.-]+:.*## / { \
		cmds[$$1] = $$2; \
		if (length($$1) > max_len) max_len = length($$1); \
	} \
	END { \
		for (cmd in cmds) { \
			printf "  \033[36m%-" max_len "s\033[0m %s\n", cmd, cmds[cmd]; \
		} \
	}' $(MAKEFILE_LIST) | sort
	@echo ""

.PHONY: create
create: ## Create K3d cluster from config file
	k3d cluster create --config $(CONFIG_FILE)

.PHONY: delete
delete: ## Delete the K3d cluster
	k3d cluster delete $(CLUSTER_NAME)

.PHONY: get.nodes
get.nodes: ## Get Kubernetes nodes
	kubectl get nodes -o wide

.PHONY: describe.cluster
describe.cluster: ## Describe the K3d cluster
	k3d cluster list $(CLUSTER_NAME)

.PHONY: restart
restart: delete create ## Restart cluster (delete + create)

.PHONY: kubeconfig
kubeconfig: ## Merge K3d kubeconfig into local ~/.kube/config
	k3d kubeconfig merge $(CLUSTER_NAME) --switch-context

create.namespace: ## Create Kubernetes namespace
	kubectl apply -f ngnix-manifest/namespace.yaml

deploy.view: ## View ngnix pods, svc
	kubectl get pods -n nginx-app
	kubectl get svc -n nginx-app

delete.namespace: ## Delete Kubernetes namespace
	kubectl delete -f ngnix-manifest/namespace.yaml

deploy.nginx: create.namespace ## Deploy NGINX using manifests
	kubectl apply -f ngnix-manifest/deployment.yaml
	kubectl apply -f ngnix-manifest/service.yaml

delete.nginx: ## Delete NGINX resources
	kubectl delete -f ngnix-manifest/service.yaml --ignore-not-found
	kubectl delete -f ngnix-manifest/deployment.yaml --ignore-not-found
	$(MAKE) delete.namespace

test.nginx: ## Test NGINX via curl
	curl --fail http://localhost:8080 || echo "NGINX not reachable"