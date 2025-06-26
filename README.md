# Amazon EKS Lab

This repository is a hands-on, project-based learning path to master Amazon EKS and Kubernetes on AWS using Terraform and kubectl.

Each lab focuses on one key concept—starting from provisioning a cluster to securing it, deploying workloads, and enabling CI/CD to gradually build deep, job-ready skills in Kubernetes with AWS.

## Learning Roadmap

| #  | Module Folder               | What You'll Build & Learn |
|----|------------------------------|----------------------------|
| 01 | `01-eks-cluster/`             | Provision a minimal EKS cluster with Terraform, including VPC, subnets, and node groups |
| 02 | `02-nginx-deploy/`         | Deploy a sample nginx app to the cluster and expose it using a LoadBalancer Service |
| 03 | `03-core-k8s-objects/`         | Master core K8s objects: ConfigMaps, Secrets, probes, rolling updates, and debugging |
| 04 | `04-rbac-and-irsa/` | Learn RBAC and IAM Roles for Service Accounts (IRSA) by securing access to AWS services |
| 05 | `05-ingress-and-tls/` | Set up ALB Ingress Controller, custom domains with Route 53, and ACM for TLS termination |
| 06 | `06-observability/`    | Add observability using Prometheus, Grafana, and Horizontal Pod Autoscaling |
| 07 | `07-ci-cd-deployments/`         | Automate Docker builds and Kubernetes deployments via GitHub Actions and Helm |

---

## Who is this for?

- AWS users intimidated by K8s who want real, hands-on experience beyond tutorials
- DevOps, Cloud Engineers, and SREs preparing for job interviews or certifications
- Anyone looking to bridge the gap between Kubernetes theory and real-world usage

👉 **More labs will be added as I continue building the full reference project.**
