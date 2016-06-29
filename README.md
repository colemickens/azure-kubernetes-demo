# Kubernetes 1.4 on Azure Demo

See [azure-kubernetes-status](https://github.com/colemickens/azure-kubernetes-status)
for a high-level status of Kubernetes on Azure (including stable releases
1.2.x and 1.3.x).

## Overview

This is a preview of things to come. This will use private forks of
[`kubernetes`](https://github.com/colemickens/kubernetes/tree/azure-cloudprovider) 
and [`kubernetes-anywhere`](https://github.com/colemickens/kubernetes-anywhere/tree/azure) 
to deploy a Kubernetes 1.4 cluster into
Azure. This will have CloudProvider support.

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
    make env
    ```

3. **Inside the environment, perform the deployment**

  Note: to perform the deployment, you must do the following things.
  This will use [a private fork of `kubernetes`](https://github.com/colemickens/kubernetes/tree/azure-cloudprovider) 
  until it is merged in and the normal repository can be used.
  ```
  * phase2.docker_registry = "docker.io/colemickens"
  * phase2.kubernetes_version = "v1.4.0-alpha.0-azure"
  * phase2.installer_container = "docker.io/colemickens/install-k8s:v1"
  ```

  Note: you must also have a ServicePrincipal provisioned.
  Currently this must have 'Contributor' access to the whole cluster.
  Eventually there will be a helper to create a resource-group-scoped 
  ServicePrincipal. The properties of the ServicePrincipal are needed for
  these options:
  ```
  * phase1.azure.tenant_id = "[account tenant_id]"
  * phase1.azure.subscription_id = "[account subscription_id]"
  * phase1.azure.client_id = "[serviceprincipal client_id]"
  * phase1.azure.client_secret = "[serviceprincipal client_secret]"
  ```

  If you're ready...

    ```shell
    make deploy
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

5. **Deploy an Application**

  You're now set to use your cluster.

  If you need some inspiration, checkout 
  [the polykube demo](https://github.com/colemickens/polykube/tree/master/DEMO.md).
