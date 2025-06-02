# ArgoCD GitOps POC (Local)

This repository contains a minimal GitOps proof-of-concept using ArgoCD, KinD (Kubernetes in Docker), and a basic NGINX deployment. It’s designed to run entirely on your local machine for learning or experimentation.

---

## Prerequisites

Make sure you have the following installed:

- [Docker](https://docs.docker.com/get-docker/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [kind](https://kind.sigs.k8s.io/)
- [argocd CLI](https://argo-cd.readthedocs.io/en/stable/cli_install/)
- [git](https://git-scm.com/)

---

## Setup Instructions

### 1. Create a Local Kubernetes Cluster with KinD

```bash
kind create cluster --name argocd-poc
```

Check if the cluster is running:

```bash
kubectl cluster-info --context kind-argocd-poc
```

### 2. Install ArgoCD

Create the `argocd` namespace:

```bash
kubectl create namespace argocd
```

Apply the ArgoCD installation manifest:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait for all pods to come up:

```bash
kubectl get pods -n argocd
```

### 3. Access the ArgoCD UI

Port-forward the ArgoCD server:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Then open your browser to: [https://localhost:8080](https://localhost:8080).

Login credentials:

- **Username:** `admin`
- **Password:** 

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

---

### 4. Create the NGINX App Manifest

Below is an example NGINX deployment manifest:

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
    name: nginx
    labels:
        app: nginx
spec:
    replicas: 1
    selector:
        matchLabels:
            app: nginx
    template:
        metadata:
            labels:
                app: nginx
        spec:
            containers:
            - name: nginx
                image: nginx:1.25
                ports:
                - containerPort: 80
```

Commit and push this file to your GitHub repository.

---

### 5. Create Namespace for App

Create a namespace for the application:

```bash
kubectl create namespace apps
```

---

### 6. Deploy App via ArgoCD CLI

Log in to ArgoCD:

```bash
argocd login localhost:8080 --insecure
```

Create the application:

```bash
argocd app create nginx-app \
    --repo https://github.com/jsh-architect/argocd-poc.git \
    --path . \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace apps \
    --sync-policy automated
```

---

### 7. Verify the Deployment

Check the pods in the `apps` namespace:

```bash
kubectl get pods -n apps
```

You should see the NGINX pod running.

---

### Cleanup

Delete the KinD cluster:

```bash
kind delete cluster --name argocd-poc
```

---

## Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [KinD Documentation](https://kind.sigs.k8s.io/)
- [NGINX Docker Image](https://hub.docker.com/_/nginx)
