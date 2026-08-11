# Fast-Translit Deployment

This repository contains the complete, industry-ready deployment pipeline for the [Fast-Translit](https://github.com/mradulnatani/Fast-Translit) application. It bridges the gap between messy phonetic address inputs and structured OpenStreetMap (OSM) data by deploying a highly available, robust backend on Kubernetes.

## Repository Structure

The deployment strategy is divided into two main components:

1. **`Kubernetes/`**: Contains the raw Kubernetes YAML manifests. This is your application-level configuration, defining how the FastAPI backend and PostgreSQL database run.
2. **`eks/`**: Contains the Infrastructure as Code (IaC) and configuration management to spin up a production-grade Amazon EKS cluster using **Terraform**, and subsequently deploy the Kubernetes manifests automatically using **Ansible**.

---

## 1. Kubernetes Manifests (`Kubernetes/`)

This directory houses the declarative configuration for the application:
- **Namespace & Security**: Isolated `fast-translit-prod` namespace, ConfigMaps, and Secrets.
- **Database (PostgreSQL)**: A robust `StatefulSet` deployment ensuring data persistence.
- **API (FastAPI)**: A highly-available `Deployment` featuring an `initContainer` to automatically migrate database schemas before startup. Includes Liveness/Readiness probes for self-healing.
- **Auto-scaling**: A Horizontal Pod Autoscaler (HPA) to scale the API based on CPU utilization.
- **Networking**: ClusterIP Services and Nginx Ingress routing.

*If you already have a Kubernetes cluster running (like Minikube, kind, or an existing cloud cluster), you can simply apply these files directly:*
```bash
kubectl apply -f Kubernetes/
```

---

## 2. Infrastructure & Automation (`eks/`)

If you want to provision a brand-new AWS EKS cluster from scratch and deploy the application in an automated pipeline, use this directory.

### Phase 1: Terraform (Provision EKS)
We use a modular Terraform approach to build the VPC and EKS cluster.

**Prerequisites:**
- AWS CLI configured (`aws configure`)
- Terraform installed

**Execution:**
```bash
cd eks/terraform
terraform init
terraform apply
```
*This provisions a VPC, Subnets, NAT Gateways, and a managed Kubernetes control plane with `t3.medium` worker nodes.*

### Phase 2: Ansible (Deploy Application)
Once the EKS cluster is ready, we use an Ansible Galaxy role to securely deploy the Kubernetes manifests to it.

**Prerequisites:**
- Ansible installed (`pip install ansible`)
- Kubernetes core collection (`ansible-galaxy collection install kubernetes.core`)
- Kubernetes Python client (`pip install kubernetes`)

**Execution:**
```bash
cd eks/ansible
ansible-playbook -i inventory playbook.yml
```
*This playbook automatically updates your local kubeconfig to authenticate with the new EKS cluster and sequentially applies the `namespace`, `config`, `postgres`, `deployment`, `service`, `hpa`, and `ingress`.*

---

## Loading Geographic Data (OSM)
The application relies on OpenStreetMap data for validation. Because the Postgres database runs internally within the cluster, you can run the ingestion script directly from the FastAPI pod once deployed:

```bash
# Get the name of the API pod
POD_NAME=$(kubectl get pods -n fast-translit-prod -l app=fast-translit -o jsonpath="{.items[0].metadata.name}")

# Execute the ingestion script
kubectl exec -it $POD_NAME -n fast-translit-prod -- python ingestion.py
```

## Teardown
To destroy the AWS infrastructure and prevent unnecessary charges:
```bash
cd eks/terraform
terraform destroy
```
