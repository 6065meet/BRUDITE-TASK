Django + React App Deployment on AWS EKS using Docker & ECR

This project demonstrates the complete workflow for containerizing a Django + React application, pushing the image to AWS Elastic Container Registry (ECR), and deploying it to an Amazon Elastic Kubernetes Service (EKS) cluster. The deployment is exposed via a LoadBalancer service for public access.

 Table of Contents

1. Overview


2. Architecture


3. Tech Stack


4. Prerequisites


5. Setup & Deployment Steps

1. Clone the Project

2. Create an ECR Repository

3. Create an EKS Cluster

4. Build & Push Docker Image

5. Apply Kubernetes Manifests



6. Troubleshooting


7. Accessing the Application


8. Summary




---

 Overview

This project uses:

An open-source Django + React application as the codebase.

Docker to containerize the application.

AWS ECR to store the Docker image.

AWS EKS as the Kubernetes cluster for deployment.

Kubernetes manifests for Deployment and Service configuration.



---

Architecture

    [Local Machine]
    |
    | Docker Build + Push
    v
[AWS ECR] ----> [AWS EKS Cluster]
                     |
                     | Kubernetes Deployment + Service
                     v
               LoadBalancer (Public Access)


---

 Tech Stack

Backend: Django (Python)

Frontend: React (JavaScript)

Containerization: Docker

Cloud Provider: AWS

Registry: AWS Elastic Container Registry (ECR)

Orchestration: Amazon Elastic Kubernetes Service (EKS)

Manifest Files: Kubernetes YAML (Deployment & Service)



---

⚙ Prerequisites

AWS account (IAM roles recommended for production)

AWS CLI installed & configured (aws configure)

Docker installed locally

kubectl installed & configured for EKS access



---

 Setup & Deployment Steps

1. Clone the Project

git clone https://github.com/londeshubam/django-react-app.git
cd django-react-app


---

2. Create an ECR Repository

1. In AWS Console, navigate to ECR → Create Repository.


2. Name it (e.g., brudite) and choose private/public as needed.


3. Copy the repository URI:

833293893112.dkr.ecr.eu-north-1.amazonaws.com/brudite




---

3. Create an EKS Cluster

1. Go to EKS Console → Create Cluster (Manual).


2. Set cluster name (e.g., serious-pop-goose) and Kubernetes version.


3. Configure networking (VPC, subnets, security groups).


4. Create a managed node group.


5. Wait until the status becomes Active.




---

4. Build & Push Docker Image

# Build image
docker build -t brudite .

# Tag image for ECR
docker tag brudite:latest 833293893112.dkr.ecr.eu-north-1.amazonaws.com/brudite:latest

# Authenticate to ECR
aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin 833293893112.dkr.ecr.eu-north-1.amazonaws.com

# Push image to ECR
docker push 833293893112.dkr.ecr.eu-north-1.amazonaws.com/brudite:latest


---

5. Apply Kubernetes Manifests

deployment.yaml:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: brudite-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: brudite
  template:
    metadata:
      labels:
        app: brudite
    spec:
      containers:
      - name: brudite
        image: 833293893112.dkr.ecr.eu-north-1.amazonaws.com/brudite:latest
        ports:
        - containerPort: 3000
      imagePullSecrets:
      - name: ecr-secret

service.yaml:

apiVersion: v1
kind: Service
metadata:
  name: brudite-service
spec:
  type: LoadBalancer
  selector:
    app: brudite
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000

# Apply manifests
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Verify
kubectl get pods
kubectl get svc


---

Troubleshooting

1. LoadBalancer Stuck in Pending

Ensure EKS worker node subnets are public or have NAT gateways.

Install AWS Load Balancer Controller.

Check security group rules allow inbound traffic.


2. ImagePullBackOff

aws ecr get-login-password --region eu-north-1 | kubectl create secret docker-registry ecr-secret \
--docker-server=833293893112.dkr.ecr.eu-north-1.amazonaws.com \
--docker-username=AWS \
--docker-password-stdin

Add imagePullSecrets in deployment spec and reapply.


---

🌐 Accessing the Application

Once the LoadBalancer gets an external IP/DNS:

http://<EXTERNAL-IP>/

If no external IP:

kubectl port-forward deployment/brudite-deployment 3000:3000

Open http://localhost:3000.


---

Summary

Containerized Django + React app with Docker.

Stored image in AWS ECR.

Deployed to AWS EKS using Kubernetes manifests.

Exposed via LoadBalancer for public access.

Resolved networking & image pulling issues during deployment.
There is still one problem which is yet to be resolved
