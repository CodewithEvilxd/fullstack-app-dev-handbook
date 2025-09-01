# 🚀 Fastlane Setup Guide for React Native
## Complete CI/CD Automation Manual

---

## 🎯 **Overview**
Fastlane is a tool for automating mobile app deployment. It automates tedious tasks like generating screenshots, dealing with code signing, and releasing apps. This guide covers complete Fastlane setup, configuration, and usage for React Native projects.

---

## 📋 **Table of Contents**
1. [What is Fastlane?](#what-is-fastlane)
2. [Installation & Setup](#installation--setup)
3. [Basic Configuration](#basic-configuration)
4. [iOS Deployment](#ios-deployment)
5. [Android Deployment](#android-deployment)
6. [Screenshots & Assets](#screenshots--assets)
7. [Code Signing](#code-signing)
8. [Testing Automation](#testing-automation)
9. [Advanced Features](#advanced-features)
10. [Troubleshooting](#troubleshooting)
11. [Best Practices](#best-practices)

---

## 🤔 **What is Fastlane?**

### **Fastlane Features:**
```bash
✅ Automated app store deployments
✅ Code signing management
✅ Screenshot generation
✅ Beta testing distribution
✅ Testing automation
✅ Release management
✅ Cross-platform support
✅ Plugin ecosystem
```

### **Why Use Fastlane:**
```bash
🚀 Automate repetitive tasks
⚡ Speed up deployment process
🔒 Secure code signing
📱 Cross-platform deployment
🧪 Automated testing
📊 Release analytics
🤖 CI/CD integration
📝 Documentation generation
```

### **Fastlane vs Other CI/CD Tools:**
```bash
Fastlane:
✅ Mobile-focused
✅ Built-in app store integrations
✅ Code signing automation
✅ Screenshot generation
❌ Steeper learning curve
❌ Ruby-based

GitHub Actions:
✅ Platform-agnostic
✅ Easy to learn
✅ Great documentation
❌ Less mobile-specific features
❌ Manual app store setup

Bitrise:
✅ Mobile-optimized
✅ Visual workflow editor
✅ Great for teams
❌ Paid service
❌ Less customizable
```

---

## 📥 **Installation & Setup**

### **System Requirements**
```bash
✅ Ruby 2.7+ (comes pre-installed on macOS)
✅ Bundler gem
✅ Xcode Command Line Tools (macOS)
✅ Android SDK (for Android builds)
✅ Git
✅ App Store Connect API access (iOS)
✅ Google Play Console access (Android)
```

### **Installation Methods**

#### **Method 1: System Installation**
```bash
# Install Fastlane gem
sudo gem install fastlane

# Or using Homebrew (macOS)
brew install fastlane

# Verify installation
fastlane --version

# Update Fastlane
sudo gem update fastlane
```

#### **Method 2: Project-Specific Installation**
```bash
# Create Gemfile in project root
# Gemfile
source "https://rubygems.org"

gem "fastlane"

# Install dependencies
bundle install

# Use Fastlane via bundle
bundle exec fastlane --version
```

#### **Method 3: Docker Installation**
```dockerfile
# Dockerfile
FROM ruby:3.1-alpine

RUN gem install fastlane

# Copy project files
COPY . /app
WORKDIR /app

CMD ["fastlane", "beta"]
```

### **Initial Setup**
```bash
# Navigate to React Native project
cd MyReactNativeApp

# Initialize Fastlane
fastlane init

# Or initialize for specific platforms
fastlane init ios
fastlane init android

# This creates:
# fastlane/
# ├── Appfile
# ├── Fastfile
# ├── Pluginfile (optional)
# └── README.md
```

---

## ⚙️ **Basic Configuration**

### **Appfile Configuration**
```ruby
# fastlane/Appfile
app_identifier("com.mycompany.myapp") # The bundle identifier of your app
apple_id("developer@mycompany.com") # Your Apple Developer Portal username

itc_team_id("123456789") # App Store Connect Team ID
team_id("ABCDEF1234") # Developer Portal Team ID

# For Android
json_key_file("./google-play-service-account.json") # Path to Google Play service account JSON
package_name("com.mycompany.myapp") # Android package name
```

### **Fastfile Basic Structure**
```ruby
# fastlane/Fastfile
default_platform(:ios)

platform :ios do
  desc "Push a new beta build to TestFlight"
  lane :beta do
    increment_build_number
    build_app
    upload_to_testflight
  end

  desc "Deploy a new version to the App Store"
  lane :release do
    increment_version_number
    build_app
    upload_to_app_store
  end
end

platform :android do
  desc "Deploy a new version to Google Play Beta"
  lane :beta do
    increment_version_code
    build_android_app
    upload_to_play_store(track: 'beta')
  end
end
```

### **Environment Variables**
```bash
# .env file (add to .gitignore)
FASTLANE_USER=developer@mycompany.com
FASTLANE_PASSWORD=your-app-specific-password
FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD=your-app-specific-password
GOOGLE_PLAY_JSON_KEY_FILE=./google-play-service-account.json
```

### **Directory Structure**
```bash
MyReactNativeApp/
├── android/
├── ios/
├── fastlane/
│   ├── Appfile
│   ├── Fastfile
│   ├── Matchfile (for code signing)
│   ├── Gymfile (for iOS builds)
│   ├── Deliverfile (for App Store)
│   ├── Supplyfile (for Play Store)
│   ├── Screengrabfile (for screenshots)
│   └── metadata/ (store metadata)
├── Gemfile
└── .env
```

---

## 🍎 **iOS Deployment**

### **iOS Fastfile Configuration**
```ruby
# fastlane/Fastfile
platform :ios do
  before_all do
    setup_ci if ENV['CI']
  end

  desc "Load ASC API Key information to use in subsequent lanes"
  lane :load_asc_api_key do
    app_store_connect_api_key(
      key_id: ENV["ASC_KEY_ID"],
      issuer_id: ENV["ASC_ISSUER_ID"],
      key_filepath: ENV["ASC_KEY_FILEPATH"],
      in_house: false
    )
  end

  desc "Bump build number based on most recent TestFlight build number"
  lane :fetch_and_increment_build_number do
    app_store_connect_api_key
    current_version = get_version_number
    latest_build_number = latest_testflight_build_number(
      api_key: lane_context[SharedValues::APP_STORE_CONNECT_API_KEY],
      version: current_version
    )
    increment_build_number(build_number: latest_build_number + 1)
  end

  desc "Check certs and profiles"
  lane :prepare_signing do
    match(type: "appstore")
  end

  desc "Build the iOS application"
  lane :build_release do
    prepare_signing
    build_app(
      scheme: "MyApp",
      export_method: "app-store",
      configuration: "Release"
    )
  end

  desc "Upload to TestFlight"
  lane :upload_testflight do
    build_release
    upload_to_testflight(
      api_key: lane_context[SharedValues::APP_STORE_CONNECT_API_KEY],
      skip_waiting_for_build_processing: true
    )
  end

  desc "Deploy to App Store"
  lane :release do
    load_asc_api_key
    fetch_and_increment_build_number
    prepare_signing
    build_release
    upload_to_app_store(
      api_key: lane_context[SharedValues::APP_STORE_CONNECT_API_KEY],
      skip_screenshots: true,
      skip_metadata: true
    )
  end
end
```

### **App Store Connect API Setup**
```bash
# Generate API Key from App Store Connect
# App Store Connect → Users and Access → Keys
# Download .p8 file

# Environment variables
export ASC_KEY_ID="your-key-id"
export ASC_ISSUER_ID="your-issuer-id"
export ASC_KEY_FILEPATH="./AuthKey.p8"
```

### **Code Signing with Match**
```ruby
# fastlane/Matchfile
git_url("git@github.com:myorg/certificates.git")
storage_mode("git")
type("appstore")
app_identifier(["com.mycompany.myapp"])
username("developer@mycompany.com")
```

### **TestFlight Deployment**
```bash
# Run TestFlight deployment
fastlane ios upload_testflight

# Or with specific parameters
fastlane ios upload_testflight beta_email:beta@mycompany.com

# Check TestFlight builds
fastlane pilot builds
```

---

## 🤖 **Android Deployment**

### **Android Fastfile Configuration**
```ruby
# fastlane/Fastfile
platform :android do
  desc "Deploy a new version to Google Play Beta"
  lane :beta do
    increment_version_code
    build_android_app(
      task: 'bundle',
      build_type: 'Release'
    )
    upload_to_play_store(
      track: 'beta',
      skip_upload_apk: true,
      skip_upload_aab: false
    )
  end

  desc "Deploy a new version to Google Play Production"
  lane :release do
    increment_version_code
    build_android_app(
      task: 'bundle',
      build_type: 'Release'
    )
    upload_to_play_store(
      track: 'production',
      skip_upload_apk: true,
      skip_upload_aab: false
    )
  end

  desc "Build Android APK for testing"
  lane :build_apk do
    increment_version_code
    build_android_app(
      task: 'assemble',
      build_type: 'Release'
    )
  end

  desc "Run tests"
  lane :test do
    gradle(task: "test")
  end
end
```

### **Google Play Console Setup**
```bash
# Create service account
# Google Cloud Console → IAM & Admin → Service Accounts
# Create service account with Play Console access
# Download JSON key file

# Grant permissions in Play Console
# Play Console → Settings → Developer account → Users and permissions
# Add service account email
# Grant release manager permissions
```

### **Supply Configuration**
```ruby
# fastlane/Supplyfile
package_name "com.mycompany.myapp"
json_key "./google-play-service-account.json"
```

### **AAB vs APK Deployment**
```ruby
# Build Android App Bundle (recommended)
lane :build_aab do
  build_android_app(
    task: 'bundle',
    build_type: 'Release',
    print_command: false,
    properties: {
      "android.injected.signing.store.file" => ENV['ANDROID_KEYSTORE_FILE'],
      "android.injected.signing.store.password" => ENV['ANDROID_KEYSTORE_PASSWORD'],
      "android.injected.signing.key.alias" => ENV['ANDROID_KEY_ALIAS'],
      "android.injected.signing.key.password" => ENV['ANDROID_KEY_PASSWORD'],
    }
  )
end

# Build APK (legacy)
lane :build_apk do
  build_android_app(
    task: 'assemble',
    build_type: 'Release'
  )
end
```

---

## 📸 **Screenshots & Assets**

### **Screenshot Generation**
```ruby
# fastlane/Snapfile (iOS)
devices([
  "iPhone 13 Pro Max",
  "iPhone 8 Plus",
  "iPad Pro (12.9-inch) (5th generation)"
])

languages([
  "en-US",
  "es-ES",
  "fr-FR",
  "de-DE"
])

scheme("MyAppUITests")
output_directory("./fastlane/screenshots")
clear_previous_screenshots(true)
```

### **Asset Management**
```ruby
# fastlane/Deliverfile (iOS)
app_identifier "com.mycompany.myapp"
username "developer@mycompany.com"
app_version "1.0.0"
app_icon "./assets/app_icon.png"
apple_watch_app_icon "./assets/watch_icon.png"
screenshots_path "./fastlane/screenshots"
metadata_path "./fastlane/metadata"
```

### **Metadata Management**
```bash
# fastlane/metadata directory structure
fastlane/
└── metadata/
    ├── en-US/
    │   ├── description.txt
    │   ├── keywords.txt
    │   ├── name.txt
    │   ├── privacy_url.txt
    │   ├── support_url.txt
    │   ├── marketing_url.txt
    │   └── release_notes.txt
    └── es-ES/
        └── ...
```

### **Automated Screenshot Generation**
```ruby
# iOS screenshot lane
lane :screenshots do
  capture_screenshots
  frame_screenshots
end

# Android screenshot lane
lane :screenshots do
  capture_android_screenshots
end
```

---

## 🔐 **Code Signing**

### **iOS Code Signing**
```ruby
# Using Match for automated code signing
lane :setup_signing do
  match(
    type: "appstore",
    readonly: true,
    app_identifier: "com.mycompany.myapp"
  )
end

# Manual code signing setup
lane :setup_manual_signing do
  update_code_signing_settings(
    use_automatic_signing: false,
    path: "MyApp.xcodeproj",
    code_sign_identity: "iPhone Distribution",
    profile_name: "MyApp App Store"
  )
end
```

### **Android Code Signing**
```ruby
# Android signing configuration
lane :build_signed_apk do
  build_android_app(
    task: 'assemble',
    build_type: 'Release',
    properties: {
      "android.injected.signing.store.file" => ENV['ANDROID_KEYSTORE_FILE'],
      "android.injected.signing.store.password" => ENV['ANDROID_KEYSTORE_PASSWORD'],
      "android.injected.signing.key.alias" => ENV['ANDROID_KEY_ALIAS'],
      "android.injected.signing.key.password" => ENV['ANDROID_KEY_PASSWORD'],
    }
  )
end
```

### **Certificate Management**
```bash
# iOS certificate management
fastlane match development
fastlane match appstore

# Android keystore generation
keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
```

---

## 🧪 **Testing Automation**

### **Unit Testing**
```ruby
# iOS testing lane
lane :test_ios do
  run_tests(
    scheme: "MyApp",
    device: "iPhone 13",
    clean: true
  )
end

# Android testing lane
lane :test_android do
  gradle(
    task: "test",
    build_type: "Release"
  )
end
```

### **UI Testing**
```ruby
# iOS UI testing
lane :ui_test_ios do
  run_tests(
    scheme: "MyAppUITests",
    device: "iPhone 13",
    clean: true,
    output_directory: "./fastlane/test_output"
  )
end

# Android UI testing
lane :ui_test_android do
  gradle(
    task: "connectedAndroidTest",
    build_type: "Release"
  )
end
```

### **Test Reporting**
```ruby
# Generate test reports
lane :test_report do
  run_tests(
    scheme: "MyApp",
    device: "iPhone 13",
    output_directory: "./fastlane/test_output",
    result_bundle: true,
    code_coverage: true
  )
end
```

---

## 🚀 **Advanced Features**

### **Custom Actions**
```ruby
# fastlane/actions/custom_action.rb
module Fastlane
  module Actions
    class CustomActionAction < Action
      def self.run(params)
        UI.message("Running custom action...")

        # Your custom logic here
        result = `echo "Custom action executed"`

        UI.success("Custom action completed!")
        return result
      end

      def self.description
        "A custom action for React Native projects"
      end

      def self.available_options
        [
          FastlaneCore::ConfigItem.new(
            key: :message,
            description: "Message to display",
            optional: true,
            default_value: "Hello World"
          )
        ]
      end
    end
  end
end
```

### **Plugin System**
```ruby
# fastlane/Pluginfile
gem 'fastlane-plugin-versioning'
gem 'fastlane-plugin-badge'
gem 'fastlane-plugin-sentry'
gem 'fastlane-plugin-slack'
```

### **Slack Integration**
```ruby
# Slack notifications
lane :notify_slack do
  slack(
    message: "App successfully deployed! 🎉",
    success: true,
    slack_url: ENV["SLACK_WEBHOOK_URL"],
    payload: {
      "Build Date" => Time.new.to_s,
      "Built by" => "Fastlane",
    }
  )
end
```

### **Version Management**
```ruby
# Version bumping
lane :bump_version do
  increment_version_number(
    bump_type: "patch" # or "minor", "major"
  )
  increment_build_number
end

# Version from git tags
lane :version_from_git do
  version = last_git_tag
  increment_version_number(version_number: version)
end
```

### **Environment-Specific Builds**
```ruby
# Environment-specific lanes
lane :build_staging do
  build_app(
    scheme: "MyApp Staging",
    configuration: "Staging",
    export_method: "ad-hoc"
  )
end

lane :build_production do
  build_app(
    scheme: "MyApp",
    configuration: "Release",
    export_method: "app-store"
  )
end
```

---

## 🔧 **Troubleshooting**

### **Common Issues & Solutions**

#### **Issue 1: Fastlane Command Not Found**
```bash
# Check if Fastlane is installed
gem list fastlane

# Reinstall Fastlane
sudo gem uninstall fastlane
sudo gem install fastlane

# Use bundle exec
bundle exec fastlane --version
```

#### **Issue 2: iOS Build Failures**
```bash
# Clean CocoaPods
cd ios && rm -rf Pods Podfile.lock
pod install

# Clean Xcode build
cd ios && rm -rf build
xcodebuild clean

# Check provisioning profiles
fastlane match appstore
```

#### **Issue 3: Android Build Failures**
```bash
# Clean Gradle cache
cd android && ./gradlew clean

# Clear React Native cache
npx react-native start --reset-cache

# Check keystore configuration
echo $ANDROID_KEYSTORE_FILE
```

#### **Issue 4: Code Signing Issues**
```bash
# iOS code signing
fastlane match development --force

# Android signing
# Verify keystore file exists
ls -la android/app/my-release-key.keystore
```

#### **Issue 5: App Store Upload Issues**
```bash
# Check API key permissions
# Verify bundle identifier
# Check app version and build number
fastlane pilot builds

# Manual upload for debugging
fastlane deliver --force
```

### **Debugging Fastlane**
```bash
# Verbose output
fastlane ios beta --verbose

# Dry run
fastlane ios beta --dry_run

# Interactive mode
fastlane ios beta --interactive

# Check Fastlane environment
fastlane env
```

---

## 📚 **Best Practices**

### **Fastfile Organization**
```ruby
# fastlane/Fastfile
# Use descriptive lane names
# Group related lanes
# Add comments and descriptions
# Use environment variables for sensitive data
# Implement error handling

platform :ios do
  error do |lane, exception|
    slack(
      message: "Fastlane failed with error: #{exception.message}",
      success: false
    )
  end

  desc "Complete beta deployment pipeline"
  lane :beta_pipeline do
    prepare_release
    test
    build
    deploy_beta
    notify_team
  end
end
```

### **Security Best Practices**
```bash
# Never commit sensitive data
# Use environment variables
# Encrypt sensitive files
# Use secure credential storage
# Rotate API keys regularly
# Limit access to deployment credentials
```

### **CI/CD Integration**
```yaml
# .github/workflows/deploy.yml
name: Deploy to Stores

on:
  push:
    tags:
      - 'v*'

jobs:
  deploy-ios:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.1'
      - run: bundle install
      - run: fastlane ios release
        env:
          ASC_KEY_ID: ${{ secrets.ASC_KEY_ID }}
          ASC_ISSUER_ID: ${{ secrets.ASC_ISSUER_ID }}
          ASC_KEY_FILEPATH: ./fastlane/AuthKey.p8

  deploy-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.1'
      - run: bundle install
      - run: fastlane android release
        env:
          GOOGLE_PLAY_JSON_KEY: ${{ secrets.GOOGLE_PLAY_JSON_KEY }}
```

### **Version Control**
```bash
# Commit Fastlane files
git add fastlane/
git commit -m "Add Fastlane configuration"

# Ignore sensitive files
echo ".env" >> .gitignore
echo "fastlane/AuthKey.p8" >> .gitignore
echo "google-play-service-account.json" >> .gitignore
```

---

## 🚀 **Advanced Configuration**

### **Multi-Environment Setup**
```ruby
# fastlane/Fastfile
platform :ios do
  desc "Deploy to staging environment"
  lane :staging do
    # Staging-specific configuration
    app_identifier = "com.mycompany.myapp.staging"
    scheme = "MyApp Staging"

    build_and_deploy(app_identifier, scheme, "staging")
  end

  desc "Deploy to production environment"
  lane :production do
    # Production-specific configuration
    app_identifier = "com.mycompany.myapp"
    scheme = "MyApp"

    build_and_deploy(app_identifier, scheme, "production")
  end

  def build_and_deploy(app_identifier, scheme, environment)
    # Common deployment logic
    increment_build_number
    build_app(scheme: scheme)
    upload_to_testflight if environment == "staging"
    upload_to_app_store if environment == "production"
  end
end
```

### **Automated Testing Pipeline**
```ruby
# fastlane/Fastfile
lane :full_test_suite do
  # Unit tests
  run_tests(scheme: "MyAppTests")

  # UI tests
  run_tests(scheme: "MyAppUITests")

  # Integration tests
  run_tests(scheme: "MyAppIntegrationTests")

  # Code coverage
  xcov(
    scheme: "MyApp",
    output_directory: "./fastlane/test_output"
  )
end
```

### **Release Management**
```ruby
# fastlane/Fastfile
lane :prepare_release do
  # Update version numbers
  increment_version_number(bump_type: "minor")
  increment_build_number

  # Update changelog
  update_changelog

  # Generate release notes
  generate_release_notes

  # Create git tag
  add_git_tag(tag: "v#{get_version_number}")
end

lane :update_changelog do
  # Read current changelog
  # Add new version entry
  # Write back to file
end
```

### **Monitoring & Analytics**
```ruby
# fastlane/Fastfile
lane :deploy_with_monitoring do
  start_time = Time.now

  begin
    # Deployment logic
    beta

    # Calculate deployment time
    deployment_time = Time.now - start_time

    # Send analytics
    send_deployment_analytics(deployment_time)

  rescue => exception
    # Handle deployment failure
    handle_deployment_failure(exception)
  end
end

def send_deployment_analytics(deployment_time)
  # Send to analytics service
  # Track deployment metrics
  # Monitor success rates
end
```

---

## 📊 **Monitoring & Analytics**

### **Deployment Metrics**
```ruby
# Track deployment metrics
lane :track_deployment do
  deployment_data = {
    platform: lane_context[:PLATFORM_NAME],
    version: get_version_number,
    build: get_build_number,
    start_time: Time.now,
    environment: ENV['DEPLOY_ENV'] || 'production'
  }

  # Store deployment data
  # Send to monitoring service
  # Update dashboards
end
```

### **Success Rate Tracking**
```ruby
# fastlane/Fastfile
before_all do |lane, options|
  @deployment_start = Time.now
  @deployment_success = false
end

after_all do |lane, options|
  deployment_duration = Time.now - @deployment_start

  # Record deployment result
  record_deployment_result(
    lane: lane,
    success: @deployment_success,
    duration: deployment_duration
  )
end

error do |lane, exception, options|
  @deployment_success = false
  # Handle deployment failure
end
```

### **Automated Reporting**
```ruby
# Generate deployment reports
lane :generate_report do
  # Collect deployment data
  deployments = get_recent_deployments

  # Generate HTML report
  generate_html_report(deployments)

  # Send report via email
  send_report_email
end
```

---

## 🎯 **Conclusion**

Fastlane is a powerful automation tool that can significantly streamline your React Native deployment process. With proper setup and configuration, it can handle everything from code signing to app store deployments.

### **Key Takeaways:**
- ✅ Install Fastlane using Ruby gems or Bundler
- ✅ Configure Appfile and Fastfile for your project
- ✅ Set up automated iOS and Android deployments
- ✅ Implement proper code signing and security
- ✅ Use Fastlane plugins for extended functionality
- ✅ Integrate with CI/CD pipelines
- ✅ Monitor deployment success and performance

### **Next Steps:**
1. **Install**: Set up Fastlane in your React Native project
2. **Configure**: Create Fastfile with basic deployment lanes
3. **Test**: Run test deployments to staging environments
4. **Automate**: Set up CI/CD pipelines with Fastlane
5. **Monitor**: Track deployment metrics and success rates
6. **Scale**: Implement advanced features and custom actions

### **Additional Resources:**
- [Fastlane Documentation](https://docs.fastlane.tools/)
- [Fastlane for iOS](https://docs.fastlane.tools/getting-started/ios/setup/)
- [Fastlane for Android](https://docs.fastlane.tools/getting-started/android/setup/)
- [Fastlane Plugins](https://docs.fastlane.tools/plugins/available-plugins/)
- [Fastlane Community](https://github.com/fastlane/fastlane)

---

*Happy Deploying! 🚀📱*