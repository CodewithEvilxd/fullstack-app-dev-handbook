# 📱 Android Studio Setup Guide for React Native
## Complete Installation & Configuration Manual

---

## 🎯 **Overview**
Android Studio is the official IDE for Android development and is essential for React Native Android development. This guide covers complete setup, configuration, and usage for React Native projects.

---

## 📋 **Table of Contents**
1. [System Requirements](#system-requirements)
2. [Download & Installation](#download--installation)
3. [Initial Configuration](#initial-configuration)
4. [SDK Manager Setup](#sdk-manager-setup)
5. [AVD Manager Setup](#avd-manager-setup)
6. [React Native Integration](#react-native-integration)
7. [Troubleshooting](#troubleshooting)
8. [Best Practices](#best-practices)

---

## 💻 **System Requirements**

### **Minimum Requirements:**
- **OS**: Windows 10/11, macOS 10.14+, Linux (Ubuntu 18.04+)
- **RAM**: 8 GB minimum, 16 GB recommended
- **Storage**: 8 GB available space
- **Processor**: Intel i5 or equivalent

### **Recommended Requirements:**
- **RAM**: 16 GB or more
- **Storage**: 20 GB SSD
- **Processor**: Intel i7/AMD Ryzen 7 or better
- **Display**: 1920x1080 resolution

---

## 📥 **Download & Installation**

### **Step 1: Download Android Studio**
```bash
# Visit official website
https://developer.android.com/studio

# Download the latest version for your OS
# Choose between:
# - Windows (.exe installer)
# - macOS (.dmg)
# - Linux (.tar.gz)
```

### **Step 2: Installation Process**

#### **Windows Installation:**
```bash
# Run the downloaded .exe file
# Follow the installation wizard
# Choose installation location (recommended: C:\Android\Android Studio)
# Select components to install:
# - Android Studio
# - Android SDK
# - Android Virtual Device
```

#### **macOS Installation:**
```bash
# Mount the .dmg file
# Drag Android Studio to Applications folder
# Launch Android Studio from Applications
```

#### **Linux Installation:**
```bash
# Extract the downloaded .tar.gz file
sudo tar -xzf android-studio-*.tar.gz -C /opt/

# Create desktop entry
sudo ln -sf /opt/android-studio/bin/studio.sh /usr/local/bin/android-studio

# Create desktop shortcut (optional)
sudo nano /usr/share/applications/android-studio.desktop
```

### **Step 3: First Launch**
```bash
# Launch Android Studio
# Choose "Do not import settings" on first run
# Select "Standard" installation type
# Wait for components to download and install
```

---

## ⚙️ **Initial Configuration**

### **Step 1: Welcome Screen Setup**
```bash
# On first launch, you'll see:
# - Welcome to Android Studio
# - Choose "Next" to configure

# Select UI Theme:
# - Light (recommended for beginners)
# - Dark (better for eyes)
# - High contrast (accessibility)
```

### **Step 2: SDK Components Installation**
```bash
# Android Studio will automatically download:
# - Latest Android SDK
# - Android SDK Platform
# - Android SDK Build-Tools
# - Android Emulator
# - Android SDK Platform-Tools
```

### **Step 3: License Agreement**
```bash
# Accept all Android SDK licenses
# Open terminal/command prompt
# Run: flutter doctor --android-licenses
# Or manually accept in Android Studio
```

---

## 🔧 **SDK Manager Setup**

### **Accessing SDK Manager**
```bash
# Method 1: From Welcome Screen
# Click "Configure" → "SDK Manager"

# Method 2: From Project
# File → Settings → Appearance & Behavior → System Settings → Android SDK

# Method 3: Direct executable
# Navigate to: %ANDROID_HOME%\tools\bin\sdkmanager.bat (Windows)
# Navigate to: $ANDROID_HOME/tools/bin/sdkmanager (macOS/Linux)
```

### **Essential SDK Components**

#### **SDK Platforms:**
```bash
# Install these platforms:
✅ Android API 33 (Android 13.0) - Latest stable
✅ Android API 31 (Android 12.0) - Widely used
✅ Android API 30 (Android 11.0) - Good compatibility
✅ Android API 29 (Android 10.0) - Minimum recommended
```

#### **SDK Tools:**
```bash
# Required Tools:
✅ Android SDK Build-Tools 33.0.0 (latest)
✅ Android SDK Build-Tools 31.0.0
✅ Android SDK Command-line Tools (latest)
✅ Android SDK Platform-Tools (latest)
✅ Android Emulator (latest)
✅ Android Emulator Hypervisor Driver
```

#### **SDK Build-Tools:**
```bash
# Install multiple versions for compatibility:
✅ 33.0.0 (latest)
✅ 32.0.0
✅ 31.0.0
✅ 30.0.1
```

### **Installing Components via Command Line**
```bash
# Windows Command Prompt/PowerShell
# Navigate to SDK tools directory
cd %ANDROID_HOME%\cmdline-tools\latest\bin

# Install platforms
sdkmanager "platforms;android-33"
sdkmanager "platforms;android-31"
sdkmanager "platforms;android-30"

# Install build tools
sdkmanager "build-tools;33.0.0"
sdkmanager "build-tools;31.0.0"

# Install platform tools
sdkmanager "platform-tools"

# Install emulator
sdkmanager "emulator"
```

---

## 📱 **AVD Manager Setup**

### **Creating Android Virtual Device (AVD)**

#### **Step 1: Access AVD Manager**
```bash
# Method 1: From Welcome Screen
# Click "Configure" → "AVD Manager"

# Method 2: From Project
# Tools → Device Manager

# Method 3: From toolbar
# Click device dropdown → "Device Manager"
```

#### **Step 2: Create New AVD**
```bash
# Click "Create Virtual Device"
# Choose device category:
# - Phone (recommended for React Native)
# - Tablet
# - Wear OS
# - TV
```

#### **Recommended Device Configurations:**

##### **For Development Testing:**
```bash
# Device: Pixel 6
# Release: Android 13.0 (API 33)
# Architecture: x86_64
# RAM: 2048 MB
# Internal Storage: 8192 MB
# SD Card: 1024 MB
```

##### **For Performance Testing:**
```bash
# Device: Pixel 6 Pro
# Release: Android 13.0 (API 33)
# Architecture: x86_64
# RAM: 4096 MB
# Internal Storage: 16384 MB
# Enable: Hardware acceleration
```

##### **For UI Testing:**
```bash
# Device: Pixel 4
# Release: Android 11.0 (API 30)
# Architecture: x86
# RAM: 3072 MB
# Enable: Multi-touch
```

#### **Step 3: AVD Advanced Settings**
```bash
# Graphics: Hardware (recommended)
# Multi-Core CPU: Enable
# Camera: Enable front/back
# Network: Enable
# Sensors: Enable all
```

### **Managing AVDs**
```bash
# Start AVD
# Click play button next to AVD name

# Edit AVD
# Right-click AVD → "Edit"

# Delete AVD
# Right-click AVD → "Delete"

# Duplicate AVD
# Right-click AVD → "Duplicate"
```

---

## ⚛️ **React Native Integration**

### **Environment Variables Setup**

#### **Windows:**
```bash
# Create/modify system environment variables
# Control Panel → System → Advanced system settings → Environment Variables

# Add to User Variables:
ANDROID_HOME = C:\Users\[USERNAME]\AppData\Local\Android\Sdk
JAVA_HOME = C:\Program Files\Android\Android Studio\jbr

# Add to System Variables PATH:
%ANDROID_HOME%\platform-tools
%ANDROID_HOME%\emulator
%ANDROID_HOME%\tools
%ANDROID_HOME%\tools\bin
```

#### **macOS:**
```bash
# Edit .zshrc or .bash_profile
nano ~/.zshrc

# Add these lines:
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

# Reload profile
source ~/.zshrc
```

#### **Linux:**
```bash
# Edit .bashrc
nano ~/.bashrc

# Add these lines:
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

# Reload profile
source ~/.bashrc
```

### **Verifying Setup**
```bash
# Check Android Studio installation
android-studio --version

# Check SDK installation
adb version
emulator -version

# Check Java installation
java -version
javac -version

# Check environment variables
echo $ANDROID_HOME
echo $JAVA_HOME
```

### **React Native Project Setup**
```bash
# Create new React Native project
npx react-native init MyProject
cd MyProject

# Install dependencies
npm install

# Run on Android emulator
npx react-native run-android

# Or specify device
npx react-native run-android --deviceId=emulator-5554
```

---

## 🔧 **Troubleshooting**

### **Common Issues & Solutions**

#### **Issue 1: SDK Location Not Found**
```bash
# Error: SDK location not found
# Solution: Set ANDROID_HOME environment variable
export ANDROID_HOME=/path/to/Android/sdk
```

#### **Issue 2: Emulator Not Starting**
```bash
# Check virtualization is enabled in BIOS
# Enable VT-x/AMD-V in BIOS settings

# Check Hyper-V is disabled (Windows)
# Control Panel → Programs → Turn Windows features on/off
# Uncheck Hyper-V

# Cold boot emulator
emulator -avd [AVD_NAME] -no-snapshot-load
```

#### **Issue 3: Build Failures**
```bash
# Clean build
cd android && ./gradlew clean

# Clear React Native cache
npx react-native start --reset-cache

# Clear Metro cache
npx react-native run-android --reset-cache
```

#### **Issue 4: Device Not Recognized**
```bash
# Check connected devices
adb devices

# Restart ADB server
adb kill-server
adb start-server

# Check USB debugging (physical device)
# Settings → Developer Options → USB Debugging
```

#### **Issue 5: Gradle Sync Issues**
```bash
# Update Gradle wrapper
cd android && ./gradlew wrapper --gradle-version=7.6

# Clear Gradle cache
rm -rf ~/.gradle/caches/

# Force Gradle sync in Android Studio
# File → Sync Project with Gradle Files
```

### **Performance Optimization**
```bash
# Enable Gradle daemon
# gradle.properties
org.gradle.daemon=true
org.gradle.parallel=true
org.gradle.configureondemand=true

# Increase memory
# gradle.properties
org.gradle.jvmargs=-Xmx4096m -XX:MaxPermSize=1024m

# Enable configuration cache (Gradle 7.4+)
# gradle.properties
org.gradle.configuration-cache=true
```

---

## 📚 **Best Practices**

### **Project Structure**
```bash
# Recommended Android project structure
android/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/yourproject/
│   │   │   ├── res/
│   │   │   └── AndroidManifest.xml
│   │   └── androidTest/
│   └── build.gradle
├── build.gradle
├── gradle.properties
└── settings.gradle
```

### **Build Optimization**
```bash
# Enable build cache
# gradle.properties
org.gradle.caching=true

# Use latest build tools
# app/build.gradle
android {
    compileSdkVersion 33
    buildToolsVersion "33.0.0"
    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 33
    }
}
```

### **Emulator Optimization**
```bash
# Use hardware acceleration
# Enable Intel HAXM (Intel processors)
# Enable AMD Hypervisor (AMD processors)

# Use snapshots for faster startup
# AVD Manager → Edit AVD → Enable "Store a snapshot for faster startup"

# Use command line for faster operations
emulator -avd [AVD_NAME] -no-boot-anim -gpu swiftshader
```

### **Development Workflow**
```bash
# Use Android Studio for:
# - Debugging native Android code
# - UI inspection
# - Performance profiling
# - Memory analysis

# Use VS Code for:
# - JavaScript/TypeScript development
# - React Native specific coding
# - Extension ecosystem

# Use Terminal for:
# - Running React Native commands
# - ADB operations
# - Gradle tasks
```

---

## 🚀 **Advanced Configuration**

### **Custom Build Variants**
```gradle
// app/build.gradle
android {
    buildTypes {
        debug {
            buildConfigField "String", "API_URL", "\"https://api.dev.example.com\""
            debuggable true
        }
        release {
            buildConfigField "String", "API_URL", "\"https://api.example.com\""
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        staging {
            initWith debug
            buildConfigField "String", "API_URL", "\"https://api.staging.example.com\""
            matchingFallbacks = ['debug', 'release']
        }
    }
}
```

### **Signing Configuration**
```gradle
// app/build.gradle
android {
    signingConfigs {
        debug {
            storeFile file('debug.keystore')
            storePassword 'android'
            keyAlias 'androiddebugkey'
            keyPassword 'android'
        }
        release {
            storeFile file('release.keystore')
            storePassword System.getenv('STORE_PASSWORD')
            keyAlias System.getenv('KEY_ALIAS')
            keyPassword System.getenv('KEY_PASSWORD')
        }
    }
}
```

### **Multi-Environment Setup**
```gradle
// gradle.properties
# Development
DEV_API_URL=https://api.dev.example.com
DEV_DATABASE_URL=dev_database_url

# Staging
STAGING_API_URL=https://api.staging.example.com
STAGING_DATABASE_URL=staging_database_url

# Production
PROD_API_URL=https://api.example.com
PROD_DATABASE_URL=prod_database_url
```

---

## 📊 **Monitoring & Analytics**

### **Android Studio Profiler**
```bash
# Access Profiler
# View → Tool Windows → Profiler

# Monitor:
# - CPU usage
# - Memory allocation
# - Network activity
# - Energy consumption

# Record sessions for analysis
# Profiler → Start recording
```

### **ADB Commands for Monitoring**
```bash
# Monitor device logs
adb logcat

# Monitor specific package
adb logcat | grep "your.package.name"

# Monitor memory usage
adb shell dumpsys meminfo your.package.name

# Monitor CPU usage
adb shell top -m 10

# Take screenshot
adb shell screencap -p /sdcard/screen.png
adb pull /sdcard/screen.png

# Record screen
adb shell screenrecord /sdcard/demo.mp4
adb pull /sdcard/demo.mp4
```

---

## 🔐 **Security Best Practices**

### **App Signing**
```bash
# Generate keystore
keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000

# Sign APK
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore app-release-unsigned.apk my-key-alias

# Align APK
zipalign -v 4 app-release-unsigned.apk app-release.apk
```

### **ProGuard Configuration**
```proguard
# proguard-rules.pro
# Basic rules
-keepattributes Signature, InnerClasses, EnclosingMethod
-keepattributes RuntimeVisibleAnnotations, RuntimeVisibleParameterAnnotations
-keepattributes AnnotationDefault

# React Native specific
-keep class com.facebook.react.** { *; }
-keep class com.facebook.jni.** { *; }
-keep class com.facebook.soloader.** { *; }

# Your app specific rules
-keep class your.package.name.** { *; }
```

---

## 🎯 **Conclusion**

Android Studio is a powerful IDE that provides essential tools for React Native Android development. Proper setup and configuration are crucial for efficient development workflow.

### **Key Takeaways:**
- ✅ Install latest Android Studio version
- ✅ Configure SDK Manager with required components
- ✅ Set up AVD for testing
- ✅ Configure environment variables correctly
- ✅ Use Android Studio for debugging and profiling
- ✅ Follow best practices for performance

### **Next Steps:**
1. **Practice**: Create and run sample React Native projects
2. **Explore**: Learn Android Studio debugging tools
3. **Optimize**: Configure build settings for better performance
4. **Integrate**: Set up CI/CD pipelines with Android builds

### **Additional Resources:**
- [Android Studio Documentation](https://developer.android.com/studio)
- [React Native Android Setup](https://reactnative.dev/docs/environment-setup)
- [Android Developer Guides](https://developer.android.com/guide)
- [Android Studio Tips & Tricks](https://developer.android.com/studio/tips)

---

*Happy Android Development! 📱🤖*