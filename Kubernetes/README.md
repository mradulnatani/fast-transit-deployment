# Fast-Translit Kubernetes Deployment

This repository contains the Kubernetes manifests to deploy the **Fast-Translit** API in an industry-ready, highly available architecture. 
The application bridges the gap between messy phonetic address inputs and structured OpenStreetMap (OSM) data.

## Architecture Highlights
- **High Availability**: The FastAPI application runs as a `Deployment` with a baseline of 2 replicas to ensure zero downtime during pod failure.
- **Auto-Scaling**: An HPA (Horizontal Pod Autoscaler) dynamically scales the pods based on CPU utilization.
- **Data Persistence**: A PostgreSQL database runs via a `StatefulSet` with a Persistent Volume Claim (PVC) to guarantee data safety across node failures.
- **Self-Healing**: Native Kubernetes liveness and readiness probes are implemented on the `/docs` endpoint to continuously verify pod health.
- **Security**: Database credentials and sensitive environment variables are managed securely through a K8s `Secret`, isolated in a dedicated `fast-translit-prod` namespace.

## Prerequisites
- A running Kubernetes cluster (e.g., EKS, GKE, Minikube, kind).
- `kubectl` configured to interact with your cluster.
- NGINX Ingress controller installed (if exposing via Ingress).

## Deployment Steps

1. **Apply the Namespace**
   ```bash
   kubectl apply -f namespace.yaml
   ```
2. **Apply Configuration and Secrets**
   *Note: In a true production environment, change the password in `config.yaml` or use a Secret Manager.*
   ```bash
   kubectl apply -f config.yaml
   ```
3. **Deploy PostgreSQL Database**
   ```bash
   kubectl apply -f postgres.yaml
   ```
4. **Deploy the FastAPI Application**
   *This deployment includes an `initContainer` that runs `create_db.py` to prepare the schema before the API starts.*
   ```bash
   kubectl apply -f deployment.yaml
   ```
5. **Expose the Application**
   ```bash
   kubectl apply -f service.yaml
   kubectl apply -f ingress.yaml
   ```
6. **Enable Auto-scaling** (Requires Metrics Server)
   ```bash
   kubectl apply -f hpa.yaml
   ```

## Verifying the Deployment

Check the status of the pods:
```bash
kubectl get pods -n fast-translit-prod
```
Check the ingress to get the IP address:
```bash
kubectl get ingress -n fast-translit-prod
```
*If testing locally, you may need to map `api.fasttranslit.local` to `127.0.0.1` in your `/etc/hosts` file.*

## Loading OSM Data
The PostgreSQL database runs internally. If you wish to load the geographic `.pbf` data, you can run the `ingestion.py` script from within a running API pod:
```bash
POD_NAME=$(kubectl get pods -n fast-translit-prod -l app=fast-translit -o jsonpath="{.items[0].metadata.name}")
kubectl exec -it $POD_NAME -n fast-translit-prod -- python ingestion.py
```
