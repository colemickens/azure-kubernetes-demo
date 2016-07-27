# Kubernetes 1.4 on Azure Demo

See [azure-kubernetes-status](https://github.com/colemickens/azure-kubernetes-status)
for a high-level status of Kubernetes (1.2.x, 1.3.x, and 1.4.x) on Azure.

## Overview

This is a preview of things to come. This will deploy a private build of Kubernetes until a new
1.4 alpha is tagged and released. This private build includes the Azure CloudProvider support.

Note that 1.4 is currently in **alpha** and will not be stable until 
September 2016.

At the end, you can go to 
[polykube's demo](https://github.com/colemickens/polykube/tree/master/DEMO.md) 
to see a full .NET application deployed as well.


## Preparation

Prerequisites:
  * `git`
  * `docker`
  * `make`


## Execution

1. **Clone `kubernetes-anywhere`**

    ```shell
    git clone https://github.com/kubernetes/kubernetes-anywhere
    cd kubernetes-anywhere
    ```

2. **Enter the `kubernetes-anywhere` docker environment**

    ```shell
    make docker-dev
    ```

3. **Inside the environment, perform the deployment**

  This will start the wizard/menu and then start the deployment:

  ```shell
  make deploy
  ```

  NOTE: To properly boot a cluster in Azure, you MUST set these values in the wizard:
  As mentioned earlier, this still uses my private `hyperkube` container until a new 1.4 alpha release is tagged.

  ```
  * phase2.docker_registry = "docker.io/colemickens"
  * phase2.kubernetes_version = "v1.4.0-alpha.1-azure"
  * phase2.installer_container = "docker.io/colemickens/install-k8s:v2"
  ```

  NOTE: You must also have a Azure Active Directory ServicePrincipal provisioned ahead of time.
  ([Hashicorp documentation can assist with this process.](https://www.packer.io/docs/builders/azure-setup.html))
  Currently, this must have 'Contributor' access to the resource group you're deploying to (the specified `deployment_name`).
  The details of the ServicePrincipal are needed for these options:
  ```
  * phase1.azure.tenant_id = "[account tenant_id]"
  * phase1.azure.subscription_id = "[account subscription_id]"
  * phase1.azure.client_id = "[serviceprincipal client_id]"
  * phase1.azure.client_secret = "[serviceprincipal client_secret]"
  ```


4. **Get the status of your nodes**

  You can access your cluster from your host machine whenever the cluster is 
  finished deploying. (This will take some time after the Terraform provisioning
  finishes.)

  The nodes may appear in the node list as `'NotReady'` for up to 60 seconds,
  after which they should become `'Ready'`.

  ```shell
  kubectl --kubeconfig="./phase1/azure/.tmp/kubeconfig" get nodes
  ```

  Example output:

  ```shell
  NAME                   STATUS                     AGE
  colemick-kube-master   Ready,SchedulingDisabled   2m
  colemick-kube-node-0   Ready                      2m
  colemick-kube-node-1   Ready                      2m
  colemick-kube-node-2   Ready                      2m
  colemick-kube-node-3   Ready                      2m
  colemick-kube-node-4   Ready                      2m
  ```

  Congratulations, you have a Kubernetes cluster!

5. **Deploy addons**

  Currently, `kubernetes-anywhere` does not deploy addons.
  You will likely want to deploy:
    * `kube-proxy` as a `DaemonSet`
    * `kube-dns`
    * `kubernetes-dashboard`

  The [`polykube` demo documentation](https://github.com/colemickens/polykube/tree/master/DEMO.md)
  walks though deploying these.
  Eventually this will be taken care of by `kubernetes-anywhere`.

5. **Deploy an Application**

  You're now set to use your cluster.

  If you need some inspiration, checkout 
  [the `polykube` demo](https://github.com/colemickens/polykube/tree/master/DEMO.md).
