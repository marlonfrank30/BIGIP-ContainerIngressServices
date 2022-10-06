## Introduction to Helm.

This is the simplest way to install CIS on a Kubernetes cluster. Helm is a package manager for Kubernetes. Helm is Kubernetes version of YUM or APT. Helm deploys something called charts, which you can think of as a packaged application. It is a collection of all your versioned, pre-configured application resources which can be deployed as one unit.


## Installing CIS Using Helm Charts¶

From Script
Helm now has an installer script that will automatically grab the latest version of Helm and install it locally.

You can fetch that script, and then execute it locally. It's well documented so that you can read through it and understand what it is doing before you run it.

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

more info is available here https://helm.sh/docs/intro/install/

## Installing BIGIP CIS in ClusterIP mode
Add BIG-IP credentials as K8S secrets.

For Kubernetes, use the following command:

```
kubectl create secret generic f5-bigip-ctlr-login -n kube-system --from-literal=username=admin --from-literal=password=<password>
```
Add the CIS chart repository in Helm using following command:

```
helm repo add f5-stable https://f5networks.github.io/charts/stable
```

Install the Helm chart using the following command:

```
helm install -f values.yaml f5-ingress f5-stable/f5-bigip-ctlr
```

Note
For Kubernetes versions lower than 1.18, please use Helm chart version 0.0.14 as follows: 
```
helm install --skip-crds -f values.yaml <new-chart-name> f5-stable/f5-bigip-ctlr --version 0.0.14.
```

## Chart parameters¶
https://clouddocs.f5.com/containers/latest/userguide/kubernetes/#chart-parameters

## Uninstalling Helm Chart¶
Run the following command to uninstall the chart.

```
helm del f5-ingress
```
