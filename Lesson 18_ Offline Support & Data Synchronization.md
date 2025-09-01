esson 18_ Offline Support & Data Synchronization.md</path>
<content">- Create sync dashboards for monitoring

---

## 📝 **Assignment 18: Offline Support & Data Synchronization**

### **Task 1: Offline Task Manager (90 minutes)**
Create a task manager with offline support:
- Local storage with AsyncStorage
- Offline queue for operations
- Sync status indicators
- Conflict resolution for task updates
- Background sync when online

### **Task 2: Offline Shopping Cart (120 minutes)**
Build a shopping cart with offline capabilities:
- Product catalog with local storage
- Add/remove items offline
- Cart sync when online
- Optimistic updates for cart operations
- Network status monitoring

### **Task 3: Offline Chat Application (150 minutes)**
Implement a chat app with offline messaging:
- Message storage in SQLite
- Offline message queue
- Message sync when online
- Read receipts and delivery status
- Background message sync

---

## 🧠 **Quiz 18: Offline Support & Data Synchronization**

### **Multiple Choice Questions (1-15)**

**Q1. Offline-first architecture mein kya important hai?**
a) Fast internet connection
b) Local data storage
c) Server-only operations
d) No local storage

**Q2. AsyncStorage ka use kab hota hai?**
a) Complex database operations
b) Simple key-value storage
c) Real-time data sync
d) Large file storage

**Q3. Conflict resolution mein Last Write Wins ka matlab kya hai?**
a) First change wins
b) Latest timestamp wins
c) Random selection
d) Manual resolution

**Q4. Optimistic updates ka benefit kya hai?**
a) Slow UI response
b) Immediate UI feedback
c) Network dependency
d) Data loss

**Q5. Background sync ka use kab hota hai?**
a) App is in foreground
b) App is in background
c) User is interacting
d) Network is slow

### **True/False Questions (16-20)**

**Q16. SQLite complex data ke liye better hai AsyncStorage se.** (True/False)

**Q17. Offline queue operations ko immediately execute karna chahiye.** (True/False)

**Q18. Conflict resolution automatic hona chahiye user input ke bina.** (True/False)

**Q19. Background sync battery life ko affect karta hai.** (True/False)

**Q20. Offline-first apps mein network optional hota hai.** (True/False)

### **Code Output Questions (21-25)**

**Q21. Is code ka output kya hoga?**
```javascript
const syncManager = new SyncManager();
syncManager.addToSyncQueue({
  type: 'CREATE_POST',
  data: { title: 'Test' }
});
```

**Q22. Is offline queue mein problem kya hai?**
```javascript
const queue = [];
const addOperation = (op) => {
  queue.push(op);
  // No persistence
};
```

**Q23. Is conflict resolution mein kya galat hai?**
```javascript
const resolve = (local, server) => {
  return local; // Always prefer local
};
```

**Q24. Is optimistic update mein kya missing hai?**
```javascript
const addItem = async (item) => {
  // Update UI immediately
  setItems([...items, item]);
  // No error handling
};
```

**Q25. Is background sync setup mein kya problem hai?**
```javascript
// Android WorkManager
PeriodicWorkRequest request = new PeriodicWorkRequest.Builder(
  SyncWorker.class,
  1, // 1 minute - too frequent
  TimeUnit.MINUTES
).build();
```

---

## 📚 **Answers & Explanations**

### **Multiple Choice Answers**
**Q1.** b) Local data storage  
**Q2.** b) Simple key-value storage  
**Q3.** b) Latest timestamp wins  
**Q4.** b) Immediate UI feedback  
**Q5.** b) App is in background  

### **True/False Answers**
**Q16.** True - SQLite complex data ke liye better hai AsyncStorage se  
**Q17.** False - Offline queue operations ko batch mein execute karna chahiye  
**Q18.** False - Conflict resolution user input ke saath better hota hai  
**Q19.** True - Background sync battery life ko affect karta hai  
**Q20.** True - Offline-first apps mein network optional hota hai  

### **Code Output Answers**
**Q21.** Operation sync queue mein add hoga  
**Q22.** Operations persist nahi honge app restart ke baad  
**Q23.** Server changes ignore ho jayenge  
**Q24.** Network failure ke case mein rollback missing hai  
**Q25.** Battery drain ka chance jyada hai  

---

## 🎯 **Assignment Solutions**

### **Task 1: Offline Task Manager Solution**

```javascript
// TaskManager.js
import React, { useState, useEffect } from 'react';
import { View, Text, TextInput, TouchableOpacity, FlatList, StyleSheet, Alert } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';
import NetInfo from '@react-native-community/netinfo';

const OfflineTaskManager = () => {
  const [tasks, setTasks] = useState([]);
  const [newTask, setNewTask] = useState('');
  const [isOnline, setIsOnline] = useState(true);
  const [syncStatus, setSyncStatus] = useState('idle');

  useEffect(() => {
    loadTasks();
    setupNetworkListener();
    loadSyncQueue();
  }, []);

  const setupNetworkListener = () => {
    const unsubscribe = NetInfo.addEventListener(state => {
      const wasOffline = !isOnline && state.isConnected;
      setIsOnline(state.isConnected);

      if (wasOffline && state.isConnected) {
        syncAllTasks();
      }
    });
    return unsubscribe;
  };

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
      setTasks(updatedTasks);
    } catch (error) {
      console.error('Error saving tasks:', error);
    }
  };

  const addTask = async () => {
    if (!newTask.trim()) return;

    const task = {
      id: Date.now().toString(),
      title: newTask.trim(),
      completed: false,
      createdAt: new Date().toISOString(),
      synced: false,
      pendingOperation: 'create'
    };

    const updatedTasks = [...tasks, task];
    await saveTasks(updatedTasks);
    setNewTask('');

    if (isOnline) {
      syncTask(task);
    } else {
      addToSyncQueue(task);
    }
  };

  const toggleTask = async (taskId) => {
    const updatedTasks = tasks.map(task =>
      task.id === taskId
        ? { ...task, completed: !task.completed, synced: false, pendingOperation: 'update' }
        : task
    );

    await saveTasks(updatedTasks);

    const task = updatedTasks.find(t => t.id === taskId);
    if (isOnline) {
      syncTask(task);
    } else {
      addToSyncQueue(task);
    }
  };

  const addToSyncQueue = async (task) => {
    try {
      const queue = await AsyncStorage.getItem('sync_queue') || '[]';
      const syncQueue = JSON.parse(queue);
      syncQueue.push({
        id: task.id,
        operation: task.pendingOperation,
        data: task,
        timestamp: new Date().toISOString()
      });
      await AsyncStorage.setItem('sync_queue', JSON.stringify(syncQueue));
    } catch (error) {
      console.error('Error adding to sync queue:', error);
    }
  };

  const syncTask = async (task) => {
    try {
      setSyncStatus('syncing');

      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 1000));

      // Mark as synced
      const updatedTasks = tasks.map(t =>
        t.id === task.id ? { ...t, synced: true, pendingOperation: null } : t
      );
      await saveTasks(updatedTasks);

      setSyncStatus('success');
      setTimeout(() => setSyncStatus('idle'), 2000);
    } catch (error) {
      console.error('Sync failed:', error);
      setSyncStatus('error');
      setTimeout(() => setSyncStatus('idle'), 2000);
    }
  };

  const syncAllTasks = async () => {
    try {
      const queue = await AsyncStorage.getItem('sync_queue') || '[]';
      const syncQueue = JSON.parse(queue);

      if (syncQueue.length === 0) return;

      setSyncStatus('syncing');

      for (const item of syncQueue) {
        await syncTask(item.data);
      }

      // Clear sync queue
      await AsyncStorage.removeItem('sync_queue');
      setSyncStatus('success');
      setTimeout(() => setSyncStatus('idle'), 2000);
    } catch (error) {
      console.error('Bulk sync failed:', error);
      setSyncStatus('error');
      setTimeout(() => setSyncStatus('idle'), 2000);
    }
  };

  const loadSyncQueue = async () => {
    try {
      const queue = await AsyncStorage.getItem('sync_queue') || '[]';
      const syncQueue = JSON.parse(queue);

      if (syncQueue.length > 0) {
        setSyncStatus('pending');
      }
    } catch (error) {
      console.error('Error loading sync queue:', error);
    }
  };

  const renderTask = ({ item }) => (
    <TouchableOpacity
      style={[styles.taskItem, item.completed && styles.completedTask]}
      onPress={() => toggleTask(item.id)}
    >
      <View style={styles.taskContent}>
        <Text style={[styles.taskText, item.completed && styles.completedText]}>
          {item.title}
        </Text>
        <View style={styles.taskMeta}>
          {!item.synced && (
            <Text style={styles.syncIndicator}>
              {item.pendingOperation === 'create' ? 'Creating...' : 'Updating...'}
            </Text>
          )}
          <Text style={styles.timestamp}>
            {new Date(item.createdAt).toLocaleDateString()}
          </Text>
        </View>
      </View>
      <View style={[styles.statusIndicator, item.completed && styles.completedIndicator]} />
    </TouchableOpacity>
  );

  const getSyncStatusColor = () => {
    switch (syncStatus) {
      case 'syncing': return '#ffc107';
      case 'success': return '#28a745';
      case 'error': return '#dc3545';
      case 'pending': return '#17a2b8';
      default: return '#6c757d';
    }
  };

  const getSyncStatusText = () => {
    switch (syncStatus) {
      case 'syncing': return 'Syncing...';
      case 'success': return 'Synced';
      case 'error': return 'Sync Failed';
      case 'pending': return 'Pending Sync';
      default: return 'Idle';
    }
  };

  return (
    <View style={styles.container}>
      <View style={[styles.statusBar, { backgroundColor: isOnline ? '#28a745' : '#dc3545' }]}>
        <Text style={styles.statusText}>
          {isOnline ? 'Online' : 'Offline'}
        </Text>
        <View style={[styles.syncStatus, { backgroundColor: getSyncStatusColor() }]}>
          <Text style={styles.syncStatusText}>{getSyncStatusText()}</Text>
        </View>
      </View>

      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Add new task..."
          value={newTask}
          onChangeText={setNewTask}
        />
        <TouchableOpacity style={styles.addButton} onPress={addTask}>
          <Text style={styles.addButtonText}>Add</Text>
        </TouchableOpacity>
      </View>

      {!isOnline && syncStatus === 'pending' && (
        <TouchableOpacity style={styles.syncButton} onPress={syncAllTasks}>
          <Text style={styles.syncButtonText}>Sync Pending Tasks</Text>
        </TouchableOpacity>
      )}

      <FlatList
        data={tasks}
        renderItem={renderTask}
        keyExtractor={(item) => item.id}
        style={styles.taskList}
        ListEmptyComponent={
          <Text style={styles.emptyText}>No tasks yet</Text>
        }
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  statusBar: {
    padding: 10,
    alignItems: 'center',
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
  statusText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  syncStatus: {
    paddingHorizontal: 10,
    paddingVertical: 5,
    borderRadius: 15,
  },
  syncStatusText: {
    color: 'white',
    fontSize: 12,
    fontWeight: 'bold',
  },
  inputContainer: {
    flexDirection: 'row',
    padding: 20,
  },
  input: {
    flex: 1,
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 10,
    marginRight: 10,
  },
  addButton: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    justifyContent: 'center',
  },
  addButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  syncButton: {
    backgroundColor: '#17a2b8',
    marginHorizontal: 20,
    marginBottom: 10,
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
  },
  syncButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  taskList: {
    flex: 1,
    padding: 20,
  },
  taskItem: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 10,
    marginBottom: 10,
    flexDirection: 'row',
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  completedTask: {
    backgroundColor: '#f8f9fa',
  },
  taskContent: {
    flex: 1,
  },
  taskText: {
    fontSize: 16,
    marginBottom: 5,
  },
  completedText: {
    textDecorationLine: 'line-through',
    color: '#6c757d',
  },
  taskMeta: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  },
  syncIndicator: {
    fontSize: 12,
    color: '#ffc107',
    fontStyle: 'italic',
  },
  timestamp: {
    fontSize: 12,
    color: '#999',
  },
  statusIndicator: {
    width: 20,
    height: 20,
    borderRadius: 10,
    backgroundColor: '#ddd',
    marginLeft: 10,
  },
  completedIndicator: {
    backgroundColor: '#28a745',
  },
  emptyText: {
    textAlign: 'center',
    color: '#666',
    fontSize: 16,
    marginTop: 50,
  },
});

export default OfflineTaskManager;
```

---

## 📚 **Additional Resources**
- [AsyncStorage Documentation](https://react-native-async-storage.github.io/async-storage/)
- [SQLite Documentation](https://github.com/andpor/react-native-sqlite-storage)
- [Realm Documentation](https://docs.mongodb.com/realm/sdk/react-native/)
- [NetInfo Documentation](https://github.com/react-native-netinfo/react-native-netinfo)
- [Offline-First Best Practices](https://offlinefirst.org/)

---

**Next Lesson**: [Lesson 19: Testing & Quality Assurance](Lesson%2019_%20Testing%20&%20Quality%20Assurance.md)