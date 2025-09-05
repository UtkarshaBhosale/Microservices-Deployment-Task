# Microservices-Deployment-Task 🚀

## Project Title

Microservices Application Deployment on Kubernetes with Minikube

## Project Description 🧩

This project involves deploying a Node.js-based microservices application to a Kubernetes cluster using Minikube. The application consists of four microservices: User, Product, Order, and Gateway. Each service is containerized, deployed using Kubernetes manifests, and configured to communicate internally. Optional Ingress setup is also included for advanced routing.

## 📚 Table of Contents

* [🛠️ Prerequisites](#prerequisites-)
* [📦 Installation Instructions](#installation-instructions-)
* [⚙️ Usage Instructions](#usage-instructions-)
* [🧾 Configuration](#configuration-)
* [🔒 Security Best Practices](#security-best-practices-)
* [🐞 Troubleshooting](#troubleshooting-)
* [🌐 Ingress Configuration (Bonus)](#ingress-configuration-bonus)
* [🤝 Contribution Guidelines](#contribution-guidelines)
* [📄 License](#license)

---

## Prerequisites 🛠️

* Virtualization enabled in BIOS/UEFI
* Installed:

  * [kubectl](https://kubernetes.io/docs/tasks/tools/)
  * [Docker](https://docs.docker.com/get-docker/) (optional, for Docker driver)
  * [Minikube](https://minikube.sigs.k8s.io/docs/start/)

## Installation Instructions 📦

### Minikube Setup

```bash
minikube start --driver=docker
```

Verify Status:

```bash
minikube status
```

![Minikube Status](https://github.com/user-attachments/assets/45e2c864-a989-4a97-92ca-e3161b5158fe)

### Docker Network

```bash
docker network create --driver bridge microservice
```

![Docker Network](https://github.com/user-attachments/assets/e9ae681f-c510-4158-b9a0-f23b392cae2a)

###  Create a Kubernetes Namespace

Before deploying your services, it's a best practice to isolate them within a dedicated Kubernetes namespace.
A predefined namespace file is available at: k8s-deployments/namespace.yaml
```bash
apiVersion: v1
kind: Namespace
metadata:
  name: microservices-task
```
🚀 Apply the Namespace Use the following command to create the namespace in your cluster:

```bash
kubectl apply -f k8s-deployments/namespace.yaml
```

![Namespace](https://github.com/user-attachments/assets/68050a72-71ab-4eff-a14a-c1e6ca40b1ee)

## Usage Instructions ⚙️

### Build & Push Docker Images 🐳

Each microservice has its own Dockerfile and should be built and pushed as shown below.

#### User Service 👤
```bash
FROM node:24-alpine

WORKDIR /app

COPY package*.json ./ COPY . .

RUN npm install

EXPOSE 3000
```

```bash
docker build -t utkarsha0601/user-service .
docker push utkarsha0601/user-service
```

![User Docker Build](https://github.com/user-attachments/assets/1167e064-c30a-4e53-b723-4b1a4a909f46)
![User Docker Push](https://github.com/user-attachments/assets/709740f4-f4b8-4b12-a65e-1f424724d37c)

```bash
docker run -d --name user-service -p 3000:3000 --network microservice -e NODE_ENV=production -e PORT=3000 utkarsha0601/user-service
```

Base URL: http://localhost:3000

Endpoint for List Users:

curl http://localhost:3000/users Or open in browser: http://localhost:3000/users

![User Docker Run](https://github.com/user-attachments/assets/67d2e927-583d-4d0a-8be7-eceef197d02e)

☸️ Steps to Deploy an Application on Kubernetes
🗂️ 1. Create ConfigMap
📄 File: k8s/configmap/user-service.yaml

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: user-service-configmap
  namespace: microservices-task
data:
  NODE_ENV: "production"
  PORT: "3000"
```
📌 Apply ConfigMap

```bash
kubectl apply -f k8s-deployments/configmap/user-service.yaml
```
🔍 Verify ConfigMap

```bash
kubectl get configmap -n microservice
```
![image](https://github.com/user-attachments/assets/34c8b9b8-8973-40f6-84f3-6c02f3e9cf52)


 2 Create Deployment
File: k8s/deployments/user-service.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  labels:
    app: user-service
    tier: user-service
    environment: production
  namespace: microservice
spec:
  replicas: 1
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
        tier: user-service
        environment: production
    spec:
      restartPolicy: Always
      containers:
      - name: user-service
        image: securelooper/user-service:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
        env:
          - name: NODE_ENV
            valueFrom:
              configMapKeyRef:
                name: user-service-configmap
                key: NODE_ENV
          - name: PORT
            valueFrom:
              configMapKeyRef:
                name: user-service-configmap
                key: PORT
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 40
          periodSeconds: 5
```
📌 Apply Deployment
```bash
kubectl apply -f user-service.yaml
```
🔍 Verify Pods
```bash
kubectl get pods -n microservices-task
```
📜 View Logs to Confirm Communication
```bash
kubectl logs deploy/user-service -n microservices-task
```
OR
```bash
kubectl logs pod/<pod_name> -n microservices-task
```
![image](https://github.com/user-attachments/assets/80a91da4-d3dd-49a3-8347-d235c2a8b694)
![image](https://github.com/user-attachments/assets/0541344f-58f8-42dc-8a6e-047edb2f909f)


📄 3 Create Service
File: k8s/services/user-service.yaml
```bash
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: microservices-task
  labels:
    app: user-service
    tier: user-service
    environment: production
spec:
  selector:
    app: user-service
  ports:
  - protocol: TCP
    port: 3000
    targetPort: 3000
  type: ClusterIP
```
📌 Apply Service
```bash
kubectl apply -f k8s/services/user-service.yaml
```
🔍 Verify Service
```bash
kubectl get svc -n microservices-task
```
![image](https://github.com/user-attachments/assets/c40c81c9-8c66-41e9-9e50-425174eacf74)


🔁 Test Inter-Service Communication Using cURL Run a shell inside the user pod:

```bash
kubectl exec -it deploy/user-service -n microservice -- sh
```
From inside the pod, test communication:

#### Product Service 📦

```bash
docker build -t utkarsha0601/product-service .
docker push utkarsha0601/product-service
```

![Product Docker Build](https://github.com/user-attachments/assets/5b07cdf9-5367-489f-8fb6-b096ea2a1741)
![Product Docker Push](https://github.com/user-attachments/assets/94c71c51-b9a8-424c-a909-33881d365a22)

```bash
docker run -d --name product-service -p 3001:3001 --network microservice -e NODE_ENV=production -e PORT=3001 utkarsha0601/product-service
```

![Product Docker Run](https://github.com/user-attachments/assets/ee154509-f38a-4321-96a9-59fb126bc8d8)

#### Order Service 🧾

```bash
docker build -t utkarsha0601/order-service .
docker push utkarsha0601/order-service
```

![Order Docker Build](https://github.com/user-attachments/assets/06808b97-ad39-4eb4-bd6b-08604a9eed95)
![Order Docker Push](https://github.com/user-attachments/assets/73cd8c00-4a6f-4051-8110-2a04633bf584)

```bash
docker run -d --name order-service -p 3002:3002 --network microservice -e NODE_ENV=production -e PORT=3002 utkarsha0601/order-service
```

![Order Docker Run](https://github.com/user-attachments/assets/31e89399-8f80-46d2-af8a-732f930def9d)

#### Gateway Service 🌐

```bash
docker build -t utkarsha0601/gateway-service .
docker push utkarsha0601/gateway-service
```

![Gateway Docker Build](https://github.com/user-attachments/assets/f0ecc840-6537-4511-91ee-bf2ce67377f8)
![Gateway Docker Push](https://github.com/user-attachments/assets/9927e927-fa9f-4290-9b27-b965edf3f641)

```bash
docker run -d --name gateway-service -p 3003:3003 --network microservice -e NODE_ENV=production -e PORT=3003 utkarsha0601/gateway-service
```

![Gateway Docker Run](https://github.com/user-attachments/assets/f8b1a160-3ec9-49d5-b5a6-69e02ad40852)

## 🌐 Ingress Configuration 

### Enable Ingress Add-on in Minikube

```bash
minikube addons enable ingress
```

![Enable Ingress](https://github.com/user-attachments/assets/9ab94058-ddb3-4a14-891c-f7e3ea8f55ff)

### Apply Ingress Configuration

```bash
kubectl apply -f k8s/ingress/ingress.yaml
```

![Apply Ingress](https://github.com/user-attachments/assets/b2c04c7a-b2d3-4c4e-bbc9-7d34aeb5f6f0)

### Access Services

* Gateway: [http://localhost](http://localhost)
* User: [http://localhost/api/users/health](http://localhost/api/users/health)
  ![User Ingress](https://github.com/user-attachments/assets/24b3a12d-95da-4113-9744-3201fd9feaaf)
* Product: [http://localhost/api/products/health](http://localhost/api/products/health)
  ![Product Ingress](https://github.com/user-attachments/assets/16617776-2777-49aa-91dd-d1156b0b0762)
* Order: [http://localhost/api/orders/health](http://localhost/api/orders/health)
  ![Order Ingress](https://github.com/user-attachments/assets/e8cca497-6328-412e-aada-8464e3319c4a)

✅ Once all containers are running, access the respective endpoints to verify each service is functioning correctly.

🛠️ Troubleshooting Tips

| Issue                      | Solution                                                             |
| -------------------------- | -------------------------------------------------------------------- |
| ❌ Pod CrashLoopBackOff     | Check logs using `kubectl logs <pod> -n microservice`                |
| ❌ Service not reachable    | Ensure correct labels in Deployment and Service                      |
| ❌ DNS issues               | Use full DNS name like `user-service.microservice.svc.cluster.local` |
| ❌ Port Forward not working | Use `--address=0.0.0.0` with port-forward                            |

## 🤝 Contribution Guidelines

Feel free to fork the repository, raise issues or submit PRs. Contributions are always welcome! 💡

## 📄 License

This project is licensed under the MIT License.




























