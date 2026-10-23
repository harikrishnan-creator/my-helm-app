# my-helm-app

This repository contains a simple Helm chart for deploying a containerized application on Kubernetes. The project was created to learn Helm basics, Kubernetes resource templating, and how to expose applications through Ingress.

It also includes a GitOps manifest, `hem-ops.yaml`, which defines an ArgoCD Application for deploying this chart directly from the Git repository.

Latest commit: `91285e485376f311b0fad6a423e5f4369907eb3a` — `Update replicaCount from 2 to 5`

The current chart reflects the latest project changes, including:

- a Kubernetes deployment scaled to 5 replicas
- a base NGINX image (`nginx:latest`)
- a Kubernetes `Service` using `ClusterIP`
- an enabled `Ingress` resource using the NGINX Ingress class
- optional autoscaling configuration in `values.yaml`

## What this project does

The chart deploys a basic application on a Kubernetes cluster and creates:

- a Deployment with a configurable replica count
- a Service that routes traffic to the app pods
- an Ingress rule for external access through NGINX
- optional autoscaling values for future scaling scenarios

This repo is intended for learning and practice rather than production-ready deployment.

## Project structure

```text
my-helm-app/
├── Chart.yaml
├── values.yaml
├── hem-ops.yaml
├── .helmignore
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
├── README.md
└── .values.yaml.swp   # temporary file created by editor, should not be committed
```

## Chart files

### `Chart.yaml`

This file defines the Helm chart metadata.

```yaml
apiVersion: v2
name: my-helm-app
description: A Helm chart for practicing deployments
type: application
version: 0.1.0
appVersion: "1.0.0"
```

### `values.yaml`

This file stores the default chart configuration.

```yaml
replicaCount: 5

image:
  repository: nginx
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP   # use ClusterIP when exposing via Ingress
  port: 80

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 5
  targetCPUUtilizationPercentage: 80

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: myapp.local
      paths:
        - path: /
          pathType: Prefix
```

These values control:

- the number of pods (`replicaCount`)
- the app image and tag (`image.repository`, `image.tag`)
- the type of Service used (`ClusterIP` for Ingress-based routing)
- whether Ingress is enabled (`ingress.enabled`)
- the host and path used by the Ingress rule

### `templates/deployment.yaml`

This template creates the Kubernetes Deployment.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: {{ .Release.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

### `templates/service.yaml`

This template creates the Kubernetes Service.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-service
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Release.Name }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: 80
```

### `templates/ingress.yaml`

This template creates the Ingress resource used to expose the app externally.

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: {{ .Values.ingress.host }}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: {{ .Release.Name }}-service
                port:
                  number: 80
{{- end }}
```

## Prerequisites

Before using this chart, ensure you have:

- a Kubernetes cluster running
- `kubectl` installed and configured
- Helm 3 installed
- an Ingress controller such as NGINX Ingress installed in the cluster
- ArgoCD installed if you want to use the GitOps deployment flow (`hem-ops.yaml`)

To confirm your tools are available:

```bash
kubectl version --client
helm version
```

## GitOps with ArgoCD

This repository includes an ArgoCD Application manifest in `hem-ops.yaml` to deploy the Helm chart using GitOps. The manifest tells ArgoCD to monitor the Git repository, sync the chart into the cluster, and automatically reconcile configuration changes.

The ArgoCD application definition includes:

- `apiVersion: argoproj.io/v1alpha1`
- `kind: Application`
- `metadata.name: my-helm-app`
- `metadata.namespace: argocd`
- `source.repoURL: https://github.com/harikrishnan-creator/my-helm-app.git`
- `targetRevision: main`
- `path: .`
- `destination.server: https://kubernetes.default.svc`
- `destination.namespace: default`
- `syncPolicy.automated.prune: true`
- `syncPolicy.automated.selfHeal: true`

### Steps to deploy through ArgoCD

1. Install ArgoCD in your cluster if it is not already installed.

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

2. Access the ArgoCD UI or use `argocd` CLI to login.

3. Apply the GitOps manifest from this repository:

```bash
kubectl apply -f hem-ops.yaml -n argocd
```

4. In the ArgoCD UI, the `my-helm-app` application should appear automatically.

5. Click Sync to initiate the deployment, or let the automated policy handle it.

6. ArgoCD will continuously track the Git repository and apply updates to the cluster.

### Verify the ArgoCD deployment

```bash
kubectl get application -n argocd
kubectl get pods -n default
kubectl get svc -n default
```

You can also inspect the application from the ArgoCD UI or with the CLI:

```bash
argocd app get my-helm-app -n argocd
```

This GitOps flow is useful when you want the cluster state to match the Git repository automatically.

## Setup and deployment

### 1. Clone the repository

```bash
git clone https://github.com/harikrishnan-creator/my-helm-app.git
cd my-helm-app
```

### 2. Validate the chart

```bash
helm lint .
```

### 3. Render the manifests

```bash
helm template my-helm-app .
```

### 4. Install the chart

```bash
helm install my-helm-app .
```

If you want to install in a specific namespace:

```bash
helm install my-helm-app . --namespace default
```

### 5. Verify the deployment

```bash
kubectl get deployment
kubectl get pods
kubectl get service
kubectl get ingress
```

## Accessing the application

Because the chart enables Ingress, the app can be accessed through the configured host.

For example, if `ingress.host` is set to `myapp.local`, add it to your `/etc/hosts` file or use a local DNS setup:

```text
127.0.0.1 myapp.local
```

Then open:

```text
http://myapp.local
```

If you prefer port forwarding instead:

```bash
kubectl port-forward svc/my-helm-app-service 8080:80
```

Then visit:

```text
http://localhost:8080
```

## Updating values

Edit `values.yaml` to adjust the characteristics of the deployment.

Example: increase replicas:

```yaml
replicaCount: 5
```

Example: change the image:

```yaml
image:
  repository: nginx
  tag: "latest"
```

Then upgrade the release:

```bash
helm upgrade my-helm-app .
```

When using ArgoCD, changes pushed to Git are automatically reconciled by ArgoCD, so you don't need to manually run `helm upgrade` unless you are performing a direct cluster-side Helm deployment.

## Autoscaling setup

The chart includes autoscaling values for future use:

```yaml
autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 5
  targetCPUUtilizationPercentage: 80
```

These values are ready to be used if you add a HorizontalPodAutoscaler template later.

## Uninstall

To remove the release from the cluster:

```bash
helm uninstall my-helm-app
```

If using ArgoCD, you can also remove the application from ArgoCD:

```bash
kubectl delete -f hem-ops.yaml -n argocd
```

## Summary

This project demonstrates:

- how Helm charts package application resources
- how to define deployment, service, and ingress settings in values
- how to expose application traffic using Kubernetes Ingress
- how to manage a basic Helm deployment workflow for learning and practice
- how to use ArgoCD GitOps to sync the repository to Kubernetes automatically

This repository is a good starting point for learning Helm, Kubernetes manifests, Ingress routing, and GitOps automation with ArgoCD.
