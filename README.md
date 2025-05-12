# GitOps Node.js App with Kubernetes and Argo CD

This project demonstrates a GitOps workflow to deploy a simple Node.js application on a Kubernetes cluster using Argo CD. It also integrates CI/CD using GitHub Actions, Docker Hub, ESLint, and optional Jest testing.

---

## 🚀 Features

* ⚙️ GitOps deployment with [Argo CD](https://argo-cd.readthedocs.io/en/stable/)
* 🐳 Docker-based containerization
* ✅ Linting with ESLint
* 🔁 Continuous Integration and Delivery using GitHub Actions
* ☸️ Kubernetes manifests for deployment

---

## 📁 Project Structure

```
.
├── .github/workflows       # GitHub Actions workflow files
├── app/                    # Node.js application source
│   ├── index.js
│   ├── package.json
│   └── Dockerfile
├── manifests/              # Kubernetes deployment manifests
│   ├── deployment.yaml
│   └── service.yaml
├── argo-app.yaml           # Argo CD Application manifest
├── README.md
└── .dockerignore
```

---

## 🔧 Prerequisites

* Docker
* kubectl
* A Kubernetes Cluster (local or managed)
* Argo CD installed
* GitHub Account
* Docker Hub Account

---

## 🧱 Setup

### Clone the repository

```bash
git clone https://github.com/Naphee25/gitops-nodejs-k8s.git
cd gitops-nodejs-k8s
```
Update `manifests/deployment.yaml` with your Docker image name.

## 🤖 GitHub Actions CI/CD

The workflow `.github/workflows/ci-cd.yml` performs:

* Install dependencies
* Lint the code
* Build Docker image
* Push to Docker Hub

### 🔐 Secrets Setup

Add the following secrets to your GitHub repository:

* `DOCKER_USERNAME`
* `DOCKER_PASSWORD/TOKEN`

---

## 📦 Deployment with Argo CD

This project uses **Argo CD** for GitOps-based continuous delivery into a local Kubernetes cluster managed by **Minikube**.

### ▶️ Start Minikube

Make sure Docker is installed and running, then start your local Kubernetes cluster:

```bash
minikube start --driver=docker

```

## 🚀 Deploy with Argo CD

Argo CD will automatically sync the state of this Git repository to the cluster.

### Install Argo CD (if not already installed)

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

```
### Apply the Argo CD application definition:

```bash
kubectl apply -f argo-app.yaml
```
### Check Argo CD application status:

```bash
kubectl get applications -n argocd
```

## 🌐 Access the Deployed Node.js App

Once Argo CD deploys the app, access it via Minikube:

```bash
minikube service gitops-nodejs-service --url
```

This will return a URL like:

```bash
http://127.0.0.1:34699

```
Open that URL in your browser to view the app.


🛠 Alternative: Port-Forwarding
If preferred, you can also port-forward the service:

```bash
kubectl port-forward svc/gitops-nodejs-service 8083:80

```
Then open http://localhost:8083 in your browser.


## 🔐 Access the Argo CD Web UI

To access the Argo CD Web UI and manage your applications, follow these steps:

### 1. Get the Argo CD Server URL

First, you need to get the **Argo CD Server** service URL.

Run:

```bash
kubectl get svc -n argocd

```
Look for the argocd-server service. It should be exposed as ClusterIP by default.

### 2. Port-forward the Argo CD server

You can port-forward the Argo CD server to access it locally:

Run:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443

```
This forwards the Argo CD UI to http://localhost:8080.

### 3. Login to the Argo CD Web UI

Open your browser and go to http://localhost:8080.

The default username is admin.

To retrieve the default password, run:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```
### 4. Manage Applications
Once logged in, you can manage your GitOps applications, monitor sync statuses, and configure Argo CD settings.


## 📚 Resources

* [Argo CD Docs](https://argo-cd.readthedocs.io/)
* [Docker Hub](https://hub.docker.com/)
* [GitHub Actions](https://docs.github.com/en/actions)
* [Kubernetes](https://kubernetes.io/)

---

## 👩🏽‍💻 Author

Nafisah — [Medium](https://medium.com/@nafisahabidemiabdulkadir)
