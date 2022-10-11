## Introduction to Helm

This is the simplest way to install CIS on a Kubernetes cluster. Helm is a package manager for Kubernetes. Helm is Kubernetes version of YUM or APT. Helm deploys something called charts, which you can think of as a packaged application. It is a collection of all your versioned, pre-configured application resources which can be deployed as one unit.


## Installing Helm Charts

From Script
Helm now has an installer script that will automatically grab the latest version of Helm and install it locally.

You can fetch that script, and then execute it locally. It's well documented so that you can read through it and understand what it is doing before you run it.

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

more info is available here https://helm.sh/docs/intro/install/


## Deploying Antrea on a Cluster with Existing CNI
The instructions above only apply when deploying Antrea in a new cluster. If you need to migrate your existing cluster from another CNI plugin to Antrea, you will need to do the following:

* Delete previous CNI, including all resources (K8s objects, iptables rules, interfaces, …) created by that CNI.
* Deploy Antrea (next item below).
* Restart all Pods in the CNI network in order for Antrea to set-up networking for them. This does not apply to Pods which use the Node’s network namespace (i.e. Pods configured with hostNetwork: true). You may use kubectl drain to drain each Node or reboot all your Nodes.

While this is in-progress, networking will be disrupted in your cluster. After deleting the previous CNI, existing Pods may not be reachable anymore.

For example, when migrating from Flannel to Antrea, you will need to do the following:

* 1) Delete Flannel with 
```
kubectl delete -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml <path to your Flannel YAML manifest>.
```

* 2) Delete Flannel bridge and tunnel interface with 
```
ip link delete flannel.1 && ip link delete flannel cni0 on each Node.
```
  
* 3) Ensure requirements are satisfied.

* 4) [Deploy Antrea](#Installing Antrea with Helm)
* 5) Drain and uncordon Nodes one-by-one. For each Node, run kubectl drain --ignore-daemonsets <node name> && kubectl uncordon <node name>. The --ignore-daemonsets flag will ignore DaemonSet-managed Pods, including the Antrea Agent Pods. If you have any other DaemonSet-managed Pods (besides the Antrea ones and system ones such as kube-proxy), they will be ignored and will not be drained from the Node. Refer to the Kubernetes documentation for more information. Alternatively, you can also restart all the Pods yourself, or simply reboot your Nodes.
  
  
## Prerequisites 

Ensure that the necessary requirements for running Antrea are met https://antrea.io/docs/main/docs/getting-started/#ensuring-requirements-are-satisfied.

Ensure that Helm 3 is installed. 
We recommend using a recent version of Helm if possible. Refer to the Helm documentation for compatibility between Helm and Kubernetes versions.

## Adding Antrea Helm chart's repository:
```
helm repo add antrea https://charts.antrea.io
helm repo update
```
## Installing Antrea with Helm 
To install the Antrea Helm chart, use the following command:
```
helm install antrea antrea/antrea --namespace kube-system
```
This will install the latest available version of Antrea. You can also install a specific version of Antrea (>= v1.8.0) with --version <TAG>.

## Upgrading Antrea with Helm
  
To upgrade the Antrea Helm chart, use the following commands:
  
**Note:** Upgrading CRDs requires an extra step; see explanation below
```
kubectl apply -f https://github.com/antrea-io/antrea/releases/download/v1.8.0/antrea-crds.yml
helm upgrade antrea antrea/antrea --namespace kube-system --version v1.8.0
```
  
**Source:** https://antrea.io/docs/main/docs/helm/

## Installing BIGIP CIS in NodePortLocal mode
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

**Note:**
For Kubernetes versions lower than 1.18, please use Helm chart version 0.0.14 as follows: 
```
helm install --skip-crds -f values.yaml <new-chart-name> f5-stable/f5-bigip-ctlr --version 0.0.14.
```

## Chart parameters
<h3>Chart parameters<a class="headerlink" href="#chart-parameters" title="Permalink to this headline"></a></h3>
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

**Source:** https://clouddocs.f5.com/containers/latest/userguide/kubernetes/#chart-parameters

## Creating VXLAN Tunnels on Kubernetes Cluster
This configuration is for Standalone BIG-IP.

Log in to BIG-IP and create a partition called kube80 for CIS.
```
tmsh create auth partition kube80
```
Create a VXLAN profile.
```
tmsh create net tunnels vxlan fl-vxlan port 8472 flooding-type none
```
Create a VXLAN tunnel.
```
tmsh create net tunnels tunnel fl-vxlan key 1 profile fl-vxlan local-address 10.1.10.249
```
Create the VXLAN tunnel self IP.
```
tmsh create net self ocp-cis-ingress-self address 10.244.20.249/255.255.0.0 allow-service none vlan fl-vxlan
```
Save the configuration.
```
tmsh save sys config
```
Before deploying CIS in ClusterIP mode, you need to configure BIG-IP as a node in the Kubernetes cluster. To do so you will need to modify f5-node.yaml with the MAC address auto-created from the previous steps. Run the following command at bigip1. Copy the displayed MAC Address.
```
tmsh show net tunnels tunnel k8s-tunnel all-properties
```

<li><p class="first">Update the MAC address obtained in the previous step to the following YAML file:</p>
<div class="literal-block-wrapper docutils container" id="id8">
<div class="code-block-caption"><span class="caption-text">f5-node.yaml (line 9)</span><a class="headerlink" href="#id8" title="Permalink to this code"></a></div>
<div class="highlight-yaml notranslate"><div class="highlight"><table class="highlighttable"><tr><td class="linenos"><div class="linenodiv"><pre><span class="normal"> 1</span>
<span class="normal"> 2</span>
<span class="normal"> 3</span>
<span class="normal"> 4</span>
<span class="normal"> 5</span>
<span class="normal"> 6</span>
<span class="normal"> 7</span>
<span class="normal"> 8</span>
<span class="normal"> 9</span>
<span class="normal">10</span>
<span class="normal">11</span>
<span class="normal">12</span>
<span class="normal">13</span>
<span class="normal">14</span></pre></div></td><td class="code"><div><pre><span></span><span class="nt">apiVersion</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">v1</span><span class="w"></span>
<span class="nt">kind</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">Node</span><span class="w"></span>
<span class="nt">metadata</span><span class="p">:</span><span class="w"></span>
<span class="w">  </span><span class="nt">name</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">bigip1</span><span class="w"></span>
<span class="w">  </span><span class="nt">annotations</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="c1">#Replace IP with self IP for your deployment</span><span class="w"></span>
<span class="w">    </span><span class="nt">flannel.alpha.coreos.com/public-ip</span><span class="p">:</span><span class="w"> </span><span class="s">&quot;10.1.10.249&quot;</span><span class="w"></span>
<span class="w">    </span><span class="c1">#Replace MAC with your BIG-IP Flannel VXLAN Tunnel MAC</span><span class="w"></span>
<span class="w">    </span><span class="nt">flannel.alpha.coreos.com/backend-data</span><span class="p">:</span><span class="w"> </span><span class="s">&#39;{&quot;VtepMAC&quot;:&quot;2c:c2:60:23:0c:58&quot;}&#39;</span><span class="w"></span>
<span class="w">    </span><span class="nt">flannel.alpha.coreos.com/backend-type</span><span class="p">:</span><span class="w"> </span><span class="s">&quot;vxlan&quot;</span><span class="w"></span>
<span class="w">    </span><span class="nt">flannel.alpha.coreos.com/kube-subnet-manager</span><span class="p">:</span><span class="w"> </span><span class="s">&quot;true&quot;</span><span class="w"></span>
<span class="nt">spec</span><span class="p">:</span><span class="w"></span>
<span class="w">  </span><span class="c1">#Replace Subnet with your BIG-IP Flannel Subnet</span><span class="w"></span>
<span class="w">  </span><span class="nt">podCIDR</span><span class="p">:</span><span class="w"> </span><span class="s">&quot;10.244.20.0/24&quot;</span><span class="w"></span>
</pre></div></td></tr></table></div>
</div>
</div>
</li>
<li><p class="first">Create the BIG-IP node:</p>
<div class="highlight-default notranslate"><div class="highlight"><pre><span></span><span class="n">kubectl</span> <span class="n">create</span> <span class="o">-</span><span class="n">f</span> <span class="n">f5</span><span class="o">-</span><span class="n">node</span><span class="o">.</span><span class="n">yaml</span>
</pre></div>
</div>
</li>
<li><p class="first">Verify “bigip1” node is created:</p>
<div class="highlight-default notranslate"><div class="highlight"><pre><span></span><span class="n">kubectl</span> <span class="n">get</span> <span class="n">nodes</span>
</pre></div>
</div>
</li>
</ol>

## Create AS3 ConfigMap
This section uses the command
```
kubectl apply -f configMap4nginxAS3.yaml
```  

## Create any webserver inside the cluster to be used by CIS
I've used nginx as my webserver enginee but any other could have been installed such as apache

```
kubectl create -f nginxdeployment.yaml
kubectl create -f nginxsvc.yaml
```

## Uninstalling Helm Chart
Run the following command to uninstall the chart.

```
helm del f5-ingress
```
