+++
title = "modifications"
date = 2020-04-01T13:00:00+02:00
weight = 1
pre = "<b></b>"
+++

##  kubeone on NetApp HCI
### Prepare kubeone environment:

first set variables
```
export VSPHERE_ALLOW_UNVERIFIED_SSL="true"
export VSPHERE_PASSWORD="passw0rd"
export VSPHERE_SERVER="hci-vcenter"
export VSPHERE_USER="administrator@vsphere.local"
```

##### enable ssh agend
this is the easiest way to execute ssh commeands remotely.
```bash
eval `ssh-agent`
ssh-add
```

##### Changes in output.tf

```
fabian@cni-jump-01:~/k8stools/kubeone/examples/terraform/vsphere$ cat outputs.tf 
/*
Copyright 2019 The KubeOne Authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

output "kubeone_api" {
  description = "kube-apiserver LB endpoint"

  value = {
    endpoint = vsphere_virtual_machine.lb[0].default_ip_address
  }
}

output "kubeone_hosts" {
  description = "Control plane endpoints to SSH to"

  value = {
    control_plane = {
      cluster_name         = var.cluster_name
      cloud_provider       = "vsphere"
      private_address      = []
      public_address       = vsphere_virtual_machine.control_plane.*.default_ip_address
      ssh_agent_socket     = var.ssh_agent_socket
      ssh_port             = var.ssh_port
      ssh_private_key_file = var.ssh_private_key_file
      ssh_user             = var.ssh_username
    }
  }
}

output "kubeone_workers" {
  description = "Workers definitions, that will be transformed into MachineDeployment object"

  value = {
    # following outputs will be parsed by kubeone and automatically merged into
    # corresponding (by name) worker definition
    "${var.cluster_name}-pool1" = {
      replicas = 1
      providerSpec = {
        sshPublicKeys   = [file(var.ssh_public_key_file)]
        operatingSystem = var.worker_os
        operatingSystemSpec = {
          distUpgradeOnBoot = false
        }
        cloudProviderSpec = {
          # provider specific fields:
          # see example under `cloudProviderSpec` section at:
          # https://github.com/kubermatic/machine-controller/blob/master/examples/vsphere-machinedeployment.yaml
          
          cluster       = var.compute_cluster_name
          cpus          = 2
          datacenter    = var.dc_name
          # Either Datastore or DatastoreCluster have to be provided.
          datastore        = var.datastore_name
          datastoreCluster = var.datastore_cluster_name
          # Optional: Resize the root disk to this size. Must be bigger than the existing size
          # Default is to leave the disk at the same size as the template
          diskSizeGB     = 10
          memoryMB       = 2048
          templateVMName = var.template_name
          vmNetName      = var.network_name
          # Red Hat subscription manager offline token (only to be used for RHEL)
          # rhsmOfflineToken = ""
          # modification for NETAPP HCI
          allowInsecure = true
          folder = var.vmfolder
        }
      }
    }
  }
}
```

##### change the terraform vars (terraform.tfvars)

```
cluster_name   = "kubeoneonhci"
datastore_name = "NetApp-HCI-Datastore-02"
network_name   = "NetApp HCI VDS 01-VM_Network"
template_name  = "ubuntu-image"
ssh_username   = "ubuntu"
dc_name = "NetApp-HCI-Datacenter-01"
compute_cluster_name = "NetApp-HCI-Cluster-01"
ssh_public_key_file = "/home/<username>/.ssh/id_rsa.pub"
ssh_private_key_file = "/home/<username>/.ssh/id_rsa"
```

##### configuration file for kubernetes (config.yaml)
```yaml
apiVersion: kubeone.io/v1alpha1
kind: KubeOneCluster
versions:
  kubernetes: '1.17.4'
cloudProvider:
  name: 'vsphere'
  cloudConfig: |
    [Global]
    secret-name = "cloud-provider-credentials"
    secret-namespace = "kube-system"
    port = "443"
    insecure-flag = "1"

    [VirtualCenter "hci-vcenter"]
    datacenters = "NetApp-HCI-Datacenter-01"

    [Workspace]
    server = "hci-vcenter"
    datacenter = "NetApp-HCI-Datacenter-01"
    default-datastore = "NetApp-HCI-Datastore-02"
    resourcepool-path = "Resources"
    folder = "k8sk1"

    [Disk]
    scsicontrollertype = pvscsi

    [Network]
    public-network = "NetApp HCI VDS 01-VM_Network"
```
