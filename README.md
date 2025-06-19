# eem-iks

EEM Documentation: https://ibm.github.io/event-automation/eem/

EEM CR examples: https://github.com/IBM/ibm-event-automation/tree/main/event-endpoint-management

# Install Argo CD

```kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml```


# EEM Install

See https://ibm.github.io/event-automation/eem/installing/installing-on-kubernetes/ for details.

## Install Cert-Manager if not available

`kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.18.0/cert-manager.yaml`

## Install EEM Operator

Prereq: Kubernetes platforms that support the Red Hat Universal Base Images (UBI) containers, versions 1.25 to 1.32

The Event Endpoint Management operator requires the following cluster-scoped permissions, even if the operator is set to manage instances in a single namespace:

- **Permission to retrieve storage classes**: The Event Endpoint Management operator uses admission webhooks to provide immediate validation and feedback about the creation and modification of the Event Manager and operator-managed Event Gateway instances. The permission to retrieve storage classes is used by the webhooks to find a default storage class.
- **Permission to list specific CustomResourceDefinitions**: This allows Event Endpoint Management to identify whether other optional dependencies have been installed into the cluster.

`kubectl create namespace ibm-event-automation`

`helm repo add ibm-helm https://raw.githubusercontent.com/IBM/charts/master/repo/ibm-helm`

`helm install eem-crds ibm-helm/ibm-eem-operator-crd -n ibm-event-automation`

`helm install eventendpointmanagement ibm-helm/ibm-eem-operator -n "ibm-event-automation`

## Create Event Manager instance

Create an image pull secret to access the IBM container registry

`kubectl create secret docker-registry ibm-entitlement-key --docker-username=cp --docker-password="<your-entitlement-key>" --docker-server="cp.icr.io" -n "ibm-event-automation"`

Find the ingress subdomain for the cluster and replace the placeholders in the sample CR from here: https://github.com/IBM/ibm-event-automation/blob/main/event-endpoint-management/cr-examples/eventendpointmanagement/kubernetes/quick-start.yaml

Replace the ingress class name if necessary. The default is nginx.

`kubectl apply -f <custom-resource-file-path> -n <target-namespace>`

Add local users to be able to login to the UI. See https://ibm.github.io/event-automation/eem/security/managing-access/ and https://ibm.github.io/event-automation/eem/security/user-roles/

Login to the UI with one of the sample users.

## Create Event Gateway instance

Login to the Event manager UI and 

