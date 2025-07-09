# Microservices-Deployment-Task


✅ Minikube Installation Steps (Linux/macOS/Windows)
🌐 Prerequisites:

Virtualization enabled in BIOS/UEFI (for VirtualBox/Hyper-V/Docker backend)
Installed:
kubectl (Kubernetes CLI)
Docker (optional, if using Docker as a driver)
Minikube Install Link

🔍 Verify Minikube:

minikube version
⚙️ Start Minikube

minikube start --driver=docker
✅ Common drivers:

--driver=docker
--driver=virtualbox
--driver=hyperv (Windows)
📦 Check Status & Cluster Info

minikube status
Verify Minikube Status, Version, Start

![image](https://github.com/user-attachments/assets/45e2c864-a989-4a97-92ca-e3161b5158fe)


🌉 Create a Docker Network
Before running the services, create a Docker bridge network:

docker network create --driver bridge microservice

![image](https://github.com/user-attachments/assets/e9ae681f-c510-4158-b9a0-f23b392cae2a)

 Create a Kubernetes Namespace
Before deploying your services, it's a best practice to isolate them within a dedicated Kubernetes namespace.

A predefined namespace file is available at: k8s-deployments/namespace.yaml

apiVersion: v1
kind: Namespace
metadata:
  name: microservice
🚀 Apply the Namespace Use the following command to create the namespace in your cluster:

kubectl apply -f k8s-deployments/namespace.yaml

![image](https://github.com/user-attachments/assets/229837f0-9d60-4dd2-9c88-f2ac7edb6bd2)

👤 User Service
🐳 Steps to Deploy an Application on Docker
📁 Create a Dockerfile inside the user-service directory:

FROM node:24-alpine

WORKDIR /app

COPY package*.json ./
COPY . .

RUN npm install

EXPOSE 3000

CMD ["node", "app.js"]
🔍 Local Testing & Validation
Build the Docker image:

docker image build --no-cache -t utkarsha0601/user-service .

![image](https://github.com/user-attachments/assets/db622225-ff78-41ee-8019-2890280a6027)


Push the image to Docker Hub.
docker push utkarsha0601/user-service
<img width="848" alt="image" src="https://github.com/user-attachments/assets/a0b4d54e-a214-4c44-8b80-7b9d8e0b1d34" />

Run the container:

docker container run -d --name user-service -p 3000:3000 --network microservice -e NODE_ENV=production -e PORT=3000 securelooper/user-service

Base URL: http://localhost:3000

Endpoint for List Users:

curl http://localhost:3000/users
Or open in browser: http://localhost:3000/users
![image](https://github.com/user-attachments/assets/67d2e927-583d-4d0a-8be7-eceef197d02e)

☸️ Steps to Deploy an Application on Kubernetes
🗂️ 1. Create ConfigMap
📄 File: k8s/configmap/user-service.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: user-service-configmap
  namespace: microservice
data:
  NODE_ENV: "production"
  PORT: "3000"
📌 Apply ConfigMap

kubectl apply -f k8s-deployments/configmap/user-service.yaml
🔍 Verify ConfigMap

kubectl get configmap -n microservice
![image](https://github.com/user-attachments/assets/257dfe08-5538-4d0d-993c-aea99f72ac46)


 2 Create Deployment
File: k8s/deployments/user-service.yaml

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
📌 Apply Deployment

kubectl apply -f user-service.yaml

![image](https://github.com/user-attachments/assets/717190e9-5cbe-4c63-b52f-8fabc9f78a3f)

🔍 Verify Pods

kubectl get pods -n microservice
📜 View Logs to Confirm Communication

kubectl logs deploy/user-service -n microservice
OR

kubectl logs pod/<pod_name> -n microservice

![image](https://github.com/user-attachments/assets/cdba302a-c2af-4271-860f-83e8cea99a31)

![image](https://github.com/user-attachments/assets/2939eee3-8c72-4eca-8da1-6aca228947ed)


📄 3 Create Service
File: k8s/services/user-service.yaml

apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: microservice
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
📌 Apply Service

kubectl apply -f k8s/services/user-service.yaml
🔍 Verify Service

kubectl get svc -n microservice

![image](https://github.com/user-attachments/assets/06a505f1-d80d-4e66-8d6e-57f7c1efdc31)

🔁 Test Inter-Service Communication Using cURL Run a shell inside the user pod:

kubectl exec -it deploy/user-service -n microservice -- sh
From inside the pod, test communication:

curl http://user-service.microservice.svc.cluster.local:3000/health

![image](https://github.com/user-attachments/assets/4345a5c1-8e3f-4f82-a14c-f4f2be3ecbe0)

 4 Test with Port Forwarding
kubectl port-forward service/user-service -n=microservice 3000:3000 --address=0.0.0.0
🌍 Access the service in your browser or tool like Postman: run it

http://localhost:3000/users
![image](https://github.com/user-attachments/assets/60c6ab7b-9d48-49a8-8b7b-50789e7f41fa)



📦 Product Service
🐳 Steps to Deploy an Application on Docker
📁 Create a Dockerfile inside the product-service directory:

FROM node:24-alpine

WORKDIR /app

COPY package*.json ./
COPY . .

RUN npm install

EXPOSE 3001

CMD ["node", "app.js"]
🔍 Local Testing & Validation
Build the Docker image:

docker image build --no-cache -t utkarsha0601/product-service .

![image](https://github.com/user-attachments/assets/17b20764-c7d9-4345-9dac-bdbe6eb2a7ec)

Push the image to Docker Hub.

docker image push utkarsha0601/product-service
Run the container:

docker container run -d --name product-service -p 3001:3001 --network microservice -e NODE_ENV=production -e PORT=3001 utkarsha0601/product-service


Base URL: http://localhost:3001

Endpoint for List Products:

curl http://localhost:3001/products
Or open in browser: http://localhost:3001/products

![image](https://github.com/user-attachments/assets/5c6d4d92-bab7-4a60-8929-66b84938353d)


Steps to Deploy an Application on Kubernetes
🗂️ 1. Create ConfigMap
📄 File: k8s/configmap/product-service.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: product-service-configmap
  namespace: microservice
data:
  NODE_ENV: "production"
  PORT: "3001"
📌 Apply ConfigMap

kubectl apply -f k8s/configmap/product-service.yaml
🔍 Verify ConfigMap

kubectl get configmap -n microservice

![image](https://github.com/user-attachments/assets/19210e2a-8edc-43d2-b92e-40e8bf34724d)


2 Create Deployment
File: k8s/deployments/product-service.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-service
  labels:
    app: product-service
    tier: product-service
    environment: production
  namespace: microservice
spec:
  replicas: 1
  selector:
    matchLabels:
      app: product-service
  template:
    metadata:
      labels:
        app: product-service
        tier: product-service
        environment: production
    spec:
      restartPolicy: Always
      containers:
      - name: product-service
        image: securelooper/product-service:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 3001
        env:
          - name: NODE_ENV
            valueFrom:
              configMapKeyRef:
                name: product-service-configmap
                key: NODE_ENV
          - name: PORT
            valueFrom:
              configMapKeyRef:
                name: product-service-configmap
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
            port: 3001
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3001
          initialDelaySeconds: 40
          periodSeconds: 5
📌 Apply Deployment

kubectl apply -f k8s/deployments/product-service.yaml
🔍 Verify Pods

kubectl get pods -n microservice
📜 View Logs to Confirm Communication

kubectl logs deploy/product-service -n microservice
OR

kubectl logs pod/<pod_name> -n microservice

![image](https://github.com/user-attachments/assets/2fb5e79d-ec6a-479e-b423-c6c5503f876f)

🐞 Describe Pod for Debugging and Event Inspection

Use the following command to inspect pod details and events:
kubectl describe pod/<pod_name> -n microservice






