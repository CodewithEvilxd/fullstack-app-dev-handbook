esson 8_ Navigation Basics (React Navigation).md</path>
<content">const App = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator
        initialRouteName="Home"
        screenOptions={{
          headerStyle: {
            backgroundColor: '#007AFF',
          },
          headerTintColor: '#fff',
          headerTitleStyle: {
            fontWeight: 'bold',
          },
          headerBackTitleVisible: false,
        }}
      >
        <Stack.Screen
          name="Home"
          component={HomeScreen}
          options={{
            title: 'React Native App',
            headerRight: () => (
              <TouchableOpacity
                onPress={() => Alert.alert('Menu', 'Menu options would go here')}
                style={{ marginRight: 15 }}
              >
                <Text style={{ color: '#fff', fontSize: 16 }}>Menu</Text>
              </TouchableOpacity>
            ),
          }}
        />
        <Stack.Screen
          name="Details"
          component={DetailsScreen}
          options={{
            headerBackTitle: 'Back',
          }}
        />
        <Stack.Screen
          name="Settings"
          component={SettingsScreen}
          options={{
            title: 'App Settings',
            presentation: 'modal',
          }}
        />
        <Stack.Screen
          name="Profile"
          component={ProfileScreen}
          options={{
            title: 'User Profile',
          }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

---

## 📱 Tab Navigation

Tab Navigation bottom tabs provide karta hai jo different sections ke beech switch karne mein help karta hai.

### 1. Basic Tab Navigation

```javascript
// App.js - Tab Navigation
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import HomeScreen from './screens/HomeScreen';
import SearchScreen from './screens/SearchScreen';
import ProfileScreen from './screens/ProfileScreen';
import SettingsScreen from './screens/SettingsScreen';

const Tab = createBottomTabNavigator();

const App = () => {
  return (
    <NavigationContainer>
      <Tab.Navigator
        initialRouteName="Home"
        screenOptions={{
          tabBarActiveTintColor: '#007AFF',
          tabBarInactiveTintColor: '#666',
          tabBarStyle: {
            backgroundColor: 'white',
            borderTopWidth: 1,
            borderTopColor: '#eee',
            paddingBottom: 5,
            paddingTop: 5,
            height: 60,
          },
          headerStyle: {
            backgroundColor: '#007AFF',
          },
          headerTintColor: '#fff',
          headerTitleStyle: {
            fontWeight: 'bold',
          },
        }}
      >
        <Tab.Screen
          name="Home"
          component={HomeScreen}
          options={{
            title: 'Home',
            tabBarLabel: 'Home',
            tabBarIcon: ({ color, size }) => (
              <Text style={{ color, fontSize: size }}>🏠</Text>
            ),
          }}
        />
        <Tab.Screen
          name="Search"
          component={SearchScreen}
          options={{
            title: 'Search',
            tabBarLabel: 'Search',
            tabBarIcon: ({ color, size }) => (
              <Text style={{ color, fontSize: size }}>🔍</Text>
            ),
          }}
        />
        <Tab.Screen
          name="Profile"
          component={ProfileScreen}
          options={{
            title: 'Profile',
            tabBarLabel: 'Profile',
            tabBarIcon: ({ color, size }) => (
              <Text style={{ color, fontSize: size }}>👤</Text>
            ),
          }}
        />
        <Tab.Screen
          name="Settings"
          component={SettingsScreen}
          options={{
            title: 'Settings',
            tabBarLabel: 'Settings',
            tabBarIcon: ({ color, size }) => (
              <Text style={{ color, fontSize: size }}>⚙️</Text>
            ),
          }}
        />
      </Tab.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

### 2. Tab Navigation with Stack

```javascript
// App.js - Nested Navigation
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createStackNavigator } from '@react-navigation/stack';

// Stack Navigators for each tab
const HomeStack = createStackNavigator();
const SearchStack = createStackNavigator();
const ProfileStack = createStackNavigator();

const HomeStackScreen = () => (
  <HomeStack.Navigator>
    <HomeStack.Screen name="Home" component={HomeScreen} />
    <HomeStack.Screen name="Details" component={DetailsScreen} />
  </HomeStack.Navigator>
);

const SearchStackScreen = () => (
  <SearchStack.Navigator>
    <SearchStack.Screen name="Search" component={SearchScreen} />
    <SearchStack.Screen name="Results" component={SearchResultsScreen} />
  </SearchStack.Navigator>
);

const ProfileStackScreen = () => (
  <ProfileStack.Navigator>
    <ProfileStack.Screen name="Profile" component={ProfileScreen} />
    <ProfileStack.Screen name="EditProfile" component={EditProfileScreen} />
  </ProfileStack.Navigator>
);

const Tab = createBottomTabNavigator();

const App = () => {
  return (
    <NavigationContainer>
      <Tab.Navigator
        screenOptions={{
          tabBarActiveTintColor: '#007AFF',
          tabBarInactiveTintColor: '#666',
        }}
      >
        <Tab.Screen
          name="Home"
          component={HomeStackScreen}
          options={{
            headerShown: false,
            tabBarIcon: ({ color, size }) => (
              <Text style={{ color, fontSize: size }}>🏠</Text>
            ),
          }}
        />
        <Tab.Screen
          name="Search"
          component={SearchStackScreen}
          options={{
            headerShown: false,
            tabBarIcon: ({ color, size }) => (
              <Text style={{ color, fontSize: size }}>🔍</Text>
            ),
          }}
        />
        <Tab.Screen
          name="Profile"
          component={ProfileStackScreen}
          options={{
            headerShown: false,
            tabBarIcon: ({ color, size }) => (
              <Text style={{ color, fontSize: size }}>👤</Text>
            ),
          }}
        />
      </Tab.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

---

## 📂 Drawer Navigation

Drawer Navigation side menu provide karta hai jo complex navigation ke liye useful hota hai.

### 1. Basic Drawer Navigation

```javascript
// App.js - Drawer Navigation
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createDrawerNavigator } from '@react-navigation/drawer';
import HomeScreen from './screens/HomeScreen';
import ProfileScreen from './screens/ProfileScreen';
import SettingsScreen from './screens/SettingsScreen';
import AboutScreen from './screens/AboutScreen';

const Drawer = createDrawerNavigator();

const App = () => {
  return (
    <NavigationContainer>
      <Drawer.Navigator
        initialRouteName="Home"
        screenOptions={{
          drawerStyle: {
            backgroundColor: '#f5f5f5',
            width: 280,
          },
          drawerActiveTintColor: '#007AFF',
          drawerInactiveTintColor: '#666',
          headerStyle: {
            backgroundColor: '#007AFF',
          },
          headerTintColor: '#fff',
          headerTitleStyle: {
            fontWeight: 'bold',
          },
        }}
      >
        <Drawer.Screen
          name="Home"
          component={HomeScreen}
          options={{
            title: 'Home',
            drawerIcon: ({ color, size }) => (
              <Text style={{ color, fontSize: size }}>🏠</Text>
            ),
          }}
        />
        <Drawer.Screen
          name="Profile"
          component={ProfileScreen}
          options={{
            title: 'My Profile',
            drawerIcon: ({ color, size }) => (
              <Text style={{ color, fontSize: size }}>👤</Text>
            ),
          }}
        />
        <Drawer.Screen
          name="Settings"
          component={SettingsScreen}
          options={{
            title: 'Settings',
            drawerIcon: ({ color, size }) => (
              <Text style={{ color, fontSize: size }}>⚙️</Text>
            ),
          }}
        />
        <Drawer.Screen
          name="About"
          component={AboutScreen}
          options={{
            title: 'About App',
            drawerIcon: ({ color, size }) => (
              <Text style={{ color, fontSize: size }}>ℹ️</Text>
            ),
          }}
        />
      </Drawer.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

### 2. Custom Drawer Content

```javascript
// components/CustomDrawer.js
import React from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  Image,
  ScrollView
} from 'react-native';
import {
  DrawerContentScrollView,
  DrawerItemList,
  DrawerItem,
} from '@react-navigation/drawer';

const CustomDrawer = (props) => {
  const user = {
    name: 'John Doe',
    email: 'john@example.com',
    avatar: 'https://picsum.photos/100/100?random=1',
  };

  const handleLogout = () => {
    // Handle logout logic
    console.log('Logout pressed');
  };

  return (
    <DrawerContentScrollView {...props} style={styles.container}>
      {/* User Profile Section */}
      <View style={styles.userSection}>
        <Image source={{ uri: user.avatar }} style={styles.avatar} />
        <View style={styles.userInfo}>
          <Text style={styles.userName}>{user.name}</Text>
          <Text style={styles.userEmail}>{user.email}</Text>
        </View>
      </View>

      {/* Navigation Items */}
      <View style={styles.navigationSection}>
        <DrawerItemList {...props} />
      </View>

      {/* Custom Items */}
      <View style={styles.customSection}>
        <TouchableOpacity style={styles.customItem}>
          <Text style={styles.customItemText}>📞 Support</Text>
        </TouchableOpacity>

        <TouchableOpacity style={styles.customItem}>
          <Text style={styles.customItemText}>⭐ Rate App</Text>
        </TouchableOpacity>

        <TouchableOpacity style={styles.customItem}>
          <Text style={styles.customItemText}>📤 Share App</Text>
        </TouchableOpacity>
      </View>

      {/* Logout Button */}
      <View style={styles.logoutSection}>
        <TouchableOpacity style={styles.logoutButton} onPress={handleLogout}>
          <Text style={styles.logoutText}>🚪 Logout</Text>
        </TouchableOpacity>
      </View>
    </DrawerContentScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  userSection: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#007AFF',
  },
  avatar: {
    width: 60,
    height: 60,
    borderRadius: 30,
    marginRight: 15,
  },
  userInfo: {
    flex: 1,
  },
  userName: {
    fontSize: 18,
    fontWeight: 'bold',
    color: 'white',
    marginBottom: 2,
  },
  userEmail: {
    fontSize: 14,
    color: 'rgba(255,255,255,0.8)',
  },
  navigationSection: {
    flex: 1,
    marginTop: 20,
  },
  customSection: {
    marginTop: 20,
    paddingHorizontal: 20,
  },
  customItem: {
    paddingVertical: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  customItemText: {
    fontSize: 16,
    color: '#333',
  },
  logoutSection: {
    marginTop: 20,
    paddingHorizontal: 20,
    paddingBottom: 20,
  },
  logoutButton: {
    backgroundColor: '#FF3B30',
    paddingVertical: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  logoutText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default CustomDrawer;
```

```javascript
// App.js - Using Custom Drawer
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createDrawerNavigator } from '@react-navigation/drawer';
import CustomDrawer from './components/CustomDrawer';
import HomeScreen from './screens/HomeScreen';
import ProfileScreen from './screens/ProfileScreen';
import SettingsScreen from './screens/SettingsScreen';

const Drawer = createDrawerNavigator();

const App = () => {
  return (
    <NavigationContainer>
      <Drawer.Navigator
        drawerContent={(props) => <CustomDrawer {...props} />}
        screenOptions={{
          headerStyle: {
            backgroundColor: '#007AFF',
          },
          headerTintColor: '#fff',
        }}
      >
        <Drawer.Screen name="Home" component={HomeScreen} />
        <Drawer.Screen name="Profile" component={ProfileScreen} />
        <Drawer.Screen name="Settings" component={SettingsScreen} />
      </Drawer.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

---

## 🔄 Navigation Parameters

### 1. Passing Parameters

```javascript
// Sending parameters
const navigateToDetails = () => {
  navigation.navigate('Details', {
    id: 123,
    title: 'Sample Item',
    data: {
      description: 'This is a description',
      category: 'Technology',
      tags: ['react', 'native', 'navigation']
    }
  });
};

// Receiving parameters
const DetailsScreen = ({ route, navigation }) => {
  const { id, title, data } = route.params;

  // Optional parameters with default values
  const description = data?.description || 'No description';
  const category = data?.category || 'Uncategorized';

  return (
    <View style={styles.container}>
      <Text style={styles.title}>{title}</Text>
      <Text style={styles.description}>{description}</Text>
      <Text style={styles.category}>{category}</Text>
    </View>
  );
};
```

### 2. Updating Parameters

```javascript
// Update current screen parameters
const updateParams = () => {
  navigation.setParams({
    title: 'Updated Title',
    timestamp: new Date().toISOString()
  });
};

// Listen to parameter changes
useEffect(() => {
  const unsubscribe = navigation.addListener('focus', () => {
    // Screen is focused, parameters might have changed
    console.log('Screen focused with params:', route.params);
  });

  return unsubscribe;
}, [navigation]);
```

---

## 🎨 Navigation Theming

### 1. Custom Theme

```javascript
// themes.js
export const lightTheme = {
  dark: false,
  colors: {
    primary: '#007AFF',
    background: '#f5f5f5',
    card: 'white',
    text: '#333',
    border: '#ddd',
    notification: '#FF3B30',
  },
};

export const darkTheme = {
  dark: true,
  colors: {
    primary: '#0A84FF',
    background: '#000000',
    card: '#1C1C1E',
    text: '#FFFFFF',
    border: '#38383A',
    notification: '#FF453A',
  },
};
```

```javascript
// App.js - Themed Navigation
import React, { useState } from 'react';
import { NavigationContainer, DefaultTheme, DarkTheme } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { lightTheme, darkTheme } from './themes';

const Stack = createStackNavigator();

const App = () => {
  const [isDarkTheme, setIsDarkTheme] = useState(false);

  const toggleTheme = () => {
    setIsDarkTheme(!isDarkTheme);
  };

  return (
    <NavigationContainer theme={isDarkTheme ? darkTheme : lightTheme}>
      <Stack.Navigator
        screenOptions={{
          headerStyle: {
            backgroundColor: isDarkTheme ? darkTheme.colors.card : lightTheme.colors.primary,
          },
          headerTintColor: isDarkTheme ? darkTheme.colors.text : lightTheme.colors.card,
        }}
      >
        <Stack.Screen
          name="Home"
          component={HomeScreen}
          initialParams={{ toggleTheme, isDarkTheme }}
        />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

---

## 📱 Deep Linking

### 1. Basic Deep Linking

```javascript
// App.js - Deep Linking Setup
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';

const Stack = createStackNavigator();

const linking = {
  prefixes: ['myapp://', 'https://myapp.com'],
  config: {
    screens: {
      Home: 'home',
      Details: {
        path: 'details/:id',
        parse: {
          id: (id) => parseInt(id, 10),
        },
        stringify: {
          id: (id) => id.toString(),
        },
      },
      Profile: 'profile',
    },
  },
};

const App = () => {
  return (
    <NavigationContainer linking={linking}>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

### 2. Handling Deep Links

```javascript
// useDeepLink.js - Custom Hook
import { useEffect } from 'react';
import { useNavigation, useRoute } from '@react-navigation/native';
import { Linking } from 'react-native';

export const useDeepLink = () => {
  const navigation = useNavigation();
  const route = useRoute();

  useEffect(() => {
    const handleDeepLink = (event) => {
      const { url } = event;
      console.log('Deep link received:', url);

      // Parse the URL and navigate accordingly
      if (url.includes('details')) {
        const id = url.split('/').pop();
        navigation.navigate('Details', { id: parseInt(id, 10) });
      }
    };

    // Listen for deep link events
    const subscription = Linking.addEventListener('url', handleDeepLink);

    // Check if app was opened from a deep link
    Linking.getInitialURL().then((url) => {
      if (url) {
        handleDeepLink({ url });
      }
    });

    return () => {
      subscription?.remove();
    };
  }, [navigation]);

  return { currentRoute: route.name };
};
```

---

## 🎯 Practical Examples

### 1. E-commerce App Navigation

```javascript
// App.js - E-commerce Navigation
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Stack = createStackNavigator();
const Tab = createBottomTabNavigator();

// Product Stack
const ProductStack = () => (
  <Stack.Navigator>
    <Stack.Screen
      name="ProductList"
      component={ProductListScreen}
      options={{ title: 'Products' }}
    />
    <Stack.Screen
      name="ProductDetails"
      component={ProductDetailsScreen}
      options={{ title: 'Product Details' }}
    />
    <Stack.Screen
      name="AddToCart"
      component={AddToCartScreen}
      options={{ title: 'Add to Cart' }}
    />
  </Stack.Navigator>
);

// Profile Stack
const ProfileStack = () => (
  <Stack.Navigator>
    <Stack.Screen
      name="Profile"
      component={ProfileScreen}
      options={{ title: 'My Profile' }}
    />
    <Stack.Screen
      name="EditProfile"
      component={EditProfileScreen}
      options={{ title: 'Edit Profile' }}
    />
    <Stack.Screen
      name="OrderHistory"
      component={OrderHistoryScreen}
      options={{ title: 'Order History' }}
    />
  </Stack.Navigator>
);

// Main Tab Navigator
const MainTabs = () => (
  <Tab.Navigator
    screenOptions={{
      tabBarActiveTintColor: '#007AFF',
      tabBarInactiveTintColor: '#666',
    }}
  >
    <Tab.Screen
      name="Home"
      component={HomeScreen}
      options={{
        tabBarIcon: ({ color, size }) => (
          <Text style={{ color, fontSize: size }}>🏠</Text>
        ),
      }}
    />
    <Tab.Screen
      name="Products"
      component={ProductStack}
      options={{
        headerShown: false,
        tabBarIcon: ({ color, size }) => (
          <Text style={{ color, fontSize: size }}>🛍️</Text>
        ),
      }}
    />
    <Tab.Screen
      name="Cart"
      component={CartScreen}
      options={{
        tabBarIcon: ({ color, size }) => (
          <Text style={{ color, fontSize: size }}>🛒</Text>
        ),
        tabBarBadge: 3, // Show cart item count
      }}
    />
    <Tab.Screen
      name="Profile"
      component={ProfileStack}
      options={{
        headerShown: false,
        tabBarIcon: ({ color, size }) => (
          <Text style={{ color, fontSize: size }}>👤</Text>
        ),
      }}
    />
  </Tab.Navigator>
);

// Root Stack
const App = () => (
  <NavigationContainer>
    <Stack.Navigator>
      <Stack.Screen
        name="MainTabs"
        component={MainTabs}
        options={{ headerShown: false }}
      />
      <Stack.Screen
        name="Checkout"
        component={CheckoutScreen}
        options={{ title: 'Checkout' }}
      />
      <Stack.Screen
        name="Payment"
        component={PaymentScreen}
        options={{ title: 'Payment' }}
      />
    </Stack.Navigator>
  </NavigationContainer>
);

export default App;
```

### 2. Social Media App Navigation

```javascript
// App.js - Social Media Navigation
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createDrawerNavigator } from '@react-navigation/drawer';

const Stack = createStackNavigator();
const Tab = createBottomTabNavigator();
const Drawer = createDrawerNavigator();

// Feed Stack
const FeedStack = () => (
  <Stack.Navigator>
    <Stack.Screen
      name="Feed"
      component={FeedScreen}
      options={{ headerShown: false }}
    />
    <Stack.Screen
      name="PostDetails"
      component={PostDetailsScreen}
      options={{ title: 'Post' }}
    />
    <Stack.Screen
      name="CreatePost"
      component={CreatePostScreen}
      options={{ title: 'Create Post' }}
    />
    <Stack.Screen
      name="Comments"
      component={CommentsScreen}
      options={{ title: 'Comments' }}
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
      name="EditProfile"
      component={EditProfileScreen}
      options={{ title: 'Edit Profile' }}
    />
    <Stack.Screen
      name="Followers"
      component={FollowersScreen}
      options={{ title: 'Followers' }}
    />
    <Stack.Screen
      name="Following"
      component={FollowingScreen}
      options={{ title: 'Following' }}
    />
  </Stack.Navigator>
);

// Main Tab Navigator
const MainTabs = () => (
  <Tab.Navigator
    screenOptions={{
      tabBarActiveTintColor: '#007AFF',
      tabBarInactiveTintColor: '#666',
      tabBarStyle: {
        backgroundColor: 'white',
        borderTopWidth: 1,
        borderTopColor: '#eee',
      },
    }}
  >
    <Tab.Screen
      name="Feed"
      component={FeedStack}
      options={{
        headerShown: false,
        tabBarIcon: ({ color, size }) => (
          <Text style={{ color, fontSize: size }}>🏠</Text>
        ),
      }}
    />
    <Tab.Screen
      name="Search"
      component={SearchScreen}
      options={{
        tabBarIcon: ({ color, size }) => (
          <Text style={{ color, fontSize: size }}>🔍</Text>
        ),
      }}
    />
    <Tab.Screen
      name="Notifications"
      component={NotificationsScreen}
      options={{
        tabBarIcon: ({ color, size }) => (
          <Text style={{ color, fontSize: size }}>🔔</Text>
        ),
        tabBarBadge: 5, // Show notification count
      }}
    />
    <Tab.Screen
      name="Profile"
      component={ProfileStack}
      options={{
        headerShown: false,
        tabBarIcon: ({ color, size }) => (
          <Text style={{ color, fontSize: size }}>👤</Text>
        ),
      }}
    />
  </Tab.Navigator>
);

// Custom Drawer Content
const CustomDrawerContent = (props) => (
  <DrawerContentScrollView {...props}>
    <View style={styles.drawerHeader}>
      <Image
        source={{ uri: 'https://picsum.photos/100/100?random=1' }}
        style={styles.drawerAvatar}
      />
      <Text style={styles.drawerName}>John Doe</Text>
      <Text style={styles.drawerUsername}>@johndoe</Text>
    </View>

    <DrawerItemList {...props} />

    <TouchableOpacity style={styles.drawerItem}>
      <Text style={styles.drawerItemText}>⚙️ Settings</Text>
    </TouchableOpacity>

    <TouchableOpacity style={styles.drawerItem}>
      <Text style={styles.drawerItemText}>🌙 Dark Mode</Text>
    </TouchableOpacity>

    <TouchableOpacity style={styles.logoutButton}>
      <Text style={styles.logoutText}>🚪 Logout</Text>
    </TouchableOpacity>
  </DrawerContentScrollView>
);

// Root Drawer Navigator
const App = () => (
  <NavigationContainer>
    <Drawer.Navigator
      drawerContent={(props) => <CustomDrawerContent {...props} />}
      screenOptions={{
        headerStyle: {
          backgroundColor: '#007AFF',
        },
        headerTintColor: '#fff',
      }}
    >
      <Drawer.Screen
        name="MainTabs"
        component={MainTabs}
        options={{
          title: 'Home',
          headerShown: false,
        }}
      />
      <Drawer.Screen
        name="Messages"
        component={MessagesScreen}
        options={{
          title: 'Messages',
        }}
      />
      <Drawer.Screen
        name="Bookmarks"
        component={BookmarksScreen}
        options={{
          title: 'Bookmarks',
        }}
      />
    </Drawer.Navigator>
  </NavigationContainer>
);

const styles = StyleSheet.create({
  drawerHeader: {
    padding: 20,
    backgroundColor: '#007AFF',
    alignItems: 'center',
  },
  drawerAvatar: {
    width: 80,
    height: 80,
    borderRadius: 40,
    marginBottom: 10,
  },
  drawerName: {
    fontSize: 18,
    fontWeight: 'bold',
    color: 'white',
    marginBottom: 2,
  },
  drawerUsername: {
    fontSize: 14,
    color: 'rgba(255,255,255,0.8)',
  },
  drawerItem: {
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  drawerItemText: {
    fontSize: 16,
    color: '#333',
  },
  logoutButton: {
    margin: 20,
    padding: 15,
    backgroundColor: '#FF3B30',
    borderRadius: 8,
    alignItems: 'center',
  },
  logoutText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default App;
```

---

## 📝 Assignment 8: Navigation Implementation

### Task 1: Basic Navigation App (90 minutes)
Create a basic navigation app with:
- Stack navigation between screens
- Pass parameters between screens
- Custom header styling
- Navigation options

### Task 2: Tab-Based App (120 minutes)
Build a tab-based application with:
- Bottom tab navigation
- Nested stack navigators
- Tab badges and icons
- Custom tab styling

### Task 3: Drawer Navigation App (150 minutes)
Implement a drawer navigation app with:
- Custom drawer content
- Multiple navigation levels
- Drawer gestures
- User profile in drawer

---

## 🧠 Quiz 8: Navigation Basics

### Multiple Choice Questions (1-15)

**Q1. React Navigation ka main purpose kya hai?**
a) UI components banana
b) Screen navigation manage karna
c) Data storage karna
d) API calls karna

**Q2. Stack Navigator mein screens kaise manage hote hain?**
a) Queue ki tarah
b) Stack ki tarah
c) List ki tarah
d) Grid ki tarah

**Q3. Tab Navigation ka use kab hota hai?**
a) Complex navigation ke liye
b) Bottom navigation ke liye
c) Side menu ke liye
d) Modal screens ke liye

**Q4. Drawer Navigation ka main benefit kya hai?**
a) Fast navigation
b) Space saving
c) Complex menu structure
d) All of the above

**Q5. Navigation parameters ka use kyu hota hai?**
a) Screen styling ke liye
b) Data pass karne ke liye
c) Animation ke liye
d) Security ke liye

### True/False Questions (16-20)

**Q16. React Navigation automatically header provide karta hai.** (True/False)

**Q17. Stack Navigator mein multiple screens same time visible ho sakte hain.** (True/False)

**Q18. Tab Navigation sirf bottom tabs support karta hai.** (True/False)

**Q19. Drawer Navigation gesture-based hota hai.** (True/False)

**Q20. Navigation parameters immutable hote hain.** (True/False)

### Code Output Questions (21-25)

**Q21. Is code ka output kya hoga?**
```javascript
navigation.navigate('Details', { id: 123, name: 'John' });
```

**Q22. Is navigation mein problem kya hai?**
```javascript
const DetailsScreen = ({ navigation }) => {
  // No route prop used
  return <Text>Details Screen</Text>;
};
```

**Q23. Is tab configuration mein kya galat hai?**
```javascript
<Tab.Screen name="Home" component={HomeScreen} />
```

**Q24. Is drawer setup mein kya missing hai?**
```javascript
const Drawer = createDrawerNavigator();
const App = () => (
  <Drawer.Navigator>
    <Drawer.Screen name="Home" component={HomeScreen} />
  </Drawer.Navigator>
);
```

**Q25. Is navigation code mein kya problem hai?**
```javascript
const goBack = () => {
  navigation.goBack();
  // No error handling
};
```

---

## 📚 Answers & Explanations

### Multiple Choice Answers
**Q1.** b) Screen navigation manage karna  
**Q2.** b) Stack ki tarah  
**Q3.** b) Bottom navigation ke liye  
**Q4.** d) All of the above  
**Q5.** b) Data pass karne ke liye  

### True/False Answers
**Q16.** True - React Navigation automatically header provide karta hai  
**Q17.** False - Stack Navigator mein ek screen at a time visible hota hai  
**Q18.** False - Tab Navigation top tabs bhi support karta hai  
**Q19.** True - Drawer Navigation gesture-based hota hai  
**Q20.** False - Navigation parameters mutable hote hain  

### Code Output Answers
**Q21.** Details screen open hoga with parameters  
**Q22.** Route parameters access nahi kar sakte  
**Q23.** Tab icon missing hai  
**Q24.** NavigationContainer missing hai  
**Q25.** Error handling missing hai  

---

## 🎯 Assignment Solutions

### Task 1: Basic Navigation Solution

```javascript
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import HomeScreen from './screens/HomeScreen';
import DetailsScreen from './screens/DetailsScreen';
import ProfileScreen from './screens/ProfileScreen';

const Stack = createStackNavigator();

const App = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator
        initialRouteName="Home"
        screenOptions={{
          headerStyle: {
            backgroundColor: '#007AFF',
          },
          headerTintColor: '#fff',
          headerTitleStyle: {
            fontWeight: 'bold',
          },
        }}
      >
        <Stack.Screen
          name="Home"
          component={HomeScreen}
          options={{ title: 'My App' }}
        />
        <Stack.Screen
          name="Details"
          component={DetailsScreen}
          options={{ title: 'Details' }}
        />
        <Stack.Screen
          name="Profile"
          component={ProfileScreen}
          options={{ title: 'Profile' }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

---

## 📝 Lesson Summary

### Key Concepts Learned
- ✅ **React Navigation Setup**: Installation and basic configuration
- ✅ **Stack Navigation**: Screen management with push/pop operations
- ✅ **Tab Navigation**: Bottom tab navigation with icons and badges
- ✅ **Drawer Navigation**: Side menu navigation with custom content
- ✅ **Navigation Parameters**: Passing and receiving data between screens
- ✅ **Navigation Theming**: Custom themes and styling
- ✅ **Deep Linking**: URL-based navigation
- ✅ **Nested Navigation**: Combining different navigation types

### Best Practices
1. **Use appropriate navigator** for your app structure
2. **Always handle navigation errors** gracefully
3. **Use TypeScript** for better navigation type safety
4. **Implement proper loading states** for navigation
5. **Test navigation flows** thoroughly
6. **Use navigation hooks** for better code organization
7. **Handle deep links** properly for better UX
8. **Optimize navigation performance** with proper memoization

### Next Steps
- Learn about React Navigation 6 advanced features
- Study navigation state persistence
- Implement authentication flows with navigation
- Explore custom navigation transitions

---

## 📚 Additional Resources
- [React Navigation Documentation](https://reactnavigation.org/docs/getting-started)
- [React Navigation Stack](https://reactnavigation.org/docs/stack-navigator)
- [React Navigation Bottom Tabs](https://reactnavigation.org/docs/bottom-tab-navigator)
- [React Navigation Drawer](https://reactnavigation.org/docs/drawer-navigator)
- [Navigation Best Practices](https://reactnavigation.org/docs/best-practices)

---

**Next Lesson**: [Lesson 9: Images, Icons & Media Handling](Lesson%209_%20Images,%20Icons%20&%20Media%20Handling.md)