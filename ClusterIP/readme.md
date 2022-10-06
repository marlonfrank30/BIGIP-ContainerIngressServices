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
<h3>Chart parameters<a class="headerlink" href="#chart-parameters" title="Permalink to this headline">¶</a></h3>
<table border="1" class="styled-table docutils">
<colgroup>
<col width="29%" />
<col width="6%" />
<col width="14%" />
<col width="51%" />
</colgroup>
<thead valign="bottom">
<tr class="row-odd"><th class="left-align head">Parameter</th>
<th class="left-align head">Required</th>
<th class="left-align head">Default</th>
<th class="left-align head">Description</th>
</tr>
</thead>
<tbody valign="top">
<tr class="row-even"><td class="left-align">bigip_login_secret</td>
<td class="left-align">Required</td>
<td class="left-align">f5-bigip-ctlr-login</td>
<td class="left-align">Secret that contains BIG-IP login credentials.</td>
</tr>
<tr class="row-odd"><td class="left-align">args.bigip_url</td>
<td class="left-align">Required</td>
<td class="left-align">N/A</td>
<td class="left-align">The management IP for your BIG-IP device.</td>
</tr>
<tr class="row-even"><td class="left-align">args.bigip_partition</td>
<td class="left-align">Required</td>
<td class="left-align">f5-bigip-ctlr</td>
<td class="left-align">BIG-IP partition the CIS Controller will manage.</td>
</tr>
<tr class="row-odd"><td class="left-align">args.namespaces</td>
<td class="left-align">Optional</td>
<td class="left-align">N/A</td>
<td class="left-align">List of Kubernetes namespaces which CIS will monitor.</td>
</tr>
<tr class="row-even"><td class="left-align">rbac.create</td>
<td class="left-align">Optional</td>
<td class="left-align">true</td>
<td class="left-align">Create ClusterRole and ClusterRoleBinding.</td>
</tr>
<tr class="row-odd"><td class="left-align">serviceAccount.name</td>
<td class="left-align">Optional</td>
<td class="left-align">f5-bigip-ctlr-
serviceaccount</td>
<td class="left-align">Name of the ServiceAccount for CIS controller.</td>
</tr>
<tr class="row-even"><td class="left-align">serviceAccount.create</td>
<td class="left-align">Optional</td>
<td class="left-align">true</td>
<td class="left-align">Create service account for the CIS controller.</td>
</tr>
<tr class="row-odd"><td class="left-align">namespace</td>
<td class="left-align">Optional</td>
<td class="left-align">kube-system</td>
<td class="left-align">Name of namespace CIS will use to create deployment and other resources.</td>
</tr>
<tr class="row-even"><td class="left-align">image.user</td>
<td class="left-align">Optional</td>
<td class="left-align">f5networks</td>
<td class="left-align">CIS Controller image repository username.</td>
</tr>
<tr class="row-odd"><td class="left-align">image.repo</td>
<td class="left-align">Optional</td>
<td class="left-align">k8s-bigip-ctlr</td>
<td class="left-align">CIS Controller image repository name.</td>
</tr>
<tr class="row-even"><td class="left-align">image.pullPolicy</td>
<td class="left-align">Optional</td>
<td class="left-align">Always</td>
<td class="left-align">CIS Controller image pull policy.</td>
</tr>
<tr class="row-odd"><td class="left-align">image.pullSecrets</td>
<td class="left-align">Optional</td>
<td class="left-align">N/A</td>
<td class="left-align">List of secrets of container registry to pull image.</td>
</tr>
<tr class="row-even"><td class="left-align">version</td>
<td class="left-align">Optional</td>
<td class="left-align">latest</td>
<td class="left-align">CIS Controller image tag.</td>
</tr>
<tr class="row-odd"><td class="left-align">nodeSelector</td>
<td class="left-align">Optional</td>
<td class="left-align">N/A</td>
<td class="left-align">Dictionary of Node selector labels.</td>
</tr>
<tr class="row-even"><td class="left-align">tolerations</td>
<td class="left-align">Optional</td>
<td class="left-align">N/A</td>
<td class="left-align">Array of labels.</td>
</tr>
<tr class="row-odd"><td class="left-align">limits_cpu</td>
<td class="left-align">Optional</td>
<td class="left-align">100m</td>
<td class="left-align">CPU limits for the pod.</td>
</tr>
<tr class="row-even"><td class="left-align">limits_memory</td>
<td class="left-align">Optional</td>
<td class="left-align">512Mi</td>
<td class="left-align">Memory limits for the pod.</td>
</tr>
<tr class="row-odd"><td class="left-align">requests_cpu</td>
<td class="left-align">Optional</td>
<td class="left-align">100m</td>
<td class="left-align">CPU request for the pod.</td>
</tr>
<tr class="row-even"><td class="left-align">requests_memory</td>
<td class="left-align">Optional</td>
<td class="left-align">512Mi</td>
<td class="left-align">Memory request for the pod.</td>
</tr>
<tr class="row-odd"><td class="left-align">affinity</td>
<td class="left-align">Optional</td>
<td class="left-align">N/A</td>
<td class="left-align">Dictionary of affinity.</td>
</tr>
<tr class="row-even"><td class="left-align">securityContext</td>
<td class="left-align">Optional</td>
<td class="left-align">N/A</td>
<td class="left-align">Dictionary of securityContext.</td>
</tr>
<tr class="row-odd"><td class="left-align">ingressClass.ingressClassName</td>
<td class="left-align">Optional</td>
<td class="left-align">f5</td>
<td class="left-align">Name of ingress class.</td>
</tr>
<tr class="row-even"><td class="left-align">ingressClass.isDefaultIngressController</td>
<td class="left-align">Optional</td>
<td class="left-align">false</td>
<td class="left-align">CIS will monitor all the ingress resources if set true.</td>
</tr>
<tr class="row-odd"><td class="left-align">ingressClass.create</td>
<td class="left-align">Optional</td>
<td class="left-align">true</td>
<td class="left-align">Create ingress class.</td>
</tr>
</tbody>
</table>
<div class="line-block">
<div class="line"><br /></div>

https://clouddocs.f5.com/containers/latest/userguide/kubernetes/#chart-parameters

## Uninstalling Helm Chart¶
Run the following command to uninstall the chart.

```
helm del f5-ingress
```
