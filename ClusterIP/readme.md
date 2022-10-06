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
For Kubernetes versions lower than 1.18, please use Helm chart version 0.0.14 as follows: helm install --skip-crds -f values.yaml <new-chart-name> f5-stable/f5-bigip-ctlr --version 0.0.14.


## Chart parameters¶
Parameter	Required	Default	Description
bigip_login_secret	Required	f5-bigip-ctlr-login	Secret that contains BIG-IP login credentials.
args.bigip_url	Required	N/A	The management IP for your BIG-IP device.
args.bigip_partition	Required	f5-bigip-ctlr	BIG-IP partition the CIS Controller will manage.
args.namespaces	Optional	N/A	List of Kubernetes namespaces which CIS will monitor.
rbac.create	Optional	true	Create ClusterRole and ClusterRoleBinding.
serviceAccount.name	Optional	f5-bigip-ctlr- serviceaccount	Name of the ServiceAccount for CIS controller.
serviceAccount.create	Optional	true	Create service account for the CIS controller.
namespace	Optional	kube-system	Name of namespace CIS will use to create deployment and other resources.
image.user	Optional	f5networks	CIS Controller image repository username.
image.repo	Optional	k8s-bigip-ctlr	CIS Controller image repository name.
image.pullPolicy	Optional	Always	CIS Controller image pull policy.
image.pullSecrets	Optional	N/A	List of secrets of container registry to pull image.
version	Optional	latest	CIS Controller image tag.
nodeSelector	Optional	N/A	Dictionary of Node selector labels.
tolerations	Optional	N/A	Array of labels.
limits_cpu	Optional	100m	CPU limits for the pod.
limits_memory	Optional	512Mi	Memory limits for the pod.
requests_cpu	Optional	100m	CPU request for the pod.
requests_memory	Optional	512Mi	Memory request for the pod.
affinity	Optional	N/A	Dictionary of affinity.
securityContext	Optional	N/A	Dictionary of securityContext.
ingressClass.ingressClassName	Optional	f5	Name of ingress class.
ingressClass.isDefaultIngressController	Optional	false	CIS will monitor all the ingress resources if set true.
ingressClass.create	Optional	true	Create ingress class.

## Uninstalling Helm Chart¶
Run the following command to uninstall the chart.

```
helm del f5-ingress
```
