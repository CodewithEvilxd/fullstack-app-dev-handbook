# Lesson 51: App Store Deployment (iOS & Android)

## 🎯 **Learning Objectives**
- Master the complete iOS App Store deployment process
- Navigate Google Play Store submission requirements
- Configure app metadata, screenshots, and descriptions
- Handle app review processes and common rejection reasons
- Implement version management and release strategies

## 📚 **Table of Contents**
1. [iOS App Store Deployment](#ios-app-store-deployment)
2. [Android Google Play Deployment](#android-google-play-deployment)
3. [App Metadata & Assets](#app-metadata--assets)
4. [App Review Process](#app-review-process)
5. [Version Management](#version-management)
6. [Release Strategies](#release-strategies)
7. [Common Issues & Solutions](#common-issues--solutions)
8. [Post-Launch Management](#post-launch-management)
9. [Beta Testing](#beta-testing)
10. [Practical Examples](#practical-examples)

---

## 🍎 **iOS App Store Deployment**

### **Prerequisites**
```bash
# Install Fastlane
brew install fastlane

# Install Xcode command line tools
xcode-select --install

# Create App Store Connect API key
# 1. Go to App Store Connect → Users and Access → Keys
# 2. Generate a new API key with App Manager access
# 3. Download the .p8 file and note the Key ID
```

### **Fastlane Setup**
```ruby
# fastlane/Fastfile
default_platform(:ios)

platform :ios do
  desc "Build and deploy to TestFlight"
  lane :beta do
    # Setup certificates and provisioning profiles
    setup_ci if ENV['CI']

    # Match certificates and profiles
    match(
      type: "appstore",
      readonly: true
    )

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
          "com.myapp.ios" => "MyApp App Store"
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
    # Setup certificates and profiles
    setup_ci if ENV['CI']

    # Match certificates and profiles
    match(
      type: "appstore",
      readonly: true
    )

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

### **App Store Connect Configuration**
```ruby
# fastlane/Matchfile
git_url("https://github.com/myorg/certificates.git")
storage_mode("git")
type("appstore")

app_identifier(["com.myapp.ios"])
username("developer@mycompany.com")
team_id("ABC123DEF4")
```

### **TestFlight Distribution**
```ruby
desc "Distribute to TestFlight with detailed changelog"
lane :testflight_detailed do
  # Build the app
  build_app(
    scheme: "MyApp",
    export_method: "app-store"
  )

  # Upload with detailed changelog
  upload_to_testflight(
    distribute_external: true,
    groups: ["Beta Testers", "Internal Testers"],
    changelog: {
      "en-US" => "• Fixed login issues\n• Improved performance\n• Added new features",
      "es-ES" => "• Corregidos problemas de inicio de sesión\n• Mejorado rendimiento\n• Añadidas nuevas funciones"
    },
    beta_app_review_info: {
      contact_email: "beta@mycompany.com",
      contact_first_name: "John",
      contact_last_name: "Doe",
      contact_phone: "+1234567890",
      demo_account_name: "demo@mycompany.com",
      demo_account_password: "DemoPassword123",
      notes: "Demo account credentials for testing"
    }
  )
end
```

---

## 🤖 **Android Google Play Deployment**

### **Prerequisites**
```bash
# Install Fastlane
gem install fastlane

# Create Google Play service account
# 1. Go to Google Play Console → Settings → Developer account → API access
# 2. Create a new service account
# 3. Download the JSON key file
# 4. Grant necessary permissions
```

### **Fastlane Android Setup**
```ruby
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

  desc "Deploy to internal testing"
  lane :internal do
    gradle(
      task: "assemble",
      build_type: "Release",
      project_dir: "./android"
    )

    upload_to_play_store(
      track: "internal",
      skip_upload_aab: true,
      skip_upload_metadata: true,
      skip_upload_changelogs: true,
      skip_upload_images: true,
      skip_upload_screenshots: true
    )
  end
end
```

### **Google Play Console Configuration**
```json
// fastlane/GooglePlay.json
{
  "type": "service_account",
  "project_id": "my-app-project",
  "private_key_id": "abc123...",
  "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n",
  "client_email": "my-service-account@my-app-project.iam.gserviceaccount.com",
  "client_id": "123456789",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/my-service-account%40my-app-project.iam.gserviceaccount.com"
}
```

### **Gradle Configuration for Release**
```gradle
// android/app/build.gradle
android {
    signingConfigs {
        release {
            storeFile file('path/to/keystore.jks')
            storePassword System.getenv('KEYSTORE_PASSWORD')
            keyAlias System.getenv('KEY_ALIAS')
            keyPassword System.getenv('KEY_PASSWORD')
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            buildConfigField "boolean", "IS_PRODUCTION", "true"
        }
    }
}
```

---

## 📝 **App Metadata & Assets**

### **App Store Metadata**
```xml
<!-- ios/MyApp/Info.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleDisplayName</key>
    <string>My Awesome App</string>
    <key>CFBundleIdentifier</key>
    <string>com.mycompany.myapp</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0.0</string>
    <key>CFBundleVersion</key>
    <string>1</string>
    <key>LSRequiresIPhoneOS</key>
    <true/>
    <key>UIRequiredDeviceCapabilities</key>
    <array>
        <string>armv7</string>
    </array>
    <key>UISupportedInterfaceOrientations</key>
    <array>
        <string>UIInterfaceOrientationPortrait</string>
        <string>UIInterfaceOrientationLandscapeLeft</string>
        <string>UIInterfaceOrientationLandscapeRight</string>
    </array>
    <key>NSLocationWhenInUseUsageDescription</key>
    <string>This app needs location access to show nearby content</string>
    <key>NSCameraUsageDescription</key>
    <string>This app needs camera access to take photos</string>
    <key>NSPhotoLibraryUsageDescription</key>
    <string>This app needs photo library access to select images</string>
</dict>
</plist>
```

### **Android Manifest**
```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.mycompany.myapp">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <application
        android:name=".MainApplication"
        android:label="@string/app_name"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:allowBackup="true"
        android:theme="@style/AppTheme"
        android:usesCleartextTraffic="true"
        android:requestLegacyExternalStorage="true">

        <activity
            android:name=".MainActivity"
            android:label="@string/app_name"
            android:configChanges="keyboard|keyboardHidden|orientation|screenSize|uiMode"
            android:launchMode="singleTask"
            android:windowSoftInputMode="adjustResize"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <meta-data
            android:name="com.google.android.geo.API_KEY"
            android:value="YOUR_GOOGLE_MAPS_API_KEY"/>
    </application>
</manifest>
```

### **Screenshots and Assets**
```ruby
# fastlane/screenshots script
desc "Generate and upload screenshots"
lane :screenshots do
  # iOS screenshots
  capture_ios_screenshots(
    scheme: "MyAppUITests",
    output_directory: "./fastlane/screenshots/ios"
  )

  # Android screenshots
  capture_android_screenshots(
    device_type: "phone",
    output_directory: "./fastlane/screenshots/android"
  )

  # Upload to stores
  upload_to_app_store(
    skip_binary_upload: true,
    skip_metadata: false,
    skip_screenshots: false
  )

  upload_to_play_store(
    skip_upload_apk: true,
    skip_upload_aab: true,
    skip_upload_metadata: false,
    skip_upload_changelogs: true,
    skip_upload_images: false,
    skip_upload_screenshots: false
  )
end
```

---

## 🔍 **App Review Process**

### **Common App Review Issues**
```javascript
// Pre-submission checklist
const preSubmissionChecklist = {
  ios: {
    required: [
      'App icons in all required sizes',
      'Launch screen configured',
      'Privacy policy URL in App Store Connect',
      'Support URL configured',
      'Test account credentials for review',
      'No placeholder text or images',
      'Proper app permissions explanations',
      'No beta features enabled',
    ],
    recommended: [
      'App Store screenshots ready',
      'App description and keywords optimized',
      'Demo video prepared',
      'Crash reporting enabled',
      'Analytics configured',
    ]
  },
  android: {
    required: [
      'App icons and feature graphic',
      'Privacy policy link',
      'Content rating completed',
      'Target API level compliance',
      'Google Play billing configured (if using IAP)',
      'No debug code or test features',
    ],
    recommended: [
      'Store listing screenshots',
      'App description optimized',
      'Translation ready',
      'Beta testing configured',
    ]
  }
};
```

### **Handling App Review Feedback**
```javascript
// App review response automation
const handleAppReviewResponse = {
  commonRejections: {
    '2.1 - App Completeness': {
      response: 'Fixed incomplete app functionality',
      actions: ['Test all app flows', 'Remove placeholder content']
    },
    '2.3.1 - App Store Review Guideline': {
      response: 'Updated app to comply with guidelines',
      actions: ['Review guidelines', 'Update app content']
    },
    '3.1.1 - In-App Purchase': {
      response: 'Configured in-app purchases properly',
      actions: ['Set up IAP in App Store Connect', 'Test purchase flow']
    },
    '4.1 - Copycats': {
      response: 'Made app unique and original',
      actions: ['Redesign UI', 'Add unique features']
    },
    '5.1.1 - Privacy': {
      response: 'Updated privacy policy and permissions',
      actions: ['Update privacy policy', 'Review permission usage']
    }
  },

  prepareResponse: (rejectionReason, version) => {
    const response = commonRejections[rejectionReason];
    if (response) {
      return {
        message: response.response,
        actions: response.actions,
        version: version,
        timestamp: new Date().toISOString()
      };
    }
    return null;
  }
};
```

---

## 📊 **Version Management**

### **Semantic Versioning Strategy**
```javascript
// version management utility
const versionManager = {
  // Parse version string
  parseVersion: (versionString) => {
    const [major, minor, patch] = versionString.split('.').map(Number);
    return { major, minor, patch };
  },

  // Increment version
  incrementVersion: (currentVersion, type) => {
    const version = parseVersion(currentVersion);

    switch (type) {
      case 'major':
        return `${version.major + 1}.0.0`;
      case 'minor':
        return `${version.major}.${version.minor + 1}.0`;
      case 'patch':
        return `${version.major}.${version.minor}.${version.patch + 1}`;
      default:
        return currentVersion;
    }
  },

  // Determine release type based on changes
  determineReleaseType: (changes) => {
    if (changes.some(change => change.type === 'breaking')) {
      return 'major';
    }
    if (changes.some(change => change.type === 'feature')) {
      return 'minor';
    }
    return 'patch';
  },

  // Generate changelog
  generateChangelog: (version, changes) => {
    const sections = {
      'breaking': [],
      'feature': [],
      'fix': [],
      'docs': [],
      'style': [],
      'refactor': [],
      'test': [],
      'chore': []
    };

    changes.forEach(change => {
      if (sections[change.type]) {
        sections[change.type].push(change.description);
      }
    });

    let changelog = `# ${version}\n\n`;

    Object.entries(sections).forEach(([type, items]) => {
      if (items.length > 0) {
        changelog += `## ${type.charAt(0).toUpperCase() + type.slice(1)}\n`;
        items.forEach(item => {
          changelog += `- ${item}\n`;
        });
        changelog += '\n';
      }
    });

    return changelog;
  }
};
```

### **Build Number Management**
```ruby
# iOS build number management
desc "Increment build number"
lane :increment_build do
  current_build = get_build_number
  new_build = current_build.to_i + 1

  increment_build_number(
    build_number: new_build.to_s
  )

  # Update build number in other files if needed
  update_info_plist(
    plist_path: "MyApp/Info.plist",
    block: proc do |plist|
      plist["CFBundleVersion"] = new_build.to_s
    end
  )
end

# Android version code management
desc "Increment version code"
lane :increment_version do
  current_code = google_play_track_version_codes(
    track: "production"
  )[0]

  new_code = current_code + 1

  increment_version_code(
    gradle_file_path: "./android/app/build.gradle",
    version_code: new_code
  )
end
```

---

## 🚀 **Release Strategies**

### **Staged Rollout**
```ruby
# Staged rollout for iOS
desc "Deploy with staged rollout"
lane :staged_rollout do
  # Initial 10% rollout
  upload_to_testflight(
    distribute_external: true,
    groups: ["Beta Testers"],
    changelog: "Staged rollout - 10% of users"
  )

  # Wait and monitor
  sleep(3600) # Wait 1 hour

  # Increase to 25%
  # Note: TestFlight doesn't support percentage rollouts
  # This would be handled in App Store Connect manually
end

# Staged rollout for Android
desc "Android staged rollout"
lane :android_staged do
  # 10% rollout
  upload_to_play_store(
    track: "production",
    rollout: "0.1"
  )

  # Monitor crash rates and user feedback
  # Then increase rollout percentage
end
```

### **Feature Flags for Releases**
```javascript
// Feature flag system
const featureFlags = {
  // Define features
  features: {
    new_chat_ui: {
      enabled: false,
      rollout_percentage: 10,
      platforms: ['ios', 'android']
    },
    dark_mode: {
      enabled: true,
      rollout_percentage: 100,
      platforms: ['ios', 'android']
    },
    advanced_search: {
      enabled: false,
      rollout_percentage: 5,
      platforms: ['ios']
    }
  },

  // Check if feature is enabled for user
  isEnabled: (featureName, userId, platform) => {
    const feature = features[featureName];
    if (!feature || !feature.enabled) return false;

    // Check platform support
    if (!feature.platforms.includes(platform)) return false;

    // For 100% rollout, enable for all
    if (feature.rollout_percentage >= 100) return true;

    // For partial rollout, use user ID for consistent assignment
    const hash = hashString(userId + featureName);
    const percentage = (hash % 100) + 1;

    return percentage <= feature.rollout_percentage;
  },

  // Update feature flag
  updateFeature: (featureName, updates) => {
    if (features[featureName]) {
      features[featureName] = { ...features[featureName], ...updates };
    }
  },

  // Get all enabled features for user
  getEnabledFeatures: (userId, platform) => {
    return Object.keys(features).filter(featureName =>
      isEnabled(featureName, userId, platform)
    );
  }
};

// Simple hash function for consistent user assignment
const hashString = (str) => {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    const char = str.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;
    hash = hash & hash; // Convert to 32-bit integer
  }
  return Math.abs(hash);
};
```

---

## 🐛 **Common Issues & Solutions**

### **iOS Common Issues**
```javascript
const iosIssues = {
  'ITMS-90022': {
    issue: 'Missing required icon',
    solution: 'Add all required icon sizes to Images.xcassets'
  },
  'ITMS-90023': {
    issue: 'Missing screenshot',
    solution: 'Add at least one screenshot for each device size'
  },
  'ITMS-90809': {
    issue: 'Missing purpose string',
    solution: 'Add NSLocationWhenInUseUsageDescription to Info.plist'
  },
  'ITMS-90683': {
    issue: 'Missing demo account',
    solution: 'Provide test account credentials in App Store Connect'
  },
  'ITMS-90842': {
    issue: 'Invalid binary architecture',
    solution: 'Ensure app supports required architectures'
  }
};
```

### **Android Common Issues**
```javascript
const androidIssues = {
  'google-play-error-1': {
    issue: 'APK is not properly signed',
    solution: 'Ensure keystore is properly configured and signing works'
  },
  'google-play-error-2': {
    issue: 'Missing required permissions',
    solution: 'Add required permissions to AndroidManifest.xml'
  },
  'google-play-error-3': {
    issue: 'App bundle contains native code',
    solution: 'Ensure 64-bit native libraries are included'
  },
  'google-play-error-4': {
    issue: 'Privacy policy required',
    solution: 'Add privacy policy URL in Play Console'
  },
  'google-play-error-5': {
    issue: 'Content rating required',
    solution: 'Complete content rating questionnaire'
  }
};
```

---

## 📈 **Post-Launch Management**

### **App Analytics Setup**
```javascript
// Post-launch analytics tracking
const postLaunchAnalytics = {
  // Track app performance
  trackAppPerformance: () => {
    // Track app start time
    analytics.trackEvent('app_launch', {
      timestamp: new Date().toISOString(),
      platform: Platform.OS,
      version: '1.0.0'
    });

    // Track user engagement
    analytics.trackEvent('user_engagement', {
      session_duration: 0, // Will be updated on session end
      screens_viewed: 0,
      actions_performed: 0
    });
  },

  // Track user feedback
  trackUserFeedback: (rating, feedback) => {
    analytics.trackEvent('user_feedback', {
      rating,
      feedback,
      timestamp: new Date().toISOString(),
      platform: Platform.OS
    });
  },

  // Track crash reports
  trackCrash: (error, context) => {
    analytics.trackEvent('app_crash', {
      error_message: error.message,
      error_stack: error.stack,
      context,
      timestamp: new Date().toISOString(),
      platform: Platform.OS
    });
  },

  // Track feature usage
  trackFeatureUsage: (featureName, data = {}) => {
    analytics.trackEvent('feature_used', {
      feature_name: featureName,
      ...data,
      timestamp: new Date().toISOString()
    });
  }
};
```

### **User Feedback Collection**
```javascript
// In-app feedback system
const feedbackSystem = {
  // Show feedback prompt
  showFeedbackPrompt: () => {
    Alert.alert(
      'Enjoying the app?',
      'Please rate your experience',
      [
        {
          text: 'Not Really',
          onPress: () => showDetailedFeedback('negative')
        },
        {
          text: 'It\'s Okay',
          onPress: () => showDetailedFeedback('neutral')
        },
        {
          text: 'Love It!',
          onPress: () => showDetailedFeedback('positive')
        }
      ]
    );
  },

  // Detailed feedback form
  showDetailedFeedback: (sentiment) => {
    // Navigate to feedback screen or show modal
    navigation.navigate('Feedback', { sentiment });
  },

  // Submit feedback
  submitFeedback: async (feedbackData) => {
    try {
      await api.post('/feedback', feedbackData);

      // Track feedback submission
      analytics.trackEvent('feedback_submitted', {
        sentiment: feedbackData.sentiment,
        rating: feedbackData.rating,
        has_comment: !!feedbackData.comment
      });

      Alert.alert('Thank you!', 'Your feedback helps us improve the app.');
    } catch (error) {
      Alert.alert('Error', 'Failed to submit feedback. Please try again.');
    }
  }
};
```

---

## 🧪 **Beta Testing**

### **TestFlight Beta Testing**
```ruby
# Advanced TestFlight management
desc "Manage TestFlight beta testing"
lane :beta_management do
  # Add testers
  add_testers_to_testflight(
    emails: ["tester1@example.com", "tester2@example.com"],
    group_name: "Beta Testers"
  )

  # Create new beta group
  create_testflight_beta_group(
    group_name: "New Features",
    public_link_enabled: true,
    public_link_limit: 10000
  )

  # Get beta testing info
  beta_info = get_testflight_beta_info
  puts "Beta testers: #{beta_info.tester_count}"
  puts "Installs: #{beta_info.install_count}"
  puts "Sessions: #{beta_info.session_count}"
end

desc "Send beta testing invitation"
lane :invite_beta_testers do
  # Send invitation emails
  testflight(
    distribute_external: true,
    groups: ["Beta Testers"],
    notify_external_testers: true,
    changelog: "New beta version available for testing"
  )
end
```

### **Google Play Beta Testing**
```ruby
# Google Play beta management
desc "Setup Google Play beta testing"
lane :setup_beta do
  # Create beta track
  upload_to_play_store(
    track: "beta",
    rollout: "0.1"
  )

  # Add beta testers
  # Note: Testers are managed in Play Console
  puts "Beta testers can be added in Google Play Console"
end

desc "Promote beta to production"
lane :promote_beta do
  # Check beta feedback and crash rates
  # If everything looks good, promote to production

  upload_to_play_store(
    track: "beta",
    track_promote_to: "production",
    rollout: "0.1" # Start with 10% production rollout
  )
end
```

---

## 🎯 **Practical Examples**

### **Complete Deployment Script**
```bash
#!/bin/bash
# deploy.sh - Complete deployment script

set -e  # Exit on any error

# Configuration
APP_NAME="MyApp"
ENVIRONMENT=$1  # staging, production
PLATFORM=$2     # ios, android, both

if [ -z "$ENVIRONMENT" ]; then
  echo "Usage: $0 <environment> [platform]"
  echo "Environments: staging, production"
  echo "Platforms: ios, android, both"
  exit 1
fi

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log() {
  echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] $1${NC}"
}

error() {
  echo -e "${RED}[ERROR] $1${NC}"
}

warning() {
  echo -e "${YELLOW}[WARNING] $1${NC}"
}

# Pre-deployment checks
pre_deployment_checks() {
  log "Running pre-deployment checks..."

  # Check if required tools are installed
  if ! command -v node &> /dev/null; then
    error "Node.js is not installed"
    exit 1
  fi

  if ! command -v npm &> /dev/null; then
    error "npm is not installed"
    exit 1
  fi

  # Check if environment variables are set
  if [ -z "$APP_STORE_CONNECT_API_KEY" ] && [ "$PLATFORM" = "ios" ]; then
    error "APP_STORE_CONNECT_API_KEY is not set"
    exit 1
  fi

  if [ -z "$GOOGLE_PLAY_SERVICE_ACCOUNT" ] && [ "$PLATFORM" = "android" ]; then
    error "GOOGLE_PLAY_SERVICE_ACCOUNT is not set"
    exit 1
  fi

  # Run tests
  log "Running tests..."
  npm run test -- --coverage --watchAll=false

  # Build check
  log "Running build check..."
  npm run build

  log "Pre-deployment checks completed"
}

# iOS deployment
deploy_ios() {
  log "Starting iOS deployment to $ENVIRONMENT..."

  cd ios

  # Install CocoaPods
  pod install

  cd ..

  # Use Fastlane
  if [ "$ENVIRONMENT" = "staging" ]; then
    npx fastlane ios beta
  else
    npx fastlane ios release
  fi

  log "iOS deployment completed"
}

# Android deployment
deploy_android() {
  log "Starting Android deployment to $ENVIRONMENT..."

  if [ "$ENVIRONMENT" = "staging" ]; then
    npx fastlane android beta
  else
    npx fastlane android production
  fi

  log "Android deployment completed"
}

# Main deployment function
main() {
  log "Starting deployment of $APP_NAME to $ENVIRONMENT..."

  # Pre-deployment checks
  pre_deployment_checks

  # Deploy based on platform
  case $PLATFORM in
    ios)
      deploy_ios
      ;;
    android)
      deploy_android
      ;;
    both)
      deploy_ios
      deploy_android
      ;;
    *)
      error "Invalid platform: $PLATFORM"
      echo "Valid platforms: ios, android, both"
      exit 1
      ;;
  esac

  log "Deployment completed successfully! 🎉"

  # Post-deployment notifications
  if [ -n "$SLACK_WEBHOOK" ]; then
    curl -X POST -H 'Content-type: application/json' \
      --data "{\"text\":\"🚀 $APP_NAME deployed to $ENVIRONMENT successfully!\"}" \
      $SLACK_WEBHOOK
  fi
}

# Run main function
main "$@"
```

### **Automated Release Process**
```javascript
// scripts/release.js
const { execSync } = require('child_process');
const fs = require('fs');
const path = require('path');

class ReleaseManager {
  constructor() {
    this.packageJson = JSON.parse(fs.readFileSync('package.json', 'utf8'));
    this.currentVersion = this.packageJson.version;
  }

  // Create release branch
  createReleaseBranch(version) {
    const branchName = `release/v${version}`;
    execSync(`git checkout -b ${branchName}`);
    console.log(`Created release branch: ${branchName}`);
  }

  // Update version in package.json
  updateVersion(newVersion) {
    this.packageJson.version = newVersion;
    fs.writeFileSync('package.json', JSON.stringify(this.packageJson, null, 2));
    console.log(`Updated version to: ${newVersion}`);
  }

  // Update version in native files
  updateNativeVersions(newVersion) {
    // iOS
    const iosPlist = path.join('ios', 'MyApp', 'Info.plist');
    if (fs.existsSync(iosPlist)) {
      // Update iOS version (simplified)
      console.log('Updated iOS version');
    }

    // Android
    const androidGradle = path.join('android', 'app', 'build.gradle');
    if (fs.existsSync(androidGradle)) {
      // Update Android version (simplified)
      console.log('Updated Android version');
    }
  }

  // Generate changelog
  generateChangelog(version) {
    const changelog = `# ${version}\n\n## Changes\n- New features\n- Bug fixes\n- Performance improvements\n`;
    const changelogPath = 'CHANGELOG.md';

    let existingChangelog = '';
    if (fs.existsSync(changelogPath)) {
      existingChangelog = fs.readFileSync(changelogPath, 'utf8');
    }

    fs.writeFileSync(changelogPath, changelog + '\n' + existingChangelog);
    console.log('Generated changelog');
  }

  // Commit changes
  commitChanges(version) {
    execSync('git add .');
    execSync(`git commit -m "Release v${version}"`);
    console.log('Committed changes');
  }

  // Create git tag
  createTag(version) {
    execSync(`git tag v${version}`);
    console.log(`Created tag: v${version}`);
  }

  // Push to repository
  pushChanges(version) {
    execSync('git push origin release/v${version}');
    execSync('git push origin v${version}');
    console.log('Pushed changes and tag');
  }

  // Create GitHub release
  createGitHubRelease(version) {
    // This would use GitHub API to create a release
    console.log(`Created GitHub release: v${version}`);
  }

  // Main release process
  async release(newVersion, options = {}) {
    try {
      console.log(`Starting release process for version ${newVersion}...`);

      // Validate version format
      if (!/^\d+\.\d+\.\d+$/.test(newVersion)) {
        throw new Error('Invalid version format. Use semantic versioning (e.g., 1.2.3)');
      }

      // Create release branch
      this.createReleaseBranch(newVersion);

      // Update versions
      this.updateVersion(newVersion);
      this.updateNativeVersions(newVersion);

      // Generate changelog
      this.generateChangelog(newVersion);

      // Commit and tag
      this.commitChanges(newVersion);
      this.createTag(newVersion);

      // Push changes
      this.pushChanges(newVersion);

      // Create GitHub release
      if (options.createGitHubRelease) {
        this.createGitHubRelease(newVersion);
      }

      console.log(`✅ Release ${newVersion} completed successfully!`);

      return {
        success: true,
        version: newVersion,
        branch: `release/v${newVersion}`,
        tag: `v${newVersion}`
      };

    } catch (error) {
      console.error('❌ Release failed:', error.message);
      return {
        success: false,
        error: error.message
      };
    }
  }
}

// CLI usage
if (require.main === module) {
  const args = process.argv.slice(2);
  const newVersion = args[0];

  if (!newVersion) {
    console.error('Usage: node release.js <version>');
    process.exit(1);
  }

  const releaseManager = new ReleaseManager();
  releaseManager.release(newVersion, {
    createGitHubRelease: true
  });
}

module.exports = ReleaseManager;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **iOS App Store Deployment**: Complete TestFlight and App Store submission process
- ✅ **Android Google Play Deployment**: Play Store submission and management
- ✅ **App Metadata & Assets**: Screenshots, descriptions, and store listings
- ✅ **App Review Process**: Handling rejections and resubmissions
- ✅ **Version Management**: Semantic versioning and build numbers
- ✅ **Release Strategies**: Staged rollouts and feature flags
- ✅ **Common Issues & Solutions**: Troubleshooting deployment problems
- ✅ **Post-Launch Management**: Analytics and user feedback
- ✅ **Beta Testing**: TestFlight and Google Play beta programs
- ✅ **Automated Deployment**: Scripts and CI/CD integration

### **Best Practices**
1. **Test thoroughly before submission** - Use beta testing extensively
2. **Prepare all required assets** - Screenshots, descriptions, privacy policies
3. **Handle app review feedback** - Address issues promptly and professionally
4. **Use semantic versioning** - Clear version numbering system
5. **Automate deployment process** - Use Fastlane and CI/CD pipelines
6. **Monitor post-launch metrics** - Track crashes, ratings, and user feedback
7. **Plan rollback strategies** - Be prepared to revert releases
8. **Document release process** - Maintain clear deployment documentation
9. **Secure sensitive data** - Never commit API keys or certificates
10. **Communicate with users** - Keep users informed about updates

### **Next Steps**
- Learn advanced deployment patterns like blue-green deployments
- Implement automated testing in CI/CD pipelines
- Set up monitoring and alerting for production apps
- Learn about app store optimization (ASO)
- Explore enterprise app distribution
- Study compliance requirements for different markets

---

## 🎯 **Assignment**

### **Task 1: Complete App Store Setup**
Set up complete deployment infrastructure for both iOS and Android:
- Configure Fastlane for automated deployment
- Set up App Store Connect and Google Play Console accounts
- Prepare all required app assets and metadata
- Configure beta testing programs
- Create deployment scripts and documentation

### **Task 2: App Submission Process**
Complete the full app submission process:
- Build production-ready app binaries
- Submit to beta testing programs
- Address any app review feedback
- Launch to production with staged rollout
- Monitor post-launch performance and user feedback

### **Task 3: Release Management System**
Implement a comprehensive release management system:
- Automated version bumping and changelog generation
- Git flow integration with release branches
- CI/CD pipeline for automated testing and deployment
- Rollback procedures and emergency fixes
- Release tracking and audit trails

---

## 📚 **Additional Resources**
- [App Store Connect Documentation](https://developer.apple.com/support/app-store-connect/)
- [Google Play Console Help](https://support.google.com/googleplay/android-developer)
- [Fastlane Documentation](https://docs.fastlane.tools/)
- [iOS App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)
- [Google Play Policies](https://play.google.com/about/developer-content-policy/)

---

**Next Lesson**: [Lesson 52: CodePush for OTA Updates](Lesson%2052_%20CodePush%20for%20OTA%20Updates.md)