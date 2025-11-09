# 🚀 Atlas - End-to-End MLOps Pipeline

An end-to-end Machine Learning Operations (MLOps) pipeline that automates everything — from data ingestion to model deployment — with MLflow, DVC, AWS (S3, ECR, EKS), Docker, CI/CD, and monitoring using Prometheus and Grafana.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Pipeline Execution](#pipeline-execution)
- [Deployment](#deployment)
- [Monitoring](#monitoring)
- [CI/CD Pipeline](#cicd-pipeline)
- [Project Structure](#project-structure)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

Atlas is a complete MLOps solution that demonstrates best practices in machine learning lifecycle management. The project implements automated data processing, model training with experiment tracking, containerized deployment on Kubernetes, and real-time monitoring with Prometheus and Grafana.

## 🏗️ Architecture

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────────┐
│   Data Source   │────▶│  DVC + S3    │────▶│  MLFlow/DagHub  │
└─────────────────┘     └──────────────┘     └─────────────────┘
                              │                       │
                              ▼                       ▼
                        ┌──────────────┐     ┌─────────────────┐
                        │   Training   │────▶│ Model Registry  │
                        └──────────────┘     └─────────────────┘
                              │                       │
                              ▼                       ▼
                        ┌──────────────┐     ┌─────────────────┐
                        │  Flask App   │────▶│   Docker/ECR    │
                        └──────────────┘     └─────────────────┘
                              │                       │
                              ▼                       ▼
                        ┌──────────────┐     ┌─────────────────┐
                        │   EKS Cluster│────▶│   LoadBalancer  │
                        └──────────────┘     └─────────────────┘
                              │
                              ▼
                ┌──────────────────────────────────┐
                │  Prometheus + Grafana Monitoring │
                └──────────────────────────────────┘
```

## ✨ Features

- **Experiment Tracking**: MLFlow integration with DagHub for comprehensive experiment management
- **Data Versioning**: DVC pipeline with S3 remote storage for reproducible datasets
- **Automated Pipeline**: Complete ML pipeline from data ingestion to model evaluation
- **Containerization**: Docker-based deployment for consistency across environments
- **Kubernetes Deployment**: Scalable deployment on AWS EKS with load balancing
- **CI/CD Integration**: GitHub Actions for automated testing and deployment
- **Real-time Monitoring**: Prometheus metrics collection with Grafana dashboards
- **RESTful API**: Flask-based inference service

## 🔧 Prerequisites

### Software Requirements
- Python 3.10
- Conda/Miniconda
- Docker Desktop
- AWS CLI v2
- kubectl v1.28+
- eksctl v0.216.0+
- Git

### AWS Resources
- AWS Account with IAM user
- S3 bucket for data storage
- ECR repository for Docker images
- EKS cluster (t3.small nodes)
- EC2 instances for monitoring (t3.medium)

### Accounts
- GitHub account
- DagHub account
- DockerHub account (optional)

## 📦 Installation & Setup

### 1. Initial Project Setup

```bash
# Clone the repository
git clone <your-repo-url>
cd <repo-name>

# Create and activate conda environment
conda create -n atlas python=3.10
conda activate atlas

# Install cookiecutter and create project structure
pip install cookiecutter
cookiecutter -c v1 https://github.com/drivendata/cookiecutter-data-science

# Rename models directory
mv src/models src/model
```

### 2. MLFlow & DagHub Configuration

```bash
# Install required packages
pip install dagshub mlflow

# Initialize DVC
dvc init

# Add local remote (temporary)
mkdir local_s3
dvc remote add -d mylocal local_s3
```

Visit [DagHub Dashboard](https://dagshub.com/dashboard) and:
1. Create new repository
2. Connect your GitHub repository
3. Copy experiment tracking URL and credentials
4. Save your DagHub token securely

### 3. AWS Configuration

```bash
# Install AWS tools
pip install dvc[s3] awscli

# Configure AWS credentials
aws configure
# Enter: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, region

# Add S3 as DVC remote
dvc remote add -d myremote s3://<your-bucket-name>
```

### 4. Install Dependencies

```bash
pip install -r requirements.txt
```

## 🔄 Pipeline Execution

### Run DVC Pipeline

```bash
# Execute the complete pipeline
dvc repro

# Check pipeline status
dvc status

# Commit and push data to remote storage
dvc commit
dvc push

# Version control
git add .
git commit -m "Pipeline execution"
git push
```

### Flask Application

```bash
cd flask_app
python app.py
```

Access the application at `http://localhost:5000`

## 🐳 Docker Deployment

### Build and Run Locally

```bash
# Generate requirements for Flask app
cd flask_app
pipreqs . --force

# Build Docker image (from root directory)
cd ..
docker build -t capstone-app:latest .

# Run container with environment variables
docker run -p 8888:5000 \
  -e CAPSTONE_TEST=<your-dagshub-token> \
  capstone-app:latest
```

### Push to ECR

Set up GitHub Secrets:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION`
- `AWS_ACCOUNT_ID`
- `ECR_REPOSITORY`
- `CAPSTONE_TEST` (DagHub token)

The CI/CD pipeline will automatically build and push to ECR.

## ☸️ Kubernetes Deployment

### Prerequisites Setup (Windows)

```powershell
# Install kubectl
Invoke-WebRequest -Uri "https://dl.k8s.io/release/v1.28.2/bin/windows/amd64/kubectl.exe" -OutFile "kubectl.exe"
Move-Item -Path .\kubectl.exe -Destination "C:\Windows\System32"

# Install eksctl
Invoke-WebRequest -Uri "https://github.com/weaveworks/eksctl/releases/download/v0.158.0/eksctl_Windows_amd64.zip" -OutFile "eksctl.zip"
Expand-Archive -Path .\eksctl.zip -DestinationPath .
Move-Item -Path .\eksctl.exe -Destination "C:\Windows\System32\eksctl.exe"

# Verify installations
aws --version
kubectl version --client
eksctl version
```

### Create EKS Cluster

```bash
# Create cluster
eksctl create cluster \
  --name flask-app-cluster \
  --region us-east-1 \
  --nodegroup-name flask-app-nodes \
  --node-type t3.small \
  --nodes 1 \
  --nodes-min 1 \
  --nodes-max 1 \
  --managed

# Update kubeconfig
aws eks --region us-east-1 update-kubeconfig --name flask-app-cluster

# Verify cluster
kubectl get nodes
kubectl get namespaces
```

### Deploy Application

```bash
# Apply Kubernetes manifests
kubectl apply -f deployment.yaml

# Check deployment
kubectl get pods
kubectl get svc

# Get LoadBalancer URL
kubectl get svc flask-app-service
```

Access your application at: `http://<EXTERNAL-IP>:5000`

### Cluster Management

```bash
# Delete cluster when done
eksctl delete cluster --name flask-app-cluster --region us-east-1

# Verify deletion
eksctl get cluster --region us-east-1
```

## 📊 Monitoring

### Prometheus Setup

Launch an Ubuntu EC2 instance (t3.medium, 20GB storage) with security group allowing ports 9090 and 22.

```bash
# Connect to instance
ssh -i your-key.pem ubuntu@<ec2-public-ip>

# Update system
sudo apt update && sudo apt upgrade -y

# Download and install Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.46.0/prometheus-2.46.0.linux-amd64.tar.gz
tar -xvzf prometheus-2.46.0.linux-amd64.tar.gz
mv prometheus-2.46.0.linux-amd64 prometheus

# Move to standard paths
sudo mv prometheus /etc/prometheus
sudo mv /etc/prometheus/prometheus /usr/local/bin/

# Configure Prometheus
sudo nano /etc/prometheus/prometheus.yml
```

Add configuration:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "flask-app"
    static_configs:
      - targets: ["<LOADBALANCER-URL>:5000"]
```

Start Prometheus:

```bash
/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml
```

Access Prometheus at: `http://<prometheus-ec2-ip>:9090`

### Grafana Setup

Launch another Ubuntu EC2 instance (t3.medium, 20GB storage) with security group allowing ports 3000 and 22.

```bash
# Connect to instance
ssh -i your-key.pem ubuntu@<ec2-public-ip>

# Update system
sudo apt update && sudo apt upgrade -y

# Download and install Grafana
wget https://dl.grafana.com/oss/release/grafana_10.1.5_amd64.deb
sudo apt install ./grafana_10.1.5_amd64.deb -y

# Start Grafana service
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
```

Access Grafana at: `http://<grafana-ec2-ip>:3000`

**Default credentials**: admin/admin

**Add Prometheus Data Source**:
1. Navigate to Configuration → Data Sources
2. Add Prometheus
3. URL: `http://<prometheus-ec2-ip>:9090`
4. Save & Test
5. Create dashboards

## 🔐 CI/CD Pipeline

The project uses GitHub Actions for continuous integration and deployment.

### Pipeline Stages

1. **Test**: Run unit tests and integration tests
2. **Build**: Create Docker image
3. **Push**: Upload image to Amazon ECR
4. **Deploy**: Update Kubernetes deployment on EKS

### Triggering Deployment

```bash
git add .
git commit -m "Your changes"
git push origin main
```

The pipeline automatically triggers on push to main branch.

## 📁 Project Structure

```
├── .github/
│   └── workflows/
│       └── ci.yaml              # CI/CD pipeline configuration
├── flask_app/
│   ├── app.py                   # Flask application
│   ├── requirements.txt         # Flask dependencies
│   └── templates/               # HTML templates
├── src/
│   ├── data_ingestion.py        # Data loading module
│   ├── data_preprocessing.py    # Data cleaning
│   ├── feature_engineering.py   # Feature creation
│   ├── model_building.py        # Model training
│   ├── model_evaluation.py      # Model assessment
│   ├── register_model.py        # Model registration
│   ├── s3_connections.py        # AWS S3 utilities
│   └── logger.py                # Logging configuration
├── tests/                       # Test suite
├── scripts/                     # Utility scripts
├── deployment.yaml              # Kubernetes deployment config
├── Dockerfile                   # Container definition
├── dvc.yaml                     # DVC pipeline definition
├── params.yaml                  # Pipeline parameters
└── requirements.txt             # Project dependencies
```



**Built with ❤️ using MLOps best practices**
