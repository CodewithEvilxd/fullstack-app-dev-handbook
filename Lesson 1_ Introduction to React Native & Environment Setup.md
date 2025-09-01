# 📱 Lesson 1: Introduction to React Native & Environment Setup

## 🎯 Learning Objectives
Is lesson ke baad aap:
- React Native kya hai aur kaise kaam karta hai samjhenge
- Development environment setup kar sakte hain
- First React Native app create kar sakte hain
- Basic project structure samjhenge

---

## 📖 What is React Native?

### Definition
React Native ek open-source mobile application framework hai jo Facebook (Meta) ne develop kiya hai. Yeh JavaScript aur React ka use karke native mobile apps banane ki facility deta hai.

### Key Features
1. **Cross-Platform Development**: Ek codebase se iOS aur Android dono ke liye apps bana sakte hain
2. **Native Performance**: JavaScript bridge ke through native components use karta hai
3. **Hot Reloading**: Code changes instantly reflect hote hain
4. **Large Community**: Bahut bada developer community aur ecosystem
5. **Reusable Components**: Web React components ko mobile mein reuse kar sakte hain

### React Native vs Other Technologies

| Feature | React Native | Flutter | Native (iOS/Android) | Ionic |
|---------|-------------|---------|---------------------|-------|
| Language | JavaScript | Dart | Swift/Kotlin | HTML/CSS/JS |
| Performance | Near Native | Native | Native | Web-based |
| Code Sharing | 90%+ | 90%+ | 0% | 80%+ |
| Learning Curve | Easy | Medium | Hard | Easy |
| Community | Large | Growing | Separate | Medium |

---

## 🛠️ Environment Setup

### Prerequisites
```bash
# Check if you have these installed
node --version    # Should be 14.x or higher
npm --version     # Should be 6.x or higher
git --version     # For version control
```

### Step 1: Install Node.js
```bash
# Download from https://nodejs.org/
# Or using chocolatey (Windows)
choco install nodejs

# Or using homebrew (Mac)
brew install node
```

### Step 2: Install React Native CLI
```bash
# Global installation
npm install -g react-native-cli

# Or using npx (recommended)
npm install -g @react-native-community/cli
```

### Step 3: Android Development Setup

#### Install Java Development Kit (JDK)
```bash
# Download JDK 11 from Oracle or OpenJDK
# Set JAVA_HOME environment variable
# Windows: C:\Program Files\Java\jdk-11.x.x
# Mac: /Library/Java/JavaVirtualMachines/jdk-11.x.x.jdk/Contents/Home
```

#### Install Android Studio
1. Download from https://developer.android.com/studio
2. Install with default settings
3. Open Android Studio
4. Go to SDK Manager
5. Install required SDK platforms:
   - Android 10 (API Level 29)
   - Android 11 (API Level 30)
   - Android 12 (API Level 31)

#### Set Environment Variables
```bash
# Windows (Add to System Environment Variables)
ANDROID_HOME=C:\Users\YourUsername\AppData\Local\Android\Sdk
Path=%Path%;%ANDROID_HOME%\platform-tools;%ANDROID_HOME%\tools

# Mac/Linux (Add to ~/.bash_profile or ~/.zshrc)
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

### Step 4: iOS Development Setup (Mac Only)

#### Install Xcode
```bash
# Install from Mac App Store
# Or download from Apple Developer Portal

# Install Xcode Command Line Tools
xcode-select --install

# Install CocoaPods
sudo gem install cocoapods
```

#### iOS Simulator Setup
```bash
# Open Xcode
# Go to Xcode > Preferences > Components
# Download required iOS Simulators
```

---

## 🚀 Creating Your First React Native App

### Method 1: Using React Native CLI
```bash
# Create new project
npx react-native init MyFirstApp

# Navigate to project directory
cd MyFirstApp

# Start Metro bundler
npx react-native start

# Run on Android (in another terminal)
npx react-native run-android

# Run on iOS (Mac only)
npx react-native run-ios
```

### Method 2: Using Expo CLI (Recommended for Beginners)
```bash
# Install Expo CLI
npm install -g expo-cli

# Create new project
expo init MyFirstApp

# Choose template (blank, tabs, etc.)
cd MyFirstApp

# Start development server
expo start
```

---

## 📁 Project Structure

```
MyFirstApp/
├── android/                 # Android native code
├── ios/                     # iOS native code
├── node_modules/            # Dependencies
├── src/                     # Source code (create this)
│   ├── components/          # Reusable components
│   ├── screens/            # App screens
│   ├── navigation/         # Navigation setup
│   ├── services/           # API calls, utilities
│   └── assets/             # Images, fonts
├── App.js                  # Main app component
├── index.js                # App entry point
├── package.json            # Dependencies and scripts
├── metro.config.js         # Metro bundler config
└── README.md               # Project documentation
```

---

## 💻 Your First React Native Component

### App.js
```javascript
import React from 'react';
import {
  SafeAreaView,
  ScrollView,
  StatusBar,
  StyleSheet,
  Text,
  View,
  TouchableOpacity,
  Alert,
} from 'react-native';

const App = () => {
  const showAlert = () => {
    Alert.alert(
      'Hello!',
      'Welcome to React Native!',
      [
        {text: 'OK', onPress: () => console.log('OK Pressed')},
      ]
    );
  };

  return (
    <SafeAreaView style={styles.container}>
      <StatusBar barStyle="dark-content" />
      <ScrollView contentInsetAdjustmentBehavior="automatic">
        <View style={styles.body}>
          <Text style={styles.title}>
            Welcome to React Native! 🚀
          </Text>
          <Text style={styles.subtitle}>
            Aapka pehla mobile app ready hai!
          </Text>
          
          <TouchableOpacity style={styles.button} onPress={showAlert}>
            <Text style={styles.buttonText}>Press Me!</Text>
          </TouchableOpacity>
          
          <View style={styles.infoContainer}>
            <Text style={styles.infoTitle}>Next Steps:</Text>
            <Text style={styles.infoText}>
              • Edit App.js to change this screen{'\n'}
              • Add new components{'\n'}
              • Learn about navigation{'\n'}
              • Build amazing apps!
            </Text>
          </View>
        </View>
      </ScrollView>
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f8f9fa',
  },
  body: {
    padding: 20,
    alignItems: 'center',
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    color: '#333',
    textAlign: 'center',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 18,
    color: '#666',
    textAlign: 'center',
    marginBottom: 30,
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 30,
    paddingVertical: 15,
    borderRadius: 25,
    marginBottom: 30,
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  infoContainer: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 10,
    width: '100%',
    shadowColor: '#000',
    shadowOffset: {
      width: 0,
      height: 2,
    },
    shadowOpacity: 0.1,
    shadowRadius: 3.84,
    elevation: 5,
  },
  infoTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 10,
  },
  infoText: {
    fontSize: 16,
    color: '#666',
    lineHeight: 24,
  },
});

export default App;
```

---

## 🔧 Common Setup Issues & Solutions

### Issue 1: Metro bundler not starting
```bash
# Clear cache
npx react-native start --reset-cache

# Or
rm -rf node_modules
npm install
```

### Issue 2: Android emulator not detected
```bash
# List available devices
adb devices

# Start emulator manually
emulator -avd YourAVDName
```

### Issue 3: iOS build fails
```bash
# Clean build folder
cd ios
rm -rf build
cd ..

# Reinstall pods
cd ios
pod install
cd ..
```

### Issue 4: Port already in use
```bash
# Kill process on port 8081
npx react-native start --port 8082

# Or kill the process
lsof -ti:8081 | xargs kill -9
```

---

## 📝 Assignment 1: Setup & First App

### Task 1: Environment Setup (30 minutes)
1. Install Node.js, React Native CLI
2. Setup Android Studio aur emulator
3. Create aur run your first app
4. Screenshot leke submit karo

### Task 2: Modify First App (45 minutes)
1. App.js mein title change karo
2. Ek naya button add karo
3. Button press pe different alert show karo
4. Background color change karo
5. Code aur screenshot submit karo

### Task 3: Explore Project Structure (30 minutes)
1. Har folder ka purpose samjho
2. package.json mein dependencies dekho
3. android/ aur ios/ folders explore karo
4. Summary report banao

---

## 🧠 Quiz 1: React Native Basics

### Multiple Choice Questions

**Q1. React Native kis company ne develop kiya hai?**
a) Google
b) Facebook (Meta)
c) Microsoft
d) Apple

**Q2. React Native mein kaunsi language use hoti hai?**
a) Java
b) Swift
c) JavaScript
d) Kotlin

**Q3. Cross-platform development ka matlab kya hai?**
a) Sirf Android ke liye
b) Sirf iOS ke liye
c) Ek code se multiple platforms
d) Web development

**Q4. Metro bundler ka kya kaam hai?**
a) App ko bundle karna
b) Database manage karna
c) UI design karna
d) Testing karna

**Q5. Hot Reloading ka faida kya hai?**
a) App fast chalti hai
b) Code changes instantly reflect hote hain
c) Memory kam use hoti hai
d) Battery save hoti hai

### True/False Questions

**Q6. React Native sirf mobile apps ke liye use hota hai.** (True/False)

**Q7. Expo CLI beginners ke liye recommended hai.** (True/False)

**Q8. React Native apps native performance dete hain.** (True/False)

**Q9. iOS development ke liye Windows machine sufficient hai.** (True/False)

**Q10. React Native mein web React components reuse kar sakte hain.** (True/False)

### Short Answer Questions

**Q11. React Native aur Flutter mein 3 main differences batao.**

**Q12. Android development setup mein kaunse environment variables set karne padte hain?**

**Q13. SafeAreaView component ka kya use hai?**

**Q14. Metro bundler ko reset kaise karte hain?**

**Q15. React Native project mein src/ folder kyu banate hain?**

---

## 🎯 Quiz Answers

**MCQ Answers:**
1. b) Facebook (Meta)
2. c) JavaScript
3. c) Ek code se multiple platforms
4. a) App ko bundle karna
5. b) Code changes instantly reflect hote hain

**True/False Answers:**
6. False (Desktop apps bhi ban sakte hain)
7. True
8. True
9. False (Mac required for iOS)
10. True

**Short Answers:**
11. Language (JS vs Dart), Performance (Near-native vs Native), Community size
12. ANDROID_HOME, PATH variables
13. Safe area boundaries handle karta hai (notch, status bar)
14. npx react-native start --reset-cache
15. Better code organization aur maintainability

---

## 🔗 Additional Resources

### Documentation
- [React Native Official Docs](https://reactnative.dev/)
- [Expo Documentation](https://docs.expo.dev/)
- [React Navigation](https://reactnavigation.org/)

### Tools
- [Flipper](https://fbflipper.com/) - Debugging tool
- [Reactotron](https://github.com/infinitered/reactotron) - Inspector
- [VS Code Extensions](https://marketplace.visualstudio.com/items?itemName=msjsdiag.vscode-react-native)

### Communities
- [React Native Community](https://github.com/react-native-community)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/react-native)
- [Reddit r/reactnative](https://www.reddit.com/r/reactnative/)

---

## 🎉 Lesson Complete!

Congratulations! Aapne successfully:
- ✅ React Native ke bare mein basic knowledge gain ki
- ✅ Development environment setup kiya
- ✅ First React Native app create kiya
- ✅ Project structure samjha
- ✅ Common issues aur solutions dekhe

**Next Lesson Preview:** JavaScript ES6+ Essentials for React Native - Modern JavaScript features jo React Native development mein zaroori hain.

---

*Happy Coding! 🚀*