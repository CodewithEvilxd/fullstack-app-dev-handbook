# 📦 CodePush Setup Guide for React Native
## Complete OTA Updates Manual

---

## 🎯 **Overview**
CodePush is a cloud service that enables React Native developers to deploy mobile app updates directly to their users' devices. This guide covers complete CodePush setup, configuration, and usage for React Native projects.

---

## 📋 **Table of Contents**
1. [What is CodePush?](#what-is-codepush)
2. [Installation & Setup](#installation--setup)
3. [App Center Integration](#app-center-integration)
4. [Basic Configuration](#basic-configuration)
5. [Deployment Setup](#deployment-setup)
6. [Update Management](#update-management)
7. [Advanced Features](#advanced-features)
8. [Troubleshooting](#troubleshooting)
9. [Best Practices](#best-practices)

---

## 🤔 **What is CodePush?**

### **CodePush Features:**
```bash
✅ Over-the-air (OTA) updates
✅ Instant app updates without app store
✅ JavaScript and assets updates
✅ Rollback capabilities
✅ Staged rollouts
✅ Version targeting
✅ Platform-specific updates
✅ Update statistics and analytics
```

### **How CodePush Works:**
```bash
1. Developer pushes update to CodePush server
2. App checks for updates on startup
3. If update available, downloads in background
4. Next app restart uses new code
5. Seamless user experience
```

### **Benefits:**
```bash
🚀 Instant bug fixes and improvements
⚡ No app store approval delays
📱 Better user experience
🔄 Easy rollback capabilities
📊 A/B testing and gradual rollouts
💰 Cost-effective updates
🎯 Targeted deployments
```

### **Limitations:**
```bash
❌ Cannot update native code
❌ Cannot add new native dependencies
❌ Cannot update app icons/metadata
❌ Cannot change app permissions
❌ Bundle size limitations
```

---

## 📥 **Installation & Setup**

### **Step 1: Install React Native CodePush**
```bash
# Install CodePush CLI globally
npm install -g appcenter-cli

# Or using npx
npx appcenter --version

# Install React Native CodePush package
npm install react-native-code-push

# For iOS (if using CocoaPods)
cd ios && pod install
```

### **Step 2: Link CodePush (React Native < 0.60)**
```bash
# Automatic linking (React Native 0.60+)
# CodePush is auto-linked

# Manual linking (older versions)
react-native link react-native-code-push
```

### **Step 3: Configure CodePush in App**
```javascript
// App.js or index.js
import codePush from 'react-native-code-push';

const codePushOptions = {
  checkFrequency: codePush.CheckFrequency.ON_APP_START,
  installMode: codePush.InstallMode.ON_NEXT_RESTART,
  minimumBackgroundDuration: 60,
};

function App() {
  // Your app code
}

export default codePush(codePushOptions)(App);
```

### **Step 4: iOS Configuration**
```objc
// ios/MyApp/AppDelegate.m
#import <CodePush/CodePush.h>

// Replace the URL scheme with CodePush
- (NSURL *)sourceURLForBridge:(RCTBridge *)bridge
{
  #if DEBUG
    return [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index"];
  #else
    return [CodePush bundleURL];
  #endif
}
```

### **Step 5: Android Configuration**
```java
// android/app/src/main/java/com/myapp/MainApplication.java
import com.microsoft.codepush.react.CodePush;

@Override
protected String getJSBundleFile() {
  return CodePush.getJSBundleFile();
}
```

---

## 📱 **App Center Integration**

### **Create App Center Account**
```bash
# Visit App Center
https://appcenter.ms/

# Sign up with GitHub, Microsoft, or email
# Create organization or use personal account
```

### **Create App Center App**
```bash
# Create iOS app
appcenter apps create -d "MyApp-iOS" -o iOS -p React-Native

# Create Android app
appcenter apps create -d "MyApp-Android" -o Android -p React-Native

# List your apps
appcenter apps list
```

### **Get Deployment Keys**
```bash
# Get deployment keys for iOS
appcenter codepush deployment list -a MyOrg/MyApp-iOS

# Get deployment keys for Android
appcenter codepush deployment list -a MyOrg/MyApp-Android

# Keys will look like:
# Production: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
# Staging: YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY
```

### **Configure Deployment Keys**
```javascript
// App.js
const codePushOptions = {
  checkFrequency: codePush.CheckFrequency.ON_APP_START,
  installMode: codePush.InstallMode.ON_NEXT_RESTART,
  deploymentKey: __DEV__
    ? "STAGING_DEPLOYMENT_KEY"
    : "PRODUCTION_DEPLOYMENT_KEY",
};
```

### **Platform-Specific Keys**
```javascript
// For dynamic key selection
const getDeploymentKey = () => {
  if (__DEV__) {
    return Platform.OS === 'ios'
      ? 'IOS_STAGING_KEY'
      : 'ANDROID_STAGING_KEY';
  }
  return Platform.OS === 'ios'
    ? 'IOS_PRODUCTION_KEY'
    : 'ANDROID_PRODUCTION_KEY';
};

const codePushOptions = {
  deploymentKey: getDeploymentKey(),
  // ... other options
};
```

---

## ⚙️ **Basic Configuration**

### **CodePush Options**
```javascript
const codePushOptions = {
  // Update check frequency
  checkFrequency: codePush.CheckFrequency.ON_APP_START,

  // Installation mode
  installMode: codePush.InstallMode.ON_NEXT_RESTART,

  // Minimum background duration (seconds)
  minimumBackgroundDuration: 60,

  // Update dialog configuration
  updateDialog: {
    appendReleaseDescription: true,
    descriptionPrefix: " [✓] ",
    mandatoryContinueButtonLabel: "Continue",
    mandatoryUpdateMessage: "An update is available that must be installed.",
    optionalIgnoreButtonLabel: "Ignore",
    optionalInstallButtonLabel: "Install",
    optionalUpdateMessage: "An update is available. Would you like to install it?",
    title: "Update available"
  },

  // Rollback retry options
  rollbackRetryOptions: {
    delayInHours: 24,
    maxRetryAttempts: 3
  }
};
```

### **Update Check Frequencies**
```javascript
// Check on app start (recommended)
checkFrequency: codePush.CheckFrequency.ON_APP_START

// Check on app resume
checkFrequency: codePush.CheckFrequency.ON_APP_RESUME

// Manual check only
checkFrequency: codePush.CheckFrequency.MANUAL
```

### **Installation Modes**
```javascript
// Install on next restart (recommended)
installMode: codePush.InstallMode.ON_NEXT_RESTART

// Install immediately
installMode: codePush.InstallMode.IMMEDIATE

// Install when app is in background
installMode: codePush.InstallMode.ON_NEXT_SUSPEND
```

### **Custom Update Dialog**
```javascript
const customUpdateDialog = {
  title: "🚀 Update Available!",
  optionalUpdateMessage: "A new version is ready. Update now for the best experience.",
  optionalInstallButtonLabel: "Update Now",
  optionalIgnoreButtonLabel: "Later",
  mandatoryUpdateMessage: "This update is required to continue using the app.",
  mandatoryContinueButtonLabel: "Update",
  appendReleaseDescription: true,
  descriptionPrefix: "\n\nChange log:\n"
};
```

---

## 🚀 **Deployment Setup**

### **Login to App Center**
```bash
# Login with token
appcenter login

# Or login interactively
appcenter login --token YOUR_TOKEN

# Check login status
appcenter profile list
```

### **Release Update**
```bash
# Release to Staging
appcenter codepush release-react -a MyOrg/MyApp-iOS -d Staging

# Release to Production
appcenter codepush release-react -a MyOrg/MyApp-Android -d Production

# Release with specific version
appcenter codepush release-react -a MyOrg/MyApp-iOS -d Production --target-binary-version "1.0.0"

# Release with description
appcenter codepush release-react -a MyOrg/MyApp-iOS -d Production --description "Fixed login bug"
```

### **Advanced Release Options**
```bash
# Release with mandatory update
appcenter codepush release-react -a MyOrg/MyApp-iOS -d Production --mandatory

# Release to percentage of users
appcenter codepush release-react -a MyOrg/MyApp-iOS -d Production --rollout 50

# Release with custom bundle name
appcenter codepush release-react -a MyOrg/MyApp-iOS -d Production --bundle-name "main.jsbundle"

# Release from specific directory
appcenter codepush release-react -a MyOrg/MyApp-iOS -d Production --sourcemap-output "./sourcemaps"
```

### **Deployment History**
```bash
# View deployment history
appcenter codepush deployment history -a MyOrg/MyApp-iOS -d Production

# View specific release details
appcenter codepush deployment history -a MyOrg/MyApp-iOS -d Production --displayKeys
```

---

## 📊 **Update Management**

### **Check for Updates Manually**
```javascript
// Manual update check
codePush.checkForUpdate()
  .then((update) => {
    if (!update) {
      console.log("No updates available");
    } else {
      console.log("Update available:", update.description);
      // Handle update
    }
  });
```

### **Sync Updates**
```javascript
// Sync with default options
codePush.sync();

// Sync with custom options
codePush.sync({
  installMode: codePush.InstallMode.IMMEDIATE,
  mandatoryInstallMode: codePush.InstallMode.IMMEDIATE,
  updateDialog: customUpdateDialog,
});
```

### **Handle Update Events**
```javascript
// Listen to update events
codePush.notifyAppReady();

// Handle download progress
codePush.sync(
  {},
  (status) => {
    switch (status) {
      case codePush.SyncStatus.DOWNLOADING_PACKAGE:
        console.log("Downloading update...");
        break;
      case codePush.SyncStatus.INSTALLING_UPDATE:
        console.log("Installing update...");
        break;
      case codePush.SyncStatus.UPDATE_INSTALLED:
        console.log("Update installed successfully");
        break;
    }
  },
  (progress) => {
    console.log(`Downloaded ${progress.receivedBytes} of ${progress.totalBytes}`);
  }
);
```

### **Rollback Updates**
```javascript
// Rollback to previous version
codePush.rollback()
  .then(() => {
    console.log("Rollback successful");
  })
  .catch((error) => {
    console.error("Rollback failed:", error);
  });
```

### **Get Update Metadata**
```javascript
// Get current package info
codePush.getCurrentPackage()
  .then((packageInfo) => {
    console.log("Current version:", packageInfo.label);
    console.log("Description:", packageInfo.description);
  });
```

---

## 🚀 **Advanced Features**

### **Targeted Deployments**
```bash
# Deploy to specific app version
appcenter codepush release-react -a MyOrg/MyApp-iOS -d Production --target-binary-version "1.2.0"

# Deploy to version range
appcenter codepush release-react -a MyOrg/MyApp-iOS -d Production --target-binary-version "~1.2.0"

# Deploy to multiple versions
appcenter codepush release-react -a MyOrg/MyApp-iOS -d Production --target-binary-version ">=1.2.0 <1.3.0"
```

### **Gradual Rollouts**
```bash
# Rollout to 25% of users
appcenter codepush release-react -a MyOrg/MyApp-iOS -d Production --rollout 25

# Increase rollout percentage
appcenter codepush rollout 50 -a MyOrg/MyApp-iOS -d Production

# Complete rollout
appcenter codepush rollout 100 -a MyOrg/MyApp-iOS -d Production
```

### **A/B Testing**
```javascript
// Custom deployment keys for A/B testing
const getABTestDeploymentKey = () => {
  const userId = getUserId();
  const testGroup = (userId % 2) === 0 ? 'A' : 'B';

  if (testGroup === 'A') {
    return Platform.OS === 'ios' ? 'IOS_A_KEY' : 'ANDROID_A_KEY';
  } else {
    return Platform.OS === 'ios' ? 'IOS_B_KEY' : 'ANDROID_B_KEY';
  }
};
```

### **Custom Update Logic**
```javascript
// Advanced update handling
class UpdateManager {
  constructor() {
    this.updateDownloaded = false;
  }

  checkForUpdates() {
    return codePush.checkForUpdate()
      .then((update) => {
        if (update) {
          return this.shouldInstallUpdate(update);
        }
        return false;
      });
  }

  shouldInstallUpdate(update) {
    // Custom logic to decide whether to install
    const isCritical = update.description.includes('[CRITICAL]');
    const isUserActive = this.isUserActive();

    if (isCritical) {
      return true; // Always install critical updates
    }

    if (isUserActive) {
      return false; // Don't install while user is active
    }

    return true; // Install in background
  }

  installUpdate() {
    return codePush.sync({
      installMode: codePush.InstallMode.ON_NEXT_SUSPEND,
      minimumBackgroundDuration: 300, // 5 minutes
    });
  }
}
```

### **Offline Support**
```javascript
// Handle offline scenarios
const syncWithOfflineSupport = async () => {
  try {
    const update = await codePush.checkForUpdate();
    if (update) {
      // Download update for offline installation
      await codePush.sync({
        installMode: codePush.InstallMode.ON_NEXT_RESTART,
        downloadProgressCallback: (progress) => {
          // Show download progress to user
          console.log(`Downloaded: ${progress.receivedBytes}/${progress.totalBytes}`);
        }
      });
    }
  } catch (error) {
    if (error.code === 'NETWORK_ERROR') {
      console.log('Offline: Update will be installed when online');
    }
  }
};
```

### **Analytics Integration**
```javascript
// Track update events
const trackUpdateEvent = (event, data) => {
  // Send to analytics service
  console.log(`Update event: ${event}`, data);

  // Example: Send to Firebase Analytics
  // analytics().logEvent('codepush_update', { event, ...data });
};

// Usage
codePush.sync(
  {},
  (status) => {
    trackUpdateEvent('status_change', { status });

    switch (status) {
      case codePush.SyncStatus.UPDATE_INSTALLED:
        trackUpdateEvent('update_installed', {
          previousVersion: codePush.getCurrentPackage().label
        });
        break;
    }
  }
);
```

---

## 🔧 **Troubleshooting**

### **Common Issues & Solutions**

#### **Issue 1: Updates Not Downloading**
```bash
# Check deployment key
console.log("Deployment key:", codePush.getConfiguration().deploymentKey);

# Verify app is registered
appcenter apps list

# Check deployment status
appcenter codepush deployment history -a MyOrg/MyApp-iOS -d Production
```

#### **Issue 2: Updates Not Installing**
```javascript
// Check install mode
const packageInfo = await codePush.getCurrentPackage();
console.log("Install mode:", packageInfo.installMode);

// Force restart after update
codePush.restartApp();
```

#### **Issue 3: Rollback Issues**
```bash
# Check rollback availability
const packageInfo = await codePush.getCurrentPackage();
console.log("Is rollback available:", !packageInfo.isFirstRun);

# Manual rollback
codePush.rollback();
```

#### **Issue 4: Network Issues**
```javascript
// Handle network errors
codePush.sync(
  {},
  (status) => {
    if (status === codePush.SyncStatus.UNKNOWN_ERROR) {
      console.log("Network error - will retry later");
    }
  }
);
```

#### **Issue 5: Version Conflicts**
```bash
# Check binary version compatibility
appcenter codepush release-react -a MyOrg/MyApp-iOS --target-binary-version "1.2.0"

# Update target version
codePush.sync({
  updateDialog: {
    appendReleaseDescription: true,
  }
});
```

### **Debugging CodePush**
```javascript
// Enable debug logging
const originalLog = console.log;
console.log = function(...args) {
  originalLog('[CodePush]', ...args);
};

// Check CodePush status
codePush.getConfiguration().then(config => {
  console.log('CodePush config:', config);
});

// Monitor sync status
codePush.sync(
  {},
  (status) => {
    console.log('Sync status:', status);
  },
  (progress) => {
    console.log('Download progress:', progress);
  }
);
```

---

## 📚 **Best Practices**

### **Update Strategy**
```javascript
// Recommended update strategy
const updateStrategy = {
  // Check for updates on app start
  checkFrequency: codePush.CheckFrequency.ON_APP_START,

  // Install on next restart for better UX
  installMode: codePush.InstallMode.ON_NEXT_RESTART,

  // Show update dialog for user awareness
  updateDialog: {
    optionalUpdateMessage: "A new version is available. Update now?",
    optionalInstallButtonLabel: "Update",
    optionalIgnoreButtonLabel: "Later",
  },

  // Handle mandatory updates
  mandatoryInstallMode: codePush.InstallMode.IMMEDIATE,
};
```

### **Version Management**
```javascript
// Version targeting strategy
const versionStrategy = {
  // Target specific versions
  targetVersion: "1.2.0",

  // Target version ranges
  targetRange: ">=1.2.0 <1.3.0",

  // Use semantic versioning
  semanticVersioning: true,
};
```

### **Error Handling**
```javascript
// Comprehensive error handling
const handleUpdateError = (error) => {
  console.error('CodePush error:', error);

  // Categorize errors
  switch (error.code) {
    case 'NETWORK_ERROR':
      // Handle network issues
      break;
    case 'ROLLBACK_FAILED':
      // Handle rollback failures
      break;
    default:
      // Handle other errors
      break;
  }
};

// Usage
codePush.sync().catch(handleUpdateError);
```

### **Performance Optimization**
```javascript
// Optimize update downloads
const optimizedSync = () => {
  codePush.sync({
    // Download in background
    installMode: codePush.InstallMode.ON_NEXT_SUSPEND,

    // Minimum background time
    minimumBackgroundDuration: 300,

    // Custom download progress
    downloadProgressCallback: (progress) => {
      // Update UI with progress
      const percentage = (progress.receivedBytes / progress.totalBytes) * 100;
      console.log(`Download progress: ${percentage.toFixed(1)}%`);
    },
  });
};
```

### **Security Considerations**
```javascript
// Secure update validation
const validateUpdate = (update) => {
  // Verify update source
  const isValidSource = update.deploymentKey === expectedKey;

  // Check update size
  const isValidSize = update.packageSize < maxPackageSize;

  // Validate description
  const isValidDescription = !update.description.includes('suspicious');

  return isValidSource && isValidSize && isValidDescription;
};

// Usage
codePush.checkForUpdate()
  .then(update => {
    if (validateUpdate(update)) {
      return codePush.sync();
    }
  });
```

---

## 🚀 **Advanced Configuration**

### **Multi-Environment Setup**
```javascript
// Environment-specific configuration
const environments = {
  development: {
    deploymentKey: __DEV__ ? 'DEV_KEY' : 'PROD_KEY',
    checkFrequency: codePush.CheckFrequency.ON_APP_START,
  },
  staging: {
    deploymentKey: 'STAGING_KEY',
    checkFrequency: codePush.CheckFrequency.ON_APP_RESUME,
  },
  production: {
    deploymentKey: 'PRODUCTION_KEY',
    checkFrequency: codePush.CheckFrequency.MANUAL,
  },
};

const getEnvironmentConfig = () => {
  const env = process.env.NODE_ENV || 'development';
  return environments[env];
};
```

### **Custom Update Manager**
```javascript
// Advanced update manager
class CodePushManager {
  constructor() {
    this.updateQueue = [];
    this.isUpdating = false;
  }

  async checkAndInstallUpdates() {
    if (this.isUpdating) return;

    try {
      this.isUpdating = true;
      const update = await codePush.checkForUpdate();

      if (update && this.shouldInstallUpdate(update)) {
        await this.installUpdate(update);
      }
    } catch (error) {
      console.error('Update check failed:', error);
    } finally {
      this.isUpdating = false;
    }
  }

  shouldInstallUpdate(update) {
    // Custom logic for update installation
    const isCritical = update.isMandatory;
    const isWifiConnected = this.checkWifiConnection();
    const batteryLevel = this.getBatteryLevel();

    return isCritical || (isWifiConnected && batteryLevel > 20);
  }

  async installUpdate(update) {
    return new Promise((resolve, reject) => {
      codePush.sync(
        {
          installMode: codePush.InstallMode.ON_NEXT_RESTART,
          mandatoryInstallMode: codePush.InstallMode.IMMEDIATE,
        },
        (status) => {
          if (status === codePush.SyncStatus.UPDATE_INSTALLED) {
            resolve();
          }
        },
        (progress) => {
          // Handle progress
        }
      ).catch(reject);
    });
  }
}
```

### **Integration with CI/CD**
```yaml
# .github/workflows/codepush.yml
name: CodePush Deployment

on:
  push:
    branches: [main, develop]

jobs:
  codepush:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Build bundle
        run: npx react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/main.jsbundle

      - name: Deploy to CodePush
        run: |
          npx appcenter codepush release-react \
            -a MyOrg/MyApp-Android \
            -d ${{ github.ref == 'refs/heads/main' && 'Production' || 'Staging' }} \
            --description "Deploy from CI" \
            --mandatory false
        env:
          APPCENTER_ACCESS_TOKEN: ${{ secrets.APPCENTER_ACCESS_TOKEN }}
```

### **Analytics and Monitoring**
```javascript
// Update analytics tracking
const trackUpdateAnalytics = async (event, data) => {
  const analyticsData = {
    event,
    timestamp: new Date().toISOString(),
    platform: Platform.OS,
    appVersion: '1.0.0', // Get from package.json
    ...data,
  };

  // Send to analytics service
  try {
    await fetch('https://api.analytics.com/track', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(analyticsData),
    });
  } catch (error) {
    console.error('Analytics error:', error);
  }
};

// Usage
codePush.sync(
  {},
  (status) => {
    switch (status) {
      case codePush.SyncStatus.UPDATE_INSTALLED:
        trackUpdateAnalytics('update_installed', {
          previousLabel: data.previousPackage?.label,
          newLabel: data.package?.label,
        });
        break;
      case codePush.SyncStatus.UPDATE_IGNORED:
        trackUpdateAnalytics('update_ignored', {
          reason: 'User cancelled',
        });
        break;
    }
  }
);
```

---

## 📊 **Monitoring & Analytics**

### **Update Metrics**
```javascript
// Track update metrics
const updateMetrics = {
  totalChecks: 0,
  successfulUpdates: 0,
  failedUpdates: 0,
  rollbacks: 0,
  averageDownloadTime: 0,
};

const trackUpdateMetrics = (event, data) => {
  updateMetrics.totalChecks++;

  switch (event) {
    case 'update_success':
      updateMetrics.successfulUpdates++;
      break;
    case 'update_failed':
      updateMetrics.failedUpdates++;
      break;
    case 'rollback':
      updateMetrics.rollbacks++;
      break;
  }

  // Send metrics to monitoring service
  console.log('Update metrics:', updateMetrics);
};
```

### **Performance Monitoring**
```javascript
// Monitor update performance
const performanceMonitor = {
  startTime: null,
  downloadStartTime: null,

  startUpdateCheck() {
    this.startTime = Date.now();
  },

  startDownload() {
    this.downloadStartTime = Date.now();
  },

  endDownload() {
    if (this.downloadStartTime) {
      const downloadTime = Date.now() - this.downloadStartTime;
      console.log(`Download time: ${downloadTime}ms`);
      this.downloadStartTime = null;
    }
  },

  endUpdate() {
    if (this.startTime) {
      const totalTime = Date.now() - this.startTime;
      console.log(`Total update time: ${totalTime}ms`);
      this.startTime = null;
    }
  },
};
```

---

## 🎯 **Conclusion**

CodePush is a powerful tool for delivering over-the-air updates to React Native applications. It enables rapid deployment of bug fixes and improvements without requiring app store approvals.

### **Key Takeaways:**
- ✅ Install and configure CodePush in React Native apps
- ✅ Set up App Center for deployment management
- ✅ Configure proper update check frequencies and install modes
- ✅ Implement rollback capabilities for safety
- ✅ Use gradual rollouts for risk mitigation
- ✅ Monitor update performance and success rates
- ✅ Follow security best practices for updates

### **Next Steps:**
1. **Setup**: Install CodePush and configure App Center
2. **Configure**: Set up deployment keys and update policies
3. **Test**: Deploy updates to staging environment
4. **Monitor**: Track update success and user adoption
5. **Scale**: Implement advanced features like A/B testing
6. **Automate**: Integrate with CI/CD pipelines

### **Additional Resources:**
- [CodePush Documentation](https://docs.microsoft.com/en-us/appcenter/distribution/codepush/)
- [App Center Documentation](https://docs.microsoft.com/en-us/appcenter/)
- [React Native CodePush](https://github.com/microsoft/react-native-code-push)
- [CodePush CLI](https://github.com/microsoft/appcenter-cli)
- [OTA Updates Best Practices](https://reactnative.dev/docs/0.60/codepush)

---

*Happy CodePushing! 📦🚀*