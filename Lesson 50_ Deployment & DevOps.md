# Lesson 50: Deployment & DevOps

## 🎯 **Learning Objectives**
- Master deployment strategies for React Native applications
- Set up CI/CD pipelines for automated deployment
- Configure monitoring and alerting systems
- Implement infrastructure as code practices
- Optimize deployment performance and reliability

## 📚 **Table of Contents**
1. [Deployment Fundamentals](#deployment-fundamentals)
2. [App Store Deployment](#app-store-deployment)
3. [CI/CD Pipeline Setup](#cicd-pipeline-setup)
4. [Infrastructure as Code](#infrastructure-as-code)
5. [Monitoring & Alerting](#monitoring--alerting)
6. [Rollback Strategies](#rollback-strategies)
7. [Performance Optimization](#performance-optimization)
8. [Security in Deployment](#security-in-deployment)
9. [Multi-Environment Setup](#multi-environment-setup)
10. [Practical Examples](#practical-examples)

---

## 🚀 **Deployment Fundamentals**

### **Deployment Strategies**
- **Blue-Green Deployment**: Zero-downtime deployments with environment switching
- **Canary Deployment**: Gradual rollout to subset of users
- **Rolling Deployment**: Update instances gradually
- **Immutable Deployment**: Replace entire infrastructure
- **Feature Flags**: Enable/disable features without redeployment

### **Environment Management**
- **Development**: Local development environment
- **Staging**: Pre-production testing environment
- **Production**: Live user environment
- **Feature Branches**: Isolated feature development

### **Version Management**
- **Semantic Versioning**: Major.Minor.Patch
- **Build Numbers**: Unique build identification
- **Release Channels**: Beta, Alpha, Stable releases
- **Hotfixes**: Emergency bug fixes

---

## 📱 **App Store Deployment**

### **iOS App Store Deployment**
```bash
# Fastlane Fastfile for iOS
# fastlane/Fastfile
default_platform(:ios)

platform :ios do
  desc "Build and deploy to TestFlight"
  lane :beta do
    # Increment build number
    increment_build_number(
      build_number: latest_testflight_build_number + 1
    )

    # Build the app
    build_app(
      scheme: "MyApp",
      export_method: "app-store",
      export_options: {
        provisioningProfiles: {
          "com.myapp.ios" => "MyApp Provisioning Profile"
        }
      }
    )

    # Upload to TestFlight
    upload_to_testflight(
      skip_waiting_for_build_processing: true,
      distribute_external: true,
      groups: ["Beta Testers"],
      changelog: "New beta release with bug fixes and improvements"
    )
  end

  desc "Deploy to App Store"
  lane :release do
    # Get the latest build number from TestFlight
    build_number = latest_testflight_build_number

    # Build for production
    build_app(
      scheme: "MyApp",
      export_method: "app-store"
    )

    # Submit to App Store
    deliver(
      submit_for_review: true,
      automatic_release: false,
      force: true,
      skip_metadata: false,
      skip_screenshots: false,
      submission_information: {
        add_id_info_limits_tracking: true,
        add_id_info_serves_ads: false,
        add_id_info_tracks_action: true,
        add_id_info_tracks_install: true,
        add_id_info_uses_idfa: false,
        content_rights_has_rights: true,
        content_rights_contains_third_party_content: true,
        export_compliance_platform: 'ios',
        export_compliance_compliance_required: false,
        export_compliance_encryption_updated: false,
        export_compliance_app_type: nil,
        export_compliance_uses_encryption: false,
        export_compliance_is_exempt: false,
        export_compliance_contains_third_party_cryptography: false,
        export_compliance_contains_proprietary_cryptography: false,
        export_compliance_available_on_french_store: true
      }
    )
  end

  desc "Update app metadata"
  lane :metadata do
    deliver(
      skip_binary_upload: true,
      skip_screenshots: false,
      submit_for_review: false,
      force: true
    )
  end
end
```

### **Android Play Store Deployment**
```bash
# Fastlane Fastfile for Android
# fastlane/Fastfile
default_platform(:android)

platform :android do
  desc "Build and deploy to Google Play Beta"
  lane :beta do
    # Increment version code
    increment_version_code(
      gradle_file_path: "./android/app/build.gradle"
    )

    # Build the bundle
    gradle(
      task: "bundle",
      build_type: "Release",
      project_dir: "./android"
    )

    # Upload to Google Play
    upload_to_play_store(
      track: "beta",
      rollout: "0.1", # 10% rollout
      skip_upload_apk: true,
      skip_upload_aab: false,
      skip_upload_metadata: false,
      skip_upload_changelogs: false,
      skip_upload_images: false,
      skip_upload_screenshots: false
    )
  end

  desc "Promote beta to production"
  lane :promote_to_production do
    upload_to_play_store(
      track: "beta",
      track_promote_to: "production",
      rollout: "0.1",
      skip_upload_apk: true,
      skip_upload_aab: true,
      skip_upload_metadata: true,
      skip_upload_changelogs: true,
      skip_upload_images: true,
      skip_upload_screenshots: true
    )
  end

  desc "Deploy to production"
  lane :production do
    # Increment version code
    increment_version_code(
      gradle_file_path: "./android/app/build.gradle"
    )

    # Build the bundle
    gradle(
      task: "bundle",
      build_type: "Release",
      project_dir: "./android"
    )

    # Upload to production
    upload_to_play_store(
      track: "production",
      rollout: "0.1"
    )
  end
end
```

### **CodePush for OTA Updates**
```javascript
// App.js - CodePush integration
import codePush from 'react-native-code-push';

const codePushOptions = {
  checkFrequency: codePush.CheckFrequency.ON_APP_RESUME,
  installMode: codePush.InstallMode.IMMEDIATE,
  mandatoryInstallMode: codePush.InstallMode.IMMEDIATE,
  updateDialog: {
    appendReleaseDescription: true,
    descriptionPrefix: "\n\nChange log:\n",
    mandatoryContinueButtonLabel: "Continue",
    mandatoryUpdateMessage: "An update is available that must be installed.",
    optionalIgnoreButtonLabel: "Ignore",
    optionalInstallButtonLabel: "Install",
    optionalUpdateMessage: "An update is available. Would you like to install it?",
    title: "Update available"
  }
};

const App = () => {
  return (
    <NavigationContainer>
      {/* Your app content */}
    </NavigationContainer>
  );
};

export default codePush(codePushOptions)(App);
```

```bash
# CodePush deployment script
#!/bin/bash

# Set your CodePush deployment keys
IOS_STAGING_KEY="your-ios-staging-key"
IOS_PRODUCTION_KEY="your-ios-production-key"
ANDROID_STAGING_KEY="your-android-staging-key"
ANDROID_PRODUCTION_KEY="your-android-production-key"

# Function to deploy to staging
deploy_staging() {
  echo "Deploying to staging..."

  # iOS staging
  npx appcenter codepush release-react \
    --app "MyApp/MyApp-iOS" \
    --deployment-name "Staging" \
    --description "Staging release" \
    --mandatory false

  # Android staging
  npx appcenter codepush release-react \
    --app "MyApp/MyApp-Android" \
    --deployment-name "Staging" \
    --description "Staging release" \
    --mandatory false
}

# Function to deploy to production
deploy_production() {
  echo "Deploying to production..."

  # iOS production
  npx appcenter codepush release-react \
    --app "MyApp/MyApp-iOS" \
    --deployment-name "Production" \
    --description "Production release" \
    --mandatory true

  # Android production
  npx appcenter codepush release-react \
    --app "MyApp/MyApp-Android" \
    --deployment-name "Production" \
    --description "Production release" \
    --mandatory true
}

# Function to promote staging to production
promote_to_production() {
  echo "Promoting staging to production..."

  # iOS
  npx appcenter codepush promote \
    --app "MyApp/MyApp-iOS" \
    --source-deployment-name "Staging" \
    --destination-deployment-name "Production" \
    --description "Promoting staging to production"

  # Android
  npx appcenter codepush promote \
    --app "MyApp/MyApp-Android" \
    --source-deployment-name "Staging" \
    --destination-deployment-name "Production" \
    --description "Promoting staging to production"
}

# Main script
case "$1" in
  "staging")
    deploy_staging
    ;;
  "production")
    deploy_production
    ;;
  "promote")
    promote_to_production
    ;;
  *)
    echo "Usage: $0 {staging|production|promote}"
    exit 1
    ;;
esac
```

---

## 🔄 **CI/CD Pipeline Setup**

### **GitHub Actions Workflow**
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
      run: npm run test -- --coverage

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info

  build-ios:
    needs: test
    runs-on: macos-latest
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

    - name: Install CocoaPods
      run: cd ios && pod install

    - name: Build iOS
      run: |
        cd ios
        xcodebuild \
          -workspace MyApp.xcworkspace \
          -scheme MyApp \
          -configuration Release \
          -destination generic/platform=iOS \
          -archivePath MyApp.xcarchive \
          archive

    - name: Upload iOS build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: ios-build
        path: ios/MyApp.xcarchive

  build-android:
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

    - name: Setup Java
      uses: actions/setup-java@v3
      with:
        distribution: 'temurin'
        java-version: '11'

    - name: Install dependencies
      run: npm ci

    - name: Build Android
      run: |
        cd android
        chmod +x gradlew
        ./gradlew assembleRelease

    - name: Upload Android build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: android-build
        path: android/app/build/outputs/

  deploy-staging:
    needs: [build-ios, build-android]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: staging

    steps:
    - uses: actions/checkout@v3

    - name: Download iOS build
      uses: actions/download-artifact@v3
      with:
        name: ios-build
        path: ios/

    - name: Download Android build
      uses: actions/download-artifact@v3
      with:
        name: android-build
        path: android/

    - name: Deploy to staging
      run: |
        # Deploy iOS to TestFlight
        # Deploy Android to Google Play Beta
        echo "Deploying to staging environment"

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production

    steps:
    - name: Deploy to production
      run: |
        # Deploy iOS to App Store
        # Deploy Android to Google Play Production
        echo "Deploying to production environment"
```

### **Fastlane Integration**
```ruby
# fastlane/Fastfile
default_platform(:ios)

platform :ios do
  desc "Run tests"
  lane :test do
    run_tests(
      scheme: "MyApp",
      devices: ["iPhone 13"],
      clean: true
    )
  end

  desc "Build for testing"
  lane :build do
    build_app(
      scheme: "MyApp",
      export_method: "development",
      skip_archive: true
    )
  end

  desc "Deploy to TestFlight"
  lane :deploy_testflight do
    # Get certificates and provisioning profiles
    get_certificates
    get_provisioning_profile

    # Build and upload
    build_app(
      scheme: "MyApp",
      export_method: "app-store"
    )

    upload_to_testflight(
      distribute_external: true,
      groups: ["Beta Testers"]
    )
  end

  desc "Deploy to App Store"
  lane :deploy_appstore do
    deliver(
      submit_for_review: true,
      automatic_release: false,
      force: true
    )
  end
end
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
    Name = "react-native-app-vpc"
  }
}

# Subnets
resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 1}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "react-native-public-subnet-${count.index + 1}"
  }
}

# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = "react-native-cluster"

  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}

# ECS Task Definition
resource "aws_ecs_task_definition" "app" {
  family                   = "react-native-app"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = "256"
  memory                   = "512"

  container_definitions = jsonencode([
    {
      name  = "react-native-app"
      image = "${aws_ecr_repository.app.repository_url}:latest"

      portMappings = [
        {
          containerPort = 8081
          hostPort      = 8081
          protocol      = "tcp"
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
          "awslogs-group"         = "/ecs/react-native-app"
          "awslogs-region"        = var.aws_region
          "awslogs-stream-prefix" = "ecs"
        }
      }
    }
  ])
}

# ECS Service
resource "aws_ecs_service" "main" {
  name            = "react-native-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 2

  network_configuration {
    security_groups  = [aws_security_group.ecs_tasks.id]
    subnets          = aws_subnet.public[*].id
    assign_public_ip = true
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.app.id
    container_name   = "react-native-app"
    container_port   = 8081
  }

  depends_on = [aws_lb_listener.app]
}

# Application Load Balancer
resource "aws_lb" "main" {
  name               = "react-native-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.lb.id]
  subnets            = aws_subnet.public[*].id
}

# RDS Database
resource "aws_db_instance" "main" {
  identifier             = "react-native-db"
  instance_class         = "db.t3.micro"
  allocated_storage      = 20
  engine                 = "postgres"
  engine_version         = "13.7"
  username               = var.db_username
  password               = var.db_password
  db_name                = var.db_name
  publicly_accessible    = false
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  skip_final_snapshot    = true
}

# S3 Bucket for assets
resource "aws_s3_bucket" "assets" {
  bucket = "react-native-app-assets-${random_string.suffix.result}"

  tags = {
    Name = "React Native App Assets"
  }
}

resource "aws_s3_bucket_versioning" "assets" {
  bucket = aws_s3_bucket.assets.id
  versioning_configuration {
    status = "Enabled"
  }
}

# CloudFront Distribution
resource "aws_cloudfront_distribution" "main" {
  origin {
    domain_name = aws_s3_bucket.assets.bucket_regional_domain_name
    origin_id   = "S3-react-native-assets"
  }

  enabled             = true
  is_ipv6_enabled     = true
  default_root_object = "index.html"

  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3-react-native-assets"

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }

    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 3600
    max_ttl                = 86400
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    cloudfront_default_certificate = true
  }
}
```

### **Docker Configuration**
```dockerfile
# Dockerfile
FROM node:18-alpine

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

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build the app
RUN npm run build

# Expose port
EXPOSE 8081

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8081/health || exit 1

# Start the application
CMD ["npm", "start"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8081:8081"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/myapp
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/ssl/certs
    depends_on:
      - app
    restart: unless-stopped

volumes:
  postgres_data:
```

---

## 📊 **Monitoring & Alerting**

### **Application Monitoring**
```javascript
// services/monitoring.js
import * as Sentry from '@sentry/react-native';
import analytics from './analyticsService';

class MonitoringService {
  constructor() {
    this.isInitialized = false;
    this.performanceMarks = new Map();
  }

  // Initialize monitoring
  initialize(sentryConfig, analyticsConfig) {
    // Initialize Sentry
    Sentry.init({
      dsn: sentryConfig.dsn,
      environment: sentryConfig.environment,
      enableTracing: true,
      tracesSampleRate: sentryConfig.tracesSampleRate || 0.1,
      integrations: [
        new Sentry.ReactNativeTracing({
          routingInstrumentation: null,
        }),
      ],
      beforeSend: this.beforeSend.bind(this),
    });

    // Initialize analytics
    analytics.initialize(analyticsConfig);

    this.isInitialized = true;
    console.log('Monitoring initialized');
  }

  // Capture exceptions
  captureException(error, context = {}) {
    if (!this.isInitialized) return;

    console.error('Capturing exception:', error);

    Sentry.withScope((scope) => {
      // Add context
      Object.keys(context).forEach(key => {
        scope.setExtra(key, context[key]);
      });

      // Add tags
      if (context.tags) {
        Object.keys(context.tags).forEach(key => {
          scope.setTag(key, context.tags[key]);
        });
      }

      Sentry.captureException(error);
    });

    // Track in analytics
    analytics.trackEvent('app_error', {
      error_message: error.message,
      error_type: error.name,
      ...context,
    });
  }

  // Capture messages
  captureMessage(message, level = 'info', context = {}) {
    if (!this.isInitialized) return;

    Sentry.withScope((scope) => {
      scope.setLevel(level);

      Object.keys(context).forEach(key => {
        scope.setExtra(key, context[key]);
      });

      Sentry.captureMessage(message);
    });
  }

  // Performance monitoring
  startPerformanceTrace(traceName) {
    if (!this.isInitialized) return null;

    const startTime = Date.now();
    this.performanceMarks.set(traceName, startTime);

    return traceName;
  }

  endPerformanceTrace(traceName, additionalData = {}) {
    if (!this.isInitialized) return;

    const startTime = this.performanceMarks.get(traceName);
    if (!startTime) return;

    const duration = Date.now() - startTime;
    this.performanceMarks.delete(traceName);

    // Track performance
    analytics.trackPerformance(traceName, duration);

    // Send to Sentry
    Sentry.withScope((scope) => {
      scope.setTag('performance', 'true');
      scope.setExtra('duration', duration);
      scope.setExtra('trace_name', traceName);

      Object.keys(additionalData).forEach(key => {
        scope.setExtra(key, additionalData[key]);
      });

      Sentry.captureMessage(`Performance: ${traceName}`, 'info');
    });

    console.log(`Performance: ${traceName} took ${duration}ms`);
  }

  // User action tracking
  trackUserAction(action, data = {}) {
    if (!this.isInitialized) return;

    analytics.trackUserAction(action, data);
  }

  // Business event tracking
  trackBusinessEvent(event, data = {}) {
    if (!this.isInitialized) return;

    analytics.trackBusinessEvent(event, data);
  }

  // Screen view tracking
  trackScreenView(screenName, properties = {}) {
    if (!this.isInitialized) return;

    analytics.trackScreen(screenName, properties);
  }

  // Error boundary integration
  beforeSend(event, hint) {
    // Filter out development errors
    if (__DEV__ && event.exception) {
      console.log('Development error filtered out:', event.exception);
      return null;
    }

    // Add app version
    event.extra = {
      ...event.extra,
      appVersion: '1.0.0',
      platform: Platform.OS,
      timestamp: new Date().toISOString(),
    };

    return event;
  }

  // Health check
  async healthCheck() {
    const checks = {
      sentry: await this.checkSentryHealth(),
      analytics: await this.checkAnalyticsHealth(),
      network: await this.checkNetworkHealth(),
    };

    const isHealthy = Object.values(checks).every(check => check);

    if (!isHealthy) {
      this.captureMessage('Health check failed', 'warning', { checks });
    }

    return {
      healthy: isHealthy,
      checks,
      timestamp: new Date().toISOString(),
    };
  }

  // Individual health checks
  async checkSentryHealth() {
    try {
      // Simple check - in production, you might ping Sentry's health endpoint
      return Sentry.getCurrentHub().getClient() !== null;
    } catch (error) {
      return false;
    }
  }

  async checkAnalyticsHealth() {
    try {
      // Check if analytics is properly initialized
      return analytics.isInitialized || false;
    } catch (error) {
      return false;
    }
  }

  async checkNetworkHealth() {
    try {
      const response = await fetch('https://httpbin.org/status/200', {
        method: 'HEAD',
        timeout: 5000,
      });
      return response.ok;
    } catch (error) {
      return false;
    }
  }

  // Alert configuration
  configureAlerts(alertsConfig) {
    this.alertsConfig = alertsConfig;
  }

  // Send alert
  sendAlert(alertType, message, data = {}) {
    if (!this.alertsConfig) return;

    const alert = {
      type: alertType,
      message,
      data,
      timestamp: new Date().toISOString(),
      environment: __DEV__ ? 'development' : 'production',
    };

    // Send to configured alert destinations
    if (this.alertsConfig.slack) {
      this.sendSlackAlert(alert);
    }

    if (this.alertsConfig.email) {
      this.sendEmailAlert(alert);
    }

    // Track alert in analytics
    analytics.trackEvent('alert_sent', alert);
  }

  // Slack alert
  async sendSlackAlert(alert) {
    try {
      await fetch(this.alertsConfig.slack.webhookUrl, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          text: `🚨 Alert: ${alert.message}`,
          attachments: [
            {
              fields: [
                {
                  title: 'Type',
                  value: alert.type,
                  short: true,
                },
                {
                  title: 'Environment',
                  value: alert.environment,
                  short: true,
                },
                {
                  title: 'Time',
                  value: alert.timestamp,
                  short: true,
                },
              ],
            },
          ],
        }),
      });
    } catch (error) {
      console.error('Slack alert failed:', error);
    }
  }

  // Email alert (placeholder - would integrate with email service)
  async sendEmailAlert(alert) {
    console.log('Email alert:', alert);
    // Implementation would send email via service like SendGrid, SES, etc.
  }
}

export default new MonitoringService();
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Deployment Strategies**: Blue-green, canary, rolling deployments
- ✅ **App Store Deployment**: iOS App Store and Google Play Store
- ✅ **CI/CD Pipelines**: GitHub Actions, Fastlane automation
- ✅ **Infrastructure as Code**: Terraform, Docker, containerization
- ✅ **Monitoring & Alerting**: Sentry, analytics, health checks
- ✅ **Rollback Strategies**: Safe rollback procedures
- ✅ **Performance Optimization**: Build optimization, caching
- ✅ **Security in Deployment**: Secure secrets, environment variables
- ✅ **Multi-Environment Setup**: Dev, staging, production environments
- ✅ **DevOps Best Practices**: Automation, monitoring, reliability

### **Best Practices**
1. **Automate everything**: Build, test, and deployment processes
2. **Use infrastructure as code**: Version control your infrastructure
3. **Implement monitoring**: Track performance and errors
4. **Plan for rollbacks**: Have safe rollback strategies
5. **Secure secrets**: Never commit secrets to version control
6. **Test deployments**: Use staging environments for testing
7. **Monitor costs**: Track cloud resource usage and costs
8. **Document processes**: Maintain deployment documentation
9. **Implement alerting**: Get notified of issues immediately
10. **Continuous improvement**: Learn from each deployment

### **Next Steps**
- Learn advanced deployment patterns
- Implement service mesh architectures
- Explore serverless deployment options
- Study site reliability engineering (SRE)
- Learn about chaos engineering
- Implement automated scaling strategies

---

## 🎯 **Assignment**

### **Task 1: CI/CD Pipeline Setup**
Create a complete CI/CD pipeline with:
- Automated testing for multiple Node.js versions
- Build artifacts for iOS and Android
- Automated deployment to staging
- Manual approval for production deployment
- Rollback capabilities
- Notification system for deployment status

### **Task 2: Infrastructure as Code**
Implement infrastructure as code using Terraform:
- VPC and networking setup
- ECS cluster for containerized deployment
- RDS database configuration
- S3 bucket for static assets
- CloudFront CDN setup
- Security groups and IAM roles

### **Task 3: Monitoring Dashboard**
Build a comprehensive monitoring dashboard with:
- Real-time error tracking
- Performance metrics visualization
- User analytics integration
- Alert configuration
- Health check monitoring
- Cost tracking and optimization

### **Task 4: Deployment Automation**
Create automated deployment scripts for:
- Fastlane configuration for mobile app deployment
- CodePush setup for OTA updates
- Docker containerization
- Multi-environment configuration
- Secret management
- Backup and recovery procedures

---

## 📚 **Additional Resources**
- [Fastlane Documentation](https://docs.fastlane.tools/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Terraform Documentation](https://www.terraform.io/docs)
- [Docker Documentation](https://docs.docker.com/)
- [AWS DevOps Documentation](https://aws.amazon.com/devops/)
- [Google Cloud DevOps](https://cloud.google.com/devops)
- [Azure DevOps](https://azure.microsoft.com/en-us/services/devops/)

---

**Final Capstone Project**: [Capstone Project - Complete React Native Application](Capstone%20Project%20-%20Complete%20React%20Native%20Application.md)