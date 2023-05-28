# My Flask App

This repository contains a simple Flask application deployed to a Kubernetes cluster using Docker, GitHub Actions for CI/CD, and FluxCD for GitOps-based deployment.

## Prerequisites

- Docker
- Kubernetes (Minikube)
- Flux
- Python 3.8
- Git

## Setup

Follow these steps to get your environment ready:

1. Install [Docker](https://docs.docker.com/get-docker/)
2. Install [Minikube](https://minikube.sigs.k8s.io/docs/start/) 
3. Clone this repository: 
```
git clone https://github.com/donsolly/my_flask_app.git
```
4. Install flux [Flux] (https://fluxcd.io/flux/installation/)

## Setup flux

flux bootstrap github \
  --owner=donsolly \
  --repository=my_flask_app \
  --path=clusters/my-cluster \
  --personal \
  --components-extra=image-reflector-controller,image-automation-controller
```

### Update key to write to repo to allow image update

```bash
kubectl -n flux get secret flux-system -o json | jq '.data."identity.pub"' -r | base64 -d
```

## Deployment

Before deploying, ensure to setup the necessary secrets and references:

1. In your GitHub repository, create secrets for `DOCKER_PASSWORD` & `DOCKER_USERNAME` & `GH_PAT` (Github Token).
2. Update the following credentials:
    - `.github/workflows/main.yml` (Line 31, 47): DockerHub credentials
    - `app/deployment.yaml`: DockerHub image reference
    - `clusters/my-cluster/flux-system/GitRepository.yaml`: Git repository URL
    - `clusters/my-cluster/flux-system/gotk-sync.yaml`: Git repository URL
    - `clusters/my-cluster/infra/image-update.yaml`: DockerHub image reference

## Usage

Follow these steps to start the application:

1. Start the Docker service. `sudo systemctl start docker`
2. Start Minikube: `minikube start`.
3. Set up a tunnel to service the cluster: `minikube tunnel`.
4. Apply the deployment configuration: `kubectl apply -f deployment.yaml`.
5. Retrieve the URL to access the Flask app: `minikube service my-flask-app --url`.
6. Monitor your running pods: `kubectl get pods --watch`.


## Continuous Delivery Plan

### Tools

For continuous delivery, this project leverages GitHub Actions and FluxCD:

- **GitHub Actions**: Integrated with GitHub, this CI/CD solution facilitates automatic building and pushing of Docker images when code changes are pushed to the repository.
- **FluxCD**: A GitOps tool, FluxCD uses Git as the single source of truth, automatically applying changes to Docker images or Kubernetes manifests in the cluster.

### Pipeline

Here's how automatic deployments to production work:

1. A change is pushed to the `main` branch on GitHub.
2. This triggers a GitHub Actions workflow which builds a Docker image from the updated code, and pushes it to DockerHub.
3. FluxCD, monitoring the DockerHub repository and the GitHub repo, pulls the new image and updates the Kubernetes manifests in the cluster.

### Rollbacks

FluxCD facilitates easy rollbacks in case of deployment issues. As it uses Git as the source of truth for deployments, rolling back to a previous version of the application is as simple as reverting the commit in Git.

### Monitoring and Troubleshooting

Monitor the application using built-in Kubernetes tools, such as `kubectl logs` and `kubectl describe`. Future enhancements may include more advanced monitoring solutions like Prometheus or Grafana.

## Troubleshooting

- **Old build still running**:
    1. Reapply the deployment configuration: `kubectl apply -f deployment.yaml ` 
- **General commands**:
    - View running pods: `kubectl get pods`
    - View pod details: `kubectl describe pod <POD_NAME>`

**Note**: When code is pushed to the repository, the GitHub Actions workflow automatically creates a new Docker build, uploads it to DockerHub, and FluxCD handles the continuous deployment.