# Install Operational Decision Manager with the Helm CLI

Refer to the top repository [readme](../../README.md#step-2-download-a-product-package-from-ppa-and-load-the-images) to find instructions on how to push and tag the product container images to your Docker registry.

1. If Helm is not installed in your Kubernetes cluster, install [Helm 2.9.1](https:/github.com/helm/helm/releases/tag/v2.9.1).
2. When Helm is ready, initialize the local CLI and install Tiller.

   ```console
   $ helm init
   ```
   Tiller is now installed in the Kubernetes cluster with the current-context configuration. By default, Tiller does not have authentication enabled. For more inforamtion about configuring strong TLS authentication, see the [Tiller TLS guide](https://helm.sh/docs/using_helm/#using-ssl-between-helm-and-tiller).

3. Download the Helm chart from the GitHub repository [ibm-odm-prod-2.1.0.tgz](ibm-odm-prod-2.1.0.tgz).

4. Install a Kubernetes release with the default configuration and a name of `my-odm-prod-release` by using the following command:

   ```console
   $ helm install --name my-odm-prod-release \
     /path/to/ibm-odm-prod-2.1.0.tgz
   ```
   The package is deployed asynchronously in a matter of minutes, and is composed of several services.

   > **Note**: You can check the status of the pods that have been created:
   ```console
   $ kubectl get pods
   NAME                                                READY   STATUS    RESTARTS   AGE
   my-odm-prod-release-dbserver-***                    1/1     Running   0          44m
   my-odm-prod-release-odm-decisioncenter-***          1/1     Running   0          44m
   my-odm-prod-release-odm-decisionrunner-***          1/1     Running   0          44m
   my-odm-prod-release-odm-decisionserverconsole-***   1/1     Running   0          44m
   my-odm-prod-release-odm-decisionserverruntime-***   1/1     Running   0          44m
   ```

5. List the helm releases in your cluster.

   ```console
   $ helm ls
   ```
   The release is an instance of the `ibm-odm-prod` chart. All the Operational Decision Manager components are now running in a Kubernetes cluster.

   To verify a deployment, go back to the [Post installation steps](../README.md#post-installation-steps).

## Customize a Kubernetes release of Operational Decision Manager

Refer to the [ODM for production Certified Kubernetes parameters](https://www.ibm.com/support/knowledgecenter/SSYHZ8_18.0.x/com.ibm.dba.ref/k8s_topics/ref_parameters_prod.html) for a complete list of values that you can configure.

### To customize the helm install with --set key=value arguments

Using the `helm install` command, you can specify each parameter with a `--set key=value` argument. For example, the following command sets 3 parameters for the internal database.

```console
$ helm install --name my-odm-prod-release \
  --set internalDatabase.databaseName=my-db \
  --set internalDatabase.user=my-user \
  --set internalDatabase.password=my-password \
  /path/to/ibm-odm-prod-2.1.0.tgz
```

### To customize the helm install with a YAML file

You can use a custom-made .yaml file to specify the values of the parameters when you install the chart. For example, the following command uses the `myvalues.yaml` file.

```console
$ helm install --name my-odm-prod-release -f myvalues.yaml /path/to/ibm-odm-prod-2.1.0.tgz
```

> **Tip**: Refer to the [`sample-values.yaml`](../configuration/sample-values.yaml) file to find the default values used by the [`ibm-odm-prod-2.1.0`](helm-charts/ibm-odm-prod-2.1.0.tgz) chart.

## Uninstall a Kubernetes release of Operational Decision Manager

To uninstall and delete a release named `my-odm-prod-release`, use the following command:

```console
$ helm delete my-odm-prod-release --purge
```

The command removes all the Kubernetes components associated with the release, except any Persistent Volume Claims (PVCs).  This is the default behavior of Kubernetes, and ensures that valuable data is not deleted. To delete the persisted data of the release, you can delete the PVC using the following command:

```console
$ kubectl delete pvc my-odm-prod-release-odm-pvclaim
```
