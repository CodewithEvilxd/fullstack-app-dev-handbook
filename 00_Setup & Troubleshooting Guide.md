# 🛠️ Setup & Troubleshooting Guide
## Complete React Native Development Environment

---

## 📋 Table of Contents

1. [System Requirements](#system-requirements)
2. [Step-by-Step Setup](#step-by-step-setup)
3. [Common Issues & Solutions](#common-issues--solutions)
4. [Performance Optimization](#performance-optimization)
5. [Development Tools](#development-tools)
6. [Best Practices](#best-practices)

---

## 💻 System Requirements

### Minimum Requirements
- **RAM**: 8GB (16GB recommended)
- **Storage**: 50GB free space
- **OS**: 
  - Windows 10/11 (64-bit)
  - macOS 10.15 or later
  - Ubuntu 18.04 LTS or later

### Software Requirements
- **Node.js**: 14.x or later (LTS recommended)
- **npm**: 6.x or later
- **Git**: Latest version
- **Code Editor**: VS Code (recommended)

---

## 🚀 Step-by-Step Setup

### Step 1: Install Node.js

#### Windows/Mac:
```bash
# Download from https://nodejs.org/
# Choose LTS version

# Verify installation
node --version
npm --version
```

#### Linux (Ubuntu):
```bash
# Using NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version
npm --version
```

### Step 2: Install React Native CLI

```bash
# Install globally
npm install -g @react-native-community/cli

# Verify installation
npx react-native --version
```

### Step 3: Android Development Setup

#### Install Java Development Kit (JDK)
```bash
# Windows (using Chocolatey)
choco install openjdk11

# Mac (using Homebrew)
brew install openjdk@11

# Linux (Ubuntu)
sudo apt install openjdk-11-jdk

# Verify installation
java -version
javac -version
```

#### Install Android Studio
1. Download from https://developer.android.com/studio
2. Run installer with default settings
3. Open Android Studio
4. Complete setup wizard
5. Install required SDK components

#### Configure Android SDK
1. Open Android Studio
2. Go to **Tools > SDK Manager**
3. Install the following:
   - **Android SDK Platform 31** (Android 12)
   - **Android SDK Platform 30** (Android 11)
   - **Android SDK Platform 29** (Android 10)
   - **Android SDK Build-Tools 31.0.0**
   - **Android Emulator**
   - **Android SDK Platform-Tools**

#### Set Environment Variables

**Windows:**
```cmd
# Add to System Environment Variables
ANDROID_HOME=C:\Users\%USERNAME%\AppData\Local\Android\Sdk
ANDROID_SDK_ROOT=C:\Users\%USERNAME%\AppData\Local\Android\Sdk

# Add to PATH
%ANDROID_HOME%\platform-tools
%ANDROID_HOME%\emulator
%ANDROID_HOME%\tools
%ANDROID_HOME%\tools\bin
```

**Mac/Linux:**
```bash
# Add to ~/.bash_profile or ~/.zshrc
export ANDROID_HOME=$HOME/Library/Android/sdk
export ANDROID_SDK_ROOT=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

# Reload shell
source ~/.bash_profile  # or ~/.zshrc
```

### Step 4: iOS Development Setup (Mac Only)

#### Install Xcode
```bash
# Install from Mac App Store
# Or download from Apple Developer Portal

# Install Command Line Tools
xcode-select --install

# Verify installation
xcode-select -p
```

#### Install CocoaPods
```bash
# Install CocoaPods
sudo gem install cocoapods

# Verify installation
pod --version
```

#### Setup iOS Simulator
1. Open Xcode
2. Go to **Xcode > Preferences > Components**
3. Download required iOS Simulators
4. Test simulator: **Xcode > Open Developer Tool > Simulator**

### Step 5: Create Your First Project

```bash
# Create new React Native project
npx react-native init MyFirstApp

# Navigate to project directory
cd MyFirstApp

# Start Metro bundler
npx react-native start

# Run on Android (new terminal)
npx react-native run-android

# Run on iOS (Mac only, new terminal)
npx react-native run-ios
```

---

## 🔧 Common Issues & Solutions

### Issue 1: Metro Bundler Not Starting

**Symptoms:**
- Metro bundler fails to start
- "Port 8081 already in use" error
- Bundle loading stuck

**Solutions:**
```bash
# Solution 1: Reset Metro cache
npx react-native start --reset-cache

# Solution 2: Kill process on port 8081
# Windows
netstat -ano | findstr :8081
taskkill /PID <PID> /F

# Mac/Linux
lsof -ti:8081 | xargs kill -9

# Solution 3: Use different port
npx react-native start --port 8082

# Solution 4: Clear npm cache
npm cache clean --force
```

### Issue 2: Android Build Failures

**Symptoms:**
- Build failed with ANDROID_HOME error
- SDK not found
- Gradle build errors

**Solutions:**
```bash
# Solution 1: Verify environment variables
echo $ANDROID_HOME  # Mac/Linux
echo %ANDROID_HOME%  # Windows

# Solution 2: Clean and rebuild
cd android
./gradlew clean  # Mac/Linux
gradlew clean    # Windows
cd ..
npx react-native run-android

# Solution 3: Update Gradle wrapper
cd android
./gradlew wrapper --gradle-version 7.4

# Solution 4: Clear React Native cache
npx react-native start --reset-cache
rm -rf node_modules
npm install
```

### Issue 3: iOS Build Failures (Mac Only)

**Symptoms:**
- Xcode build errors
- CocoaPods issues
- Simulator not found

**Solutions:**
```bash
# Solution 1: Clean iOS build
cd ios
rm -rf build
xcodebuild clean
cd ..

# Solution 2: Reinstall pods
cd ios
rm -rf Pods
rm Podfile.lock
pod install
cd ..

# Solution 3: Reset iOS Simulator
xcrun simctl erase all

# Solution 4: Update CocoaPods
sudo gem install cocoapods
cd ios
pod repo update
pod install
```

### Issue 4: Emulator/Simulator Issues

**Android Emulator:**
```bash
# List available AVDs
emulator -list-avds

# Start specific AVD
emulator -avd <AVD_NAME>

# Create new AVD
# Open Android Studio > AVD Manager > Create Virtual Device

# Check if emulator is detected
adb devices
```

**iOS Simulator:**
```bash
# List available simulators
xcrun simctl list devices

# Boot specific simulator
xcrun simctl boot <DEVICE_ID>

# Open Simulator app
open -a Simulator
```

### Issue 5: Node.js and npm Issues

**Symptoms:**
- npm install failures
- Permission errors
- Version conflicts

**Solutions:**
```bash
# Solution 1: Clear npm cache
npm cache clean --force

# Solution 2: Delete node_modules and reinstall
rm -rf node_modules
rm package-lock.json
npm install

# Solution 3: Fix npm permissions (Mac/Linux)
sudo chown -R $(whoami) ~/.npm

# Solution 4: Use yarn instead of npm
npm install -g yarn
yarn install

# Solution 5: Update Node.js and npm
# Download latest LTS from nodejs.org
npm install -g npm@latest
```

### Issue 6: React Native CLI Issues

**Symptoms:**
- Command not found
- Version conflicts
- Global vs local CLI issues

**Solutions:**
```bash
# Solution 1: Uninstall old CLI and install new one
npm uninstall -g react-native-cli
npm install -g @react-native-community/cli

# Solution 2: Use npx instead of global installation
npx react-native init MyApp
npx react-native run-android

# Solution 3: Clear npm cache
npm cache clean --force

# Solution 4: Check PATH variable
echo $PATH  # Mac/Linux
echo %PATH%  # Windows
```

### Issue 7: Debugging and Development Issues

**React DevTools:**
```bash
# Install React DevTools
npm install -g react-devtools

# Run React DevTools
react-devtools

# Enable in app (shake device or Cmd+D/Ctrl+M)
# Select "Debug with Chrome" or "Open React DevTools"
```

**Flipper Setup:**
```bash
# Download Flipper from https://fbflipper.com/
# Install and run Flipper
# Your React Native app should appear automatically
```

---

## ⚡ Performance Optimization

### Development Performance

```bash
# Enable Hermes (JavaScript engine)
# Edit android/app/build.gradle
project.ext.react = [
    enableHermes: true
]

# For iOS, edit ios/Podfile
use_react_native!(
  :path => config[:reactNativePath],
  :hermes_enabled => true
)
```

### Build Performance

```bash
# Increase Gradle memory
# Create/edit android/gradle.properties
org.gradle.jvmargs=-Xmx4096m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
org.gradle.parallel=true
org.gradle.configureondemand=true
org.gradle.daemon=true

# Enable multidex (if needed)
# In android/app/build.gradle
android {
    defaultConfig {
        multiDexEnabled true
    }
}
```

### Metro Configuration

```javascript
// metro.config.js
module.exports = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: true,
      },
    }),
  },
  resolver: {
    alias: {
      '@': './src',
    },
  },
};
```

---

## 🛠️ Development Tools

### Essential VS Code Extensions

```json
{
  "recommendations": [
    "ms-vscode.vscode-react-native",
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-eslint",
    "formulahendry.auto-rename-tag",
    "christian-kohler.path-intellisense",
    "ms-vscode.vscode-json"
  ]
}
```

### Useful npm Packages

```bash
# Development dependencies
npm install --save-dev @babel/core @babel/runtime
npm install --save-dev eslint prettier
npm install --save-dev @react-native-community/eslint-config

# Useful libraries
npm install react-navigation
npm install react-native-vector-icons
npm install react-native-async-storage
npm install react-native-image-picker
```

### Debugging Tools

```bash
# Reactotron (debugging tool)
npm install --save-dev reactotron-react-native

# React Native Debugger
# Download from GitHub releases
# https://github.com/jhen0409/react-native-debugger/releases
```

---

## 📱 Device Testing

### Physical Device Setup

**Android:**
1. Enable Developer Options
2. Enable USB Debugging
3. Connect device via USB
4. Run `adb devices` to verify connection
5. Run `npx react-native run-android`

**iOS:**
1. Connect device via USB
2. Open Xcode
3. Select your device as target
4. Build and run from Xcode

### Wireless Debugging

**Android:**
```bash
# Enable wireless debugging
adb tcpip 5555
adb connect <DEVICE_IP>:5555
adb devices  # Verify connection
```

**iOS:**
1. Connect device via USB once
2. In Xcode: Window > Devices and Simulators
3. Check "Connect via network"
4. Device will appear wirelessly

---

## 🔍 Troubleshooting Checklist

### Before Starting Development

- [ ] Node.js and npm installed and updated
- [ ] React Native CLI installed globally
- [ ] Android Studio installed with required SDKs
- [ ] Environment variables set correctly
- [ ] Emulator/Simulator working
- [ ] Git configured

### When Issues Occur

- [ ] Check error messages carefully
- [ ] Clear Metro cache: `npx react-native start --reset-cache`
- [ ] Clean and rebuild: `cd android && ./gradlew clean`
- [ ] Restart Metro bundler
- [ ] Restart emulator/simulator
- [ ] Check device/emulator connection: `adb devices`
- [ ] Update dependencies: `npm update`
- [ ] Check React Native version compatibility

### Performance Issues

- [ ] Enable Hermes JavaScript engine
- [ ] Optimize images and assets
- [ ] Use FlatList for large lists
- [ ] Implement proper key props
- [ ] Use React.memo for expensive components
- [ ] Profile with Flipper or React DevTools

---

## 📚 Additional Resources

### Official Documentation
- [React Native Docs](https://reactnative.dev/)
- [Android Developer Docs](https://developer.android.com/)
- [iOS Developer Docs](https://developer.apple.com/documentation/)

### Community Resources
- [React Native Community](https://github.com/react-native-community)
- [Awesome React Native](https://github.com/jondot/awesome-react-native)
- [React Native Elements](https://reactnativeelements.com/)

### Learning Platforms
- [React Native School](https://www.reactnativeschool.com/)
- [Expo Documentation](https://docs.expo.dev/)
- [React Navigation](https://reactnavigation.org/)

---

## 🆘 Getting Help

### When You're Stuck

1. **Read Error Messages**: Often contain exact solution
2. **Check Official Docs**: Most comprehensive and up-to-date
3. **Search Stack Overflow**: Common issues already solved
4. **GitHub Issues**: Check library-specific issues
5. **Community Forums**: React Native Community Discord/Reddit

### Reporting Issues

When reporting issues, include:
- React Native version
- Operating system and version
- Node.js and npm versions
- Complete error message
- Steps to reproduce
- Code snippets (minimal reproducible example)

---

## ✅ Success Indicators

You've successfully set up React Native when:

- [ ] `npx react-native --version` shows version info
- [ ] `npx react-native init TestApp` creates new project
- [ ] `npx react-native run-android` builds and runs on emulator
- [ ] `npx react-native run-ios` builds and runs on simulator (Mac)
- [ ] Hot reloading works when you save files
- [ ] React DevTools connects to your app
- [ ] You can debug JavaScript in Chrome DevTools

---

**Happy Coding! 🎉**

*Remember: Setup issues are normal and part of the learning process. Don't get discouraged - every developer faces these challenges!*