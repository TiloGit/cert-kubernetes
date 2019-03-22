# Install IBM Operational Decision Manager 8.10.1 on Certified Kubernetes

The installation of Operational Decision Manager 8.10.1 uses the [`ibm-odm-prod-2.1.0`](helm-charts/ibm-odm-prod-2.1.0.tgz) Helm chart, also known as the ODM for production Helm chart. The chart is a package of preconfigured Kubernetes resources that bootstraps an ODM for production deployment on a Kubernetes cluster. You customize the deployment by changing and adding configuration parameters. The default values are appropriate to a production environment, but it is likely that you want to configure at least the security of your kubernetes deployment.

The following architectures are supported for Operational Decision Manager 8.10.1 on Certified Kubernetes:
- AMD64 (or x86_64), which is the 64-bit edition for Linux x86.

The [`ibm-odm-prod-2.1.0`](helm-charts/ibm-odm-prod-2.1.0.tgz) Helm chart includes five containers corresponding to the following services.
- Decision Center Business Console and Enterprise Console
- Decision Server Console
- Decision Server Runtime
- Decision Server Runner
- (Optional) Internal PostgreSQL DB

The services require CPU and memory resources. The following table lists the minimum requirements that are used as default values.

| Service  | CPU Minimum (m) | Memory Minimum (Mi) |
| ---------- | ----------- | ------------------- |
| Decision Center | 500           | 512                  |
| Decision Runner     | 500           | 512                  |
| Decision Server Console  | 500           | 512                  |
| Decision Server Runtime    | 500           | 512                   |
| **Total**  | **2000** (2CPU)     | **2048** (2Gb)             |
| (Optional) Internal DB    | 500           | 512                   |

## Before you install

Go to the [IBM Digital Business Automation for MultiCloud 18.0.x](https://www.ibm.com/support/knowledgecenter/SSYHZ8_18.0.x/com.ibm.dba.install/k8s_topics/tsk_install_odm.html) Knowledge Center and choose which customizations you want to apply to your installation.
   * [Configuring PVUs](https://www.ibm.com/support/knowledgecenter/SSYHZ8_18.0.x/com.ibm.dba.install/k8s_topics/tsk_config_pvu.html)
   * [Defining the security certificate](https://www.ibm.com/support/knowledgecenter/SSYHZ8_18.0.x/com.ibm.dba.install/k8s_topics/tsk_replace_security_certificate.html)
   * [Configuring the LDAP and user registry](https://www.ibm.com/support/knowledgecenter/SSYHZ8_18.0.x/com.ibm.dba.install/k8s_topics/con_config_user_registry.html)
   * [Configuring a custom external database](https://www.ibm.com/support/knowledgecenter/SSYHZ8_18.0.x/com.ibm.dba.install/k8s_topics/tsk_custom_external_db.html)
   * [Configuring the ODM event emitter](https://www.ibm.com/support/knowledgecenter/SSYHZ8_18.0.x/com.ibm.dba.install/k8s_topics/tsk_custom_emitters.html)

After you noted the values for all of the configuration parameters that you need to customize Operational Decision Manager, choose one of the following options to help you complete the installation.

> **Note**: The [configuration](configuration) folder provides sample configuration files that you might find useful. Download the files and edit them for your own customizations.

## Option 1: Install Operational Decision Manager by using the Helm chart and Tiller
A [Helm chart](https://helm.sh/) is a Package Manager for Kubernetes to help you manage (install/upgrade/update) your Kubernetes deployment. If you are using Helm on a cluster that you completely control, like Minikube or a cluster on a private network in which sharing is not a concern, the default installation that applies no security configuration is the easiest option.

However, if your cluster is exposed to a larger network or if you share your cluster with others – production clusters fall into this category – you must secure your installation to prevent careless or malicious actors from damaging the cluster or its data. To secure Helm for use in a production environment and other multi-tenant scenarios, see [Securing a Helm installation](https://helm.sh/docs/using_helm/#securing-your-helm-installation).

Refer to [helm-charts/README.md](helm-charts/README.md)

## Option 2: Install Operational Decision Manager by using Kubernetes YAML
If you prefer to use a simpler deployment process that uses a native Kubernetes authorization mechanism (RBAC) instead of Helm and Tiller, use the Helm command line interface (CLI) to generate a Kubernetes manifest. If you choose to use Kubernetes YAML you cannot use certain capabilities of Helm to manage your deployment.

Refer to [k8s-yaml/README.md](k8s-yaml/README.md)

## Option 3: Instructions for example Certified Kubernetes offerings
The following instructions are for specific offerings:
  * [Install Operational Decision Manager on MiniKube](platform/README_Minikube.md)
  * [Install Operational Decision Manager on Openshift](platform/README_Openshift.md)

## Post-installation steps

### Step 1: Verify a deployment

You can check the status of the pods by using the following command:
```console
$ kubectl get pods
```

When all of the pods are *Running* and *Ready*, retrieve the <b>cluster-info-ip</b> name and <b>port</b> numbers with the following commands:

<pre>
$ kubectl cluster-info
Kubernetes master is running at https://<b>cluster-info-ip</b>:8443
CoreDNS is running at https://<b>cluster-info-ip</b>:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

$ kubectl get services
NAME                                                  TYPE        CLUSTER-IP  EXTERNAL-IP   PORT(S)                    AGE
kubernetes                                            ClusterIP   ****        none          443/TCP                    9m
my-odm-prod-release-dbserver                          ClusterIP   ****        none          5432/TCP                   3m
my-odm-prod-release-odm-decisioncenter                NodePort    ****        none          9453:<b>dcs-port</b>/TCP   3m
my-odm-prod-release-odm-decisionrunner                NodePort    ****        none          9443:<b>dr-port</b>/TCP    3m
my-odm-prod-release-odm-decisionserverconsole         NodePort    ****        none          9443:<b>dsc-port</b>/TCP   3m
my-odm-prod-release-odm-decisionserverconsole-notif   ClusterIP   ****        none          1883/TCP                   3m
my-odm-prod-release-odm-decisionserverruntime         NodePort    ****        none          9443:<b>dsr-port</b>/TCP   3m
</pre>

With the <b>cluster-info-ip</b> name and <b>port</b> numbers, you have access to the applications with the following URLs:

|Component|URL|Username|Password|
|:-----:|:-----:|:-----:|:-----:|
| Decision Server Console | https://*cluster-info-ip*:*dsc-port*/res |resAdmin|resAdmin|
| Decision Server Runtime |https://*cluster-info-ip*:*dsr-port*/DecisionService |N/A|N/A|
| Decision Center Business Console |  https://*cluster-info-ip*:*dcs-port*/decisioncenter |rtsAdmin|rtsAdmin|
| Decision Center Enterprise Console |  https://*cluster-info-ip*:*dcs-port*/teamserver |rtsAdmin|rtsAdmin|
| Decision Runner |  https://*cluster-info-ip*:*dr-port*/DecisionRunner |resDeployer|resDeployer|

To further debug and diagnose deployment problems in the Kubernetes cluster, use the `kubectl cluster-info dump` command.

For more information about how to check the state and recent events of your pods, see
[Troubleshooting](https://www.ibm.com/support/knowledgecenter/SSYHZ8_18.0.x/com.ibm.dba.install/k8s_topics/tsk_troubleshooting.html).

### Step 2: Synchronize users and groups

If you customized the default user registry, you must synchronize the registry with the Decision Center database. For more information, see
[Synchronizing users and groups in Decision Center](https://www.ibm.com/support/knowledgecenter/SSYHZ8_18.0.x/com.ibm.dba.install/k8s_topics/tsk_synchronize_users.html).

### Step 3: Manage your Operational Decision Manager deployment

It is possible to update a deployment after it is installed. Use the following tasks in IBM Knowledge Center to update a deployment whenever you need, and as many times as you need.
   * [Scaling deployments](https://www.ibm.com/support/knowledgecenter/SSYHZ8_18.0.x/com.ibm.dba.managing/k8s_topics/tsk_odm_scaling.html?view=kc)
   * [Customizing log levels](https://www.ibm.com/support/knowledgecenter/SSYHZ8_18.0.x/com.ibm.dba.managing/k8s_topics/tsk_odm_custom_logging.html?view=kc)
