# Lesson 14: Native Modules & Platform-Specific Code

## 🎯 **Learning Objectives**
- Understand native module architecture
- Create custom native modules for iOS and Android
- Handle platform-specific code effectively
- Integrate third-party native libraries
- Debug native module issues

## 📚 **Table of Contents**
1. [Introduction to Native Modules](#introduction-to-native-modules)
2. [Creating Native Modules](#creating-native-modules)
3. [Platform-Specific Code](#platform-specific-code)
4. [Native Module Bridge](#native-module-bridge)
5. [Third-Party Library Integration](#third-party-library-integration)
6. [Error Handling & Debugging](#error-handling--debugging)
7. [Best Practices](#best-practices)
8. [Practical Examples](#practical-examples)

---

## 🔧 **Introduction to Native Modules**

Native modules allow you to write custom native code (Java/Kotlin for Android, Objective-C/Swift for iOS) and expose it to JavaScript in your React Native app.

### **When to Use Native Modules**
- **Performance-critical operations**
- **Access to native APIs** not available in JavaScript
- **Hardware integration** (camera, sensors, Bluetooth)
- **Custom UI components** requiring native rendering
- **Existing native code** integration

### **Architecture Overview**
```
JavaScript Layer
        ↓
  Bridge Layer (JSON)
        ↓
Native Module Layer
        ↓
Platform APIs (iOS/Android)
```

---

## 🏗️ **Creating Native Modules**

### **Android Native Module**

#### **Step 1: Create Java/Kotlin Class**
```java
// ToastModule.java
package com.yourapp.modules;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import android.widget.Toast;

public class ToastModule extends ReactContextBaseJavaModule {

    private static ReactApplicationContext reactContext;

    ToastModule(ReactApplicationContext context) {
        super(context);
        reactContext = context;
    }

    @Override
    public String getName() {
        return "ToastModule";
    }

    @ReactMethod
    public void show(String message, int duration) {
        Toast.makeText(getReactApplicationContext(), message, duration).show();
    }
}
```

#### **Step 2: Create Package Class**
```java
// ToastPackage.java
package com.yourapp.modules;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ToastPackage implements ReactPackage {

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }

    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        List<NativeModule> modules = new ArrayList<>();
        modules.add(new ToastModule(reactContext));
        return modules;
    }
}
```

#### **Step 3: Register Package in MainApplication.java**
```java
// MainApplication.java
@Override
protected List<ReactPackage> getPackages() {
    return Arrays.<ReactPackage>asList(
        new MainReactPackage(),
        new ToastPackage() // Add your package here
    );
}
```

### **iOS Native Module**

#### **Step 1: Create Objective-C/Swift Class**
```objc
// ToastModule.m
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(ToastModule, NSObject)

RCT_EXTERN_METHOD(show:(NSString *)message duration:(NSInteger *)duration)

@end
```

```objc
// ToastModule.m (Implementation)
#import "ToastModule.h"
#import <UIKit/UIKit.h>

@implementation ToastModule

RCT_EXPORT_MODULE();

RCT_EXPORT_METHOD(show:(NSString *)message duration:(NSInteger *)duration)
{
  dispatch_async(dispatch_get_main_queue(), ^{
    UIAlertController *alert = [UIAlertController
      alertControllerWithTitle:@"Toast"
      message:message
      preferredStyle:UIAlertControllerStyleAlert];

    UIViewController *rootViewController = [UIApplication sharedApplication].delegate.window.rootViewController;
    [rootViewController presentViewController:alert animated:YES completion:nil];

    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
      [alert dismissViewControllerAnimated:YES completion:nil];
    });
  });
}

@end
```

### **JavaScript Interface**
```javascript
// ToastModule.js
import { NativeModules } from 'react-native';

const { ToastModule } = NativeModules;

export default {
  show: (message, duration = 2000) => {
    ToastModule.show(message, duration);
  },
};
```

---

## 📱 **Platform-Specific Code**

### **Platform-Specific File Extensions**
```
Component.js          // Universal
Component.ios.js      // iOS only
Component.android.js  // Android only
```

### **Platform Module Usage**
```javascript
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    ...Platform.select({
      ios: {
        paddingTop: 20,
      },
      android: {
        paddingTop: 0,
      },
    }),
  },
});

// Platform detection
if (Platform.OS === 'ios') {
  // iOS specific code
} else if (Platform.OS === 'android') {
  // Android specific code
}

// Version checking
if (Platform.Version >= 21) {
  // Android API 21+ specific code
}
```

### **Platform-Specific Components**
```javascript
// CustomButton.js
import React from 'react';
import { TouchableOpacity, TouchableNativeFeedback, Platform, View } from 'react-native';

const CustomButton = ({ onPress, children, style }) => {
  if (Platform.OS === 'android' && Platform.Version >= 21) {
    return (
      <TouchableNativeFeedback
        onPress={onPress}
        background={TouchableNativeFeedback.Ripple('#rgba(0,0,0,0.2)', false)}
      >
        <View style={style}>
          {children}
        </View>
      </TouchableNativeFeedback>
    );
  }

  return (
    <TouchableOpacity onPress={onPress} style={style}>
      {children}
    </TouchableOpacity>
  );
};

export default CustomButton;
```

---

## 🌉 **Native Module Bridge**

### **Passing Data Types**
```javascript
// JavaScript to Native
@ReactMethod
public void processData(ReadableMap data) {
    String name = data.getString("name");
    int age = data.getInt("age");
    boolean isActive = data.getBoolean("isActive");
}

// Native to JavaScript
@ReactMethod
public void getUserData(Promise promise) {
    try {
        WritableMap userData = Arguments.createMap();
        userData.putString("name", "John Doe");
        userData.putInt("age", 30);
        promise.resolve(userData);
    } catch (Exception e) {
        promise.reject("ERROR", e.getMessage());
    }
}
```

### **Callback Implementation**
```javascript
// Native Module
@ReactMethod
public void performAsyncTask(Callback successCallback, Callback errorCallback) {
    new Thread(() -> {
        try {
            // Perform async operation
            String result = "Task completed";
            successCallback.invoke(result);
        } catch (Exception e) {
            errorCallback.invoke(e.getMessage());
        }
    }).start();
}

// JavaScript Usage
import { NativeModules } from 'react-native';

const { MyNativeModule } = NativeModules;

MyNativeModule.performAsyncTask(
  (result) => {
    console.log('Success:', result);
  },
  (error) => {
    console.log('Error:', error);
  }
);
```

### **Event Emission**
```javascript
// Native Module
private void sendEvent(String eventName, WritableMap params) {
    getReactApplicationContext()
        .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
        .emit(eventName, params);
}

@ReactMethod
public void startSensorUpdates() {
    // Start sensor updates and emit events
    WritableMap sensorData = Arguments.createMap();
    sensorData.putDouble("x", accelerometerData.x);
    sensorData.putDouble("y", accelerometerData.y);
    sensorData.putDouble("z", accelerometerData.z);
    sendEvent("SensorData", sensorData);
}

// JavaScript Listener
import { DeviceEventEmitter } from 'react-native';

useEffect(() => {
  const subscription = DeviceEventEmitter.addListener(
    'SensorData',
    (data) => {
      console.log('Sensor data:', data);
    }
  );

  return () => subscription.remove();
}, []);
```

---

## 📚 **Third-Party Library Integration**

### **Android Library Integration**

#### **Step 1: Add to build.gradle**
```gradle
dependencies {
    implementation 'com.squareup.okhttp3:okhttp:4.9.3'
    implementation 'com.google.code.gson:gson:2.8.8'
}
```

#### **Step 2: Create Native Module**
```java
public class NetworkModule extends ReactContextBaseJavaModule {

    public NetworkModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }

    @Override
    public String getName() {
        return "NetworkModule";
    }

    @ReactMethod
    public void makeRequest(String url, Promise promise) {
        OkHttpClient client = new OkHttpClient();

        Request request = new Request.Builder()
            .url(url)
            .build();

        client.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                promise.reject("NETWORK_ERROR", e.getMessage());
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                if (response.isSuccessful()) {
                    promise.resolve(response.body().string());
                } else {
                    promise.reject("HTTP_ERROR", "Request failed with code: " + response.code());
                }
            }
        });
    }
}
```

### **iOS Library Integration**

#### **Step 1: Add to Podfile**
```ruby
pod 'AFNetworking', '~> 4.0'
```

#### **Step 2: Create Native Module**
```objc
#import <AFNetworking/AFNetworking.h>

@interface RCT_EXTERN_MODULE(NetworkModule, NSObject)

RCT_EXTERN_METHOD(makeRequest:(NSString *)url resolver:(RCTPromiseResolveBlock)resolve rejecter:(RCTPromiseRejectBlock)reject)

@end

@implementation NetworkModule

RCT_EXPORT_MODULE();

RCT_EXPORT_METHOD(makeRequest:(NSString *)url resolver:(RCTPromiseResolveBlock)resolve rejecter:(RCTPromiseRejectBlock)reject)
{
  AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

  [manager GET:url parameters:nil headers:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
    resolve(responseObject);
  } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
    reject(@"NETWORK_ERROR", error.localizedDescription, error);
  }];
}

@end
```

---

## 🐛 **Error Handling & Debugging**

### **Native Module Error Handling**
```javascript
// JavaScript error handling
const callNativeModule = async () => {
  try {
    const result = await NativeModules.MyModule.performAction();
    console.log('Success:', result);
  } catch (error) {
    console.error('Native module error:', error);
    // Handle error appropriately
  }
};
```

### **Debugging Native Code**

#### **Android Debugging**
```bash
# Log native module calls
adb logcat | grep ReactNative

# Debug native code in Android Studio
# 1. Open Android project in Android Studio
# 2. Set breakpoints in native code
# 3. Run app in debug mode
```

#### **iOS Debugging**
```bash
# View device logs
xcrun simctl spawn booted log stream --level debug

# Debug in Xcode
# 1. Open iOS project in Xcode
# 2. Set breakpoints in native code
# 3. Run app in debug mode
```

### **Common Issues & Solutions**

#### **Module Not Found**
```javascript
// Check if module is properly registered
console.log(NativeModules); // Should list your module

// Ensure proper import
import { NativeModules } from 'react-native';
const { MyModule } = NativeModules;
```

#### **Bridge Communication Issues**
```javascript
// Add error boundaries
class ErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    console.error('Native module error:', error, errorInfo);
  }

  render() {
    return this.props.children;
  }
}
```

---

## ✅ **Best Practices**

### **Performance Considerations**
1. **Minimize bridge communication** - Batch operations when possible
2. **Use appropriate thread** - UI operations on main thread, heavy work on background
3. **Memory management** - Clean up resources properly
4. **Error handling** - Always handle errors gracefully

### **Code Organization**
```javascript
// Directory structure
android/
  src/main/java/com/yourapp/
    modules/
      ToastModule.java
      NetworkModule.java
    packages/
      CustomPackage.java

ios/
  modules/
    ToastModule.m
    ToastModule.h
    NetworkModule.m

src/
  native/
    Toast.js
    Network.js
    index.js
```

### **Security Considerations**
```java
// Validate input parameters
@ReactMethod
public void saveData(String data, Promise promise) {
    if (data == null || data.isEmpty()) {
        promise.reject("INVALID_INPUT", "Data cannot be empty");
        return;
    }
    // Process data safely
}
```

---

## 🎯 **Practical Examples**

### **Custom Camera Module**
```javascript
// CameraModule.js
import { NativeModules, Platform } from 'react-native';

const { CameraModule } = NativeModules;

export default {
  takePhoto: () => {
    return CameraModule.takePhoto();
  },

  startRecording: () => {
    return CameraModule.startRecording();
  },

  stopRecording: () => {
    return CameraModule.stopRecording();
  },

  getPhotoLibrary: () => {
    return CameraModule.getPhotoLibrary();
  },
};
```

### **Device Information Module**
```javascript
// DeviceModule.js
import { NativeModules, Platform } from 'react-native';

const { DeviceModule } = NativeModules;

export default {
  getDeviceInfo: async () => {
    try {
      const info = await DeviceModule.getDeviceInfo();
      return {
        ...info,
        platform: Platform.OS,
        version: Platform.Version,
      };
    } catch (error) {
      console.error('Failed to get device info:', error);
      return null;
    }
  },

  getBatteryLevel: () => {
    return DeviceModule.getBatteryLevel();
  },

  getStorageInfo: () => {
    return DeviceModule.getStorageInfo();
  },
};
```

### **Biometric Authentication Module**
```javascript
// BiometricModule.js
import { NativeModules, Platform } from 'react-native';

const { BiometricModule } = NativeModules;

export default {
  isBiometricAvailable: async () => {
    try {
      return await BiometricModule.isBiometricAvailable();
    } catch (error) {
      return false;
    }
  },

  authenticate: (reason = 'Authenticate to continue') => {
    return BiometricModule.authenticate(reason);
  },

  getBiometricType: () => {
    return BiometricModule.getBiometricType();
  },
};
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Native Module Creation**: Android (Java/Kotlin) and iOS (Objective-C/Swift)
- ✅ **Platform-Specific Code**: File extensions and Platform module usage
- ✅ **Bridge Communication**: Data types, callbacks, promises, and events
- ✅ **Third-Party Integration**: Adding native libraries to projects
- ✅ **Error Handling**: Debugging native code and handling errors
- ✅ **Best Practices**: Performance, security, and code organization

### **Best Practices Recap**
1. **Use native modules sparingly** - Only when JavaScript isn't sufficient
2. **Always handle errors** - Both in native code and JavaScript
3. **Test on real devices** - Emulators may behave differently
4. **Document your modules** - Clear API documentation
5. **Version compatibility** - Handle different OS versions properly

### **Next Steps**
- Create your own native module
- Integrate a third-party native library
- Implement platform-specific features
- Debug and optimize native module performance

---

## 🎯 **Assignment**

### **Task 1: Create a Device Info Module**
Build a native module that provides:
- Device model and manufacturer
- OS version and API level
- Screen dimensions and density
- Available storage space
- Battery level and charging status

### **Task 2: Custom Notification Module**
Create a native module for:
- Scheduling local notifications
- Custom notification sounds
- Notification actions and categories
- Handling notification taps

### **Task 3: File System Module**
Implement a native module that:
- Reads and writes files
- Creates directories
- Lists directory contents
- Handles file permissions
- Supports different file formats

---

## 📚 **Additional Resources**
- [React Native Native Modules Documentation](https://reactnative.dev/docs/native-modules-intro)
- [Android Native Development](https://developer.android.com/guide)
- [iOS Native Development](https://developer.apple.com/documentation/)
- [React Native Community Libraries](https://github.com/react-native-community)
- [Native Module Examples](https://github.com/facebook/react-native/tree/main/packages/rn-tester)

---

**Next Lesson**: [Lesson 15: Camera & Media Integration](Lesson%2015_%20Camera%20&%20Media%20Integration.md)