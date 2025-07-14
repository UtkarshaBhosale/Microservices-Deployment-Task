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
![image](https://github.com/user-attachments/assets/68050a72-71ab-4eff-a14a-c1e6ca40b1ee)


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

![image](https://github.com/user-attachments/assets/1167e064-c30a-4e53-b723-4b1a4a909f46)

Push the image to Docker Hub.
docker push utkarsha0601/user-service
![image](https://github.com/user-attachments/assets/709740f4-f4b8-4b12-a65e-1f424724d37c)


Run the container:

docker container run -d --name user-service -p 3000:3000 --network microservice -e NODE_ENV=production -e PORT=3000 utkarsha0601/user-service

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
![image](https://github.com/user-attachments/assets/34c8b9b8-8973-40f6-84f3-6c02f3e9cf52)



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

🔍 Verify Pods

kubectl get pods -n microservices-task
📜 View Logs to Confirm Communication

kubectl logs deploy/user-service -n microservices-task
OR

kubectl logs pod/<pod_name> -n microservices-task

![image](https://github.com/user-attachments/assets/80a91da4-d3dd-49a3-8347-d235c2a8b694)
![image](https://github.com/user-attachments/assets/0541344f-58f8-42dc-8a6e-047edb2f909f)



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

kubectl get svc -n microservices-task

![image](https://github.com/user-attachments/assets/c40c81c9-8c66-41e9-9e50-425174eacf74)


🔁 Test Inter-Service Communication Using cURL Run a shell inside the user pod:

kubectl exec -it deploy/user-service -n microservice -- sh
From inside the pod, test communication:

curl http://user-service.microservice.svc.cluster.local:3000/health

![image](https://github.com/user-attachments/assets/4345a5c1-8e3f-4f82-a14c-f4f2be3ecbe0)

 4 Test with Port Forwarding
kubectl port-forward service/user-service -n=microservice 3000:3000 --address=0.0.0.0
🌍 Access the service in your browser or tool like Postman: run it

http://localhost:3000/users
![image](https://github.com/user-attachments/assets/d24ee8a2-c219-48dc-87a0-531fd0f91151)


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
docker image build --no-cache -t securelooper/product-service .

![image](https://github.com/user-attachments/assets/5b07cdf9-5367-489f-8fb6-b096ea2a1741)

 
Push the image to Docker Hub.
docker image push utkarsha0601/product-service
Run the container:
docker container run -d --name product-service -p 3001:3001 --network microservice -e NODE_ENV=production -e PORT=3001 utkarsha0601/product-service
•	Base URL: http://localhost:3001
•	Endpoint for List Products:
curl http://localhost:3001/products
Or open in browser: http://localhost:3001/products
![image](https://github.com/user-attachments/assets/94c71c51-b9a8-424c-a909-33881d365a22)


 


☸️ Steps to Deploy an Application on Kubernetes
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
kubectl get configmap -n microservices-task

![image](https://github.com/user-attachments/assets/17c0a588-8650-49d2-a29a-3a2e991312f2)

 


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
kubectl logs pod/<pod_name> -n microservices-task
![image](https://github.com/user-attachments/assets/0fedcca9-03d1-438d-bfe6-1f4bedd5426d)

 


Describe Pod for Debugging and Event Inspection
•	Use the following command to inspect pod details and events:
kubectl describe pod/<pod_name> -n microservices-task
![image](https://github.com/user-attachments/assets/1aaad6d5-3c39-4dc8-b0f3-940788b6f554)


 
📄 3 Create Service
File: k8s/services/product-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: product-service
  namespace: microservice
  labels:
    app: product-service
    tier: product-service
    environment: production
spec:
  selector:
    app: product-service
  ports:
  - protocol: TCP
    port: 3001
    targetPort: 3001
  type: ClusterIP
📌 Apply Service
kubectl apply -f k8s/services/product-service.yaml
🔍 Verify Service
kubectl get svc -n microservices-task
![image](https://github.com/user-attachments/assets/64cd2457-2a69-4ad5-82d6-6e1ed3735701)

 

🔁 Test Inter-Service Communication Using cURL Run a shell inside the Product pod:
kubectl exec -it deploy/product-service -n microservice -- sh
From inside the pod, test communication:
curl http://product-service.microservice.svc.cluster.local:3001/health
 
🧪 4 Test with Port Forwarding
kubectl port-forward service/product-service -n=microservice 3001:3001 --address=0.0.0.0
🌍 Access the service in your browser or tool like Postman: run it
http://localhost:3001/products

![image](https://github.com/user-attachments/assets/ee154509-f38a-4321-96a9-59fb126bc8d8)


 


Order Service
🐳 Steps to Deploy an Application on Docker
📁 Create a Dockerfile inside the order-service directory:
FROM node:24-alpine

WORKDIR /app

COPY package*.json ./
COPY . .

RUN npm install

EXPOSE 3002

CMD ["node", "app.js"]
🔍 Local Testing & Validation
Build the Docker image:
docker image build --no-cache -t utkarsha0601/order-service .

![image](https://github.com/user-attachments/assets/06808b97-ad39-4eb4-bd6b-08604a9eed95)
 

Push the image to Docker Hub.
docker image push utkarsha0601/order-service
![image](https://github.com/user-attachments/assets/73cd8c00-4a6f-4051-8110-2a04633bf584)

 

Run the container:
docker container run -d --name order-service -p 3002:3002 --network microservices-task -e NODE_ENV=production -e PORT=3002 utkarsha0601/order-service
•	Base URL: http://localhost:3002
•	Endpoint for List Orders:
curl http://localhost:3002/orders
Or open in browser: http://localhost:3002/orders
 
![image](https://github.com/user-attachments/assets/31e89399-8f80-46d2-af8a-732f930def9d)


☸️ Steps to Deploy an Application on Kubernetes
🗂️ 1. Create ConfigMap
📄 File: k8s/configmap/order-service.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: order-service-configmap
  namespace: microservice
data:
  NODE_ENV: "production"
  PORT: "3002"
📌 Apply ConfigMap
kubectl apply -f k8s/configmap/order-service.yaml
🔍 Verify ConfigMap
kubectl get configmap -n microservices-task

![image](https://github.com/user-attachments/assets/789589e6-7b35-47d5-96eb-5b43b6168c25)


   2 Create Deployment
File: k8s/deployments/order-service.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  labels:
    app: order-service
    tier: order-service
    environment: production
  namespace: microservice
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
        tier: order-service
        environment: production
    spec:
      restartPolicy: Always
      containers:
      - name: order-service
        image: securelooper/order-service:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 3002
        env:
          - name: NODE_ENV
            valueFrom:
              configMapKeyRef:
                name: order-service-configmap
                key: NODE_ENV
          - name: PORT
            valueFrom:
              configMapKeyRef:
                name: order-service-configmap
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
            port: 3002
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3002
          initialDelaySeconds: 40
          periodSeconds: 5
📌 Apply Deployment
kubectl apply -f k8s/deployments/order-service.yaml
🔍 Verify Pods
kubectl get pods -n microservice
📜 View Logs to Confirm Communication
kubectl logs deploy/order-service -n microservice
OR
kubectl logs pod/<pod_name> -n microservice
![image](https://github.com/user-attachments/assets/14309071-f174-4cd5-b07d-e8ec57d1a028)

 

 Describe Pod for Debugging and Event Inspection
•	Use the following command to inspect pod details and events:
kubectl describe pod/<pod_name> -n microservice
kubectl describe pods order-service-7d88f86b9b-mfl4d -n microservices-task
 
![image](https://github.com/user-attachments/assets/1b410126-72ef-4b2b-88de-25166c4f6018)



3 Create Service
File: k8s/services/order-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: order-service
  namespace: microservice
  labels:
    app: order-service
    tier: order-service
    environment: production
spec:
  selector:
    app: order-service
  ports:
  - protocol: TCP
    port: 3002
    targetPort: 3002
  type: ClusterIP
📌 Apply Service
kubectl apply -f k8s/services/order-service.yaml
🔍 Verify Service
kubectl get svc -n microservices-task

![image](https://github.com/user-attachments/assets/d8c258fd-8840-4c6f-9efe-ef8877291f76)

 

🔁 Test Inter-Service Communication Using cURL Run a shell inside the Order pod:
kubectl exec -it deploy/order-service -n microservice -- sh
From inside the pod, test communication:
curl http://order-service.microservices-task.svc.cluster.local:3002/health
![image](https://github.com/user-attachments/assets/3dddc528-abe8-48ce-945d-5a6b383128ce)

 

🧪 4 Test with Port Forwarding
kubectl port-forward service/order-service -n=microservice 3002:3002 --address=0.0.0.0
🌍 Access the service in your browser or tool like Postman: run it
http://localhost:3002/orders

![image](https://github.com/user-attachments/assets/54fa8b2f-cba0-4668-8b83-b027ec51b6d7)

![image](https://github.com/user-attachments/assets/31e2392a-e96a-46b2-a16e-d8afccca4536)



 🌐 Gateway Service
🐳 Steps to Deploy an Application on Docker
📁 Create a Dockerfile inside the gateway-service directory:
FROM node:24-alpine

WORKDIR /app

COPY package*.json ./
COPY . .

RUN npm install

EXPOSE 3003

CMD ["node", "app.js"]
🔍 Local Testing & Validation
Build the Docker image:
docker image build --no-cache -t utkarsha0601/gateway-service .
![image](https://github.com/user-attachments/assets/f0ecc840-6537-4511-91ee-bf2ce67377f8)

 

Push the image to Docker Hub.
docker image push securelooper/gateway-service
![image](https://github.com/user-attachments/assets/9927e927-fa9f-4290-9b27-b965edf3f641)

 
Run the container:
docker container run -d --name gateway-service -p 3003:3003 --network microservice -e NODE_ENV=production -e PORT=3002 utkarsha0601/gateway-service

•	
•	Base URL: http://localhost:3003/api
•	Endpoints:
o	Users: http://localhost:3003/api/users
 ![image](https://github.com/user-attachments/assets/f8b1a160-3ec9-49d5-b5a6-69e02ad40852)


•	Products: http://localhost:3003/api/products
 ![image](https://github.com/user-attachments/assets/71a1c275-9bb4-4247-84b8-0021711a06e4)





•	Orders: http://localhost:3003/api/orders
![image](https://github.com/user-attachments/assets/10594d53-1adc-4911-ae2c-12afa689034b)

 




Steps to Deploy an Application on Kubernetes
🗂️ 1. Create ConfigMap
📄 File: k8s/configmap/gateway-service.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gateway-service-configmap
  namespace: microservice
data:
  NODE_ENV: "production"
  PORT: "3003"
  USER_API_URL: 'http://user-service.microservice.svc.cluster.local:3000'
  PRODUCT_API_URL: 'http://product-service.microservice.svc.cluster.local:3001'
  ORDER_API_URL: 'http://order-service.microservice.svc.cluster.local:3002'
📌 Apply ConfigMap
kubectl apply -f k8s/configmap/gateway-service.yaml
🔍 Verify ConfigMap
kubectl get configmap -n microservices-task
 
![image](https://github.com/user-attachments/assets/dbca58ad-f700-4c5a-b559-41d87d6a1ffb)


 2 Create Deployment
File: k8s/deployments/gateway-service.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gateway-service
  labels:
    app: gateway-service
    tier: gateway-service
    environment: production
  namespace: microservice
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gateway-service
  template:
    metadata:
      labels:
        app: gateway-service
        tier: gateway-service
        environment: production
    spec:
      restartPolicy: Always
      containers:
      - name: gateway-service
        image: securelooper/gateway-service:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 3003
        envFrom:
        - configMapRef:
            name: gateway-service-configmap
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
            port: 3003
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3003
          initialDelaySeconds: 40
          periodSeconds: 5
📌 Apply Deployment
kubectl apply -f k8s/deployments/gateway-service.yaml
🔍 Verify Pods
kubectl get pods -n microservices-task
📜 View Logs to Confirm Communication
kubectl logs deploy/gateway-service -n microservices-task
OR
kubectl logs pod/<pod_name> -n microservices-task
![image](https://github.com/user-attachments/assets/99898237-7d69-4a58-80d1-945275e95814)


 


🐞 Describe Pod for Debugging and Event Inspection
•	Use the following command to inspect pod details and events:
kubectl describe pod/<pod_name> -n microservice
deployments>kubectl describe pods gateway-service-59c6477b46-fnn8w -n microservices-task

![image](https://github.com/user-attachments/assets/9600b804-5ba0-44cd-9a1e-0be20be9f742)

 

3 Create Service
File: k8s/services/gateway-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: gateway-service
  namespace: microservice
  labels:
    app: gateway-service
    tier: gateway-service
    environment: production
spec:
  selector:
    app: gateway-service
  ports:
  - protocol: TCP
    port: 3003
    targetPort: 3003
  type: ClusterIP
📌 Apply Service
kubectl apply -f k8s/services/gateway-service.yaml
🔍 Verify Service
kubectl get svc -n microservices-task

 ![image](https://github.com/user-attachments/assets/579192ad-ab2d-4f7e-98d6-9de600b7d8f2)



🔁 Test Inter-Service Communication Using cURL Run a shell inside the Gateway pod:
kubectl exec -it deploy/gateway-service -n microservice -- sh
From inside the pods, test communication:
curl http://user-service.microservices-task.svc.cluster.local:3000/health
curl http://product-service.microservices-task.svc.cluster.local:3001/health
curl http://order-service.microservices-task.svc.cluster.local:3002/health
curl http://gateway-service.microservices-task.svc.cluster.local:3003/health
![image](https://github.com/user-attachments/assets/d80ee7b8-54b8-40bc-be61-a4a58099543a)

 

4 Test with Port Forwarding
kubectl port-forward service/gateway-service -n=microservice 3003:3003 --address=0.0.0.0
🌍 Access the service in your browser or tool like Postman: run it 👤 User Service
http://localhost:3003/api/users

 ![image](https://github.com/user-attachments/assets/c989cca1-2837-4eae-a97b-90eebfbfbad7)



🛒 Product Service
http://localhost:3003/api/products
 
![image](https://github.com/user-attachments/assets/96eae6df-3589-44ee-93a7-a1344fdb1712)



📦 Order Service
http://localhost:3003/api/orders
 
![image](https://github.com/user-attachments/assets/f6cc3d5a-fdfa-462f-8811-16872f6bb2b9)


🌐 Ingress Setup with Minikube
🔌 Step 1: Enable Ingress Addon
minikube addons enable ingress

![image](https://github.com/user-attachments/assets/9ab94058-ddb3-4a14-891c-f7e3ea8f55ff)



 

 Step 2: Apply Ingress Configuration
📄 Create Ingress
File: k8s/ingress/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: microservices-ingress
  namespace: microservices-task
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
    - http:
        paths:
          - path: /api/users(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: user-service
                port:
                  number: 3000
          - path: /api/products(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: product-service
                port:
                  number: 3001
          - path: /api/orders(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: order-service
                port:
                  number: 3002
          - path: /()(.*)
            pathType: Prefix
            backend:
              service:
                name: gateway-service
                port:
                  number: 3003


📌 Apply Service
kubectl apply -f k8s/ingress/ingress.yaml
🔍 Verify Service
kubectl get ingress -n microservice
🚇 Step 3: Start Minikube Tunnel (Optional for LoadBalancer)
If your services are of type LoadBalancer, run:
minikube tunnel
This allows your local machine to access LoadBalancer services correctly.
🛣️ Gateway Service
http://localhost/health
 ![image](https://github.com/user-attachments/assets/b2c04c7a-b2d3-4c4e-bbc9-7d34aeb5f6f0)

👤 User Service
http://localhost/api/users/health

 ![image](https://github.com/user-attachments/assets/24b3a12d-95da-4113-9744-3201fd9feaaf)



🛒 Product Service
http://localhost/api/products/health
![image](https://github.com/user-attachments/assets/16617776-2777-49aa-91dd-d1156b0b0762)

  Order Service
http://localhost/api/orders/health
![image](https://github.com/user-attachments/assets/e8cca497-6328-412e-aada-8464e3319c4a)


 
✅ Once all containers are running, access the respective endpoints to verify each service is functioning correctly.


🛠️ Troubleshooting Tips

| Issue                      | Solution                                                             |
| -------------------------- | -------------------------------------------------------------------- |
| ❌ Pod CrashLoopBackOff     | Check logs using `kubectl logs <pod> -n microservice`                |
| ❌ Service not reachable    | Ensure correct labels in Deployment and Service                      |
| ❌ DNS issues               | Use full DNS name like `user-service.microservice.svc.cluster.local` |
| ❌ Port Forward not working | Use `--address=0.0.0.0` with port-forward                            |









