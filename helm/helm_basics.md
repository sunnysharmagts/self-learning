# Helm
Helm is kubernetes package manager. It helps you manage kubernetes applications. Helm Charts helps you define, install and even upgrade kubernetes applications.

## Prerequisites
- A Kubernetes cluster.
- Security configurations
- Helm & Tiller

### Installing kubernetes

Kubernetes configuration resides in `$HOME/.kube/config`. Helm figures out where to install tiller by reading this configuration. This kubernetes configuration is a default configuration. `KUBECONFIG` can also be customized to other file. This is the same configuration that kubectl uses. In case KUBECONFIG is not configured then `$HOME/.kube/config` is taken by default.

To checkout which cluster would tiller be install to, you can run
`kubectl config current-context` or `kubectl cluster-info`

### Understanding Security context

If helm is used on a cluster that we completely control, like minikube or cluster on a private network in which sharing is not a concern, the default installation - which applies no security configuration is fine.

However, if your cluster is exposed to a larger network or if you share your cluster with others like production cluster, you must take extra steps to protect your cluster.

`Helm init` easily installs tiller, the server side component with which helm corresponds, without any security configurations.

#### Who needs security configurations ?

- Clusters that are exposed to uncontrolled network environments where untrusted application can access the network environment.
- Clusters that are for many people to use i.e multitenant clusters.
- Clusters that have access to or use high-value data or networks of any type.

#### Understanding the security context of your cluster

`helm init` installs `Tiller` into the cluster in the `kube-system` namespace and without any RBAC (Role Based Access Control) rules applied because it helps your to be productive immediately. 

There are 4 areas to consider when securing a tiller installation:

1. RBAC or Role based access control.
2. Tiller's gRPC and its usage by Helm.
3. Tiller release information.
4. Helm charts.

#### RBAC

RBAC are there if credentials are misused or bugs exist. Even when an identity is hijacked, the identity has only so many permissions to a control space. This effectivvely adds a layer of security to limit the scope of any attack to that identity. 

Helm and Tiller are designed to install, remove, modify logical applications that may contain many services interacting together. Specific users and teams - developers, operators, system and network administrators will have their own portions of cluster in which they can use helm and tiller without risking other portions of the cluster.

#### Enable TLS

Use `helm init` along with `--tiller-tls-verify` to install tiller with tls enabled and to verify remote certificates and all other helm command should use -tls option. When Helm clients are connecting from outside the cluster, the security between the helm client and the API server is managed by kubernetes itself. This link should be secure. Note that if we are using the TLS configuration then not even the kubernetes API server has the access to the encrypted messages between client and tiller.

#### Tiller's Release Information

For historical reasons, tiller stores its release information in ConfigMaps. This release infomation should be stored in secrets. Secrets are k8 accepted mechanism for storing sensitive configuration data. While secrets doesn't offer many protections, K8 cluster management software treats them differently from other objects. Thus secrets should be used to store releases.

`--storage=secret` flag in tiller-deploy deployment will enable this feature. This requires modifying the deployment or using `helm init --override 'spec.template.spec.containers[0].command'='{/tiller,--storage=secret}` 

#### Helm charts

Charts are a kind of package that not only installs containers you may or may not have validated yourself, but it may also install into more than one namespace. As with all shared software, in a controlled or shared environment you must validate all software you install yourself before you install it. If you have secured Tiller with TLS and have installed it with permissions to only one or a subset of namespaces, some charts may fail to install – but in these environments, that is exactly what you want. If you need to use the chart, you may have to work with the creator or modify it yourself in order to use it securely in a multitenant cluster with proper RBAC rules applied. The helm template command renders the chart locally and displays the output.


## Initialize Helm and Install Tiller
Once you have Helm ready, you can initialize the local CLI and also install Tiller into your Kubernetes cluster in one step `helm init --history-max 200`.

TIP: Setting --history-max on helm init is recommended as configmaps and other objects in helm history can grow large in number if not purged by max limit. Without a max history set the history is kept indefinitely, leaving a large number of records for helm and tiller to maintain.

Upgrade Tiller: `helm init -- upgrade`
Install chart: `helm install <chart-name>`
To get idea of features of installed chart: `helm inspect <chart-name>`
See the latest release: `helm ls`
List of all deployed release: `helm list`
Delete a release: `helm delete <release-name>`. This will uninstall that particular release from k8 but still you will be able to see information about that release by using `helm status <release-name>`

Whenever you install a chart, a new release is created. So one chart can be installed multiple times in same cluster. And each one can be separately managed and upgraded. 

Because Helm tracks your releases even after you’ve deleted them, you can audit a cluster’s history, and even undelete a release (with `helm rollback`).

## Deleting or Reinstalling Tiller

Because tiller stores its data in configMaps, you can safely delete and re-install it without worrying about losing any data. The recommended way is `kubectl delete deployment tiller-deploy --namespace kube-system` or `helm reset`. Tiller can be re-installed with `helm init`

## Advanced Usage

`helm init` provides additional flags for modifying tiller's deployment manifest before its installed. 

- The `--node-selectors` flag allows us to specify the node labels required for scheduling the Tiller pod. For eg: `helm init --node-selectors "beta.kubernetes.io/os"="linux"`
- `--override` allows you to specify properties of Tiller’s deployment manifest. For eg: `helm init --override metadata.annotations."deployment\.kubernetes\.io/revision"="1"`
- The `--output flag` allows us skip the installation of Tiller’s deployment manifest and simply output the deployment manifest to stdout in either JSON or YAML format. The output may then be modified with tools like jq and installed manually with kubectl. `helm init --output json`



