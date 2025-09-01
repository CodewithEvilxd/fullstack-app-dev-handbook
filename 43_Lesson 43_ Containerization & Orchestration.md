# Lesson 43: Containerization & Orchestration

## 🎯 **Learning Objectives**
- Master Docker containerization for React Native development
- Implement container orchestration with Kubernetes
- Create scalable deployment pipelines
- Manage containerized microservices
- Optimize container performance and security

## 📚 **Table of Contents**
1. [Docker Fundamentals](#docker-fundamentals)
2. [Containerizing React Native](#containerizing-react-native)
3. [Docker Compose](#docker-compose)
4. [Kubernetes Basics](#kubernetes-basics)
5. [Microservices Deployment](#microservices-deployment)
6. [CI/CD with Containers](#cicd-with-containers)
7. [Monitoring & Logging](#monitoring--logging)
8. [Security Best Practices](#security-best-practices)
9. [Performance Optimization](#performance-optimization)
10. [Practical Examples](#practical-examples)

---

## 🐳 **Docker Fundamentals**

### **Docker Concepts**
- **Images**: Read-only templates containing application code and dependencies
- **Containers**: Running instances of Docker images
- **Dockerfile**: Instructions for building Docker images
- **Registry**: Repository for storing and sharing Docker images
- **Volumes**: Persistent data storage for containers

### **Basic Docker Commands**
```bash
# Build image
docker build -t my-app .

# Run container
docker run -p 3000:3000 my-app

# List containers
docker ps

# Stop container
docker stop container_id

# Remove container
docker rm container_id

# List images
docker images

# Remove image
docker rmi image_id

# View logs
docker logs container_id

# Execute command in container
docker exec -it container_id bash
```

---

## 📱 **Containerizing React Native**

### **React Native Dockerfile**
```dockerfile
# Use Node.js base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Install system dependencies for React Native
RUN apk add --no-cache \
    git \
    curl \
    python3 \
    make \
    g++ \
    && npm install -g @react-native-community/cli

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S reactnative -u 1001

# Change ownership of app directory
RUN chown -R reactnative:nodejs /app
USER reactnative

# Expose port
EXPOSE 8081

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8081/status || exit 1

# Start Metro bundler
CMD ["npx", "react-native", "start", "--host", "0.0.0.0"]
```

### **Multi-stage Dockerfile**
```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install all dependencies (including dev dependencies)
RUN npm ci

# Copy source code
COPY . .

# Build the app
RUN npm run build

# Production stage
FROM node:18-alpine AS production

# Install serve to serve static files
RUN npm install -g serve

WORKDIR /app

# Copy built app from builder stage
COPY --from=builder /app/build ./build

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S reactnative -u 1001

# Change ownership
RUN chown -R reactnative:nodejs /app
USER reactnative

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000 || exit 1

# Start the app
CMD ["serve", "-s", "build", "-l", "3000"]
```

### **Development Dockerfile**
```dockerfile
FROM node:18-alpine

# Install system dependencies
RUN apk add --no-cache \
    git \
    curl \
    python3 \
    make \
    g++ \
    openjdk11 \
    android-tools \
    && npm install -g @react-native-community/cli

# Set environment variables
ENV ANDROID_HOME=/opt/android-sdk
ENV PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools

# Create workspace
WORKDIR /workspace

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy source code
COPY . .

# Expose ports
EXPOSE 8081 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8081/status || exit 1

# Default command
CMD ["npm", "start"]
```

---

## 🐙 **Docker Compose**

### **Development Environment**
```yaml
# docker-compose.yml
version: '3.8'

services:
  # React Native development server
  react-native:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "8081:8081"
      - "3000:3000"
    volumes:
      - .:/workspace
      - /workspace/node_modules
    environment:
      - NODE_ENV=development
    networks:
      - react-native-network

  # Android emulator (optional)
  android-emulator:
    image: us-docker.pkg.dev/android-emulator-268719/images/29-playstore-x64:30.1.2
    ports:
      - "5555:5555"
    environment:
      - ADBKEY="$(cat ~/.android/adbkey)"
    volumes:
      - ~/.android:/root/.android
    networks:
      - react-native-network
    depends_on:
      - react-native

  # Backend API
  api:
    build:
      context: ../backend
      dockerfile: Dockerfile
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:password@db:5432/myapp
    depends_on:
      - db
    networks:
      - react-native-network

  # Database
  db:
    image: postgres:14-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - react-native-network

  # Redis for caching
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - react-native-network

volumes:
  postgres_data:
  redis_data:

networks:
  react-native-network:
    driver: bridge
```

### **Production Environment**
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  # React Native web build
  web:
    build:
      context: .
      dockerfile: Dockerfile.prod
    ports:
      - "80:3000"
    environment:
      - NODE_ENV=production
    networks:
      - production-network

  # API Gateway
  api-gateway:
    build:
      context: ./api-gateway
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - NODE_ENV=production
    depends_on:
      - user-service
      - product-service
      - order-service
    networks:
      - production-network

  # User Service
  user-service:
    build:
      context: ./services/user-service
      dockerfile: Dockerfile
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/userdb
    depends_on:
      - db
    networks:
      - production-network
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure

  # Product Service
  product-service:
    build:
      context: ./services/product-service
      dockerfile: Dockerfile
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/productdb
    depends_on:
      - db
    networks:
      - production-network
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure

  # Order Service
  order-service:
    build:
      context: ./services/order-service
      dockerfile: Dockerfile
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/orderdb
    depends_on:
      - db
    networks:
      - production-network
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure

  # Database
  db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - production-network
    deploy:
      placement:
        constraints:
          - node.role == manager

  # Redis
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    networks:
      - production-network
    deploy:
      replicas: 1

volumes:
  postgres_data:
  redis_data:

networks:
  production-network:
    driver: overlay
```

### **Docker Compose Commands**
```bash
# Start development environment
docker-compose up -d

# Start production environment
docker-compose -f docker-compose.prod.yml up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down

# Rebuild and restart
docker-compose up --build

# Scale services
docker-compose up -d --scale user-service=3

# Execute command in service
docker-compose exec api-gateway bash
```

---

## ☸️ **Kubernetes Basics**

### **Kubernetes Concepts**
- **Pods**: Smallest deployable units containing containers
- **Services**: Network abstraction for pods
- **Deployments**: Manage replica sets and pod updates
- **ConfigMaps**: Store configuration data
- **Secrets**: Store sensitive data
- **Ingress**: Manage external access to services

### **Pod Definition**
```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: react-native-pod
  labels:
    app: react-native
spec:
  containers:
  - name: react-native
    image: myapp/react-native:latest
    ports:
    - containerPort: 8081
    env:
    - name: NODE_ENV
      value: "production"
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "256Mi"
        cpu: "200m"
    livenessProbe:
      httpGet:
        path: /health
        port: 8081
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /ready
        port: 8081
      initialDelaySeconds: 5
      periodSeconds: 5
```

### **Deployment Definition**
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-native-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: react-native
  template:
    metadata:
      labels:
        app: react-native
    spec:
      containers:
      - name: react-native
        image: myapp/react-native:latest
        ports:
        - containerPort: 8081
        env:
        - name: NODE_ENV
          value: "production"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8081
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8081
          initialDelaySeconds: 5
          periodSeconds: 5
      imagePullSecrets:
      - name: registry-secret
```

### **Service Definition**
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: react-native-service
spec:
  selector:
    app: react-native
  ports:
  - port: 80
    targetPort: 8081
    protocol: TCP
  type: LoadBalancer
```

### **ConfigMap & Secret**
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: react-native-config
data:
  API_URL: "https://api.myapp.com"
  LOG_LEVEL: "info"
  MAX_CONNECTIONS: "100"

---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: react-native-secret
type: Opaque
data:
  # Base64 encoded values
  DATABASE_PASSWORD: cGFzc3dvcmQ=
  JWT_SECRET: c2VjcmV0LWtleQ==
  API_KEY: YXBpLWtleQ==
```

### **Ingress Definition**
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: react-native-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - myapp.com
    secretName: react-native-tls
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: react-native-service
            port:
              number: 80
```

---

## 🚀 **Microservices Deployment**

### **Service Mesh with Istio**
```yaml
# istio-gateway.yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: react-native-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: react-native-tls
    hosts:
    - myapp.com

---
# istio-virtualservice.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: react-native-virtualservice
spec:
  hosts:
  - myapp.com
  gateways:
  - react-native-gateway
  http:
  - match:
    - uri:
        prefix: "/api"
    route:
    - destination:
        host: api-gateway
  - match:
    - uri:
        prefix: "/"
    route:
    - destination:
        host: react-native-web
```

### **Helm Chart Structure**
```
myapp/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   └── _helpers.tpl
└── charts/
    ├── api-gateway/
    ├── user-service/
    ├── product-service/
    └── order-service/
```

### **Helm Chart.yaml**
```yaml
# Chart.yaml
apiVersion: v2
name: myapp
description: A Helm chart for MyApp
type: application
version: 0.1.0
appVersion: "1.0.0"
dependencies:
  - name: postgresql
    version: "12.1.0"
    repository: "https://charts.bitnami.com/bitnami"
  - name: redis
    version: "17.3.0"
    repository: "https://charts.bitnami.com/bitnami"
```

### **Helm Values**
```yaml
# values.yaml
replicaCount: 2

image:
  repository: myapp/react-native
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  hosts:
    - host: myapp.com
      paths:
        - path: /
          pathType: Prefix

config:
  apiUrl: "https://api.myapp.com"
  logLevel: "info"

secrets:
  databasePassword: "password"
  jwtSecret: "secret-key"
  apiKey: "api-key"

postgresql:
  enabled: true
  postgresqlUsername: myapp
  postgresqlPassword: password
  postgresqlDatabase: myapp

redis:
  enabled: true
  password: password
```

---

## 🔄 **CI/CD with Containers**

### **GitHub Actions CI/CD**
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linting
        run: npm run lint

      - name: Run tests
        run: npm run test -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3

  build:
    needs: test
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: myapp/react-native:latest,myapp/react-native:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'

    steps:
      - uses: actions/checkout@v3

      - name: Deploy to staging
        run: |
          kubectl config set-context staging
          kubectl set image deployment/react-native react-native=myapp/react-native:${{ github.sha }}
          kubectl rollout status deployment/react-native

  deploy-production:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3

      - name: Deploy to production
        run: |
          kubectl config set-context production
          kubectl set image deployment/react-native react-native=myapp/react-native:${{ github.sha }}
          kubectl rollout status deployment/react-native
```

### **Jenkins Pipeline**
```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'myapp/react-native'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh 'npm ci'
                sh 'npm run lint'
                sh 'npm run test -- --coverage'
            }
            post {
                always {
                    publishCoverage adapters: [coberturaAdapter('coverage/cobertura-coverage.xml')]
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh 'kubectl config use-context staging'
                sh "kubectl set image deployment/react-native react-native=${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh 'kubectl rollout status deployment/react-native'
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Deploy to production?', ok: 'Deploy'
                }
                sh 'kubectl config use-context production'
                sh "kubectl set image deployment/react-native react-native=${DOCKER_IMAGE}:${DOCKER_TAG}"
                sh 'kubectl rollout status deployment/react-native'
            }
        }
    }

    post {
        always {
            sh 'docker system prune -f'
        }
        success {
            slackSend color: 'good', message: "Build ${env.BUILD_NUMBER} succeeded!"
        }
        failure {
            slackSend color: 'danger', message: "Build ${env.BUILD_NUMBER} failed!"
        }
    }
}
```

---

## 📊 **Monitoring & Logging**

### **Container Monitoring**
```yaml
# prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s

    rule_files:
      # - "first_rules.yml"
      # - "second_rules.yml"

    scrape_configs:
      - job_name: 'react-native'
        static_configs:
          - targets: ['react-native-service:8081']
        metrics_path: '/metrics'
        scrape_interval: 5s

      - job_name: 'api-gateway'
        static_configs:
          - targets: ['api-gateway:8080']
        metrics_path: '/metrics'
        scrape_interval: 5s

      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
            action: keep
            regex: true
          - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
            action: replace
            target_label: __metrics_path__
            regex: (.+)
          - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
            action: replace
            regex: ([^:]+)(?::\d+)?;(\d+)
            replacement: $1:$2
            target_label: __address__
          - action: labelmap
            regex: __meta_kubernetes_pod_label_(.+)
          - source_labels: [__meta_kubernetes_namespace]
            action: replace
            target_label: kubernetes_namespace
          - source_labels: [__meta_kubernetes_pod_name]
            action: replace
            target_label: kubernetes_pod_name
```

### **Application Metrics**
```javascript
// services/metrics.js
import client from 'prom-client';

// Create a Registry which registers the metrics
const register = new client.Registry();

// Add a default label which is added to all metrics
register.setDefaultLabels({
  app: 'react-native',
});

// Create metrics
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5, 10],
});

const activeConnections = new client.Gauge({
  name: 'active_connections',
  help: 'Number of active connections',
});

const errorCounter = new client.Counter({
  name: 'errors_total',
  help: 'Total number of errors',
  labelNames: ['type', 'service'],
});

const userActivity = new client.Counter({
  name: 'user_activity_total',
  help: 'Total user activities',
  labelNames: ['activity_type', 'user_id'],
});

// Register metrics
register.registerMetric(httpRequestDuration);
register.registerMetric(activeConnections);
register.registerMetric(errorCounter);
register.registerMetric(userActivity);

// Middleware to collect HTTP metrics
const metricsMiddleware = (req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;

    httpRequestDuration
      .labels(req.method, req.route?.path || req.path, res.statusCode.toString())
      .observe(duration);
  });

  next();
};

// Metrics endpoint
const metricsEndpoint = async (req, res) => {
  try {
    res.set('Content-Type', register.contentType);
    res.end(await register.metrics());
  } catch (error) {
    res.status(500).end(error);
  }
};

// Utility functions
const recordError = (type, service = 'unknown') => {
  errorCounter.labels(type, service).inc();
};

const recordUserActivity = (activityType, userId) => {
  userActivity.labels(activityType, userId).inc();
};

const updateActiveConnections = (count) => {
  activeConnections.set(count);
};

export {
  metricsMiddleware,
  metricsEndpoint,
  recordError,
  recordUserActivity,
  updateActiveConnections,
};
```

### **Centralized Logging**
```yaml
# fluent-bit-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         5
        Log_Level     info
        Daemon        off

    [INPUT]
        Name              tail
        Path              /var/log/containers/*react-native*.log
        Parser            docker
        Tag               app.*
        Refresh_Interval  5

    [FILTER]
        Name                kubernetes
        Match               app.*
        Kube_URL           https://kubernetes.default.svc:443
        Kube_CA_File       /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        Kube_Token_File    /var/run/secrets/kubernetes.io/serviceaccount/token

    [OUTPUT]
        Name  es
        Match *
        Host  elasticsearch
        Port  9200
        Index react_native_logs
        Type  app_logs
```

---

## 🔒 **Security Best Practices**

### **Container Security**
```dockerfile
# Secure Dockerfile
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 -S appuser && \
    adduser -S -D -H -u 1001 -h /app -s /sbin/nologin -G appuser -g appuser appuser

# Install dependencies
RUN apk add --no-cache --virtual .build-deps \
    python3 \
    make \
    g++ \
    git \
    && apk add --no-cache \
    curl \
    && rm -rf /var/cache/apk/*

WORKDIR /app

# Copy package files first for better caching
COPY --chown=appuser:appuser package*.json ./

# Install dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy application code
COPY --chown=appuser:appuser . .

# Remove unnecessary files
RUN rm -rf /tmp/* /var/tmp/* && \
    rm -rf .git .gitignore .gitattributes && \
    rm -rf tests/ *.test.js *.spec.js

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8081/health || exit 1

EXPOSE 8081

CMD ["npm", "start"]
```

### **Security Scanning**
```yaml
# security-scan.yml
apiVersion: batch/v1
kind: Job
metadata:
  name: security-scan
spec:
  template:
    spec:
      containers:
      - name: trivy
        image: aquasecurity/trivy:latest
        command:
        - trivy
        - image
        - --exit-code
        - "1"
        - --no-progress
        - myapp/react-native:latest
        volumeMounts:
        - name: docker-socket
          mountPath: /var/run/docker.sock
      volumes:
      - name: docker-socket
        hostPath:
          path: /var/run/docker.sock
      restartPolicy: Never
```

### **Network Policies**
```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: react-native-network-policy
spec:
  podSelector:
    matchLabels:
      app: react-native
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api-gateway
    ports:
    - protocol: TCP
      port: 8081
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - podSelector:
        matchLabels:
          app: redis
    ports:
    - protocol: TCP
      port: 6379
  - to: []
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
```

---

## ⚡ **Performance Optimization**

### **Multi-stage Builds**
```dockerfile
# Optimized Dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package.json package-lock.json* ./
RUN \
  if [ -f package-lock.json ]; then npm ci --only=production; \
  else echo "Lockfile not found." && exit 1; \
  fi

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

RUN npm run build

FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 reactnative

COPY --from=builder /app/public ./public
COPY --from=builder --chown=reactnative:nodejs /app/.next/standalone ./
COPY --from=builder --chown=reactnative:nodejs /app/.next/static ./.next/static

USER reactnative

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

### **Container Resource Limits**
```yaml
# resource-limits.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: react-native-limits
spec:
  limits:
  - default:
      memory: 512Mi
      cpu: 500m
    defaultRequest:
      memory: 256Mi
      cpu: 250m
    type: Container

---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: react-native-quota
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
    pods: "10"
```

### **Horizontal Pod Autoscaling**
```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: react-native-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: react-native-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Max
```

---

## 🎯 **Practical Examples**

### **Complete Docker Setup**
```dockerfile
# Dockerfile
FROM node:18-alpine AS base

# Install system dependencies
RUN apk add --no-cache \
    git \
    curl \
    python3 \
    make \
    g++ \
    && npm install -g @react-native-community/cli

# Create app directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && npm cache clean --force

# Copy source code
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S reactnative -u 1001 && \
    chown -R reactnative:nodejs /app

USER reactnative

# Expose port
EXPOSE 8081

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8081/status || exit 1

# Start the app
CMD ["npx", "react-native", "start", "--host", "0.0.0.0", "--port", "8081"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  react-native:
    build: .
    ports:
      - "8081:8081"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
    networks:
      - react-native-network

  api:
    image: node:18-alpine
    working_dir: /api
    ports:
      - "3000:3000"
    volumes:
      - ./api:/api
    command: npm run dev
    depends_on:
      - db
    networks:
      - react-native-network

  db:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: react_native_dev
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - react-native-network

volumes:
  postgres_data:

networks:
  react-native-network:
    driver: bridge
```

### **Kubernetes Deployment**
```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-native-deployment
  labels:
    app: react-native
spec:
  replicas: 3
  selector:
    matchLabels:
      app: react-native
  template:
    metadata:
      labels:
        app: react-native
    spec:
      containers:
      - name: react-native
        image: myapp/react-native:latest
        ports:
        - containerPort: 8081
        env:
        - name: NODE_ENV
          value: "production"
        - name: API_URL
          valueFrom:
            configMapKeyRef:
              name: react-native-config
              key: API_URL
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8081
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8081
          initialDelaySeconds: 5
          periodSeconds: 5
      imagePullSecrets:
      - name: docker-registry-secret

---
apiVersion: v1
kind: Service
metadata:
  name: react-native-service
spec:
  selector:
    app: react-native
  ports:
  - port: 80
    targetPort: 8081
  type: LoadBalancer

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: react-native-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: react-native-service
            port:
              number: 80
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Docker Fundamentals**: Containerization basics and commands
- ✅ **Containerizing React Native**: Creating Dockerfiles for RN apps
- ✅ **Docker Compose**: Multi-container development environments
- ✅ **Kubernetes Basics**: Container orchestration concepts
- ✅ **Microservices Deployment**: Deploying distributed systems
- ✅ **CI/CD with Containers**: Automated deployment pipelines
- ✅ **Monitoring & Logging**: Container and application monitoring
- ✅ **Security Best Practices**: Container security and scanning
- ✅ **Performance Optimization**: Resource limits and autoscaling
- ✅ **Practical Examples**: Complete container setups

### **Best Practices**
1. **Use multi-stage builds** to reduce image size
2. **Implement proper resource limits** to prevent resource exhaustion
3. **Use health checks** to ensure container reliability
4. **Implement proper logging** for debugging and monitoring
5. **Use secrets management** for sensitive configuration
6. **Implement rolling updates** for zero-downtime deployments
7. **Use container registries** for image management
8. **Implement proper networking** between containers
9. **Use orchestration tools** for production deployments
10. **Monitor container metrics** for performance optimization

### **Next Steps**
- Learn advanced Kubernetes features
- Implement service mesh patterns
- Explore container security scanning
- Set up automated testing in containers
- Implement canary deployments
- Learn about container orchestration alternatives

---

## 🎯 **Assignment**

### **Task 1: Docker Setup**
Create a complete Docker setup for a React Native project with:
- Multi-stage Dockerfile for production builds
- Docker Compose for development environment
- Volume mounts for hot reloading
- Environment-specific configurations
- Health checks and monitoring

### **Task 2: Kubernetes Deployment**
Implement a Kubernetes deployment for a microservices architecture:
- Deployments for each microservice
- Services for internal communication
- Ingress for external access
- ConfigMaps and Secrets for configuration
- Horizontal Pod Autoscaling

### **Task 3: CI/CD Pipeline**
Build a complete CI/CD pipeline with:
- Automated testing in containers
- Docker image building and pushing
- Kubernetes deployments
- Rollback strategies
- Monitoring and alerting
- Security scanning integration

---

## 📚 **Additional Resources**
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Helm Charts](https://helm.sh/docs/)
- [Istio Service Mesh](https://istio.io/)
- [Prometheus Monitoring](https://prometheus.io/)
- [ELK Stack](https://www.elastic.co/elastic-stack)
- [Container Security](https://www.docker.com/security)

---

**Next Lesson**: [Lesson 44: Cloud Deployment & Scaling](Lesson%2044_%20Cloud%20Deployment%20&%20Scaling.md)