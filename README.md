# Kubernetes
This repository aims at learning Kubernetes working on a very simple tutorial. It explains pods, services, deployments, persistent volumes + claims, helm charts, operators and finally managing everything through Rancher, a management plane.

The tutorial would initially start working through different components on an adhoc basis, try it out setting it up using `kubectl`. At the end, when it would look like that the components have been well understood, everything would be simply thrown away and deployed using helm charts.

## Prerequisites

For our setup we need to install Docker desktop, `kubectl`, `kind` and `helm`.

- **kubectl:** Kubectl is the command-line tool used to communicate with and manage Kubernetes clusters. It allows users to create, inspect, update, and delete Kubernetes resources through the Kubernetes API.

- **kind:** kind (Kubernetes in Docker) is a tool for running local Kubernetes clusters using Docker containers as the cluster's nodes. It is mainly used for development, testing and CI pipelines.

- **helm:** Helm is the package manager for Kubernetes. It bundles Kubernetes configuration files into reusable packages called charts, making applications easier to install, configure, upgrade, and roll back.

To install the above three components run:

```bash
brew install kubectl kind helm
```

Docker desktop can be downloaded from Docker's website. With this we are set to start working on our `web` application.


## How to navigate this project?

Inside the `configurations` directory you will find different directories each containing its own configuration. The directory name itself contains numbers and the configuration files should be run in the same sequence.

Each configuration directory has its own `README.md` file which explains important terms and concepts from the configuration file.

To begin with we would first need to setup a local Kubernetes cluster, which we can start with `kind`.

```bash
kind create cluster --name learning
```

When `kind` creates a cluster locally, it creates Docker containers that act as Kubernetes nodes. A node is a machine (physical or virtual) that runs Kubernetes workloads. In a local `kind` cluster, each node is typically a Docker container.

A node usually runs pods and a Kubernetes cluster contains one or more nodes.

Quickly verify that the cluster works:

```bash
kubectl get nodes
```

You should see something similar as follows:

```
NAME                     STATUS   ROLES           AGE    VERSION
learning-control-plane   Ready    control-plane   3d9h   v1.33.12
```

The name of our node is `learning-control-plane` and is in ready status.

> NOTE: Please note that all the commands would be executed from the root directory of the project.

As all the development is carried out in the namespace `development`, you will always have to include the namespace in the commands `-n development`, else you might get not found error.

To avoid this set `development` as the default namespace for our current `kubectl context`.

```bash
kubectl config set-context --current --namespace=development
```