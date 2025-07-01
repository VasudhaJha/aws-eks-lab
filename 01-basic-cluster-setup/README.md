# Project 01: Basic EKS Cluster Setup

**Estimated Time**: 45-60 minutes

## Learning Objectives

By completing this lab, you will:

- Create your first EKS cluster using eksctl
- Understand EKS cluster architecture and components
- Configure kubectl to interact with your cluster
- Deploy and manage worker nodes with managed node groups
- Explore basic cluster operations and troubleshooting
- Understand the AWS resources that make up an EKS cluster

## Prerequisites

- AWS CLI installed and configured with appropriate permissions
- `eksctl` installed (v0.150.0 or later)
- `kubectl` installed
- Basic understanding of Kubernetes concepts (pods, deployments, services)
- AWS account with EKS, EC2, and VPC permissions

## Architecture Overview

┌─────────────────────────────────────────────────────────────┐
│                        AWS Account                          │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                     VPC (Auto-created)                  ││
│  │                                                         ││
│  │  ┌──────────────────┐    ┌──────────────────────────────┐││
│  │  │   Public Subnet  │    │      Private Subnet          │││
│  │  │                  │    │                              │││
│  │  │  ┌─────────────┐ │    │  ┌─────────────────────────┐ │││
│  │  │  │     NAT     │ │    │  │                         │ │││
│  │  │  │   Gateway   │ │    │  │     EKS Worker Nodes    │ │││
│  │  │  └─────────────┘ │    │  │    (Managed Node Group) │ │││
│  │  └──────────────────┘    │  │                         │ │││
│  │                          │  └─────────────────────────┘ │││
│  └─────────────────────────────────────────────────────────┘││
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                  EKS Control Plane                      ││
│  │              (AWS Managed Service)                      ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘

**Key Components:**

- EKS Control Plane: AWS-managed Kubernetes API server, etcd, scheduler
- Managed Node Group: Auto-scaling group of EC2 instances running kubelet
- VPC: Isolated network with public/private subnets across multiple AZs
- Security Groups: Firewall rules for cluster and node communication

## Step-by-Step Instructions

### Step 1: Create the Basic Cluster

Create a simple cluster with eksctl:

```bash
eksctl create cluster \
  --name my-first-eks-cluster \
  --region <chosen aws region> \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed
```

**What this command does:**

- Creates EKS cluster `control plane`
- Sets up VPC with public/private subnets across all `AZs`
- Creates managed node group with 2 `t3.medium` instances
- Configures `kubectl context` automatically
- Sets up `IAM roles` and `security groups`

⏱️ Wait time: 15-20 minutes (perfect for coffee!)

You'll see something like this in your terminal:

```bash
2025-06-26 15:32:15 [ℹ]  eksctl version 0.208.0
2025-06-26 15:32:15 [ℹ]  using region ap-south-1
2025-06-26 15:32:15 [!]  Amazon EKS will no longer publish EKS-optimized Amazon Linux 2 (AL2) AMIs after November 26th, 2025. Additionally, Kubernetes version 1.32 is the last version for which Amazon EKS will release AL2 AMIs. From version 1.33 onwards, Amazon EKS will continue to release AL2023 and Bottlerocket based AMIs. The default AMI family when creating clusters and nodegroups in Eksctl will be changed to AL2023 in the future.
2025-06-26 15:32:15 [ℹ]  setting availability zones to [ap-south-1a ap-south-1b ap-south-1c]
2025-06-26 15:32:15 [ℹ]  subnets for ap-south-1a - public:192.168.0.0/19 private:192.168.96.0/19
2025-06-26 15:32:15 [ℹ]  subnets for ap-south-1b - public:192.168.32.0/19 private:192.168.128.0/19
2025-06-26 15:32:15 [ℹ]  subnets for ap-south-1c - public:192.168.64.0/19 private:192.168.160.0/19
2025-06-26 15:32:15 [ℹ]  nodegroup "workers" will use "" [AmazonLinux2/1.32]
2025-06-26 15:32:15 [ℹ]  using Kubernetes version 1.32
2025-06-26 15:32:15 [ℹ]  creating EKS cluster "my-first-eks-cluster" in "ap-south-1" region with managed nodes
2025-06-26 15:32:15 [ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial managed nodegroup
2025-06-26 15:32:15 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=ap-south-1 --cluster=my-first-eks-cluster'
2025-06-26 15:32:15 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "my-first-eks-cluster" in "ap-south-1"
2025-06-26 15:32:15 [ℹ]  CloudWatch logging will not be enabled for cluster "my-first-eks-cluster" in "ap-south-1"
2025-06-26 15:32:15 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=ap-south-1 --cluster=my-first-eks-cluster'
2025-06-26 15:32:15 [ℹ]  default addons kube-proxy, coredns, metrics-server, vpc-cni were not specified, will install them as EKS addons
2025-06-26 15:32:15 [ℹ]
2 sequential tasks: { create cluster control plane "my-first-eks-cluster",
    2 sequential sub-tasks: {
        2 sequential sub-tasks: {
            1 task: { create addons },
            wait for control plane to become ready,
        },
        create managed nodegroup "workers",
    }
}
2025-06-26 15:32:15 [ℹ]  building cluster stack "eksctl-my-first-eks-cluster-cluster"
2025-06-26 15:32:15 [ℹ]  deploying stack "eksctl-my-first-eks-cluster-cluster"
2025-06-26 15:32:45 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-cluster"
2025-06-26 15:33:16 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-cluster"
2025-06-26 15:34:16 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-cluster"
2025-06-26 15:35:16 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-cluster"
2025-06-26 15:36:16 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-cluster"
2025-06-26 15:37:16 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-cluster"
2025-06-26 15:38:17 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-cluster"
2025-06-26 15:39:17 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-cluster"
2025-06-26 15:40:17 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-cluster"
2025-06-26 15:40:18 [ℹ]  creating addon: kube-proxy
2025-06-26 15:40:19 [ℹ]  successfully created addon: kube-proxy
2025-06-26 15:40:19 [ℹ]  creating addon: coredns
2025-06-26 15:40:19 [ℹ]  successfully created addon: coredns
2025-06-26 15:40:20 [ℹ]  creating addon: metrics-server
2025-06-26 15:40:20 [ℹ]  successfully created addon: metrics-server
2025-06-26 15:40:21 [!]  recommended policies were found for "vpc-cni" addon, but since OIDC is disabled on the cluster, eksctl cannot configure the requested permissions; the recommended way to provide IAM permissions for "vpc-cni" addon is via pod identity associations; after addon creation is completed, add all recommended policies to the config file, under `addon.PodIdentityAssociations`, and run `eksctl update addon`
2025-06-26 15:40:21 [ℹ]  creating addon: vpc-cni
2025-06-26 15:40:21 [ℹ]  successfully created addon: vpc-cni
2025-06-26 15:42:22 [ℹ]  building managed nodegroup stack "eksctl-my-first-eks-cluster-nodegroup-workers"
2025-06-26 15:42:22 [ℹ]  deploying stack "eksctl-my-first-eks-cluster-nodegroup-workers"
2025-06-26 15:42:22 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-nodegroup-workers"
2025-06-26 15:42:53 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-nodegroup-workers"
2025-06-26 15:43:26 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-nodegroup-workers"
2025-06-26 15:43:57 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-nodegroup-workers"
2025-06-26 15:45:10 [ℹ]  waiting for CloudFormation stack "eksctl-my-first-eks-cluster-nodegroup-workers"
2025-06-26 15:45:10 [ℹ]  waiting for the control plane to become ready
2025-06-26 15:45:11 [✔]  saved kubeconfig as "/Users/vasudhajha/.kube/config"
2025-06-26 15:45:11 [ℹ]  no tasks
2025-06-26 15:45:11 [✔]  all EKS cluster resources for "my-first-eks-cluster" have been created
2025-06-26 15:45:11 [ℹ]  nodegroup "workers" has 2 node(s)
2025-06-26 15:45:11 [ℹ]  node "ip-192-168-4-159.ap-south-1.compute.internal" is ready
2025-06-26 15:45:11 [ℹ]  node "ip-192-168-72-44.ap-south-1.compute.internal" is ready
2025-06-26 15:45:11 [ℹ]  waiting for at least 1 node(s) to become ready in "workers"
2025-06-26 15:45:11 [ℹ]  nodegroup "workers" has 2 node(s)
2025-06-26 15:45:11 [ℹ]  node "ip-192-168-4-159.ap-south-1.compute.internal" is ready
2025-06-26 15:45:11 [ℹ]  node "ip-192-168-72-44.ap-south-1.compute.internal" is ready
2025-06-26 15:45:11 [✔]  created 1 managed nodegroup(s) in cluster "my-first-eks-cluster"
2025-06-26 15:45:12 [ℹ]  kubectl command should work with "/Users/vasudhajha/.kube/config", try 'kubectl get nodes'
2025-06-26 15:45:12 [✔]  EKS cluster "my-first-eks-cluster" in "ap-south-1" region is ready
```

### Step 2: Verify Cluster Setup

```bash
# Check cluster status
eksctl get cluster

# Verify kubectl configuration
kubectl config current-context

# Check nodes are ready
kubectl get nodes

# View cluster information
kubectl cluster-info
```

### Step 3: Explore System Components

```bash
# View all system pods
kubectl get pods --all-namespaces

# Check specific system components
kubectl get pods -n kube-system

# Look at AWS-specific components
kubectl get pods -n kube-system | grep aws
```

### Step 4: Deploy a Test Application

```bash
# Create a simple deployment
kubectl create deployment nginx --image=nginx:1.21

# Check deployment status
kubectl get deployments
kubectl get pods

# View deployment details
kubectl describe deployment nginx
```

### Step 5: Expose and Test the Application

```bash
# Create a LoadBalancer service
kubectl expose deployment nginx --type=LoadBalancer --port=80

# Wait for external IP (takes 2-3 minutes)
kubectl get service nginx --watch

# Test the application
curl $(kubectl get service nginx -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
```

## Key Concepts Explained

### EKS Control Plane

- Managed by AWS: You don't manage master nodes
- High Availability: Runs across multiple AZs automatically
- Automatic Updates: AWS handles Kubernetes version updates
- API Server: Your kubectl commands go here

### Managed Node Groups

- Auto Scaling: Nodes scale based on pod requirements
- Managed Updates: AWS can update node AMIs automatically
- Instance Types: Choose based on workload requirements
- Spot Integration: Can use spot instances for cost savings

### Networking

- VPC: Isolated network for your cluster
- Subnets: Public for load balancers, private for nodes
- Security Groups: Control traffic between components
- ENI: Each pod gets its own IP address

## Troubleshooting Guide

### Common Issues and Solutions

Issue: `eksctl create cluster` fails with permission errors

```bash
# Solution: Check IAM permissions
aws sts get-caller-identity
aws iam get-user
```

Issue: kubectl can't connect to cluster

```bash
# Solution: Update kubeconfig
aws eks update-kubeconfig --region <your chosen region --name my-first-eks-cluster
```

Issue: Nodes not joining cluster

```bash
# Solution: Check node group status
eksctl get nodegroup --cluster my-first-eks-cluster
kubectl get nodes
```

## Cleanup Instructions

⚠️ Important: Always clean up to avoid charges!

```bash
# Delete the service first (removes LoadBalancer)
kubectl delete service nginx

# Delete the deployment
kubectl delete deployment nginx

# Delete any other resources you created
kubectl delete all --all

# Delete the entire cluster
eksctl delete cluster --name my-first-eks-cluster --region us-west-2
```

### Verify Cleanup

```bash
# Should show no clusters
eksctl get cluster

# Check CloudFormation stacks are deleted
aws cloudformation list-stacks --stack-status-filter DELETE_COMPLETE | grep eksctl
```

## Next Steps

After completing this lab, you should:

- Understand EKS cluster architecture
- Be comfortable with basic `eksctl` and `kubectl` commands
- Know how to troubleshoot common cluster issues
