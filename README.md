# GitOps Node.js Application on Kubernetes

This repository contains the configuration and source code for a simple Node.js web application deployed to Kubernetes using GitOps principles, with **GitHub Actions** handling the Continuous Integration and Continuous Delivery (CI/CD) pipeline that integrates with **Argo CD**.

## Overview

This project demonstrates an automated workflow for:

* Building and containerizing a Node.js application using Docker via GitHub Actions.
* Pushing the Docker image to a container registry via GitHub Actions.
* Defining Kubernetes deployment and service manifests.
* Automating the deployment and management of the application on a Kubernetes cluster using Argo CD, triggered by changes in this repository.

The Node.js application itself is a simple web server that responds with a greeting.

## Repository Structure
```
.
├── app/                     # Source code for the Node.js application
│   └── index.js
├── manifests/               # Kubernetes deployment and service definitions
│   └── deployment.yaml
│   └── service.yaml
├── .github/workflows/      # GitHub Actions for CI/CD
│   └── ci-cd.yaml
├── .dockerignore            # Specifies intentionally untracked files that Docker should ignore
├── .gitignore               # Specifies intentionally untracked files that Git should ignore
├── argo-app.yaml            # Argo CD Application definition
├── Dockerfile               # Instructions for building the Docker image
├── eslint.config.mjs        # ESLint configuration for code linting
├── package-lock.json        # Records the exact versions of dependencies
├── package.json             # Project metadata and dependencies
└── README.md                # This file

```

## Prerequisites

Before you can leverage this setup, you'll need:

* **A Kubernetes Cluster:** You can use Minikube, kind, a local Kubernetes distribution, or a cloud-based Kubernetes service.
* **kubectl:** The Kubernetes command-line tool configured to connect to your cluster.
* **A Container Registry:** Docker Hub, GitHub Container Registry (GHCR), or another registry to store your Docker images. You'll need credentials configured as secrets in your GitHub repository.
* **Argo CD:** Installed and configured in your Kubernetes cluster. This GitOps tool will watch this repository and apply the configurations in the `manifests/` directory.
* **Git:** Installed on your local machine to interact with this repository.

## Getting Started

The deployment process is largely automated through GitHub Actions and Argo CD. Here's a general overview:

1.  **Clone the Repository:**
    ```bash
    git clone [https://github.com/Naphee25/gitops-nodejs-k8s.git](https://github.com/Naphee25/gitops-nodejs-k8s.git)
    cd gitops-nodejs-k8s
    ```

2.  **Configure Argo CD:**
    You need to create an Argo CD `Application` that points to this repository and the `manifests/` directory. The `argo-app.yaml` file in this repository provides an example. You can apply it using `kubectl` (adjust the namespace if needed):
    ```bash
    kubectl apply -f argo-app.yaml -n argocd
    ```

    **Example `argo-app.yaml` (you might need to adjust `repoURL` and other parameters):**
    ```yaml
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: gitops-nodejs-app
      namespace: argocd
    spec:
      project: default
      source:
        repoURL: [https://github.com/Naphee25/gitops-nodejs-k8s.git](https://github.com/Naphee25/gitops-nodejs-k8s.git)
        targetRevision: HEAD
        path: manifests
      destination:
        server: [https://kubernetes.default.svc](https://kubernetes.default.svc)
        namespace: default
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
        - CreateNamespace=true
    ```

3.  **GitHub Actions Workflow (`ci-cd.yaml`):**
    The `.github/workflows/ci-cd.yaml` file defines your CI/CD pipeline. This workflow is likely configured to:
    * On code changes (e.g., on `push` to the main branch or `pull_request`), it will:
        * Lint the Node.js code.
        * Build the Docker image using the `Dockerfile`.
        * Push the Docker image to your configured container registry (e.g., Docker Hub, GHCR). **You will need to set up secrets in your GitHub repository (e.g., `DOCKER_USERNAME`, `DOCKER_TOKEN/PASSWORD` or similar for your registry).**
        * Potentially update the `deployment.yaml` in the `manifests/` directory with the new image tag (this might involve tools like `sed` or `yq`).
        * Commit and push the updated `deployment.yaml` back to the repository, which will then trigger Argo CD to sync and deploy the changes to your Kubernetes cluster.

    **Note:** The exact steps in your `ci-cd.yaml` might vary. Refer to the file for the specific implementation.

## Kubernetes Manifests

The `manifests/` directory contains the Kubernetes configurations that Argo CD will apply to your cluster:

* **`deployment.yaml`:** Defines the deployment for the Node.js application, specifying the number of replicas and the Docker image to use (likely updated by the CI/CD pipeline).
* **`service.yaml`:** Defines a Kubernetes service to expose the Node.js application within the cluster.

## Continuous Integration/Continuous Delivery (CI/CD) with GitHub Actions

This project leverages GitHub Actions for a fully automated CI/CD pipeline:

1.  **Code Changes:** When you push new code to the repository (or merge a pull request), the GitHub Actions workflow is triggered.
2.  **Build and Test:** The workflow builds the Docker image and may run tests or linters.
3.  **Push Image:** The newly built Docker image is pushed to your container registry.
4.  **Update Manifests (GitOps Step):** The workflow updates the `deployment.yaml` file in the `manifests/` directory to use the latest image tag.
5.  **Commit and Push:** The updated `deployment.yaml` is committed back to the repository.
6.  **Argo CD Sync:** Argo CD, which is continuously monitoring the `manifests/` directory of this repository, detects the change and automatically synchronizes the desired state to your Kubernetes cluster, deploying the new version of your application.

