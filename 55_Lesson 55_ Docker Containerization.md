# Lesson 55: Docker Containerization

## 🎯 **Learning Objectives**
- Master Docker containerization for React Native applications
- Create optimized Docker images for mobile app development
- Implement multi-stage builds for efficient deployments
- Set up container orchestration with Docker Compose
- Handle development and production environments with containers

## 📚 **Table of Contents**
1. [Docker Fundamentals](#docker-fundamentals)
2. [React Native Docker Setup](#react-native-docker-setup)
3. [Multi-Stage Builds](#multi-stage-builds)
4. [Development Environment](#development-environment)
5. [Docker Compose](#docker-compose)
6. [Container Orchestration](#container-orchestration)
7. [Docker Best Practices](#docker-best-practices)
8. [CI/CD Integration](#cicd-integration)
9. [Troubleshooting](#troubleshooting)
10. [Practical Examples](#practical-examples)

---

## 🐳 **Docker Fundamentals**

### **What is Docker?**
Docker is a platform for developing, shipping, and running applications in containers. Containers are lightweight, portable, and self-sufficient units that can run applications and their dependencies.

### **Key Docker Concepts**
- **Images**: Read-only templates containing application code and dependencies
- **Containers**: Running instances of Docker images
- **Dockerfile**: Script containing instructions to build a Docker image
- **Registry**: Repository for storing and sharing Docker images
- **Volumes**: Persistent data storage for containers
- **Networks**: Communication between containers

### **Benefits for React Native**
- **Consistent Environments**: Same setup across development, staging, and production
- **Isolation**: Dependencies don't conflict with host system
- **Scalability**: Easy to scale and deploy
- **Version Control**: Docker images are versioned and reproducible
- **CI/CD Integration**: Seamless integration with automated pipelines

---

## 📱 **React Native Docker Setup**

### **Basic Dockerfile for React Native**
```dockerfile
# Use Node.js official image
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
    openjdk11 \
    android-tools \
    bash

# Install Watchman (optional, for better performance)
RUN apk add --no-cache libgcc
RUN cd /tmp && \
    git clone https://github.com/facebook/watchman.git && \
    cd watchman && \
    git checkout v2022.02.21.00 && \
    ./autogen.sh && \
    ./configure && \
    make && \
    make install

# Copy package files
COPY package*.json ./

# Install Node.js dependencies
RUN npm ci

# Copy source code
COPY . .

# Create Android keystore directory
RUN mkdir -p android/app/src/main/assets

# Expose Metro bundler port
EXPOSE 8081

# Default command
CMD ["npm", "start"]
```

### **Android Build Dockerfile**
```dockerfile
# Android build environment
FROM openjdk:11-jdk-slim

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl \
    unzip \
    git \
    nodejs \
    npm \
    && rm -rf /var/lib/apt/lists/*

# Install Android SDK
ENV ANDROID_HOME /opt/android-sdk
RUN mkdir -p ${ANDROID_HOME} && \
    curl -o android-sdk.zip https://dl.google.com/android/repository/commandlinetools-linux-7583922_latest.zip && \
    unzip android-sdk.zip -d ${ANDROID_HOME} && \
    rm android-sdk.zip

# Set Android SDK path
ENV PATH ${PATH}:${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools

# Install Android SDK components
RUN yes | sdkmanager --licenses && \
    sdkmanager \
    "platform-tools" \
    "platforms;android-31" \
    "build-tools;31.0.0" \
    "ndk;23.1.7779620"

# Set working directory
WORKDIR /app

# Copy project files
COPY . .

# Build Android APK
RUN cd android && \
    ./gradlew assembleRelease

# Output APK location
RUN echo "APK built at: android/app/build/outputs/apk/release/app-release.apk"
```

### **iOS Build Dockerfile (macOS required)**
```dockerfile
# iOS build environment (requires macOS host)
FROM macos:latest

# Install Xcode and iOS SDK
RUN xcode-select --install

# Install CocoaPods
RUN gem install cocoapods

# Install Node.js
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash && \
    export NVM_DIR="$HOME/.nvm" && \
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" && \
    nvm install 18 && \
    nvm use 18

# Set working directory
WORKDIR /app

# Copy project files
COPY . .

# Install dependencies
RUN npm ci
RUN cd ios && pod install

# Build iOS app
RUN cd ios && \
    xcodebuild \
    -workspace MyApp.xcworkspace \
    -scheme MyApp \
    -configuration Release \
    -destination generic/platform=iOS \
    -archivePath MyApp.xcarchive \
    archive

# Export IPA
RUN cd ios && \
    xcodebuild \
    -exportArchive \
    -archivePath MyApp.xcarchive \
    -exportOptionsPlist exportOptions.plist \
    -exportPath .

# Output IPA location
RUN echo "IPA built at: MyApp.ipa"
```

---

## 🏗️ **Multi-Stage Builds**

### **Optimized React Native Build**
```dockerfile
# Multi-stage Dockerfile for React Native
FROM node:18-alpine AS base

# Install system dependencies
RUN apk add --no-cache \
    git \
    curl \
    python3 \
    make \
    g++

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Dependencies stage
FROM base AS dependencies

# Install Node.js dependencies
RUN npm ci --only=production && npm cache clean --force

# Development dependencies stage
FROM dependencies AS dev-dependencies

# Install all dependencies (including dev dependencies)
RUN npm ci

# Build stage
FROM dev-dependencies AS build

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Android build stage
FROM openjdk:11-jdk-slim AS android-build

# Copy Android project
COPY --from=build /app/android ./android
COPY --from=build /app/package*.json ./

# Install Android SDK and build tools
RUN apt-get update && apt-get install -y \
    curl \
    unzip \
    && rm -rf /var/lib/apt/lists/*

ENV ANDROID_HOME /opt/android-sdk
RUN mkdir -p ${ANDROID_HOME} && \
    curl -o android-sdk.zip https://dl.google.com/android/repository/commandlinetools-linux-7583922_latest.zip && \
    unzip android-sdk.zip -d ${ANDROID_HOME} && \
    rm android-sdk.zip

ENV PATH ${PATH}:${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools

# Accept Android SDK licenses
RUN yes | sdkmanager --licenses

# Install required Android components
RUN sdkmanager \
    "platform-tools" \
    "platforms;android-31" \
    "build-tools;31.0.0"

# Build Android APK
WORKDIR /android
RUN ./gradlew assembleRelease

# Production stage
FROM node:18-alpine AS production

# Install production dependencies
RUN apk add --no-cache \
    dumb-init \
    curl

# Create app user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Set working directory
WORKDIR /app

# Copy built application from build stage
COPY --from=build --chown=nextjs:nodejs /app/.next ./.next
COPY --from=build --chown=nextjs:nodejs /app/public ./public
COPY --from=build --chown=nextjs:nodejs /app/package*.json ./

# Copy Android APK from android-build stage
COPY --from=android-build /android/app/build/outputs/apk/release/app-release.apk ./android-release.apk

# Install production dependencies only
RUN npm ci --only=production && npm cache clean --force

# Switch to non-root user
USER nextjs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/api/health || exit 1

# Start the application
ENTRYPOINT ["dumb-init", "--"]
CMD ["npm", "start"]
```

### **Backend API Container**
```dockerfile
# Backend API Dockerfile
FROM node:18-alpine AS base

# Install system dependencies
RUN apk add --no-cache \
    git \
    curl \
    python3 \
    make \
    g++ \
    postgresql-client

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Dependencies stage
FROM base AS dependencies

# Install Node.js dependencies
RUN npm ci --only=production && npm cache clean --force

# Development dependencies stage
FROM dependencies AS dev-dependencies

# Install all dependencies
RUN npm ci

# Build stage
FROM dev-dependencies AS build

# Copy source code
COPY . .

# Generate Prisma client
RUN npx prisma generate

# Build the application
RUN npm run build

# Production stage
FROM base AS production

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Create app user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy built application
COPY --from=build --chown=nodejs:nodejs /app/dist ./dist
COPY --from=build --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=build --chown=nodejs:nodejs /app/package*.json ./
COPY --from=build --chown=nodejs:nodejs /app/prisma ./prisma

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3001

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3001/health || exit 1

# Start the application
ENTRYPOINT ["dumb-init", "--"]
CMD ["npm", "run", "start:prod"]
```

---

## 💻 **Development Environment**

### **Development Docker Setup**
```dockerfile
# Development Dockerfile
FROM node:18-alpine

# Install system dependencies
RUN apk add --no-cache \
    git \
    curl \
    python3 \
    make \
    g++ \
    watchman \
    openjdk11 \
    android-tools \
    bash \
    vim \
    htop

# Install React Native CLI
RUN npm install -g @react-native-community/cli

# Create app user
RUN addgroup -g 1000 -S reactnative && \
    adduser -S reactnative -u 1000 -G reactnative

# Set working directory
WORKDIR /app

# Change ownership of working directory
RUN chown -R reactnative:reactnative /app

# Switch to app user
USER reactnative

# Copy package files
COPY --chown=reactnative:reactnative package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY --chown=reactnative:reactnative . .

# Expose ports
EXPOSE 8081 3000 3001

# Default command
CMD ["npm", "start"]
```

### **Development Docker Compose**
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  react-native:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "8081:8081"  # Metro bundler
      - "3000:3000"  # Frontend dev server
    volumes:
      - .:/app
      - /app/node_modules
      - /app/android/app/build
      - /app/ios/build
    environment:
      - NODE_ENV=development
      - REACT_NATIVE_PACKAGER_HOSTNAME=localhost
    networks:
      - react-native-network

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    ports:
      - "3001:3001"
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:password@postgres:5432/myapp_dev
    depends_on:
      - postgres
    networks:
      - react-native-network

  postgres:
    image: postgres:13-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=myapp_dev
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - react-native-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - react-native-network

volumes:
  postgres_data:

networks:
  react-native-network:
    driver: bridge
```

### **Development Scripts**
```bash
#!/bin/bash
# dev.sh - Development environment management

set -e

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

log() {
  echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] $1${NC}"
}

error() {
  echo -e "${RED}[ERROR] $1${NC}"
}

info() {
  echo -e "${BLUE}[INFO] $1${NC}"
}

warning() {
  echo -e "${YELLOW}[WARNING] $1${NC}"
}

# Start development environment
start_dev() {
  log "Starting development environment..."

  # Start services
  docker-compose -f docker-compose.dev.yml up -d

  # Wait for services to be ready
  log "Waiting for services to be ready..."
  sleep 10

  # Check service health
  check_service_health

  log "Development environment started successfully!"
  log "Metro bundler: http://localhost:8081"
  log "Backend API: http://localhost:3001"
  log "PostgreSQL: localhost:5432"
  log "Redis: localhost:6379"
}

# Stop development environment
stop_dev() {
  log "Stopping development environment..."
  docker-compose -f docker-compose.dev.yml down
  log "Development environment stopped"
}

# Restart development environment
restart_dev() {
  log "Restarting development environment..."
  stop_dev
  start_dev
}

# Check service health
check_service_health() {
  # Check PostgreSQL
  if docker-compose -f docker-compose.dev.yml exec -T postgres pg_isready -U user -d myapp_dev > /dev/null 2>&1; then
    log "✅ PostgreSQL is ready"
  else
    warning "⚠️  PostgreSQL is not ready yet"
  fi

  # Check Redis
  if docker-compose -f docker-compose.dev.yml exec -T redis redis-cli ping | grep -q PONG; then
    log "✅ Redis is ready"
  else
    warning "⚠️  Redis is not ready yet"
  fi

  # Check backend
  if curl -s http://localhost:3001/health > /dev/null; then
    log "✅ Backend API is ready"
  else
    warning "⚠️  Backend API is not ready yet"
  fi
}

# View logs
view_logs() {
  local service=${1:-react-native}
  log "Viewing logs for $service..."
  docker-compose -f docker-compose.dev.yml logs -f $service
}

# Run tests
run_tests() {
  log "Running tests..."
  docker-compose -f docker-compose.dev.yml exec react-native npm test
}

# Clean up
cleanup() {
  log "Cleaning up development environment..."
  docker-compose -f docker-compose.dev.yml down -v
  docker system prune -f
  log "Cleanup completed"
}

# Main function
main() {
  case "$1" in
    start)
      start_dev
      ;;
    stop)
      stop_dev
      ;;
    restart)
      restart_dev
      ;;
    logs)
      view_logs "$2"
      ;;
    test)
      run_tests
      ;;
    cleanup)
      cleanup
      ;;
    *)
      echo "Usage: $0 {start|stop|restart|logs|test|cleanup} [service]"
      echo "Services: react-native, backend, postgres, redis"
      exit 1
      ;;
  esac
}

# Run main function
main "$@"
```

---

## 🐙 **Docker Compose**

### **Production Docker Compose**
```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - API_URL=https://api.myapp.com
    depends_on:
      - backend
    networks:
      - app-network
    restart: unless-stopped

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@postgres:5432/myapp_prod
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - postgres
      - redis
    volumes:
      - ./backend/uploads:/app/uploads
    networks:
      - app-network
    restart: unless-stopped

  postgres:
    image: postgres:13-alpine
    environment:
      - POSTGRES_DB=myapp_prod
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backend/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app-network
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
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
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/ssl/certs
      - ./nginx/logs:/var/log/nginx
    depends_on:
      - frontend
      - backend
    networks:
      - app-network
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

### **Nginx Configuration**
```nginx
# nginx/nginx.conf
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # Logging
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/javascript
        application/xml+rss
        application/json;

    # Upstream servers
    upstream frontend {
        server frontend:3000;
    }

    upstream backend {
        server backend:3001;
    }

    server {
        listen 80;
        server_name localhost;

        # Security headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header Referrer-Policy "no-referrer-when-downgrade" always;
        add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

        # Frontend routes
        location / {
            proxy_pass http://frontend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
        }

        # API routes
        location /api/ {
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
        }

        # Static files caching
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
            proxy_pass http://frontend;
        }
# Health check
if health_check; then
  log "✅ Deployment successful!"
  log "Application is running at: http://localhost:3000"
else
  error "❌ Health check failed!"
  warning "Attempting rollback..."
  rollback

  # Final health check after rollback
  if health_check; then
    log "✅ Rollback successful!"
  else
    error "❌ Rollback failed! Manual intervention required."
    exit 1
  fi
fi
}

# Run main function
main "$@"
```

---

## 📚 **Summary**

### **Key Takeaways**
- **Docker Fundamentals**: Understanding containers, images, and Docker architecture
- **React Native Integration**: Creating optimized Dockerfiles for mobile app development
- **Multi-Stage Builds**: Efficient build processes with smaller production images
- **Development Environment**: Consistent development setup across teams
- **Docker Compose**: Orchestrating multi-container applications
- **Container Orchestration**: Scaling and managing containerized applications
- **Best Practices**: Security, performance, and maintainability considerations
- **CI/CD Integration**: Automated build, test, and deployment pipelines
- **Troubleshooting**: Common issues and debugging techniques

### **Benefits Achieved**
- **Consistency**: Same environment from development to production
- **Scalability**: Easy horizontal and vertical scaling
- **Isolation**: Dependencies don't conflict between projects
- **Reproducibility**: Identical environments across different machines
- **Efficiency**: Faster deployment and rollback capabilities
- **Cost-Effectiveness**: Better resource utilization and reduced overhead

### **Next Steps**
1. **Practice**: Set up Docker for your React Native project
2. **Explore**: Learn Kubernetes for advanced orchestration
3. **Integrate**: Implement CI/CD pipelines with Docker
4. **Monitor**: Set up logging and monitoring for containerized apps
5. **Security**: Implement security scanning and compliance checks

### **Additional Resources**
- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Best Practices Guide](https://docs.docker.com/develop/dev-best-practices/)
- [React Native Docker Guide](https://reactnative.dev/docs/environment-setup)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

---

## 🎯 **Lesson Complete!**

Congratulations! You've completed **Lesson 55: Docker Containerization**. You now have the knowledge to:

- ✅ Containerize React Native applications
- ✅ Set up development and production environments
- ✅ Implement multi-stage builds for optimization
- ✅ Use Docker Compose for multi-container orchestration
- ✅ Integrate Docker with CI/CD pipelines
- ✅ Troubleshoot common Docker issues
- ✅ Follow Docker best practices for security and performance

**Ready for the next lesson?** Continue to [Lesson 56: Monitoring & Alerting](Lesson 56_ Monitoring & Alerting.md) to learn about application monitoring and alerting strategies.

---

*Happy Dockering! 🐳*
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
}
```

---

## 🎛️ **Container Orchestration**

### **Docker Swarm Setup**
```bash
#!/bin/bash
# swarm-setup.sh

set -e

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

log() {
  echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] $1${NC}"
}

error() {
  echo -e "${RED}[ERROR] $1${NC}"
}

info() {
  echo -e "${BLUE}[INFO] $1${NC}"
}

warning() {
  echo -e "${YELLOW}[WARNING] $1${NC}"
}

# Initialize Docker Swarm
init_swarm() {
  log "Initializing Docker Swarm..."

  # Get manager IP
  MANAGER_IP=$(hostname -I | awk '{print $1}')

  # Initialize swarm
  docker swarm init --advertise-addr $MANAGER_IP

  # Get join token
  JOIN_TOKEN=$(docker swarm join-token worker -q)

  log "Swarm initialized successfully"
  log "Manager IP: $MANAGER_IP"
  log "Worker join token: $JOIN_TOKEN"
}

# Deploy stack
deploy_stack() {
  local stack_name=${1:-myapp}
  local compose_file=${2:-docker-compose.yml}

  log "Deploying stack: $stack_name"

  # Deploy stack
  docker stack deploy -c $compose_file $stack_name

  # Wait for services to be ready
  log "Waiting for services to be ready..."
  sleep 30

  # Check service status
  docker stack services $stack_name

  log "Stack deployed successfully"
}

# Scale service
scale_service() {
  local service_name=$1
  local replicas=$2

  log "Scaling service $service_name to $replicas replicas"

  docker service scale $service_name=$replicas

  log "Service scaled successfully"
}

# Update service
update_service() {
  local service_name=$1
  local image=$2

  log "Updating service $service_name with image $image"

  docker service update --image $image $service_name

  log "Service updated successfully"
}

# View logs
view_logs() {
  local service_name=$1

  log "Viewing logs for service: $service_name"

  docker service logs -f $service_name
}

# Remove stack
remove_stack() {
  local stack_name=${1:-myapp}

  log "Removing stack: $stack_name"

  docker stack rm $stack_name

  log "Stack removed successfully"
}

# Main function
main() {
  case "$1" in
    init)
      init_swarm
      ;;
    deploy)
      deploy_stack "$2" "$3"
      ;;
    scale)
      scale_service "$2" "$3"
      ;;
    update)
      update_service "$2" "$3"
      ;;
    logs)
      view_logs "$2"
      ;;
    remove)
      remove_stack "$2"
      ;;
    status)
      docker stack services ${2:-myapp}
      ;;
    *)
      echo "Usage: $0 {init|deploy|scale|update|logs|remove|status} [args...]"
      echo "Examples:"
      echo "  $0 init"
      echo "  $0 deploy myapp docker-compose.yml"
      echo "  $0 scale myapp_backend 3"
      echo "  $0 update myapp_backend myapp/backend:v2.0"
      echo "  $0 logs myapp_backend"
      echo "  $0 remove myapp"
      echo "  $0 status myapp"
      exit 1
      ;;
  esac
}

# Run main function
main "$@"
```

### **Docker Swarm Compose File**
```yaml
# docker-compose.swarm.yml
version: '3.8'

services:
  frontend:
    image: myapp/frontend:latest
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - API_URL=https://api.myapp.com
    networks:
      - app-network
    deploy:
      mode: replicated
      replicas: 3
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
        monitor: 60s
        max_failure_ratio: 0.3
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.frontend.rule=Host(`myapp.com`)"
        - "traefik.http.routers.frontend.entrypoints=websecure"
        - "traefik.http.routers.frontend.tls.certresolver=letsencrypt"
        - "traefik.http.services.frontend.loadbalancer.server.port=3000"

  backend:
    image: myapp/backend:latest
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@postgres:5432/myapp_prod
      - REDIS_URL=redis://redis:6379
    networks:
      - app-network
    depends_on:
      - postgres
      - redis
    deploy:
      mode: replicated
      replicas: 2
      restart_policy:
        condition: on-failure
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
        monitor: 60s
        max_failure_ratio: 0.3

  postgres:
    image: postgres:13-alpine
    environment:
      - POSTGRES_DB=myapp_prod
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.role == manager

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - app-network
    deploy:
      mode: replicated
      replicas: 1

  traefik:
    image: traefik:v2.5
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./traefik/traefik.yml:/etc/traefik/traefik.yml
      - ./traefik/dynamic.yml:/etc/traefik/dynamic.yml
      - traefik_certs:/letsencrypt
    networks:
      - app-network
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.role == manager
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.traefik.rule=Host(`traefik.myapp.com`)"
        - "traefik.http.routers.traefik.entrypoints=websecure"
        - "traefik.http.routers.traefik.tls.certresolver=letsencrypt"
        - "traefik.http.services.traefik.loadbalancer.server.port=8080"

volumes:
  postgres_data:
  redis_data:
  traefik_certs:

networks:
  app-network:
    driver: overlay
```

---

## ✅ **Docker Best Practices**

### **Security Best Practices**
```dockerfile
# Secure Dockerfile
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Set working directory
WORKDIR /app

# Change ownership before copying files
RUN chown -R nodejs:nodejs /app

# Copy package files first for better caching
COPY --chown=nodejs:nodejs package*.json ./

# Install dependencies
RUN npm ci --only=production && npm cache clean --force

# Copy source code
COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Use dumb-init for proper signal handling
ENTRYPOINT ["dumb-init", "--"]
CMD ["npm", "start"]
```

### **Performance Optimization**
```dockerfile
# Optimized Dockerfile
FROM node:18-alpine AS base

# Install only necessary packages
RUN apk add --no-cache \
    dumb-init \
    curl

# Create app directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && npm cache clean --force

# Production stage
FROM node:18-alpine AS production

# Copy from base
COPY --from=base /usr/bin/dumb-init /usr/bin/dumb-init
COPY --from=base /app/node_modules ./node_modules

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy application code
COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

# Expose port
EXPOSE 3000

# Start application
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "server.js"]
```

### **Multi-Architecture Builds**
```dockerfile
# Multi-architecture Dockerfile
FROM --platform=$BUILDPLATFORM node:18-alpine AS base

# Install build dependencies
RUN apk add --no-cache python3 make g++

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Build stage
FROM base AS build

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production stage
FROM --platform=$TARGETPLATFORM node:18-alpine AS production

# Copy built application
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/package*.json ./

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Start application
CMD ["npm", "start"]
```

```bash
# Build multi-architecture images
docker buildx create --use --name multi-arch-builder

# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --tag myapp:latest \
  --push \
  .
```

---

## 🔄 **CI/CD Integration**

### **GitHub Actions with Docker**
```yaml
# .github/workflows/docker.yml
name: Docker CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Log in to Container Registry
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}

    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        platforms: linux/amd64,linux/arm64
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production

    steps:
    - name: Deploy to production
      run: |
        echo "Deploying to production..."
        # Add your deployment commands here
```

### **Docker in Jenkins Pipeline**
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

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").inside {
                        sh 'npm test'
                    }
                }
            }
        }

        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry('https://registry.example.com', 'registry-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Deploy to staging
                    if (env.BRANCH_NAME == 'develop') {
                        sh 'docker-compose -f docker-compose.staging.yml up -d'
                    }
                    // Deploy to production
                    else if (env.BRANCH_NAME == 'main') {
                        sh 'docker-compose -f docker-compose.prod.yml up -d'
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker system prune -f'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

---

## 🐛 **Troubleshooting**

### **Common Docker Issues**
```bash
# Debug container
docker run -it --entrypoint /bin/bash myapp:latest

# View container logs
docker logs <container_id>

# Inspect container
docker inspect <container_id>

# Check container resource usage
docker stats <container_id>

# Clean up Docker system
docker system prune -a --volumes

# View Docker disk usage
docker system df

# Remove unused images
docker image prune -f

# Remove stopped containers
docker container prune -f
```

### **React Native Docker Issues**
```bash
# Metro bundler not accessible
# Solution: Set REACT_NATIVE_PACKAGER_HOSTNAME
docker run -e REACT_NATIVE_PACKAGER_HOSTNAME=localhost myapp:latest

# Android build issues
# Solution: Ensure Android SDK is properly installed
docker run --volume /opt/android-sdk:/opt/android-sdk myapp:latest

# iOS build issues
# Solution: iOS builds require macOS host
# Use macOS runners in CI/CD

# Permission issues
# Solution: Create proper user and set permissions
RUN addgroup -g 1000 -S reactnative && \
    adduser -S reactnative -u 1000 -G reactnative
USER reactnative

# Port conflicts
# Solution: Map different host ports
docker run -p 8082:8081 myapp:latest
```

### **Performance Issues**
```bash
# Monitor container performance
docker stats

# View container processes
docker top <container_id>

# Check container logs
docker logs -f <container_id>

# Debug slow builds
# Use Docker buildkit
export DOCKER_BUILDKIT=1
docker build --progress=plain .

# Optimize layer caching
# Order COPY commands by change frequency
COPY package*.json ./
RUN npm ci
COPY . .
```

---

## 🎯 **Practical Examples**

### **Complete React Native Docker Setup**
```dockerfile
# Complete React Native Dockerfile
FROM node:18-alpine AS base

# Install system dependencies
RUN apk add --no-cache \
    git \
    curl \
    python3 \
    make \
    g++ \
    openjdk11 \
    android-tools \
    bash \
    watchman \
    vim

# Install React Native CLI
RUN npm install -g @react-native-community/cli react-native-cli

# Create app user
RUN addgroup -g 1000 -S reactnative && \
    adduser -S reactnative -u 1000 -G reactnative

# Set working directory
WORKDIR /app

# Change ownership
RUN chown -R reactnative:reactnative /app

# Switch to app user
USER reactnative

# Copy package files
COPY --chown=reactnative:reactnative package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY --chown=reactnative:reactnative . .

# Android setup
ENV ANDROID_HOME /opt/android-sdk
ENV PATH ${PATH}:${ANDROID_HOME}/emulator:${ANDROID_HOME}/tools:${ANDROID_HOME}/tools/bin:${ANDROID_HOME}/platform-tools

# iOS setup (for macOS)
ENV DEVELOPER_DIR /Applications/Xcode.app/Contents/Developer

# Expose ports
EXPOSE 8081 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8081/status || exit 1

# Default command
CMD ["npm", "start"]
```

### **Docker Compose for Full Development**
```yaml
# docker-compose.full.yml
version: '3.8'

services:
  # React Native development
  react-native:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "8081:8081"  # Metro bundler
      - "3000:3000"  # Dev server
    volumes:
      - .:/app
      - /app/node_modules
      - /app/android/app/build
      - /app/ios/build
    environment:
      - NODE_ENV=development
      - REACT_NATIVE_PACKAGER_HOSTNAME=localhost
    depends_on:
      - backend
      - postgres
      - redis
    networks:
      - dev-network

  # Backend API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    ports:
      - "3001:3001"
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:password@postgres:5432/myapp_dev
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - dev-network

  # PostgreSQL database
  postgres:
    image: postgres:13-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=myapp_dev
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backend/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - dev-network

  # Redis cache
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - dev-network

  # Adminer (database management)
  adminer:
    image: adminer
    ports:
      - "8080:8080"
    depends_on:
      - postgres
    networks:
      - dev-network

  # MailHog (email testing)
  mailhog:
    image: mailhog/mailhog
    ports:
      - "1025:1025"  # SMTP server
      - "8025:8025"  # Web interface
    networks:
      - dev-network

volumes:
  postgres_data:
  redis_data:

networks:
  dev-network:
    driver: bridge
```

### **Production Deployment Script**
```bash
#!/bin/bash
# deploy-production.sh

set -e

# Configuration
APP_NAME="myapp"
ENVIRONMENT="production"
DOCKER_REGISTRY="myregistry.com"
DOCKER_USERNAME=${DOCKER_USERNAME}
DOCKER_PASSWORD=${DOCKER_PASSWORD}

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

log() {
  echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] $1${NC}"
}

error() {
  echo -e "${RED}[ERROR] $1${NC}"
}

info() {
  echo -e "${BLUE}[INFO] $1${NC}"
}

warning() {
  echo -e "${YELLOW}[WARNING] $1${NC}"
}

# Login to Docker registry
docker_login() {
  log "Logging in to Docker registry..."
  echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin "$DOCKER_REGISTRY"
}

# Build and push images
build_and_push() {
  local service=$1
  local image_name="$DOCKER_REGISTRY/$APP_NAME/$service"

  log "Building $service image..."

  # Build image
  docker build \
    --target production \
    --tag "$image_name:latest" \
    --tag "$image_name:$(git rev-parse --short HEAD)" \
    --cache-from "$image_name:latest" \
    -f "Dockerfile.$service" \
    .

  # Push images
  log "Pushing $service image..."
  docker push "$image_name:latest"
  docker push "$image_name:$(git rev-parse --short HEAD)"
}

# Deploy to production
deploy_production() {
  log "Deploying to production..."

  # Update docker-compose.prod.yml with new image tags
  sed -i "s|image:.*|image: $DOCKER_REGISTRY/$APP_NAME/\${service}:$(git rev-parse --short HEAD)|g" docker-compose.prod.yml

  # Deploy using docker stack or docker-compose
  if command -v docker stack &> /dev/null; then
    docker stack deploy -c docker-compose.prod.yml $APP_NAME
  else
    docker-compose -f docker-compose.prod.yml up -d
  fi
}

# Health check
health_check() {
  log "Performing health checks..."

  # Wait for services to be ready
  sleep 30

  # Check service health
  local services=("frontend" "backend" "postgres" "redis")

  for service in "${services[@]}"; do
    if curl -f "http://localhost:3000/health" > /dev/null 2>&1; then
      log "✅ $service is healthy"
    else
      error "❌ $service is not healthy"
      return 1
    fi
  done
}

# Rollback function
rollback() {
  log "Rolling back to previous version..."

  # Get previous commit
  local previous_commit=$(git rev-parse HEAD~1 | cut -c1-7)

  # Update with previous image
  sed -i "s|image:.*|image: $DOCKER_REGISTRY/$APP_NAME/\${service}:$previous_commit|g" docker-compose.prod.yml

  # Redeploy
  if command -v docker stack &> /dev/null; then
    docker stack deploy -c docker-compose.prod.yml $APP_NAME
  else
    docker-compose -f docker-compose.prod.yml up -d
  fi
}

# Main deployment function
main() {
  log "Starting production deployment of $APP_NAME..."

  # Login to registry
  docker_login

  # Build and push images
  build_and_push "frontend"
  build_and_push "backend"

  # Deploy to production
  deploy_production

  # Health check
 