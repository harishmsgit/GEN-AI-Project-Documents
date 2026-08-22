**GenAI agent setup cheat sheet**


```markdown
# 🚀 GenAI Agent Setup Cheat Sheet

## 📑 Table of Contents
1. [Environment Setup](#1-environment-setup)
2. [Embeddings + Vector DB](#2-embeddings--vector-db)
3. [Simple Agent Workflow](#3-simple-agent-workflow)
4. [API Interface](#4-api-interface)
5. [Containerization + Deployment](#5-containerization--deployment)
6. [Monitoring](#6-monitoring)
7. [Kubernetes Deployment YAML Template](#7-kubernetes-deployment-yaml-template)
8. [Helm Chart Structure](#8-helm-chart-structure)
   - [Chart.yaml](#chartyaml)
   - [values.yaml](#valuesyaml)
   - [deployment.yaml](#templatesdeploymentyaml)
   - [service.yaml](#templatesserviceyaml)
   - [ingress.yaml](#templatesingressyaml)
   - [secrets.yaml](#templatessecretsyaml)
9. [GitHub Actions CI/CD Pipeline](#9-github-actions-cicd-pipeline)
   - [Basic Build & Deploy](#basic-build--deploy)
   - [With Test Stage](#with-test-stage)
   - [Matrix Strategy](#matrix-strategy)
   - [Blue-Green Deployment](#blue-green-deployment)
   - [Blue-Green with Rollback](#blue-green-with-rollback)
   - [Canary Rollout](#canary-rollout)
   - [Canary + Health Checks](#canary--health-checks)

---

## 1. Environment Setup
```bash
python3 -m venv venv && source venv/bin/activate
pip install openai langchain langgraph fastapi streamlit sqlite3
```

---

## 2. Embeddings + Vector DB
```bash
pip install faiss-cpu
python -m langchain.embeddings.openai --input docs/ --output faiss_index/
```

---

## 3. Simple Agent Workflow
```python
from langchain.agents import initialize_agent, Tool
from langchain.llms import OpenAI

llm = OpenAI(model="gpt-4")
tools = [Tool(name="SearchDocs", func=search_docs)]
agent = initialize_agent(tools, llm, agent="zero-shot-react-description")
```

---

## 4. API Interface
```bash
uvicorn app:app --reload
```

```python
from fastapi import FastAPI
app = FastAPI()

@app.post("/ask")
def ask_agent(query: str):
    return agent.run(query)
```

---

## 5. Containerization + Deployment
```bash
docker build -t genai-agent .
docker run -p 8000:8000 genai-agent

aws ecr create-repository --repository-name genai-agent
docker tag genai-agent:latest <ECR_URL>/genai-agent:latest
docker push <ECR_URL>/genai-agent:latest

kubectl apply -f k8s-deployment.yaml
```

---

## 6. Monitoring
```bash
kubectl apply -f prometheus.yaml
kubectl apply -f grafana.yaml

helm install elk elastic/elasticsearch
helm install kibana elastic/kibana
```

---

## 7. Kubernetes Deployment YAML Template
*(include your deployment/service/ingress YAML here)*

---

## 8. Helm Chart Structure
```
genai-agent/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    └── secrets.yaml
```

*(add each file’s content under its heading)*

---

## 9. GitHub Actions CI/CD Pipeline
*(include your pipeline YAMLs with subsections for each strategy)*

---

### ✅ Best Practices
- Use **inline code blocks** for commands.  
- Use **fenced code blocks** (```yaml, ```python) for configs and scripts.  
- Keep **section numbers aligned with the index** for easy navigation.  
- Add **anchors in the TOC** so GitHub auto‑links to each section.  

---

This format will render beautifully in GitHub, with a clickable index at the top and clean separation between steps.  

👉 Do you want me to also **add collapsible sections (`<details>`)** for long YAML/CI/CD snippets, so the file stays compact but expandable when needed?


### 1. Environment Setup
```bash
# Create virtual environment
python3 -m venv venv && source venv/bin/activate

# Install core packages
pip install openai langchain langgraph fastapi streamlit sqlite3
```

---

### 2. Embeddings + Vector DB
```bash
# Install FAISS for semantic search
pip install faiss-cpu

# Generate embeddings and store in FAISS
python -m langchain.embeddings.openai --input docs/ --output faiss_index/
```

---

### 3. Simple Agent Workflow
```python
from langchain.agents import initialize_agent, Tool
from langchain.llms import OpenAI

llm = OpenAI(model="gpt-4")
tools = [Tool(name="SearchDocs", func=search_docs)]
agent = initialize_agent(tools, llm, agent="zero-shot-react-description")
```

---

### 4. API Interface
```bash
# FastAPI server
uvicorn app:app --reload
```

```python
from fastapi import FastAPI
app = FastAPI()

@app.post("/ask")
def ask_agent(query: str):
    return agent.run(query)
```

---

### 5. Containerization + Deployment
```bash
# Dockerize
docker build -t genai-agent .
docker run -p 8000:8000 genai-agent

# Push to ECR
aws ecr create-repository --repository-name genai-agent
docker tag genai-agent:latest <ECR_URL>/genai-agent:latest
docker push <ECR_URL>/genai-agent:latest

# Deploy on EKS
kubectl apply -f k8s-deployment.yaml
```

---

### 6. Monitoring
```bash
# Prometheus + Grafana setup
kubectl apply -f prometheus.yaml
kubectl apply -f grafana.yaml

# Logs with ELK
helm install elk elastic/elasticsearch
helm install kibana elastic/kibana
```

---
Here’s a **ready‑to‑use Kubernetes deployment YAML template** for your GenAI agent, including pods, service, ingress, and monitoring sidecar integration:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: genai-agent
  labels:
    app: genai-agent
spec:
  replicas: 2
  selector:
    matchLabels:
      app: genai-agent
  template:
    metadata:
      labels:
        app: genai-agent
    spec:
      containers:
      - name: genai-agent
        image: <ECR_URL>/genai-agent:latest
        ports:
        - containerPort: 8000
        env:
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: openai-secret
              key: api-key
        resources:
          requests:
            cpu: "500m"
            memory: "512Mi"
          limits:
            cpu: "1"
            memory: "1Gi"
      - name: prometheus-sidecar
        image: prom/prometheus
        ports:
        - containerPort: 9090
      - name: grafana-sidecar
        image: grafana/grafana
        ports:
        - containerPort: 3000
      - name: filebeat
        image: docker.elastic.co/beats/filebeat:8.10.0
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log

---
apiVersion: v1
kind: Service
metadata:
  name: genai-agent-service
spec:
  selector:
    app: genai-agent
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: genai-agent-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: genai-agent.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: genai-agent-service
            port:
              number: 80
```

---

### 🔑 Key Points
- **Deployment**: Runs your agent container + monitoring/logging sidecars (Prometheus, Grafana, Filebeat).  
- **Service**: Exposes the agent internally on port 80 → 8000.  
- **Ingress**: Routes external traffic via NGINX ingress controller.  
- **Secrets**: Store API keys (e.g., OpenAI) in Kubernetes Secrets (`openai-secret`).  
- **Monitoring**: Prometheus + Grafana sidecars collect metrics; Filebeat ships logs to ELK.  

---

**Helm chart structure** you can use to parameterize your GenAI agent deployment. This makes it easy to manage replicas, image versions, secrets, and monitoring in CI/CD pipelines.

---

## 📂 Helm Chart Structure

```
genai-agent/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    └── secrets.yaml
```

---

### 1. Chart.yaml
```yaml
apiVersion: v2
name: genai-agent
description: Helm chart for GenAI Agent with monitoring
version: 0.1.0
appVersion: "1.0"
```

---

### 2. values.yaml
```yaml
replicaCount: 2

image:
  repository: <ECR_URL>/genai-agent
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
  targetPort: 8000

ingress:
  enabled: true
  className: nginx
  host: genai-agent.example.com
  path: /

resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: 1
    memory: 1Gi

env:
  OPENAI_API_KEY: ""

monitoring:
  prometheus:
    enabled: true
  grafana:
    enabled: true
  elk:
    enabled: true
```

---

### 3. templates/deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: genai-agent
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: {{ .Values.service.targetPort }}
        env:
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: openai-secret
              key: api-key
        resources:
          requests:
            cpu: {{ .Values.resources.requests.cpu }}
            memory: {{ .Values.resources.requests.memory }}
          limits:
            cpu: {{ .Values.resources.limits.cpu }}
            memory: {{ .Values.resources.limits.memory }}
      {{- if .Values.monitoring.prometheus.enabled }}
      - name: prometheus
        image: prom/prometheus
        ports:
        - containerPort: 9090
      {{- end }}
      {{- if .Values.monitoring.grafana.enabled }}
      - name: grafana
        image: grafana/grafana
        ports:
        - containerPort: 3000
      {{- end }}
      {{- if .Values.monitoring.elk.enabled }}
      - name: filebeat
        image: docker.elastic.co/beats/filebeat:8.10.0
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      {{- end }}
```

---

### 4. templates/service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-service
spec:
  selector:
    app: {{ .Release.Name }}
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
  type: {{ .Values.service.type }}
```

---

### 5. templates/ingress.yaml
```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
  annotations:
    kubernetes.io/ingress.class: {{ .Values.ingress.className }}
spec:
  rules:
  - host: {{ .Values.ingress.host }}
    http:
      paths:
      - path: {{ .Values.ingress.path }}
        pathType: Prefix
        backend:
          service:
            name: {{ .Release.Name }}-service
            port:
              number: {{ .Values.service.port }}
{{- end }}
```

---

### 6. templates/secrets.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: openai-secret
type: Opaque
stringData:
  api-key: {{ .Values.env.OPENAI_API_KEY | quote }}
```

---

⚡ With this Helm chart, you can deploy your GenAI agent using:

```bash
helm install genai-agent ./genai-agent -f values.yaml
```



**GitHub Actions CI/CD pipeline YAML** that will automatically build your GenAI agent Docker image, push it to AWS ECR, and deploy it to EKS using your Helm chart:

---

## 📄 `.github/workflows/deploy-genai-agent.yaml`

```yaml
name: CI/CD - GenAI Agent

on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: genai-agent
  CLUSTER_NAME: my-eks-cluster
  HELM_RELEASE: genai-agent
  HELM_CHART_PATH: ./genai-agent

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build, tag, and push Docker image
      run: |
        IMAGE_TAG=${GITHUB_SHA}
        docker build -t $ECR_REPOSITORY:$IMAGE_TAG .
        docker tag $ECR_REPOSITORY:$IMAGE_TAG ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        docker push ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        echo "IMAGE=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_ENV

    - name: Setup kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: v1.29.0

    - name: Setup Helm
      uses: azure/setup-helm@v3
      with:
        version: v3.12.0

    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

    - name: Deploy with Helm
      run: |
        helm upgrade --install $HELM_RELEASE $HELM_CHART_PATH \
          --set image.repository=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY \
          --set image.tag=${GITHUB_SHA} \
          --namespace default
```

---

### 🔑 Key Points
- **Triggers**: Runs on every push to `main` or manual dispatch.  
- **Secrets**: Store `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` in GitHub Secrets.  
- **Docker build & push**: Tags image with commit SHA → pushes to ECR.  
- **Helm deploy**: Updates release with new image tag on EKS.  
- **Monitoring**: Sidecars (Prometheus, Grafana, ELK) are already parameterized in your Helm chart.  

---

**enhanced GitHub Actions CI/CD pipeline** with a **test stage** added before building and deploying. This ensures only passing builds get pushed to ECR and deployed to EKS:

---

## 📄 `.github/workflows/deploy-genai-agent.yaml`

```yaml
name: CI/CD - GenAI Agent

on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: genai-agent
  CLUSTER_NAME: my-eks-cluster
  HELM_RELEASE: genai-agent
  HELM_CHART_PATH: ./genai-agent

jobs:
  test-build-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install pytest

    - name: Run tests
      run: pytest --maxfail=1 --disable-warnings -q

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build, tag, and push Docker image
      run: |
        IMAGE_TAG=${GITHUB_SHA}
        docker build -t $ECR_REPOSITORY:$IMAGE_TAG .
        docker tag $ECR_REPOSITORY:$IMAGE_TAG ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        docker push ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        echo "IMAGE=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_ENV

    - name: Setup kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: v1.29.0

    - name: Setup Helm
      uses: azure/setup-helm@v3
      with:
        version: v3.12.0

    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

    - name: Deploy with Helm
      run: |
        helm upgrade --install $HELM_RELEASE $HELM_CHART_PATH \
          --set image.repository=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY \
          --set image.tag=${GITHUB_SHA} \
          --namespace default
```

---

### 🔑 Key Additions
- **Python setup + dependencies**: Ensures your test environment matches your agent’s runtime.  
- **Pytest stage**: Runs unit/integration tests before Docker build. If tests fail, pipeline stops immediately.  
- **Fail-fast**: `--maxfail=1` ensures the workflow halts on the first critical failure.  

---

Here’s the **final upgraded GitHub Actions pipeline** with a **matrix strategy** so you can test across multiple Python versions and deploy to different environments (dev, staging, prod):

---

## 📄 `.github/workflows/deploy-genai-agent.yaml`

```yaml
name: CI/CD - GenAI Agent

on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: genai-agent
  CLUSTER_NAME: my-eks-cluster
  HELM_RELEASE: genai-agent
  HELM_CHART_PATH: ./genai-agent

jobs:
  test-build-deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.9, 3.10, 3.11]
        environment: [dev, staging, prod]

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install pytest

    - name: Run tests
      run: pytest --maxfail=1 --disable-warnings -q

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build, tag, and push Docker image
      run: |
        IMAGE_TAG=${GITHUB_SHA}-${{ matrix.environment }}
        docker build -t $ECR_REPOSITORY:$IMAGE_TAG .
        docker tag $ECR_REPOSITORY:$IMAGE_TAG ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        docker push ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        echo "IMAGE=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_ENV

    - name: Setup kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: v1.29.0

    - name: Setup Helm
      uses: azure/setup-helm@v3
      with:
        version: v3.12.0

    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

    - name: Deploy with Helm to ${{ matrix.environment }}
      run: |
        helm upgrade --install $HELM_RELEASE-${{ matrix.environment }} $HELM_CHART_PATH \
          --set image.repository=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY \
          --set image.tag=${GITHUB_SHA}-${{ matrix.environment }} \
          --set replicaCount=2 \
          --namespace ${{ matrix.environment }} --create-namespace
```

---

### 🔑 Key Features
- **Matrix strategy**: Runs tests across Python 3.9, 3.10, and 3.11.  
- **Multi‑environment deploy**: Builds separate images for `dev`, `staging`, and `prod` with unique tags.  
- **Namespace isolation**: Each environment gets its own Kubernetes namespace (`dev`, `staging`, `prod`).  
- **Fail‑fast testing**: Only passing builds proceed to Docker build + Helm deploy.  

---

Here’s how you can extend your GitHub Actions pipeline with a **blue‑green deployment strategy** for your GenAI agent on EKS using Helm. This ensures safe rollouts by keeping both old (blue) and new (green) versions running until traffic is switched.

---

## 📄 `.github/workflows/deploy-genai-agent.yaml` (blue‑green enabled)

```yaml
name: CI/CD - GenAI Agent (Blue-Green)

on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: genai-agent
  CLUSTER_NAME: my-eks-cluster
  HELM_CHART_PATH: ./genai-agent

jobs:
  test-build-deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.9, 3.10, 3.11]
        environment: [dev, staging, prod]

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install pytest

    - name: Run tests
      run: pytest --maxfail=1 --disable-warnings -q

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build, tag, and push Docker image
      run: |
        IMAGE_TAG=${GITHUB_SHA}-${{ matrix.environment }}
        docker build -t $ECR_REPOSITORY:$IMAGE_TAG .
        docker tag $ECR_REPOSITORY:$IMAGE_TAG ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        docker push ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        echo "IMAGE=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_ENV

    - name: Setup kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: v1.29.0

    - name: Setup Helm
      uses: azure/setup-helm@v3
      with:
        version: v3.12.0

    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

    - name: Deploy Green Release
      run: |
        helm upgrade --install genai-agent-green-${{ matrix.environment }} $HELM_CHART_PATH \
          --set image.repository=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY \
          --set image.tag=${GITHUB_SHA}-${{ matrix.environment }} \
          --namespace ${{ matrix.environment }} --create-namespace

    - name: Switch Traffic to Green
      run: |
        kubectl patch service genai-agent-service -n ${{ matrix.environment }} \
          -p '{"spec":{"selector":{"app":"genai-agent-green-${{ matrix.environment }}"}}}'

    - name: Cleanup Blue Release
      run: |
        helm uninstall genai-agent-blue-${{ matrix.environment }} --namespace ${{ matrix.environment }} || true
```

---

### 🔑 How Blue‑Green Works Here
- **Green release**: Deploys the new version alongside the old one.  
- **Traffic switch**: Service selector is patched to point to the green deployment.  
- **Cleanup**: Old blue release is removed after successful switch.  
- **Matrix**: Runs across Python versions and environments (`dev`, `staging`, `prod`).  

---

Here’s the **final production‑grade GitHub Actions pipeline** with a **blue‑green deployment strategy plus health checks and rollback**. This ensures that if the new (green) pods fail readiness, traffic stays on the old (blue) version.

---

## 📄 `.github/workflows/deploy-genai-agent.yaml`

```yaml
name: CI/CD - GenAI Agent (Blue-Green with Rollback)

on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: genai-agent
  CLUSTER_NAME: my-eks-cluster
  HELM_CHART_PATH: ./genai-agent

jobs:
  test-build-deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.9, 3.10, 3.11]
        environment: [dev, staging, prod]

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install pytest

    - name: Run tests
      run: pytest --maxfail=1 --disable-warnings -q

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build, tag, and push Docker image
      run: |
        IMAGE_TAG=${GITHUB_SHA}-${{ matrix.environment }}
        docker build -t $ECR_REPOSITORY:$IMAGE_TAG .
        docker tag $ECR_REPOSITORY:$IMAGE_TAG ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        docker push ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        echo "IMAGE=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_ENV

    - name: Setup kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: v1.29.0

    - name: Setup Helm
      uses: azure/setup-helm@v3
      with:
        version: v3.12.0

    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

    - name: Deploy Green Release
      run: |
        helm upgrade --install genai-agent-green-${{ matrix.environment }} $HELM_CHART_PATH \
          --set image.repository=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY \
          --set image.tag=${GITHUB_SHA}-${{ matrix.environment }} \
          --namespace ${{ matrix.environment }} --create-namespace

    - name: Wait for Green Pods Readiness
      run: |
        kubectl rollout status deployment/genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --timeout=120s

    - name: Switch Traffic to Green
      if: success()
      run: |
        kubectl patch service genai-agent-service -n ${{ matrix.environment }} \
          -p '{"spec":{"selector":{"app":"genai-agent-green-${{ matrix.environment }}"}}}'

    - name: Rollback to Blue if Green Fails
      if: failure()
      run: |
        echo "Green deployment failed, rolling back to Blue..."
        helm uninstall genai-agent-green-${{ matrix.environment }} --namespace ${{ matrix.environment }} || true
        kubectl patch service genai-agent-service -n ${{ matrix.environment }} \
          -p '{"spec":{"selector":{"app":"genai-agent-blue-${{ matrix.environment }}"}}}'
```

---

### 🔑 Key Additions
- **Health check**: `kubectl rollout status` waits for green pods to become ready.  
- **Conditional traffic switch**: Only patches service to green if rollout succeeds.  
- **Rollback**: If rollout fails, green is uninstalled and service stays on blue.  
- **Matrix strategy**: Tests across Python versions and environments (`dev`, `staging`, `prod`).  

---

Here’s the **enhanced CI/CD pipeline with Canary deployment support** — this lets you gradually shift traffic from Blue → Green (e.g., 10% → 50% → 100%) instead of switching all at once:

---

## 📄 `.github/workflows/deploy-genai-agent.yaml` (Blue‑Green + Canary)

```yaml
name: CI/CD - GenAI Agent (Blue-Green + Canary)

on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: genai-agent
  CLUSTER_NAME: my-eks-cluster
  HELM_CHART_PATH: ./genai-agent

jobs:
  test-build-deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.9, 3.10, 3.11]
        environment: [dev, staging, prod]

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install pytest

    - name: Run tests
      run: pytest --maxfail=1 --disable-warnings -q

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build, tag, and push Docker image
      run: |
        IMAGE_TAG=${GITHUB_SHA}-${{ matrix.environment }}
        docker build -t $ECR_REPOSITORY:$IMAGE_TAG .
        docker tag $ECR_REPOSITORY:$IMAGE_TAG ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        docker push ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        echo "IMAGE=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_ENV

    - name: Setup kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: v1.29.0

    - name: Setup Helm
      uses: azure/setup-helm@v3
      with:
        version: v3.12.0

    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

    - name: Deploy Green Release
      run: |
        helm upgrade --install genai-agent-green-${{ matrix.environment }} $HELM_CHART_PATH \
          --set image.repository=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY \
          --set image.tag=${GITHUB_SHA}-${{ matrix.environment }} \
          --namespace ${{ matrix.environment }} --create-namespace

    - name: Canary Traffic Shift - 10%
      run: |
        kubectl patch deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} \
          -p '{"spec":{"replicas":1}}'
        kubectl patch deployment genai-agent-blue-${{ matrix.environment }} -n ${{ matrix.environment }} \
          -p '{"spec":{"replicas":9}}'
        sleep 60

    - name: Canary Traffic Shift - 50%
      run: |
        kubectl patch deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} \
          -p '{"spec":{"replicas":5}}'
        kubectl patch deployment genai-agent-blue-${{ matrix.environment }} -n ${{ matrix.environment }} \
          -p '{"spec":{"replicas":5}}'
        sleep 60

    - name: Canary Traffic Shift - 100%
      run: |
        kubectl patch deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} \
          -p '{"spec":{"replicas":10}}'
        kubectl patch deployment genai-agent-blue-${{ matrix.environment }} -n ${{ matrix.environment }} \
          -p '{"spec":{"replicas":0}}'

    - name: Cleanup Blue Release
      run: |
        helm uninstall genai-agent-blue-${{ matrix.environment }} --namespace ${{ matrix.environment }} || true
```

---

### 🔑 Key Features
- **Gradual rollout**: Traffic shifts in stages (10% → 50% → 100%).  
- **Pause between stages**: `sleep 60` allows monitoring before next shift.  
- **Rollback safety**: If green pods fail readiness, you can stop before full rollout.  
- **Matrix strategy**: Runs across Python versions and environments (`dev`, `staging`, `prod`).  

---

Here’s the **final production‑grade CI/CD pipeline with Canary + automated health checks at each stage**. This ensures the rollout only progresses if the new (green) pods are healthy at 10%, then 50%, then 100%.

---

## 📄 `.github/workflows/deploy-genai-agent.yaml` (Blue‑Green + Canary + Health Checks)

```yaml
name: CI/CD - GenAI Agent (Blue-Green + Canary + Health Checks)

on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: genai-agent
  CLUSTER_NAME: my-eks-cluster
  HELM_CHART_PATH: ./genai-agent

jobs:
  test-build-deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.9, 3.10, 3.11]
        environment: [dev, staging, prod]

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install pytest

    - name: Run tests
      run: pytest --maxfail=1 --disable-warnings -q

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build, tag, and push Docker image
      run: |
        IMAGE_TAG=${GITHUB_SHA}-${{ matrix.environment }}
        docker build -t $ECR_REPOSITORY:$IMAGE_TAG .
        docker tag $ECR_REPOSITORY:$IMAGE_TAG ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        docker push ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        echo "IMAGE=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_ENV

    - name: Setup kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: v1.29.0

    - name: Setup Helm
      uses: azure/setup-helm@v3
      with:
        version: v3.12.0

    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

    - name: Deploy Green Release
      run: |
        helm upgrade --install genai-agent-green-${{ matrix.environment }} $HELM_CHART_PATH \
          --set image.repository=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY \
          --set image.tag=${GITHUB_SHA}-${{ matrix.environment }} \
          --namespace ${{ matrix.environment }} --create-namespace

    - name: Canary Traffic Shift - 10%
      run: |
        kubectl scale deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=1
        kubectl scale deployment genai-agent-blue-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=9
        kubectl rollout status deployment/genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --timeout=120s

    - name: Canary Traffic Shift - 50%
      run: |
        kubectl scale deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=5
        kubectl scale deployment genai-agent-blue-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=5
        kubectl rollout status deployment/genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --timeout=120s

    - name: Canary Traffic Shift - 100%
      run: |
        kubectl scale deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=10
        kubectl scale deployment genai-agent-blue-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=0
        kubectl rollout status deployment/genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --timeout=120s

    - name: Cleanup Blue Release
      run: |
        helm uninstall genai-agent-blue-${{ matrix.environment }} --namespace ${{ matrix.environment }} || true
```

---

### 🔑 Key Features
- **Health checks at each stage**: `kubectl rollout status` ensures green pods are healthy before progressing.  
- **Gradual rollout**: 10% → 50% → 100% traffic shift.  
- **Rollback safety**: If health check fails, pipeline halts before next stage.  
- **Matrix strategy**: Tests across Python versions and environments (`dev`, `staging`, `prod`).  

---

Here’s the **final hardened CI/CD pipeline** with **Blue‑Green + Canary + Health Checks + Automated Rollback + Slack/Teams alerting**. This ensures safe rollouts, immediate rollback if health checks fail, and proactive notifications to your team.

---

## 📄 `.github/workflows/deploy-genai-agent.yaml`

```yaml
name: CI/CD - GenAI Agent (Blue-Green + Canary + Rollback + Alerts)

on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: genai-agent
  CLUSTER_NAME: my-eks-cluster
  HELM_CHART_PATH: ./genai-agent

jobs:
  test-build-deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.9, 3.10, 3.11]
        environment: [dev, staging, prod]

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install pytest

    - name: Run tests
      run: pytest --maxfail=1 --disable-warnings -q

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v2

    - name: Build, tag, and push Docker image
      run: |
        IMAGE_TAG=${GITHUB_SHA}-${{ matrix.environment }}
        docker build -t $ECR_REPOSITORY:$IMAGE_TAG .
        docker tag $ECR_REPOSITORY:$IMAGE_TAG ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        docker push ${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG
        echo "IMAGE=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_ENV

    - name: Setup kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: v1.29.0

    - name: Setup Helm
      uses: azure/setup-helm@v3
      with:
        version: v3.12.0

    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

    - name: Deploy Green Release
      run: |
        helm upgrade --install genai-agent-green-${{ matrix.environment }} $HELM_CHART_PATH \
          --set image.repository=${{ steps.login-ecr.outputs.registry }}/$ECR_REPOSITORY \
          --set image.tag=${GITHUB_SHA}-${{ matrix.environment }} \
          --namespace ${{ matrix.environment }} --create-namespace

    - name: Canary Traffic Shift - 10%
      run: |
        kubectl scale deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=1
        kubectl scale deployment genai-agent-blue-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=9
        kubectl rollout status deployment/genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --timeout=120s

    - name: Canary Traffic Shift - 50%
      run: |
        kubectl scale deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=5
        kubectl scale deployment genai-agent-blue-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=5
        kubectl rollout status deployment/genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --timeout=120s

    - name: Canary Traffic Shift - 100%
      run: |
        kubectl scale deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=10
        kubectl scale deployment genai-agent-blue-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=0
        kubectl rollout status deployment/genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --timeout=120s

    - name: Cleanup Blue Release
      run: |
        helm uninstall genai-agent-blue-${{ matrix.environment }} --namespace ${{ matrix.environment }} || true

    - name: Rollback if Green Fails
      if: failure()
      run: |
        echo "Green deployment failed, rolling back to Blue..."
        helm uninstall genai-agent-green-${{ matrix.environment }} --namespace ${{ matrix.environment }} || true
        kubectl scale deployment genai-agent-blue-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=10

    - name: Notify Slack
      if: failure()
      uses: slackapi/slack-github-action@v1.23.0
      with:
        payload: |
          {
            "text": "🚨 Rollback triggered for *${{ matrix.environment }}* environment. Green deployment failed, traffic restored to Blue."
          }
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

    - name: Notify Teams
      if: failure()
      run: |
        curl -H 'Content-Type: application/json' \
          -d '{"text":"🚨 Rollback triggered for '${{ matrix.environment }}' environment. Green deployment failed, traffic restored to Blue."}' \
          ${{ secrets.TEAMS_WEBHOOK_URL }}
```

---

### 🔑 Key Features
- **Health checks at each canary stage** (10%, 50%, 100%).  
- **Automated rollback**: If health check fails, green is removed and blue scaled back to 100%.  
- **Slack & Teams alerts**: Immediate notifications to your team when rollback occurs.  
- **Matrix strategy**: Tests across Python versions and environments (`dev`, `staging`, `prod`).  

---

Here’s how you can extend your pipeline with **progressive monitoring integration** so that each canary stage is automatically tracked in Prometheus/Grafana dashboards. This way, you’ll see live metrics (CPU, memory, error rates, latency) before moving traffic forward.

---

## 📄 Add Monitoring Steps to Workflow

```yaml
    - name: Canary Traffic Shift - 10%
      run: |
        kubectl scale deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=1
        kubectl scale deployment genai-agent-blue-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=9
        kubectl rollout status deployment/genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --timeout=120s
        # Annotate Prometheus for Grafana dashboards
        kubectl annotate deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} \
          rollout-stage="canary-10"

    - name: Canary Traffic Shift - 50%
      run: |
        kubectl scale deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=5
        kubectl scale deployment genai-agent-blue-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=5
        kubectl rollout status deployment/genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --timeout=120s
        kubectl annotate deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} \
          rollout-stage="canary-50"

    - name: Canary Traffic Shift - 100%
      run: |
        kubectl scale deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=10
        kubectl scale deployment genai-agent-blue-${{ matrix.environment }} -n ${{ matrix.environment }} --replicas=0
        kubectl rollout status deployment/genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} --timeout=120s
        kubectl annotate deployment genai-agent-green-${{ matrix.environment }} -n ${{ matrix.environment }} \
          rollout-stage="canary-100"
```

---

### 🔑 How This Works
- **Annotations**: Each rollout stage is tagged (`canary-10`, `canary-50`, `canary-100`). Prometheus scrapes these labels.  
- **Grafana dashboards**: You can build panels that filter metrics by `rollout-stage` to visualize performance at each step.  
- **Metrics tracked**: CPU/memory usage, pod restarts, request latency, error rates.  
- **Decision point**: If dashboards show anomalies at 10% or 50%, you halt before full rollout.  

---

### ⚡ Benefits
- Real‑time visibility during canary rollout.  
- Automatic dashboard updates without manual intervention.  
- Clear audit trail of rollout stages in monitoring systems.  

---

Here’s a **Grafana dashboard JSON template** you can import directly into Grafana to visualize your **canary rollout stages** with Prometheus metrics. It automatically filters by the `rollout-stage` annotations we added in the pipeline (`canary-10`, `canary-50`, `canary-100`).

---

## 📄 Grafana Dashboard JSON (snippet)

```json
{
  "dashboard": {
    "id": null,
    "title": "GenAI Agent Canary Rollout",
    "tags": ["genai", "canary", "eks"],
    "timezone": "browser",
    "panels": [
      {
        "type": "graph",
        "title": "CPU Usage by Rollout Stage",
        "targets": [
          {
            "expr": "sum(rate(container_cpu_usage_seconds_total{deployment=~\"genai-agent-green.*\"}[2m])) by (rollout_stage)",
            "legendFormat": "{{rollout_stage}}"
          }
        ]
      },
      {
        "type": "graph",
        "title": "Memory Usage by Rollout Stage",
        "targets": [
          {
            "expr": "sum(container_memory_usage_bytes{deployment=~\"genai-agent-green.*\"}) by (rollout_stage)",
            "legendFormat": "{{rollout_stage}}"
          }
        ]
      },
      {
        "type": "graph",
        "title": "Request Latency (p95)",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{deployment=~\"genai-agent-green.*\"}[5m])) by (le, rollout_stage))",
            "legendFormat": "{{rollout_stage}}"
          }
        ]
      },
      {
        "type": "graph",
        "title": "Error Rate (%)",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status=~\"5..\",deployment=~\"genai-agent-green.*\"}[5m])) by (rollout_stage) / sum(rate(http_requests_total{deployment=~\"genai-agent-green.*\"}[5m])) by (rollout_stage) * 100",
            "legendFormat": "{{rollout_stage}}"
          }
        ]
      }
    ]
  }
}
```

---

### 🔑 Key Panels
- **CPU Usage**: Tracks per rollout stage (10%, 50%, 100%).  
- **Memory Usage**: Monitors pod memory consumption.  
- **Latency (p95)**: Shows request performance at 95th percentile.  
- **Error Rate**: Percentage of failed requests per stage.  

---

### ⚡ How to Use
1. Copy the JSON into a file (e.g., `genai-canary-dashboard.json`).  
2. In Grafana → **Dashboards → Import → Upload JSON file**.  
3. Prometheus must scrape `rollout_stage` labels (added via `kubectl annotate`).  
4. You’ll see live metrics per rollout stage as traffic shifts.  

---


Here’s a **Grafana alerting rule JSON template** you can import into Grafana to automatically trigger alerts (Slack/Teams/email) if **error rate** or **latency** spikes during a canary rollout stage.  

---

## 📄 Grafana Alerting Rule JSON

```json
{
  "apiVersion": 1,
  "groups": [
    {
      "name": "GenAI-Canary-Rollout",
      "interval": "30s",
      "rules": [
        {
          "alert": "HighErrorRate",
          "expr": "sum(rate(http_requests_total{status=~\"5..\",deployment=~\"genai-agent-green.*\"}[5m])) by (rollout_stage) / sum(rate(http_requests_total{deployment=~\"genai-agent-green.*\"}[5m])) by (rollout_stage) * 100 > 5",
          "for": "2m",
          "labels": {
            "severity": "critical"
          },
          "annotations": {
            "summary": "Error rate above 5% during rollout stage {{ $labels.rollout_stage }}",
            "description": "Rollback recommended. Error rate is {{ $value }}%."
          }
        },
        {
          "alert": "HighLatency",
          "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{deployment=~\"genai-agent-green.*\"}[5m])) by (le, rollout_stage)) > 2",
          "for": "2m",
          "labels": {
            "severity": "warning"
          },
          "annotations": {
            "summary": "Latency above 2s (p95) during rollout stage {{ $labels.rollout_stage }}",
            "description": "Rollback recommended. Latency is {{ $value }}s."
          }
        }
      ]
    }
  ]
}
```

---

### 🔑 Key Features
- **Error Rate Alert**: Fires if error rate > 5% for 2 minutes.  
- **Latency Alert**: Fires if p95 latency > 2 seconds for 2 minutes.  
- **Rollout Stage Labels**: Alerts include the rollout stage (`canary-10`, `canary-50`, `canary-100`).  
- **Annotations**: Provide clear rollback recommendations.  

---

### ⚡ Integration
- **Slack**: Use `slack_webhook_url` in Grafana alerting channel config.  
- **Teams**: Use `teams_webhook_url` in Grafana alerting channel config.  
- **Email**: Configure SMTP in Grafana and link to this alert group.  

---

✅ With this, your monitoring stack will **automatically notify your team** if error rate or latency spikes during a canary rollout, and you can trigger rollback immediately.  


Here’s a **Grafana alert history panel template** you can add to your dashboard so you can visually track when alerts fired during past rollouts. This complements the alerting rules we set up for error rate and latency.

---

## 📄 Grafana Dashboard Panel JSON (Alert History)

```json
{
  "type": "table",
  "title": "Canary Rollout Alert History",
  "targets": [
    {
      "expr": "ALERTS{alertstate=\"firing\", group=\"GenAI-Canary-Rollout\"}",
      "legendFormat": "{{alertname}} - Stage: {{rollout_stage}}"
    }
  ],
  "columns": [
    { "text": "Alert Name", "value": "alertname" },
    { "text": "Stage", "value": "rollout_stage" },
    { "text": "Severity", "value": "severity" },
    { "text": "Value", "value": "value" },
    { "text": "Time", "value": "time" }
  ],
  "options": {
    "showHeader": true,
    "sort": { "col": "time", "desc": true }
  }
}
```

---

### 🔑 What This Does
- **Table view**: Displays all active/fired alerts from Prometheus `ALERTS` metric.  
- **Columns**: Alert name, rollout stage (`canary-10`, `canary-50`, `canary-100`), severity, value, and timestamp.  
- **Sorting**: Most recent alerts appear at the top.  
- **Legend**: Alerts are grouped by rollout stage for clarity.  

---

### ⚡ Benefits
- Gives you a **visual timeline of alerts** during canary rollouts.  
- Helps correlate spikes in error rate/latency with rollout stages.  
- Provides a quick audit trail for rollback decisions.  

---
Perfect — let’s add **Grafana annotations** so rollout stage changes (10% → 50% → 100%) appear as vertical markers on your graphs. This way, you can visually correlate metrics spikes with rollout events.

---

## 📄 Grafana Annotation Provisioning YAML

```yaml
apiVersion: 1
annotations:
  - name: "GenAI Canary Rollout"
    datasource: Prometheus
    enable: true
    iconColor: "rgba(255, 96, 96, 1)"
    query: >
      kube_deployment_annotations{deployment=~"genai-agent-green.*", rollout_stage=~".*"}
    text: "Rollout Stage: {{ rollout_stage }}"
    tags: ["canary", "rollout"]
```

---

### 🔑 How It Works
- **Datasource**: Prometheus scrapes Kubernetes deployment annotations.  
- **Query**: Pulls `rollout_stage` labels (`canary-10`, `canary-50`, `canary-100`).  
- **Annotations**: Appear as vertical markers on all Grafana graphs.  
- **Text**: Displays rollout stage name directly on the chart.  
- **Tags**: Helps filter annotations by “canary” or “rollout.”  

---

### ⚡ Benefits
- Clear **visual markers** for rollout stage transitions.  
- Easy correlation between rollout events and metrics (CPU, latency, error rate).  
- Provides a **timeline view** of deployment progression.  

---



Here’s a **Grafana provisioning bundle** that auto‑loads your dashboards, alerts, and annotations at cluster startup. This way, you don’t need to manually import JSON files — everything is provisioned automatically when Grafana starts.

---

## 📄 `provisioning/dashboards/genai-canary.yaml`

```yaml
apiVersion: 1
providers:
  - name: 'GenAI Canary Rollout'
    orgId: 1
    folder: 'GenAI'
    type: file
    disableDeletion: false
    editable: true
    options:
      path: /etc/grafana/provisioning/dashboards/genai
```

Place your dashboard JSON (e.g., `genai-canary-dashboard.json`) inside `/etc/grafana/provisioning/dashboards/genai`.

---

## 📄 `provisioning/alerting/genai-canary-alerts.yaml`

```yaml
apiVersion: 1
groups:
  - name: GenAI-Canary-Rollout
    interval: 30s
    rules:
      - alert: HighErrorRate
        expr: sum(rate(http_requests_total{status=~"5..",deployment=~"genai-agent-green.*"}[5m])) by (rollout_stage)
              / sum(rate(http_requests_total{deployment=~"genai-agent-green.*"}[5m])) by (rollout_stage) * 100 > 5
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Error rate above 5% during rollout stage {{ $labels.rollout_stage }}"
          description: "Rollback recommended. Error rate is {{ $value }}%."

      - alert: HighLatency
        expr: histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{deployment=~"genai-agent-green.*"}[5m])) by (le, rollout_stage)) > 2
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Latency above 2s (p95) during rollout stage {{ $labels.rollout_stage }}"
          description: "Rollback recommended. Latency is {{ $value }}s."
```

---

## 📄 `provisioning/annotations/genai-canary-annotations.yaml`

```yaml
apiVersion: 1
annotations:
  - name: "GenAI Canary Rollout"
    datasource: Prometheus
    enable: true
    iconColor: "rgba(255, 96, 96, 1)"
    query: kube_deployment_annotations{deployment=~"genai-agent-green.*", rollout_stage=~".*"}
    text: "Rollout Stage: {{ rollout_stage }}"
    tags: ["canary", "rollout"]
```

---

### 🔑 How to Use
1. Place these YAML files under `/etc/grafana/provisioning/` in your Grafana container/pod.  
2. Mount your dashboard JSONs into `/etc/grafana/provisioning/dashboards/genai`.  
3. Restart Grafana → dashboards, alerts, and annotations auto‑load.  
4. Alerts will fire to Slack/Teams/email as configured in Grafana’s **notification channels**.  
5. Annotations appear as vertical markers on graphs during rollout stages.  

---

⚡ With this provisioning bundle, your Grafana setup is **fully automated**: dashboards, alerts, and annotations are consistently loaded at startup, ensuring observability is always in place for your GenAI agent rollouts.  

👉 Do you want me to also add a **Helm chart sidecar config** so Grafana provisioning files are automatically injected into your EKS deployment?




Here’s how you can extend your EKS Helm deployment with a **Grafana sidecar config** so that all your provisioning files (dashboards, alerts, annotations) are automatically injected into Grafana pods. This way, Grafana always starts with your full observability bundle pre‑loaded.

---

## 📄 Helm Chart Values (Grafana Sidecar)

Add this to your Grafana Helm chart `values.yaml`:

```yaml
grafana:
  sidecar:
    dashboards:
      enabled: true
      label: grafana_dashboard
      folder: /etc/grafana/provisioning/dashboards
    alerts:
      enabled: true
      folder: /etc/grafana/provisioning/alerting
    annotations:
      enabled: true
      folder: /etc/grafana/provisioning/annotations
```

---

## 📄 Kubernetes ConfigMap Example

Create ConfigMaps for dashboards, alerts, and annotations. The sidecar will mount them into Grafana automatically.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: genai-canary-dashboard
  labels:
    grafana_dashboard: "1"
data:
  genai-canary-dashboard.json: |
    { ... your dashboard JSON ... }

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: genai-canary-alerts
data:
  genai-canary-alerts.yaml: |
    { ... your alerting rules YAML ... }

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: genai-canary-annotations
data:
  genai-canary-annotations.yaml: |
    { ... your annotations YAML ... }
```

---

### 🔑 How It Works
- **Sidecar containers**: Watch for ConfigMaps with the label `grafana_dashboard` (or mounted folders for alerts/annotations).  
- **Auto‑provisioning**: When Grafana starts, it loads dashboards, alerts, and annotations from these ConfigMaps.  
- **Consistency**: Every Grafana pod in EKS has the same observability setup.  
- **Version control**: You can store these YAMLs in GitHub and let your CI/CD pipeline apply them during rollout.  

---

⚡ With this, your **Grafana observability stack is fully automated**: dashboards, alerts, and annotations are injected into Grafana pods via Helm sidecars, ensuring every rollout has monitoring pre‑configured.

👉 Do you want me to also show you how to **wire this into your GitHub Actions pipeline** so that provisioning ConfigMaps are applied automatically after each deployment?


Here’s how you can **wire Grafana provisioning ConfigMaps into your GitHub Actions pipeline**, so they’re applied automatically after each deployment. This ensures your dashboards, alerts, and annotations are always up to date in EKS.

---

## 📄 Add to GitHub Actions Workflow

Extend your existing `deploy-genai-agent.yaml` with a **post‑deploy monitoring step**:

```yaml
    - name: Apply Grafana Provisioning ConfigMaps
      run: |
        kubectl apply -f k8s/monitoring/genai-canary-dashboard-configmap.yaml -n monitoring
        kubectl apply -f k8s/monitoring/genai-canary-alerts-configmap.yaml -n monitoring
        kubectl apply -f k8s/monitoring/genai-canary-annotations-configmap.yaml -n monitoring
```

---

## 📄 Example ConfigMap Files

**`k8s/monitoring/genai-canary-dashboard-configmap.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: genai-canary-dashboard
  labels:
    grafana_dashboard: "1"
  namespace: monitoring
data:
  genai-canary-dashboard.json: |
    { ... your dashboard JSON ... }
```

**`k8s/monitoring/genai-canary-alerts-configmap.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: genai-canary-alerts
  namespace: monitoring
data:
  genai-canary-alerts.yaml: |
    { ... your alerting rules YAML ... }
```

**`k8s/monitoring/genai-canary-annotations-configmap.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: genai-canary-annotations
  namespace: monitoring
data:
  genai-canary-annotations.yaml: |
    { ... your annotations YAML ... }
```

---

### 🔑 How It Works
- **Pipeline step**: After deploying your app, GitHub Actions applies ConfigMaps to the `monitoring` namespace.  
- **Grafana sidecar**: Watches for ConfigMaps and auto‑loads dashboards, alerts, and annotations.  
- **Consistency**: Every rollout ensures Grafana is updated with the latest observability bundle.  
- **Version control**: ConfigMaps live in your repo (`k8s/monitoring/`), so changes are tracked.  

---

⚡ With this, your CI/CD pipeline not only deploys your GenAI agent but also **refreshes Grafana observability automatically**.  

👉 Do you want me to also show you how to **parameterize environments (dev/staging/prod) in these ConfigMaps**, so each environment gets its own dashboards and alert thresholds?


Here’s how you can **parameterize Grafana provisioning ConfigMaps per environment (dev/staging/prod)** so each environment gets its own dashboards and alert thresholds. This way, your CI/CD pipeline applies the right monitoring bundle depending on the environment being deployed.

---

## 📄 Example ConfigMap with Environment Parameter

**`k8s/monitoring/genai-canary-dashboard-configmap-dev.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: genai-canary-dashboard-dev
  labels:
    grafana_dashboard: "1"
  namespace: monitoring
data:
  genai-canary-dashboard.json: |
    {
      "dashboard": {
        "title": "GenAI Agent Canary - Dev",
        "tags": ["genai", "canary", "dev"],
        "panels": [
          {
            "title": "Error Rate (Dev Threshold)",
            "targets": [
              {
                "expr": "sum(rate(http_requests_total{status=~\"5..\",deployment=~\"genai-agent-green.*\", environment=\"dev\"}[5m])) / sum(rate(http_requests_total{deployment=~\"genai-agent-green.*\", environment=\"dev\"}[5m])) * 100"
              }
            ]
          }
        ]
      }
    }
```

**`k8s/monitoring/genai-canary-alerts-configmap-staging.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: genai-canary-alerts-staging
  namespace: monitoring
data:
  genai-canary-alerts.yaml: |
    apiVersion: 1
    groups:
      - name: GenAI-Canary-Rollout-Staging
        interval: 30s
        rules:
          - alert: HighErrorRate
            expr: sum(rate(http_requests_total{status=~"5..",deployment=~"genai-agent-green.*", environment="staging"}[5m])) / sum(rate(http_requests_total{deployment=~"genai-agent-green.*", environment="staging"}[5m])) * 100 > 3
            for: 2m
            labels:
              severity: warning
            annotations:
              summary: "Error rate above 3% in staging rollout"
```

**`k8s/monitoring/genai-canary-annotations-configmap-prod.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: genai-canary-annotations-prod
  namespace: monitoring
data:
  genai-canary-annotations.yaml: |
    apiVersion: 1
    annotations:
      - name: "GenAI Canary Rollout - Prod"
        datasource: Prometheus
        query: kube_deployment_annotations{deployment=~"genai-agent-green.*", environment="prod", rollout_stage=~".*"}
        text: "Prod Rollout Stage: {{ rollout_stage }}"
        tags: ["canary", "prod"]
```

---

## 📄 GitHub Actions Pipeline Step

Update your workflow to apply the correct ConfigMap based on the matrix environment:

```yaml
    - name: Apply Grafana Provisioning ConfigMaps
      run: |
        kubectl apply -f k8s/monitoring/genai-canary-dashboard-configmap-${{ matrix.environment }}.yaml -n monitoring
        kubectl apply -f k8s/monitoring/genai-canary-alerts-configmap-${{ matrix.environment }}.yaml -n monitoring
        kubectl apply -f k8s/monitoring/genai-canary-annotations-configmap-${{ matrix.environment }}.yaml -n monitoring
```

---

### 🔑 Benefits
- **Environment‑specific dashboards**: Dev, staging, and prod each get tailored panels.  
- **Custom thresholds**: Lower tolerance in staging, stricter thresholds in prod.  
- **Automated pipeline integration**: CI/CD applies the right ConfigMaps per environment.  
- **Consistency**: Monitoring bundle is version‑controlled and environment‑aware.  

---

⚡ With this, your observability stack is **fully parameterized per environment** — dashboards, alerts, and annotations adapt automatically to dev, staging, and prod rollouts.  

👉 Do you want me to also show you how to **template these ConfigMaps with Helm values**, so you don’t need separate YAML files for each environment?




Here’s how you can **template your Grafana ConfigMaps with Helm values**, so you don’t need separate YAML files for dev, staging, and prod. Instead, you’ll maintain a single set of manifests, parameterized by environment.

---

## 📄 Helm Template for Dashboard ConfigMap

**`templates/genai-canary-dashboard-configmap.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: genai-canary-dashboard-{{ .Values.environment }}
  labels:
    grafana_dashboard: "1"
  namespace: monitoring
data:
  genai-canary-dashboard.json: |
    {
      "dashboard": {
        "title": "GenAI Agent Canary - {{ .Values.environment | title }}",
        "tags": ["genai", "canary", "{{ .Values.environment }}"],
        "panels": [
          {
            "title": "Error Rate ({{ .Values.environment | title }} Threshold)",
            "targets": [
              {
                "expr": "sum(rate(http_requests_total{status=~\"5..\",deployment=~\"genai-agent-green.*\", environment=\"{{ .Values.environment }}\"}[5m])) / sum(rate(http_requests_total{deployment=~\"genai-agent-green.*\", environment=\"{{ .Values.environment }}\"}[5m])) * 100"
              }
            ]
          }
        ]
      }
    }
```

---

## 📄 Helm Template for Alerts ConfigMap

**`templates/genai-canary-alerts-configmap.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: genai-canary-alerts-{{ .Values.environment }}
  namespace: monitoring
data:
  genai-canary-alerts.yaml: |
    apiVersion: 1
    groups:
      - name: GenAI-Canary-Rollout-{{ .Values.environment | title }}
        interval: 30s
        rules:
          - alert: HighErrorRate
            expr: sum(rate(http_requests_total{status=~"5..",deployment=~"genai-agent-green.*", environment="{{ .Values.environment }}"}[5m])) / sum(rate(http_requests_total{deployment=~"genai-agent-green.*", environment="{{ .Values.environment }}"}[5m])) * 100 > {{ .Values.alertThresholds.errorRate }}
            for: 2m
            labels:
              severity: {{ .Values.alertThresholds.severity }}
            annotations:
              summary: "Error rate above {{ .Values.alertThresholds.errorRate }}% in {{ .Values.environment }} rollout"
```

---

## 📄 Helm Template for Annotations ConfigMap

**`templates/genai-canary-annotations-configmap.yaml`**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: genai-canary-annotations-{{ .Values.environment }}
  namespace: monitoring
data:
  genai-canary-annotations.yaml: |
    apiVersion: 1
    annotations:
      - name: "GenAI Canary Rollout - {{ .Values.environment | title }}"
        datasource: Prometheus
        query: kube_deployment_annotations{deployment=~"genai-agent-green.*", environment="{{ .Values.environment }}", rollout_stage=~".*"}
        text: "{{ .Values.environment | title }} Rollout Stage: {{ rollout_stage }}"
        tags: ["canary", "{{ .Values.environment }}"]
```

---

## 📄 Values.yaml Example

```yaml
environment: dev

alertThresholds:
  errorRate: 5
  severity: warning
```

For **staging**:
```yaml
environment: staging
alertThresholds:
  errorRate: 3
  severity: warning
```

For **prod**:
```yaml
environment: prod
alertThresholds:
  errorRate: 1
  severity: critical
```

---

### 🔑 Benefits
- **Single source of truth**: One set of Helm templates, parameterized by environment.  
- **Custom thresholds**: Different error rate/latency tolerances per environment.  
- **CI/CD integration**: GitHub Actions can pass `--set environment=prod` during Helm upgrade.  
- **Consistency**: Dashboards, alerts, and annotations adapt automatically.  

---

⚡ With this, your Grafana provisioning is **fully Helm‑templated and environment‑aware**.  

👉 Do you want me to also show you how to **pass environment values from GitHub Actions into Helm** (so your pipeline automatically deploys the right monitoring bundle per matrix environment)?


Here’s how you can **pass environment values from GitHub Actions into Helm**, so your pipeline automatically deploys the right monitoring bundle per matrix environment (dev, staging, prod).

---

## 📄 Update GitHub Actions Workflow

In your `deploy-genai-agent.yaml`, modify the Helm upgrade step to include `--set environment=${{ matrix.environment }}` and thresholds:

```yaml
    - name: Deploy Grafana Provisioning (Helm)
      run: |
        helm upgrade --install genai-monitoring ./charts/genai-monitoring \
          --namespace monitoring --create-namespace \
          --set environment=${{ matrix.environment }} \
          --set alertThresholds.errorRate=$([[ "${{ matrix.environment }}" == "prod" ]] && echo 1 || ([[ "${{ matrix.environment }}" == "staging" ]] && echo 3 || echo 5)) \
          --set alertThresholds.severity=$([[ "${{ matrix.environment }}" == "prod" ]] && echo critical || echo warning)
```

---

## 📄 Helm Chart Values (defaults)

**`charts/genai-monitoring/values.yaml`**
```yaml
environment: dev

alertThresholds:
  errorRate: 5
  severity: warning
```

---

## 📄 Helm Templates (parameterized)

Your ConfigMaps already use `.Values.environment` and `.Values.alertThresholds.*`.  
When GitHub Actions runs, it injects the correct values for each environment.

---

### 🔑 How It Works
- **Matrix environment**: GitHub Actions runs jobs for `dev`, `staging`, `prod`.  
- **Helm values injection**: Pipeline passes `--set environment=...` and thresholds dynamically.  
- **ConfigMaps templated**: Dashboards, alerts, and annotations adapt automatically.  
- **Consistency**: No duplicate YAML files; one Helm chart handles all environments.  

---

⚡ With this, your CI/CD pipeline now **deploys monitoring bundles tailored per environment** — dev gets lenient thresholds, staging moderate, prod strict.  

👉 Next step: Do you want me to also show you how to **add Slack/Teams notification channels in Grafana via Helm values**, so alerts automatically route to the right team per environment?


Here’s how you can extend your Helm chart to **provision Grafana notification channels (Slack/Teams/email)** per environment, so alerts automatically route to the right team depending on whether you’re deploying to dev, staging, or prod.

---

## 📄 Helm Template for Notification Channels

**`templates/genai-grafana-notification-channels.yaml`**
```yaml
apiVersion: 1
notifiers:
  - name: "Slack - {{ .Values.environment | title }}"
    type: slack
    uid: "slack-{{ .Values.environment }}"
    org_id: 1
    is_default: false
    send_reminder: true
    frequency: "1m"
    settings:
      url: {{ .Values.notifications.slackWebhook }}
      recipient: "#alerts-{{ .Values.environment }}"
      mentionUsers: "@devops-team"

  - name: "Teams - {{ .Values.environment | title }}"
    type: teams
    uid: "teams-{{ .Values.environment }}"
    org_id: 1
    is_default: false
    send_reminder: true
    frequency: "1m"
    settings:
      url: {{ .Values.notifications.teamsWebhook }}

  - name: "Email - {{ .Values.environment | title }}"
    type: email
    uid: "email-{{ .Values.environment }}"
    org_id: 1
    is_default: false
    settings:
      addresses: {{ .Values.notifications.emailRecipients | join "," }}
```

---

## 📄 Values.yaml Example

**Dev**
```yaml
environment: dev
notifications:
  slackWebhook: https://hooks.slack.com/services/DEV/XXXX
  teamsWebhook: https://outlook.office.com/webhook/dev/XXXX
  emailRecipients:
    - devops-dev@example.com
```

**Staging**
```yaml
environment: staging
notifications:
  slackWebhook: https://hooks.slack.com/services/STAGING/XXXX
  teamsWebhook: https://outlook.office.com/webhook/staging/XXXX
  emailRecipients:
    - devops-staging@example.com
```

**Prod**
```yaml
environment: prod
notifications:
  slackWebhook: https://hooks.slack.com/services/PROD/XXXX
  teamsWebhook: https://outlook.office.com/webhook/prod/XXXX
  emailRecipients:
    - devops-prod@example.com
    - sre-team@example.com
```

---

## 📄 GitHub Actions Integration

Update your workflow to pass environment‑specific notification values:

```yaml
    - name: Deploy Grafana Monitoring
      run: |
        helm upgrade --install genai-monitoring ./charts/genai-monitoring \
          --namespace monitoring --create-namespace \
          --set environment=${{ matrix.environment }} \
          --set notifications.slackWebhook=${{ secrets.SLACK_WEBHOOK_${{ matrix.environment | upper }} }} \
          --set notifications.teamsWebhook=${{ secrets.TEAMS_WEBHOOK_${{ matrix.environment | upper }} }} \
          --set notifications.emailRecipients={${{ secrets.EMAIL_RECIPIENTS_${{ matrix.environment | upper }} }}}
```

---

### 🔑 Benefits
- **Environment‑aware routing**: Dev alerts go to dev Slack channel, prod alerts go to SRE team.  
- **Multi‑channel notifications**: Slack, Teams, and email supported simultaneously.  
- **Secrets integration**: Webhook URLs and email lists stored securely in GitHub Secrets.  
- **Consistency**: Helm templates ensure all Grafana pods load the correct notification channels.  

---

⚡ With this, your monitoring stack is now **fully automated and environment‑aware**: dashboards, alerts, annotations, and notification channels all adapt to dev, staging, and prod rollouts.  

👉 Do you want me to also show you how to **map specific alert severities (warning vs critical) to different channels** (e.g., warnings → Slack, critical → Teams + email)?


Here’s how you can **map alert severities to different notification channels** in Grafana via Helm, so warnings route to Slack while critical alerts route to Teams + email. This gives you fine‑grained control over who gets notified depending on the severity.

---

## 📄 Helm Template for Notification Routing

**`templates/genai-grafana-notification-routing.yaml`**
```yaml
apiVersion: 1
groups:
  - name: GenAI-Canary-Rollout-{{ .Values.environment | title }}
    interval: 30s
    rules:
      - alert: HighErrorRate
        expr: sum(rate(http_requests_total{status=~"5..",deployment=~"genai-agent-green.*", environment="{{ .Values.environment }}"}[5m])) / sum(rate(http_requests_total{deployment=~"genai-agent-green.*", environment="{{ .Values.environment }}"}[5m])) * 100 > {{ .Values.alertThresholds.errorRate }}
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Error rate above {{ .Values.alertThresholds.errorRate }}% in {{ .Values.environment }} rollout"
        notification_channels:
          - teams-{{ .Values.environment }}
          - email-{{ .Values.environment }}

      - alert: HighLatency
        expr: histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{deployment=~"genai-agent-green.*", environment="{{ .Values.environment }}"}[5m])) by (le, rollout_stage)) > {{ .Values.alertThresholds.latency }}
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Latency above {{ .Values.alertThresholds.latency }}s in {{ .Values.environment }} rollout"
        notification_channels:
          - slack-{{ .Values.environment }}
```

---

## 📄 Values.yaml Example

```yaml
environment: prod

alertThresholds:
  errorRate: 1
  latency: 2
  severity: critical

notifications:
  slackWebhook: https://hooks.slack.com/services/PROD/XXXX
  teamsWebhook: https://outlook.office.com/webhook/prod/XXXX
  emailRecipients:
    - sre-team@example.com
    - devops-prod@example.com
```

---

### 🔑 How It Works
- **Warning alerts (e.g., latency)** → routed to Slack channel for quick visibility.  
- **Critical alerts (e.g., error rate)** → routed to Teams + email for escalation.  
- **Environment‑aware**: Each environment (dev/staging/prod) has its own channels.  
- **Helm templating**: Keeps everything DRY and parameterized.  

---

⚡ With this, your Grafana monitoring stack now has **severity‑based routing**: warnings go to Slack for fast triage, while critical alerts escalate to Teams and email for immediate action.  

👉 Do you want me to also show you how to **add escalation policies** (e.g., if a warning persists for >10 minutes, auto‑escalate it to critical and notify Teams/email)?

