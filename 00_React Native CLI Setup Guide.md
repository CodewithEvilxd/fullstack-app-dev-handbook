# 📱 React Native CLI Setup Guide
## Complete Installation & Development Manual

---

## 🎯 **Overview**
React Native CLI is the official command-line interface for React Native development. It provides tools for creating, building, and managing React Native projects. This guide covers complete CLI setup, usage, and best practices.

---

## 📋 **Table of Contents**
1. [What is React Native CLI?](#what-is-react-native-cli)
2. [System Requirements](#system-requirements)
3. [Installation & Setup](#installation--setup)
4. [Project Creation](#project-creation)
5. [Development Commands](#development-commands)
6. [Build Commands](#build-commands)
7. [Device Management](#device-management)
8. [Troubleshooting](#troubleshooting)
9. [Advanced Usage](#advanced-usage)
10. [Best Practices](#best-practices)

---

## 🤔 **What is React Native CLI?**

### **CLI Components:**
- **@react-native-community/cli**: Core CLI functionality
- **react-native**: Global CLI command
- **Platform-specific tools**: iOS and Android build tools
- **Development server**: Metro bundler integration
- **Code generation**: Template and boilerplate generation

### **Key Features:**
```bash
✅ Project scaffolding and templates
✅ Platform-specific build commands
✅ Development server management
✅ Device and emulator management
✅ Code generation and automation
✅ Plugin and module management
✅ Debugging and profiling tools
```

### **CLI vs Expo:**
```bash
React Native CLI:
✅ Full control over native code
✅ Custom native modules
✅ Advanced platform features
✅ Bare workflow flexibility
❌ More complex setup
❌ Manual native configuration

Expo:
✅ Simplified setup
✅ Managed workflow
✅ Pre-built modules
✅ Easy deployment
❌ Limited native customization
❌ Vendor lock-in concerns
```

---

## 💻 **System Requirements**

### **Cross-Platform Requirements:**
- **Node.js**: 16.0.0 or later
- **npm**: 7.0.0 or later (or Yarn 1.22+)
- **Git**: Latest version
- **Watchman** (recommended): For better file watching

### **Android Development:**
- **Java Development Kit (JDK)**: 11 or 17
- **Android Studio**: Latest stable version
- **Android SDK**: API 31+ (Android 12.0)
- **Android SDK Build-Tools**: 31.0.0+
- **Android Emulator**: Latest version

### **iOS Development (macOS only):**
- **Xcode**: 13.0 or later
- **iOS Simulator**: Included with Xcode
- **Command Line Tools**: `xcode-select --install`

### **Windows Requirements:**
```bash
# Windows 10/11 Pro/Enterprise/Education
# Enable Windows Subsystem for Linux (WSL2) for better development experience
# Install Ubuntu from Microsoft Store
```

### **macOS Requirements:**
```bash
# macOS 10.15 (Catalina) or later
# Xcode Command Line Tools
# Homebrew (recommended for package management)
```

### **Linux Requirements:**
```bash
# Ubuntu 18.04+ or equivalent
# Build tools: make, gcc, g++
# Python 3.x
# curl, unzip
```

---

## 📥 **Installation & Setup**

### **Step 1: Install Node.js**
```bash
# Using Node Version Manager (recommended)
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Restart terminal or run:
source ~/.bashrc

# Install latest LTS version
nvm install --lts
nvm use --lts

# Verify installation
node --version
npm --version
```

### **Step 2: Install React Native CLI**
```bash
# Install React Native CLI globally
npm install -g @react-native-community/cli

# Or install react-native globally (alternative)
npm install -g react-native-cli

# Verify installation
npx react-native --version
# or
react-native --version
```

### **Step 3: Install Watchman (Recommended)**
```bash
# macOS
brew install watchman

# Ubuntu/Debian
sudo apt-get install watchman

# Windows (using Chocolatey)
choco install watchman

# Verify installation
watchman --version
```

### **Step 4: Android Setup (All Platforms)**
```bash
# Download Android Studio
# Visit: https://developer.android.com/studio

# Install Android SDK
# Android Studio → SDK Manager → SDK Platforms
# Install:
# - Android 13.0 (API 33)
# - Android 12.0 (API 31)
# - Android 11.0 (API 30)

# Install SDK Tools
# SDK Manager → SDK Tools
# Install:
# - Android SDK Build-Tools 33.0.0
# - Android SDK Command-line Tools
# - Android Emulator
# - Android SDK Platform-Tools
```

### **Step 5: iOS Setup (macOS only)**
```bash
# Install Xcode from App Store
# Or download from: https://developer.apple.com/xcode/

# Install Command Line Tools
xcode-select --install

# Accept Xcode license
sudo xcodebuild -license accept

# Install CocoaPods (for iOS dependencies)
sudo gem install cocoapods

# Verify installations
xcodebuild -version
pod --version
```

### **Step 6: Environment Variables**
```bash
# Windows - System Environment Variables
# Control Panel → System → Advanced system settings → Environment Variables

# Add to PATH:
# C:\Users\[USERNAME]\AppData\Local\Android\Sdk\platform-tools
# C:\Users\[USERNAME]\AppData\Local\Android\Sdk\tools
# C:\Users\[USERNAME]\AppData\Local\Android\Sdk\tools\bin

# Set ANDROID_HOME:
# ANDROID_HOME = C:\Users\[USERNAME]\AppData\Local\Android\Sdk

# macOS/Linux - Add to ~/.bashrc or ~/.zshrc
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

# Reload profile
source ~/.bashrc
```

---

## 🚀 **Project Creation**

### **Creating New Projects**
```bash
# Create basic React Native project
npx react-native init MyApp

# Create project with specific version
npx react-native init MyApp --version 0.70.0

# Create TypeScript project
npx react-native init MyApp --template react-native-template-typescript

# Create project with custom template
npx react-native init MyApp --template https://github.com/my-template.git
```

### **Project Structure**
```bash
MyReactNativeApp/
├── android/                    # Android native code
│   ├── app/
│   ├── build.gradle
│   └── gradle.properties
├── ios/                       # iOS native code
│   ├── MyApp/
│   ├── MyApp.xcodeproj/
│   └── Podfile
├── node_modules/              # Dependencies
├── src/                       # Source code
│   ├── components/
│   ├── screens/
│   └── utils/
├── index.js                   # Entry point
├── package.json
├── metro.config.js            # Metro bundler config
├── babel.config.js            # Babel configuration
└── tsconfig.json             # TypeScript config (if used)
```

### **Custom Project Templates**
```bash
# Create custom template
npx react-native template

# Use community templates
npx react-native init MyApp --template react-native-template-navigation
npx react-native init MyApp --template react-native-template-redux
npx react-native init MyApp --template react-native-template-hooks
```

---

## 💻 **Development Commands**

### **Starting Development Server**
```bash
# Start Metro bundler
npx react-native start

# Start with specific options
npx react-native start --port 8082
npx react-native start --host 192.168.1.100
npx react-native start --reset-cache
npx react-native start --verbose

# Start with custom config
npx react-native start --config metro.config.js
```

### **Running on Devices/Emulators**
```bash
# Run on Android device/emulator
npx react-native run-android

# Run on iOS simulator
npx react-native run-ios

# Run on specific iOS simulator
npx react-native run-ios --simulator="iPhone 14"

# Run on Android with specific device
npx react-native run-android --deviceId=emulator-5554

# Run with clean build
npx react-native run-android --clean
```

### **Development Utilities**
```bash
# Check React Native version
npx react-native --version

# Display project info
npx react-native info

# Clean project cache
npx react-native start --reset-cache
cd android && ./gradlew clean
cd ios && rm -rf build

# Generate debug APK
cd android && ./gradlew assembleDebug

# Install dependencies
npm install
cd ios && pod install
```

### **Code Generation**
```bash
# Generate component
npx react-native generate component MyComponent

# Generate screen
npx react-native generate screen HomeScreen

# Generate service
npx react-native generate service ApiService

# Generate hook
npx react-native generate hook useAuth
```

---

## 🏗️ **Build Commands**

### **Android Build Commands**
```bash
# Navigate to android directory
cd android

# Clean build
./gradlew clean

# Debug build
./gradlew assembleDebug

# Release build
./gradlew assembleRelease

# Install debug APK
./gradlew installDebug

# Build bundle for Play Store
./gradlew bundleRelease

# Run tests
./gradlew test

# Lint code
./gradlew lint
```

### **iOS Build Commands**
```bash
# Navigate to ios directory
cd ios

# Install CocoaPods
pod install

# Clean build
rm -rf build
xcodebuild clean

# Build for simulator
xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -sdk iphonesimulator -configuration Debug

# Build for device
xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -sdk iphoneos -configuration Release

# Archive for App Store
xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -sdk iphoneos -configuration Release archive -archivePath ./build/MyApp.xcarchive
```

### **Cross-Platform Build Scripts**
```json
// package.json
{
  "scripts": {
    "android": "react-native run-android",
    "ios": "react-native run-ios",
    "start": "react-native start",
    "test": "jest",
    "lint": "eslint .",
    "clean": "react-native-clean-project",
    "build:android": "cd android && ./gradlew assembleRelease",
    "build:ios": "cd ios && xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release",
    "postinstall": "cd ios && pod install"
  }
}
```

---

## 📱 **Device Management**

### **Android Device Management**
```bash
# List connected devices
adb devices

# Connect to device over WiFi
adb tcpip 5555
adb connect 192.168.1.100:5555

# Install APK
adb install -r android/app/build/outputs/apk/debug/app-debug.apk

# Uninstall app
adb uninstall com.myapp

# View device logs
adb logcat

# Take screenshot
adb shell screencap -p /sdcard/screen.png
adb pull /sdcard/screen.png

# Record screen
adb shell screenrecord /sdcard/demo.mp4
adb pull /sdcard/demo.mp4
```

### **iOS Device Management**
```bash
# List connected devices
xcrun xctrace list devices

# Install app on device
xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -destination generic/platform=iOS install

# View device logs
xcrun simctl spawn booted log stream --level debug

# Take screenshot
xcrun simctl io booted screenshot screenshot.png

# Record screen
xcrun simctl io booted recordVideo video.mp4
```

### **Emulator Management**
```bash
# Android Emulator
# List available emulators
emulator -list-avds

# Start specific emulator
emulator -avd Pixel_6_API_33

# Start with options
emulator -avd Pixel_6_API_33 -no-boot-anim -gpu swiftshader

# iOS Simulator
# List available simulators
xcrun simctl list devices

# Start specific simulator
xcrun simctl boot "iPhone 14"

# Open simulator
open /Applications/Xcode.app/Contents/Developer/Applications/Simulator.app
```

---

## 🔧 **Troubleshooting**

### **Common Issues & Solutions**

#### **Issue 1: Metro Bundler Issues**
```bash
# Port already in use
npx react-native start --port 8082

# Clear cache
npx react-native start --reset-cache

# Kill Metro process
pkill -f "metro"
pkill -f "react-native"

# Check Metro config
# metro.config.js should exist and be valid
```

#### **Issue 2: Android Build Failures**
```bash
# Clean Gradle cache
cd android && ./gradlew clean

# Clear React Native cache
npx react-native start --reset-cache

# Update Gradle wrapper
cd android && ./gradlew wrapper --gradle-version=7.6

# Check Java version
java -version
javac -version

# Verify Android SDK
echo $ANDROID_HOME
ls $ANDROID_HOME/platforms/
```

#### **Issue 3: iOS Build Failures**
```bash
# Clean CocoaPods
cd ios && rm -rf Pods Podfile.lock
pod install

# Clean Xcode build
cd ios && rm -rf build
xcodebuild clean

# Reset Xcode
# Xcode → Product → Clean Build Folder

# Check Xcode version
xcodebuild -version
```

#### **Issue 4: Device Connection Issues**
```bash
# Android device not recognized
# Enable USB debugging in developer options
# Accept RSA key on device
adb kill-server
adb start-server

# iOS device not recognized
# Trust computer on device
# Check provisioning profile
# Verify bundle identifier
```

#### **Issue 5: Dependency Issues**
```bash
# Clear node_modules
rm -rf node_modules package-lock.json
npm install

# Clear Metro cache
npx react-native start --reset-cache

# Clear Watchman cache
watchman watch-del-all

# Reinstall CocoaPods
cd ios && pod deintegrate && pod install
```

### **Performance Issues**
```bash
# Enable Hermes engine (Android)
# android/app/build.gradle
project.ext.react = [
    enableHermes: true,
]

# Optimize Metro config
# metro.config.js
module.exports = {
  transformer: {
    enableBabelRCLookup: true,
    minifierConfig: {
      compress: {
        drop_console: true,
      },
    },
  },
};
```

---

## 🚀 **Advanced Usage**

### **Custom CLI Commands**
```javascript
// Create custom CLI script
#!/usr/bin/env node

const { execSync } = require('child_process');
const fs = require('fs');
const path = require('path');

function createComponent(name) {
  const componentDir = path.join('src', 'components', name);
  const componentFile = path.join(componentDir, `${name}.js`);

  // Create directory
  fs.mkdirSync(componentDir, { recursive: true });

  // Create component file
  const componentCode = `
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

const ${name} = () => {
  return (
    <View style={styles.container}>
      <Text>${name} Component</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default ${name};
`;

  fs.writeFileSync(componentFile, componentCode);
  console.log(`Component ${name} created successfully!`);
}

// CLI interface
const [,, command, ...args] = process.argv;

switch (command) {
  case 'component':
    createComponent(args[0]);
    break;
  default:
    console.log('Usage: custom-cli component <name>');
}
```

### **Automated Build Scripts**
```bash
#!/bin/bash
# build.sh

set -e

echo "🚀 Starting React Native build process..."

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

# Clean previous builds
log "Cleaning previous builds..."
cd android && ./gradlew clean
cd ..
rm -rf ios/build

# Install dependencies
log "Installing dependencies..."
npm install
cd ios && pod install
cd ..

# Run tests
log "Running tests..."
npm test

# Build Android
log "Building Android..."
cd android
./gradlew assembleRelease
cd ..

# Build iOS
log "Building iOS..."
cd ios
xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release
cd ..

log "✅ Build completed successfully!"
```

### **CI/CD Integration**
```yaml
# .github/workflows/build.yml
name: Build and Test

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

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

      - name: Run tests
        run: npm test

      - name: Build Android
        run: |
          cd android
          chmod +x gradlew
          ./gradlew assembleDebug

  build-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Java
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '11'

      - name: Build Android Release
        run: |
          cd android
          chmod +x gradlew
          ./gradlew assembleRelease

  build-ios:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: |
          npm ci
          cd ios && pod install

      - name: Build iOS
        run: |
          cd ios
          xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release -sdk iphoneos
```

---

## 📚 **Best Practices**

### **Project Organization**
```bash
# Recommended structure
MyReactNativeApp/
├── src/
│   ├── components/          # Reusable components
│   ├── screens/            # Screen components
│   ├── navigation/         # Navigation configuration
│   ├── services/           # API services
│   ├── utils/              # Utility functions
│   ├── constants/          # App constants
│   ├── hooks/              # Custom hooks
│   └── types/              # TypeScript types
├── android/                # Android native code
├── ios/                   # iOS native code
├── __tests__/             # Test files
├── scripts/               # Build and utility scripts
├── assets/                # Images, fonts, etc.
├── index.js               # Entry point
└── package.json
```

### **Script Organization**
```json
// package.json
{
  "scripts": {
    "start": "react-native start",
    "android": "react-native run-android",
    "ios": "react-native run-ios",
    "test": "jest",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "clean": "react-native-clean-project",
    "build:android": "cd android && ./gradlew assembleRelease",
    "build:ios": "cd ios && xcodebuild -workspace MyApp.xcworkspace -scheme MyApp -configuration Release",
    "postinstall": "cd ios && pod install",
    "bundle:android": "react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res",
    "bundle:ios": "react-native bundle --platform ios --dev false --entry-file index.js --bundle-output ios/main.jsbundle --assets-dest ios"
  }
}
```

### **Environment Management**
```javascript
// config/index.js
const config = {
  development: {
    API_URL: 'http://localhost:3000/api',
    DEBUG: true,
  },
  staging: {
    API_URL: 'https://api-staging.myapp.com',
    DEBUG: true,
  },
  production: {
    API_URL: 'https://api.myapp.com',
    DEBUG: false,
  },
};

const env = process.env.NODE_ENV || 'development';
export default config[env];
```

### **Version Management**
```json
// package.json
{
  "version": "1.0.0",
  "react-native": "0.70.0",
  "scripts": {
    "version:patch": "npm version patch",
    "version:minor": "npm version minor",
    "version:major": "npm version major",
    "postversion": "react-native-version --never-increment-build-number"
  }
}
```

---

## 🔐 **Security Best Practices**

### **Secure Build Configuration**
```gradle
// android/app/build.gradle
android {
    buildTypes {
        debug {
            debuggable true
            minifyEnabled false
        }
        release {
            debuggable false
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }
    }
}
```

### **Environment Variables**
```bash
# .env file (add to .gitignore)
API_KEY=your-api-key
SECRET_KEY=your-secret-key
DATABASE_URL=your-database-url

# Load in React Native
import Config from 'react-native-config';

// Use in code
const apiKey = Config.API_KEY;
```

### **Code Signing**
```gradle
// android/app/build.gradle
android {
    signingConfigs {
        release {
            storeFile file('release.keystore')
            storePassword System.getenv('STORE_PASSWORD')
            keyAlias System.getenv('KEY_ALIAS')
            keyPassword System.getenv('KEY_PASSWORD')
        }
    }
}
```

---

## 📊 **Monitoring & Analytics**

### **Build Monitoring**
```bash
# Monitor build times
time npm run android
time npm run ios

# Check bundle size
npx react-native bundle --platform android --dev false --entry-file index.js --bundle-output bundle.js --sourcemap-output bundle.map

# Analyze dependencies
npx npm-check
npx depcheck
```

### **Performance Monitoring**
```javascript
// Add performance monitoring
import { PerformanceMonitor } from 'react-native';

PerformanceMonitor.start();

// Monitor specific operations
const startTime = performance.now();
// ... your code ...
const endTime = performance.now();
console.log(`Operation took ${endTime - startTime} milliseconds`);
```

---

## 🎯 **Conclusion**

React Native CLI is a powerful tool for building and managing React Native applications. It provides complete control over the development process and native platform integration.

### **Key Takeaways:**
- ✅ Install and configure all required dependencies
- ✅ Use CLI commands for efficient development workflow
- ✅ Set up proper build configurations for both platforms
- ✅ Implement automated build and deployment scripts
- ✅ Follow best practices for project organization
- ✅ Monitor performance and optimize builds

### **Next Steps:**
1. **Setup**: Configure your development environment
2. **Create**: Build your first React Native project
3. **Develop**: Use CLI commands for efficient workflow
4. **Build**: Set up automated build pipelines
5. **Deploy**: Configure app store deployments
6. **Scale**: Implement advanced build optimizations

### **Additional Resources:**
- [React Native CLI Documentation](https://github.com/react-native-community/cli)
- [React Native Environment Setup](https://reactnative.dev/docs/environment-setup)
- [Android Development](https://developer.android.com/)
- [iOS Development](https://developer.apple.com/ios/)
- [React Native Community](https://reactnative.dev/community/overview)

---

*Happy React Native Development! 📱⚛️*