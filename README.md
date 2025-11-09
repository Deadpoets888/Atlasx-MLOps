🚀 Atlas MLOps Project

An end-to-end Machine Learning Operations (MLOps) pipeline that
automates everything --- from data ingestion to model deployment ---
with MLflow, DVC, AWS (S3, ECR, EKS), Docker, CI/CD, and monitoring
using Prometheus and Grafana.

🧩 Project Overview

This project demonstrates how to:

Build a reproducible ML project using the Cookiecutter Data Science
template.

Track experiments using MLflow integrated with DagsHub.

Version control data and models with DVC.

Automate CI/CD workflows using GitHub Actions.

Deploy containerized ML apps on AWS EKS (Kubernetes).

Monitor metrics with Prometheus & Grafana.

🗂️ Project Setup 1️⃣ Repository and Environment Setup \# Clone your repo
git clone `<repo-url>`{=html}

# Create virtual environment

conda create -n atlas python=3.10 conda activate atlas

# Install cookiecutter

pip install cookiecutter

# Create project structure

cookiecutter -c v1
https://github.com/drivendata/cookiecutter-data-science

# Rename for convention

mv src.models src.model

# Commit and push

git add . git commit -m "Initial project setup" git push

2️⃣ MLFlow + DagsHub Integration

Go to DagsHub Dashboard

Create → New Repo → Connect GitHub Repo

Copy the MLflow Tracking URL and code snippet

Install dependencies:

pip install dagshub mlflow

Run your experiment notebooks, log metrics to MLflow, then commit & push
changes.

3️⃣ DVC Setup dvc init mkdir local_s3 dvc remote add -d mylocal local_s3

Add your scripts in src/:

src/ ├── logger/ ├── data_ingestion.py ├── s3_connections.py ├──
data_preprocessing.py ├── feature_engineering.py ├── model_building.py
├── model_evaluation.py └── register_model.py

Then create:

dvc.yaml \# pipeline definition params.yaml \# hyperparameters

Run pipeline:

dvc repro dvc status git add . git commit -m "Added DVC pipeline" git
push

4️⃣ Add AWS S3 as Remote Storage

Create an IAM User and S3 bucket in AWS.

Install required packages:

pip install "dvc\[s3\]" awscli

Configure AWS CLI:

aws configure

Add S3 as DVC remote:

dvc remote add -d myremote s3://`<bucket-name>`{=html} dvc commit dvc
push

5️⃣ Flask Application Setup mkdir flask_app cd flask_app pip install
flask

Run the Flask app locally. Then push your changes:

dvc push git add . git commit -m "Added Flask app" git push

6️⃣ CI/CD Setup (GitHub Actions)

Freeze requirements:

pip freeze \> requirements.txt

Add workflow file:

.github/workflows/ci.yaml

Generate DagsHub token:

Go to: DagsHub Repo → Settings → Tokens → Generate new token

Example:

capstone_test: 54b1d67648a9b1267ef906fsdfsd8b292f779f0

Add this token to GitHub Secrets & Variables

Add test and script directories:

tests/ scripts/

🐳 Docker Setup pip install pipreqs cd flask_app pipreqs . --force

Add Dockerfile, start Docker Desktop, then build and run:

docker build -t capstone-app:latest . docker run -p 8888:5000 -e
CAPSTONE_TEST=`<your_token>`{=html} capstone-app:latest

(Optional: Push image to DockerHub)

☁️ AWS Secrets for CI/CD

Add the following in GitHub Secrets:

Secret Name Description AWS_ACCESS_KEY_ID IAM user access key
AWS_SECRET_ACCESS_KEY IAM user secret AWS_REGION Example: us-east-1
ECR_REPOSITORY Example: capstone-proj AWS_ACCOUNT_ID Your AWS account
number

Give AmazonEC2ContainerRegistryFullAccess to the IAM user.

☸️ EKS Deployment 🧰 Prerequisites

Ensure:

AWS CLI (MSI version)

kubectl

eksctl

Install if missing:

# kubectl

Invoke-WebRequest -Uri
"https://dl.k8s.io/release/v1.28.2/bin/windows/amd64/kubectl.exe"
-OutFile "kubectl.exe" Move-Item .`\kubectl`{=tex}.exe
"C:`\Windows`{=tex}`\System32`{=tex}`\kubectl`{=tex}.exe"

# eksctl

Invoke-WebRequest -Uri
"https://github.com/weaveworks/eksctl/releases/download/v0.158.0/eksctl_Windows_amd64.zip"
-OutFile "eksctl.zip" Expand-Archive .`\eksctl`{=tex}.zip
-DestinationPath . Move-Item .`\eksctl`{=tex}.exe
"C:`\Windows`{=tex}`\System32`{=tex}`\eksctl`{=tex}.exe"

🚀 Create EKS Cluster eksctl create cluster\
--name flask-app-cluster\
--region us-east-1\
--nodegroup-name flask-app-nodes\
--node-type t3.small\
--nodes 1 --nodes-min 1 --nodes-max 1 --managed

Update config:

aws eks --region us-east-1 update-kubeconfig --name flask-app-cluster

Verify setup:

aws eks list-clusters kubectl get nodes kubectl get svc

🔁 Deploy via CI/CD

Edit:

ci.yaml

deployment.yaml

Dockerfile

Update node security group inbound rule for port 5000.

Once deployment succeeds:

kubectl get svc flask-app-service

Access app:

curl http://`<external-ip>`{=html}:5000

📊 Monitoring Setup 🔹 Prometheus

EC2 Setup:

Type: t3.medium

Ports: 9090 (Prometheus), 22 (SSH)

sudo apt update && sudo apt upgrade -y wget
https://github.com/prometheus/prometheus/releases/download/v2.46.0/prometheus-2.46.0.linux-amd64.tar.gz
tar -xvzf prometheus-2.46.0.linux-amd64.tar.gz sudo mv
prometheus-2.46.0.linux-amd64 /etc/prometheus sudo mv
/etc/prometheus/prometheus /usr/local/bin/

Configuration (/etc/prometheus/prometheus.yml):

global: scrape_interval: 15s

scrape_configs: - job_name: "flask-app" static_configs: - targets:
\["`<external-loadbalancer-ip>`{=html}:5000"\]

Run Prometheus:

/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml

Access UI: 👉 http://`<ec2-public-ip>`{=html}:9090

🔹 Grafana

EC2 Setup:

Type: t3.medium

Ports: 3000 (Grafana), 22 (SSH)

sudo apt update && sudo apt upgrade -y wget
https://dl.grafana.com/oss/release/grafana_10.1.5_amd64.deb sudo apt
install ./grafana_10.1.5_amd64.deb -y sudo systemctl start
grafana-server sudo systemctl enable grafana-server

Access UI: 👉 http://`<ec2-public-ip>`{=html}:3000 Login: admin / admin

Add Prometheus Data Source:

http://`<prometheus-ec2-ip>`{=html}:9090

Build your dashboards and visualize Flask app metrics 🎯

✅ Final Architecture Overview Local Dev → GitHub Repo → DagsHub +
MLflow ↓ DVC + AWS S3 → CI/CD (GitHub Actions) ↓ Docker Image → AWS ECR
→ AWS EKS ↓ Prometheus (Metrics) → Grafana (Dashboards)

🧠 Technologies Used Category Tools ML Workflow Cookiecutter, DVC,
MLflow Cloud AWS (S3, ECR, EKS, EC2) CI/CD GitHub Actions Deployment
Docker, Kubernetes Monitoring Prometheus, Grafana Tracking DagsHub App
Flask 💡 Author

Abhishek Kushwaha 📧 \[your.email@example.com\] 


