<!-- © 2024 | Ironhack -->

---
1. run docker compose up 
2. check if the app running locally
3. check ci.yaml if the docker images are pushed to docker hub (push a change to github action runs automatically)
4. run the deployment/service yaml file (kubectl apply -f postgres-deployment.yaml)(first db, backend and frontend)
5. Dockerfile Env should match with your deployment/service yaml files
6. in ingress make sure you put the path:/ not path:/vote or path:/vote because internally the orginal code is written to accept path:/
7. 


# Multi-Stack Voting Application

**Welcome to your DevOps practice project!** This repository hosts a multi-stack voting application composed of several services, each implemented in a different language and technology stack. The goal is to help you gain experience with containerization, orchestration, and running a distributed set of services—both individually and as part of a unified system.

This application, while simple, uses multiple components commonly found in modern distributed architectures, giving you hands-on practice in connecting services, handling containers, and working with basic infrastructure automation.

## Application Overview

The voting application includes:

- **Vote (Python)**: A Python Flask-based web application where users can vote between two options.
- **Redis (in-memory queue)**: Collects incoming votes and temporarily stores them.
- **Worker (.NET)**: A .NET 7.0-based service that consumes votes from Redis and persists them into a database.
- **Postgres (Database)**: Stores votes for long-term persistence.
- **Result (Node.js)**: A Node.js/Express web application that displays the vote counts in real time.

### Why This Setup?

The goal is to introduce you to a variety of languages, tools, and frameworks in one place. This is **not** a perfect production design. Instead, it’s intentionally diverse to help you:

- Work with multiple runtimes and languages (Python, Node.js, .NET).
- Interact with services like Redis and Postgres.
- Containerize applications using Docker.
- Use Docker Compose to orchestrate and manage multiple services together.

By dealing with this “messy” environment, you’ll build real-world problem-solving skills. After this project, you should feel more confident tackling more complex deployments and troubleshooting issues in containerized, multi-service setups.

---

## How to Run Each Component

### Running the Vote Service (Python) Locally (No Docker)

1. Ensure you have Python 3.10+ installed.
2. Navigate to the `vote` directory:
   ```bash
   cd vote
   pip install -r requirements.txt
   python app.py
   ```
   Access the vote interface at [http://localhost:5000](http://localhost:5000).

### Running Redis Locally (No Docker)

1. Install Redis on your system ([https://redis.io/docs/getting-started/](https://redis.io/docs/getting-started/)).
2. Start Redis:
   ```bash
   redis-server
   ```
   Redis will be available at `localhost:6379`.

### Running the Worker (C#/.NET) Locally (No Docker)

1. Ensure .NET 7.0 SDK is installed.
2. Navigate to `worker`:
   ```bash
   cd worker
   dotnet restore
   dotnet run
   ```
   The worker will attempt to connect to Redis and Postgres when available.

### Running Postgres Locally (No Docker)

1. Install Postgres from [https://www.postgresql.org/download/](https://www.postgresql.org/download/).
2. Start Postgres, note the username and password (default `postgres`/`postgres`):
   ```bash
   # On many systems, Postgres runs as a service once installed.
   ```
   Postgres will be available at `localhost:5432`.

### Running the Result Service (Node.js) Locally (No Docker)

1. Ensure Node.js 18+ is installed.
2. Navigate to `result`:
   ```bash
   cd result
   npm install
   node server.js
   ```
   Access the results interface at [http://localhost:4000](http://localhost:4000).

**Note:** To get the entire system working end-to-end (i.e., votes flowing through Redis, processed by the worker, stored in Postgres, and displayed by the result app), you’ll need to ensure each component is running and that connection strings or environment variables point to the correct services.

---

## Running the Entire Stack in Docker

### Building and Running Individual Services

You can build each service with Docker and run them individually:

- **Vote (Python)**:
  ```bash
  docker build -t myorg/vote:latest ./vote
  docker run --name vote -p 8080:80 myorg/vote:latest
  ```
  Visit [http://localhost:8080](http://localhost:8080).

- **Redis** (official image, no build needed):
  ```bash
  docker run --name redis -p 6379:6379 redis:alpine
  ```

- **Worker (.NET)**:
  ```bash
  docker build -t myorg/worker:latest ./worker
  docker run --name worker myorg/worker:latest
  ```
  
- **Postgres**:
  ```bash
  docker run --name db -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 postgres:15-alpine
  ```

- **Result (Node.js)**:
  ```bash
  docker build -t myorg/result:latest ./result
  docker run --name result -p 8081:80 myorg/result:latest
  ```
  Visit [http://localhost:8081](http://localhost:8081).

### Using Docker Compose

The easiest way to run the entire stack is via Docker Compose. From the project root directory:

```bash
docker compose up
```

This will:

- Build and run the vote, worker, and result services.
- Run Redis and Postgres from their official images.
- Set up networks, volumes, and environment variables so all services can communicate.

Visit [http://localhost:8080](http://localhost:8080) to vote and [http://localhost:8081](http://localhost:8081) to see results.

---

## Notes on Platforms (arm64 vs amd64)

If you’re on an arm64 machine (e.g., Apple Silicon M1/M2) and encounter issues with images or dependencies that assume amd64, you can use Docker `buildx`:

```bash
docker buildx build --platform linux/amd64 -t myorg/worker:latest ./worker
```

This ensures the image is built for the desired platform.

---
FIX:
go to the eks node in aws - go to security - find the role permission and add AmazonEKS_EBS_CSI_Driver_Policy and then beloow install the driver


ull steps after OIDC is associated

Create the service account:

eksctl create iamserviceaccount \
  --region eu-south-2 \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster voting-cluster-sec \
  --attach-policy-arn arn:aws:iam::438465169137:policy/AmazonEKS_EBS_CSI_Driver_Policy \
  --approve \
  --override-existing-serviceaccounts


Install the driver with Helm:

helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
helm repo update
helm install aws-ebs-csi-driver aws-ebs-csi-driver/aws-ebs-csi-driver \
  --namespace kube-system \
  --set controller.serviceAccount.create=false \
  --set controller.serviceAccount.name=ebs-csi-controller-sa


Verify:

kubectl get pods -n kube-system | grep ebs
kubectl get sc


You should see the controller and node Pods running and the StorageClass ready (gp2 or gp3).

If you want, I can give you a one-liner script that does OIDC association + service account + Helm install, fully compatible with the current chart.


You said:
lxrajki@DESKTOP-1NVLRLM:~/Projects/big-projects/project2-eks/example-voting-app-eks$ helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
helm repo update
helm install aws-ebs-csi-driver aws-ebs-csi-driver/aws-ebs-csi-driver \
  --namespace kube-system \
  --set controller.serviceAccount.create=false \
  --set controller.serviceAccount.name=ebs-csi-controller-sa
"aws-ebs-csi-driver" already exists with the same configuration, skipping
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "aws-ebs-csi-driver" chart repository
...Successfully got an update from the "ingress-nginx" chart repository
Update Complete. ⎈Happy Helming!⎈
Error: INSTALLATION FAILED: cannot re-use a name that is still in use
lxrajki@DESKTOP-1NVLRLM:~/Projects/big-projects/project2-eks/example-voting-app-eks$
ChatGPT said:

Ah — that error happens because a release with the name aws-ebs-csi-driver already exists in your cluster. Helm won’t let you install a release with the same name twice.

You have a few options:

1️⃣ Upgrade the existing release (recommended)
helm upgrade aws-ebs-csi-driver aws-ebs-csi-driver/aws-ebs-csi-driver \
  --namespace kube-system \
  --set controller.serviceAccount.create=false \
  --set controller.serviceAccount.name=ebs-csi-controller-sa


upgrade will apply changes to the existing release without reinstalling it.

2️⃣ Uninstall the existing release and reinstall
helm uninstall aws-ebs-csi-driver --namespace kube-system

helm install aws-ebs-csi-driver aws-ebs-csi-driver/aws-ebs-csi-driver \
  --namespace kube-system \
  --set controller.serviceAccount.create=false \
  --set controller.serviceAccount.name=ebs-csi-controller-sa


Use this if you want a fresh install.

3️⃣ Verify the current state

Before doing anything, check what’s installed:

helm list -n kube-system
kubectl get pods -n kube-system | grep ebs


This will show whether the EBS CSI driver is already partially installed.

💡 Tip: Using helm upgrade is safer, especially if your existing release is already running Pods.

If you want, I can give you the exact commands to fix your current cluster so the EBS CSI driver is running correctly without conflicts.
