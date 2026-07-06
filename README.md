# Kubernetes Minikube Tutorials

Hands-on Kubernetes learning labs using Minikube, `kubectl`, and simple NGINX workloads. This repository is intended for beginners who want to practice core Kubernetes objects locally before working with larger clusters.

The examples cover:

- Creating and inspecting Pods
- Applying YAML manifests with `kubectl`
- Using labels and selectors
- Managing ReplicaSets and Deployments
- Scaling workloads
- Rolling out and rolling back application changes
- Exposing workloads with ClusterIP and NodePort Services
- Creating a basic Ingress definition

## Prerequisites

Install the following tools before running the labs:

- Docker or another Minikube-supported container runtime
- Minikube
- kubectl
- Git

Verify the installation:

```bash
docker --version
minikube version
kubectl version --client
```

Detailed Linux setup notes are available in [ClusterSetup/Installing_MiniKube_Kubectl.txt](ClusterSetup/Installing_MiniKube_Kubectl.txt).

## Repository Contents

| Path | Purpose |
| --- | --- |
| `ClusterSetup/Installing_MiniKube_Kubectl.txt` | Installation and cluster setup notes for Docker, Minikube, and kubectl |
| `YAML Files/Pods_Demo.txt` | Pod lab command walkthrough |
| `YAML Files/RS_Deployment_Demo.txt` | ReplicaSet and Deployment lab command walkthrough |
| `Services_Demo.txt` | Service lab command walkthrough |
| `nginx-pod.yaml` | Standalone NGINX Pod manifest |
| `nginx-replicaset.yaml` | NGINX ReplicaSet manifest with 3 replicas |
| `nginx-deployment.yaml` | NGINX Deployment manifest |
| `nginx-deploymentUpdated.yaml` | Updated Deployment example using the `nginx` namespace and change-cause annotation |
| `nginx-service.yaml` | ClusterIP-style Service for the NGINX Deployment |
| `nginx-service_ClusterIP.yaml` | Explicit ClusterIP Service example |
| `nginx-service_NodePort.yaml` | NodePort Service example |
| `nginx-ingress.yaml` | Ingress example for `nginx-demo.com` |
| `nginx-namesapce.yaml` | Namespace manifest for the `nginx` namespace |

> Note: `nginx-namesapce.yaml` keeps the existing repository filename, including the spelling.

## Quick Start

Clone the repository:

```bash
git clone https://github.com/UiPath-Coders/Shared-Kubernetes_MiniKube_Tutorials.git
cd Shared-Kubernetes_MiniKube_Tutorials
```

Start a local Minikube cluster:

```bash
minikube start --driver=docker
kubectl cluster-info
```

For a named local cluster with multiple nodes:

```bash
minikube start -p local-cluster --driver=docker
minikube node add --worker -p local-cluster
```

## Lab 1: Pods

Create a standalone NGINX Pod:

```bash
kubectl apply -f nginx-pod.yaml
kubectl get pods
kubectl describe pod nginx-pod1
```

Filter Pods by labels:

```bash
kubectl get pods -l team=integrations
kubectl get pods -l team=integrations,app=todo
```

Open a shell inside the container:

```bash
kubectl exec -it nginx-pod1 -c nginx-container -- bash
```

Forward local traffic to the Pod:

```bash
kubectl port-forward nginx-pod1 8083:80
curl http://localhost:8083
```

View logs and clean up:

```bash
kubectl logs nginx-pod1
kubectl delete -f nginx-pod.yaml
```

## Lab 2: ReplicaSets

Apply the ReplicaSet manifest:

```bash
kubectl apply -f nginx-replicaset.yaml
kubectl get rs
kubectl get pods -o wide
```

Delete one Pod and observe Kubernetes recreate it:

```bash
kubectl delete pod <pod-name>
kubectl get pods -o wide
```

Clean up before moving to Deployments:

```bash
kubectl delete -f nginx-replicaset.yaml
```

## Lab 3: Deployments

Apply the Deployment:

```bash
kubectl apply -f nginx-deployment.yaml
kubectl get all
kubectl get pods --show-labels
```

Scale the Deployment:

```bash
kubectl scale --replicas=5 deployment/nginx-deployment
kubectl get pods
```

Update the container image:

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.21
kubectl rollout status deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment
```

Roll back to a previous revision:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
kubectl rollout status deployment/nginx-deployment
```

## Lab 4: Namespace, Services, and NodePort

Create the namespace used by the namespace-scoped manifests:

```bash
kubectl apply -f nginx-namesapce.yaml
```

Apply the updated Deployment and ClusterIP Service:

```bash
kubectl apply -f nginx-deploymentUpdated.yaml
kubectl apply -f nginx-service_ClusterIP.yaml
kubectl get all -n nginx
kubectl get endpoints -n nginx
```

Test the Service from inside the cluster:

```bash
kubectl exec -it -n nginx <pod-name> -- sh
curl nginx-service:8082
```

Expose the Service through NodePort:

```bash
kubectl apply -f nginx-service_NodePort.yaml
kubectl get svc -n nginx
minikube service nginx-service -n nginx
```

## Lab 5: Ingress

Enable the Minikube Ingress addon:

```bash
minikube addons enable ingress
```

Apply the Ingress manifest:

```bash
kubectl apply -n nginx -f nginx-ingress.yaml
kubectl get ingress -n nginx
```

The sample Ingress is configured for `nginx-demo.com`. For local testing, map the Minikube IP to that host in your machine's hosts file:

```bash
minikube ip
```

Example hosts entry:

```text
<minikube-ip> nginx-demo.com
```

Then test:

```bash
curl http://nginx-demo.com
```

## Cleanup

Remove the lab resources:

```bash
kubectl delete -n nginx -f nginx-ingress.yaml --ignore-not-found
kubectl delete -f nginx-service_NodePort.yaml --ignore-not-found
kubectl delete -f nginx-service_ClusterIP.yaml --ignore-not-found
kubectl delete -f nginx-deploymentUpdated.yaml --ignore-not-found
kubectl delete -f nginx-deployment.yaml --ignore-not-found
kubectl delete -f nginx-replicaset.yaml --ignore-not-found
kubectl delete -f nginx-pod.yaml --ignore-not-found
kubectl delete -f nginx-namesapce.yaml --ignore-not-found
```

Stop or delete the Minikube cluster:

```bash
minikube stop
# or
minikube delete
```

## Troubleshooting

- If `kubectl` cannot connect to the cluster, run `minikube status` and `kubectl cluster-info`.
- If Docker permissions fail on Linux, make sure your user is in the `docker` group and start a new shell session.
- If a Service has no endpoints, confirm that the Service selector matches the Pod labels. The Service examples select `app: nginx`, so they are intended to work with the ReplicaSet or Deployment examples, not the standalone Pod manifest.
- If NodePort access fails, use `minikube service nginx-service -n nginx` to open or print the correct local URL.
- If Ingress does not respond, confirm that the Ingress addon is enabled and that `nginx-demo.com` points to the current `minikube ip`.

## Additional References

- [Minikube documentation](https://minikube.sigs.k8s.io/docs/)
- [Kubernetes kubectl documentation](https://kubernetes.io/docs/reference/kubectl/)
- [Kubernetes Services, Load Balancing, and Networking](https://kubernetes.io/docs/concepts/services-networking/)
- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for branching, commit message, and pull request guidance.
