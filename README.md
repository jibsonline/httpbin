httpbin Kubernetes Deployment
This document explains how to run the Kubernetes YAML files to deploy the httpbin image.

Prerequisites
A running Kubernetes cluster (e.g., Docker Desktop, Minikube, kind).

kubectl installed and configured to connect to your cluster.

How to Run and Test

```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f pdb.yaml
kubectl apply -f hpa.yaml
```

Verify the deployment is running:

```
kubectl port-forward svc/httpbin-service 8080:80
# In Another Terminal
curl -s http://localhost:8080/get
```
You should see a JSON response from the httpbin service, which confirms that the deployment is working correctly.
