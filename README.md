# my-helm-app

This repository is a simple Helm chart used to learn how to containerize an application and deploy it on Kubernetes using Helm. It demonstrates the basic structure of a Helm chart, how values are parameterized, and how Kubernetes resources such as a Deployment and a Service are created from templates.

## What this project does

The chart deploys a basic Nginx web application on a Kubernetes cluster. It creates:

- a Deployment with 2 replicas by default
- a Service exposing the app on a NodePort
- a configurable image and port setup via `values.yaml`

This repo is intentionally minimal and is designed for learning and practice rather than production deployment.

## Project structure

```text
my-helm-app/
├── Chart.yaml
├── values.yaml
├── .helmignore
├── templates/
│   ├── deployment.yaml
│   └── service.yaml
└── README.md
```

### File explanations

#### `Chart.yaml`
This file defines the Helm chart metadata.

```yaml
apiVersion: v2
name: my-helm-app
description: A Helm chart for practicing deployments
type: application
version: 0.1.0
appVersion: "1.0.0"
```

It tells Helm:

- the chart name is `my-helm-app`
- the chart version is `0.1.0`
- the application version is `1.0.0`

#### `values.yaml`
This stores default configuration values used by the templates.

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 80
```

These values control:

- how many pods should run (`replicaCount`)
- which container image to use (`nginx:latest`)
- the type of Kubernetes Service (`NodePort`)
- the exposed application port (`80`)

#### `templates/deployment.yaml`
This template creates a Kubernetes Deployment.

It uses:

- `{{ .Release.Name }}` to name resources based on the Helm release
- `{{ .Values.replicaCount }}` to define pod replicas
- `{{ .Values.image.repository }}:{{ .Values.image.tag }}` to build the container image

The deployment runs the Nginx container and exposes port `80`.

#### `templates/service.yaml`
This template creates a Kubernetes Service for the app.

It selects the pods using the label `app: {{ .Release.Name }}` and exposes the Service on port `80`, targeting the container port `80`.

## Prerequisites

Before using this chart, make sure you have:

- a Kubernetes cluster running
- `kubectl` installed and configured to connect to the cluster
- Helm 3 installed

To check installation:

```bash
kubectl version --client
helm version
```

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

This checks for syntax issues and common chart problems.

### 3. Render the Kubernetes manifests

```bash
helm template my-helm-app .
```

This prints the YAML that Helm would apply without actually deploying anything.

### 4. Install the Helm chart

```bash
helm install my-helm-app .
```

If you want to install into a specific namespace:

```bash
helm install my-helm-app . --namespace default
```

### 5. Verify the deployment

```bash
kubectl get deployment
kubectl get pods
kubectl get service
```

You should see:

- a Deployment named like `my-helm-app-deployment`
- pods created from the Nginx image
- a Service named like `my-helm-app-service`

## Accessing the application

Since the Service type is set to `NodePort`, you can access it using the cluster node IP and the assigned NodePort.

Check the service port:

```bash
kubectl get svc
```

Example:

```bash
kubectl get svc my-helm-app-service
```

Then open the app in a browser using:

```text
http://<NodeIP>:<NodePort>
```

If you are running locally with a cluster that supports port forwarding, you can also do:

```bash
kubectl port-forward svc/my-helm-app-service 8080:80
```

Then visit:

```text
http://localhost:8080
```

## Updating values

If you want to change the deployment behavior, edit `values.yaml`.

For example, to increase replicas:

```yaml
replicaCount: 3
```

To change the image:

```yaml
image:
  repository: nginx
  tag: "1.25.4"
```

Then upgrade the release:

```bash
helm upgrade my-helm-app .
```

## Uninstall

To remove the release from the cluster:

```bash
helm uninstall my-helm-app
```

## Summary

This project is a basic Helm learning example that shows:

- how to package an application as a Helm chart
- how to use `values.yaml` for configuration
- how Kubernetes Deployment and Service resources are created from templates
- how to install, upgrade, and uninstall an app on Kubernetes using Helm

This repo is a good starting point for anyone learning Helm basics and Kubernetes deployment workflows.
