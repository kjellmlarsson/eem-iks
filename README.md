# Event Endpoint Management on Kubernetes example deployment

This repo contains sample Kubernetes manifests and ArgoCD config to install Event Endpoint Management on an (IBM Cloud) Kubernetes cluster. Use this to either setup automated install of EEM with ArgoCD or use the manifests and apply them manually.

The full EEM Documentation with detailed instructions is available here: https://ibm.github.io/event-automation/eem/

An overview of the installation steps is available here: https://ibm.github.io/event-automation/eem/installing/overview/

EEM CR examples: https://github.com/IBM/ibm-event-automation/tree/main/event-endpoint-management

# Install ArgoCD

Installing ArgoCD is optional. Skip this step if you prefer to apply manifests manually.

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Fork this repo and update the repoURL in [argocd/bootstrap.yaml](./argocd/bootstrap.yaml) and in [argocd/kustomization.yaml](./argocd/kustomization.yaml)

# Install Cert-Manager

A certificate manager is required to **automatically** manage the process of creating, renewing and using Event Endpoint Management internal and system-to-system certificates. 
It is also used by default for managing Event Endpoint Management certificates that are visible to clients and users, simplifying their configuration.

You can also create certificates **manually** and provide them to Event Endpoint Management as Kubernetes secrets.

Documentation: https://ibm.github.io/event-automation/eem/installing/prerequisites/#certificate-management

# EEM Install

Event Endpoint Management is an operator-based release and uses custom resources to deploy and manage the lifecycle of your Event Endpoint Management installation. The operator and its CRDs are installed using Helm.

See https://ibm.github.io/event-automation/eem/installing/installing-on-kubernetes/ for details.

Prereq is a Kubernetes platform that support the Red Hat Universal Base Images (UBI) containers, versions 1.25 to 1.32

## 1. Install EEM Operator

### Required Cluster-scope Permissions

The Event Endpoint Management operator requires the following cluster-scoped permissions, even if the operator is set to manage instances in a single namespace:

- **Permission to retrieve storage classes**: The Event Endpoint Management operator uses admission webhooks to provide immediate validation and feedback about the creation and modification of the Event Manager and operator-managed Event Gateway instances. The permission to retrieve storage classes is used by the webhooks to find a default storage class.
- **Permission to list specific CustomResourceDefinitions**: This allows Event Endpoint Management to identify whether other optional dependencies have been installed into the cluster. 

### Install of CRDs and Operator

#### Installing in an offline environment
If you're installing in an offline environment, mirror the images to your internal registry and install CRDs and Operators according to instructions here: https://ibm.github.io/event-automation/eem/installing/offline/

#### Installing in an online environment using ArgoCD
There are two ArgoCD applications configured, one for the EEM CRDs ([crds.yaml](./argocd/crds.yaml)) and one for the operator ([operator.yaml](./argocd/operator.yaml)). Applying the [bootstrap.yaml](./argocd/bootstrap.yaml) file to your clusters ArgoCD namespace will install these using Helm.

#### Installing in an online environment manually

```
helm repo add ibm-helm https://raw.githubusercontent.com/IBM/charts/master/repo/ibm-helm
kubectl create namespace ibm-event-automation
helm install eem-crds ibm-helm/ibm-eem-operator-crd -n ibm-event-automation
helm install eventendpointmanagement ibm-helm/ibm-eem-operator -n ibm-event-automation
```

Parameters for the operator install: https://ibm.github.io/event-automation/eem/installing/installing-on-kubernetes/#installing-the-operator

## 2. Create Event Manager instance

### Ingress

SSL passthrough must be enabled in the ingress controller for your Event Endpoint Management services to work.

Ingress prereqs: https://ibm.github.io/event-automation/eem/installing/prerequisites/#ingress-controllers\
Ingress configuration: https://ibm.github.io/event-automation/eem/installing/configuring/#configuring-ingress

### Storage

If not using ephemeral storage, Event Endpoint Management requires persistent volumes with the following capabilities: volumeMode: Filesystem and accessMode: ReadWriteOnce. 

See https://ibm.github.io/event-automation/support/matrix/#event-endpoint-management\
Storage prereqs: https://ibm.github.io/event-automation/eem/installing/prerequisites/#data-storage-requirements\
Storage configuration: https://ibm.github.io/event-automation/eem/installing/configuring/#enabling-persistent-storage

### Authentication

Authentication to the Event Manager UI can be either local, with users, passwords and roles stored in Kubernetes secrets as json, or using OIDC.

This example uses local authentication.

Sample secrets for local are created with admin, author and viewer users. Remove these and create them manually if you don't want these cleartext users and passwords in your repo. You can change this here: [./components/eventmanager/kustomization.yaml](./components/eventmanager/kustomization.yaml)

For manually adding local users, see. See https://ibm.github.io/event-automation/eem/security/managing-access/ and https://ibm.github.io/event-automation/eem/security/user-roles/

### TLS

For Event Manager, select one of the following ways of configuring TLS:

* [Operator-configured CA](https://ibm.github.io/event-automation/eem/installing/configuring/#operator-configured-ca-certifcate)  - used in this example
* [User-provided CA certificate](https://ibm.github.io/event-automation/eem/installing/configuring/#user-provided-ca-certificate)
* [User-provided (server) certificate](https://ibm.github.io/event-automation/eem/installing/configuring/#user-provided-certificates)

### Create image pull secret

Create an image pull secret to access the IBM container registry

`kubectl create secret docker-registry ibm-entitlement-key --docker-username=cp --docker-password="<your-entitlement-key>" --docker-server="cp.icr.io" -n "ibm-event-automation"`

### Update event manager CR sample with values for your cluster

Update the event manager cr sample with correct ingress subdomain, ingress class and storage class names here: [eventendpointmanagement.yaml](./components/eventmanager/eventendpointmanagement.yaml)

## 3. Create Event Gateway instance

Full documentation: https://ibm.github.io/event-automation/eem/installing/install-gateway/

The Event Endpoint Management UI is used to generate configuration for the Gateway instance. 3 deployment methods exist:

* Docker container
* Kubernetes deployment
* Operator-managed Custom Resource - used in this example.

Login to the Event Endpoint Management UI to create the configuration. An example configuration (update with your manager endpoint, ingress subdomain, apikey to event manager and certificate references as necessary) is here: [./components/gateway/gateway_cr.yaml](./components/gateway/gateway_cr.yaml)

### TLS

For Event Gateway, provide a CA certificate for the Event Manager, and a certificate securing TLS to Kafka clients. Examples (update with your ingress subdomain) here: [./components/gateway/tls](./components/gateway/tls)

