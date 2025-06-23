# eem-iks

This repo contains Kubernetes manifests and ArgoCD config to install Event Endpoint Management on a Kubernetes cluster. Use this to either setup automated install of EEM with ArgoCD or use the manifests and apply them manually.

The full EEM Documentation with detailed instructions is available here: https://ibm.github.io/event-automation/eem/

An overview of the installation steps is available here: https://ibm.github.io/event-automation/eem/installing/overview/

EEM CR examples: https://github.com/IBM/ibm-event-automation/tree/main/event-endpoint-management

# Install ArgoCD

Install ArgoCD

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

# Install Cert-Manager if not available

A certificate manager is required to automatically manage the process of creating, renewing and using Event Endpoint Management internal and system-to-system certificates. It is also used by default for managing Event Endpoint Management certificates that are visible to clients and users, simplifying their configuration.

`kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.18.0/cert-manager.yaml`


# Fork this repo and adapt to your environment

Fork this repo and update the repoURL in [argocd/bootstrap.yaml](./argocd/bootstrap.yaml) and in [argocd/kustomization.yaml](./argocd/kustomization.yaml)

# EEM Install

See https://ibm.github.io/event-automation/eem/installing/installing-on-kubernetes/ for details.

Prereq is a Kubernetes platform that support the Red Hat Universal Base Images (UBI) containers, versions 1.25 to 1.32

## 1. Install EEM Operator

The Event Endpoint Management operator requires the following cluster-scoped permissions, even if the operator is set to manage instances in a single namespace:

- **Permission to retrieve storage classes**: The Event Endpoint Management operator uses admission webhooks to provide immediate validation and feedback about the creation and modification of the Event Manager and operator-managed Event Gateway instances. The permission to retrieve storage classes is used by the webhooks to find a default storage class.
- **Permission to list specific CustomResourceDefinitions**: This allows Event Endpoint Management to identify whether other optional dependencies have been installed into the cluster.

There are two ArgoCD applications configured, one for the EEM CRDs (./argocd/eem-crds.yaml) and one for the operator (./argocd/eem-operator.yaml). 

```helm repo add ibm-helm https://raw.githubusercontent.com/IBM/charts/master/repo/ibm-helm
kubectl create namespace ibm-event-automation
helm install eem-crds ibm-helm/ibm-eem-operator-crd -n ibm-event-automation
helm install eventendpointmanagement ibm-helm/ibm-eem-operator -n "ibm-event-automation
```

## 2. Create Event Manager instance

### Create image pull secret

Create an image pull secret to access the IBM container registry

`kubectl create secret docker-registry ibm-entitlement-key --docker-username=cp --docker-password="<your-entitlement-key>" --docker-server="cp.icr.io" -n "ibm-event-automation"`

### Update event manager CR sample with values for your cluster

Update the event manager cr sample with correct ingress subdomain, ingress class and storage class names here: [eventendpointmanagement.yaml](./components/eventmanager/eventendpointmanagement.yaml)

Sample secrets are created with admin, author and viewer users. Remove these and create them manually if you don't want these cleartext users and passwords in your repo. You can change this here: [./components/eventmanager/kustomization.yaml](./components/eventmanager/kustomization.yaml)

Add local users to be able to login to the UI. See https://ibm.github.io/event-automation/eem/security/managing-access/ and https://ibm.github.io/event-automation/eem/security/user-roles/

Login to the UI with one of the sample users.

## 3. Create Event Gateway instance

Login to the Event manager UI and 

