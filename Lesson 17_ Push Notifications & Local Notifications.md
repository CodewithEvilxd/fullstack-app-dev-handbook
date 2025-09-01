# Lesson 17: Push Notifications & Local Notifications

## 🎯 **Learning Objectives**
- Implement push notifications in React Native apps
- Set up local notifications for scheduled alerts
- Handle notification permissions and user preferences
- Integrate with Firebase Cloud Messaging (FCM)
- Create rich notifications with media and actions

## 📚 **Table of Contents**
1. [Introduction to Notifications](#introduction-to-notifications)
2. [Local Notifications](#local-notifications)
3. [Push Notifications Setup](#push-notifications-setup)
4. [Firebase Cloud Messaging](#firebase-cloud-messaging)
5. [Notification Permissions](#notification-permissions)
6. [Rich Notifications](#rich-notifications)
7. [Notification Actions](#notification-actions)
8. [Background Processing](#background-processing)
9. [Testing & Debugging](#testing--debugging)
10. [Practical Examples](#practical-examples)

---

## 🔔 **Introduction to Notifications**

Notifications are essential for engaging users and providing timely information. React Native supports both local notifications (scheduled on device) and push notifications (sent from server).

### **Types of Notifications**
- **Local Notifications**: Scheduled and triggered on the device
- **Push Notifications**: Sent from a server through FCM/APNs
- **Rich Notifications**: Include images, buttons, and custom layouts
- **Silent Notifications**: Background data updates without UI

### **Popular Libraries**
- **@react-native-async-storage/async-storage**: For storing notification preferences
- **@react-native-community/push-notification-ios**: iOS push notifications
- **react-native-notifications**: Cross-platform notification library
- **@react-native-firebase/messaging**: Firebase integration

---

## 📱 **Local Notifications**

### **Basic Local Notification**
```javascript
import PushNotification from 'react-native-push-notification';

// Configure push notifications
PushNotification.configure({
  onRegister: function (token) {
    console.log('TOKEN:', token);
  },

  onNotification: function (notification) {
    console.log('NOTIFICATION:', notification);
    notification.finish(PushNotificationIOS.FetchResult.NoData);
  },

  permissions: {
    alert: true,
    badge: true,
    sound: true,
  },

  popInitialNotification: true,
  requestPermissions: true,
});

// Schedule a local notification
const scheduleNotification = () => {
  PushNotification.localNotificationSchedule({
    id: '123',
    title: 'Reminder',
    message: 'This is a scheduled notification',
    date: new Date(Date.now() + 60 * 1000), // 1 minute from now
    allowWhileIdle: true,
    playSound: true,
    soundName: 'default',
    vibrate: true,
    vibration: 300,
  });
};

// Immediate local notification
const showNotification = () => {
  PushNotification.localNotification({
    id: '456',
    title: 'Hello!',
    message: 'This is an immediate notification',
    playSound: true,
    soundName: 'default',
    vibrate: true,
    vibration: 300,
  });
};
```

### **Advanced Local Notification Scheduling**
```javascript
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, TextInput, StyleSheet } from 'react-native';
import PushNotification from 'react-native-push-notification';
import DateTimePicker from '@react-native-community/datetimepicker';

const NotificationScheduler = () => {
  const [title, setTitle] = useState('');
  const [message, setMessage] = useState('');
  const [scheduledTime, setScheduledTime] = useState(new Date());
  const [showPicker, setShowPicker] = useState(false);

  const scheduleNotification = () => {
    if (!title.trim() || !message.trim()) {
      alert('Please enter both title and message');
      return;
    }

    PushNotification.localNotificationSchedule({
      id: Date.now().toString(),
      title: title.trim(),
      message: message.trim(),
      date: scheduledTime,
      allowWhileIdle: true,
      playSound: true,
      soundName: 'default',
      vibrate: true,
      vibration: 1000,
      repeatType: 'day', // 'day', 'week', 'month', 'year'
      repeatTime: 1,
    });

    alert('Notification scheduled successfully!');
    setTitle('');
    setMessage('');
  };

  const onDateChange = (event, selectedDate) => {
    setShowPicker(false);
    if (selectedDate) {
      setScheduledTime(selectedDate);
    }
  };

  const cancelAllNotifications = () => {
    PushNotification.cancelAllLocalNotifications();
    alert('All notifications cancelled');
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Schedule Notification</Text>

      <TextInput
        style={styles.input}
        placeholder="Notification Title"
        value={title}
        onChangeText={setTitle}
      />

      <TextInput
        style={styles.input}
        placeholder="Notification Message"
        value={message}
        onChangeText={setMessage}
        multiline
      />

      <TouchableOpacity style={styles.dateButton} onPress={() => setShowPicker(true)}>
        <Text style={styles.dateButtonText}>
          Scheduled for: {scheduledTime.toLocaleString()}
        </Text>
      </TouchableOpacity>

      {showPicker && (
        <DateTimePicker
          value={scheduledTime}
          mode="datetime"
          display="default"
          onChange={onDateChange}
          minimumDate={new Date()}
        />
      )}

      <TouchableOpacity style={styles.scheduleButton} onPress={scheduleNotification}>
        <Text style={styles.scheduleButtonText}>Schedule Notification</Text>
      </TouchableOpacity>

      <TouchableOpacity style={styles.cancelButton} onPress={cancelAllNotifications}>
        <Text style={styles.cancelButtonText}>Cancel All Notifications</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
  },
  input: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 10,
    marginBottom: 15,
    fontSize: 16,
  },
  dateButton: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    marginBottom: 15,
    alignItems: 'center',
  },
  dateButtonText: {
    color: 'white',
    fontSize: 16,
  },
  scheduleButton: {
    backgroundColor: '#28a745',
    padding: 15,
    borderRadius: 10,
    marginBottom: 15,
    alignItems: 'center',
  },
  scheduleButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  cancelButton: {
    backgroundColor: '#dc3545',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
  },
  cancelButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default NotificationScheduler;
```

---

## 🚀 **Push Notifications Setup**

### **Firebase Configuration**
```javascript
// android/app/google-services.json
{
  "project_info": {
    "project_number": "123456789",
    "project_id": "your-project-id",
    "storage_bucket": "your-project-id.appspot.com"
  },
  "client": [
    {
      "client_info": {
        "mobilesdk_app_id": "1:123456789:android:abc123def456"
      },
      "oauth_client": [],
      "api_key": [
        {
          "current_key": "AIzaSyAbCdEfGhIjKlMnOpQrStUvWxYz"
        }
      ],
      "services": {
        "appinvite_service": {
          "other_platform_oauth_client": []
        }
      }
    }
  ],
  "configuration_version": "1"
}
```

### **iOS Configuration**
```javascript
// ios/YourApp/AppDelegate.m
#import <Firebase.h>
#import <React/RCTBridge.h>
#import <React/RCTBundleURLProvider.h>
#import <React/RCTRootView.h>
#import <React/RCTAppSetupUtils.h>

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  [FIRApp configure];
  // ... rest of the setup
}

- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
  [FIRMessaging messaging].APNSToken = deviceToken;
}

@end
```

---

## 🔥 **Firebase Cloud Messaging**

### **FCM Integration**
```javascript
import messaging from '@react-native-firebase/messaging';
import { Alert } from 'react-native';

// Request permission
const requestPermission = async () => {
  const authStatus = await messaging().requestPermission();
  const enabled =
    authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
    authStatus === messaging.AuthorizationStatus.PROVISIONAL;

  if (enabled) {
    console.log('Authorization status:', authStatus);
  }
};

// Get FCM token
const getFCMToken = async () => {
  const fcmToken = await messaging().getToken();
  if (fcmToken) {
    console.log('FCM Token:', fcmToken);
    // Send token to your server
  }
};

// Handle background messages
messaging().setBackgroundMessageHandler(async remoteMessage => {
  console.log('Message handled in the background!', remoteMessage);
});

// Handle foreground messages
const unsubscribe = messaging().onMessage(async remoteMessage => {
  Alert.alert(
    remoteMessage.notification.title,
    remoteMessage.notification.body,
    [{ text: 'OK' }]
  );
});

// Handle notification opened
messaging().onNotificationOpenedApp(remoteMessage => {
  console.log('Notification caused app to open:', remoteMessage);
});

// Check initial notification
messaging()
  .getInitialNotification()
  .then(remoteMessage => {
    if (remoteMessage) {
      console.log('App opened from notification:', remoteMessage);
    }
  });
```

### **Server-Side FCM**
```javascript
// Node.js server example
const admin = require('firebase-admin');

const serviceAccount = require('./serviceAccountKey.json');

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
});

const sendNotification = async (token, title, body, data = {}) => {
  const message = {
    notification: {
      title,
      body,
    },
    data,
    token,
  };

  try {
    const response = await admin.messaging().send(message);
    console.log('Successfully sent message:', response);
  } catch (error) {
    console.log('Error sending message:', error);
  }
};

// Usage
sendNotification(
  'device-fcm-token',
  'New Message',
  'You have a new message from John',
  { userId: '123', messageId: '456' }
);
```

---

## 🔐 **Notification Permissions**

### **Permission Management**
```javascript
import { PermissionsAndroid, Platform, Alert } from 'react-native';
import messaging from '@react-native-firebase/messaging';

const checkNotificationPermission = async () => {
  if (Platform.OS === 'android') {
    try {
      const granted = await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.POST_NOTIFICATIONS
      );
      return granted === PermissionsAndroid.RESULTS.GRANTED;
    } catch (err) {
      console.warn(err);
      return false;
    }
  } else {
    // iOS permissions are handled by Firebase
    const authStatus = await messaging().requestPermission();
    return authStatus === messaging.AuthorizationStatus.AUTHORIZED;
  }
};

const NotificationPermissionManager = () => {
  const [permissionGranted, setPermissionGranted] = useState(false);

  useEffect(() => {
    checkPermissionStatus();
  }, []);

  const checkPermissionStatus = async () => {
    const hasPermission = await checkNotificationPermission();
    setPermissionGranted(hasPermission);
  };

  const requestPermission = async () => {
    const granted = await checkNotificationPermission();
    setPermissionGranted(granted);

    if (granted) {
      Alert.alert('Success', 'Notification permission granted!');
    } else {
      Alert.alert(
        'Permission Denied',
        'Please enable notifications in settings to receive updates.',
        [
          { text: 'Cancel', style: 'cancel' },
          { text: 'Settings', onPress: () => Linking.openSettings() },
        ]
      );
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Notification Permissions</Text>

      <View style={styles.statusContainer}>
        <Text style={styles.statusLabel}>Permission Status:</Text>
        <Text style={[styles.status, permissionGranted ? styles.granted : styles.denied]}>
          {permissionGranted ? 'Granted' : 'Denied'}
        </Text>
      </View>

      {!permissionGranted && (
        <TouchableOpacity style={styles.button} onPress={requestPermission}>
          <Text style={styles.buttonText}>Request Permission</Text>
        </TouchableOpacity>
      )}

      <TouchableOpacity style={styles.refreshButton} onPress={checkPermissionStatus}>
        <Text style={styles.refreshButtonText}>Refresh Status</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    justifyContent: 'center',
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
  },
  statusContainer: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 10,
    marginBottom: 20,
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  },
  statusLabel: {
    fontSize: 16,
    fontWeight: 'bold',
  },
  status: {
    fontSize: 16,
    fontWeight: 'bold',
  },
  granted: {
    color: '#28a745',
  },
  denied: {
    color: '#dc3545',
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
    marginBottom: 15,
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  refreshButton: {
    backgroundColor: '#6c757d',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
  },
  refreshButtonText: {
    color: 'white',
    fontSize: 16,
  },
});

export default NotificationPermissionManager;
```

---

## 🎨 **Rich Notifications**

### **Notifications with Images**
```javascript
import PushNotification from 'react-native-push-notification';

// Rich notification with image
const showRichNotification = () => {
  PushNotification.localNotification({
    id: '789',
    title: 'Photo Update',
    message: 'Check out this amazing photo!',
    bigPictureUrl: 'https://example.com/image.jpg', // Android
    largeIconUrl: 'https://example.com/icon.jpg', // Android
    userInfo: {
      image: 'https://example.com/image.jpg', // iOS
    },
    playSound: true,
    soundName: 'default',
  });
};

// iOS rich notification setup
// Add this to AppDelegate.m for iOS rich notifications
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
{
  // Handle rich notification content
  NSDictionary *aps = userInfo[@"aps"];
  NSDictionary *alert = aps[@"alert"];

  if (alert[@"title"] && alert[@"body"]) {
    UNMutableNotificationContent *content = [[UNMutableNotificationContent alloc] init];
    content.title = alert[@"title"];
    content.body = alert[@"body"];
    content.sound = [UNNotificationSound defaultSound];

    // Add image attachment
    if (userInfo[@"image"]) {
      NSURL *imageUrl = [NSURL URLWithString:userInfo[@"image"]];
      UNNotificationAttachment *attachment = [UNNotificationAttachment attachmentWithIdentifier:@"image" URL:imageUrl options:nil error:nil];
      if (attachment) {
        content.attachments = @[attachment];
      }
    }

    UNTimeIntervalNotificationTrigger *trigger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:1 repeats:NO];
    UNNotificationRequest *request = [UNNotificationRequest requestWithIdentifier:@"rich-notification" content:content trigger:trigger];

    [[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request withCompletionHandler:nil];
  }

  completionHandler(UIBackgroundFetchResultNewData);
}
```

### **Expandable Notifications**
```javascript
const showExpandableNotification = () => {
  PushNotification.localNotification({
    id: '101',
    title: 'Long Message',
    message: 'This is a short message',
    bigText: 'This is the expanded version of the notification with much more detailed information about what happened. You can include multiple lines of text here to provide comprehensive details to the user.',
    subText: 'Additional context',
    playSound: true,
    soundName: 'default',
    vibrate: true,
    vibration: 300,
  });
};
```

---

## 🎯 **Notification Actions**

### **Interactive Notifications**
```javascript
import PushNotification from 'react-native-push-notification';

// Configure notification actions
PushNotification.configure({
  // ... other config
  onNotification: function (notification) {
    if (notification.action === 'Accept') {
      // Handle accept action
      console.log('User accepted the notification');
    } else if (notification.action === 'Decline') {
      // Handle decline action
      console.log('User declined the notification');
    }

    notification.finish(PushNotificationIOS.FetchResult.NoData);
  },
});

// Create notification with actions
const showActionNotification = () => {
  PushNotification.localNotification({
    id: '202',
    title: 'Meeting Request',
    message: 'John wants to schedule a meeting',
    actions: '["Accept", "Decline"]', // Android
    userInfo: {
      actions: ['Accept', 'Decline'], // iOS
    },
    playSound: true,
    soundName: 'default',
  });
};

// iOS notification categories
// Add this to AppDelegate.m
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  // Register notification categories
  UNNotificationCategory *meetingCategory = [UNNotificationCategory
    categoryWithIdentifier:@"MEETING_REQUEST"
    actions:@[
      [UNNotificationAction actionWithIdentifier:@"ACCEPT" title:@"Accept" options:UNNotificationActionOptionForeground],
      [UNNotificationAction actionWithIdentifier:@"DECLINE" title:@"Decline" options:UNNotificationActionOptionDestructive]
    ]
    intentIdentifiers:@[]
    options:UNNotificationCategoryOptionNone];

  [[UNUserNotificationCenter currentNotificationCenter] setNotificationCategories:[NSSet setWithObject:meetingCategory]];

  return YES;
}
```

### **Quick Reply Notifications**
```javascript
const showQuickReplyNotification = () => {
  PushNotification.localNotification({
    id: '303',
    title: 'New Message',
    message: 'Sarah: Hey, how are you?',
    reply_placeholder_text: 'Type your reply...', // Android
    reply_button_text: 'Reply', // Android
    userInfo: {
      reply: true, // iOS
    },
    playSound: true,
    soundName: 'default',
  });
};

// Handle quick reply
PushNotification.configure({
  onNotification: function (notification) {
    if (notification.reply_text) {
      // Handle the reply text
      console.log('User replied:', notification.reply_text);
      // Send reply to server
    }

    notification.finish(PushNotificationIOS.FetchResult.NoData);
  },
});
```

---

## 🔄 **Background Processing**

### **Background Notification Handling**
```javascript
import messaging from '@react-native-firebase/messaging';

// Handle background messages
messaging().setBackgroundMessageHandler(async remoteMessage => {
  console.log('Message handled in the background!', remoteMessage);

  // Process the message data
  const { data } = remoteMessage;

  if (data.type === 'update_location') {
    // Update user location in background
    // This could trigger location updates or other background tasks
  } else if (data.type === 'sync_data') {
    // Sync data in background
    // Perform data synchronization
  }

  return Promise.resolve();
});

// Handle notification when app is killed
messaging().getInitialNotification().then(remoteMessage => {
  if (remoteMessage) {
    console.log('App opened from killed state:', remoteMessage);
    // Navigate to appropriate screen based on notification
  }
});
```

### **Silent Notifications**
```javascript
// Server-side silent notification
const sendSilentNotification = async (token) => {
  const message = {
    data: {
      type: 'background_sync',
      userId: '123',
      syncType: 'messages',
    },
    token,
    android: {
      priority: 'high',
    },
    apns: {
      headers: {
        'apns-priority': '5',
      },
      payload: {
        aps: {
          contentAvailable: true,
        },
      },
    },
  };

  await admin.messaging().send(message);
};

// Handle silent notification in app
messaging().onMessage(async remoteMessage => {
  const { data } = remoteMessage;

  if (data.type === 'background_sync') {
    // Perform background sync
    await performBackgroundSync(data.userId, data.syncType);
  }
});
```

---

## 🐛 **Testing & Debugging**

### **Notification Testing Tools**
```javascript
// Test notification helper
const testNotification = (type = 'local') => {
  const notificationId = Date.now().toString();

  if (type === 'local') {
    PushNotification.localNotification({
      id: notificationId,
      title: 'Test Notification',
      message: `This is a test ${type} notification`,
      playSound: true,
      soundName: 'default',
      vibrate: true,
      vibration: 300,
    });
  } else if (type === 'scheduled') {
    PushNotification.localNotificationSchedule({
      id: notificationId,
      title: 'Scheduled Test',
      message: 'This notification was scheduled',
      date: new Date(Date.now() + 10 * 1000), // 10 seconds
      allowWhileIdle: true,
    });
  }
};

// Debug notification events
PushNotification.configure({
  onRegister: function (token) {
    console.log('Device Token:', token);
  },

  onNotification: function (notification) {
    console.log('Notification Received:', {
      id: notification.id,
      title: notification.title,
      message: notification.message,
      data: notification.data,
      action: notification.action,
      reply_text: notification.reply_text,
    });

    notification.finish(PushNotificationIOS.FetchResult.NoData);
  },

  onAction: function (notification) {
    console.log('Action Received:', notification.action);
  },

  onRegistrationError: function (err) {
    console.error('Registration Error:', err);
  },
});
```

### **Notification Logger Component**
```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, TouchableOpacity, StyleSheet } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

const NotificationLogger = () => {
  const [logs, setLogs] = useState([]);

  useEffect(() => {
    loadLogs();
  }, []);

  const loadLogs = async () => {
    try {
      const storedLogs = await AsyncStorage.getItem('notification_logs');
      if (storedLogs) {
        setLogs(JSON.parse(storedLogs));
      }
    } catch (error) {
      console.error('Error loading logs:', error);
    }
  };

  const addLog = async (message, type = 'info') => {
    const newLog = {
      id: Date.now(),
      timestamp: new Date().toISOString(),
      message,
      type,
    };

    const updatedLogs = [newLog, ...logs].slice(0, 100); // Keep last 100 logs
    setLogs(updatedLogs);

    try {
      await AsyncStorage.setItem('notification_logs', JSON.stringify(updatedLogs));
    } catch (error) {
      console.error('Error saving logs:', error);
    }
  };

  const clearLogs = async () => {
    setLogs([]);
    await AsyncStorage.removeItem('notification_logs');
  };

  // Expose addLog function globally for use in notification handlers
  useEffect(() => {
    global.notificationLogger = addLog;
    return () => {
      delete global.notificationLogger;
    };
  }, [logs]);

  const getLogColor = (type) => {
    switch (type) {
      case 'error': return '#dc3545';
      case 'warning': return '#ffc107';
      case 'success': return '#28a745';
      default: return '#007AFF';
    }
  };

  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Notification Logs</Text>
        <TouchableOpacity style={styles.clearButton} onPress={clearLogs}>
          <Text style={styles.clearButtonText}>Clear</Text>
        </TouchableOpacity>
      </View>

      <ScrollView style={styles.logContainer}>
        {logs.length === 0 ? (
          <Text style={styles.emptyText}>No logs yet</Text>
        ) : (
          logs.map((log) => (
            <View key={log.id} style={styles.logItem}>
              <Text style={[styles.logTimestamp, { color: getLogColor(log.type) }]}>
                {new Date(log.timestamp).toLocaleString()}
              </Text>
              <Text style={styles.logMessage}>{log.message}</Text>
            </View>
          ))
        )}
      </ScrollView>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 15,
    backgroundColor: 'white',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
  },
  clearButton: {
    backgroundColor: '#dc3545',
    padding: 8,
    borderRadius: 5,
  },
  clearButtonText: {
    color: 'white',
    fontSize: 14,
  },
  logContainer: {
    flex: 1,
    padding: 15,
  },
  emptyText: {
    textAlign: 'center',
    color: '#666',
    fontSize: 16,
    marginTop: 50,
  },
  logItem: {
    backgroundColor: 'white',
    padding: 10,
    borderRadius: 5,
    marginBottom: 10,
  },
  logTimestamp: {
    fontSize: 12,
    fontWeight: 'bold',
    marginBottom: 5,
  },
  logMessage: {
    fontSize: 14,
    color: '#333',
  },
});

export default NotificationLogger;
```

---

## 🎯 **Practical Examples**

### **Task Reminder App**
```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, TextInput, TouchableOpacity, FlatList, StyleSheet } from 'react-native';
import PushNotification from 'react-native-push-notification';
import AsyncStorage from '@react-native-async-storage/async-storage';

const TaskReminderApp = () => {
  const [task, setTask] = useState('');
  const [reminderTime, setReminderTime] = useState('');
  const [tasks, setTasks] = useState([]);

  useEffect(() => {
    loadTasks();
  }, []);

  const loadTasks = async () => {
    try {
      const storedTasks = await AsyncStorage.getItem('tasks');
      if (storedTasks) {
        setTasks(JSON.parse(storedTasks));
      }
    } catch (error) {
      console.error('Error loading tasks:', error);
    }
  };

  const saveTasks = async (updatedTasks) => {
    try {
      await AsyncStorage.setItem('tasks', JSON.stringify(updatedTasks));
    } catch (error) {
      console.error('Error saving tasks:', error);
    }
  };

  const addTask = () => {
    if (!task.trim() || !reminderTime.trim()) {
      alert('Please enter both task and reminder time');
      return;
    }

    const minutes = parseInt(reminderTime);
    if (isNaN(minutes) || minutes <= 0) {
      alert('Please enter a valid number of minutes');
      return;
    }

    const newTask = {
      id: Date.now().toString(),
      title: task.trim(),
      reminderTime: minutes,
      createdAt: new Date().toISOString(),
    };

    const updatedTasks = [...tasks, newTask];
    setTasks(updatedTasks);
    saveTasks(updatedTasks);

    // Schedule notification
    PushNotification.localNotificationSchedule({
      id: newTask.id,
      title: 'Task Reminder',
      message: `Don't forget: ${newTask.title}`,
      date: new Date(Date.now() + minutes * 60 * 1000),
      allowWhileIdle: true,
      playSound: true,
      soundName: 'default',
      vibrate: true,
      vibration: 1000,
    });

    setTask('');
    setReminderTime('');
    alert(`Task scheduled! You'll be reminded in ${minutes} minutes.`);
  };

  const deleteTask = (taskId) => {
    const updatedTasks = tasks.filter(t => t.id !== taskId);
    setTasks(updatedTasks);
    saveTasks(updatedTasks);

    // Cancel notification
    PushNotification.cancelLocalNotifications({ id: taskId });
  };

  const renderTask = ({ item }) => (
    <View style={styles.taskItem}>
      <View style={styles.taskContent}>
        <Text style={styles.taskTitle}>{item.title}</Text>
        <Text style={styles.taskTime}>
          Reminder in {item.reminderTime} minutes
        </Text>
        <Text style={styles.taskCreated}>
          Created: {new Date(item.createdAt).toLocaleString()}
        </Text>
      </View>
      <TouchableOpacity
        style={styles.deleteButton}
        onPress={() => deleteTask(item.id)}
      >
        <Text style={styles.deleteButtonText}>Delete</Text>
      </TouchableOpacity>
    </View>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Task Reminder</Text>

      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Enter task..."
          value={task}
          onChangeText={setTask}
        />

        <TextInput
          style={styles.input}
          placeholder="Reminder time (minutes)..."
          value={reminderTime}
          onChangeText={setReminderTime}
          keyboardType="numeric"
        />

        <TouchableOpacity style={styles.addButton} onPress={addTask}>
          <Text style={styles.addButtonText}>Add Task</Text>
        </TouchableOpacity>
      </View>

      <FlatList
        data={tasks}
        renderItem={renderTask}
        keyExtractor={(item) => item.id}
        style={styles.taskList}
        ListEmptyComponent={
          <Text style={styles.emptyText}>No tasks scheduled</Text>
        }
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
  },
  inputContainer: {
    marginBottom: 20,
  },
  input: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 10,
    marginBottom: 10,
    fontSize: 16,
  },
  addButton: {
    backgroundColor: '#28a745',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
  },
  addButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  taskList: {
    flex: 1,
  },
  taskItem: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 10,
    marginBottom: 10,
    flexDirection: 'row',
    alignItems: 'center',
  },
  taskContent: {
    flex: 1,
  },
  taskTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 5,
  },
  taskTime: {
    fontSize: 14,
    color: '#007AFF',
    marginBottom: 5,
  },
  taskCreated: {
    fontSize: 12,
    color: '#666',
  },
  deleteButton: {
    backgroundColor: '#dc3545',
    padding: 10,
    borderRadius: 5,
  },
  deleteButtonText: {
    color: 'white',
    fontSize: 14,
  },
  emptyText: {
    textAlign: 'center',
    color: '#666',
    fontSize: 16,
    marginTop: 50,
  },
});

export default TaskReminderApp;
```

### **Chat App with Notifications**
```javascript
import React, { useEffect, useState } from 'react';
import { View, Text, FlatList, TextInput, TouchableOpacity, StyleSheet } from 'react-native';
import messaging from '@react-native-firebase/messaging';
import PushNotification from 'react-native-push-notification';

const ChatApp = () => {
  const [messages, setMessages] = useState([]);
  const [newMessage, setNewMessage] = useState('');
  const [fcmToken, setFcmToken] = useState('');

  useEffect(() => {
    // Get FCM token
    getFCMToken();

    // Handle foreground messages
    const unsubscribe = messaging().onMessage(async remoteMessage => {
      const { notification, data } = remoteMessage;

      // Add message to chat
      const message = {
        id: Date.now().toString(),
        text: notification.body,
        sender: data.sender || 'Unknown',
        timestamp: new Date().toISOString(),
        isMine: false,
      };

      setMessages(prev => [message, ...prev]);

      // Show notification
      PushNotification.localNotification({
        id: message.id,
        title: `New message from ${message.sender}`,
        message: message.text,
        playSound: true,
        soundName: 'default',
        vibrate: true,
        vibration: 300,
      });
    });

    return unsubscribe;
  }, []);

  const getFCMToken = async () => {
    try {
      const token = await messaging().getToken();
      setFcmToken(token);
      console.log('FCM Token:', token);
      // Send token to your server
    } catch (error) {
      console.error('Error getting FCM token:', error);
    }
  };

  const sendMessage = () => {
    if (!newMessage.trim()) return;

    const message = {
      id: Date.now().toString(),
      text: newMessage.trim(),
      sender: 'You',
      timestamp: new Date().toISOString(),
      isMine: true,
    };

    setMessages(prev => [message, ...prev]);
    setNewMessage('');

    // In a real app, send message to server
    // Server would then send push notification to other users
  };

  const renderMessage = ({ item }) => (
    <View style={[styles.messageContainer, item.isMine ? styles.myMessage : styles.theirMessage]}>
      <Text style={[styles.sender, item.isMine ? styles.mySender : styles.theirSender]}>
        {item.sender}
      </Text>
      <Text style={[styles.messageText, item.isMine ? styles.myMessageText : styles.theirMessageText]}>
        {item.text}
      </Text>
      <Text style={[styles.timestamp, item.isMine ? styles.myTimestamp : styles.theirTimestamp]}>
        {new Date(item.timestamp).toLocaleTimeString()}
      </Text>
    </View>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Chat with Notifications</Text>

      <FlatList
        data={messages}
        renderItem={renderMessage}
        keyExtractor={(item) => item.id}
        style={styles.messageList}
        inverted
        ListEmptyComponent={
          <Text style={styles.emptyText}>No messages yet</Text>
        }
      />

      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Type a message..."
          value={newMessage}
          onChangeText={setNewMessage}
          multiline
        />
        <TouchableOpacity style={styles.sendButton} onPress={sendMessage}>
          <Text style={styles.sendButtonText}>Send</Text>
        </TouchableOpacity>
      </View>

      {fcmToken && (
        <Text style={styles.tokenText} numberOfLines={1}>
          Token: {fcmToken}
        </Text>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    textAlign: 'center',
    padding: 15,
    backgroundColor: 'white',
  },
  messageList: {
    flex: 1,
    padding: 15,
  },
  messageContainer: {
    maxWidth: '80%',
    padding: 10,
    borderRadius: 10,
    marginBottom: 10,
  },
  myMessage: {
    alignSelf: 'flex-end',
    backgroundColor: '#007AFF',
  },
  theirMessage: {
    alignSelf: 'flex-start',
    backgroundColor: 'white',
  },
  sender: {
    fontSize: 12,
    fontWeight: 'bold',
    marginBottom: 5,
  },
  mySender: {
    color: 'rgba(255,255,255,0.8)',
  },
  theirSender: {
    color: '#666',
  },
  messageText: {
    fontSize: 16,
    marginBottom: 5,
  },
  myMessageText: {
    color: 'white',
  },
  theirMessageText: {
    color: '#333',
  },
  timestamp: {
    fontSize: 10,
  },
  myTimestamp: {
    color: 'rgba(255,255,255,0.6)',
    alignSelf: 'flex-end',
  },
  theirTimestamp: {
    color: '#999',
  },
  inputContainer: {
    flexDirection: 'row',
    padding: 15,
    backgroundColor: 'white',
    alignItems: 'flex-end',
  },
  input: {
    flex: 1,
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 20,
    padding: 10,
    marginRight: 10,
    maxHeight: 100,
  },
  sendButton: {
    backgroundColor: '#007AFF',
    padding: 10,
    borderRadius: 20,
    minWidth: 60,
    alignItems: 'center',
  },
  sendButtonText: {
    color: 'white',
    fontWeight: 'bold',
  },
  emptyText: {
    textAlign: 'center',
    color: '#666',
    fontSize: 16,
    marginTop: 50,
  },
  tokenText: {
    fontSize: 10,
    color: '#666',
    padding: 5,
    backgroundColor: 'white',
    textAlign: 'center',
  },
});

export default ChatApp;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Local Notifications**: Scheduling and displaying local notifications
- ✅ **Push Notifications**: Firebase Cloud Messaging integration
- ✅ **Permission Management**: Requesting and handling notification permissions
- ✅ **Rich Notifications**: Images, actions, and expandable content
- ✅ **Background Processing**: Handling notifications when app is backgrounded
- ✅ **Interactive Notifications**: Action buttons and quick replies
- ✅ **Testing & Debugging**: Tools and techniques for notification debugging

### **Best Practices**
1. **Always request permissions** before sending notifications
2. **Handle different app states** (foreground, background, killed)
3. **Use appropriate notification types** for different use cases
4. **Test on real devices** for accurate notification behavior
5. **Provide user controls** to manage notification preferences
6. **Handle notification data securely** and validate inputs
7. **Optimize battery usage** with efficient notification scheduling

### **Next Steps**
- Implement advanced notification features like grouping and channels
- Add notification analytics and tracking
- Create custom notification sounds and vibration patterns
- Implement notification scheduling with calendar integration

---

## 🎯 **Assignment**

### **Task 1: Reminder App**
Create a comprehensive reminder app with:
- Multiple reminder types (one-time, recurring, location-based)
- Custom notification sounds and vibration patterns
- Reminder categories and priorities
- Snooze functionality
- Notification history and statistics

### **Task 2: Social Media Notifications**
Build a social media notification system that:
- Handles different notification types (likes, comments, follows)
- Groups similar notifications
- Shows rich content previews
- Allows quick actions (like, reply, follow back)
- Manages notification preferences per user

### **Task 3: Emergency Alert System**
Implement an emergency alert system with:
- High-priority notifications that bypass Do Not Disturb
- Location-based emergency alerts
- Emergency contact notifications
- Silent alerts for specific situations
- Notification escalation for critical alerts

---

## 📚 **Additional Resources**
- [React Native Push Notifications](https://github.com/zo0r/react-native-push-notification)
- [Firebase Cloud Messaging Documentation](https://firebase.google.com/docs/cloud-messaging)
- [Apple Push Notification Service](https://developer.apple.com/notifications/)
- [Android Notification Channels](https://developer.android.com/guide/topics/ui/notifiers/notification-channels)
- [Notification Best Practices](https://material.io/design/communication/notifications.html)

---

**Next Lesson**: [Lesson 18: Offline Support & Data Synchronization](Lesson%2018_%20Offline%20Support%20&%20Data%20Synchronization.md)