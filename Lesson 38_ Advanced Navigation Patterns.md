# Lesson 38: Advanced Navigation Patterns

## 🎯 **Learning Objectives**
- Master advanced React Navigation patterns and techniques
- Implement complex navigation flows with nested navigators
- Create custom navigation components and transitions
- Handle deep linking and universal links
- Implement authentication flows with navigation
- Optimize navigation performance and user experience

## 📚 **Table of Contents**
1. [Advanced Navigation Concepts](#advanced-navigation-concepts)
2. [Nested Navigators](#nested-navigators)
3. [Custom Navigation Components](#custom-navigation-components)
4. [Authentication Flow Navigation](#authentication-flow-navigation)
5. [Deep Linking](#deep-linking)
6. [Navigation State Management](#navigation-state-management)
7. [Custom Transitions](#custom-transitions)
8. [Navigation Performance](#navigation-performance)
9. [Testing Navigation](#testing-navigation)
10. [Practical Examples](#practical-examples)

---

## 🌐 **Advanced Navigation Concepts**

### **Navigation Container vs Navigator**
```javascript
// App.js - Root Navigation Container
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Stack = createStackNavigator();
const Tab = createBottomTabNavigator();

const App = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="MainTabs" component={MainTabNavigator} />
        <Stack.Screen name="Modal" component={ModalScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

const MainTabNavigator = () => {
  return (
    <Tab.Navigator>
      <Tab.Screen name="Home" component={HomeScreen} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
      <Tab.Screen name="Settings" component={SettingsScreen} />
    </Tab.Navigator>
  );
};
```

### **Navigation Prop Types**
```javascript
// Navigation prop types for different navigators
import { useNavigation, useRoute, useFocusEffect } from '@react-navigation/native';

// Stack Navigation Props
const StackScreen = ({ navigation, route }) => {
  // navigation: navigate, goBack, replace, push, pop, etc.
  // route: params, name, key

  const handleNavigate = () => {
    navigation.navigate('Detail', { itemId: 123 });
  };

  return <View>...</View>;
};

// Tab Navigation Props
const TabScreen = ({ navigation, route }) => {
  // Limited navigation options in tabs
  // Use navigation from parent stack if needed

  return <View>...</View>;
};
```

---

## 🏗️ **Nested Navigators**

### **Complex Navigation Structure**
```javascript
// navigation/AppNavigator.js
import React from 'react';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createDrawerNavigator } from '@react-navigation/drawer';

const Stack = createStackNavigator();
const Tab = createBottomTabNavigator();
const Drawer = createDrawerNavigator();

// Home Tab Navigator
const HomeTab = () => {
  return (
    <Stack.Navigator>
      <Stack.Screen
        name="Home"
        component={HomeScreen}
        options={{ headerShown: false }}
      />
      <Stack.Screen
        name="PostDetail"
        component={PostDetailScreen}
        options={{ title: 'Post' }}
      />
      <Stack.Screen
        name="CreatePost"
        component={CreatePostScreen}
        options={{ title: 'New Post' }}
      />
    </Stack.Navigator>
  );
};

// Profile Tab Navigator
const ProfileTab = () => {
  return (
    <Stack.Navigator>
      <Stack.Screen
        name="Profile"
        component={ProfileScreen}
        options={{ headerShown: false }}
      />
      <Stack.Screen
        name="EditProfile"
        component={EditProfileScreen}
        options={{ title: 'Edit Profile' }}
      />
      <Stack.Screen
        name="Settings"
        component={SettingsScreen}
        options={{ title: 'Settings' }}
      />
    </Stack.Navigator>
  );
};

// Main Tab Navigator
const MainTabNavigator = () => {
  return (
    <Tab.Navigator
      screenOptions={{
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: '#8E8E93',
      }}
    >
      <Tab.Screen
        name="HomeTab"
        component={HomeTab}
        options={{
          title: 'Home',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home" color={color} size={size} />
          ),
        }}
      />
      <Tab.Screen
        name="SearchTab"
        component={SearchScreen}
        options={{
          title: 'Search',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="search" color={color} size={size} />
          ),
        }}
      />
      <Tab.Screen
        name="ProfileTab"
        component={ProfileTab}
        options={{
          title: 'Profile',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="person" color={color} size={size} />
          ),
        }}
      />
    </Tab.Navigator>
  );
};

// Root Navigator with Drawer
const AppNavigator = () => {
  return (
    <Drawer.Navigator
      drawerContent={(props) => <CustomDrawerContent {...props} />}
    >
      <Drawer.Screen
        name="Main"
        component={MainTabNavigator}
        options={{ title: 'Home' }}
      />
      <Drawer.Screen
        name="Notifications"
        component={NotificationsScreen}
        options={{ title: 'Notifications' }}
      />
      <Drawer.Screen
        name="Bookmarks"
        component={BookmarksScreen}
        options={{ title: 'Bookmarks' }}
      />
    </Drawer.Navigator>
  );
};

export default AppNavigator;
```

### **Navigation between Nested Navigators**
```javascript
// utils/navigationHelpers.js
import { CommonActions, StackActions } from '@react-navigation/native';

// Navigate to a screen in a nested navigator
export const navigateToNestedScreen = (navigation, tabName, screenName, params = {}) => {
  navigation.navigate(tabName, {
    screen: screenName,
    params,
  });
};

// Reset navigation state
export const resetToScreen = (navigation, screenName, params = {}) => {
  navigation.dispatch(
    CommonActions.reset({
      index: 0,
      routes: [{ name: screenName, params }],
    })
  );
};

// Replace current screen
export const replaceScreen = (navigation, screenName, params = {}) => {
  navigation.dispatch(
    StackActions.replace(screenName, params)
  );
};

// Pop to specific screen in stack
export const popToScreen = (navigation, screenName) => {
  navigation.dispatch(
    StackActions.popToTop()
  );
  // Then navigate to the desired screen
  navigation.navigate(screenName);
};
```

---

## 🎨 **Custom Navigation Components**

### **Custom Tab Bar**
```javascript
// components/CustomTabBar.js
import React from 'react';
import { View, TouchableOpacity, Text, StyleSheet } from 'react-native';
import { useSafeAreaInsets } from 'react-native-safe-area-context';
import Animated, { useAnimatedStyle, withSpring } from 'react-native-reanimated';

const CustomTabBar = ({ state, descriptors, navigation }) => {
  const insets = useSafeAreaInsets();

  return (
    <View style={[styles.container, { paddingBottom: insets.bottom }]}>
      {state.routes.map((route, index) => {
        const { options } = descriptors[route.key];
        const label = options.tabBarLabel || options.title || route.name;
        const isFocused = state.index === index;

        const onPress = () => {
          const event = navigation.emit({
            type: 'tabPress',
            target: route.key,
            canPreventDefault: true,
          });

          if (!isFocused && !event.defaultPrevented) {
            navigation.navigate(route.name);
          }
        };

        const animatedStyle = useAnimatedStyle(() => ({
          transform: [{ scale: withSpring(isFocused ? 1.1 : 1) }],
        }));

        return (
          <TouchableOpacity
            key={route.key}
            onPress={onPress}
            style={styles.tab}
          >
            <Animated.View style={[styles.tabContent, animatedStyle]}>
              {options.tabBarIcon &&
                options.tabBarIcon({
                  color: isFocused ? '#007AFF' : '#8E8E93',
                  size: 24,
                })
              }
              <Text
                style={[
                  styles.label,
                  { color: isFocused ? '#007AFF' : '#8E8E93' },
                ]}
              >
                {label}
              </Text>
              {isFocused && <View style={styles.activeIndicator} />}
            </Animated.View>
          </TouchableOpacity>
        );
      })}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    backgroundColor: '#FFFFFF',
    borderTopWidth: 1,
    borderTopColor: '#E5E5E5',
    paddingTop: 8,
  },
  tab: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    paddingVertical: 8,
  },
  tabContent: {
    alignItems: 'center',
    justifyContent: 'center',
  },
  label: {
    fontSize: 12,
    fontWeight: '500',
    marginTop: 4,
  },
  activeIndicator: {
    position: 'absolute',
    bottom: -8,
    width: 30,
    height: 3,
    backgroundColor: '#007AFF',
    borderRadius: 2,
  },
});

export default CustomTabBar;
```

### **Custom Header Component**
```javascript
// components/CustomHeader.js
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { Ionicons } from '@expo/vector-icons';

const CustomHeader = ({
  title,
  showBack = true,
  rightComponent,
  onBackPress,
  backgroundColor = '#FFFFFF',
  textColor = '#000000',
}) => {
  const navigation = useNavigation();

  const handleBackPress = () => {
    if (onBackPress) {
      onBackPress();
    } else if (navigation.canGoBack()) {
      navigation.goBack();
    }
  };

  return (
    <View style={[styles.container, { backgroundColor }]}>
      <View style={styles.leftContainer}>
        {showBack && (
          <TouchableOpacity
            onPress={handleBackPress}
            style={styles.backButton}
          >
            <Ionicons name="arrow-back" size={24} color={textColor} />
          </TouchableOpacity>
        )}
      </View>

      <View style={styles.centerContainer}>
        <Text style={[styles.title, { color: textColor }]} numberOfLines={1}>
          {title}
        </Text>
      </View>

      <View style={styles.rightContainer}>
        {rightComponent}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'center',
    height: 56,
    paddingHorizontal: 16,
    borderBottomWidth: 1,
    borderBottomColor: '#E5E5E5',
  },
  leftContainer: {
    width: 40,
    alignItems: 'flex-start',
  },
  centerContainer: {
    flex: 1,
    alignItems: 'center',
  },
  rightContainer: {
    width: 40,
    alignItems: 'flex-end',
  },
  backButton: {
    padding: 8,
    marginLeft: -8,
  },
  title: {
    fontSize: 18,
    fontWeight: '600',
  },
});

export default CustomHeader;
```

---

## 🔐 **Authentication Flow Navigation**

### **Conditional Navigation Based on Auth State**
```javascript
// navigation/AuthNavigator.js
import React, { useContext } from 'react';
import { createStackNavigator } from '@react-navigation/stack';
import { AuthContext } from '../contexts/AuthContext';

// Auth Screens
import LoginScreen from '../screens/auth/LoginScreen';
import RegisterScreen from '../screens/auth/RegisterScreen';
import ForgotPasswordScreen from '../screens/auth/ForgotPasswordScreen';

// Main App Screens
import MainTabNavigator from './MainTabNavigator';

const Stack = createStackNavigator();

const AuthNavigator = () => {
  const { isAuthenticated, isLoading } = useContext(AuthContext);

  if (isLoading) {
    return <LoadingScreen />;
  }

  return (
    <Stack.Navigator
      screenOptions={{
        headerShown: false,
        cardStyle: { backgroundColor: '#FFFFFF' },
      }}
    >
      {isAuthenticated ? (
        // Authenticated user screens
        <Stack.Screen name="MainApp" component={MainTabNavigator} />
      ) : (
        // Unauthenticated user screens
        <>
          <Stack.Screen name="Login" component={LoginScreen} />
          <Stack.Screen name="Register" component={RegisterScreen} />
          <Stack.Screen name="ForgotPassword" component={ForgotPasswordScreen} />
        </>
      )}
    </Stack.Navigator>
  );
};

export default AuthNavigator;
```

### **Protected Route Component**
```javascript
// components/ProtectedRoute.js
import React, { useContext } from 'react';
import { View, Text, ActivityIndicator, StyleSheet } from 'react-native';
import { AuthContext } from '../contexts/AuthContext';

const ProtectedRoute = ({ children, fallback = null }) => {
  const { isAuthenticated, isLoading } = useContext(AuthContext);

  if (isLoading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" color="#007AFF" />
        <Text style={styles.loadingText}>Loading...</Text>
      </View>
    );
  }

  if (!isAuthenticated) {
    return fallback || (
      <View style={styles.unauthorizedContainer}>
        <Text style={styles.unauthorizedText}>
          Please log in to access this content
        </Text>
      </View>
    );
  }

  return children;
};

const styles = StyleSheet.create({
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#FFFFFF',
  },
  loadingText: {
    marginTop: 16,
    fontSize: 16,
    color: '#666',
  },
  unauthorizedContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#FFFFFF',
    padding: 20,
  },
  unauthorizedText: {
    fontSize: 18,
    textAlign: 'center',
    color: '#666',
  },
});

export default ProtectedRoute;
```

---

## 🔗 **Deep Linking**

### **Deep Link Configuration**
```javascript
// navigation/LinkingConfiguration.js
import { Linking } from 'react-native';

export const linking = {
  prefixes: [
    'myapp://',
    'https://myapp.com',
    'https://www.myapp.com',
  ],

  config: {
    screens: {
      Auth: {
        screens: {
          Login: 'login',
          Register: 'register',
          ForgotPassword: 'forgot-password',
        },
      },
      MainTabs: {
        screens: {
          HomeTab: {
            screens: {
              Home: 'home',
              PostDetail: 'post/:id',
              CreatePost: 'create-post',
            },
          },
          SearchTab: {
            screens: {
              Search: 'search',
              SearchResults: 'search/:query',
            },
          },
          ProfileTab: {
            screens: {
              Profile: 'profile',
              EditProfile: 'profile/edit',
              Settings: 'settings',
            },
          },
        },
      },
      Modal: 'modal',
    },
  },
};

// Handle incoming links
export const handleDeepLink = (url) => {
  console.log('Received deep link:', url);

  // Parse the URL and navigate accordingly
  const { pathname, queryParams } = Linking.parse(url);

  // Custom logic for handling different link types
  if (pathname.startsWith('/post/')) {
    const postId = pathname.split('/post/')[1];
    // Navigate to post detail
  } else if (pathname === '/profile') {
    // Navigate to profile
  }
};

// Set up deep link listener
export const setupDeepLinking = (navigation) => {
  const linkingSubscription = Linking.addEventListener('url', ({ url }) => {
    handleDeepLink(url);
  });

  // Handle initial URL
  Linking.getInitialURL().then((url) => {
    if (url) {
      handleDeepLink(url);
    }
  });

  return linkingSubscription;
};
```

### **Universal Links (iOS) and App Links (Android)**
```javascript
// ios/myapp/Info.plist (for iOS Universal Links)
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>CFBundleURLTypes</key>
  <array>
    <dict>
      <key>CFBundleURLSchemes</key>
      <array>
        <string>myapp</string>
      </array>
    </dict>
  </array>
  <key>com.apple.developer.associated-domains</key>
  <array>
    <string>applinks:myapp.com</string>
    <string>applinks:www.myapp.com</string>
  </array>
</dict>
</plist>

// android/app/src/main/AndroidManifest.xml (for Android App Links)
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
              android:host="www.myapp.com" />
      </intent-filter>
    </activity>
  </application>
</manifest>
```

---

## 📊 **Navigation State Management**

### **Navigation State Persistence**
```javascript
// utils/navigationState.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import { CommonActions } from '@react-navigation/native';

const NAVIGATION_STATE_KEY = '@navigation_state';

export const loadNavigationState = async () => {
  try {
    const jsonString = await AsyncStorage.getItem(NAVIGATION_STATE_KEY);
    return jsonString ? JSON.parse(jsonString) : undefined;
  } catch (error) {
    console.error('Failed to load navigation state:', error);
    return undefined;
  }
};

export const saveNavigationState = async (state) => {
  try {
    const jsonString = JSON.stringify(state);
    await AsyncStorage.setItem(NAVIGATION_STATE_KEY, jsonString);
  } catch (error) {
    console.error('Failed to save navigation state:', error);
  }
};

export const clearNavigationState = async () => {
  try {
    await AsyncStorage.removeItem(NAVIGATION_STATE_KEY);
  } catch (error) {
    console.error('Failed to clear navigation state:', error);
  }
};

// Filter out sensitive data before saving
export const getPersistableState = (state) => {
  return {
    ...state,
    routes: state.routes.map(route => ({
      ...route,
      // Remove sensitive params
      params: route.params ? {
        ...route.params,
        token: undefined,
        password: undefined,
      } : undefined,
    })),
  };
};
```

### **Navigation History Tracking**
```javascript
// hooks/useNavigationHistory.js
import { useRef, useEffect } from 'react';
import { useNavigation } from '@react-navigation/native';

export const useNavigationHistory = () => {
  const navigation = useNavigation();
  const historyRef = useRef([]);

  useEffect(() => {
    const unsubscribe = navigation.addListener('state', (e) => {
      const currentRoute = navigation.getCurrentRoute();
      if (currentRoute) {
        historyRef.current = [
          ...historyRef.current.slice(-9), // Keep last 10 routes
          {
            name: currentRoute.name,
            params: currentRoute.params,
            timestamp: Date.now(),
          },
        ];
      }
    });

    return unsubscribe;
  }, [navigation]);

  const getHistory = () => historyRef.current;

  const getPreviousRoute = () => {
    const history = historyRef.current;
    return history.length > 1 ? history[history.length - 2] : null;
  };

  const canGoBack = () => navigation.canGoBack();

  return {
    getHistory,
    getPreviousRoute,
    canGoBack,
  };
};
```

---

## 🎭 **Custom Transitions**

### **Custom Screen Transitions**
```javascript
// navigation/CustomTransition.js
import { CardStyleInterpolators } from '@react-navigation/stack';
import { TransitionSpecs } from '@react-navigation/stack';

// Custom slide from bottom transition
export const slideFromBottom = {
  gestureDirection: 'vertical',
  transitionSpec: {
    open: TransitionSpecs.TransitionIOSSpec,
    close: TransitionSpecs.TransitionIOSSpec,
  },
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
};

// Custom fade transition
export const fadeTransition = {
  transitionSpec: {
    open: {
      animation: 'timing',
      config: {
        duration: 300,
      },
    },
    close: {
      animation: 'timing',
      config: {
        duration: 200,
      },
    },
  },
  cardStyleInterpolator: ({ current }) => ({
    cardStyle: {
      opacity: current.progress,
    },
  }),
};

// Custom scale transition
export const scaleTransition = {
  cardStyleInterpolator: ({ current, next, layouts }) => {
    const progress = current.progress;

    return {
      cardStyle: {
        transform: [
          {
            scale: progress.interpolate({
              inputRange: [0, 1],
              outputRange: [0.8, 1],
            }),
          },
        ],
        opacity: progress.interpolate({
          inputRange: [0, 1],
          outputRange: [0.5, 1],
        }),
      },
    };
  },
};

// Custom slide from right with bounce
export const bounceSlideRight = {
  gestureDirection: 'horizontal',
  transitionSpec: {
    open: {
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
    close: {
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
  },
  cardStyleInterpolator: CardStyleInterpolators.forHorizontalIOS,
};
```

### **Modal Transitions**
```javascript
// navigation/ModalTransition.js
import { TransitionSpecs, CardStyleInterpolators } from '@react-navigation/stack';

// Modal slide up from bottom
export const modalSlideUp = {
  presentation: 'modal',
  gestureDirection: 'vertical',
  transitionSpec: {
    open: TransitionSpecs.TransitionIOSSpec,
    close: TransitionSpecs.TransitionIOSSpec,
  },
  cardStyleInterpolator: ({ current, layouts }) => ({
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
    overlayStyle: {
      opacity: current.progress.interpolate({
        inputRange: [0, 1],
        outputRange: [0, 0.5],
      }),
    },
  }),
};

// Modal fade in
export const modalFade = {
  presentation: 'modal',
  cardStyleInterpolator: ({ current }) => ({
    cardStyle: {
      opacity: current.progress,
      transform: [
        {
          scale: current.progress.interpolate({
            inputRange: [0, 1],
            outputRange: [0.9, 1],
          }),
        },
      ],
    },
    overlayStyle: {
      opacity: current.progress.interpolate({
        inputRange: [0, 1],
        outputRange: [0, 0.3],
      }),
    },
  }),
};

// Full screen modal
export const fullScreenModal = {
  presentation: 'modal',
  cardStyleInterpolator: CardStyleInterpolators.forVerticalIOS,
  gestureEnabled: true,
  gestureDirection: 'vertical',
};
```

---

## ⚡ **Navigation Performance**

### **Navigation Optimization Techniques**
```javascript
// components/LazyScreen.js
import React, { Suspense } from 'react';
import { View, ActivityIndicator, StyleSheet } from 'react-native';

// Lazy load screen components
const LazyHomeScreen = React.lazy(() => import('../screens/HomeScreen'));
const LazyProfileScreen = React.lazy(() => import('../screens/ProfileScreen'));
const LazySettingsScreen = React.lazy(() => import('../screens/SettingsScreen'));

const LoadingFallback = () => (
  <View style={styles.loadingContainer}>
    <ActivityIndicator size="large" color="#007AFF" />
  </View>
);

const styles = StyleSheet.create({
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#FFFFFF',
  },
});

// Wrap screens with Suspense
export const HomeScreen = (props) => (
  <Suspense fallback={<LoadingFallback />}>
    <LazyHomeScreen {...props} />
  </Suspense>
);

export const ProfileScreen = (props) => (
  <Suspense fallback={<LoadingFallback />}>
    <LazyProfileScreen {...props} />
  </Suspense>
);

export const SettingsScreen = (props) => (
  <Suspense fallback={<LoadingFallback />}>
    <LazySettingsScreen {...props} />
  </Suspense>
);
```

### **Navigation State Optimization**
```javascript
// hooks/useOptimizedNavigation.js
import { useCallback, useMemo } from 'react';
import { useNavigation, useRoute } from '@react-navigation/native';

// Memoized navigation functions
export const useOptimizedNavigation = () => {
  const navigation = useNavigation();
  const route = useRoute();

  const navigateToHome = useCallback(() => {
    navigation.navigate('Home');
  }, [navigation]);

  const navigateToProfile = useCallback((userId) => {
    navigation.navigate('Profile', { userId });
  }, [navigation]);

  const goBack = useCallback(() => {
    navigation.goBack();
  }, [navigation]);

  const currentRouteName = useMemo(() => route.name, [route.name]);

  const isCurrentRoute = useCallback((routeName) => {
    return currentRouteName === routeName;
  }, [currentRouteName]);

  return {
    navigateToHome,
    navigateToProfile,
    goBack,
    currentRouteName,
    isCurrentRoute,
  };
};

// Optimized screen component
const OptimizedScreen = React.memo(({ children, ...props }) => {
  const { currentRouteName } = useOptimizedNavigation();

  // Only render if this is the current screen
  if (props.route?.name !== currentRouteName) {
    return null;
  }

  return children;
});
```

---

## 🧪 **Testing Navigation**

### **Navigation Testing Utilities**
```javascript
// __tests__/navigation/AppNavigator.test.js
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { NavigationContainer } from '@react-navigation/native';

// Mock navigation
const mockNavigate = jest.fn();
const mockGoBack = jest.fn();

jest.mock('@react-navigation/native', () => ({
  ...jest.requireActual('@react-navigation/native'),
  useNavigation: () => ({
    navigate: mockNavigate,
    goBack: mockGoBack,
  }),
}));

// Test component wrapper
const TestWrapper = ({ children }) => (
  <NavigationContainer>
    {children}
  </NavigationContainer>
);

describe('Navigation', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('should navigate to profile screen', () => {
    const { getByText } = render(
      <TestWrapper>
        <HomeScreen />
      </TestWrapper>
    );

    const profileButton = getByText('Go to Profile');
    fireEvent.press(profileButton);

    expect(mockNavigate).toHaveBeenCalledWith('Profile', {
      userId: '123',
    });
  });

  test('should go back when back button is pressed', () => {
    const { getByText } = render(
      <TestWrapper>
        <ProfileScreen />
      </TestWrapper>
    );

    const backButton = getByText('Back');
    fireEvent.press(backButton);

    expect(mockGoBack).toHaveBeenCalled();
  });
});

// Deep linking tests
describe('Deep Linking', () => {
  test('should handle post deep link', () => {
    const mockUrl = 'myapp://post/123';

    // Mock Linking
    jest.mock('react-native/Libraries/Linking/Linking', () => ({
      addEventListener: jest.fn(),
      getInitialURL: jest.fn(() => Promise.resolve(mockUrl)),
    }));

    // Test deep link handling
    const { handleDeepLink } = require('../navigation/LinkingConfiguration');
    handleDeepLink(mockUrl);

    expect(mockNavigate).toHaveBeenCalledWith('PostDetail', { id: '123' });
  });
});
```

### **Integration Testing**
```javascript
// __tests__/integration/NavigationFlow.test.js
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { NavigationContainer } from '@react-navigation/native';
import App from '../App';

// Mock API calls
jest.mock('../services/api', () => ({
  login: jest.fn(),
  getPosts: jest.fn(),
}));

describe('Navigation Flow Integration', () => {
  test('should complete login to home navigation flow', async () => {
    const mockLogin = require('../services/api').login;
    const mockGetPosts = require('../services/api').getPosts;

    mockLogin.mockResolvedValue({
      token: 'fake-token',
      user: { id: 1, name: 'Test User' },
    });

    mockGetPosts.mockResolvedValue([
      { id: 1, title: 'Test Post', content: 'Test content' },
    ]);

    const { getByText, getByPlaceholderText, queryByText } = render(
      <NavigationContainer>
        <App />
      </NavigationContainer>
    );

    // Should start on login screen
    expect(getByText('Login')).toBeTruthy();

    // Fill login form
    const emailInput = getByPlaceholderText('Email');
    const passwordInput = getByPlaceholderText('Password');

    fireEvent.changeText(emailInput, 'test@example.com');
    fireEvent.changeText(passwordInput, 'password');

    // Submit login
    const loginButton = getByText('Login');
    fireEvent.press(loginButton);

    // Should navigate to home screen
    await waitFor(() => {
      expect(getByText('Welcome, Test User!')).toBeTruthy();
    });

    // Should show posts
    await waitFor(() => {
      expect(getByText('Test Post')).toBeTruthy();
    });
  });
});
```

---

## 🎯 **Practical Examples**

### **Complete E-commerce Navigation**
```javascript
// navigation/EcommerceNavigator.js
import React from 'react';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createDrawerNavigator } from '@react-navigation/drawer';

const Stack = createStackNavigator();
const Tab = createBottomTabNavigator();
const Drawer = createDrawerNavigator();

// Product Stack
const ProductStack = () => (
  <Stack.Navigator>
    <Stack.Screen
      name="ProductList"
      component={ProductListScreen}
      options={{ title: 'Products' }}
    />
    <Stack.Screen
      name="ProductDetail"
      component={ProductDetailScreen}
      options={{ title: 'Product Details' }}
    />
    <Stack.Screen
      name="ProductSearch"
      component={ProductSearchScreen}
      options={{ title: 'Search Products' }}
    />
  </Stack.Navigator>
);

// Cart Stack
const CartStack = () => (
  <Stack.Navigator>
    <Stack.Screen
      name="Cart"
      component={CartScreen}
      options={{ title: 'Shopping Cart' }}
    />
    <Stack.Screen
      name="Checkout"
      component={CheckoutScreen}
      options={{ title: 'Checkout' }}
    />
    <Stack.Screen
      name="OrderConfirmation"
      component={OrderConfirmationScreen}
      options={{ title: 'Order Confirmed' }}
    />
  </Stack.Navigator>
);

// Profile Stack
const ProfileStack = () => (
  <Stack.Navigator>
    <Stack.Screen
      name="Profile"
      component={ProfileScreen}
      options={{ headerShown: false }}
    />
    <Stack.Screen
      name="OrderHistory"
      component={OrderHistoryScreen}
      options={{ title: 'Order History' }}
    />
    <Stack.Screen
      name="Wishlist"
      component={WishlistScreen}
      options={{ title: 'Wishlist' }}
    />
    <Stack.Screen
      name="AddressBook"
      component={AddressBookScreen}
      options={{ title: 'Address Book' }}
    />
  </Stack.Navigator>
);

// Main Tab Navigator
const MainTabNavigator = () => (
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
        tabBarIcon: ({ color, size }) => (
          <Ionicons name="home" color={color} size={size} />
        ),
      }}
    />
    <Tab.Screen
      name="Products"
      component={ProductStack}
      options={{
        headerShown: false,
        tabBarIcon: ({ color, size }) => (
          <Ionicons name="basket" color={color} size={size} />
        ),
      }}
    />
    <Tab.Screen
      name="Cart"
      component={CartStack}
      options={{
        headerShown: false,
        tabBarIcon: ({ color, size }) => (
          <Ionicons name="cart" color={color} size={size} />
        ),
      }}
    />
    <Tab.Screen
      name="Profile"
      component={ProfileStack}
      options={{
        headerShown: false,
        tabBarIcon: ({ color, size }) => (
          <Ionicons name="person" color={color} size={size} />
        ),
      }}
    />
  </Tab.Navigator>
);

// Root Navigator
const EcommerceNavigator = () => (
  <Drawer.Navigator
    drawerContent={(props) => <CustomDrawer {...props} />}
  >
    <Drawer.Screen
      name="Main"
      component={MainTabNavigator}
      options={{ title: 'Shop' }}
    />
    <Drawer.Screen
      name="Categories"
      component={CategoriesScreen}
      options={{ title: 'Categories' }}
    />
    <Drawer.Screen
      name="Settings"
      component={SettingsScreen}
      options={{ title: 'Settings' }}
    />
  </Drawer.Navigator>
);

export default EcommerceNavigator;
```

### **Social Media Navigation**
```javascript
// navigation/SocialNavigator.js
import React from 'react';
import { createStackNavigator } from '@react-navigation/stack';
import { createMaterialTopTabNavigator } from '@react-navigation/material-top-tabs';

const Stack = createStackNavigator();
const TopTab = createMaterialTopTabNavigator();

// Feed Tabs
const FeedTabs = () => (
  <TopTab.Navigator
    screenOptions={{
      tabBarIndicatorStyle: { backgroundColor: '#007AFF' },
      tabBarActiveTintColor: '#007AFF',
      tabBarInactiveTintColor: '#8E8E93',
    }}
  >
    <TopTab.Screen name="ForYou" component={ForYouFeedScreen} />
    <TopTab.Screen name="Following" component={FollowingFeedScreen} />
    <TopTab.Screen name="Trending" component={TrendingFeedScreen} />
  </TopTab.Navigator>
);

// Main Stack Navigator
const SocialNavigator = () => (
  <Stack.Navigator
    screenOptions={{
      headerStyle: {
        backgroundColor: '#FFFFFF',
        elevation: 0,
        shadowOpacity: 0,
      },
      headerTintColor: '#000000',
      headerTitleStyle: {
        fontWeight: 'bold',
      },
    }}
  >
    <Stack.Screen
      name="Feed"
      component={FeedTabs}
      options={{
        headerTitle: 'Social App',
        headerRight: () => <NotificationButton />,
      }}
    />
    <Stack.Screen
      name="PostDetail"
      component={PostDetailScreen}
      options={({ route }) => ({
        title: route.params?.post?.title || 'Post',
        headerRight: () => <PostActionsMenu />,
      })}
    />
    <Stack.Screen
      name="CreatePost"
      component={CreatePostScreen}
      options={{
        title: 'Create Post',
        headerLeft: () => <CancelButton />,
        headerRight: () => <PostButton />,
        gestureEnabled: false,
      }}
    />
    <Stack.Screen
      name="UserProfile"
      component={UserProfileScreen}
      options={({ route }) => ({
        title: route.params?.user?.name || 'Profile',
        headerRight: () => <UserActionsMenu />,
      })}
    />
    <Stack.Screen
      name="Chat"
      component={ChatScreen}
      options={({ route }) => ({
        title: route.params?.user?.name || 'Chat',
        headerRight: () => <ChatActionsMenu />,
      })}
    />
    <Stack.Screen
      name="Search"
      component={SearchScreen}
      options={{
        headerTitle: '',
        headerTransparent: true,
        headerTintColor: '#FFFFFF',
      }}
    />
  </Stack.Navigator>
);

export default SocialNavigator;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Advanced Navigation Concepts**: Navigation containers, nested navigators, navigation props
- ✅ **Nested Navigators**: Complex navigation structures with stack, tab, and drawer navigators
- ✅ **Custom Navigation Components**: Custom tab bars, headers, and navigation components
- ✅ **Authentication Flow Navigation**: Conditional navigation based on auth state
- ✅ **Deep Linking**: URL-based navigation and universal links
- ✅ **Navigation State Management**: State persistence and history tracking
- ✅ **Custom Transitions**: Custom screen transitions and modal animations
- ✅ **Navigation Performance**: Lazy loading, optimization techniques
- ✅ **Testing Navigation**: Unit and integration testing for navigation
- ✅ **Practical Implementation**: Complete e-commerce and social media navigation examples

### **Best Practices**
1. **Plan navigation structure** - Design navigation hierarchy before implementation
2. **Use nested navigators** - Organize screens logically with nested navigation
3. **Implement deep linking** - Support universal links and app links
4. **Handle authentication flows** - Conditional navigation based on auth state
5. **Optimize performance** - Use lazy loading and memoization
6. **Create custom components** - Build reusable navigation components
7. **Test navigation flows** - Comprehensive testing of navigation scenarios
8. **Handle edge cases** - Back button, deep linking, state persistence
9. **Use TypeScript** - Type safety for navigation props and params
10. **Monitor navigation** - Track navigation patterns and performance

### **Next Steps**
- Learn about React Navigation v6 advanced features
- Implement navigation with Redux integration
- Study navigation patterns in large-scale apps
- Explore custom navigation solutions
- Learn about navigation in React Native web
- Implement A/B testing for navigation flows
- Study accessibility in navigation
- Explore navigation with animations and gestures

---

## 🎯 **Assignment**

### **Task 1: Advanced E-commerce Navigation**
Create a comprehensive e-commerce navigation system with:
- Multi-level category navigation with breadcrumbs
- Product filtering and search with navigation state
- Shopping cart with persistent state across navigation
- User authentication flow with protected routes
- Order tracking with deep linking support
- Wishlist management with navigation integration
- Custom transitions for product detail screens

### **Task 2: Social Media Navigation**
Build a social media app navigation with:
- Feed tabs (For You, Following, Trending) with swipe gestures
- Post detail screens with nested comment navigation
- User profile navigation with follower/following lists
- Direct messaging with conversation navigation
- Search with recent searches and suggestions
- Notification center with deep linking
- Custom bottom tab bar with badges

### **Task 3: Custom Navigation Components**
Implement custom navigation components including:
- Animated tab bar with custom icons and transitions
- Custom header with search bar and action buttons
- Drawer menu with user profile and navigation items
- Modal navigation with custom presentations
- Floating action button navigation
- Bottom sheet navigation for filters and actions

### **Task 4: Navigation State Management**
Create advanced navigation state management with:
- Navigation history tracking and analytics
- State persistence across app restarts
- Navigation-based caching strategies
- Undo/redo functionality for navigation
- Navigation breadcrumbs for complex flows
- State synchronization across multiple navigators

---

## 📚 **Additional Resources**
- [React Navigation Documentation](https://reactnavigation.org/docs/getting-started)
- [React Navigation Advanced Patterns](https://reactnavigation.org/docs/advanced-guides/)
- [Deep Linking Guide](https://reactnavigation.org/docs/deep-linking/)
- [Custom Transitions](https://reactnavigation.org/docs/transitioner/)
- [Navigation Performance](https://reactnavigation.org/docs/navigation-performance/)

---

**Next Lesson**: [Lesson 39: Performance Monitoring & Analytics](Lesson%2039_%20Performance%20Monitoring%20&%20Analytics.md)