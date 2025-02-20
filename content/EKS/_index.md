+++
title = "EKS Workshop"
type = "chapter"
weight = 3
chapter = true
+++

Welcome to the Elastic Kubernetes Service (EKS) with FSxN Workshop.

### Deploy your workshop environment

First, the environment must be deployed in your AWS Account.

Here is an overview of the steps that are carried out when the script is executed:
- Generate new SSH key pair for EC2 instances
- Create VPC, IGW and NAT GW
- Create FSxN Fileservice and an SVM for EKS
- Create a new EKS Cluster in the same VPC as FSxN
- Deploy Trident to EKS Cluster

**Note** The Deployment takes around 60 Minutes!

```bash
./generate-config.sh -s create -w <workshop> -v true -r <aws region>
```


