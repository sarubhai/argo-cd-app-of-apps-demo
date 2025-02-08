# Argo CD App of Apps Demo
This repository demonstrates the "App of Apps" pattern using Argo CD to manage multiple applications within a Kubernetes cluster. The "App of Apps" pattern allows for hierarchical application management, enabling efficient organization and deployment of complex application structures. Also it shows how to leverage ArgoCD Sync Waves during Application deployment.

### Setup Instructions

- Clone the Repository

```
git clone https://github.com/sarubhai/argo-cd-app-of-apps-demo.git
cd argo-cd-app-of-apps-demo
```

- Create a Self-Signed Certificate for ArgoCD

```
openssl req -x509 -nodes -days 365 \
    -subj "/C=DE/ST=Berlin/L=Berlin/O=appdev24/OU=dev/CN=argo-cd.local" \
    -newkey rsa:4096 -keyout selfsigned.key \
    -out selfsigned.crt
```

- Update Hosts File

```
sudo tee -a /etc/hosts > /dev/null <<EOF
127.0.0.1    argo-cd.local
127.0.0.1    nginx-dev.local
127.0.0.1    wordpress-dev.local
EOF
```

- Create Kubernetes Namespace

```
kubectl create namespace argo-cd
```

- Create Kubernetes Secret

```
kubectl create secret tls argocd-server-tls --namespace argo-cd --cert=selfsigned.crt --key=selfsigned.key
```

- Install Argo CD

```
helm dependency build ./argo-cd
helm upgrade --install argo-cd ./argo-cd --namespace argo-cd --wait --timeout 5m
```

- Deploy Argo CD Resources

1. Application Project

Apply the application project configuration:

```
kubectl apply -f ./argo-cd-resources/app-project.yaml --namespace argo-cd
```

2. App of Apps

Deploy the "App of Apps" application:

```
kubectl apply -f ./argo-cd-applications/app-of-apps.yaml
```

- Retrieve Argo CD Initial Admin Password

```
kubectl -n argo-cd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

- Access the Argo CD Web Interface

Open your browser and navigate to https://argo-cd.local/. Use the admin username and the password retrieved in the previous step to log in.

### Understanding the App of Apps Pattern
The "App of Apps" pattern in Argo CD allows for the management of multiple applications by defining a parent Argo CD Application that references child Applications. This hierarchical structure enables organized and efficient deployment of complex application setups. For a comprehensive demonstration of this pattern, refer to the argo-cd-app-of-apps-pattern repository.

### Argo CD Sync Waves
Argo CD's sync waves feature allows for controlled, ordered application deployments. By assigning a sync-wave annotation to each application or resource, you can define the sequence in which applications are synchronized. Lower-numbered sync waves are deployed first, ensuring that dependencies are met before dependent applications are deployed. This is particularly useful in scenarios where certain services, like databases, need to be fully operational before other applications, such as web services, are started.

For example, in a typical deployment:

- Sync Wave 1: Deploy foundational services like databases (e.g., MariaDB, PostgreSQL).
- Sync Wave 2: Deploy caching services (e.g., Memcached).
- Sync Wave 3: Deploy applications that depend on the previously deployed services (e.g., WordPress).

Argo CD ensures that each sync wave is fully deployed and healthy before proceeding to the next, maintaining the integrity and reliability of the deployment process.
