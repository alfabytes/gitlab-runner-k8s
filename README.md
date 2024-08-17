# GitLab Runners on Kubernetes using Helm

This repository provides a guide and configuration files to deploy GitLab Runners on a Kubernetes cluster using Helm.

## Prerequisites

Before you begin, ensure you have the following:

1. A Kubernetes cluster up and running.
2. Helm installed on your local machine.
3. A GitLab account with the necessary permissions to configure Runners.
4. `kubectl` configured to interact with your Kubernetes cluster.

## Setup Guide

### Step 1: Add the GitLab Helm Repository

Add the official GitLab Helm repository to your Helm configuration:

```bash
helm repo add gitlab https://charts.gitlab.io/
helm repo update
```

### Step 2: Create a Namespace for GitLab Runners

Create a namespace called gitlab-runners:

```bash
kubectl create namespace gitlab-runners
```

### Step 3: Install GitLab Runner Using Helm

Install the GitLab Runner Helm chart. Replace `YOUR_REGISTRATION_TOKEN` with your GitLab Runner registration token:

```bash
helm install gitlab-runner gitlab/gitlab-runner --namespace gitlab-runners --set gitlabUrl=https://gitlab.com/,runnerToken=YOUR_REGISTRATION_TOKEN
```

### Step 4: Verify the Installation

Verify that the GitLab Runner pods are running:

```bash
kubectl get pods -n gitlab-runners
```

### Step 5: Configure GitLab Runner

Create a `values.yaml` file with the following content:

```yaml
gitlabUrl: https://gitlab.com/
runnerToken: YOUR_REGISTRATION_TOKEN
runners:
  config: |
    [[runners]]
      [runners.kubernetes]
        image = "ubuntu:20.04"
        privileged = true
      [[runners.kubernetes.volumes.empty_dir]]
        name = "docker-certs"
        mount_path = "/certs/client"
        medium = "Memory"
```

Apply the changes:

```bash
helm upgrade gitlab-runner gitlab/gitlab-runner --namespace gitlab-runners -f values.yaml
```

### Optional Step: Configure Access to a Private Docker Repository

If your Docker image is hosted in a private repository, create a Kubernetes secret:

```bash
kubectl create secret docker-registry regcred \
  --docker-username=YOUR_DOCKER_USERNAME \
  --docker-password=YOUR_DOCKER_PASSWORD \
  --docker-email=YOUR_DOCKER_EMAIL \
  --docker-server=YOUR_DOCKER_REGISTRY \
  --namespace=gitlab-runners
```

or

```bash
kubectl create secret docker-registry regcred \
  --docker-server=<aws_account_id>.dkr.ecr.<region>.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password) \
  --namespace=gitlab-runners
```

Update the `values.yaml` file to reference this secret:

```yaml
gitlabUrl: https://gitlab.com/
runnerToken: YOUR_REGISTRATION_TOKEN
runners:
  config: |
    [[runners]]
      [runners.kubernetes]
        image = "<aws_account_id>.dkr.ecr.<region>.amazonaws.com/private/image:latest"
        privileged = true
        imagePullSecrets = ["regcred"]
      [[runners.kubernetes.volumes.empty_dir]]
        name = "docker-certs"
        mount_path = "/certs/client"
        medium = "Memory"
```

Apply the updated configuration:

```bash
helm upgrade gitlab-runner gitlab/gitlab-runner --namespace gitlab-runners -f values.yaml
```

### Step 6: Monitor and Manage GitLab Runners

Monitor the GitLab Runner logs:

```bash
kubectl logs -f <pod-name> -n gitlab-runners
```

Replace <pod-name> with the name of your GitLab Runner pod.

### Conclusion

Deploying GitLab Runners on Kubernetes using Helm is a straightforward process that leverages the power and flexibility of both Kubernetes and Helm. Configuring access to private Docker repositories ensures that your runners can use all necessary images securely. This setup allows for scalable and resilient CI/CD pipelines, making it an excellent choice for modern DevOps practices.

For more details and example configurations, refer to the [Medium article](https://blog.alfabyte.xyz/gitlab-runners-on-kubernetes-a927161f6d7c).

## Repository Content
- values.yaml: Example configuration file for GitLab Runner.
- README.md: This file, providing setup and configuration instructions.


