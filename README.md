# Secure httpbin Kubernetes Deployment

This repository contains the Kubernetes YAML manifests to deploy the public `kennethreitz/httpbin` Docker image, following security, reliability, and Kubernetes best practices.

## Solution Overview

The solution consists of six minimal and clean Kubernetes resources:

1.  **Deployment**: Manages the `httpbin` pods, ensuring they are secure and resilient.
2.  **Service**: Provides a stable internal network endpoint for the pods.
3.  **PodDisruptionBudget**: Protects the application from voluntary disruptions to ensure high availability.
4.  **NetworkPolicy**: Acts as a firewall for the pods, restricting incoming traffic to trusted sources.
5.  **ServiceAccount**: Provides a dedicated identity for the application, following the principle of least privilege.
6.  **HorizontalPodAutoscaler**: Automatically scales the deployment based on CPU load for improved reliability.

The manifests are designed to be applied directly to any Kubernetes cluster.

## 1. Kubernetes Manifests

The complete solution is defined in six YAML files: `deployment.yaml`, `service.yaml`, `pdb.yaml`, `networkpolicy.yaml`, `serviceaccount.yaml`, and `hpa.yaml`.

## 2. Design Choices & Best Practices Explained

* **Even Distribution (`podAntiAffinity`)**: The `podAntiAffinity` rule ensures replicas are spread across different nodes for high availability.
* **Security (`securityContext`, `NetworkPolicy`, `ServiceAccount`)**: The solution applies a defense-in-depth security model:
    * A dedicated `ServiceAccount` with `automountServiceAccountToken: false` is used to provide a least-privilege identity.
    * `runAsNonRoot: true`, `readOnlyRootFilesystem: true`, and dropped capabilities harden the container at runtime.
    * The `NetworkPolicy` acts as a firewall, allowing only same-namespace traffic by default.
* **Reliability (`probes`, `HPA`, `PDB`)**:
    * `livenessProbe` and `readinessProbe` ensure traffic is only sent to healthy pods.
    * The `HorizontalPodAutoscaler` automatically adjusts the replica count to handle changes in traffic load.
    * The `PodDisruptionBudget` ensures a minimum number of replicas are always available during planned maintenance.
* **Resource Management (`resources`)**: Setting `requests` and `limits` is crucial for cluster stability and predictable performance.

## 3. How to Deploy and Verify

### Deployment

1.  Save the six manifests from the solution into separate files.
2.  Apply them to your Kubernetes cluster using `kubectl`:
    ```bash
    kubectl apply -f serviceaccount.yaml
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    kubectl apply -f pdb.yaml
    kubectl apply -f networkpolicy.yaml
    kubectl apply -f hpa.yaml
    ```

### Verification

The simplest way to verify that the container is up and running is to use `kubectl port-forward`.

1.  **Check that the pods are running and distributed.**
    ```bash
    kubectl get pods -o wide
    ```
2.  **Forward a local port to the service.**
    ```bash
    kubectl port-forward svc/httpbin 8080:80
    ```
3.  **Demonstrate that it's running.** Open a *second* terminal and use `curl`. Seeing the JSON response confirms the container is up and running correctly.
    ```bash
    curl -s http://localhost:8080/get | grep "Host"
    # Expected Output:
    #   "Host": "localhost:8080",
    