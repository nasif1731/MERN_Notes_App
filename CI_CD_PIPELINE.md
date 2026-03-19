# CI/CD Pipeline Guide

Automated testing, building, and deployment configuration.

---

## Table of Contents

1. [Overview](#overview)
2. [GitHub Actions](#github-actions)
3. [GitLab CI/CD](#gitlab-cicd)
4. [Jenkins](#jenkins)
5. [Docker Registry](#docker-registry)
6. [Kubernetes Deployment](#kubernetes-deployment)
7. [Monitoring & Alerts](#monitoring--alerts)

---

## Overview

This guide covers setting up automated pipelines to:
- Run tests on code changes
- Build Docker images
- Push to container registry
- Deploy to Kubernetes
- Run health checks

---

## GitHub Actions

### Setup

1. Create `.github/workflows/` directory:
```bash
mkdir -p .github/workflows
```

2. Create pipeline files in `.github/workflows/`

---

### CI/CD Pipeline File

Create `.github/workflows/main.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Test backend
  test-backend:
    runs-on: ubuntu-latest
    
    services:
      mongo:
        image: mongo:6
        options: >-
          --health-cmd mongosh
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 27017:27017
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: server/package-lock.json
      
      - name: Install dependencies
        working-directory: ./server
        run: npm ci
      
      - name: Run linter
        working-directory: ./server
        run: npm run lint --if-present
      
      - name: Run tests
        working-directory: ./server
        run: npm test --if-present
        env:
          MONGODB_URI: mongodb://localhost:27017/notes-test

  # Test frontend
  test-frontend:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      
      - name: Install dependencies
        working-directory: ./frontend
        run: npm ci
      
      - name: Run linter
        working-directory: ./frontend
        run: npm run lint --if-present
      
      - name: Build
        working-directory: ./frontend
        run: npm run build

  # Build and push Docker images
  build-and-push:
    needs: [test-backend, test-frontend]
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    permissions:
      contents: read
      packages: write
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      # Build backend
      - name: Build and push backend
        uses: docker/build-push-action@v4
        with:
          context: ./server
          push: true
          tags: |
            ${{ env.REGISTRY }}/mern-notes-app/backend:latest
            ${{ env.REGISTRY }}/mern-notes-app/backend:${{ github.sha }}
          cache-from: type=registry,ref=${{ env.REGISTRY }}/mern-notes-app/backend:latest
          cache-to: type=inline
      
      # Build frontend
      - name: Build and push frontend
        uses: docker/build-push-action@v4
        with:
          context: ./frontend
          push: true
          tags: |
            ${{ env.REGISTRY }}/mern-notes-app/frontend:latest
            ${{ env.REGISTRY }}/mern-notes-app/frontend:${{ github.sha }}
          cache-from: type=registry,ref=${{ env.REGISTRY }}/mern-notes-app/frontend:latest
          cache-to: type=inline

  # Deploy to Kubernetes
  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure kubectl
        run: |
          mkdir -p $HOME/.kube
          echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > $HOME/.kube/config
          chmod 600 $HOME/.kube/config
      
      - name: Update images
        run: |
          kubectl set image deployment/notes-backend \
            notes-backend=${{ env.REGISTRY }}/mern-notes-app/backend:${{ github.sha }} \
            -n default
          
          kubectl set image deployment/notes-frontend \
            notes-frontend=${{ env.REGISTRY }}/mern-notes-app/frontend:${{ github.sha }} \
            -n default
      
      - name: Wait for rollout
        run: |
          kubectl rollout status deployment/notes-backend -n default --timeout=5m
          kubectl rollout status deployment/notes-frontend -n default --timeout=5m
      
      - name: Health check
        run: |
          kubectl get pods -n default
          kubectl get services -n default
```

---

### Setting up Secrets

In GitHub repository:
1. Go to Settings → Secrets and variables → Actions
2. Add secrets:
   - `KUBE_CONFIG`: Base64-encoded kubeconfig file
   - `DOCKER_USERNAME`: Docker registry username
   - `DOCKER_PASSWORD`: Docker registry password

---

### Environment Variables

Create `.env` file (don't commit):
```env
MONGODB_URI=mongodb://localhost:27017/notes
PORT=3000
NODE_ENV=production
```

---

## GitLab CI/CD

### Setup

Create `.gitlab-ci.yml` in project root:

```yaml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_HOST: tcp://docker:2375
  DOCKER_TLS_CERTDIR: ""

# Cache strategy
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - server/node_modules/
    - frontend/node_modules/

# Test backend
test:backend:
  stage: test
  image: node:18-alpine
  services:
    - mongo:6
  before_script:
    - cd server
    - npm install
  script:
    - npm run lint --if-present
    - npm test --if-present
  variables:
    MONGODB_URI: mongodb://mongo:27017/notes-test

# Test frontend
test:frontend:
  stage: test
  image: node:18-alpine
  before_script:
    - cd frontend
    - npm install
  script:
    - npm run lint --if-present
    - npm run build

# Build and push backend
build:backend:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE/backend:$CI_COMMIT_SHA ./server
    - docker push $CI_REGISTRY_IMAGE/backend:$CI_COMMIT_SHA
    - docker tag $CI_REGISTRY_IMAGE/backend:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE/backend:latest
    - docker push $CI_REGISTRY_IMAGE/backend:latest
  only:
    - main

# Build and push frontend
build:frontend:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE/frontend:$CI_COMMIT_SHA ./frontend
    - docker push $CI_REGISTRY_IMAGE/frontend:$CI_COMMIT_SHA
    - docker tag $CI_REGISTRY_IMAGE/frontend:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE/frontend:latest
    - docker push $CI_REGISTRY_IMAGE/frontend:latest
  only:
    - main

# Deploy to staging
deploy:staging:
  stage: deploy
  image: bitnami/kubectl:latest
  before_script:
    - mkdir -p $HOME/.kube
    - echo "$KUBE_CONFIG_STAGING" | base64 -d > $HOME/.kube/config
    - chmod 600 $HOME/.kube/config
  script:
    - kubectl set image deployment/notes-backend notes-backend=$CI_REGISTRY_IMAGE/backend:$CI_COMMIT_SHA -n staging
    - kubectl set image deployment/notes-frontend notes-frontend=$CI_REGISTRY_IMAGE/frontend:$CI_COMMIT_SHA -n staging
    - kubectl rollout status deployment/notes-backend -n staging --timeout=5m
    - kubectl rollout status deployment/notes-frontend -n staging --timeout=5m
  environment:
    name: staging
    url: https://staging.notes-app.com
  only:
    - develop

# Deploy to production
deploy:production:
  stage: deploy
  image: bitnami/kubectl:latest
  before_script:
    - mkdir -p $HOME/.kube
    - echo "$KUBE_CONFIG_PROD" | base64 -d > $HOME/.kube/config
    - chmod 600 $HOME/.kube/config
  script:
    - kubectl set image deployment/notes-backend notes-backend=$CI_REGISTRY_IMAGE/backend:$CI_COMMIT_SHA -n production
    - kubectl set image deployment/notes-frontend notes-frontend=$CI_REGISTRY_IMAGE/frontend:$CI_COMMIT_SHA -n production
    - kubectl rollout status deployment/notes-backend -n production --timeout=5m
    - kubectl rollout status deployment/notes-frontend -n production --timeout=5m
  environment:
    name: production
    url: https://notes-app.com
  when: manual  # Require manual approval
  only:
    - main
```

---

## Jenkins

### Setup

1. Install Jenkins
2. Install plugins:
   - Docker Pipeline
   - Kubernetes CLI
   - Pipeline

### Jenkinsfile

Create `Jenkinsfile` in project root:

```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        IMAGE_NAME = 'mern-notes-app'
        KUBE_CONFIG = credentials('kube-config')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Test Backend') {
            steps {
                dir('server') {
                    sh 'npm install'
                    sh 'npm run lint --if-present'
                    sh 'npm test --if-present'
                }
            }
        }
        
        stage('Test Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }
        
        stage('Build Backend') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh '''
                        docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}/backend:${BUILD_NUMBER} ./server
                        docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}/backend:latest ./server
                    '''
                }
            }
        }
        
        stage('Build Frontend') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh '''
                        docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}/frontend:${BUILD_NUMBER} ./frontend
                        docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}/frontend:latest ./frontend
                    '''
                }
            }
        }
        
        stage('Push Images') {
            when {
                branch 'main'
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh '''
                            docker login -u $USERNAME -p $PASSWORD
                            docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}/backend:${BUILD_NUMBER}
                            docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}/backend:latest
                            docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}/frontend:${BUILD_NUMBER}
                            docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}/frontend:latest
                        '''
                    }
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                script {
                    withCredentials([file(credentialsId: 'kube-config', variable: 'KUBE_CONFIG')]) {
                        sh '''
                            export KUBECONFIG=$KUBE_CONFIG
                            kubectl set image deployment/notes-backend notes-backend=${DOCKER_REGISTRY}/${IMAGE_NAME}/backend:${BUILD_NUMBER} -n staging
                            kubectl set image deployment/notes-frontend notes-frontend=${DOCKER_REGISTRY}/${IMAGE_NAME}/frontend:${BUILD_NUMBER} -n staging
                            kubectl rollout status deployment/notes-backend -n staging
                        '''
                    }
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            input {
                message "Deploy to production?"
                ok "Deploy"
            }
            steps {
                script {
                    withCredentials([file(credentialsId: 'kube-config', variable: 'KUBE_CONFIG')]) {
                        sh '''
                            export KUBECONFIG=$KUBE_CONFIG
                            kubectl set image deployment/notes-backend notes-backend=${DOCKER_REGISTRY}/${IMAGE_NAME}/backend:${BUILD_NUMBER} -n production
                            kubectl set image deployment/notes-frontend notes-frontend=${DOCKER_REGISTRY}/${IMAGE_NAME}/frontend:${BUILD_NUMBER} -n production
                            kubectl rollout status deployment/notes-backend -n production
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        
        failure {
            echo 'Pipeline failed!'
            // Send notification
        }
        
        success {
            echo 'Pipeline succeeded!'
            // Send notification
        }
    }
}
```

---

## Docker Registry

### Docker Hub

1. Create Docker Hub account: https://hub.docker.com

2. Build and push:
```bash
# Tag images
docker tag mern_notes_app-backend:latest username/mern-notes-app:backend-latest
docker tag mern_notes_app-frontend:latest username/mern-notes-app:frontend-latest

# Push
docker push username/mern-notes-app:backend-latest
docker push username/mern-notes-app:frontend-latest
```

### GitHub Container Registry

```bash
# Login
echo $PAT | docker login ghcr.io -u USERNAME --password-stdin

# Tag
docker tag mern_notes_app-backend:latest ghcr.io/username/mern-notes-app:backend-latest

# Push
docker push ghcr.io/username/mern-notes-app:backend-latest
```

### GitLab Container Registry

```bash
# Login
docker login registry.gitlab.com

# Tag
docker tag mern_notes_app-backend:latest registry.gitlab.com/group/project/backend:latest

# Push
docker push registry.gitlab.com/group/project/backend:latest
```

---

## Kubernetes Deployment

### Update k8s manifests for registry

**k8s/backend-deployment.yaml:**
```yaml
spec:
  template:
    spec:
      containers:
      - name: notes-backend
        image: ghcr.io/username/mern-notes-app:backend-latest
        imagePullPolicy: Always  # Always pull latest
        env:
        - name: dbURL
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: connection-string
```

### Create Docker Registry Secret

```bash
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=USERNAME \
  --docker-password=PAT \
  --docker-email=email@example.com
```

Use in deployment:
```yaml
spec:
  imagePullSecrets:
  - name: ghcr-secret
```

---

## Monitoring & Alerts

### Prometheus + Grafana

1. Install Prometheus Operator
2. Create PrometheusRule for alerts

**PrometheusRule.yaml:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: notes-app-alerts
spec:
  groups:
  - name: notes-app.rules
    interval: 30s
    rules:
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
      for: 10m
      annotations:
        summary: "High error rate detected"
    
    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total[15m]) > 0.1
      for: 5m
      annotations:
        summary: "Pod is crash looping"
    
    - alert: HighMemoryUsage
      expr: container_memory_usage_bytes{pod=~"notes-.*"} / 1024^3 > 0.5
      for: 10m
      annotations:
        summary: "High memory usage detected"
```

### Alerts Index

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alertmanager-config
data:
  alertmanager.yml: |
    route:
      receiver: 'default'
    receivers:
    - name: 'default'
      slack_configs:
      - api_url: YOUR_SLACK_WEBHOOK
        channel: '#alerts'
        title: 'Alert: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
```

---

## Testing Strategy

### Backend Tests (Jest)

**server/package.json:**
```json
{
  "scripts": {
    "test": "jest --coverage",
    "test:watch": "jest --watch"
  },
  "devDependencies": {
    "jest": "^29.0.0",
    "supertest": "^6.3.0"
  }
}
```

**server/__tests__/routes.test.js:**
```javascript
const request = require('supertest');
const app = require('../server');

describe('GET /allNotes', () => {
  it('should return all notes', async () => {
    const res = await request(app).get('/allNotes');
    expect(res.status).toBe(200);
    expect(Array.isArray(res.body)).toBe(true);
  });
});

describe('POST /addNote', () => {
  it('should create a note', async () => {
    const res = await request(app)
      .post('/addNote')
      .send({
        title: 'Test',
        content: 'Test content'
      });
    expect(res.status).toBe(201);
    expect(res.body).toHaveProperty('_id');
  });
});
```

### Frontend Tests (Vitest)

**frontend/package.json:**
```json
{
  "scripts": {
    "test": "vitest",
    "test:coverage": "vitest --coverage"
  },
  "devDependencies": {
    "vitest": "^0.34.0",
    "@testing-library/react": "^14.0.0"
  }
}
```

---

## Production Checklist

- [ ] All tests passing
- [ ] Docker images built and tested
- [ ] Images pushed to registry
- [ ] Kubernetes manifests updated with correct image tags
- [ ] Secrets configured (kubeconfig, registry credentials)
- [ ] Health checks configured
- [ ] Resource requests and limits set
- [ ] HPA configured
- [ ] Monitoring and alerts enabled
- [ ] Backup strategy implemented
- [ ] SSL/TLS certificates configured
- [ ] Domain DNS configured
- [ ] Firewall rules verified
- [ ] Runbooks created
- [ ] Team trained

---

## Troubleshooting CI/CD

### Pipeline won't trigger
```bash
# Check branch protection rules
# Verify webhook is configured
# Check pipeline logs
```

### Build fails
```bash
# Check Docker logs
docker build -t test ./server

# Verify all dependencies
npm audit

# Check Node/npm versions
node --version
npm --version
```

### Deployment fails
```bash
# Check Kubernetes status
kubectl get pods
kubectl describe pod <pod-name>

# Check image exists
docker image inspect image:tag

# Verify credentials
kubectl get secrets
```

---

## Resources

- **GitHub Actions**: https://docs.github.com/en/actions
- **GitLab CI/CD**: https://docs.gitlab.com/ee/ci/
- **Jenkins**: https://www.jenkins.io/doc/
- **Docker**: https://docs.docker.com/
- **Kubernetes**: https://kubernetes.io/docs/

---

**Last Updated:** March 19, 2026

Next: Set up your chosen CI/CD platform and test the pipeline!
