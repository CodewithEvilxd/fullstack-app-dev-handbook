# 🟢 Node.js Setup Guide for React Native
## Complete Installation & Configuration Manual

---

## 🎯 **Overview**
Node.js is the runtime environment for React Native development. This guide covers complete Node.js setup, npm configuration, and optimization for React Native projects.

---

## 📋 **Table of Contents**
1. [What is Node.js?](#what-is-nodejs)
2. [Version Requirements](#version-requirements)
3. [Installation Methods](#installation-methods)
4. [NPM Configuration](#npm-configuration)
5. [Yarn Setup](#yarn-setup)
6. [Environment Setup](#environment-setup)
7. [Performance Optimization](#performance-optimization)
8. [Troubleshooting](#troubleshooting)
9. [Best Practices](#best-practices)

---

## 🤔 **What is Node.js?**

### **Node.js in React Native Context:**
- **JavaScript Runtime**: Executes JavaScript outside browser
- **NPM Package Manager**: Manages project dependencies
- **Build Tools**: Runs build scripts and development servers
- **Metro Bundler**: Powers React Native's bundling system
- **CLI Tools**: Provides command-line development tools

### **Key Components:**
```bash
✅ Node.js Runtime Engine (V8)
✅ NPM (Node Package Manager)
✅ NPX (NPM Package Runner)
✅ Core Modules (fs, path, http, etc.)
✅ Event-Driven Architecture
✅ Non-Blocking I/O
```

### **Why Node.js for React Native:**
```bash
🚀 Fast JavaScript execution
📦 Rich ecosystem of packages
🛠️ Powerful build tools
🔧 Easy dependency management
⚡ Hot reloading capabilities
📱 Cross-platform development
```

---

## 📊 **Version Requirements**

### **React Native Version Compatibility:**

| React Native Version | Node.js Version | NPM Version | Status |
|---------------------|----------------|-------------|---------|
| 0.70+ | 16.0.0 - 18.x | 7.0.0+ | ✅ Recommended |
| 0.69 | 14.0.0 - 16.x | 6.0.0+ | ⚠️ Legacy |
| 0.68 | 14.0.0 - 16.x | 6.0.0+ | ⚠️ Legacy |
| 0.67 | 12.0.0 - 17.x | 6.0.0+ | ❌ Not supported |

### **Recommended Versions:**
```bash
✅ Node.js: 18.x LTS (Latest LTS)
✅ NPM: 8.x or 9.x (Latest)
✅ Yarn: 1.22+ (Optional but recommended)
```

### **Platform-Specific Considerations:**
```bash
# Windows
✅ Node.js 18.x LTS
✅ Use Windows Terminal or PowerShell
✅ Consider WSL2 for better performance

# macOS
✅ Node.js 18.x LTS
✅ Use Terminal or iTerm2
✅ Homebrew for package management

# Linux
✅ Node.js 18.x LTS
✅ Use system package manager
✅ Consider NVM for version management
```

---

## 📥 **Installation Methods**

### **Method 1: Node Version Manager (NVM) - Recommended**

#### **Install NVM:**
```bash
# Linux/macOS
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Windows
# Download nvm-windows from GitHub
# https://github.com/coreybutler/nvm-windows/releases

# Restart terminal or run:
source ~/.bashrc  # Linux/macOS
# Windows: Restart command prompt
```

#### **Install Node.js with NVM:**
```bash
# Install latest LTS version
nvm install --lts

# Install specific version
nvm install 18.17.0

# List installed versions
nvm list

# Use specific version
nvm use 18.17.0

# Set default version
nvm alias default 18.17.0

# Verify installation
node --version
npm --version
```

### **Method 2: Official Installer**

#### **Download and Install:**
```bash
# Visit official website
https://nodejs.org/

# Download LTS version for your platform
# Run installer with default settings

# Windows: .msi installer
# macOS: .pkg installer
# Linux: .tar.xz or distribution package
```

#### **Verify Installation:**
```bash
# Check versions
node --version
npm --version

# Check npm configuration
npm config list

# Test basic functionality
node -e "console.log('Hello Node.js!')"
```

### **Method 3: Package Managers**

#### **Windows (Chocolatey):**
```bash
# Install Chocolatey first
# https://chocolatey.org/

# Install Node.js
choco install nodejs

# Install specific version
choco install nodejs --version=18.17.0
```

#### **macOS (Homebrew):**
```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Node.js
brew install node

# Install specific version
brew install node@18
```

#### **Ubuntu/Debian:**
```bash
# Using NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version
npm --version
```

#### **CentOS/RHEL/Fedora:**
```bash
# Using NodeSource repository
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs  # CentOS/RHEL
sudo dnf install -y nodejs  # Fedora
```

### **Method 4: Docker (For Development)**
```dockerfile
# Dockerfile
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Start application
CMD ["npm", "start"]
```

---

## ⚙️ **NPM Configuration**

### **Basic NPM Setup:**
```bash
# Check current configuration
npm config list

# Set global configurations
npm config set init-author-name "Your Name"
npm config set init-author-email "your.email@example.com"
npm config set init-license "MIT"

# Set registry (if using private registry)
npm config set registry https://registry.npmjs.org/

# Set proxy (if behind corporate proxy)
npm config set proxy http://proxy.company.com:8080
npm config set https-proxy http://proxy.company.com:8080
```

### **NPM Performance Optimization:**
```bash
# Increase network timeout
npm config set fetch-timeout 60000

# Enable progress bar
npm config set progress true

# Set cache settings
npm config set cache-max 1000000
npm config set cache-min 10000

# Enable fund messages (optional)
npm config set fund false

# Enable audit on install
npm config set audit true
```

### **NPM Scripts Configuration:**
```json
// package.json
{
  "name": "my-react-native-app",
  "version": "1.0.0",
  "scripts": {
    "start": "react-native start",
    "android": "react-native run-android",
    "ios": "react-native run-ios",
    "test": "jest",
    "lint": "eslint .",
    "clean": "react-native-clean-project",
    "postinstall": "cd ios && pod install"
  }
}
```

### **NPM Cache Management:**
```bash
# Clear npm cache
npm cache clean --force

# Verify cache integrity
npm cache verify

# View cache location
npm config get cache

# Set custom cache location
npm config set cache /path/to/custom/cache
```

---

## 🧶 **Yarn Setup**

### **Install Yarn:**
```bash
# Using npm
npm install -g yarn

# Using corepack (Node.js 16.10+)
corepack enable
corepack prepare yarn@stable --activate

# Verify installation
yarn --version
```

### **Yarn Configuration:**
```bash
# Set Yarn version
yarn set version stable

# Configure Yarn
yarn config set init-author-name "Your Name"
yarn config set init-author-email "your.email@example.com"

# Set registry
yarn config set registry https://registry.yarnpkg.com

# Enable PnP (Plug'n'Play)
yarn config set nodeLinker pnp

# Set cache folder
yarn config set cacheFolder /path/to/yarn/cache
```

### **Yarn vs NPM Comparison:**
```bash
NPM:
✅ Built-in with Node.js
✅ More familiar to developers
✅ Better for small projects
❌ Slower installation
❌ Nested node_modules

Yarn:
✅ Faster installation
✅ Better dependency resolution
✅ Workspaces support
✅ Plug'n'Play mode
❌ Separate installation required
❌ Less familiar syntax
```

### **Yarn Commands:**
```bash
# Install dependencies
yarn install
yarn

# Add packages
yarn add react-native-vector-icons
yarn add -D @types/react

# Remove packages
yarn remove react-native-vector-icons

# Run scripts
yarn start
yarn android
yarn ios

# Clean cache
yarn cache clean

# Upgrade packages
yarn upgrade
yarn upgrade react-native
```

---

## 🌍 **Environment Setup**

### **Environment Variables:**
```bash
# Linux/macOS (.bashrc or .zshrc)
export NODE_ENV=development
export REACT_NATIVE_PACKAGER_HOSTNAME=localhost
export ANDROID_HOME=$HOME/Android/Sdk
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

# Windows (System Environment Variables)
# Control Panel → System → Advanced system settings → Environment Variables

# Add to User Variables:
NODE_ENV = development
REACT_NATIVE_PACKAGER_HOSTNAME = localhost

# Add to System Variables PATH:
%USERPROFILE%\AppData\Roaming\npm
C:\Users\%USERNAME%\AppData\Local\Android\Sdk\platform-tools
```

### **NVM Configuration:**
```bash
# .nvmrc file in project root
18.17.0

# Auto-switch Node version
# Add to .bashrc or .zshrc
autoload -U add-zsh-hook
load-nvmrc() {
  local node_version="$(nvm version)"
  local nvmrc_path="$(nvm_find_nvmrc)"

  if [ -n "$nvmrc_path" ]; then
    local nvmrc_node_version=$(nvm version "$(cat "${nvmrc_path}")")

    if [ "$nvmrc_node_version" = "N/A" ]; then
      nvm install
    elif [ "$nvmrc_node_version" != "$node_version" ]; then
      nvm use
    fi
  elif [ "$node_version" != "$(nvm version default)" ]; then
    echo "Reverting to nvm default version"
    nvm use default
  fi
}
add-zsh-hook chpwd load-nvmrc
load-nvmrc
```

### **Project-Specific Setup:**
```json
// .nvmrc
18.17.0

// package.json
{
  "engines": {
    "node": ">=16.0.0",
    "npm": ">=7.0.0",
    "yarn": ">=1.22.0"
  }
}
```

---

## ⚡ **Performance Optimization**

### **NPM Performance Tips:**
```bash
# Use npm ci for production builds
npm ci

# Enable package-lock.json
npm install --package-lock-only

# Use npm cache
npm install --prefer-offline

# Parallel installations
npm install --maxsockets 1  # Reduce for slow networks
npm install --maxsockets 5  # Increase for fast networks
```

### **Yarn Performance Tips:**
```bash
# Use Yarn PnP
yarn config set nodeLinker pnp

# Enable cache
yarn install --cache-folder /path/to/cache

# Use frozen lockfile
yarn install --frozen-lockfile

# Parallel processing
yarn config set networkConcurrency 8
```

### **Node.js Performance Tuning:**
```bash
# Increase memory limit
export NODE_OPTIONS="--max-old-space-size=4096"

# Enable garbage collection tuning
export NODE_OPTIONS="--optimize-for-size --max-old-space-size=4096"

# Use latest V8 flags (experimental)
export NODE_OPTIONS="--harmony --harmony-sloppy"
```

### **Metro Bundler Optimization:**
```javascript
// metro.config.js
module.exports = {
  transformer: {
    enableBabelRCLookup: true,
    minifierConfig: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
  },
  cacheStores: [
    {
      type: 'file',
      cacheDirectory: './.metro-cache',
    },
  ],
  maxWorkers: 4,
};
```

---

## 🔧 **Troubleshooting**

### **Common Issues & Solutions**

#### **Issue 1: Permission Errors**
```bash
# Fix npm permission issues
sudo chown -R $(whoami) ~/.npm
sudo chown -R $(whoami) /usr/local/lib/node_modules

# Or use nvm to avoid permission issues
nvm install --lts
nvm use --lts
```

#### **Issue 2: Version Conflicts**
```bash
# Check current versions
node --version
npm --version

# Update npm
npm install -g npm@latest

# Clear npm cache
npm cache clean --force

# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install
```

#### **Issue 3: Network Issues**
```bash
# Set npm registry
npm config set registry https://registry.npmjs.org/

# Use different registry
npm config set registry https://registry.npmmirror.com

# Set network timeout
npm config set fetch-timeout 60000

# Use proxy
npm config set proxy http://proxy.company.com:8080
```

#### **Issue 4: Memory Issues**
```bash
# Increase Node.js memory limit
export NODE_OPTIONS="--max-old-space-size=4096"

# For specific commands
NODE_OPTIONS="--max-old-space-size=4096" npm run build

# Check memory usage
node -e "console.log(process.memoryUsage())"
```

#### **Issue 5: Path Issues**
```bash
# Check PATH
echo $PATH

# Add npm to PATH (Linux/macOS)
export PATH=$PATH:$(npm config get prefix)/bin

# Add npm to PATH (Windows)
# Add %APPDATA%\npm to PATH environment variable
```

### **Debugging Node.js Issues:**
```bash
# Enable debug mode
NODE_DEBUG=cluster,net,http,fs,tls,module,timers node app.js

# Use node inspector
node --inspect app.js

# Generate heap dump
node --heap-prof app.js

# Check for memory leaks
node --expose-gc --inspect app.js
```

---

## 📚 **Best Practices**

### **Version Management:**
```bash
# Use .nvmrc for project-specific versions
echo "18.17.0" > .nvmrc

# Use engines in package.json
{
  "engines": {
    "node": ">=16.0.0 <19.0.0",
    "npm": ">=7.0.0"
  }
}

# Use volta for automatic version switching
# https://volta.sh/
```

### **Dependency Management:**
```json
// package.json best practices
{
  "name": "my-react-native-app",
  "version": "1.0.0",
  "private": true,
  "engines": {
    "node": ">=16.0.0",
    "npm": ">=7.0.0"
  },
  "scripts": {
    "preinstall": "npx only-allow npm",
    "postinstall": "cd ios && pod install",
    "clean": "rm -rf node_modules && npm install"
  }
}
```

### **Security Best Practices:**
```bash
# Audit dependencies
npm audit
yarn audit

# Fix security issues
npm audit fix
yarn audit fix

# Use npm ci for production
npm ci --production

# Avoid sudo with npm
# Use nvm or proper permissions instead
```

### **Performance Monitoring:**
```bash
# Monitor npm install time
time npm install

# Check bundle size
npx react-native bundle --platform android --dev false --entry-file index.js --bundle-output bundle.js

# Analyze dependencies
npx npm-check
npx depcheck

# Monitor memory usage
node -e "console.log(process.memoryUsage())"
```

---

## 🚀 **Advanced Configuration**

### **Custom NPM Scripts:**
```json
// package.json
{
  "scripts": {
    "dev": "react-native start --reset-cache",
    "build:android": "cd android && ./gradlew assembleRelease",
    "build:ios": "cd ios && xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint:fix": "eslint . --fix",
    "clean:all": "rm -rf node_modules ios/build android/app/build",
    "doctor": "react-native doctor",
    "bundle:visualize": "npx react-native-bundle-visualizer"
  }
}
```

### **NPM Hooks:**
```json
// package.json
{
  "scripts": {
    "prepare": "husky install",
    "precommit": "lint-staged",
    "prepush": "npm run test"
  }
}
```

### **Custom Registry Setup:**
```bash
# Set up private registry
npm config set registry https://npm.mycompany.com/

# Authenticate
npm login --registry https://npm.mycompany.com/

# Use scoped packages
npm install @mycompany/ui-components
```

### **CI/CD Configuration:**
```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run test
      - run: npm run lint
```

---

## 📊 **Monitoring & Analytics**

### **NPM Usage Analytics:**
```bash
# View npm usage
npm config get

# Check installed packages
npm list -g --depth=0

# View package info
npm view react-native

# Check outdated packages
npm outdated

# Update packages
npm update
```

### **Performance Metrics:**
```bash
# Measure install time
time npm install

# Check bundle size
npx bundlesize

# Analyze dependencies
npx cost-of-modules

# Monitor memory usage
node --expose-gc -e "console.log(process.memoryUsage())"
```

### **Health Checks:**
```bash
# Run React Native health check
npx @react-native-community/cli doctor

# Check Node.js health
node -e "console.log('Node.js is working!')"

# Verify npm functionality
npm --version && npm ping
```

---

## 🔐 **Security Considerations**

### **Secure Package Management:**
```bash
# Enable npm audit
npm config set audit true

# Use npm audit regularly
npm audit
npm audit fix

# Use Snyk for security scanning
npx snyk test
npx snyk wizard
```

### **Environment Security:**
```bash
# Never commit sensitive data
# Use .env files (add to .gitignore)
/.env
.env.local
.env.production

# Use environment variables for secrets
process.env.API_KEY
process.env.DATABASE_URL
```

### **Package Verification:**
```bash
# Verify package integrity
npm verify

# Check package signatures (experimental)
npm config set fund false
npm config set audit true
```

---

## 🎯 **Conclusion**

Node.js is the foundation of React Native development. Proper setup and configuration are crucial for efficient development workflow and optimal performance.

### **Key Takeaways:**
- ✅ Install Node.js using NVM for version management
- ✅ Configure NPM for optimal performance
- ✅ Use Yarn for faster dependency management
- ✅ Set up proper environment variables
- ✅ Implement security best practices
- ✅ Monitor performance and health

### **Next Steps:**
1. **Install**: Set up Node.js and npm on your system
2. **Configure**: Optimize npm and yarn settings
3. **Create**: Start your first React Native project
4. **Develop**: Use npm scripts for efficient workflow
5. **Monitor**: Track performance and security
6. **Scale**: Implement advanced configurations for teams

### **Additional Resources:**
- [Node.js Official Documentation](https://nodejs.org/docs/)
- [NPM Documentation](https://docs.npmjs.com/)
- [Yarn Documentation](https://yarnpkg.com/getting-started)
- [NVM Documentation](https://github.com/nvm-sh/nvm)
- [React Native Environment Setup](https://reactnative.dev/docs/environment-setup)

---

*Happy Node.js Development! 🟢📦*