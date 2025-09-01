# Lesson 36: Advanced Navigation Patterns

## 🎯 **Learning Objectives**
- Master advanced navigation patterns in React Native
- Implement complex navigation flows with nested navigators
- Create custom navigation components and transitions
- Handle deep linking and universal links
- Implement authentication flows and protected routes

## 📚 **Table of Contents**
1. [Advanced Navigation Structure](#advanced-navigation-structure)
2. [Custom Navigators](#custom-navigators)
3. [Navigation Hooks](#navigation-hooks)
4. [Deep Linking](#deep-linking)
5. [Authentication Flow](#authentication-flow)
6. [Modal Navigation](#modal-navigation)
7. [Custom Transitions](#custom-transitions)
8. [Navigation State Management](#navigation-state-management)
9. [Performance Optimization](#performance-optimization)
10. [Practical Examples](#practical-examples)

---

## 🏗️ **Advanced Navigation Structure**

### **Nested Navigators**
```javascript
// navigation/AppNavigator.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createDrawerNavigator } from '@react-navigation/drawer';

// Screens
import HomeScreen from '../screens/HomeScreen';
import ProfileScreen from '../screens/ProfileScreen';
import SettingsScreen from '../screens/SettingsScreen';
import LoginScreen from '../screens/LoginScreen';

// Create navigators
const Stack = createStackNavigator();
const Tab = createBottomTabNavigator();
const Drawer = createDrawerNavigator();

// Main Tab Navigator
function MainTabNavigator() {
  return (
    <Tab.Navigator
      screenOptions={{
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: '#8E8E93',
        tabBarStyle: {
          backgroundColor: '#FFFFFF',
          borderTopColor: '#E5E5EA',
        },
        headerStyle: {
          backgroundColor: '#007AFF',
        },
        headerTintColor: '#FFFFFF',
        headerTitleStyle: {
          fontWeight: 'bold',
        },
      }}
    >
      <Tab.Screen
        name="Home"
        component={HomeScreen}
        options={{
          tabBarLabel: 'Home',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home" color={color} size={size} />
          ),
        }}
      />
      <Tab.Screen
        name="Profile"
        component={ProfileStackNavigator}
        options={{
          tabBarLabel: 'Profile',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="person" color={color} size={size} />
          ),
        }}
      />
    </Tab.Navigator>
  );
}

// Profile Stack Navigator (nested)
function ProfileStackNavigator() {
  return (
    <Stack.Navigator
      screenOptions={{
        headerStyle: {
          backgroundColor: '#007AFF',
        },
        headerTintColor: '#FFFFFF',
        headerTitleStyle: {
          fontWeight: 'bold',
        },
      }}
    >
      <Stack.Screen
        name="ProfileMain"
        component={ProfileScreen}
        options={{ title: 'Profile' }}
      />
      <Stack.Screen
        name="Settings"
        component={SettingsScreen}
        options={{ title: 'Settings' }}
      />
    </Stack.Navigator>
  );
}

// Authentication Stack
function AuthStackNavigator() {
  return (
    <Stack.Navigator
      screenOptions={{
        headerShown: false,
      }}
    >
      <Stack.Screen name="Login" component={LoginScreen} />
    </Stack.Navigator>
  );
}

// Main App Navigator with Authentication
function AppNavigator() {
  const { isAuthenticated } = useAuth();

  return (
    <NavigationContainer>
      {isAuthenticated ? (
        <Drawer.Navigator
          drawerContent={(props) => <CustomDrawerContent {...props} />}
          screenOptions={{
            headerStyle: {
              backgroundColor: '#007AFF',
            },
            headerTintColor: '#FFFFFF',
          }}
        >
          <Drawer.Screen
            name="MainTabs"
            component={MainTabNavigator}
            options={{ title: 'Home' }}
          />
          <Drawer.Screen
            name="Settings"
            component={SettingsScreen}
            options={{ title: 'Settings' }}
          />
        </Drawer.Navigator>
      ) : (
        <AuthStackNavigator />
      )}
    </NavigationContainer>
  );
}

export default AppNavigator;
```

### **Dynamic Navigation Structure**
```javascript
// navigation/DynamicNavigator.js
import React, { useEffect, useState } from 'react';
import { createStackNavigator } from '@react-navigation/stack';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Dynamic screen loading
const DynamicStackNavigator = () => {
  const [screens, setScreens] = useState([]);
  const Stack = createStackNavigator();

  useEffect(() => {
    loadDynamicScreens();
  }, []);

  const loadDynamicScreens = async () => {
    try {
      // Load screen configuration from storage or API
      const screenConfig = await AsyncStorage.getItem('screenConfig');
      const parsedConfig = screenConfig ? JSON.parse(screenConfig) : getDefaultScreens();

      setScreens(parsedConfig);
    } catch (error) {
      console.error('Error loading screens:', error);
      setScreens(getDefaultScreens());
    }
  };

  const getDefaultScreens = () => [
    { name: 'Home', component: HomeScreen, title: 'Home' },
    { name: 'Profile', component: ProfileScreen, title: 'Profile' },
  ];

  return (
    <Stack.Navigator>
      {screens.map((screen) => (
        <Stack.Screen
          key={screen.name}
          name={screen.name}
          component={screen.component}
          options={{
            title: screen.title,
            headerStyle: {
              backgroundColor: screen.headerColor || '#007AFF',
            },
          }}
        />
      ))}
    </Stack.Navigator>
  );
};
```

---

## 🛠️ **Custom Navigators**

### **Custom Tab Navigator**
```javascript
// components/CustomTabNavigator.js
import React, { useState } from 'react';
import { View, TouchableOpacity, Text, StyleSheet, Animated } from 'react-native';
import { useNavigation, useRoute } from '@react-navigation/native';

const CustomTabNavigator = ({ tabs, children }) => {
  const navigation = useNavigation();
  const route = useRoute();
  const [activeTab, setActiveTab] = useState(0);
  const [slideAnim] = useState(new Animated.Value(0));

  const handleTabPress = (index, tab) => {
    setActiveTab(index);

    // Animate tab indicator
    Animated.spring(slideAnim, {
      toValue: index,
      useNativeDriver: true,
    }).start();

    // Navigate to tab
    navigation.navigate(tab.name);
  };

  const tabWidth = 100 / tabs.length; // Percentage width per tab

  return (
    <View style={styles.container}>
      {/* Content Area */}
      <View style={styles.content}>
        {children}
      </View>

      {/* Custom Tab Bar */}
      <View style={styles.tabBar}>
        {/* Animated Indicator */}
        <Animated.View
          style={[
            styles.indicator,
            {
              width: `${tabWidth}%`,
              transform: [{
                translateX: slideAnim.interpolate({
                  inputRange: [0, tabs.length - 1],
                  outputRange: [0, (tabs.length - 1) * (100 / tabs.length)],
                }),
              }],
            },
          ]}
        />

        {/* Tab Buttons */}
        {tabs.map((tab, index) => (
          <TouchableOpacity
            key={tab.name}
            style={styles.tabButton}
            onPress={() => handleTabPress(index, tab)}
          >
            <View style={styles.tabContent}>
              <Text style={[
                styles.tabIcon,
                activeTab === index && styles.activeTabIcon
              ]}>
                {tab.icon}
              </Text>
              <Text style={[
                styles.tabLabel,
                activeTab === index && styles.activeTabLabel
              ]}>
                {tab.label}
              </Text>
            </View>
          </TouchableOpacity>
        ))}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  content: {
    flex: 1,
  },
  tabBar: {
    height: 80,
    backgroundColor: '#FFFFFF',
    borderTopWidth: 1,
    borderTopColor: '#E5E5EA',
    flexDirection: 'row',
    position: 'relative',
  },
  indicator: {
    position: 'absolute',
    height: 3,
    backgroundColor: '#007AFF',
    bottom: 0,
  },
  tabButton: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  tabContent: {
    alignItems: 'center',
  },
  tabIcon: {
    fontSize: 24,
    color: '#8E8E93',
    marginBottom: 4,
  },
  activeTabIcon: {
    color: '#007AFF',
  },
  tabLabel: {
    fontSize: 12,
    color: '#8E8E93',
    fontWeight: '500',
  },
  activeTabLabel: {
    color: '#007AFF',
  },
});

export default CustomTabNavigator;
```

### **Wizard Navigator**
```javascript
// components/WizardNavigator.js
import React, { useState, useEffect } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

const WizardNavigator = ({
  steps,
  onComplete,
  onCancel,
  initialStep = 0
}) => {
  const [currentStep, setCurrentStep] = useState(initialStep);
  const [stepData, setStepData] = useState({});

  const currentStepConfig = steps[currentStep];
  const CurrentComponent = currentStepConfig.component;

  const handleNext = (data) => {
    // Save step data
    setStepData(prev => ({
      ...prev,
      [currentStep]: data,
    }));

    if (currentStep < steps.length - 1) {
      setCurrentStep(currentStep + 1);
    } else {
      // Wizard complete
      onComplete(stepData);
    }
  };

  const handlePrevious = () => {
    if (currentStep > 0) {
      setCurrentStep(currentStep - 1);
    }
  };

  const handleCancel = () => {
    onCancel(stepData);
  };

  const progress = ((currentStep + 1) / steps.length) * 100;

  return (
    <View style={styles.container}>
      {/* Progress Bar */}
      <View style={styles.progressContainer}>
        <View style={[styles.progressBar, { width: `${progress}%` }]} />
        <Text style={styles.progressText}>
          Step {currentStep + 1} of {steps.length}
        </Text>
      </View>

      {/* Step Indicator */}
      <View style={styles.stepIndicator}>
        {steps.map((step, index) => (
          <View
            key={index}
            style={[
              styles.stepDot,
              index <= currentStep && styles.activeStepDot,
            ]}
          >
            <Text style={[
              styles.stepNumber,
              index <= currentStep && styles.activeStepNumber,
            ]}>
              {index + 1}
            </Text>
          </View>
        ))}
      </View>

      {/* Step Content */}
      <View style={styles.content}>
        <Text style={styles.stepTitle}>{currentStepConfig.title}</Text>
        <Text style={styles.stepDescription}>
          {currentStepConfig.description}
        </Text>

        <CurrentComponent
          onNext={handleNext}
          onPrevious={handlePrevious}
          data={stepData[currentStep]}
          isFirstStep={currentStep === 0}
          isLastStep={currentStep === steps.length - 1}
        />
      </View>

      {/* Navigation Buttons */}
      <View style={styles.buttonContainer}>
        <TouchableOpacity
          style={[styles.button, styles.cancelButton]}
          onPress={handleCancel}
        >
          <Text style={styles.cancelButtonText}>Cancel</Text>
        </TouchableOpacity>

        <View style={styles.rightButtons}>
          {currentStep > 0 && (
            <TouchableOpacity
              style={[styles.button, styles.secondaryButton]}
              onPress={handlePrevious}
            >
              <Text style={styles.secondaryButtonText}>Previous</Text>
            </TouchableOpacity>
          )}

          <TouchableOpacity
            style={[styles.button, styles.primaryButton]}
            onPress={() => handleNext(stepData[currentStep])}
          >
            <Text style={styles.primaryButtonText}>
              {currentStep === steps.length - 1 ? 'Complete' : 'Next'}
            </Text>
          </TouchableOpacity>
        </View>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#FFFFFF',
  },
  progressContainer: {
    height: 4,
    backgroundColor: '#E5E5EA',
    position: 'relative',
  },
  progressBar: {
    height: '100%',
    backgroundColor: '#007AFF',
  },
  progressText: {
    position: 'absolute',
    right: 16,
    top: -20,
    fontSize: 12,
    color: '#666',
  },
  stepIndicator: {
    flexDirection: 'row',
    justifyContent: 'center',
    padding: 20,
  },
  stepDot: {
    width: 40,
    height: 40,
    borderRadius: 20,
    backgroundColor: '#E5E5EA',
    justifyContent: 'center',
    alignItems: 'center',
    marginHorizontal: 8,
  },
  activeStepDot: {
    backgroundColor: '#007AFF',
  },
  stepNumber: {
    color: '#666',
    fontSize: 16,
    fontWeight: 'bold',
  },
  activeStepNumber: {
    color: '#FFFFFF',
  },
  content: {
    flex: 1,
    padding: 20,
  },
  stepTitle: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  stepDescription: {
    fontSize: 16,
    color: '#666',
    marginBottom: 20,
  },
  buttonContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    padding: 20,
    borderTopWidth: 1,
    borderTopColor: '#E5E5EA',
  },
  button: {
    padding: 15,
    borderRadius: 8,
    minWidth: 100,
    alignItems: 'center',
  },
  cancelButton: {
    backgroundColor: '#DC3545',
  },
  cancelButtonText: {
    color: '#FFFFFF',
    fontWeight: 'bold',
  },
  rightButtons: {
    flexDirection: 'row',
  },
  secondaryButton: {
    backgroundColor: '#6C757D',
    marginRight: 10,
  },
  secondaryButtonText: {
    color: '#FFFFFF',
  },
  primaryButton: {
    backgroundColor: '#007AFF',
  },
  primaryButtonText: {
    color: '#FFFFFF',
    fontWeight: 'bold',
  },
});

export default WizardNavigator;
```

---

## 🎣 **Navigation Hooks**

### **Custom Navigation Hooks**
```javascript
// hooks/useNavigationState.js
import { useNavigationState } from '@react-navigation/native';

export const useCurrentRoute = () => {
  const state = useNavigationState(state => state);
  return state.routes[state.index];
};

export const usePreviousRoute = () => {
  const state = useNavigationState(state => state);
  return state.index > 0 ? state.routes[state.index - 1] : null;
};

export const useNavigationHistory = () => {
  const state = useNavigationState(state => state);
  return state.routes.map(route => route.name);
};

// hooks/useNavigationFocus.js
import { useFocusEffect, useIsFocused } from '@react-navigation/native';
import { useCallback } from 'react';

export const useNavigationFocus = (callback) => {
  const isFocused = useIsFocused();

  useFocusEffect(
    useCallback(() => {
      if (isFocused) {
        callback();
      }
    }, [isFocused, callback])
  );

  return isFocused;
};

// hooks/useDeepLink.js
import { useEffect } from 'react';
import { useNavigation, useRoute } from '@react-navigation/native';

export const useDeepLink = () => {
  const navigation = useNavigation();
  const route = useRoute();

  useEffect(() => {
    // Handle deep link parameters
    if (route.params?.deepLinkAction) {
      handleDeepLinkAction(route.params.deepLinkAction);
    }
  }, [route.params]);

  const handleDeepLinkAction = (action) => {
    switch (action.type) {
      case 'navigate':
        navigation.navigate(action.screen, action.params);
        break;
      case 'reset':
        navigation.reset(action.state);
        break;
      case 'goBack':
        navigation.goBack();
        break;
    }
  };

  return {
    navigateToDeepLink: handleDeepLinkAction,
  };
};
```

### **Navigation State Persistence**
```javascript
// hooks/useNavigationPersistence.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import { useNavigationState } from '@react-navigation/native';
import { useEffect } from 'react';

const NAVIGATION_STATE_KEY = '@navigation_state';

export const useNavigationPersistence = () => {
  const state = useNavigationState(state => state);

  // Save navigation state
  useEffect(() => {
    const saveNavigationState = async () => {
      try {
        const jsonState = JSON.stringify(state);
        await AsyncStorage.setItem(NAVIGATION_STATE_KEY, jsonState);
      } catch (error) {
        console.error('Failed to save navigation state:', error);
      }
    };

    saveNavigationState();
  }, [state]);

  // Load navigation state
  const loadNavigationState = async () => {
    try {
      const jsonState = await AsyncStorage.getItem(NAVIGATION_STATE_KEY);
      return jsonState ? JSON.parse(jsonState) : undefined;
    } catch (error) {
      console.error('Failed to load navigation state:', error);
      return undefined;
    }
  };

  return {
    loadNavigationState,
  };
};
```

---

## 🔗 **Deep Linking**

### **Deep Link Configuration**
```javascript
// navigation/DeepLinkConfig.js
import { Linking } from 'react-native';

export const deepLinkConfig = {
  prefixes: [
    'myapp://',
    'https://myapp.com',
    'https://app.myapp.com',
  ],
  config: {
    screens: {
      Home: {
        path: 'home',
        screens: {
          Feed: 'feed',
          Profile: 'profile/:userId',
        },
      },
      Product: {
        path: 'product/:productId',
        parse: {
          productId: (productId) => parseInt(productId, 10),
        },
        stringify: {
          productId: (productId) => productId.toString(),
        },
      },
      Search: {
        path: 'search',
        parse: {
          query: (query) => decodeURIComponent(query),
        },
      },
      NotFound: '*',
    },
  },
};

// Deep link handler
export const handleDeepLink = (url) => {
  console.log('Received deep link:', url);

  // Parse URL and extract parameters
  const parsedUrl = Linking.parse(url);
  console.log('Parsed URL:', parsedUrl);

  // Handle different deep link types
  if (parsedUrl.path?.startsWith('/product/')) {
    const productId = parsedUrl.path.split('/')[2];
    return {
      screen: 'Product',
      params: { productId: parseInt(productId, 10) },
    };
  }

  if (parsedUrl.path?.startsWith('/profile/')) {
    const userId = parsedUrl.path.split('/')[2];
    return {
      screen: 'Profile',
      params: { userId },
    };
  }

  return {
    screen: 'Home',
    params: {},
  };
};

// Subscribe to deep links
export const subscribeToDeepLinks = (navigation) => {
  const linkingSubscription = Linking.addEventListener('url', ({ url }) => {
    const navigationParams = handleDeepLink(url);
    navigation.navigate(navigationParams.screen, navigationParams.params);
  });

  return linkingSubscription;
};
```

### **Universal Links (iOS)**
```objc
// ios/MyApp/AppDelegate.m
#import <React/RCTLinkingManager.h>

- (BOOL)application:(UIApplication *)application
            openURL:(NSURL *)url
            options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
{
  return [RCTLinkingManager application:application openURL:url options:options];
}

- (BOOL)application:(UIApplication *)application
  continueUserActivity:(NSUserActivity *)userActivity
    restorationHandler:(void (^)(NSArray<id<UIUserActivityRestoring>> * _Nullable))restorationHandler
{
  if ([userActivity.activityType isEqualToString:NSUserActivityTypeBrowsingWeb]) {
    return [RCTLinkingManager application:application
                      continueUserActivity:userActivity
                        restorationHandler:restorationHandler];
  }
  return NO;
}
```

### **App Links (Android)**
```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
  <application>
    <activity
      android:name=".MainActivity"
      android:launchMode="singleTask">
      <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="https"
              android:host="myapp.com" />
        <data android:scheme="https"
              android:host="app.myapp.com" />
      </intent-filter>
    </activity>
  </application>
</manifest>
```

---

## 🔐 **Authentication Flow**

### **Protected Routes**
```javascript
// navigation/ProtectedNavigator.js
import React from 'react';
import { createStackNavigator } from '@react-navigation/stack';
import { useAuth } from '../hooks/useAuth';

// Screens
import LoginScreen from '../screens/LoginScreen';
import RegisterScreen from '../screens/RegisterScreen';
import HomeScreen from '../screens/HomeScreen';
import ProfileScreen from '../screens/ProfileScreen';

const Stack = createStackNavigator();

const ProtectedNavigator = () => {
  const { isAuthenticated, isLoading } = useAuth();

  if (isLoading) {
    return <LoadingScreen />;
  }

  return (
    <Stack.Navigator
      screenOptions={{
        headerShown: false,
      }}
    >
      {!isAuthenticated ? (
        // Authentication Stack
        <>
          <Stack.Screen name="Login" component={LoginScreen} />
          <Stack.Screen name="Register" component={RegisterScreen} />
        </>
      ) : (
        // Protected Stack
        <>
          <Stack.Screen name="Home" component={HomeScreen} />
          <Stack.Screen name="Profile" component={ProfileScreen} />
        </>
      )}
    </Stack.Navigator>
  );
};

export default ProtectedNavigator;
```

### **Route Guards**
```javascript
// components/RouteGuard.js
import React, { useEffect } from 'react';
import { useNavigation } from '@react-navigation/native';
import { useAuth } from '../hooks/useAuth';

const RouteGuard = ({
  children,
  requiresAuth = false,
  requiredRole = null,
  fallbackScreen = 'Login'
}) => {
  const navigation = useNavigation();
  const { isAuthenticated, user, isLoading } = useAuth();

  useEffect(() => {
    if (isLoading) return;

    if (requiresAuth && !isAuthenticated) {
      navigation.replace(fallbackScreen);
      return;
    }

    if (requiredRole && user?.role !== requiredRole) {
      navigation.replace('Unauthorized');
      return;
    }
  }, [isAuthenticated, user, isLoading, navigation]);

  if (isLoading) {
    return <LoadingScreen />;
  }

  if (requiresAuth && !isAuthenticated) {
    return null;
  }

  if (requiredRole && user?.role !== requiredRole) {
    return <UnauthorizedScreen />;
  }

  return children;
};

// Usage
const AdminScreen = () => (
  <RouteGuard requiresAuth requiredRole="admin">
    <AdminDashboard />
  </RouteGuard>
);
```

---

## 📱 **Modal Navigation**

### **Modal Stack Navigator**
```javascript
// navigation/ModalNavigator.js
import React from 'react';
import { createStackNavigator } from '@react-navigation/stack';

// Screens
import HomeScreen from '../screens/HomeScreen';
import ProductModal from '../screens/ProductModal';
import FilterModal from '../screens/FilterModal';
import ConfirmModal from '../screens/ConfirmModal';

const Stack = createStackNavigator();

const ModalNavigator = () => {
  return (
    <Stack.Navigator
      mode="modal"
      screenOptions={{
        headerShown: false,
        cardStyle: { backgroundColor: 'transparent' },
        cardOverlayEnabled: true,
        cardStyleInterpolator: ({ current: { progress } }) => ({
          cardStyle: {
            opacity: progress.interpolate({
              inputRange: [0, 0.5, 0.9, 1],
              outputRange: [0, 0.25, 0.7, 1],
            }),
          },
          overlayStyle: {
            opacity: progress.interpolate({
              inputRange: [0, 1],
              outputRange: [0, 0.5],
              extrapolate: 'clamp',
            }),
          },
        }),
      }}
    >
      <Stack.Screen name="Home" component={HomeScreen} />
      <Stack.Screen
        name="ProductModal"
        component={ProductModal}
        options={{
          cardStyle: {
            backgroundColor: 'white',
            marginTop: 100,
            marginBottom: 100,
            marginLeft: 20,
            marginRight: 20,
            borderRadius: 10,
          },
        }}
      />
      <Stack.Screen
        name="FilterModal"
        component={FilterModal}
        options={{
          cardStyle: {
            backgroundColor: 'white',
            marginTop: 200,
            marginBottom: 200,
            marginLeft: 40,
            marginRight: 40,
            borderRadius: 15,
          },
          presentation: 'transparentModal',
        }}
      />
      <Stack.Screen
        name="ConfirmModal"
        component={ConfirmModal}
        options={{
          cardStyle: {
            backgroundColor: 'white',
            marginTop: 300,
            marginBottom: 300,
            marginLeft: 60,
            marginRight: 60,
            borderRadius: 20,
          },
          presentation: 'transparentModal',
        }}
      />
    </Stack.Navigator>
  );
};

export default ModalNavigator;
```

### **Custom Modal Presentation**
```javascript
// components/CustomModal.js
import React, { useEffect, useRef } from 'react';
import {
  View,
  TouchableOpacity,
  Animated,
  StyleSheet,
  Dimensions,
  PanResponder,
} from 'react-native';

const { height: screenHeight } = Dimensions.get('window');

const CustomModal = ({
  visible,
  onClose,
  children,
  height = 400,
  position = 'bottom',
}) => {
  const slideAnim = useRef(new Animated.Value(screenHeight)).current;
  const opacityAnim = useRef(new Animated.Value(0)).current;

  const panResponder = useRef(
    PanResponder.create({
      onMoveShouldSetPanResponder: (evt, gestureState) => {
        return Math.abs(gestureState.dy) > 20;
      },
      onPanResponderMove: (evt, gestureState) => {
        if (gestureState.dy > 0) {
          slideAnim.setValue(screenHeight - height + gestureState.dy);
        }
      },
      onPanResponderRelease: (evt, gestureState) => {
        if (gestureState.dy > 100) {
          closeModal();
        } else {
          Animated.spring(slideAnim, {
            toValue: screenHeight - height,
            useNativeDriver: true,
          }).start();
        }
      },
    })
  ).current;

  useEffect(() => {
    if (visible) {
      openModal();
    } else {
      closeModal();
    }
  }, [visible]);

  const openModal = () => {
    Animated.parallel([
      Animated.spring(slideAnim, {
        toValue: screenHeight - height,
        useNativeDriver: true,
      }),
      Animated.timing(opacityAnim, {
        toValue: 1,
        duration: 300,
        useNativeDriver: true,
      }),
    ]).start();
  };

  const closeModal = () => {
    Animated.parallel([
      Animated.timing(slideAnim, {
        toValue: screenHeight,
        duration: 300,
        useNativeDriver: true,
      }),
      Animated.timing(opacityAnim, {
        toValue: 0,
        duration: 300,
        useNativeDriver: true,
      }),
    ]).start(() => {
      onClose();
    });
  };

  if (!visible) return null;

  const modalPosition = position === 'bottom'
    ? { bottom: 0 }
    : position === 'top'
    ? { top: 0 }
    : { bottom: 0 };

  return (
    <View style={styles.overlay}>
      <Animated.View
        style={[
          styles.backdrop,
          { opacity: opacityAnim },
        ]}
      >
        <TouchableOpacity style={styles.backdropTouchable} onPress={closeModal} />
      </Animated.View>

      <Animated.View
        style={[
          styles.modal,
          modalPosition,
          {
            height,
            transform: [{ translateY: slideAnim }],
          },
        ]}
        {...panResponder.panHandlers}
      >
        <View style={styles.handle} />
        {children}
      </Animated.View>
    </View>
  );
};

const styles = StyleSheet.create({
  overlay: {
    ...StyleSheet.absoluteFillObject,
    zIndex: 1000,
  },
  backdrop: {
    ...StyleSheet.absoluteFillObject,
    backgroundColor: 'rgba(0, 0, 0, 0.5)',
  },
  backdropTouchable: {
    flex: 1,
  },
  modal: {
    position: 'absolute',
    left: 0,
    right: 0,
    backgroundColor: 'white',
    borderTopLeftRadius: 20,
    borderTopRightRadius: 20,
  },
  handle: {
    width: 40,
    height: 5,
    backgroundColor: '#DDD',
    borderRadius: 3,
    alignSelf: 'center',
    marginTop: 10,
    marginBottom: 10,
  },
});

export default CustomModal;
```

---

## 🎨 **Custom Transitions**

### **Custom Screen Transitions**
```javascript
// navigation/CustomTransition.js
import { CardStyleInterpolators } from '@react-navigation/stack';

export const customTransitions = {
  slideFromRight: {
    gestureDirection: 'horizontal',
    cardStyleInterpolator: CardStyleInterpolators.forHorizontalIOS,
  },

  slideFromBottom: {
    gestureDirection: 'vertical',
    cardStyleInterpolator: ({ current, next, layouts }) => {
      return {
        cardStyle: {
          transform: [
            {
              translateY: current.progress.interpolate({
                inputRange: [0, 1],
                outputRange: [layouts.screen.height, 0],
              }),
            },
          ],
        },
      };
    },
  },

  fadeTransition: {
    cardStyleInterpolator: ({ current }) => ({
      cardStyle: {
        opacity: current.progress,
      },
    }),
  },

  scaleTransition: {
    cardStyleInterpolator: ({ current }) => ({
      cardStyle: {
        transform: [
          {
            scale: current.progress.interpolate({
              inputRange: [0, 1],
              outputRange: [0.8, 1],
            }),
          },
        ],
        opacity: current.progress,
      },
    }),
  },

  flipTransition: {
    cardStyleInterpolator: ({ current, next, inverted, layouts }) => {
      const rotate = current.progress.interpolate({
        inputRange: [0, 1],
        outputRange: ['180deg', '0deg'],
      });

      return {
        cardStyle: {
          transform: [
            {
              rotateY: rotate,
            },
          ],
        },
      };
    },
  },
};
```

### **Advanced Transition Config**
```javascript
// navigation/TransitionConfig.js
import { TransitionSpecs, CardStyleInterpolators } from '@react-navigation/stack';

export const transitionConfig = {
  // Custom transition spec
  customTransition: {
    animation: 'spring',
    config: {
      stiffness: 1000,
      damping: 500,
      mass: 3,
      overshootClamping: true,
      restDisplacementThreshold: 0.01,
      restSpeedThreshold: 0.01,
    },
  },

  // Timing transition
  timingTransition: {
    animation: 'timing',
    config: {
      duration: 300,
      easing: Easing.out(Easing.poly(4)),
    },
  },

  // Gesture configuration
  gestureConfig: {
    gestureDirection: 'horizontal',
    gestureVelocityImpact: 0.3,
    gestureResponseDistance: {
      horizontal: 50,
      vertical: 135,
    },
  },
};

// Combined transition preset
export const createCustomTransition = (options = {}) => {
  const {
    type = 'slide',
    duration = 300,
    easing = Easing.out(Easing.poly(4)),
  } = options;

  return {
    gestureDirection: 'horizontal',
    transitionSpec: {
      open: {
        animation: 'timing',
        config: {
          duration,
          easing,
        },
      },
      close: {
        animation: 'timing',
        config: {
          duration: duration * 0.8,
          easing,
        },
      },
    },
    cardStyleInterpolator: ({ current, next, layouts }) => {
      switch (type) {
        case 'slide':
          return CardStyleInterpolators.forHorizontalIOS({ current, next, layouts });
        case 'fade':
          return {
            cardStyle: {
              opacity: current.progress,
            },
          };
        case 'scale':
          return {
            cardStyle: {
              transform: [
                {
                  scale: current.progress.interpolate({
                    inputRange: [0, 1],
                    outputRange: [0.9, 1],
                  }),
                },
              ],
            },
          };
        default:
          return CardStyleInterpolators.forHorizontalIOS({ current, next, layouts });
      }
    },
  };
};
```

---

## 📊 **Navigation State Management**

### **Navigation State Persistence**
```javascript
// store/slices/navigationSlice.js
import { createSlice } from '@reduxjs/toolkit';
import AsyncStorage from '@react-native-async-storage/async-storage';

const NAVIGATION_STATE_KEY = '@navigation_state';

const navigationSlice = createSlice({
  name: 'navigation',
  initialState: {
    currentRoute: null,
    navigationHistory: [],
    isReady: false,
    persistedState: null,
  },
  reducers: {
    setCurrentRoute: (state, action) => {
      state.currentRoute = action.payload;
      state.navigationHistory = [
        ...state.navigationHistory.slice(-9), // Keep last 10 routes
        action.payload,
      ];
    },
    setNavigationReady: (state, action) => {
      state.isReady = action.payload;
    },
    setPersistedState: (state, action) => {
      state.persistedState = action.payload;
    },
    clearNavigationHistory: (state) => {
      state.navigationHistory = [];
    },
  },
});

export const {
  setCurrentRoute,
  setNavigationReady,
  setPersistedState,
  clearNavigationHistory,
} = navigationSlice.actions;

// Thunks
export const loadPersistedNavigationState = () => async (dispatch) => {
  try {
    const persistedState = await AsyncStorage.getItem(NAVIGATION_STATE_KEY);
    if (persistedState) {
      const parsedState = JSON.parse(persistedState);
      dispatch(setPersistedState(parsedState));
    }
  } catch (error) {
    console.error('Failed to load persisted navigation state:', error);
  } finally {
    dispatch(setNavigationReady(true));
  }
};

export const saveNavigationState = (state) => async () => {
  try {
    await AsyncStorage.setItem(NAVIGATION_STATE_KEY, JSON.stringify(state));
  } catch (error) {
    console.error('Failed to save navigation state:', error);
  }
};

export default navigationSlice.reducer;
```

### **Navigation Analytics**
```javascript
// services/NavigationAnalytics.js
class NavigationAnalytics {
  constructor() {
    this.sessionStartTime = Date.now();
    this.pageViews = [];
    this.userJourney = [];
  }

  trackPageView(routeName, params = {}) {
    const pageView = {
      routeName,
      params,
      timestamp: Date.now(),
      sessionDuration: Date.now() - this.sessionStartTime,
    };

    this.pageViews.push(pageView);
    this.userJourney.push(routeName);

    // Send to analytics service
    this.sendAnalyticsEvent('page_view', pageView);
  }

  trackNavigation(from, to, method = 'push') {
    const navigationEvent = {
      from,
      to,
      method,
      timestamp: Date.now(),
      sessionDuration: Date.now() - this.sessionStartTime,
    };

    // Send to analytics service
    this.sendAnalyticsEvent('navigation', navigationEvent);
  }

  trackUserFlow() {
    const flow = {
      journey: this.userJourney,
      totalPages: this.pageViews.length,
      sessionDuration: Date.now() - this.sessionStartTime,
      averageTimePerPage: this.calculateAverageTimePerPage(),
      bounceRate: this.calculateBounceRate(),
    };

    this.sendAnalyticsEvent('user_flow', flow);
  }

  calculateAverageTimePerPage() {
    if (this.pageViews.length < 2) return 0;

    const totalTime = this.pageViews[this.pageViews.length - 1].timestamp -
                     this.pageViews[0].timestamp;
    return totalTime / this.pageViews.length;
  }

  calculateBounceRate() {
    if (this.pageViews.length === 1) return 100;
    return (this.pageViews.length === 1) ? 100 : 0;
  }

  sendAnalyticsEvent(eventName, data) {
    // Send to your analytics service (Firebase, Mixpanel, etc.)
    console.log(`Analytics Event: ${eventName}`, data);

    // Example: Firebase Analytics
    // analytics().logEvent(eventName, data);
  }

  getAnalyticsReport() {
    return {
      sessionDuration: Date.now() - this.sessionStartTime,
      totalPageViews: this.pageViews.length,
      uniqueRoutes: [...new Set(this.userJourney)].length,
      userJourney: this.userJourney,
      averageTimePerPage: this.calculateAverageTimePerPage(),
    };
  }
}

export default new NavigationAnalytics();
```

---

## ⚡ **Performance Optimization**

### **Navigation Performance Tips**
```javascript
// Lazy loading screens
const LazyHomeScreen = lazy(() => import('../screens/HomeScreen'));
const LazyProfileScreen = lazy(() => import('../screens/ProfileScreen'));

// Screen options optimization
const screenOptions = {
  headerShown: false,
  cardStyle: { backgroundColor: 'transparent' },
  // Disable animations for better performance
  animationEnabled: false,
};

// Navigation container optimization
<NavigationContainer
  theme={theme}
  // Enable linking for better performance
  linking={linking}
  // Disable fallback for production
  fallback={<LoadingScreen />}
  // Enable state persistence
  initialState={initialNavigationState}
  onStateChange={onNavigationStateChange}
>
  {/* Your navigators */}
</NavigationContainer>

// Screen preloading
const usePreloadScreen = (screenName) => {
  useEffect(() => {
    // Preload screen component
    const preloadScreen = async () => {
      try {
        await import(`../screens/${screenName}`);
      } catch (error) {
        console.error(`Failed to preload ${screenName}:`, error);
      }
    };

    preloadScreen();
  }, [screenName]);
};

// Debounced navigation
const useDebouncedNavigation = (delay = 300) => {
  const navigation = useNavigation();
  const timeoutRef = useRef();

  const debouncedNavigate = useCallback((screen, params) => {
    clearTimeout(timeoutRef.current);
    timeoutRef.current = setTimeout(() => {
      navigation.navigate(screen, params);
    }, delay);
  }, [navigation]);

  useEffect(() => {
    return () => clearTimeout(timeoutRef.current);
  }, []);

  return debouncedNavigate;
};
```

### **Memory Management**
```javascript
// Clean up navigation listeners
useEffect(() => {
  const unsubscribe = navigation.addListener('focus', () => {
    // Screen focused
  });

  return unsubscribe;
}, [navigation]);

// Optimize screen re-renders
const ScreenComponent = React.memo(({ route, navigation }) => {
  // Component logic
  return <View>...</View>;
});

// Use React.memo for screen components
export default React.memo(ScreenComponent);
```

---

## 🎯 **Practical Examples**

### **E-commerce Navigation**
```javascript
// navigation/EcommerceNavigator.js
import React from 'react';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

// Screens
import HomeScreen from '../screens/HomeScreen';
import ProductListScreen from '../screens/ProductListScreen';
import ProductDetailScreen from '../screens/ProductDetailScreen';
import CartScreen from '../screens/CartScreen';
import ProfileScreen from '../screens/ProfileScreen';
import SearchScreen from '../screens/SearchScreen';

const Stack = createStackNavigator();
const Tab = createBottomTabNavigator();

// Product stack
function ProductStack() {
  return (
    <Stack.Navigator
      screenOptions={{
        headerStyle: { backgroundColor: '#007AFF' },
        headerTintColor: '#FFFFFF',
      }}
    >
      <Stack.Screen
        name="ProductList"
        component={ProductListScreen}
        options={{ title: 'Products' }}
      />
      <Stack.Screen
        name="ProductDetail"
        component={ProductDetailScreen}
        options={({ route }) => ({
          title: route.params?.product?.name || 'Product Details',
        })}
      />
    </Stack.Navigator>
  );
}

// Search stack
function SearchStack() {
  return (
    <Stack.Navigator>
      <Stack.Screen
        name="Search"
        component={SearchScreen}
        options={{ headerShown: false }}
      />
    </Stack.Navigator>
  );
}

// Main tab navigator
function MainTabNavigator() {
  return (
    <Tab.Navigator
      screenOptions={{
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: '#8E8E93',
      }}
    >
      <Tab.Screen
        name="Home"
        component={HomeScreen}
        options={{
          tabBarLabel: 'Home',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home" color={color} size={size} />
          ),
        }}
      />
      <Tab.Screen
        name="Products"
        component={ProductStack}
        options={{
          tabBarLabel: 'Products',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="grid" color={color} size={size} />
          ),
        }}
      />
      <Tab.Screen
        name="Search"
        component={SearchStack}
        options={{
          tabBarLabel: 'Search',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="search" color={color} size={size} />
          ),
        }}
      />
      <Tab.Screen
        name="Cart"
        component={CartScreen}
        options={({ route }) => ({
          tabBarLabel: 'Cart',
          tabBarBadge: route.params?.cartCount || 0,
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="cart" color={color} size={size} />
          ),
        })}
      />
      <Tab.Screen
        name="Profile"
        component={ProfileScreen}
        options={{
          tabBarLabel: 'Profile',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="person" color={color} size={size} />
          ),
        }}
      />
    </Tab.Navigator>
  );
}

// Root navigator with modals
function EcommerceNavigator() {
  return (
    <Stack.Navigator mode="modal">
      <Stack.Screen
        name="MainTabs"
        component={MainTabNavigator}
        options={{ headerShown: false }}
      />
      <Stack.Screen
        name="ProductModal"
        component={ProductDetailScreen}
        options={{
          title: 'Product Details',
          headerStyle: { backgroundColor: '#007AFF' },
          headerTintColor: '#FFFFFF',
        }}
      />
    </Stack.Navigator>
  );
}

export default EcommerceNavigator;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Advanced Navigation Structure**: Nested navigators and complex flows
- ✅ **Custom Navigators**: Building custom tab bars and wizard flows
- ✅ **Navigation Hooks**: Custom hooks for navigation state and focus
- ✅ **Deep Linking**: URL-based navigation and universal links
- ✅ **Authentication Flow**: Protected routes and route guards
- ✅ **Modal Navigation**: Custom modal presentations and transitions
- ✅ **Custom Transitions**: Advanced screen transition animations
- ✅ **Navigation State Management**: Persistence and analytics
- ✅ **Performance Optimization**: Lazy loading and memory management

### **Best Practices**
1. **Plan navigation structure** before implementation
2. **Use nested navigators** for complex app structures
3. **Implement deep linking** for better user experience
4. **Handle authentication flows** properly with route guards
5. **Optimize performance** with lazy loading and debouncing
6. **Test navigation flows** thoroughly on real devices
7. **Monitor navigation analytics** for user behavior insights
8. **Handle edge cases** like network failures and invalid states

### **Next Steps**
- Learn about React Navigation v6 features
- Implement advanced state management with navigation
- Create custom navigation themes and styling
- Add accessibility features to navigation
- Implement navigation-based animations and micro-interactions

---

## 🎯 **Assignment**

### **Task 1: Advanced E-commerce Navigation**
Create a comprehensive e-commerce navigation system with:
- Nested product categories navigation
- Shopping cart with badge indicators
- Wishlist and