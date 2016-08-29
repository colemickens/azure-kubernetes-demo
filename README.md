# Kubernetes 1.4 on Azure - Demo/Testing

## General Status of Kubernetes on Azure
See [azure-kubernetes-status](https://github.com/colemickens/azure-kubernetes-status)
for a high-level status of Kubernetes (1.3.x, and 1.4.x) on Azure.

## Demo Overview
This demonstrates:
 * Native Azure Cloudprovider support (automatic pod networking, automatic L4 load balancers)
 * AzureDisk persistent storage plugin

## Demo Features
The [yaml file that will be deployed](./test-azure-disk.yaml) demonstrates some core Kubernetes functionality:
* Service object that abstract over Pods meeting a certain selector.
* Service object that declares a need for an external Azure load balancer.
* Deployment objects that contain a Pod template and ensure a certain replica count are available.
* Pod object (inside the Deployment) that shows:
  * Two containers, sharing a volume mount
  * The volume mount being backed by an Azure Disk
  * One busybox container appending a timestamp to a file on the volume mount
  * One busybox container serving the file on the volume mount over HTTP on port 80

## Requirements

* `kubectl` (v1.4.0-alpha.3 or newer) [(linux/amd64)](https://storage.googleapis.com/kubernetes-release/release/v1.4.0-alpha.3/bin/linux/amd64/kubectl) [(darwin/amd64)](https://storage.googleapis.com/kubernetes-release/release/v1.4.0-alpha.3/bin/darwin/amd64/kubectl)

## Deploy a Cluster

Follow the instructions in the `kubernetes-anywhere` project to deploy a cluster: https://github.com/kubernetes/kubernetes-anywhere/tree/master/phase1/azure/README.md.

## Create a VHD Disk

1. Use the `azure-tools` container to create an ext4-formatted VHD in an Azure storage account:

    (The script will create everything for you, if missing. It will default to `westus2`.)

    ```bash
    docker pull docker.io/colemickens/azure-tools:latest
    docker run -it docker.io/colemickens/azure-tools:latest
    
    # (inside the container)
    
    export AZURE_SUBSCRIPTION_ID=6f368760-9ad2-4aef-8ff1-fb038d2e75bf
    export AZURE_RESOURCE_GROUP=colemick-vhds2
    export AZURE_STORAGE_ACCOUNT=colemickvhds2
    export AZURE_STORAGE_CONTAINER=colemickvhds2
    export IMAGE_SIZE=10G
    
    ./make-vhd.sh
    # ....
    # ....
    # VHD_URL=https://colemickvhds2.blob.core.windows.net/colemickvhds2/data-disk-082916103645.vhd
    # (end)
    ```
    
    You can leave the `azure-tools` container now.

## Prove the VHD support works correctly

1. Download [`test-azure-vhd.yaml`](./test-azure-disk.yaml) from this repo:

  ```shell
  wget https://raw.githubusercontent.com/colemickens/azure-kubernetes-demo/pr-azure-demo2.0/test-azure-disk.yaml
  ```
  
2. Edit in the VHD_URL:
   ```shell
   export VHD_URL=https://colemickvhds2.blob.core.windows.net/colemickvhds2/data-disk-082916103645.vhd
   sed -i "s|VHD_URL|${VHD_URL}|g" ./test-azure-disk.yaml
   ```
   (or you can simply do this by hand...)

2. Deploy the `test-azure-disk` deployment/service:
   ```shell
   kubectl apply -f test-azure-disk.yaml
   ```

3. Wait for the pod to start running, and wait for the service to get an external load balancer IP address.
   It will take some time for the Pod to begin running as the VHD must be attached and mounted first.
   ```shell
   kubectl get pods
   kubectl get services
   ```

4. After the pod has become healthy, query the service to see that it has started putting some data on the persistent disk.
  ```shell
  kubectl get service test-azure-disk
  ```
   
  Connect to the `external ip` using a browser and check that you see entries.

5. Kill the pod!
  ```shell
  kubectl get pods
  kubectl delete pod test-azure-disk-XXXXX
  ```

6. Kubernetes will schedule a new Pod, potentially on a new VM. This will take some time too as the disk
   must be unmounted and unattached from the old node, then attached and mounted to the new node.

  Wait for it be to healthy again:
  ```shell
  kubectl get pods
  ```

7. Check the `external ip` again in your browser and see that the old entries were indeed persisted.

## Troubleshooting

1. Make sure the VHD you're using is in a storage account in the same region as the VM.
2. Check `kubectl logs --namespace=kube-system kube-controller-manger-<TAB> | grep --text azure` to see what important actions took place on the master to create the LB or attach/detach the disk.
3. Check `kubectl describe pod test-azure-disk-<TAB>` to see the status of the volume mount and the containers in the Pod.

## Bonus

1. You can query the cluster nodes  with `kubectl get nodes` and then use `kubectl cordon {NODE}` before killing
   the Pod to guarantee that it is not rescheduled onto the same node. (Don't forget to `kubectl uncordon {NODE}` later!)

## Send Feedback

Please feel file issues in the `kubernetes-anywhere` or `kubernetes` projects as relevant.
