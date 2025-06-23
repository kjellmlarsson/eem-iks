# eem-iks

This repo contains Kubernetes manifests and ArgoCD config to install Event Endpoint Management on a Kubernetes cluster. Use this to either setup automated install of EEM with ArgoCD or use the manifests and manually apply them.

The full EEM Documentation with detailed instructions is available here: https://ibm.github.io/event-automation/eem/

An overview of the installation steps is available here: https://ibm.github.io/event-automation/eem/installing/overview/

EEM CR examples: https://github.com/IBM/ibm-event-automation/tree/main/event-endpoint-management

# Install ArgoCD

Install ArgoCD to adopt a GitOps approach and automate installation.  

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Fork this repo and update the repoURL in [argocd/bootstrap.yaml](./argocd/bootstrap.yaml), then apply the file to your cluster to create Argo Applications.

# Install Cert-Manager if not available

A certificate manager is required to automatically manage the process of creating, renewing and using Event Endpoint Management internal and system-to-system certificates. It is also used by default for managing Event Endpoint Management certificates that are visible to clients and users, simplifying their configuration.

`kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.18.0/cert-manager.yaml`

# EEM Install

See https://ibm.github.io/event-automation/eem/installing/installing-on-kubernetes/ for details.

Prereq is a Kubernetes platforms that support the Red Hat Universal Base Images (UBI) containers, versions 1.25 to 1.32

## 1. Install EEM Operator

The Event Endpoint Management operator requires the following cluster-scoped permissions, even if the operator is set to manage instances in a single namespace:

- **Permission to retrieve storage classes**: The Event Endpoint Management operator uses admission webhooks to provide immediate validation and feedback about the creation and modification of the Event Manager and operator-managed Event Gateway instances. The permission to retrieve storage classes is used by the webhooks to find a default storage class.
- **Permission to list specific CustomResourceDefinitions**: This allows Event Endpoint Management to identify whether other optional dependencies have been installed into the cluster.

`kubectl create namespace ibm-event-automation`

`helm repo add ibm-helm https://raw.githubusercontent.com/IBM/charts/master/repo/ibm-helm`

`helm install eem-crds ibm-helm/ibm-eem-operator-crd -n ibm-event-automation`

`helm install eventendpointmanagement ibm-helm/ibm-eem-operator -n "ibm-event-automation`

## 2. Create Event Manager instance

Create an image pull secret to access the IBM container registry

`kubectl create secret docker-registry ibm-entitlement-key --docker-username=cp --docker-password="<your-entitlement-key>" --docker-server="cp.icr.io" -n "ibm-event-automation"`

Find the ingress subdomain for the cluster and replace the placeholders in the sample CR from here: https://github.com/IBM/ibm-event-automation/blob/main/event-endpoint-management/cr-examples/eventendpointmanagement/kubernetes/quick-start.yaml

Replace the ingress class name if necessary. The default is nginx.

`kubectl apply -f <custom-resource-file-path> -n <target-namespace>`

Add local users to be able to login to the UI. See https://ibm.github.io/event-automation/eem/security/managing-access/ and https://ibm.github.io/event-automation/eem/security/user-roles/

Login to the UI with one of the sample users.

## 3. Create Event Gateway instance

Login to the Event manager UI and 

