

---

# 🚀 Hybrid Cloud CI/CD Platform with OpenShift and GitHub Actions

---

## 📋 Project Objective

Build a **Hybrid Cloud** CI/CD setup to deploy applications across:
- **OpenShift Sandbox** → Staging environment
- **AWS EKS** → Production environment
- **GitHub Actions** → Automate build, scan, and deploy pipelines
- Integrate **container scanning** (Trivy)
- Configure **Networking** and **Storage** across platforms

---

## 🛠️ Technologies Used

- **OpenShift**
- **AWS EKS**
- **GitHub Actions**
- **Docker / DockerHub**
- **Helm**
- **Trivy (Image Scanning)**

---

## 📁 Project Structure

```
hybrid-cicd-platform/
├── app/
│   ├── Dockerfile
│   ├── app.js
│   ├── package.json
├── helm/
│   ├── Chart.yaml
│   ├── values.yaml
│   └── templates/
│       ├── deployment.yaml
│       └── service.yaml
├── .github/
│   └── workflows/
│       └── ci-cd-pipeline.yml
├── openshift/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── route.yaml
├── eks/
│   └── deployment.yaml
├── README.md
```
---

**`setup_hybrid_cicd.py`**

```python
import os

# Base directory
base_dir = "/home/lilia/VIDEOS/hybrid-cicd-platform"

# Define all file paths and their content
files_content = {
    "app/Dockerfile": """FROM node:18
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
""",
    "app/app.js": """const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('Hello Hybrid Cloud!'));
app.listen(3000, () => console.log('App listening on port 3000'));
""",
    "app/package.json": """{
  "name": "hybrid-cloud-app",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.17.1"
  }
}
""",
    "helm/Chart.yaml": """apiVersion: v2
name: hybrid-cloud-app
version: 0.1.0
""",
    "helm/values.yaml": """replicaCount: 2
image:
  repository: your-dockerhub-username/hybrid-cloud-app
  tag: latest
  pullPolicy: IfNotPresent
service:
  type: LoadBalancer
  port: 80
  targetPort: 3000
""",
    "helm/templates/deployment.yaml": """apiVersion: apps/v1
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
      - name: app
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: {{ .Values.service.targetPort }}
""",
    "helm/templates/service.yaml": """apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Release.Name }}
  ports:
  - port: {{ .Values.service.port }}
    targetPort: {{ .Values.service.targetPort }}
""",
    ".github/workflows/ci-cd-pipeline.yml": """name: Hybrid Cloud CI/CD Pipeline

on:
  push:
    branches: [ main ]

jobs:
  build-scan-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Docker
        uses: docker/setup-buildx-action@v2

      - name: Log in to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and Push Docker Image
        run: |
          docker build -t your-dockerhub-username/hybrid-cloud-app:latest .
          docker push your-dockerhub-username/hybrid-cloud-app:latest

      - name: Scan Docker Image with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: your-dockerhub-username/hybrid-cloud-app:latest

  deploy-openshift:
    runs-on: ubuntu-latest
    needs: build-scan-push
    steps:
      - name: Deploy to OpenShift
        run: |
          oc login https://api.sandbox-m2.ll9k.p1.openshiftapps.com:6443 --token=${{ secrets.OPENSHIFT_TOKEN }}
          oc project your-project-name
          oc apply -f openshift/deployment.yaml
          oc apply -f openshift/service.yaml
          oc apply -f openshift/route.yaml

  deploy-eks:
    runs-on: ubuntu-latest
    needs: build-scan-push
    steps:
      - name: Set up kubeconfig
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBECONFIG_EKS }}" > ~/.kube/config

      - name: Deploy to EKS
        run: |
          helm upgrade --install hybrid-cloud-app ./helm
""",
    "openshift/deployment.yaml": """apiVersion: apps/v1
kind: Deployment
metadata:
  name: hybrid-cloud-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hybrid-cloud-app
  template:
    metadata:
      labels:
        app: hybrid-cloud-app
    spec:
      containers:
      - name: app
        image: your-dockerhub-username/hybrid-cloud-app:latest
        ports:
        - containerPort: 3000
""",
    "openshift/service.yaml": """apiVersion: v1
kind: Service
metadata:
  name: hybrid-cloud-service
spec:
  selector:
    app: hybrid-cloud-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
""",
    "openshift/route.yaml": """apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: hybrid-cloud-route
spec:
  to:
    kind: Service
    name: hybrid-cloud-service
  port:
    targetPort: 3000
""",
    "eks/deployment.yaml": """apiVersion: apps/v1
kind: Deployment
metadata:
  name: hybrid-cloud-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hybrid-cloud-app
  template:
    metadata:
      labels:
        app: hybrid-cloud-app
    spec:
      containers:
      - name: app
        image: your-dockerhub-username/hybrid-cloud-app:latest
        ports:
        - containerPort: 3000
"""
}

# Create all directories and files
for file_path, content in files_content.items():
    full_path = os.path.join(base_dir, file_path)
    os.makedirs(os.path.dirname(full_path), exist_ok=True)
    with open(full_path, "w") as f:
        f.write(content)

print(f"✅ All files and folders created successfully under {base_dir}")


```


---

## ✅ Step-by-Step Implementation

### 1. Create the Node.js Application

**`app/app.js`**
```javascript
const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('Hello Hybrid Cloud!'));
app.listen(3000, () => console.log('App listening on port 3000'));
```

**`app/package.json`**
```json
{
  "name": "hybrid-cloud-app",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

---

### 2. Dockerize the Application

**`app/Dockerfile`**
```dockerfile
FROM node:18
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
```

---

### 3. Create Helm Chart for Kubernetes Deployments

**`helm/Chart.yaml`**
```yaml
apiVersion: v2
name: hybrid-cloud-app
version: 0.1.0
```

**`helm/values.yaml`**
```yaml
replicaCount: 2
image:
  repository: your-dockerhub-username/hybrid-cloud-app
  tag: latest
  pullPolicy: IfNotPresent
service:
  type: LoadBalancer
  port: 80
  targetPort: 3000
```

**`helm/templates/deployment.yaml`**
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
      - name: app
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: {{ .Values.service.targetPort }}
```

**`helm/templates/service.yaml`**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ .Release.Name }}
  ports:
  - port: {{ .Values.service.port }}
    targetPort: {{ .Values.service.targetPort }}
```

---

### 4. Create OpenShift Manifests for Staging Deployment

**`openshift/deployment.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hybrid-cloud-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hybrid-cloud-app
  template:
    metadata:
      labels:
        app: hybrid-cloud-app
    spec:
      containers:
      - name: app
        image: your-dockerhub-username/hybrid-cloud-app:latest
        ports:
        - containerPort: 3000
```

**`openshift/service.yaml`**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hybrid-cloud-service
spec:
  selector:
    app: hybrid-cloud-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
```

**`openshift/route.yaml`**
```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: hybrid-cloud-route
spec:
  to:
    kind: Service
    name: hybrid-cloud-service
  port:
    targetPort: 3000
```

---

### 5. Set Up GitHub Actions for CI/CD

**`.github/workflows/ci-cd-pipeline.yml`**
```yaml
name: Hybrid Cloud CI/CD Pipeline

on:
  push:
    branches: [ main ]

jobs:
  build-scan-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Docker
        uses: docker/setup-buildx-action@v2

      - name: Log in to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and Push Docker Image
        run: |
          docker build -t your-dockerhub-username/hybrid-cloud-app:latest .
          docker push your-dockerhub-username/hybrid-cloud-app:latest

      - name: Scan Docker Image with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: your-dockerhub-username/hybrid-cloud-app:latest

  deploy-openshift:
    runs-on: ubuntu-latest
    needs: build-scan-push
    steps:
      - name: Deploy to OpenShift
        run: |
          oc login https://api.sandbox-m2.ll9k.p1.openshiftapps.com:6443 --token=${{ secrets.OPENSHIFT_TOKEN }}
          oc project your-project-name
          oc apply -f openshift/deployment.yaml
          oc apply -f openshift/service.yaml
          oc apply -f openshift/route.yaml

  deploy-eks:
    runs-on: ubuntu-latest
    needs: build-scan-push
    steps:
      - name: Set up kubeconfig
        run: |
          mkdir -p ~/.kube
          echo "${{ secrets.KUBECONFIG_EKS }}" > ~/.kube/config

      - name: Deploy to EKS
        run: |
          helm upgrade --install hybrid-cloud-app ./helm
```

---

### 6. GitHub Secrets Needed

| Secret Name | Purpose |
|:------------|:--------|
| `DOCKER_USERNAME` | DockerHub Username |
| `DOCKER_PASSWORD` | DockerHub Password |
| `OPENSHIFT_TOKEN` | OpenShift login token |
| `KUBECONFIG_EKS` | Kubeconfig content for EKS |

Add them in GitHub → Repository → Settings → Secrets → Actions.

---

## ⚙️ Useful CLI Commands

| Command | Purpose |
|:--------|:--------|
| `oc login` | Log into OpenShift cluster |
| `oc apply -f <file.yaml>` | Apply Kubernetes resources |
| `docker build -t <image>` | Build Docker image |
| `docker push <image>` | Push Docker image to registry |
| `trivy image <image>` | Scan image for vulnerabilities |
| `helm upgrade --install <release-name> ./helm` | Deploy app to Kubernetes |

---

## 📸 Deployment Flow

```plaintext
Developer Pushes Code ➔ GitHub Actions ➔
  ➔ Build + Push Docker Image
  ➔ Trivy Scan
  ➔ Deploy to OpenShift (Staging)
  ➔ Deploy to AWS EKS (Production)
```

---

## 🔥 Highlights

- Hybrid Deployment across OpenShift and EKS
- Automated CI/CD with GitHub Actions
- Integrated security scanning with Trivy
- Helm and Kubernetes best practices
- Scalable and portable solution

---

# 🏁 Conclusion

This project builds a **modern, secure, and scalable** hybrid cloud application pipeline leveraging **OpenShift**, **AWS EKS**, **GitHub Actions**, **Trivy**, and **Helm**.

---

