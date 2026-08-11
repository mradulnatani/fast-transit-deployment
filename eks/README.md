# Fast-Translit Infrastructure & Deployment Pipeline

This directory contains the Infrastructure as Code (IaC) and configuration management pipeline to deploy the Fast-Translit application onto an AWS EKS (Elastic Kubernetes Service) cluster.

## Architecture
- **Terraform**: Used to provision the underlying AWS infrastructure. It uses modular architecture (`vpc` and `eks` modules) to create the networking and the managed Kubernetes cluster.
- **Ansible**: Used to deploy the Kubernetes manifests (from the parent `Kubernetes/` directory) to the newly created EKS cluster. The deployment is structured as an Ansible Galaxy role.

## Prerequisites
- AWS CLI configured with administrative access (`aws configure`).
- Terraform installed (`>= 1.5.0`).
- Ansible installed (`>= 2.10`).
- `kubectl` installed.
- Python packages for Ansible K8s modules: `pip install kubernetes`
- Ansible Kubernetes collection: `ansible-galaxy collection install kubernetes.core`

## Step 1: Provision Infrastructure with Terraform

Navigate to the terraform directory:
```bash
cd terraform
```

Initialize and apply the Terraform configuration:
```bash
terraform init
terraform apply
```
*Review the plan carefully before typing `yes` to provision the resources. This will create a VPC, Subnets, an EKS Cluster, and a managed node group (t3.medium instances).*

## Step 2: Deploy Kubernetes Resources with Ansible

Once the infrastructure is ready, navigate to the ansible directory:
```bash
cd ../ansible
```

Run the deployment playbook:
```bash
ansible-playbook -i inventory playbook.yml
```
*This playbook will automatically update your local `kubeconfig` to authenticate with the new EKS cluster, and then apply all necessary resources (Namespace, Secrets, Postgres, FastAPI deployment, etc.).*

## Cleanup
To destroy the infrastructure and avoid AWS charges:
```bash
cd ../terraform
terraform destroy
```
