# Install IBM Operational Decision Manager on Red Hat OpenShift

## Step 1: Install the OpenShift command line interface (CLI) and Helm

1. The OpenShift Container Platform CLI exposes commands for managing your applications, as well as lower level tools to interact with each component of your system. Refer to the OpenShift [documentation](https://docs.openshift.com/container-platform/3.11/cli_reference/get_started_cli.html).

2. Install [Helm 2.11.0](https://github.com/helm/helm/releases/tag/v2.11.0).

## Step 2: Push and tag the downloaded images in the OpenShift registry

1. Follow the instructions to download the IBM Operational Decision Manager images and the loadimages.sh file in [Download PPA and load images](../../README.md#step-2-download-a-product-package-from-ppa-and-load-the-images).

   > **Note**: **DO NOT** run the loadimages.sh script at this point.

2. Login to the cluster with an administrator user.
   ```console
   $ oc login https://CLUSTERIP:8443 --token=<hidden>
   ```

3. Use the following command to run the loadimages.sh script file.
   ```console
   $ docker login docker-registry.default.svc:5000 -u ocadmin -p $(oc whoami -t)
   $ ./loadimages.sh -p <PPA-ARCHIVE>.tar.gz -r docker-registry.default.svc:5000/odm
   ```

## Step 3: Install a Kubernetes release of Operational Decision Manager

1. Create a new project.
   ```console
    $ oc login <ODMUSER>
    $ oc new-project odmapp
    $ oc project odmapp
    $ oc adm policy add-scc-to-user privileged -z default -nodmapp
   ```
   > **Note**: The project must have pull request privileges to the registry where the Operational Decision Manager images are loaded.  

2. Download the [ibm-odm-prod-2.1.0.tgz](../helm-charts/ibm-odm-prod-2.1.0.tgz) file. The archive contains the `ODM for production (ibm-odm-prod)` Helm chart.

3. Install a release with the default configuration and a name of `my-odm-prod-release`.

You have 2 options to install Operation Decision Manager on Openshift depending on the your security policy.

   * Option 1 : Use the following command from the Kubernetes CLI:

     ```console
     $ helm template \
       --name my-odm-prod-release \
     /path/to/ibm-odm-prod-2.1.0.tgz --set image.repository=docker-registry.default.svc:5000/odm/ > odm-k8s.yaml
     $ oc create --save-config=true -f odm-k8s.yaml
     ```

     > **Note**: For more information about installing with the Kubernetes CLI, see [k8s-yaml/README.md](../k8s-yaml/README.md).

   * Option 2 : Use the following command with Helm Tiller:

     ```console
     $ helm install \
       --name my-odm-prod-release \
     /path/to/ibm-odm-prod-2.1.0.tgz --set image.repository=docker-registry.default.svc:5000/odm/
     ```

     > **Note**: For more information about installing with Helm Tiller, see [helm-charts/README.md](../helm-charts/README.md).

4. The package is deployed asynchronously in a matter of minutes, and is composed of several services.

   > **Note**: You can check the status of the pods that you created:
   ```console
   $ kubectl get pods
   NAME                                                READY   STATUS    RESTARTS   AGE
   my-odm-prod-release-dbserver-***                    1/1     Running   0          44m
   my-odm-prod-release-odm-decisioncenter-***          1/1     Running   0          44m
   my-odm-prod-release-odm-decisionrunner-***          1/1     Running   0          44m
   my-odm-prod-release-odm-decisionserverconsole-***   1/1     Running   0          44m
   my-odm-prod-release-odm-decisionserverruntime-***   1/1     Running   0          44m
   ```

   The release is an instance of the `ibm-odm-prod` chart. All of the components are now running in a Kubernetes cluster.

## Step 4: Verify that the deployment is running

When all of the pods are *Running*, you can access the status of your application with the following command.
```console
$ oc status
In project odm on server https://localhost:8443

svc/odm-release-dbserver - xxx.xx.xx.xx:5432
  deployment/odm-release-dbserver deploys docker-registry.default.svc:5000/odm/dbserver:8.10.1-amd64
    deployment #1 running for 27 minutes - 1 pod

svc/odm-release-odm-decisioncenter (all nodes):31070 -> 9453
  deployment/odm-release-odm-decisioncenter deploys docker-registry.default.svc:5000/odm/odm-decisioncenter:8.10.1-amd64
    deployment #1 running for 27 minutes - 1 pod

svc/odm-release-odm-decisionrunner (all nodes):31705 -> 9443
  deployment/odm-release-odm-decisionrunner deploys docker-registry.default.svc:5000/odm/odm-decisionrunner:8.10.1-amd64
    deployment #1 running for 27 minutes - 1 pod

svc/odm-release-odm-decisionserverconsole-notif - xxx.xx.xx:1883
http://odm-release-odm-decisionserverconsole-odm.xxx.xx.xx.nip.io to pod port decisionserverconsole-https (svc/odm-release-odm-decisionserverconsole)
  deployment/odm-release-odm-decisionserverconsole deploys docker-registry.default.svc:5000/odm/odm-decisionserverconsole:8.10.1-amd64
    deployment #1 running for 27 minutes - 1 pod

http://myserver to pod port decisionserverruntime-https (svc/odm-release-odm-decisionserverruntime)
  deployment/odm-release-odm-decisionserverruntime deploys docker-registry.default.svc:5000/odm/odm-decisionserverruntime:8.10.1-amd64
    deployment #1 running for 27 minutes - 1 pod

1 info identified, use 'oc status --suggest' to see details.
```

You can now expose the service to your users.

> **Tip**: Refer to [Verify a deployment](../README.md#step-1-verify-a-deployment) post installation step to get the URLs of the services.

## To customize a release

Refer to the customizing instructions in [k8s-yaml/README.md](../k8s-yaml/README.md#customize-a-kubernetes-release-of-operational-decision-manager).

## To uninstall the Helm chart

   * To uninstall and delete a release named `my-odm-prod-release` from the Kubernetes CLI, use the following command:

     ```console
     $ oc delete -f odm-k8s.yaml
     ```

     The `odm-k8s.yaml` is the file you created in step 3: [Install an Operational Decision Manager release](README_Openshift.md#step-3-install-a-kubernetes-release-of-operational-decision-manager).

  * To uninstall and delete a release named `my-odm-prod-release` with Helm Tiller, use the following command:

     ```console
     $ helm delete my-odm-prod-release --purge
     ```

     The command removes all the Kubernetes components associated with the chart, except Persistent Volume Claims (PVCs). This is the default behavior of Kubernetes, and ensures that valuable data is not deleted. To delete the data, you can delete the PVC by using the following command:

     ```console
     $ kubectl delete pvc <release_name>-odm-pvclaim -n <namespace>
     ```
