# Lesson 28: Deployment & DevOps

## 🎯 **Learning Objectives**
- Master application deployment strategies for React Native and Node.js
- Set up CI/CD pipelines for automated testing and deployment
- Configure production environments and monitoring
- Implement containerization with Docker
- Manage infrastructure as code

## 📚 **Table of Contents**
1. [Deployment Strategies](#deployment-strategies)
2. [React Native App Deployment](#react-native-app-deployment)
3. [Backend Deployment](#backend-deployment)
4. [CI/CD Pipeline Setup](#cicd-pipeline-setup)
5. [Docker Containerization](#docker-containerization)
6. [Infrastructure as Code](#infrastructure-as-code)
7. [Monitoring & Logging](#monitoring--logging)
8. [Security in Production](#security-in-production)
9. [Performance Optimization](#performance-optimization)
10. [Practical Examples](#practical-examples)

---

## 🚀 **Deployment Strategies**

### **Deployment Types**
```javascript
// Blue-Green Deployment Strategy
const blueGreenDeployment = {
  environments: {
    blue: {
      version: '1.0.0',
      status: 'active',
      traffic: 100,
    },
    green: {
      version: '1.1.0',
      status: 'standby',
      traffic: 0,
    },
  },

  switchTraffic: function(target) {
    if (target === 'green') {
      this.environments.blue.traffic = 0;
      this.environments.green.traffic = 100;
      this.environments.blue.status = 'standby';
      this.environments.green.status = 'active';
    } else {
      this.environments.green.traffic = 0;
      this.environments.blue.traffic = 100;
      this.environments.green.status = 'standby';
      this.environments.blue.status = 'active';
    }
  },

  rollback: function() {
    this.switchTraffic('blue');
  },
};

// Rolling Deployment Strategy
const rollingDeployment = {
  instances: [
    { id: 1, version: '1.0.0', status: 'active' },
    { id: 2, version: '1.0.0', status: 'active' },
    { id: 3, version: '1.0.0', status: 'active' },
    { id: 4, version: '1.1.0', status: 'updating' },
    { id: 5, version: '1.1.0', status: 'updating' },
  ],

  updateInstance: function(instanceId, newVersion) {
    const instance = this.instances.find(i => i.id === instanceId);
    if (instance) {
      instance.version = newVersion;
      instance.status = 'active';
    }
  },

  checkDeploymentProgress: function() {
    const total = this.instances.length;
    const updated = this.instances.filter(i => i.version === '1.1.0').length;
    return {
      progress: (updated / total) * 100,
      completed: updated === total,
    };
  },
};

// Canary Deployment Strategy
const canaryDeployment = {
  traffic: {
    stable: 90,    // 90% of traffic
    canary: 10,    // 10% of traffic to new version
  },

  versions: {
    stable: '1.0.0',
    canary: '1.1.0',
  },

  increaseCanaryTraffic: function(percentage) {
    const increase = Math.min(percentage, this.traffic.stable);
    this.traffic.canary += increase;
    this.traffic.stable -= increase;
  },

  promoteCanary: function() {
    this.versions.stable = this.versions.canary;
    this.traffic = { stable: 100, canary: 0 };
  },

  rollbackCanary: function() {
    this.traffic = { stable: 100, canary: 0 };
  },
};
```

---

## 📱 **React Native App Deployment**

### **iOS App Store Deployment**
```bash
# 1. Build for iOS
cd ios
pod install

# 2. Create production build
react-native run-ios --configuration Release

# 3. Archive the app
xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release -archivePath ./build/MyApp.xcarchive archive

# 4. Export IPA
xcodebuild -exportArchive -archivePath ./build/MyApp.xcarchive -exportOptionsPlist exportOptions.plist -exportPath ./build

# 5. Upload to App Store Connect
xcrun altool --upload-app -f ./build/MyApp.ipa -u "your-apple-id" -p "app-specific-password"
```

### **Android Play Store Deployment**
```bash
# 1. Generate signed APK/AAB
cd android

# Create key store if not exists
keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000

# 2. Configure gradle for signing
# android/app/build.gradle
android {
  signingConfigs {
    release {
      storeFile file('my-release-key.keystore')
      storePassword 'your-store-password'
      keyAlias 'my-key-alias'
      keyPassword 'your-key-password'
    }
  }

  buildTypes {
    release {
      signingConfig signingConfigs.release
      minifyEnabled true
      proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
  }
}

# 3. Build release APK
./gradlew assembleRelease

# 4. Build Android App Bundle (AAB)
./gradlew bundleRelease

# 5. Upload to Play Console
# Use Google Play Console web interface or fastlane
```

### **CodePush for Over-the-Air Updates**
```javascript
// Install CodePush
npm install react-native-code-push

// Configure in MainApplication.java (Android)
import com.microsoft.codepush.react.CodePush;

@Override
protected List<ReactPackage> getPackages() {
  return Arrays.<ReactPackage>asList(
    new MainReactPackage(),
    CodePush.getJSBundleFile(), // Add this line
  );
}

// Configure in AppDelegate.m (iOS)
#import <CodePush/CodePush.h>

- (NSURL *)sourceURLForBridge:(RCTBridge *)bridge
{
  return [CodePush bundleURL];
}

// Configure in App.js
import CodePush from 'react-native-code-push';

const codePushOptions = {
  checkFrequency: CodePush.CheckFrequency.ON_APP_RESUME,
  installMode: CodePush.InstallMode.IMMEDIATE,
  mandatoryInstallMode: CodePush.InstallMode.IMMEDIATE,
};

const App = () => {
  // Your app code
};

export default CodePush(codePushOptions)(App);

// Release update
code-push release-react MyApp ios
code-push release-react MyApp android

// Check for updates programmatically
CodePush.sync({
  updateDialog: true,
  installMode: CodePush.InstallMode.IMMEDIATE,
});
```

---

## 🖥️ **Backend Deployment**

### **Node.js Production Deployment**
```javascript
// server.js - Production server setup
const express = require('express');
const helmet = require('helmet');
const compression = require('compression');
const rateLimit = require('express-rate-limit');
const cors = require('cors');

const app = express();

// Security middleware
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", 'data:', 'https:'],
    },
  },
}));

// Compression middleware
app.use(compression());

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP, please try again later.',
});
app.use('/api/', limiter);

// CORS configuration
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
  credentials: true,
}));

// Body parsing
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({
    status: 'OK',
    uptime: process.uptime(),
    timestamp: new Date().toISOString(),
    version: process.env.npm_package_version,
  });
});

// API routes
app.use('/api', require('./routes'));

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);

  res.status(err.status || 500).json({
    success: false,
    error: process.env.NODE_ENV === 'production' ? 'Something went wrong!' : err.message,
  });
});

// 404 handler
app.use('*', (req, res) => {
  res.status(404).json({
    success: false,
    error: 'Route not found',
  });
});

const PORT = process.env.PORT || 3000;

if (require.main === module) {
  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT} in ${process.env.NODE_ENV} mode`);
  });
}

module.exports = app;
```

### **PM2 Process Manager**
```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'myapp',
    script: 'server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'development',
      PORT: 3000,
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000,
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_file: './logs/combined.log',
    time: true,
    watch: false,
    max_memory_restart: '1G',
    restart_delay: 4000,
    autorestart: true,
    min_uptime: '10s',
  }],

  deploy: {
    production: {
      user: 'node',
      host: 'your-server.com',
      ref: 'origin/master',
      repo: 'git@github.com:yourusername/yourrepo.git',
      path: '/var/www/production',
      'pre-deploy-local': '',
      'post-deploy': 'npm install && pm2 reload ecosystem.config.js --env production',
      'pre-setup': '',
    },
  },
};

// PM2 commands
// Start application
pm2 start ecosystem.config.js

// Start in production mode
pm2 start ecosystem.config.js --env production

// Monitor processes
pm2 monit

// View logs
pm2 logs

// Restart application
pm2 restart myapp

// Stop application
pm2 stop myapp

// Delete application
pm2 delete myapp

// Save current process list
pm2 save

// Generate startup script
pm2 startup
```

---

## 🔄 **CI/CD Pipeline Setup**

### **GitHub Actions CI/CD**
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x, 18.x]

    steps:
    - uses: actions/checkout@v3

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linting
      run: npm run lint

    - name: Run tests
      run: npm test -- --coverage

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info

  build-and-deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
    - uses: actions/checkout@v3

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build and push Docker image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: myapp
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

    - name: Deploy to ECS
      run: |
        aws ecs update-service \
          --cluster myapp-cluster \
          --service myapp-service \
          --force-new-deployment \
          --query 'service.deployments[0].id'

  mobile-deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 18.x
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Setup Ruby
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: 3.0

    - name: Setup Fastlane
      run: |
        gem install bundler
        bundle install

    - name: Deploy to TestFlight
      run: bundle exec fastlane ios beta
      env:
        APPLE_ID: ${{ secrets.APPLE_ID }}
        APPLE_APP_SPECIFIC_PASSWORD: ${{ secrets.APPLE_APP_SPECIFIC_PASSWORD }}
        MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}

    - name: Deploy to Google Play Beta
      run: bundle exec fastlane android beta
      env:
        GOOGLE_SERVICE_ACCOUNT_KEY: ${{ secrets.GOOGLE_SERVICE_ACCOUNT_KEY }}
```

### **Jenkins Pipeline**
```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'myapp'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test -- --coverage'
                publishCoverage adapters: [coberturaAdapter('coverage/cobertura-coverage.xml')]
            }
        }

        stage('Security Scan') {
            steps {
                sh 'npm audit --audit-level high'
                // Add SAST tools like SonarQube
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
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
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
                sh 'docker-compose -f docker-compose.staging.yml up -d'
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Deploy to Production?', ok: 'Deploy'
                }
                sh 'docker-compose -f docker-compose.prod.yml up -d'
            }
        }

        stage('Post-Deployment Tests') {
            steps {
                sh 'npm run test:e2e'
            }
        }
    }

    post {
        always {
            sh 'docker system prune -f'
            cleanWs()
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

## 🐳 **Docker Containerization**

### **Dockerfile for Node.js Backend**
```dockerfile
# Use multi-stage build for smaller final image
FROM node:18-alpine AS base

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Create app directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && npm cache clean --force

# Production stage
FROM base AS production

# Copy installed dependencies from base stage
COPY --from=base /app/node_modules ./node_modules

# Copy application code
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Change ownership of app directory
RUN chown -R nextjs:nodejs /app
USER nextjs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

# Start application
ENTRYPOINT ["dumb-init", "--"]
CMD ["npm", "start"]
```

### **Dockerfile for React Native**
```dockerfile
# React Native Android build
FROM reactnativecommunity/react-native-android:latest

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Build Android APK
RUN cd android && ./gradlew assembleRelease

# React Native iOS build
FROM reactnativecommunity/react-native-ios:latest

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Build iOS app
RUN cd ios && xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release -archivePath ./build/MyApp.xcarchive archive
```

### **Docker Compose for Development**
```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - MONGODB_URI=mongodb://mongodb:27017/myapp
      - REDIS_URL=redis://redis:6379
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      - mongodb
      - redis
    networks:
      - app-network

  mongodb:
    image: mongo:5.0
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app
    networks:
      - app-network

volumes:
  mongodb_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

### **Docker Compose for Production**
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    image: myapp:latest
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongodb:27017/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongodb
      - redis
    networks:
      - app-network
    restart: unless-stopped

  mongodb:
    image: mongo:5.0
    volumes:
      - mongodb_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_ROOT_USERNAME}
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_ROOT_PASSWORD}
    networks:
      - app-network
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    networks:
      - app-network
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.prod.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/ssl/certs
    depends_on:
      - app
    networks:
      - app-network
    restart: unless-stopped

volumes:
  mongodb_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

---

## 🏗️ **Infrastructure as Code**

### **Terraform Configuration**
```hcl
# main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# VPC Configuration
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "myapp-vpc"
  }
}

# Subnets
resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "myapp-public-${count.index}"
  }
}

# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = "myapp-cluster"

  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}

# ECS Task Definition
resource "aws_ecs_task_definition" "app" {
  family                   = "myapp"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = 256
  memory                   = 512

  container_definitions = jsonencode([
    {
      name  = "myapp"
      image = "${aws_ecr_repository.app.repository_url}:latest"

      portMappings = [
        {
          containerPort = 3000
          hostPort      = 3000
        }
      ]

      environment = [
        {
          name  = "NODE_ENV"
          value = "production"
        }
      ]

      logConfiguration = {
        logDriver = "awslogs"
        options = {
          "awslogs-group"         = "/ecs/myapp"
          "awslogs-region"        = var.aws_region
          "awslogs-stream-prefix" = "ecs"
        }
      }
    }
  ])
}

# ECS Service
resource "aws_ecs_service" "main" {
  name            = "myapp-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 2

  network_configuration {
    security_groups = [aws_security_group.ecs.id]
    subnets         = aws_subnet.public[*].id
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "myapp"
    container_port   = 3000
  }

  depends_on = [aws_lb_listener.app]
}

# Application Load Balancer
resource "aws_lb" "app" {
  name               = "myapp-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = aws_subnet.public[*].id
}

# RDS Database
resource "aws_db_instance" "main" {
  identifier             = "myapp-db"
  engine                 = "postgres"
  engine_version         = "13.7"
  instance_class         = "db.t3.micro"
  allocated_storage      = 20
  username               = var.db_username
  password               = var.db_password
  db_name                = "myapp"
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  skip_final_snapshot    = true
}

# ElastiCache Redis
resource "aws_elasticache_cluster" "redis" {
  cluster_id           = "myapp-redis"
  engine               = "redis"
  node_type            = "cache.t3.micro"
  num_cache_nodes      = 1
  parameter_group_name = "default.redis6.x"
  port                 = 6379
  security_group_ids   = [aws_security_group.redis.id]
  subnet_group_name    = aws_db_subnet_group.main.name
}
```

### **Ansible Playbook**
```yaml
# deploy.yml
---
- name: Deploy MyApp
  hosts: webservers
  become: yes
  vars:
    app_name: myapp
    app_port: 3000
    node_version: 18

  tasks:
    - name: Update package cache
      apt:
        update_cache: yes

    - name: Install Node.js
      apt:
        name: nodejs
        state: present

    - name: Install npm
      apt:
        name: npm
        state: present

    - name: Create app directory
      file:
        path: /var/www/{{ app_name }}
        state: directory
        owner: www-data
        group: www-data

    - name: Copy application files
      copy:
        src: ../
        dest: /var/www/{{ app_name }}
        owner: www-data
        group: www-data

    - name: Install dependencies
      npm:
        path: /var/www/{{ app_name }}
        state: present

    - name: Create environment file
      template:
        src: .env.j2
        dest: /var/www/{{ app_name }}/.env

    - name: Create PM2 ecosystem file
      template:
        src: ecosystem.config.js.j2
        dest: /var/www/{{ app_name }}/ecosystem.config.js

    - name: Start application with PM2
      command: pm2 start ecosystem.config.js
      args:
        chdir: /var/www/{{ app_name }}

    - name: Configure nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/{{ app_name }}

    - name: Enable nginx site
      file:
        src: /etc/nginx/sites-available/{{ app_name }}
        dest: /etc/nginx/sites-enabled/{{ app_name }}
        state: link

    - name: Restart nginx
      service:
        name: nginx
        state: restarted

    - name: Configure firewall
      ufw:
        rule: allow
        port: "{{ app_port }}"
        proto: tcp
```

---

## 📊 **Monitoring & Logging**

### **Application Monitoring with PM2**
```javascript
// pm2.config.js
module.exports = {
  apps: [{
    name: 'myapp',
    script: 'server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000,
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_file: './logs/combined.log',
    time: true,
    watch: false,
    max_memory_restart: '1G',
    restart_delay: 4000,
    autorestart: true,
    min_uptime: '10s',
    // Monitoring configuration
    vizion: false,
    ignore_watch: ['node_modules', 'logs'],
    env_production: {
      NODE_ENV: 'production',
      PM2_MONITORING: true,
    },
  }],
};
```

### **Winston Logging**
```javascript
const winston = require('winston');
const path = require('path');

// Define log levels
const levels = {
  error: 0,
  warn: 1,
  info: 2,
  http: 3,
  debug: 4,
};

// Define colors for each level
const colors = {
  error: 'red',
  warn: 'yellow',
  info: 'green',
  http: 'magenta',
  debug: 'white',
};

winston.addColors(colors);

// Create logs directory
const logsDir = path.join(__dirname, 'logs');

// Define transports
const transports = [
  // Error log file
  new winston.transports.File({
    filename: path.join(logsDir, 'error.log'),
    level: 'error',
    format: winston.format.combine(
      winston.format.timestamp(),
      winston.format.errors({ stack: true }),
      winston.format.json()
    ),
  }),

  // Combined log file
  new winston.transports.File({
    filename: path.join(logsDir, 'combined.log'),
    format: winston.format.combine(
      winston.format.timestamp(),
      winston.format.json()
    ),
  }),

  // Console transport for development
  new winston.transports.Console({
    level: process.env.NODE_ENV === 'production' ? 'info' : 'debug',
    format: winston.format.combine(
      winston.format.colorize({ all: true }),
      winston.format.timestamp(),
      winston.format.printf(({ timestamp, level, message, ...meta }) => {
        return `${timestamp} ${level}: ${message} ${Object.keys(meta).length ? JSON.stringify(meta) : ''}`;
      })
    ),
  }),
];

// Create logger
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  levels,
  transports,
});

// Add request logging middleware
const requestLogger = (req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    logger.http(`${req.method} ${req.originalUrl} ${res.statusCode} - ${duration}ms`, {
      method: req.method,
      url: req.originalUrl,
      status: res.statusCode,
      duration,
      ip: req.ip,
      userAgent: req.get('User-Agent'),
    });
  });

  next();
};

// Error logging middleware
const errorLogger = (err, req, res, next) => {
  logger.error('Unhandled error', {
    error: err.message,
    stack: err.stack,
    url: req.originalUrl,
    method: req.method,
    ip: req.ip,
    userAgent: req.get('User-Agent'),
  });

  next(err);
};

module.exports = {
  logger,
  requestLogger,
  errorLogger,
};
```

### **Health Checks**
```javascript
// health.js
const mongoose = require('mongoose');
const redis = require('redis');

class HealthChecker {
  constructor() {
    this.checks = [];
    this.addDefaultChecks();
  }

  addDefaultChecks() {
    // Database health check
    this.addCheck('database', async () => {
      try {
        await mongoose.connection.db.admin().ping();
        return { status: 'healthy', responseTime: 0 };
      } catch (error) {
        return { status: 'unhealthy', error: error.message };
      }
    });

    // Redis health check
    this.addCheck('redis', async () => {
      return new Promise((resolve) => {
        const client = redis.createClient();
        const start = Date.now();

        client.on('connect', () => {
          const responseTime = Date.now() - start;
          client.quit();
          resolve({ status: 'healthy', responseTime });
        });

        client.on('error', (error) => {
          resolve({ status: 'unhealthy', error: error.message });
        });
      });
    });

    // Application health check
    this.addCheck('application', async () => {
      return {
        status: 'healthy',
        uptime: process.uptime(),
        memory: process.memoryUsage(),
        version: process.env.npm_package_version,
      };
    });
  }

  addCheck(name, checkFunction) {
    this.checks.push({ name, checkFunction });
  }

  async runChecks() {
    const results = {};
    const startTime = Date.now();

    for (const check of this.checks) {
      try {
        const result = await check.checkFunction();
        results[check.name] = {
          status: result.status,
          timestamp: new Date().toISOString(),
          ...result,
        };
      } catch (error) {
        results[check.name] = {
          status: 'error',
          error: error.message,
          timestamp: new Date().toISOString(),
        };
      }
    }

    const totalTime = Date.now() - startTime;
    const overallStatus = Object.values(results).every(r => r.status === 'healthy') ? 'healthy' : 'unhealthy';

    return {
      status: overallStatus,
      timestamp: new Date().toISOString(),
      responseTime: totalTime,
      checks: results,
    };
  }
}

const healthChecker = new HealthChecker();

// Health endpoint
app.get('/health', async (req, res) => {
  try {
    const health = await healthChecker.runChecks();
    const statusCode = health.status === 'healthy' ? 200 : 503;

    res.status(statusCode).json(health);
  } catch (error) {
    res.status(503).json({
      status: 'error',
      error: error.message,
      timestamp: new Date().toISOString(),
    });
  }
});

// Detailed health endpoint
app.get('/health/detailed', async (req, res) => {
  try {
    const health = await healthChecker.runChecks();
    res.json(health);
  } catch (error) {
    res.status(500).json({
      status: 'error',
      error: error.message,
      timestamp: new Date().toISOString(),
    });
  }
});

module.exports = healthChecker;
```

---

## 🔒 **Security in Production**

### **Security Headers**
```javascript
const helmet = require('helmet');
const express = require('express');

const app = express();

// Basic security headers
app.use(helmet());

// Content Security Policy
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'", 'https://fonts.googleapis.com'],
    fontSrc: ["'self'", 'https://fonts.gstatic.com'],
    scriptSrc: ["'self'", "'unsafe-inline'", "'unsafe-eval'"],
    imgSrc: ["'self'", 'data:', 'https:'],
    connectSrc: ["'self'", 'https://api.example.com'],
    objectSrc: ["'none'"],
    upgradeInsecureRequests: [],
  },
}));

// HSTS (HTTP Strict Transport Security)
app.use(helmet.hsts({
  maxAge: 31536000,
  includeSubDomains: true,
  preload: true,
}));

// Prevent clickjacking
app.use(helmet.frameguard({ action: 'deny' }));

// XSS protection
app.use(helmet.xssFilter());

// Prevent MIME type sniffing
app.use(helmet.noSniff());

// Referrer policy
app.use(helmet.referrerPolicy({ policy: 'strict-origin-when-cross-origin' }));
```

### **Rate Limiting**
```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

// General API rate limiting
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: {
    error: 'Too many requests from this IP, please try again later.',
    retryAfter: 15 * 60, // seconds
  },
  standardHeaders: true,
  legacyHeaders: false,
});

// Auth endpoints rate limiting
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // limit each IP to 5 auth attempts per windowMs
  message: {
    error: 'Too many authentication attempts, please try again later.',
    retryAfter: 15 * 60,
  },
  skipSuccessfulRequests: true, // Don't count successful requests
});

// File upload rate limiting
const uploadLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 10, // limit each IP to 10 uploads per hour
  message: {
    error: 'Upload limit exceeded, please try again later.',
    retryAfter: 60 * 60,
  },
});

// Redis-backed rate limiting for scalability
const redisLimiter = rateLimit({
  store: new RedisStore({
    client: redis.createClient(),
    prefix: 'rate-limit:',
  }),
  windowMs: 15 * 60 * 1000,
  max: 100,
});

// Apply rate limiters
app.use('/api/', apiLimiter);
app.use('/api/auth/', authLimiter);
app.use('/api/upload', uploadLimiter);

// Custom rate limiter for specific endpoints
const createCustomLimiter = (windowMs, max, message) => {
  return rateLimit({
    windowMs,
    max,
    message: {
      error: message,
      retryAfter: Math.ceil(windowMs / 1000),
    },
  });
};

// Heavy operation limiter
const heavyOpLimiter = createCustomLimiter(
  60 * 60 * 1000, // 1 hour
  3, // 3 heavy operations per hour
  'Heavy operation limit exceeded'
);

app.use('/api/heavy-operation', heavyOpLimiter);
```

---

## ⚡ **Performance Optimization**

### **Caching Strategies**
```javascript
const redis = require('redis');
const { promisify } = require('util');

// Redis client
const client = redis.createClient();
const getAsync = promisify(client.get).bind(client);
const setAsync = promisify(client.set).bind(client);
const delAsync = promisify(client.del).bind(client);

// Cache middleware
const cache = (duration = 300) => { // 5 minutes default
  return async (req, res, next) => {
    if (req.method !== 'GET') {
      return next();
    }

    const key = `cache:${req.originalUrl}`;

    try {
      const cachedResponse = await getAsync(key);
      if (cachedResponse) {
        const parsed = JSON.parse(cachedResponse);
        return res.json(parsed);
      }

      // Store original send method
      const originalSend = res.json;

      // Override json method to cache response
      res.json = function(data) {
        setAsync(key, JSON.stringify(data), 'EX', duration);
        originalSend.call(this, data);
      };

      next();
    } catch (error) {
      console.error('Cache error:', error);
      next();
    }
  };
};

// Cache invalidation
const invalidateCache = async (pattern) => {
  const keys = await promisify(client.keys).bind(client)(pattern);
  if (keys.length > 0) {
    await delAsync(keys);
  }
};

// Response compression
const compression = require('compression');

app.use(compression({
  level: 6, // compression level
  threshold: 1024, // compress responses larger than 1KB
  filter: (req, res) => {
    // Don't compress responses with this request header
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  },
}));

// Static file optimization
app.use(express.static('public', {
  maxAge: '1d', // cache static files for 1 day
  etag: true,
  lastModified: true,
}));

// Database query optimization
const mongoose = require('mongoose');

// Add indexes for frequently queried fields
const UserSchema = new mongoose.Schema({
  email: { type: String, unique: true, index: true },
  createdAt: { type: Date, index: true },
  role: { type: String, index: true },
});

// Compound index for complex queries
UserSchema.index({ role: 1, createdAt: -1 });

// Query optimization middleware
const optimizeQuery = (req, res, next) => {
  // Add timeout to prevent long-running queries
  req.queryTimeout = 10000; // 10 seconds

  // Log slow queries
  const start = Date.now();
  const originalSend = res.json;

  res.json = function(data) {
    const duration = Date.now() - start;
    if (duration > 1000) { // Log queries taking more than 1 second
      console.log(`Slow query: ${req.method} ${req.originalUrl} - ${duration}ms`);
    }
    originalSend.call(this, data);
  };

  next();
};

app.use(optimizeQuery);
```

---

## 🎯 **Practical Examples**

### **Complete CI/CD Pipeline**
```yaml
# .github/workflows/production.yml
name: Production Deployment

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Run security scan
      uses: securecodewarrior/github-action-security-scan@v1
      with:
        language: javascript

  test:
    needs: security-scan
    runs-on: ubuntu-latest
    services:
      mongodb:
        image: mongo:5.0
        ports:
        - 27017:27017
      redis:
        image: redis:7-alpine
        ports:
        - 6379:6379

    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-node@v3
      with:
        node-version: 18
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test -- --coverage --watchAll=false
      env:
        MONGODB_URI: mongodb://localhost:27017/test
        REDIS_URL: redis://localhost:6379

    - name: Upload coverage
      uses: codecov/codecov-action@v3

  build-backend:
    needs: test
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Build Docker image
      run: |
        docker build -t myapp-backend:${{ github.sha }} .
        docker tag myapp-backend:${{ github.sha }} myapp-backend:latest

    - name: Push to registry
      run: |
        echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        docker push myapp-backend:${{ github.sha }}
        docker push myapp-backend:latest

  build-mobile:
    needs: test
    runs-on: macos-latest

    steps:
    - uses: actions/checkout@v3

    - uses: actions/setup-node@v3
      with:
        node-version: 18
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Setup Ruby
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: 3.0

    - name: Setup Fastlane
      run: bundle install

    - name: Build iOS
      run: bundle exec fastlane ios build
      env:
        APPLE_ID: ${{ secrets.APPLE_ID }}
        APPLE_APP_SPECIFIC_PASSWORD: ${{ secrets.APPLE_APP_SPECIFIC_PASSWORD }}

    - name: Build Android
      run: bundle exec fastlane android build
      env:
        GOOGLE_SERVICE_ACCOUNT_KEY: ${{ secrets.GOOGLE_SERVICE_ACCOUNT_KEY }}

  deploy-backend:
    needs: build-backend
    runs-on: ubuntu-latest

    steps:
    - name: Deploy to production
      run: |
        # Update docker-compose with new image
        sed -i "s|image: myapp-backend:.*|image: myapp-backend:${{ github.sha }}|g" docker-compose.prod.yml

        # Deploy using docker-compose
        docker-compose -f docker-compose.prod.yml up -d

        # Run database migrations if needed
        docker-compose -f docker-compose.prod.yml exec -T app npm run migrate

        # Health check
        sleep 30
        curl -f http://localhost/health || exit 1

  deploy-mobile:
    needs: build-mobile
    runs-on: ubuntu-latest

    steps:
    - name: Deploy iOS to TestFlight
      run: |
        # Upload to TestFlight
        xcrun altool --upload-app -f ios/build/MyApp.ipa -u ${{ secrets.APPLE_ID }} -p ${{ secrets.APPLE_APP_SPECIFIC_PASSWORD }}

    - name: Deploy Android to Play Store
      run: |
        # Upload to Google Play
        fastlane supply --apk android/app/build/outputs/apk/release/app-release.apk --track beta

  notify:
    needs: [deploy-backend, deploy-mobile]
    runs-on: ubuntu-latest
    if: always()

    steps:
    - name: Send notification
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        text: "Production deployment ${{ job.status == 'success' && 'succeeded' || 'failed' }}"
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Deployment Strategies**: Blue-green, rolling, and canary deployments
- ✅ **React Native Deployment**: iOS App Store and Android Play Store
- ✅ **Backend Deployment**: PM2, Docker, and production server setup
- ✅ **CI/CD Pipelines**: GitHub Actions and Jenkins