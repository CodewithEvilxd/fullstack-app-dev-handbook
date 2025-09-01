# Lesson 53: CI/CD Pipeline Setup

## 🎯 **Learning Objectives**
- Set up comprehensive CI/CD pipelines for React Native applications
- Implement automated testing, building, and deployment processes
- Configure multiple environment deployments (dev, staging, production)
- Integrate code quality checks and security scanning
- Monitor pipeline performance and optimize build times

## 📚 **Table of Contents**
1. [CI/CD Fundamentals](#cicd-fundamentals)
2. [GitHub Actions Setup](#github-actions-setup)
3. [Automated Testing](#automated-testing)
4. [Build Automation](#build-automation)
5. [Deployment Strategies](#deployment-strategies)
6. [Environment Management](#environment-management)
7. [Code Quality Gates](#code-quality-gates)
8. [Security Integration](#security-integration)
9. [Pipeline Monitoring](#pipeline-monitoring)
10. [Practical Examples](#practical-examples)

---

## 🔄 **CI/CD Fundamentals**

### **CI/CD Pipeline Stages**
```
Source Code → Build → Test → Security Scan → Deploy → Monitor
     ↑                                                            ↓
  Code Review ← Feature Branch ← Staging ← Production ← Rollback
```

### **Pipeline Components**
- **Source Control**: Git repositories with branching strategy
- **Build System**: Automated compilation and packaging
- **Test Automation**: Unit, integration, and end-to-end tests
- **Artifact Management**: Build artifacts storage and versioning
- **Deployment Automation**: Environment-specific deployments
- **Monitoring**: Pipeline performance and deployment tracking

### **Benefits**
- **Faster Releases**: Automated processes reduce manual work
- **Quality Assurance**: Automated testing catches bugs early
- **Consistency**: Standardized deployment processes
- **Reliability**: Reduced human error in deployments
- **Visibility**: Clear tracking of changes and deployments

---

## 🚀 **GitHub Actions Setup**

### **Basic Workflow Configuration**
```yaml
# .github/workflows/ci.yml
name: CI

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
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linting
      run: npm run lint

    - name: Run type checking
      run: npm run type-check

    - name: Run tests
      run: npm run test -- --coverage --watchAll=false

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        fail_ci_if_error: false
```

### **Advanced Workflow with Matrix Builds**
```yaml
# .github/workflows/ci-advanced.yml
name: CI Advanced

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: 18.x

jobs:
  # Code quality checks
  quality:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run ESLint
      run: npm run lint

    - name: Run Prettier check
      run: npm run prettier:check

    - name: Run TypeScript check
      run: npm run type-check

    - name: Check for security vulnerabilities
      run: npm audit --audit-level moderate

  # Unit and integration tests
  test:
    runs-on: ubuntu-latest
    needs: quality
    strategy:
      matrix:
        test-type: ['unit', 'integration']

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Cache Jest cache
      uses: actions/cache@v3
      with:
        path: ~/.jest
        key: jest-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          jest-${{ runner.os }}-

    - name: Run ${{ matrix.test-type }} tests
      run: npm run test:${{ matrix.test-type }}

    - name: Upload test results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: test-results-${{ matrix.test-type }}
        path: test-results/

  # Build artifacts
  build:
    runs-on: ubuntu-latest
    needs: [quality, test]
    strategy:
      matrix:
        platform: ['ios', 'android']

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Setup Ruby (for iOS)
      if: matrix.platform == 'ios'
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: 2.7

    - name: Install CocoaPods
      if: matrix.platform == 'ios'
      run: |
        cd ios
        pod install

    - name: Setup Java (for Android)
      if: matrix.platform == 'android'
      uses: actions/setup-java@v3
      with:
        distribution: 'temurin'
        java-version: '11'

    - name: Cache Gradle packages
      if: matrix.platform == 'android'
      uses: actions/cache@v3
      with:
        path: |
          ~/.gradle/caches
          ~/.gradle/wrapper
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
        restore-keys: |
          ${{ runner.os }}-gradle-

    - name: Build ${{ matrix.platform }}
      run: npm run build:${{ matrix.platform }}

    - name: Upload build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: build-${{ matrix.platform }}
        path: |
          ${{ matrix.platform == 'ios' && 'ios/build' || 'android/app/build/outputs' }}
        retention-days: 30
```

---

## 🧪 **Automated Testing**

### **Test Configuration**
```javascript
// jest.config.js
module.exports = {
  preset: 'react-native',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  testEnvironment: 'jsdom',
  testMatch: [
    '**/__tests__/**/*.test.js',
    '**/__tests__/**/*.test.ts',
    '**/?(*.)+(spec|test).js',
    '**/?(*.)+(spec|test).ts',
  ],
  testPathIgnorePatterns: [
    '<rootDir>/node_modules/',
    '<rootDir>/android/',
    '<rootDir>/ios/',
  ],
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js',
    '!src/**/*.stories.js',
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html', 'json'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^@components/(.*)$': '<rootDir>/src/components/$1',
    '^@utils/(.*)$': '<rootDir>/src/utils/$1',
    '^@services/(.*)$': '<rootDir>/src/services/$1',
  },
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?|react-navigation|@react-navigation/.*))',
  ],
  testTimeout: 10000,
};
```

### **Jest Setup File**
```javascript
// jest.setup.js
import 'react-native-gesture-handler/jestSetup';
import mockAsyncStorage from '@react-native-async-storage/async-storage/jest/async-storage-mock';

// Mock AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () => mockAsyncStorage);

// Mock react-native-reanimated
jest.mock('react-native-reanimated', () => {
  const Reanimated = require('react-native-reanimated/mock');
  return Reanimated;
});

// Mock react-native-vector-icons
jest.mock('react-native-vector-icons', () => ({
  createIconSet: () => 'IconMock',
}));

// Mock react-native-permissions
jest.mock('react-native-permissions', () => ({
  check: jest.fn(() => Promise.resolve('granted')),
  request: jest.fn(() => Promise.resolve('granted')),
}));

// Mock NetInfo
jest.mock('@react-native-community/netinfo', () => ({
  addEventListener: jest.fn(),
  fetch: jest.fn(() => Promise.resolve({ isConnected: true, type: 'wifi' })),
  useNetInfo: jest.fn(() => ({ isConnected: true, type: 'wifi' })),
}));

// Mock react-native-device-info
jest.mock('react-native-device-info', () => ({
  getVersion: jest.fn(() => '1.0.0'),
  getBuildNumber: jest.fn(() => '1'),
  getSystemName: jest.fn(() => 'iOS'),
  getSystemVersion: jest.fn(() => '14.0'),
}));

// Global test utilities
global.fetch = jest.fn();

// Mock timers
jest.useFakeTimers();

// Silence console warnings during tests
const originalWarn = console.warn;
console.warn = (...args) => {
  if (args[0]?.includes?.('Warning:')) return;
  originalWarn.call(console, ...args);
};

// Custom matchers
expect.extend({
  toBeValidDate(received) {
    const pass = received instanceof Date && !isNaN(received);
    return {
      message: () => `expected ${received} to be a valid Date`,
      pass,
    };
  },
});

// Setup test environment
beforeAll(() => {
  // Global test setup
});

afterAll(() => {
  // Global test cleanup
});
```

### **Test Scripts in package.json**
```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:unit": "jest --testPathPattern=unit",
    "test:integration": "jest --testPathPattern=integration",
    "test:e2e": "detox test",
    "test:ci": "jest --coverage --watchAll=false --passWithNoTests",
    "lint": "eslint src --ext .js,.jsx,.ts,.tsx",
    "lint:fix": "eslint src --ext .js,.jsx,.ts,.tsx --fix",
    "type-check": "tsc --noEmit",
    "prettier": "prettier --check src/**/*.{js,jsx,ts,tsx,json,css,md}",
    "prettier:fix": "prettier --write src/**/*.{js,jsx,ts,tsx,json,css,md}"
  }
}
```

---

## 🏗️ **Build Automation**

### **iOS Build Automation**
```ruby
# fastlane/Fastfile (iOS)
default_platform(:ios)

platform :ios do
  desc "Build iOS app"
  lane :build do |options|
    # Setup certificates and provisioning profiles
    setup_ci if ENV['CI']

    # Match certificates
    match(
      type: "appstore",
      readonly: true,
      api_key: app_store_connect_api_key
    )

    # Update build number
    increment_build_number(
      build_number: ENV['BUILD_NUMBER'] || latest_testflight_build_number + 1
    )

    # Update version number
    increment_version_number(
      version_number: ENV['VERSION_NUMBER'] || get_version_number
    )

    # Install pods
    cocoapods(
      clean_install: true,
      podfile: "./ios/Podfile"
    )

    # Build the app
    build_app(
      scheme: "MyApp",
      workspace: "ios/MyApp.xcworkspace",
      export_method: options[:export_method] || "app-store",
      export_options: {
        provisioningProfiles: {
          "com.myapp.ios" => "MyApp App Store"
        }
      },
      include_bitcode: false,
      include_symbols: true,
      xcargs: "DEBUG_INFORMATION_FORMAT=dwarf-with-dsym"
    )
  end

  desc "Build for TestFlight"
  lane :build_testflight do
    build(export_method: "app-store")
  end

  desc "Build for App Store"
  lane :build_appstore do
    build(export_method: "app-store")
  end
end
```

### **Android Build Automation**
```ruby
# fastlane/Fastfile (Android)
default_platform(:android)

platform :android do
  desc "Build Android app"
  lane :build do |options|
    # Increment version code
    increment_version_code(
      gradle_file_path: "./android/app/build.gradle"
    )

    # Update version name
    increment_version_name(
      gradle_file_path: "./android/app/build.gradle",
      version_name: ENV['VERSION_NAME'] || android_get_version_name
    )

    # Build the bundle
    gradle(
      task: "bundle",
      build_type: "Release",
      project_dir: "./android",
      properties: {
        "android.injected.signing.store.file" => ENV['KEYSTORE_FILE'],
        "android.injected.signing.store.password" => ENV['KEYSTORE_PASSWORD'],
        "android.injected.signing.key.alias" => ENV['KEY_ALIAS'],
        "android.injected.signing.key.password" => ENV['KEY_PASSWORD'],
      }
    )

    # Copy APK and AAB to artifacts
    copy_artifacts(
      target_path: "artifacts",
      artifacts: [
        "android/app/build/outputs/bundle/release/app-release.aab",
        "android/app/build/outputs/apk/release/app-release.apk"
      ]
    )
  end

  desc "Build debug APK"
  lane :build_debug do
    gradle(
      task: "assemble",
      build_type: "Debug",
      project_dir: "./android"
    )
  end

  desc "Build release APK"
  lane :build_release do
    build
  end
end
```

### **Cross-Platform Build Script**
```bash
#!/bin/bash
# build.sh - Cross-platform build script

set -e

# Configuration
PLATFORM=${1:-both}
ENVIRONMENT=${2:-development}
BUILD_TYPE=${3:-release}

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

# Validate inputs
validate_inputs() {
  if [[ ! "$PLATFORM" =~ ^(ios|android|both)$ ]]; then
    error "Invalid platform: $PLATFORM. Must be ios, android, or both"
    exit 1
  fi

  if [[ ! "$ENVIRONMENT" =~ ^(development|staging|production)$ ]]; then
    error "Invalid environment: $ENVIRONMENT. Must be development, staging, or production"
    exit 1
  fi

  if [[ ! "$BUILD_TYPE" =~ ^(debug|release)$ ]]; then
    error "Invalid build type: $BUILD_TYPE. Must be debug or release"
    exit 1
  fi
}

# Setup environment
setup_environment() {
  log "Setting up environment for $ENVIRONMENT..."

  # Load environment variables
  if [ -f ".env.$ENVIRONMENT" ]; then
    export $(cat .env.$ENVIRONMENT | xargs)
  fi

  # Install dependencies
  npm ci

  # Create build directory
  mkdir -p build
}

# Build iOS
build_ios() {
  log "Building iOS app..."

  # Check if running on macOS
  if [[ "$OSTYPE" != "darwin"* ]]; then
    warning "iOS builds can only be done on macOS. Skipping iOS build."
    return
  fi

  cd ios

  # Install CocoaPods
  pod install

  cd ..

  # Use Fastlane
  if [ "$BUILD_TYPE" = "release" ]; then
    npx fastlane ios build_appstore
  else
    npx fastlane ios build_testflight
  fi
}

# Build Android
build_android() {
  log "Building Android app..."

  # Use Fastlane
  if [ "$BUILD_TYPE" = "release" ]; then
    npx fastlane android build_release
  else
    npx fastlane android build_debug
  fi
}

# Main build function
main() {
  log "Starting build process..."
  log "Platform: $PLATFORM"
  log "Environment: $ENVIRONMENT"
  log "Build Type: $BUILD_TYPE"

  validate_inputs
  setup_environment

  case $PLATFORM in
    ios)
      build_ios
      ;;
    android)
      build_android
      ;;
    both)
      build_ios
      build_android
      ;;
  esac

  log "Build completed successfully! 🎉"
}

# Run main function
main "$@"
```

---

## 🚀 **Deployment Strategies**

### **Multi-Environment Deployment**
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [ main ]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy to'
        required: true
        default: 'staging'
        type: choice
        options:
        - staging
        - production

env:
  NODE_VERSION: 18.x

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/main' || github.event.inputs.environment == 'staging'
    runs-on: ubuntu-latest
    environment: staging

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm run test:ci

    - name: Build app
      run: npm run build

    - name: Deploy to staging
      run: |
        # Deploy to staging environment
        echo "Deploying to staging..."

  deploy-production:
    if: github.event.inputs.environment == 'production'
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run full test suite
      run: npm run test:ci

    - name: Build app
      run: npm run build

    - name: Run security scan
      run: npm run security:scan

    - name: Deploy to production
      run: |
        # Deploy to production environment
        echo "Deploying to production..."

    - name: Run smoke tests
      run: npm run test:smoke

    - name: Notify team
      run: |
        # Send notification to team
        echo "Deployment completed successfully!"
```

### **Blue-Green Deployment**
```bash
#!/bin/bash
# blue-green-deploy.sh

set -e

# Configuration
APP_NAME="myapp"
BLUE_PORT=3000
GREEN_PORT=3001
HEALTH_CHECK_ENDPOINT="/health"

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

log() {
  echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] $1${NC}"
}

error() {
  echo -e "${RED}[ERROR] $1${NC}"
}

# Get current active environment
get_active_environment() {
  if curl -s "http://localhost:$BLUE_PORT$HEALTH_CHECK_ENDPOINT" > /dev/null; then
    echo "blue"
  elif curl -s "http://localhost:$GREEN_PORT$HEALTH_CHECK_ENDPOINT" > /dev/null; then
    echo "green"
  else
    echo "none"
  fi
}

# Deploy to environment
deploy_to_environment() {
  local env=$1
  local port=$2

  log "Deploying to $env environment (port $port)..."

  # Stop existing container
  docker stop $APP_NAME-$env 2>/dev/null || true
  docker rm $APP_NAME-$env 2>/dev/null || true

  # Start new container
  docker run -d \
    --name $APP_NAME-$env \
    -p $port:3000 \
    --env-file .env.production \
    $APP_NAME:latest

  # Wait for health check
  log "Waiting for $env environment to be healthy..."
  for i in {1..30}; do
    if curl -s "http://localhost:$port$HEALTH_CHECK_ENDPOINT" > /dev/null; then
      log "$env environment is healthy"
      return 0
    fi
    sleep 2
  done

  error "$env environment failed health check"
  return 1
}

# Switch traffic to new environment
switch_traffic() {
  local new_env=$1
  local new_port=$2

  log "Switching traffic to $new_env environment..."

  # Update load balancer configuration
  # This would typically update nginx, haproxy, or cloud load balancer
  echo "New active environment: $new_env (port $new_port)"

  # Update environment variable or configuration file
  echo "ACTIVE_ENV=$new_env" > .active_env
}

# Rollback function
rollback() {
  local failed_env=$1
  local active_env=$(get_active_environment)

  if [ "$active_env" = "$failed_env" ]; then
    error "Cannot rollback: $failed_env is currently active"
    return 1
  fi

  log "Rolling back to $active_env..."
  switch_traffic $active_env $([ "$active_env" = "blue" ] && echo $BLUE_PORT || echo $GREEN_PORT)
}

# Main deployment function
main() {
  log "Starting blue-green deployment..."

  local active_env=$(get_active_environment)
  local new_env
  local new_port

  # Determine which environment to deploy to
  if [ "$active_env" = "blue" ]; then
    new_env="green"
    new_port=$GREEN_PORT
  elif [ "$active_env" = "green" ]; then
    new_env="blue"
    new_port=$BLUE_PORT
  else
    # No active environment, deploy to blue
    new_env="blue"
    new_port=$BLUE_PORT
  fi

  log "Active environment: $active_env"
  log "Deploying to: $new_env"

  # Deploy to new environment
  if deploy_to_environment $new_env $new_port; then
    # Switch traffic
    switch_traffic $new_env $new_port

    # Clean up old environment
    if [ "$active_env" != "none" ]; then
      log "Cleaning up $active_env environment..."
      docker stop $APP_NAME-$active_env 2>/dev/null || true
      docker rm $APP_NAME-$active_env 2>/dev/null || true
    fi

    log "✅ Blue-green deployment completed successfully!"
  else
    error "❌ Deployment failed"
    rollback $new_env
    exit 1
  fi
}

# Run main function
main "$@"
```

---

## 🌍 **Environment Management**

### **Environment Configuration**
```javascript
// config/environments.js
const environments = {
  development: {
    API_BASE_URL: 'http://localhost:3000/api',
    WEBSOCKET_URL: 'ws://localhost:3000',
    SENTRY_DSN: null,
    ANALYTICS_KEY: null,
    CODEPUSH_DEPLOYMENT_KEY: {
      ios: 'DEV_IOS_KEY',
      android: 'DEV_ANDROID_KEY',
    },
    FEATURE_FLAGS: {
      analytics: false,
      crash_reporting: false,
      push_notifications: false,
    },
  },

  staging: {
    API_BASE_URL: 'https://api-staging.myapp.com/api',
    WEBSOCKET_URL: 'wss://api-staging.myapp.com',
    SENTRY_DSN: 'https://staging-sentry-dsn@sentry.io/project',
    ANALYTICS_KEY: 'staging-analytics-key',
    CODEPUSH_DEPLOYMENT_KEY: {
      ios: 'STAGING_IOS_KEY',
      android: 'STAGING_ANDROID_KEY',
    },
    FEATURE_FLAGS: {
      analytics: true,
      crash_reporting: true,
      push_notifications: true,
    },
  },

  production: {
    API_BASE_URL: 'https://api.myapp.com/api',
    WEBSOCKET_URL: 'wss://api.myapp.com',
    SENTRY_DSN: 'https://prod-sentry-dsn@sentry.io/project',
    ANALYTICS_KEY: 'prod-analytics-key',
    CODEPUSH_DEPLOYMENT_KEY: {
      ios: 'PROD_IOS_KEY',
      android: 'PROD_ANDROID_KEY',
    },
    FEATURE_FLAGS: {
      analytics: true,
      crash_reporting: true,
      push_notifications: true,
    },
  },
};

// Get current environment
const getCurrentEnvironment = () => {
  // Check for environment variable
  if (process.env.NODE_ENV) {
    return process.env.NODE_ENV;
  }

  // Check for __DEV__ (React Native)
  if (typeof __DEV__ !== 'undefined') {
    return __DEV__ ? 'development' : 'production';
  }

  // Default to development
  return 'development';
};

// Get environment configuration
export const getEnvironmentConfig = () => {
  const env = getCurrentEnvironment();
  return environments[env] || environments.development;
};

// Environment-specific helpers
export const isDevelopment = () => getCurrentEnvironment() === 'development';
export const isStaging = () => getCurrentEnvironment() === 'staging';
export const isProduction = () => getCurrentEnvironment() === 'production';

export default environments;
```

### **Environment Variables Management**
```bash
#!/bin/bash
# env-manager.sh - Environment variables management

set -e

# Configuration
ENV_FILE=".env"
ENV_TEMPLATE=".env.template"

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

log() {
  echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] $1${NC}"
}

error() {
  echo -e "${RED}[ERROR] $1${NC}"
}

warning() {
  echo -e "${YELLOW}[WARNING] $1${NC}"
}

# Create environment file from template
create_env_file() {
  local env=$1

  if [ ! -f "$ENV_TEMPLATE" ]; then
    error "Environment template file not found: $ENV_TEMPLATE"
    exit 1
  fi

  local env_file=".env.$env"

  if [ -f "$env_file" ]; then
    warning "Environment file already exists: $env_file"
    read -p "Overwrite? (y/N): " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
      return
    fi
  fi

  cp "$ENV_TEMPLATE" "$env_file"
  log "Created environment file: $env_file"
}

# Validate environment file
validate_env_file() {
  local env_file=$1

  if [ ! -f "$env_file" ]; then
    error "Environment file not found: $env_file"
    return 1
  fi

  log "Validating $env_file..."

  # Check for required variables
  local required_vars=("API_BASE_URL" "DATABASE_URL")

  for var in "${required_vars[@]}"; do
    if ! grep -q "^$var=" "$env_file"; then
      error "Missing required variable: $var"
      return 1
    fi
  done

  # Check for placeholder values
  if grep -q "your-" "$env_file"; then
    warning "Found placeholder values in $env_file"
  fi

  log "Environment file validation passed"
  return 0
}

# Encrypt sensitive environment variables
encrypt_env_file() {
  local env_file=$1
  local encrypted_file="$env_file.enc"

  if [ ! -f "$env_file" ]; then
    error "Environment file not found: $env_file"
    exit 1
  fi

  # Encrypt using openssl (you would use a proper encryption method)
  openssl enc -aes-256-cbc -salt -in "$env_file" -out "$encrypted_file" -k "$ENCRYPTION_KEY"

  log "Encrypted environment file: $encrypted_file"
}

# Decrypt environment file
decrypt_env_file() {
  local encrypted_file=$1
  local env_file="${encrypted_file%.enc}"

  if [ ! -f "$encrypted_file" ]; then
    error "Encrypted file not found: $encrypted_file"
    exit 1
  fi

  # Decrypt using openssl
  openssl enc -d -aes-256-cbc -in "$encrypted_file" -out "$env_file" -k "$ENCRYPTION_KEY"

  log "Decrypted environment file: $env_file"
}

# Set environment for deployment
set_deployment_env() {
  local env=$1

  if [ ! -f ".env.$env" ]; then
    error "Environment file not found: .env.$env"
    exit 1
  fi

  # Copy environment file
  cp ".env.$env" ".env"

  # Export variables
  export $(cat .env | xargs)

  log "Set deployment environment to: $env"
}

# Main function
main() {
  case "$1" in
    create)
      create_env_file "$2"
      ;;
    validate)
      validate_env_file ".env.$2"
      ;;
    encrypt)
      encrypt_env_file ".env.$2"
      ;;
    decrypt)
      decrypt_env_file ".env.$2.enc"
      ;;
    set)
      set_deployment_env "$2"
      ;;
    *)
      echo "Usage: $0 {create|validate|encrypt|decrypt|set} <environment>"
      echo "Environments: development, staging, production"
      exit 1
      ;;
  esac
}

# Run main function
main "$@"
```

---

## 🔍 **Code Quality Gates**

### **ESLint Configuration**
```javascript
// .eslintrc.js
module.exports = {
  root: true,
  extends: [
    '@react-native-community',
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'prettier',
  ],
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint', 'react', 'react-native', 'import'],
  env: {
    'react-native/react-native': true,
    es6: true,
    node: true,
  },
  rules: {
    // TypeScript rules
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-explicit-any': 'warn',

    // React rules
    'react/prop-types': 'off', // Using TypeScript for prop validation
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',

    // React Native rules
    'react-native/no-unused-styles': 'error',
    'react-native/split-platform-components': 'error',
    'react-native/no-inline-styles': 'warn',
    'react-native/no-color-literals': 'warn',

    // Import rules
    'import/order': [
      'error',
      {
        groups: ['builtin', 'external', 'internal', 'parent', 'sibling', 'index'],
        'newlines-between': 'always',
      },
    ],

    // General rules
    'no-console': process.env.NODE_ENV === 'production' ? 'error' : 'warn',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'warn',
    'prefer-const': 'error',
    'no-var': 'error',
  },
  settings: {
    react: {
      version: 'detect',
    },
    'import/resolver': {
      'babel-module': {
        root: ['./src'],
        alias: {
          '@': './src',
          '@components': './src/components',
          '@utils': './src/utils',
          '@services': './src/services',
        },
      },
    },
  },
  overrides: [
    {
      files: ['*.test.js', '*.test.ts', '*.test.tsx'],
      env: {
        jest: true,
      },
      rules: {
        '@typescript-eslint/no-explicit-any': 'off',
        'react-native/no-inline-styles': 'off',
      },
    },
  ],
};
```

### **Pre-commit Hooks**
```javascript
// .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Run linting
npm run lint

# Run type checking
npm run type-check

# Run tests
npm run test:ci

# Check for secrets
npm run security:check
```

```json
// package.json - Husky configuration
{
  "husky": {
    "hooks": {
      "pre-commit": "npm run pre-commit",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  }
}
```

### **CommitLint Configuration**
```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'build',
        'chore',
        'ci',
        'docs',
        'feat',
        'fix',
        'perf',
        'refactor',
        'revert',
        'style',
        'test',
      ],
    ],
    'subject-case': [2, 'never', ['start-case', 'pascal-case', 'upper-case']],
    'subject-empty': [2, 'never'],
    'subject-full-stop': [2, 'never', '.'],
    'header-max-length': [2, 'always', 72],
  },
};
```

---

## 🔒 **Security Integration**

### **Security Scanning**
```yaml
# .github/workflows/security.yml
name: Security

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  schedule:
    # Run weekly security scan
    - cron: '0 0 * * 0'

jobs:
  security-scan:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 18.x
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run npm audit
      run: npm audit --audit-level moderate

    - name: Run Snyk security scan
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        args: --severity-threshold=high

    - name: Run CodeQL Analysis
      uses: github/codeql-action/init@v2
      with:
        languages: javascript

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2

    - name: Run dependency check
      uses: dependency-check/Dependency-Check_Action@main
      with:
        project: 'MyApp'
        path: '.'
        format: 'ALL'

    - name: Upload security scan results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: security-scan-results
        path: |
          reports/
          *.sarif
```

### **Secrets Management**
```javascript
// scripts/secrets-manager.js
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');

class SecretsManager {
  constructor() {
    this.secretsFile = '.secrets.enc';
    this.keyFile = '.secrets.key';
  }

  // Generate encryption key
  generateKey() {
    const key = crypto.randomBytes(32).toString('hex');
    fs.writeFileSync(this.keyFile, key);
    console.log('Generated encryption key:', this.keyFile);
    return key;
  }

  // Load encryption key
  loadKey() {
    if (!fs.existsSync(this.keyFile)) {
      throw new Error('Encryption key not found. Run generate-key first.');
    }
    return fs.readFileSync(this.keyFile, 'utf8');
  }

  // Encrypt secrets
  encryptSecrets(secrets) {
    const key = this.loadKey();
    const cipher = crypto.createCipher('aes-256-cbc', key);
    let encrypted = cipher.update(JSON.stringify(secrets), 'utf8', 'hex');
    encrypted += cipher.final('hex');
    return encrypted;
  }

  // Decrypt secrets
  decryptSecrets(encryptedData) {
    const key = this.loadKey();
    const decipher = crypto.createDecipher('aes-256-cbc', key);
    let decrypted = decipher.update(encryptedData, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    return JSON.parse(decrypted);
  }

  // Save encrypted secrets
  saveSecrets(secrets) {
    const encrypted = this.encryptSecrets(secrets);
    fs.writeFileSync(this.secretsFile, encrypted);
    console.log('Secrets saved to:', this.secretsFile);
  }

  // Load decrypted secrets
  loadSecrets() {
    if (!fs.existsSync(this.secretsFile)) {
      throw new Error('Encrypted secrets file not found.');
    }

    const encryptedData = fs.readFileSync(this.secretsFile, 'utf8');
    return this.decryptSecrets(encryptedData);
  }

  // Add secret
  addSecret(key, value) {
    const secrets = this.loadSecrets();
    secrets[key] = value;
    this.saveSecrets(secrets);
    console.log(`Added secret: ${key}`);
  }

  // Remove secret
  removeSecret(key) {
    const secrets = this.loadSecrets();
    delete secrets[key];
    this.saveSecrets(secrets);
    console.log(`Removed secret: ${key}`);
  }

  // List secrets (keys only)
  listSecrets() {
    const secrets = this.loadSecrets();
    return Object.keys(secrets);
  }

  // Validate secrets
  validateSecrets(requiredSecrets) {
    const secrets = this.loadSecrets();
    const missing = requiredSecrets.filter(key => !secrets[key]);

    if (missing.length > 0) {
      throw new Error(`Missing required secrets: ${missing.join(', ')}`);
    }

    console.log('All required secrets are present');
  }
}

// CLI usage
if (require.main === module) {
  const args = process.argv.slice(2);
  const command = args[0];
  const secretsManager = new SecretsManager();

  try {
    switch (command) {
      case 'generate-key':
        secretsManager.generateKey();
        break;

      case 'add':
        if (args.length < 3) {
          console.error('Usage: node secrets-manager.js add <key> <value>');
          process.exit(1);
        }
        secretsManager.addSecret(args[1], args[2]);
        break;

      case 'remove':
        if (args.length < 2) {
          console.error('Usage: node secrets-manager.js remove <key>');
          process.exit(1);
        }
        secretsManager.removeSecret(args[1]);
        break;

      case 'list':
        const keys = secretsManager.listSecrets();
        console.log('Secrets:', keys);
        break;

      case 'validate':
        const required = args.slice(1);
        secretsManager.validateSecrets(required);
        break;

      default:
        console.error('Usage: node secrets-manager.js {generate-key|add|remove|list|validate}');
        process.exit(1);
    }
  } catch (error) {
    console.error('Error:', error.message);
    process.exit(1);
  }
}

module.exports = SecretsManager;
```

---

## 📊 **Pipeline Monitoring**

### **Pipeline Analytics**
```javascript
// scripts/pipeline-analytics.js
const fs = require('fs');
const path = require('path');

class PipelineAnalytics {
  constructor() {
    this.analyticsFile = '.pipeline-analytics.json';
    this.analytics = this.loadAnalytics();
  }

  // Load analytics data
  loadAnalytics() {
    try {
      if (fs.existsSync(this.analyticsFile)) {
        return JSON.parse(fs.readFileSync(this.analyticsFile, 'utf8'));
      }
    } catch (error) {
      console.error('Error loading analytics:', error);
    }

    return {
      builds: [],
      deployments: [],
      failures: [],
      performance: {},
    };
  }

  // Save analytics data
  saveAnalytics() {
    try {
      fs.writeFileSync(this.analyticsFile, JSON.stringify(this.analytics, null, 2));
    } catch (error) {
      console.error('Error saving analytics:', error);
    }
  }

  // Track build
  trackBuild(buildData) {
    const build = {
      id: buildData.id || Date.now().toString(),
      timestamp: new Date().toISOString(),
      duration: buildData.duration,
      status: buildData.status,
      branch: buildData.branch,
      commit: buildData.commit,
      platform: buildData.platform,
      environment: buildData.environment,
    };

    this.analytics.builds.push(build);

    // Keep only last 1000 builds
    if (this.analytics.builds.length > 1000) {
      this.analytics.builds = this.analytics.builds.slice(-1000);
    }

    this.saveAnalytics();
  }

  // Track deployment
  trackDeployment(deploymentData) {
    const deployment = {
      id: deploymentData.id || Date.now().toString(),
      timestamp: new Date().toISOString(),
      duration: deploymentData.duration,
      status: deploymentData.status,
      environment: deploymentData.environment,
      version: deploymentData.version,
      platform: deploymentData.platform,
    };

    this.analytics.deployments.push(deployment);

    // Keep only last 500 deployments
    if (this.analytics.deployments.length > 500) {
      this.analytics.deployments = this.analytics.deployments.slice(-500);
    }

    this.saveAnalytics();
  }

  // Track failure
  trackFailure(failureData) {
    const failure = {
      id: Date.now().toString(),
      timestamp: new Date().toISOString(),
      type: failureData.type,
      stage: failureData.stage,
      error: failureData.error,
      context: failureData.context,
    };

    this.analytics.failures.push(failure);

    // Keep only last 500 failures
    if (this.analytics.failures.length > 500) {
      this.analytics.failures = this.analytics.failures.slice(-500);
    }

    this.saveAnalytics();
  }

  // Get build statistics
  getBuildStats() {
    const builds = this.analytics.builds;
    const total = builds.length;

    if (total === 0) return { total: 0 };

    const successful = builds.filter(b => b.status === 'success').length;
    const failed = builds.filter(b => b.status === 'failure').length;
    const successRate = (successful / total) * 100;

    const avgDuration = builds.reduce((sum, b) => sum + (b.duration || 0), 0) / total;

    return {
      total,
      successful,
      failed,
      successRate: Math.round(successRate * 100) / 100,
      avgDuration: Math.round(avgDuration),
    };
  }

  // Get deployment statistics
  getDeploymentStats() {
    const deployments = this.analytics.deployments;
    const total = deployments.length;

    if (total === 0) return { total: 0 };

    const successful = deployments.filter(d => d.status === 'success').length;
    const failed = deployments.filter(d => d.status === 'failure').length;
    const successRate = (successful / total) * 100;

    const avgDuration = deployments.reduce((sum, d) => sum + (d.duration || 0), 0) / total;

    return {
      total,
      successful,
      failed,
      successRate: Math.round(successRate * 100) / 100,
      avgDuration: Math.round(avgDuration),
    };
  }

  // Get failure analysis
  getFailureAnalysis() {
    const failures = this.analytics.failures;

    const failureTypes = {};
    const failureStages = {};

    failures.forEach(failure => {
      failureTypes[failure.type] = (failureTypes[failure.type] || 0) + 1;
      failureStages[failure.stage] = (failureStages[failure.stage] || 0) + 1;
    });

    return {
      totalFailures: failures.length,
      failureTypes,
      failureStages,
      recentFailures: failures.slice(-10), // Last 10 failures
    };
  }

  // Generate report
  generateReport() {
    const buildStats = this.getBuildStats();
    const deploymentStats = this.getDeploymentStats();
    const failureAnalysis = this.getFailureAnalysis();

    return {
      generatedAt: new Date().toISOString(),
       start: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString(), // Last 30 days
        end: new Date().toISOString(),
      },
      buildStats,
      deploymentStats,
      failureAnalysis,
      recommendations: this.generateRecommendations(),
    };
  }

  // Generate recommendations based on analytics
  generateRecommendations() {
    const recommendations = [];
    const buildStats = this.getBuildStats();
    const failureAnalysis = this.getFailureAnalysis();

    // Build success rate recommendations
    if (buildStats.successRate < 80) {
      recommendations.push('Improve build success rate by fixing flaky tests and build issues');
    }

    // Deployment recommendations
    if (failureAnalysis.totalFailures > 10) {
      recommendations.push('Investigate and fix recurring deployment failures');
    }

    // Performance recommendations
    if (buildStats.avgDuration > 600) { // 10 minutes
      recommendations.push('Optimize build times by implementing caching and parallelization');
    }

    if (recommendations.length === 0) {
      recommendations.push('Continue monitoring pipeline performance and maintain current practices');
    }

    return recommendations;
  }
}

export default new PipelineAnalytics();
```

---

## 📱 **Practical Examples**

### **Complete CI/CD Pipeline for React Native**
```yaml
# .github/workflows/complete-cicd.yml
name: Complete CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  release:
    types: [ published ]

env:
  NODE_VERSION: 18.x
  RUBY_VERSION: 2.7

jobs:
  # Code Quality & Security
  quality-gate:
    runs-on: ubuntu-latest
    outputs:
      quality-passed: ${{ steps.quality.outputs.passed }}

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run ESLint
      run: npm run lint

    - name: Run TypeScript check
      run: npm run type-check

    - name: Run Prettier check
      run: npm run prettier:check

    - name: Check for security vulnerabilities
      run: npm audit --audit-level moderate

    - name: Quality gate check
      id: quality
      run: echo "::set-output name=passed::true"

  # Unit & Integration Tests
  test:
    needs: quality-gate
    runs-on: ubuntu-latest
    strategy:
      matrix:
        test-type: ['unit', 'integration']

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Cache Jest cache
      uses: actions/cache@v3
      with:
        path: ~/.jest
        key: jest-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          jest-${{ runner.os }}-

    - name: Run ${{ matrix.test-type }} tests
      run: npm run test:${{ matrix.test-type }} -- --coverage --watchAll=false

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        flags: ${{ matrix.test-type }}
        fail_ci_if_error: false

    - name: Upload test results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: test-results-${{ matrix.test-type }}
        path: test-results/

  # Build Android
  build-android:
    needs: [quality-gate, test]
    runs-on: ubuntu-latest
    if: github.event_name != 'release' || contains(github.event.release.tag_name, 'android') || contains(github.event.release.tag_name, 'both')

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Setup Java
      uses: actions/setup-java@v3
      with:
        distribution: 'temurin'
        java-version: '11'

    - name: Install dependencies
      run: npm ci

    - name: Cache Gradle packages
      uses: actions/cache@v3
      with:
        path: |
          ~/.gradle/caches
          ~/.gradle/wrapper
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
        restore-keys: |
          ${{ runner.os }}-gradle-

    - name: Build Android release
      run: |
        cd android
        chmod +x gradlew
        ./gradlew assembleRelease --no-daemon

    - name: Sign Android APK
      if: github.event_name == 'release'
      run: |
        # Sign APK for release
        echo "APK signing would happen here"

    - name: Upload Android build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: android-build
        path: |
          android/app/build/outputs/apk/release/app-release.apk
          android/app/build/outputs/bundle/release/app-release.aab
        retention-days: 30

  # Build iOS
  build-ios:
    needs: [quality-gate, test]
    runs-on: macos-latest
    if: github.event_name != 'release' || contains(github.event.release.tag_name, 'ios') || contains(github.event.release.tag_name, 'both')

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Setup Ruby
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: ${{ env.RUBY_VERSION }}

    - name: Install dependencies
      run: npm ci

    - name: Install CocoaPods
      run: |
        cd ios
        pod install

    - name: Build iOS release
      run: |
        cd ios
        xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release -destination generic/platform=iOS -archivePath build/MyApp.xcarchive archive

    - name: Export iOS IPA
      run: |
        cd ios
        xcodebuild -exportArchive -archivePath build/MyApp.xcarchive -exportPath build -exportOptionsPlist ExportOptions.plist

    - name: Upload iOS build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: ios-build
        path: ios/build/MyApp.ipa
        retention-days: 30

  # Deploy to TestFlight
  deploy-testflight:
    needs: build-ios
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop' || (github.event_name == 'release' && contains(github.event.release.tag_name, 'beta'))

    steps:
    - name: Download iOS build
      uses: actions/download-artifact@v3
      with:
        name: ios-build
        path: ios-build/

    - name: Deploy to TestFlight
      run: |
        # Upload to TestFlight using Fastlane or App Store Connect API
        echo "Deploying to TestFlight..."

  # Deploy to Google Play Beta
  deploy-play-beta:
    needs: build-android
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop' || (github.event_name == 'release' && contains(github.event.release.tag_name, 'beta'))

    steps:
    - name: Download Android build
      uses: actions/download-artifact@v3
      with:
        name: android-build
        path: android-build/

    - name: Deploy to Google Play Beta
      run: |
        # Upload to Google Play Beta using Fastlane or Play Developer API
        echo "Deploying to Google Play Beta..."

  # Deploy to Production
  deploy-production:
    needs: [build-android, build-ios]
    runs-on: ubuntu-latest
    if: github.event_name == 'release' && !contains(github.event.release.tag_name, 'beta')
    environment: production

    steps:
    - name: Download Android build
      uses: actions/download-artifact@v3
      with:
        name: android-build
        path: android-build/

    - name: Download iOS build
      uses: actions/download-artifact@v3
      with:
        name: ios-build
        path: ios-build/

    - name: Deploy to App Store
      run: |
        # Deploy iOS to App Store
        echo "Deploying iOS to App Store..."

    - name: Deploy to Google Play
      run: |
        # Deploy Android to Google Play
        echo "Deploying Android to Google Play..."

    - name: Create GitHub release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: ${{ github.ref }}
        release_name: Release ${{ github.ref }}
        body: |
          ## Changes
          - Automated deployment to production
          - iOS and Android builds deployed
        draft: false
        prerelease: false

  # Notify team
  notify:
    needs: [deploy-production]
    runs-on: ubuntu-latest
    if: always()

    steps:
    - name: Notify Slack
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        text: "CI/CD Pipeline completed with status: ${{ job.status }}"
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
      if: always()
```

### **Pipeline Monitoring Dashboard**
```javascript
// components/PipelineDashboard.js
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, StyleSheet, RefreshControl } from 'react-native';
import PipelineAnalytics from '../scripts/pipeline-analytics';

const PipelineDashboard = () => {
  const [analytics, setAnalytics] = useState(null);
  const [refreshing, setRefreshing] = useState(false);

  useEffect(() => {
    loadAnalytics();
  }, []);

  const loadAnalytics = async () => {
    try {
      const pipelineAnalytics = new PipelineAnalytics();
      const report = await pipelineAnalytics.generateReport();
      setAnalytics(report);
    } catch (error) {
      console.error('Error loading analytics:', error);
    }
  };

  const onRefresh = async () => {
    setRefreshing(true);
    await loadAnalytics();
    setRefreshing(false);
  };

  const formatDuration = (seconds) => {
    if (!seconds) return 'N/A';
    const minutes = Math.floor(seconds / 60);
    const remainingSeconds = Math.round(seconds % 60);
    return `${minutes}m ${remainingSeconds}s`;
  };

  const formatPercentage = (value) => {
    return value ? `${value.toFixed(1)}%` : 'N/A';
  };

  if (!analytics) {
    return (
      <View style={styles.loading}>
        <Text>Loading pipeline analytics...</Text>
      </View>
    );
  }

  return (
    <ScrollView
      style={styles.container}
      refreshControl={
        <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
      }
    >
      <Text style={styles.title}>CI/CD Pipeline Analytics</Text>
      <Text style={styles.lastUpdate}>
        Last updated: {new Date(analytics.generatedAt).toLocaleString()}
      </Text>

      {/* Build Statistics */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Build Statistics</Text>

        <View style={styles.statsGrid}>
          <View style={styles.stat}>
            <Text style={styles.statValue}>{analytics.buildStats.total}</Text>
            <Text style={styles.statLabel}>Total Builds</Text>
          </View>

          <View style={styles.stat}>
            <Text style={styles.statValue}>{analytics.buildStats.successful}</Text>
            <Text style={styles.statLabel}>Successful</Text>
          </View>

          <View style={styles.stat}>
            <Text style={styles.statValue}>{analytics.buildStats.failed}</Text>
            <Text style={styles.statLabel}>Failed</Text>
          </View>

          <View style={styles.stat}>
            <Text style={styles.statValue}>
              {formatPercentage(analytics.buildStats.successRate)}
            </Text>
            <Text style={styles.statLabel}>Success Rate</Text>
          </View>
        </View>

        <Text style={styles.metric}>
          Average Build Time: {formatDuration(analytics.buildStats.avgDuration)}
        </Text>
      </View>

      {/* Deployment Statistics */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Deployment Statistics</Text>

        <View style={styles.statsGrid}>
          <View style={styles.stat}>
            <Text style={styles.statValue}>{analytics.deploymentStats.total}</Text>
            <Text style={styles.statLabel}>Total Deployments</Text>
          </View>

          <View style={styles.stat}>
            <Text style={styles.statValue}>{analytics.deploymentStats.successful}</Text>
            <Text style={styles.statLabel}>Successful</Text>
          </View>

          <View style={styles.stat}>
            <Text style={styles.statValue}>
              {formatPercentage(analytics.deploymentStats.successRate)}
            </Text>
            <Text style={styles.statLabel}>Success Rate</Text>
          </View>
        </View>

        <Text style={styles.metric}>
          Average Deployment Time: {formatDuration(analytics.deploymentStats.avgDuration)}
        </Text>
      </View>

      {/* Failure Analysis */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Failure Analysis</Text>

        <Text style={styles.metric}>
          Total Failures: {analytics.failureAnalysis.totalFailures}
        </Text>

        <Text style={styles.subTitle}>Failure Types:</Text>
        {Object.entries(analytics.failureAnalysis.failureTypes).map(([type, count]) => (
          <Text key={type} style={styles.failureItem}>
            {type}: {count}
          </Text>
        ))}

        <Text style={styles.subTitle}>Recent Failures:</Text>
        {analytics.failureAnalysis.recentFailures.slice(0, 5).map((failure, index) => (
          <View key={index} style={styles.recentFailure}>
            <Text style={styles.failureType}>{failure.type}</Text>
            <Text style={styles.failureTime}>
              {new Date(failure.timestamp).toLocaleString()}
            </Text>
          </View>
        ))}
      </View>

      {/* Recommendations */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Recommendations</Text>
        {analytics.recommendations.map((recommendation, index) => (
          <Text key={index} style={styles.recommendation}>
            • {recommendation}
          </Text>
        ))}
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  loading: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    margin: 20,
  },
  lastUpdate: {
    textAlign: 'center',
    color: '#666',
    marginBottom: 20,
  },
  section: {
    backgroundColor: 'white',
    margin: 10,
    borderRadius: 10,
    padding: 15,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
  },
  statsGrid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-between',
    marginBottom: 15,
  },
  stat: {
    width: '48%',
    alignItems: 'center',
    marginBottom: 10,
  },
  statValue: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#007AFF',
  },
  statLabel: {
    fontSize: 12,
    color: '#666',
    marginTop: 5,
  },
  metric: {
    fontSize: 14,
    color: '#333',
    marginBottom: 10,
  },
  subTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginTop: 10,
    marginBottom: 5,
  },
  failureItem: {
    fontSize: 14,
    color: '#666',
    marginLeft: 10,
    marginBottom: 5,
  },
  recentFailure: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    paddingVertical: 5,
    paddingHorizontal: 10,
    backgroundColor: '#f9f9f9',
    marginBottom: 5,
    borderRadius: 5,
  },
  failureType: {
    fontSize: 14,
    fontWeight: 'bold',
  },
  failureTime: {
    fontSize: 12,
    color: '#666',
  },
  recommendation: {
    fontSize: 14,
    color: '#333',
    marginBottom: 5,
    lineHeight: 20,
  },
});

export default PipelineDashboard;
```

---

## 📚 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **CI/CD Fundamentals**: Pipeline stages, components, and benefits
- ✅ **GitHub Actions**: Workflow configuration, matrix builds, caching
- ✅ **Automated Testing**: Jest configuration, test scripts, coverage
- ✅ **Build Automation**: Fastlane for iOS/Android, cross-platform builds
- ✅ **Deployment Strategies**: Multi-environment, blue-green deployment
- ✅ **Environment Management**: Configuration, variables, secrets
- ✅ **Code Quality Gates**: ESLint, Prettier, Husky, CommitLint
- ✅ **Security Integration**: Scanning, secrets management
- ✅ **Pipeline Monitoring**: Analytics, dashboards, alerting

### **Best Practices**
1. **Automate everything**: From code quality to deployment
2. **Use matrix builds**: Test across multiple Node.js versions
3. **Implement caching**: Speed up builds with dependency caching
4. **Secure secrets**: Use GitHub secrets and encrypted files
5. **Monitor pipelines**: Track performance and failure rates
6. **Quality gates**: Prevent bad code from reaching production
7. **Environment parity**: Keep all environments as similar as possible
8. **Rollback strategy**: Always have a way to rollback deployments
9. **Documentation**: Keep pipeline configuration well-documented
10. **Continuous improvement**: Regularly review and optimize pipelines

### **Next Steps**
- Learn advanced GitHub Actions features
- Implement Kubernetes deployments
- Set up canary deployments
- Learn infrastructure as code
- Explore serverless deployment options

---

## 🎯 **Assignment**

### **Task 1: Basic CI Pipeline**
Create a GitHub Actions workflow that:
- Runs on push and pull requests
- Installs dependencies and caches them
- Runs linting and type checking
- Executes unit tests with coverage
- Builds the application
- Uploads build artifacts

### **Task 2: Advanced CI/CD Pipeline**
Extend the basic pipeline to include:
- Matrix builds for multiple Node.js versions
- Integration tests
- Security scanning with npm audit
- Code quality checks with ESLint and Prettier
- Automated deployment to staging environment
- Slack notifications for build status

### **Task 3: Multi-Platform Build System**
Implement a complete build system that:
- Builds both iOS and Android applications
- Uses Fastlane for mobile deployments
- Implements blue-green deployment strategy
- Includes environment-specific configurations
- Sets up proper secrets management
- Creates deployment dashboards

---

## 📚 **Additional Resources**
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Fastlane Documentation](https://docs.fastlane.tools/)
- [Jest Testing Framework](https://jestjs.io/)
- [ESLint Configuration](https://eslint.org/docs/user-guide/configuring/)
- [Prettier Code Formatter](https://prettier.io/)
- [Husky Git Hooks](https://typicode.github.io/husky/)
- [Codecov Coverage Reports](https://about.codecov.io/)

---

*This lesson provides comprehensive coverage of CI/CD pipeline setup for React Native applications, with practical examples and best practices for automated development workflows.*
      period: {
