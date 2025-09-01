# Lesson 52: CodePush for OTA Updates

## 🎯 **Learning Objectives**
- Implement over-the-air (OTA) updates using CodePush
- Configure CodePush for iOS and Android platforms
- Manage app updates without App Store reviews
- Handle update rollbacks and error recovery
- Monitor update deployment and user adoption

## 📚 **Table of Contents**
1. [Introduction to CodePush](#introduction-to-codepush)
2. [CodePush Setup](#codepush-setup)
3. [iOS Configuration](#ios-configuration)
4. [Android Configuration](#android-configuration)
5. [Update Deployment](#update-deployment)
6. [Rollback Strategies](#rollback-strategies)
7. [Update Management](#update-management)
8. [Error Handling](#error-handling)
9. [Monitoring & Analytics](#monitoring--analytics)
10. [Practical Examples](#practical-examples)

---

## 🚀 **Introduction to CodePush**

### **What is CodePush?**
CodePush is a cloud service that enables React Native developers to deploy mobile app updates directly to their users' devices. It works by pushing JavaScript and asset updates without requiring users to download a new version from the App Store or Google Play.

### **Benefits of CodePush**
- **Faster Updates**: Deploy bug fixes and improvements instantly
- **No App Store Review**: Bypass lengthy review processes for non-native changes
- **Gradual Rollout**: Release updates to percentage of users
- **Instant Rollback**: Quickly revert problematic updates
- **Platform Support**: Works on both iOS and Android
- **Version Targeting**: Target specific app versions or user groups

### **What Can Be Updated**
✅ **JavaScript Code**: All React Native JavaScript code
✅ **Assets**: Images, fonts, and other static assets
✅ **Configuration**: App configuration and feature flags
❌ **Native Code**: Cannot update native iOS/Android code
❌ **Native Dependencies**: Cannot update native libraries
❌ **App Permissions**: Cannot change app permissions

---

## ⚙️ **CodePush Setup**

### **Installation**
```bash
# Install CodePush CLI
npm install -g code-push-cli

# Login to CodePush (requires App Center account)
code-push login

# Alternatively, use App Center CLI
npm install -g appcenter-cli
appcenter login
```

### **App Center Setup**
```bash
# Create App Center app for iOS
appcenter apps create -d "MyApp-iOS" -o "iOS" -p "React-Native"

# Create App Center app for Android
appcenter apps create -d "MyApp-Android" -o "Android" -p "React-Native"

# Get deployment keys
appcenter codepush deployment list -a MyOrg/MyApp-iOS -k
appcenter codepush deployment list -a MyOrg/MyApp-Android -k
```

### **React Native Integration**
```bash
# Install CodePush package
npm install react-native-code-push

# For iOS
cd ios && pod install

# Link the package (React Native 0.59 and below)
react-native link react-native-code-push
```

---

## 🍎 **iOS Configuration**

### **AppDelegate Configuration**
```objc
// ios/MyApp/AppDelegate.m
#import <CodePush/CodePush.h>

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  // ... existing code ...

  // Add CodePush configuration
  [CodePush setDeploymentKey:@"YOUR_IOS_DEPLOYMENT_KEY"];

  // Optional: Configure update dialog
  [CodePush overrideAppVersion:@"1.0.0"]; // Force update check
  [CodePush setDeploymentKeyForEnvironment:@"Staging"]; // Environment-specific keys

  return YES;
}

@end
```

### **Info.plist Updates**
```xml
<!-- ios/MyApp/Info.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- ... existing entries ... -->

    <!-- CodePush configuration -->
    <key>CodePushDeploymentKey</key>
    <string>YOUR_IOS_DEPLOYMENT_KEY</string>

    <!-- Optional: Configure update behavior -->
    <key>CodePushPublicKey</key>
    <string>YOUR_PUBLIC_KEY</string>
</dict>
</plist>
```

---

## 🤖 **Android Configuration**

### **MainApplication Configuration**
```java
// android/app/src/main/java/com/myapp/MainApplication.java
import com.microsoft.codepush.react.CodePush;

public class MainApplication extends Application implements ReactApplication {

    // ... existing code ...

    @Override
    protected String getJSBundleFile() {
        return CodePush.getJSBundleFile();
    }

    @Override
    protected List<ReactPackage> getPackages() {
        List<ReactPackage> packages = new PackageList(this).getPackages();
        packages.add(new CodePush("YOUR_ANDROID_DEPLOYMENT_KEY", getApplicationContext(), BuildConfig.DEBUG));
        return packages;
    }
}
```

### **AndroidManifest Updates**
```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.myapp">

    <application>
        <!-- ... existing configuration ... -->

        <!-- CodePush metadata -->
        <meta-data
            android:name="CodePushDeploymentKey"
            android:value="YOUR_ANDROID_DEPLOYMENT_KEY" />

        <!-- Optional: Configure update behavior -->
        <meta-data
            android:name="CodePushPublicKey"
            android:value="YOUR_PUBLIC_KEY" />
    </application>
</manifest>
```

### **Gradle Configuration**
```gradle
// android/app/build.gradle
apply from: "../../node_modules/react-native-code-push/android/codepush.gradle"

// Optional: Configure CodePush
codePush {
    appKey = "YOUR_ANDROID_DEPLOYMENT_KEY"
}
```

---

## 📱 **React Native App Integration**

### **Basic CodePush Integration**
```javascript
// App.js
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

### **Advanced CodePush Configuration**
```javascript
// services/codePushService.js
import codePush from 'react-native-code-push';
import { Alert, Platform } from 'react-native';

class CodePushService {
  constructor() {
    this.updateDialog = {
      title: "Update Available",
      optionalUpdateMessage: "A new version is available. Would you like to update now?",
      optionalIgnoreButtonLabel: "Later",
      optionalInstallButtonLabel: "Update",
      appendReleaseDescription: true,
      descriptionPrefix: "\n\nWhat's new:\n",
      mandatoryUpdateMessage: "A mandatory update is available. Please update to continue.",
      mandatoryContinueButtonLabel: "Update Now"
    };

    this.syncOptions = {
      installMode: codePush.InstallMode.IMMEDIATE,
      mandatoryInstallMode: codePush.InstallMode.IMMEDIATE,
      deploymentKey: this.getDeploymentKey(),
      updateDialog: this.updateDialog
    };
  }

  // Get platform-specific deployment key
  getDeploymentKey() {
    if (Platform.OS === 'ios') {
      return __DEV__ ? 'IOS_STAGING_KEY' : 'IOS_PRODUCTION_KEY';
    } else {
      return __DEV__ ? 'ANDROID_STAGING_KEY' : 'ANDROID_PRODUCTION_KEY';
    }
  }

  // Check for updates manually
  async checkForUpdates() {
    try {
      const update = await codePush.checkForUpdate();

      if (update) {
        Alert.alert(
          "Update Available",
          `Version ${update.appVersion} is available. ${update.description}`,
          [
            {
              text: "Install",
              onPress: () => this.installUpdate(update)
            },
            {
              text: "Cancel",
              style: "cancel"
            }
          ]
        );
      } else {
        Alert.alert("No Updates", "Your app is up to date!");
      }
    } catch (error) {
      console.error('Update check failed:', error);
      Alert.alert("Error", "Failed to check for updates");
    }
  }

  // Install update
  async installUpdate(update) {
    try {
      await update.download();
      await codePush.restartApp();
    } catch (error) {
      console.error('Update installation failed:', error);
      Alert.alert("Error", "Failed to install update");
    }
  }

  // Sync with CodePush server
  async sync() {
    try {
      const syncStatus = await codePush.sync(this.syncOptions);

      switch (syncStatus) {
        case codePush.SyncStatus.UP_TO_DATE:
          console.log('App is up to date');
          break;
        case codePush.SyncStatus.UPDATE_INSTALLED:
          console.log('Update installed and will be applied on restart');
          break;
        case codePush.SyncStatus.UPDATE_IGNORED:
          console.log('Update ignored by user');
          break;
        case codePush.SyncStatus.ERROR:
          console.log('Sync error occurred');
          break;
      }

      return syncStatus;
    } catch (error) {
      console.error('Sync failed:', error);
      return codePush.SyncStatus.ERROR;
    }
  }

  // Get current package info
  async getCurrentPackage() {
    try {
      const packageInfo = await codePush.getCurrentPackage();
      return {
        label: packageInfo.label,
        appVersion: packageInfo.appVersion,
        deploymentKey: packageInfo.deploymentKey,
        description: packageInfo.description,
        isMandatory: packageInfo.isMandatory,
        packageSize: packageInfo.packageSize
      };
    } catch (error) {
      console.error('Failed to get package info:', error);
      return null;
    }
  }

  // Configure update behavior
  configure(options) {
    this.syncOptions = { ...this.syncOptions, ...options };
  }

  // Handle app resume
  onAppResume() {
    // Check for updates when app resumes
    this.sync();
  }

  // Handle app start
  onAppStart() {
    // Initial sync
    this.sync();
  }
}

export const codePushService = new CodePushService();
```

---

## 🚀 **Update Deployment**

### **Basic Deployment**
```bash
# Deploy to staging
code-push release-react MyApp-iOS ios --deploymentName Staging --description "Bug fixes and improvements"

# Deploy to production
code-push release-react MyApp-iOS ios --deploymentName Production --description "New features release"

# Android deployment
code-push release-react MyApp-Android android --deploymentName Production --mandatory
```

### **Advanced Deployment Options**
```bash
# Deploy with specific target version
code-push release-react MyApp-iOS ios \
  --deploymentName Production \
  --description "Version 2.0.0 updates" \
  --targetBinaryVersion "2.0.0" \
  --mandatory

# Deploy to percentage of users
code-push release-react MyApp-iOS ios \
  --deploymentName Production \
  --description "Gradual rollout" \
  --rollout 25

# Deploy with custom bundle name
code-push release-react MyApp-iOS ios \
  --deploymentName Production \
  --outputDir ./build \
  --sourcemapOutput ./build/sourcemap.js
```

### **Automated Deployment Script**
```bash
#!/bin/bash
# deploy-codepush.sh

set -e

# Configuration
APP_NAME="MyApp"
DEPLOYMENT_ENV=$1
PLATFORM=$2
ROLLOUT_PERCENTAGE=${3:-100}
DESCRIPTION=${4:-"Automated deployment"}

if [ -z "$DEPLOYMENT_ENV" ] || [ -z "$PLATFORM" ]; then
  echo "Usage: $0 <environment> <platform> [rollout_percentage] [description]"
  echo "Environments: staging, production"
  echo "Platforms: ios, android, both"
  exit 1
fi

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

# Validate rollout percentage
if ! [[ "$ROLLOUT_PERCENTAGE" =~ ^[0-9]+$ ]] || [ "$ROLLOUT_PERCENTAGE" -gt 100 ]; then
  error "Invalid rollout percentage: $ROLLOUT_PERCENTAGE"
  exit 1
fi

# Build the app bundle
log "Building app bundle..."
npx react-native bundle \
  --platform $PLATFORM \
  --dev false \
  --entry-file index.js \
  --bundle-output ./build/main.jsbundle \
  --sourcemap-output ./build/main.jsbundle.map

# Deploy based on platform
deploy_platform() {
  local platform=$1
  local app_name="$APP_NAME-$platform"

  log "Deploying to $platform ($DEPLOYMENT_ENV)..."

  # Set deployment name
  local deployment_name="Production"
  if [ "$DEPLOYMENT_ENV" = "staging" ]; then
    deployment_name="Staging"
  fi

  # Build CodePush command
  local cmd="code-push release-react $app_name $platform"
  cmd="$cmd --deploymentName $deployment_name"
  cmd="$cmd --description \"$DESCRIPTION\""

  if [ "$ROLLOUT_PERCENTAGE" -lt 100 ]; then
    cmd="$cmd --rollout $ROLLOUT_PERCENTAGE"
  fi

  if [ "$DEPLOYMENT_ENV" = "production" ]; then
    cmd="$cmd --mandatory"
  fi

  # Execute deployment
  log "Executing: $cmd"
  eval $cmd

  if [ $? -eq 0 ]; then
    log "✅ $platform deployment completed successfully!"
  else
    error "❌ $platform deployment failed!"
    exit 1
  fi
}

# Main deployment logic
case $PLATFORM in
  ios)
    deploy_platform "iOS"
    ;;
  android)
    deploy_platform "Android"
    ;;
  both)
    deploy_platform "iOS"
    deploy_platform "Android"
    ;;
  *)
    error "Invalid platform: $PLATFORM"
    echo "Valid platforms: ios, android, both"
    exit 1
    ;;
esac

# Send notification
if [ -n "$SLACK_WEBHOOK" ]; then
  curl -X POST -H 'Content-type: application/json' \
    --data "{\"text\":\"🚀 CodePush deployment completed: $APP_NAME ($PLATFORM) - $DEPLOYMENT_ENV ($ROLLOUT_PERCENTAGE%)\"}" \
    $SLACK_WEBHOOK
fi

log "🎉 All deployments completed successfully!"
```

---

## 🔄 **Rollback Strategies**

### **Immediate Rollback**
```bash
# Rollback to previous version
code-push rollback MyApp-iOS Production

# Rollback with custom description
code-push rollback MyApp-iOS Production --description "Rolling back due to critical bug"
```

### **Gradual Rollback**
```javascript
// services/rollbackService.js
import codePush from 'react-native-code-push';

class RollbackService {
  constructor() {
    this.rollbackThreshold = 0.1; // 10% error rate threshold
    this.monitoringPeriod = 300000; // 5 minutes
  }

  // Monitor update health
  async monitorUpdateHealth(updateLabel) {
    const startTime = Date.now();
    let errorCount = 0;
    let totalRequests = 0;

    // Monitor for specified period
    const monitoringInterval = setInterval(async () => {
      try {
        // Get error metrics (implement based on your error tracking)
        const metrics = await this.getErrorMetrics(updateLabel);

        totalRequests = metrics.totalRequests;
        errorCount = metrics.errorCount;

        const errorRate = errorCount / totalRequests;

        if (errorRate > this.rollbackThreshold) {
          console.warn(`High error rate detected: ${(errorRate * 100).toFixed(2)}%`);
          await this.initiateRollback(updateLabel, `High error rate: ${(errorRate * 100).toFixed(2)}%`);
          clearInterval(monitoringInterval);
        }

        if (Date.now() - startTime > this.monitoringPeriod) {
          clearInterval(monitoringInterval);
          console.log('Monitoring period completed successfully');
        }
      } catch (error) {
        console.error('Monitoring error:', error);
      }
    }, 60000); // Check every minute
  }

  // Initiate rollback
  async initiateRollback(updateLabel, reason) {
    try {
      console.log(`Initiating rollback for update ${updateLabel}: ${reason}`);

      // Notify team
      await this.notifyTeam('rollback', {
        updateLabel,
        reason,
        timestamp: new Date().toISOString()
      });

      // Perform rollback
      const rollbackResult = await codePush.rollback();

      if (rollbackResult) {
        console.log('Rollback completed successfully');
        await this.notifyTeam('rollback_success', {
          updateLabel,
          timestamp: new Date().toISOString()
        });
      } else {
        console.error('Rollback failed');
        await this.notifyTeam('rollback_failed', {
          updateLabel,
          timestamp: new Date().toISOString()
        });
      }
    } catch (error) {
      console.error('Rollback error:', error);
      await this.notifyTeam('rollback_error', {
        updateLabel,
        error: error.message,
        timestamp: new Date().toISOString()
      });
    }
  }

  // Get error metrics (placeholder - implement based on your error tracking)
  async getErrorMetrics(updateLabel) {
    // This should integrate with your error tracking service
    return {
      totalRequests: 1000,
      errorCount: 50,
      updateLabel
    };
  }

  // Notify team (implement based on your notification system)
  async notifyTeam(type, data) {
    console.log(`Team notification: ${type}`, data);

    // Implement Slack, email, or other notification methods
    // Example: Slack webhook
    if (type === 'rollback') {
      // Send rollback alert
    }
  }

  // Configure monitoring parameters
  configure(options) {
    if (options.threshold) {
      this.rollbackThreshold = options.threshold;
    }
    if (options.monitoringPeriod) {
      this.monitoringPeriod = options.monitoringPeriod;
    }
  }
}

export const rollbackService = new RollbackService();
```

---

## 📊 **Update Management**

### **Version Targeting**
```bash
# Target specific app versions
code-push release-react MyApp-iOS ios \
  --deploymentName Production \
  --targetBinaryVersion "~1.2.0" \
  --description "Update for version 1.2.x"

# Target version range
code-push release-react MyApp-iOS ios \
  --deploymentName Production \
  --targetBinaryVersion "1.2.0-1.3.0" \
  --description "Update for versions 1.2.0 to 1.3.0"
```

### **User Group Targeting**
```javascript
// services/targetingService.js
class TargetingService {
  constructor() {
    this.userGroups = {
      beta: ['user1@example.com', 'user2@example.com'],
      vip: ['vip1@example.com', 'vip2@example.com'],
      enterprise: ['enterprise@example.com']
    };
  }

  // Check if user should receive update
  shouldReceiveUpdate(userEmail, updateConfig) {
    if (!updateConfig.targetGroups) return true;

    return updateConfig.targetGroups.some(group =>
      this.userGroups[group]?.includes(userEmail)
    );
  }

  // Get user group
  getUserGroup(userEmail) {
    for (const [group, emails] of Object.entries(this.userGroups)) {
      if (emails.includes(userEmail)) {
        return group;
      }
    }
    return 'general';
  }

  // Add user to group
  addUserToGroup(userEmail, group) {
    if (!this.userGroups[group]) {
      this.userGroups[group] = [];
    }

    if (!this.userGroups[group].includes(userEmail)) {
      this.userGroups[group].push(userEmail);
    }
  }

  // Remove user from group
  removeUserFromGroup(userEmail, group) {
    if (this.userGroups[group]) {
      const index = this.userGroups[group].indexOf(userEmail);
      if (index > -1) {
        this.userGroups[group].splice(index, 1);
      }
    }
  }

  // Get all groups
  getAllGroups() {
    return Object.keys(this.userGroups);
  }

  // Get group members
  getGroupMembers(group) {
    return this.userGroups[group] || [];
  }
}

export const targetingService = new TargetingService();
```

---

## 🚨 **Error Handling**

### **Update Error Handling**
```javascript
// services/updateErrorHandler.js
import codePush from 'react-native-code-push';
import { Alert } from 'react-native';

class UpdateErrorHandler {
  constructor() {
    this.errorCounts = new Map();
    this.maxRetries = 3;
  }

  // Handle sync errors
  async handleSyncError(error, syncOptions) {
    console.error('CodePush sync error:', error);

    const errorType = this.categorizeError(error);

    // Track error
    this.trackError(errorType, error);

    // Handle based on error type
    switch (errorType) {
      case 'network':
        await this.handleNetworkError(syncOptions);
        break;
      case 'corruption':
        await this.handleCorruptionError();
        break;
      case 'storage':
        await this.handleStorageError();
        break;
      default:
        await this.handleGenericError(error);
    }
  }

  // Categorize error
  categorizeError(error) {
    const message = error.message?.toLowerCase() || '';

    if (message.includes('network') || message.includes('timeout')) {
      return 'network';
    }
    if (message.includes('corrupt') || message.includes('integrity')) {
      return 'corruption';
    }
    if (message.includes('storage') || message.includes('disk')) {
      return 'storage';
    }
    return 'generic';
  }

  // Handle network errors
  async handleNetworkError(syncOptions) {
    console.log('Handling network error - will retry later');

    // Schedule retry
    setTimeout(() => {
      codePush.sync(syncOptions);
    }, 30000); // Retry in 30 seconds
  }

  // Handle corruption errors
  async handleCorruptionError() {
    console.log('Handling corruption error - clearing cache');

    Alert.alert(
      'Update Error',
      'There was an issue with the update. The app will restart to fix the problem.',
      [
        {
          text: 'OK',
          onPress: () => {
            // Clear CodePush cache and restart
            codePush.clearUpdates();
            codePush.restartApp();
          }
        }
      ]
    );
  }

  // Handle storage errors
  async handleStorageError() {
    console.log('Handling storage error');

    Alert.alert(
      'Storage Error',
      'Not enough storage space for the update. Please free up some space and try again.',
      [{ text: 'OK' }]
    );
  }

  // Handle generic errors
  async handleGenericError(error) {
    console.log('Handling generic error');

    Alert.alert(
      'Update Error',
      'An error occurred while updating the app. Please try again later.',
      [
        {
          text: 'Retry',
          onPress: () => codePush.sync()
        },
        {
          text: 'Cancel',
          style: 'cancel'
        }
      ]
    );
  }

  // Track errors
  trackError(type, error) {
    const count = this.errorCounts.get(type) || 0;
    this.errorCounts.set(type, count + 1);

    // Report to analytics
    analytics.trackEvent('codepush_error', {
      error_type: type,
      error_message: error.message,
      error_count: count + 1,
      timestamp: new Date().toISOString()
    });

    // If too many errors, trigger rollback
    if (count + 1 >= this.maxRetries) {
      console.warn(`Too many ${type} errors, considering rollback`);
      // Implement rollback logic
    }
  }

  // Get error statistics
  getErrorStats() {
    const stats = {};
    for (const [type, count] of this.errorCounts.entries()) {
      stats[type] = count;
    }
    return stats;
  }

  // Reset error counts
  resetErrorCounts() {
    this.errorCounts.clear();
  }
}

export const updateErrorHandler = new UpdateErrorHandler();
```

---

## 📈 **Monitoring & Analytics**

### **Update Analytics**
```javascript
// services/updateAnalytics.js
import analytics from '../services/analyticsService';

class UpdateAnalytics {
  constructor() {
    this.updateStats = {
      totalUpdates: 0,
      successfulUpdates: 0,
      failedUpdates: 0,
      skippedUpdates: 0,
      rollbackCount: 0
    };
  }

  // Track update start
  trackUpdateStart(updateInfo) {
    analytics.trackEvent('codepush_update_started', {
      update_label: updateInfo.label,
      update_description: updateInfo.description,
      is_mandatory: updateInfo.isMandatory,
      package_size: updateInfo.packageSize,
      timestamp: new Date().toISOString()
    });

    console.log('Update started:', updateInfo.label);
  }

  // Track update success
  trackUpdateSuccess(updateInfo) {
    this.updateStats.totalUpdates++;
    this.updateStats.successfulUpdates++;

    analytics.trackEvent('codepush_update_success', {
      update_label: updateInfo.label,
      update_description: updateInfo.description,
      is_mandatory: updateInfo.isMandatory,
      package_size: updateInfo.packageSize,
      timestamp: new Date().toISOString()
    });

    console.log('Update successful:', updateInfo.label);
  }

  // Track update failure
  trackUpdateFailure(updateInfo, error) {
    this.updateStats.totalUpdates++;
    this.updateStats.failedUpdates++;

    analytics.trackEvent('codepush_update_failed', {
      update_label: updateInfo.label,
      update_description: updateInfo.description,
      error_message: error.message,
      error_type: error.name,
      timestamp: new Date().toISOString()
    });

    console.error('Update failed:', updateInfo.label, error);
  }

  // Track update skip
  trackUpdateSkipped(updateInfo, reason) {
    this.updateStats.skippedUpdates++;

    analytics.trackEvent('codepush_update_skipped', {
      update_label: updateInfo.label,
      skip_reason: reason,
      timestamp: new Date().toISOString()
    });

    console.log('Update skipped:', updateInfo.label, reason);
  }

  // Track rollback
  trackRollback(updateLabel, reason) {
    this.updateStats.rollbackCount++;

    analytics.trackEvent('codepush_rollback', {
      update_label: updateLabel,
      rollback_reason: reason,
      timestamp: new Date().toISOString()
    });

    console.log('Rollback performed:', updateLabel, reason);
  }

  // Track sync status
  trackSyncStatus(status, details = {}) {
    analytics.trackEvent('codepush_sync', {
      sync_status: status,
      ...details,
      timestamp: new Date().toISOString()
    });

    console.log('Sync status:', status, details);
  }

  // Get update statistics
  getUpdateStats() {
    const successRate = this.updateStats.totalUpdates > 0
      ? (this.updateStats.successfulUpdates / this.updateStats.totalUpdates) * 100
      : 0;

    return {
      ...this.updateStats,
      successRate: Math.round(successRate * 100) / 100
    };
  }

  // Generate update report
  generateUpdateReport() {
    const stats = this.getUpdateStats();
    const report = {
      period: {
        start: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString(),
        end: new Date().toISOString()
      },
      statistics: stats,
      recommendations: this.generateRecommendations(stats)
    };

    return report;
  }

  // Generate recommendations based on stats
  generateRecommendations(stats) {
    const recommendations = [];

    if (stats.successRate < 95) {
      recommendations.push('High failure rate detected. Review update process and error handling.');
    }

    if (stats.rollbackCount > stats.totalUpdates * 0.1) {
      recommendations.push('Frequent rollbacks detected. Improve testing before deployment.');
    }

    if (stats.skippedUpdates > stats.totalUpdates * 0.2) {
      recommendations.push('Many updates are being skipped. Check targeting and mandatory settings.');
    }

    return recommendations;
  }

  // Reset statistics
  resetStats() {
    this.updateStats = {
      totalUpdates: 0,
      successfulUpdates: 0,
      failedUpdates: 0,
      skippedUpdates: 0,
      rollbackCount: 0
    };
  }
}

export const updateAnalytics = new UpdateAnalytics();
```

---

## 🎯 **Practical Examples**

### **Complete CodePush Integration**
```javascript
// App.js - Complete CodePush integration
import React, { useEffect } from 'react';
import { Alert, AppState } from 'react-native';
import codePush from 'react-native-code-push';
import { NavigationContainer } from '@react-navigation/native';
import { codePushService } from './services/codePushService';
import { updateAnalytics } from './services/updateAnalytics';
import { updateErrorHandler } from './services/updateErrorHandler';

const codePushOptions = {
  checkFrequency: codePush.CheckFrequency.ON_APP_RESUME,
  installMode: codePush.InstallMode.ON_NEXT_RESUME,
  mandatoryInstallMode: codePush.InstallMode.IMMEDIATE,
  updateDialog: {
    appendReleaseDescription: true,
    descriptionPrefix: "\n\nWhat's new:\n",
    mandatoryContinueButtonLabel: "Update Now",
    mandatoryUpdateMessage: "A mandatory update is available. Please update to continue using the app.",
    optionalIgnoreButtonLabel: "Later",
    optionalInstallButtonLabel: "Update",
    optionalUpdateMessage: "An update is available. Would you like to update now?",
    title: "Update Available"
  }
};

const App = () => {
  useEffect(() => {
    // Initialize CodePush service
    codePushService.onAppStart();

    // Handle app state changes
    const subscription = AppState.addEventListener('change', (nextAppState) => {
      if (nextAppState === 'active') {
        codePushService.onAppResume();
      }
    });

    // Set up CodePush event listeners
    const updateListener = codePush.codePushDidUpdate.subscribe((update) => {
      updateAnalytics.trackUpdateSuccess(update);
    });

    const errorListener = codePush.codePushError.subscribe((error) => {
      updateErrorHandler.handleSyncError(error, codePushOptions);
    });

    return () => {
      subscription?.remove();
      updateListener?.remove();
      errorListener?.remove();
    };
  }, []);

  // Handle CodePush sync status
  const handleCodePushSync = async () => {
    try {
      const syncStatus = await codePushService.sync();

      switch (syncStatus) {
        case codePush.SyncStatus.UPDATE_INSTALLED:
          Alert.alert(
            'Update Installed',
            'The update has been installed. Restart the app to apply changes.',
            [
              {
                text: 'Restart Now',
                onPress: () => codePush.restartApp()
              },
              {
                text: 'Later',
                style: 'cancel'
              }
            ]
          );
          break;

        case codePush.SyncStatus.UP_TO_DATE:
          Alert.alert('Up to Date', 'Your app is already up to date!');
          break;

        case codePush.SyncStatus.UPDATE_IGNORED:
          console.log('Update ignored by user');
          break;

        case codePush.SyncStatus.ERROR:
          Alert.alert('Update Error', 'Failed to check for updates. Please try again later.');
          break;
      }

      updateAnalytics.trackSyncStatus(syncStatus);
    } catch (error) {
      console.error('Sync error:', error);
      updateErrorHandler.handleSyncError(error, codePushOptions);
    }
  };

  return (
    <NavigationContainer>
      {/* Your app navigation */}
      <AppNavigator />

      {/* CodePush update button for testing */}
      {__DEV__ && (
        <TouchableOpacity
          style={styles.updateButton}
          onPress={handleCodePushSync}
        >
          <Text style={styles.updateButtonText}>Check for Updates</Text>
        </TouchableOpacity>
      )}
    </NavigationContainer>
  );
};

const styles = StyleSheet.create({
  updateButton: {
    position: 'absolute',
    bottom: 20,
    right: 20,
    backgroundColor: '#007AFF',
    padding: 10,
    borderRadius: 5,
  },
  updateButtonText: {
    color: 'white',
    fontSize: 12,
  },
});

export default codePush(codePushOptions)(App);
```

### **CodePush Dashboard Component**
```javascript
// components/CodePushDashboard.js
import React, { useState, useEffect } from 'react';
import { View, Text, TouchableOpacity, ScrollView, StyleSheet, Alert } from 'react-native';
import { codePushService } from '../services/codePushService';
import { updateAnalytics } from '../services/updateAnalytics';

const CodePushDashboard = () => {
  const [currentPackage, setCurrentPackage] = useState(null);
  const [updateStats, setUpdateStats] = useState({});
  const [isChecking, setIsChecking] = useState(false);

  useEffect(() => {
    loadPackageInfo();
    loadUpdateStats();
  }, []);

  const loadPackageInfo = async () => {
    try {
      const packageInfo = await codePushService.getCurrentPackage();
      setCurrentPackage(packageInfo);
    } catch (error) {
      console.error('Failed to load package info:', error);
    }
  };

  const loadUpdateStats = () => {
    const stats = updateAnalytics.getUpdateStats();
    setUpdateStats(stats);
  };

  const handleCheckForUpdates = async () => {
    setIsChecking(true);
    try {
      await codePushService.checkForUpdates();
    } catch (error) {
      Alert.alert('Error', 'Failed to check for updates');
    } finally {
      setIsChecking(false);
    }
  };

  const handleSync = async () => {
    try {
      await codePushService.sync();
    } catch (error) {
      Alert.alert('Error', 'Failed to sync updates');
    }
  };

  const formatBytes = (bytes) => {
    if (!bytes) return 'N/A';
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(1024));
    return Math.round(bytes / Math.pow(1024, i) * 100) / 100 + ' ' + sizes[i];
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>CodePush Dashboard</Text>

      {/* Current Package Info */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Current Package</Text>
        {currentPackage ? (
          <View style={styles.packageInfo}>
            <Text style={styles.infoText}>Label: {currentPackage.label || 'N/A'}</Text>
            <Text style={styles.infoText}>App Version: {currentPackage.appVersion || 'N/A'}</Text>
            <Text style={styles.infoText}>Size: {formatBytes(currentPackage.packageSize)}</Text>
            <Text style={styles.infoText}>Mandatory: {currentPackage.isMandatory ? 'Yes' : 'No'}</Text>
            {currentPackage.description && (
              <Text style={styles.description}>Description: {currentPackage.description}</Text>
            )}
          </View>
        ) : (
          <Text style={styles.noData}>No package information available</Text>
        )}
      </View>

      {/* Update Statistics */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Update Statistics</Text>
        <View style={styles.statsGrid}>
          <View style={styles.stat}>
            <Text style={styles.statValue}>{updateStats.totalUpdates || 0}</Text>
            <Text style={styles.statLabel}>Total Updates</Text>
          </View>
          <View style={styles.stat}>
            <Text style={styles.statValue}>{updateStats.successfulUpdates || 0}</Text>
            <Text style={styles.statLabel}>Successful</Text>
          </View>
          <View style={styles.stat}>
            <Text style={styles.statValue}>{updateStats.failedUpdates || 0}</Text>
            <Text style={styles.statLabel}>Failed</Text>
          </View>
          <View style={styles.stat}>
            <Text style={styles.statValue}>{`${updateStats.successRate || 0}%`}</Text>
            <Text style={styles.statLabel}>Success Rate</Text>
          </View>
        </View>
      </View>

      {/* Action Buttons */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Actions</Text>
        <View style={styles.buttonContainer}>
          <TouchableOpacity
            style={[styles.button, isChecking && styles.disabledButton]}
            onPress={handleCheckForUpdates}
            disabled={isChecking}
          >
            <Text style={styles.buttonText}>
              {isChecking ? 'Checking...' : 'Check for Updates'}
            </Text>
          </TouchableOpacity>

          <TouchableOpacity style={styles.button} onPress={handleSync}>
            <Text style={styles.buttonText}>Sync Updates</Text>
          </TouchableOpacity>

          <TouchableOpacity style={styles.button} onPress={loadPackageInfo}>
            <Text style={styles.buttonText}>Refresh Info</Text>
          </TouchableOpacity>
        </View>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    margin: 20,
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
    marginBottom: 10,
  },
  packageInfo: {
    backgroundColor: '#f8f9fa',
    padding: 10,
    borderRadius: 5,
  },
  infoText: {
    fontSize: 14,
    marginBottom: 5,
  },
  description: {
    fontSize: 14,
    fontStyle: 'italic',
    color: '#666',
  },
  noData: {
    fontSize: 14,
    color: '#666',
    fontStyle: 'italic',
  },
  statsGrid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-between',
  },
  stat: {
    width: '48%',
    alignItems: 'center',
    marginBottom: 15,
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
  buttonContainer: {
    gap: 10,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
  },
  disabledButton: {
    backgroundColor: '#ccc',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default CodePushDashboard;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **CodePush Setup**: Installation and configuration for iOS and Android
- ✅ **OTA Updates**: Over-the-air updates without App Store reviews
- ✅ **Update Deployment**: Release management and rollout strategies
- ✅ **Rollback Strategies**: Quick reversion of problematic updates
- ✅ **Update Management**: Version targeting and user group management
- ✅ **Error Handling**: Comprehensive error handling and recovery
- ✅ **Monitoring & Analytics**: Update tracking and performance monitoring
- ✅ **Best Practices**: Security, testing, and deployment strategies

### **Best Practices**
1. **Test thoroughly before deployment** - Use staging environments extensively
2. **Implement gradual rollouts** - Start with small percentage of users
3. **Monitor update performance** - Track success rates and error patterns
4. **Have rollback plans** - Be prepared to revert quickly
5. **Use version targeting** - Target specific app versions for updates
6. **Handle errors gracefully** - Provide fallback options for failed updates
7. **Communicate with users** - Keep users informed about updates
8. **Secure deployment keys** - Never commit sensitive keys to version control
9. **Automate deployment process** - Use scripts and CI/CD for consistency
10. **Monitor resource usage** - Track bandwidth and storage impact

### **Next Steps**
- Learn advanced deployment patterns like A/B testing with CodePush
- Implement automated rollback based on error thresholds
- Set up advanced monitoring and alerting systems
- Learn about hybrid deployment strategies
- Explore integration with other update mechanisms

---

## 🎯 **Assignment**

### **Task 1: CodePush Integration**
Set up complete CodePush integration for a React Native app:
- Configure App Center accounts for iOS and Android
- Set up deployment keys and environments
- Implement CodePush in the React Native app
- Create staging and production deployment configurations
- Test update deployment and rollback functionality

### **Task 2: Update Management System**
Build a comprehensive update management system:
- Create automated deployment scripts
- Implement gradual rollout strategies
- Set up update monitoring and analytics
- Build error handling and recovery mechanisms
- Create user communication system for updates

### **Task 3: Advanced CodePush Features**
Implement advanced CodePush features:
- User group targeting for updates
- Mandatory vs optional update handling
- Update dependency management
- Performance monitoring and optimization
- Integration with CI/CD pipelines

---

## 📚 **Additional Resources**
- [CodePush Documentation](https://docs.microsoft.com/en-us/appcenter/distribution/codepush/)
- [App Center Documentation](https://docs.microsoft.com/en-us/appcenter/)
- [React Native CodePush](https://github.com/microsoft/react-native-code-push)
- [CodePush CLI Reference](https://docs.microsoft.com/en-us/appcenter/distribution/codepush/cli)
- [OTA Updates Best Practices](https://microsoft.github.io/code-push/docs/best-practices.html)

---

**Next Lesson**: [Lesson 53: CI/CD Pipeline Setup](Lesson%2053_%20CI_CD%20Pipeline%20Setup.md)