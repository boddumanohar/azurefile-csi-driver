## NFS support
[NFS 4.1 support for Azure Files](https://azure.microsoft.com/en-us/blog/nfs-41-support-for-azure-files-is-now-in-preview/preview/) is now in Public Preview. This service is optimized for random access workloads with in-place data updates and provides full POSIX file system support. This page shows how to use NFS feature by Azure File CSI driver on Azure Kubernetes cluster.

#### Feature Status: Beta
> supported OS: Linux

#### Supported CSI driver version: `v0.9.0`
> storage account dynamic provisioning is supported from `v0.10.0`.

#### [Available regions](https://aka.ms/azurefiles/nfs/preview/regions)
We are continually adding more regions. Latest region list is available on [Azure NFS documentation](https://aka.ms/azurefiles/nfs/preview/regions)

#### Prerequisite
 - [install CSI driver](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/docs/install-csi-driver-master.md)
 - Register `AllowNfsFileShares` feature under your subscription
```console
az feature register --name AllowNfsFileShares --namespace Microsoft.Storage
az feature list -o table --query "[?contains(name, 'Microsoft.Storage/AllowNfsFileShares')].{Name:name,State:properties.state}"
az provider register --namespace Microsoft.Storage
```
 - [Optional] Create a `Premium_LRS` Azure storage account with following configurations to support NFS share
   - account kind: `FileStorage`
   - secure transfer required(enable HTTPS traffic only): `false`
   - select virtual network of agent nodes in `Firewalls and virtual networks`
   - specify `storageAccount` in below storage class `parameters`
 - [Optional] If cluster identity is Managed Service Identity(MSI), make sure user assigned identity has Contributor role on node resource group

#### How to use NFS feature
 - Create an Azure File storage class
> specify `protocol: nfs` in storage class `parameters`
> </br>for more details, refer to [driver parameters](../../../docs/driver-parameters.md)
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azurefile-csi-nfs
provisioner: file.csi.azure.com
parameters:
  protocol: nfs
```

run following commands to create a storage class:
```console
wget https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/storageclass-azurefile-nfs.yaml
# set `storageAccount` in storageclass-azurefile-nfs.yaml
kubectl create -f storageclass-azurefile-nfs.yaml
```

### Example#1
 - Create a deployment with NFS volume
```console
kubectl create -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/nfs/statefulset.yaml
```

 - enter pod to check
```console
$ kubectl exec -it statefulset-azurefile-0 sh
# df -h
Filesystem      Size  Used Avail Use% Mounted on
...
/dev/sda1                                                                                 29G   11G   19G  37% /etc/hosts
accountname.file.core.windows.net:/accountname/pvc-fa72ec43-ae64-42e4-a8a2-556606f5da38  100G     0  100G   0% /mnt/azurefile
...
```

### Example#2
 - Create a [Wordpress](https://github.com/bitnami/charts/tree/master/bitnami/wordpress) application with NFS volume
```console
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install --set persistence.storageClass="azurefile-csi-nfs" --set persistence.size=100Gi --generate-name bitnami/wordpress
```

#### Links
 - [Troubleshoot Azure NFS file shares](https://docs.microsoft.com/en-us/azure/storage/files/storage-troubleshooting-files-nfs)
