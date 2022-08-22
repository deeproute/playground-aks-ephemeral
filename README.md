
## How Ephemeral Disks works in AKS
https://docs.microsoft.com/en-us/azure/aks/cluster-configuration#ephemeral-os

## How to enable Ephemeral Disks in Terraform:

https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/kubernetes_cluster#os_disk_type

> Note: The maximum `OS Disk Size` for an ephemeral disk is the VM's cached disk size. If its higher then that then the deploy will fail.

## With VM Size Standard_D4d_v5 supports Ephemeral Disks:
```sh
az aks nodepool list -g playground-aks-ephemeral --cluster-name aks-ephemeral | grep -i disk
    "kubeletDiskType": "OS",
    "osDiskSizeGb": 128,
    "osDiskType": "Ephemeral",
```
## Standard_E4_v3 doesn't support Ephemeral Disks (from the VM docs it seems none "Memory Optimized" VMs support ephemeral):
```sh
Error: creating Cluster: (Managed Cluster Name "aks-ephemeral" / Resource Group "playground-aks-ephemeral"): containerservice.ManagedClustersClient#CreateOrUpdate: Failure sending request: StatusCode=400 -- Original Error: Code="VMSizeDoesNotSupportEphemeralOS" Message="The Virtual Machine size Standard_E4_v3 does not support Ephemeral OS disk."
```

## Standard_F8s_v2 supports Ephemeral Disks:
```sh
az aks nodepool list -g playground-aks-ephemeral --cluster-name aks-ephemeral | grep -i disk
    "kubeletDiskType": "OS",
    "osDiskSizeGb": 128,
    "osDiskType": "Ephemeral",

az aks get-credentials --admin -g playground-aks-ephemeral -n aks-ephemeral --overwrite-existing
```

## Another type of error that can happen is with too high OS Disk Size for the VM Size:
```
Error: creating Cluster: (Managed Cluster Name "aks-ephemeral" / Resource Group "playground-aks-ephemeral"): containerservice.ManagedClustersClient#CreateOrUpdate: Failure sending request: StatusCode=400 -- Original Error: Code="VMCannotFitEphemeralOSDisk" Message="The virtual machine size Standard_D8ds_v4 has a cache size of 214748364800 bytes and temporary disk size of 322122547200 bytes, but the OS disk requires 332859965440 bytes. Use a VM size with larger cache, larger temp disk, or disable ephemeral OS."
```

To check the maximum os_disk_size that ephemeral supports then check the "Temp Storage (SSD) GiB" column in the VM docs for the VM Size:
https://docs.microsoft.com/en-us/azure/virtual-machines/ddv5-ddsv5-series