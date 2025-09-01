# ⚡ Expo Setup Guide for React Native
## Complete Installation & Development Manual

---

## 🎯 **Overview**
Expo is a framework and platform for universal React applications. It provides a set of tools and services built around React Native and web standards. This guide covers complete Expo setup, development workflow, and deployment.

---

## 📋 **Table of Contents**
1. [What is Expo?](#what-is-expo)
2. [System Requirements](#system-requirements)
3. [Installation & Setup](#installation--setup)
4. [Expo CLI Commands](#expo-cli-commands)
5. [Development Workflow](#development-workflow)
6. [Expo SDK & Libraries](#expo-sdk--libraries)
7. [Building & Deployment](#building--deployment)
8. [Troubleshooting](#troubleshooting)
9. [Best Practices](#best-practices)

---

## 🤔 **What is Expo?**

### **Expo Framework Features:**
- **Managed Workflow**: No need to configure native projects
- **Expo SDK**: Pre-built native modules and APIs
- **Expo CLI**: Command-line tools for development
- **Expo Go**: Development client app
- **EAS Build**: Cloud build service
- **OTA Updates**: Over-the-air app updates

### **When to Use Expo:**
```bash
✅ Prototyping and MVPs
✅ Apps without complex native modules
✅ Teams with limited native development experience
✅ Quick development and iteration
✅ Cross-platform apps with similar UI
```

### **When NOT to Use Expo:**
```bash
❌ Apps requiring custom native modules
❌ Apps with complex native integrations
❌ Enterprise apps with specific security requirements
❌ Apps needing bare workflow control
❌ Large teams with extensive native expertise
```

---

## 💻 **System Requirements**

### **Minimum Requirements:**
- **Node.js**: 16.0.0 or later
- **npm**: 7.0.0 or later (comes with Node.js)
- **Git**: Latest version
- **OS**: Windows 10+, macOS 10.15+, Ubuntu 18.04+

### **Recommended Requirements:**
- **Node.js**: 18.x LTS
- **npm**: 8.x or later
- **Yarn**: 1.22+ (optional, but recommended)
- **Watchman**: For better file watching (macOS/Linux)

### **Mobile Requirements:**
- **iOS**: iOS 11.0+
- **Android**: Android 5.0+ (API 21+)
- **Expo Go App**: Latest version from App Store/Play Store

---

## 📥 **Installation & Setup**

### **Step 1: Install Node.js**
```bash
# Download from official website
https://nodejs.org/

# Or use package manager

# Windows (Chocolatey)
choco install nodejs

# macOS (Homebrew)
brew install node

# Linux (Ubuntu/Debian)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version
npm --version
```

### **Step 2: Install Expo CLI**
```bash
# Install Expo CLI globally
npm install -g @expo/cli

# Or using npx (recommended for one-time use)
npx @expo/cli --version

# Verify installation
expo --version
```

### **Step 3: Install Expo Go App**
```bash
# iOS: Download from App Store
# Search for "Expo Go"

# Android: Download from Google Play Store
# Search for "Expo Go"

# Or scan QR codes during development
```

### **Step 4: Create Expo Account (Optional but Recommended)**
```bash
# Create account at
https://expo.dev/

# Or via CLI
expo login

# This enables:
# - EAS Build service
# - Push notifications
# - App store deployments
# - Analytics
```

---

## 🚀 **Expo CLI Commands**

### **Project Creation**
```bash
# Create new Expo project
npx create-expo-app MyApp

# Create with TypeScript
npx create-expo-app MyApp --template typescript

# Create blank project
npx create-expo-app MyApp --template blank

# Create with tabs navigation
npx create-expo-app MyApp --template tabs

# Navigate to project
cd MyApp
```

### **Development Commands**
```bash
# Start development server
npx expo start

# Start with specific platform
npx expo start --ios
npx expo start --android
npx expo start --web

# Clear cache and start
npx expo start --clear

# Start with tunnel (for physical devices)
npx expo start --tunnel

# Start with specific host
npx expo start --host localhost
```

### **Build Commands**
```bash
# Build for development
npx expo run:ios
npx expo run:android

# Build with specific configuration
npx expo run:ios --configuration Release
npx expo run:android --configuration Debug

# Install dependencies
npx expo install
```

### **EAS Build Commands**
```bash
# Install EAS CLI
npm install -g @expo/eas-cli

# Login to EAS
eas login

# Build for production
eas build --platform ios
eas build --platform android
eas build --platform all

# Submit to stores
eas submit --platform ios
eas submit --platform android
```

### **Utility Commands**
```bash
# Check Expo configuration
npx expo config

# Install specific Expo SDK version
npx expo install --fix

# Upgrade Expo SDK
npx expo upgrade

# Check for updates
npx expo install --check

# View project info
npx expo doctor
```

---

## 💻 **Development Workflow**

### **Project Structure**
```bash
MyExpoApp/
├── app/
│   ├── _layout.tsx          # Root layout
│   ├── index.tsx            # Home screen
│   └── (tabs)/              # Tab navigation
│       ├── _layout.tsx
│       ├── index.tsx
│       └── explore.tsx
├── components/              # Reusable components
├── constants/               # App constants
├── assets/                  # Images, fonts, etc.
├── app.json                 # Expo configuration
├── package.json
├── metro.config.js          # Metro bundler config
└── tsconfig.json           # TypeScript config
```

### **Basic App Structure**
```typescript
// app/index.tsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

export default function HomeScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Expo!</Text>
      <Text style={styles.subtitle}>Your first Expo app</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
  },
});
```

### **Navigation Setup**
```typescript
// app/_layout.tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Home' }} />
      <Stack.Screen name="profile" options={{ title: 'Profile' }} />
    </Stack>
  );
}
```

### **State Management**
```typescript
// Simple state management with Context
import React, { createContext, useContext, useState } from 'react';

interface AppContextType {
  user: any;
  setUser: (user: any) => void;
}

const AppContext = createContext<AppContextType | undefined>(undefined);

export function AppProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState(null);

  return (
    <AppContext.Provider value={{ user, setUser }}>
      {children}
    </AppContext.Provider>
  );
}

export function useApp() {
  const context = useContext(AppContext);
  if (context === undefined) {
    throw new Error('useApp must be used within AppProvider');
  }
  return context;
}
```

---

## 📚 **Expo SDK & Libraries**

### **Core Expo SDK Libraries**

#### **expo-av (Audio/Video)**
```bash
npx expo install expo-av

# Usage
import { Audio } from 'expo-av';

const playSound = async () => {
  const { sound } = await Audio.Sound.createAsync(
    { uri: 'https://example.com/sound.mp3' }
  );
  await sound.playAsync();
};
```

#### **expo-camera**
```bash
npx expo install expo-camera

# Usage
import { Camera, CameraType } from 'expo-camera';

function CameraScreen() {
  const [permission, requestPermission] = Camera.useCameraPermissions();

  if (!permission?.granted) {
    return <Text>No access to camera</Text>;
  }

  return (
    <Camera style={{ flex: 1 }} type={CameraType.back} />
  );
}
```

#### **expo-location**
```bash
npx expo install expo-location

# Usage
import * as Location from 'expo-location';

const getLocation = async () => {
  const { status } = await Location.requestForegroundPermissionsAsync();
  if (status !== 'granted') {
    return;
  }

  const location = await Location.getCurrentPositionAsync({});
  console.log(location);
};
```

#### **expo-notifications**
```bash
npx expo install expo-notifications

# Usage
import * as Notifications from 'expo-notifications';

const scheduleNotification = async () => {
  await Notifications.scheduleNotificationAsync({
    content: {
      title: 'Reminder',
      body: 'Time for your daily task!',
    },
    trigger: { seconds: 60 },
  });
};
```

#### **expo-secure-store**
```bash
npx expo install expo-secure-store

# Usage
import * as SecureStore from 'expo-secure-store';

const saveToken = async (token: string) => {
  await SecureStore.setItemAsync('userToken', token);
};

const getToken = async () => {
  return await SecureStore.getItemAsync('userToken');
};
```

#### **expo-file-system**
```bash
npx expo install expo-file-system

# Usage
import * as FileSystem from 'expo-file-system';

const downloadFile = async (url: string, filename: string) => {
  const downloadResumable = FileSystem.createDownloadResumable(
    url,
    FileSystem.documentDirectory + filename
  );

  const result = await downloadResumable.downloadAsync();
  console.log('Downloaded to:', result?.uri);
};
```

### **UI Libraries**

#### **expo-router (Navigation)**
```bash
npx expo install expo-router

# File-based routing
app/
├── _layout.tsx
├── index.tsx
├── profile.tsx
└── settings.tsx
```

#### **expo-font**
```bash
npx expo install expo-font

# Usage
import { useFonts } from 'expo-font';

function App() {
  const [fontsLoaded] = useFonts({
    'Inter-Bold': require('./assets/fonts/Inter-Bold.ttf'),
  });

  if (!fontsLoaded) {
    return <Text>Loading...</Text>;
  }

  return <Text style={{ fontFamily: 'Inter-Bold' }}>Hello World</Text>;
}
```

#### **expo-splash-screen**
```bash
npx expo install expo-splash-screen

# Usage
import * as SplashScreen from 'expo-splash-screen';

SplashScreen.preventAutoHideAsync();

setTimeout(() => {
  SplashScreen.hideAsync();
}, 3000);
```

---

## 🏗️ **Building & Deployment**

### **Development Builds**
```bash
# Run on iOS simulator
npx expo run:ios

# Run on Android emulator
npx expo run:android

# Run on physical device
# Scan QR code from Expo Go app
npx expo start
```

### **Production Builds with EAS**

#### **Step 1: Configure EAS**
```json
// eas.json
{
  "cli": {
    "version": ">= 0.52.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal"
    },
    "production": {}
  },
  "submit": {
    "production": {}
  }
}
```

#### **Step 2: Build for Stores**
```bash
# Build for iOS
eas build --platform ios --profile production

# Build for Android
eas build --platform android --profile production

# Build for both platforms
eas build --platform all --profile production
```

#### **Step 3: Submit to App Stores**
```bash
# Submit to Apple App Store
eas submit --platform ios --profile production

# Submit to Google Play Store
eas submit --platform android --profile production
```

### **OTA Updates**
```bash
# Install expo-updates
npx expo install expo-updates

# Configure updates
// app.json
{
  "expo": {
    "updates": {
      "url": "https://u.expo.dev/your-project-id"
    }
  }
}

# Publish update
npx expo publish
```

---

## 🔧 **Troubleshooting**

### **Common Issues & Solutions**

#### **Issue 1: Metro Bundler Issues**
```bash
# Clear Metro cache
npx expo start --clear

# Reset Metro bundler
npx expo r -c

# Check Metro configuration
# metro.config.js
module.exports = {
  resolver: {
    alias: {
      '@': './src',
    },
  },
};
```

#### **Issue 2: Expo Go Connection Issues**
```bash
# Use tunnel mode
npx expo start --tunnel

# Check firewall settings
# Allow ports 19000-19006

# Use local network
npx expo start --host lan

# Check device and computer on same network
```

#### **Issue 3: Build Failures**
```bash
# Check Expo CLI version
expo --version

# Update Expo CLI
npm install -g @expo/cli@latest

# Clear npm cache
npm cache clean --force

# Reinstall dependencies
rm -rf node_modules package-lock.json
npm install
```

#### **Issue 4: SDK Version Mismatch**
```bash
# Check current SDK version
npx expo config

# Upgrade to latest SDK
npx expo upgrade

# Install specific SDK version
npx expo upgrade --to 48.0.0
```

#### **Issue 5: Permission Issues**
```bash
# iOS permissions
npx expo install expo-media-library
npx expo install expo-contacts

# Android permissions in app.json
{
  "expo": {
    "plugins": [
      [
        "expo-media-library",
        {
          "photosPermission": "Allow $(PRODUCT_NAME) to access your photos.",
          "savePhotosPermission": "Allow $(PRODUCT_NAME) to save photos."
        }
      ]
    ]
  }
}
```

### **Performance Issues**
```bash
# Enable Hermes engine
// app.json
{
  "expo": {
    "jsEngine": "hermes"
  }
}

# Optimize bundle size
// metro.config.js
module.exports = {
  transformer: {
    minifierConfig: {
      compress: {
        drop_console: true,
      },
    },
  },
};
```

---

## 📚 **Best Practices**

### **Project Organization**
```bash
# Recommended structure
MyExpoApp/
├── app/                    # File-based routing
│   ├── _layout.tsx        # Root layout
│   ├── (tabs)/           # Tab navigation
│   └── modal/            # Modal screens
├── components/           # Reusable components
│   ├── ui/              # UI components
│   ├── forms/           # Form components
│   └── layouts/         # Layout components
├── hooks/               # Custom hooks
├── utils/               # Utility functions
├── constants/           # App constants
├── assets/              # Images, fonts, icons
├── services/            # API services
└── types/               # TypeScript types
```

### **Code Quality**
```typescript
// Use TypeScript
interface User {
  id: string;
  name: string;
  email: string;
}

interface Props {
  user: User;
  onUpdate: (user: User) => void;
}

// Custom hooks for reusable logic
function useAuth() {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    // Login logic
  };

  const logout = () => {
    setUser(null);
  };

  return { user, login, logout };
}
```

### **Performance Optimization**
```typescript
// Lazy loading
const LazyProfile = lazy(() => import('./screens/Profile'));

// Image optimization
import { Image } from 'expo-image';

<Image
  source={{ uri: 'https://example.com/image.jpg' }}
  placeholder={require('./placeholder.png')}
  contentFit="cover"
  transition={1000}
/>

// Memoization
const MemoizedComponent = React.memo(({ data }) => {
  return <Text>{data.title}</Text>;
});
```

### **Security Best Practices**
```typescript
// Secure storage
import * as SecureStore from 'expo-secure-store';

const saveSensitiveData = async (key: string, value: string) => {
  await SecureStore.setItemAsync(key, value, {
    keychainAccessible: SecureStore.AFTER_FIRST_UNLOCK,
  });
};

// Environment variables
// .env
API_URL=https://api.example.com
SECRET_KEY=your-secret-key

// app.json
{
  "expo": {
    "extra": {
      "apiUrl": process.env.API_URL,
      "secretKey": process.env.SECRET_KEY
    }
  }
}
```

---

## 🚀 **Advanced Features**

### **Custom Development Client**
```bash
# Create custom dev client
npx expo run:ios --device
npx expo run:android --device

# Build development client
eas build --platform ios --profile development
eas build --platform android --profile development
```

### **Expo Application Services (EAS)**

#### **EAS Build**
```json
// eas.json
{
  "build": {
    "production": {
      "ios": {
        "bundleIdentifier": "com.example.myapp"
      },
      "android": {
        "package": "com.example.myapp"
      }
    }
  }
}
```

#### **EAS Submit**
```json
// eas.json
{
  "submit": {
    "production": {
      "ios": {
        "appleId": "your-apple-id@example.com",
        "ascAppId": "1234567890"
      },
      "android": {
        "serviceAccountKeyPath": "./google-service-account-key.json",
        "track": "internal"
      }
    }
  }
}
```

### **Expo Modules API**
```typescript
// Create custom native module
import { registerRootComponent } from 'expo';
import { NativeModules } from 'react-native';

const { MyCustomModule } = NativeModules;

// Use in component
function MyComponent() {
  const handlePress = () => {
    MyCustomModule.doSomething();
  };

  return (
    <TouchableOpacity onPress={handlePress}>
      <Text>Call Native Module</Text>
    </TouchableOpacity>
  );
}
```

---

## 📊 **Monitoring & Analytics**

### **Expo Analytics**
```bash
# View analytics in Expo dashboard
https://expo.dev/accounts/[username]/projects/[project-name]/analytics

# Track custom events
import { Analytics } from 'expo-analytics';

Analytics.track('button_pressed', {
  screen: 'home',
  button: 'signup'
});
```

### **Performance Monitoring**
```typescript
// Use expo-dev-client for performance monitoring
npx expo install expo-dev-client

// Add performance monitoring
import { PerformanceMonitor } from 'expo-dev-client';

function App() {
  return (
    <PerformanceMonitor>
      {/* Your app content */}
    </PerformanceMonitor>
  );
}
```

---

## 🎯 **Conclusion**

Expo provides a streamlined development experience for React Native applications. It's perfect for rapid prototyping, MVPs, and applications that don't require complex native integrations.

### **Key Takeaways:**
- ✅ Use Expo for quick development and prototyping
- ✅ Leverage Expo SDK for common native features
- ✅ Use EAS Build for production builds
- ✅ Implement proper navigation with expo-router
- ✅ Follow React Native and Expo best practices
- ✅ Use TypeScript for better development experience

### **Next Steps:**
1. **Start Small**: Create your first Expo app
2. **Explore SDK**: Learn available Expo modules
3. **Build & Deploy**: Use EAS for production builds
4. **Scale Up**: Consider ejecting to bare workflow if needed
5. **Contribute**: Help improve the Expo ecosystem

### **Additional Resources:**
- [Expo Documentation](https://docs.expo.dev/)
- [Expo Router](https://docs.expo.dev/routing/introduction/)
- [Expo SDK API](https://docs.expo.dev/versions/latest/)
- [EAS Build](https://docs.expo.dev/build/introduction/)
- [Expo Community](https://expo.dev/community)

---

*Happy Expo Development! ⚡📱*