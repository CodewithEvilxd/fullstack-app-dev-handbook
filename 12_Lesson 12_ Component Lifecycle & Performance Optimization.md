
# ⚡ Lesson 12: Component Lifecycle & Performance Optimization

## 🎯 Learning Objectives
Is lesson ke baad aap:
- React Native component lifecycle samjhenge
- Performance bottlenecks identify kar sakte hain
- Optimization techniques implement kar sakte hain
- Memory management kar sakte hain
- Production-ready performance achieve kar sakte hain

---

## 📖 Component Lifecycle in React Native

React Native mein functional components useEffect hook ke through lifecycle events handle karte hain.

### 1. Lifecycle Phases

```javascript
import React, { useState, useEffect, useRef } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  AppState,
  Alert
} from 'react-native';

const LifecycleExample = () => {
  const [count, setCount] = useState(0);
  const [data, setData] = useState(null);
  const [appState, setAppState] = useState(AppState.currentState);
  const mountTime = useRef(Date.now());

  // Component Did Mount (runs once after first render)
  useEffect(() => {
    console.log('🟢 Component Did Mount');
    
    // Setup subscriptions, timers, etc.
    const subscription = AppState.addEventListener('change', handleAppStateChange);
    
    // Fetch initial data
    fetchInitialData();
    
    // Cleanup function (Component Will Unmount)
    return () => {
      console.log('🔴 Component Will Unmount');
      subscription.remove();
      // Cleanup subscriptions, timers, etc.
    };
  }, []); // Empty dependency array = runs once

  // Component Did Update (runs after every render except first)
  useEffect(() => {
    if (mountTime.current !== Date.now()) {
      console.log('🟡 Component Did Update');
    }
  }); // No dependency array = runs after every render

  // Specific state change (runs when count changes)
  useEffect(() => {
    console.log('🔵 Count changed:', count);
    
    // Side effect based on count
    if (count > 0) {
      console.log(`Count is now ${count}`);
    }
    
    if (count >= 10) {
      Alert.alert('Milestone!', 'You reached 10!');
    }
  }, [count]); // Dependency array with count

  // App state change handler
  const handleAppStateChange = (nextAppState) => {
    console.log('App state changed:', appState, '->', nextAppState);
    
    if (appState.match(/inactive|background/) && nextAppState === 'active') {
      console.log('App has come to the foreground!');
      // Refresh data when app becomes active
      fetchInitialData();
    }
    
    setAppState(nextAppState);
  };

  // Fetch data function
  const fetchInitialData = async () => {
    try {
      console.log('Fetching initial data...');
      const response = await fetch('https://jsonplaceholder.typicode.com/posts/1');
      const result = await response.json();
      setData(result);
      console.log('Data fetched successfully');
    } catch (error) {
      console.error('Failed to fetch data:', error);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Component Lifecycle</Text>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Count: {count}</Text>
        <View style={styles.buttonRow}>
          <TouchableOpacity
            style={styles.button}
            onPress={() => setCount(count + 1)}
          >
            <Text style={styles.buttonText}>Increment</Text>
          </TouchableOpacity>
          
          <TouchableOpacity
            style={[styles.button, styles.resetButton]}
            onPress={() => setCount(0)}
          >
            <Text style={styles.buttonText}>Reset</Text>
          </TouchableOpacity>
        </View>
      </View>

      <View style={styles.section}>
        <Text style={styles.sectionTitle}>App State: {appState}</Text>
        <Text style={styles.description}>
          Try switching between apps to see app state changes
        </Text>
      </View>

      {data && (
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Fetched Data:</Text>
          <Text style={styles.dataTitle}>{data.title}</Text>
          <Text style={styles.dataBody} numberOfLines={3}>
            {data.body}
          </Text>
        </View>
      )}

      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Lifecycle Events</Text>
        <Text style={styles.description}>
          Check console for lifecycle event logs:
          {'\n'}🟢 Mount
          {'\n'}🟡 Update
          {'\n'}🔵 Count Change
          {'\n'}🔴 Unmount
        </Text>
      </View>
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
    color: '#333',
  },
  section: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 12,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  description: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
  },
  buttonRow: {
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 20,
    paddingVertical: 12,
    borderRadius: 8,
    minWidth: 100,
    alignItems: 'center',
  },
  resetButton: {
    backgroundColor: '#FF3B30',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  dataTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 8,
    color: '#333',
  },
  dataBody: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
  },
});

export default LifecycleExample;
```

---

## 🚀 Performance Optimization Techniques

### 1. React.memo for Component Optimization

```javascript
import React, { useState, useMemo, useCallback } from 'react';
import {
  View,
  Text,
  FlatList,
  TouchableOpacity,
  TextInput,
  StyleSheet
} from 'react-native';

// ❌ Component without optimization
const SlowListItem = ({ item, onPress }) => {
  console.log('Rendering item:', item.id);
  
  // Expensive calculation on every render
  const expensiveValue = (() => {
    let result = 0;
    for (let i = 0; i < 1000; i++) {
      result += Math.random();
    }
    return result.toFixed(2);
  })();
  
  return (
    <TouchableOpacity style={styles.listItem} onPress={() => onPress(item.id)}>
      <Text style={styles.itemTitle}>{item.title}</Text>
      <Text style={styles.itemSubtitle}>Expensive calc: {expensiveValue}</Text>
    </TouchableOpacity>
  );
};

// ✅ Optimized component with React.memo
const OptimizedListItem = React.memo(({ item, onPress }) => {
  console.log('Rendering optimized item:', item.id);
  
  // Memoized expensive calculation
  const expensiveValue = useMemo(() => {
    let result = 0;
    for (let i = 0; i < 1000; i++) {
      result += Math.random();
    }
    return result.toFixed(2);
  }, [item.id]); // Only recalculate when item.id changes
  
  return (
    <TouchableOpacity style={styles.listItem} onPress={() => onPress(item.id)}>
      <Text style={styles.itemTitle}>{item.title}</Text>
      <Text style={styles.itemSubtitle}>Expensive calc: {expensiveValue}</Text>
    </TouchableOpacity>
  );
}, (prevProps, nextProps) => {
  // Custom comparison function
  return prevProps.item.id === nextProps.item.id &&
         prevProps.item.title === nextProps.item.title;
});

const PerformanceOptimizationExample = () => {
  const [data, setData] = useState([]);
  const [filter, setFilter] = useState('');
  const [useOptimized, setUseOptimized] = useState(true);
  const [renderCount, setRenderCount] = useState(0);

  // Generate sample data
  const generateData = useCallback(() => {
    const newData = Array.from({ length: 100 }, (_, i) => ({
      id: i + 1,
      title: `Item ${i + 1}`,
      subtitle: `Description for item ${i + 1}`,
      value: Math.random() * 100
    }));
    setData(newData);
  }, []);

  // Memoized filtered data
  const filteredData = useMemo(() => {
    console.log('Filtering data...');
    return data.filter(item =>
      item.title.toLowerCase().includes(filter.toLowerCase())
    );
  }, [data, filter]);

  // Memoized callback to prevent unnecessary re-renders
  const handleItemPress = useCallback((itemId) => {
    console.log('Item pressed:', itemId);
  }, []);

  // Memoized render function
  const renderItem = useCallback(({ item }) => {
    const ItemComponent = useOptimized ? OptimizedListItem : SlowListItem;
    return <ItemComponent item={item} onPress={handleItemPress} />;
  }, [useOptimized, handleItemPress]);

  // Track renders
  useEffect(() => {
    setRenderCount(prev => prev + 1);
  });

  useEffect(() => {
    generateData();
  }, [generateData]);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Performance Optimization</Text>
      
      <View style={styles.controls}>
        <Text style={styles.renderCount}>Renders: {renderCount}</Text>
        
        <TouchableOpacity
          style={styles.toggleButton}
          onPress={() => setUseOptimized(!useOptimized)}
        >
          <Text style={styles.toggleButtonText}>
            {useOptimized ? 'Use Slow' : 'Use Optimized'} Components
          </Text>
        </TouchableOpacity>
      </View>

      <TextInput
        style={styles.searchInput}
        placeholder="Search items..."
        value={filter}
        onChangeText={setFilter}
      />

      <Text style={styles.itemCount}>
        Showing {filteredData.length} of {data.length} items
      </Text>

      <FlatList
        data={filteredData}
        renderItem={renderItem}
        keyExtractor={item => item.id.toString()}
        style={styles.list}
        // Performance optimizations
        removeClippedSubviews={true}
        maxToRenderPerBatch={10}
        windowSize={10}
        initialNumToRender={10}
        getItemLayout={(data, index) => ({
          length: 80,
          offset: 80 * index,
          index,
        })}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    padding: 20,
    backgroundColor: 'white',
    color: '#333',
  },
  controls: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    backgroundColor: 'white',
    paddingHorizontal: 20,
    paddingVertical: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  renderCount: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#007AFF',
  },
  toggleButton: {
    backgroundColor: '#28A745',
    paddingHorizontal: 15,
    paddingVertical: 8,
    borderRadius: 6,
  },
  toggleButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
  searchInput: {
    backgroundColor: 'white',
    margin: 15,
    paddingHorizontal: 15,
    paddingVertical: 12,
    borderRadius: 8,
    fontSize: 16,
    borderWidth: 1,
    borderColor: '#ddd',
  },
  itemCount: {
    fontSize: 14,
    color: '#666',
    marginHorizontal: 15,
    marginBottom: 10,
  },
  list: {
    flex: 1,
  },
  listItem: {
    backgroundColor: 'white',
    padding: 15,
    marginHorizontal: 15,
    marginVertical: 5,
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.2,
    shadowRadius: 2,
    elevation: 3,
  },
  itemTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 5,
  },
  itemSubtitle: {
    fontSize: 14,
    color: '#666',
  },
});

export default PerformanceOptimizationExample;
```

### 2. useMemo and useCallback Optimization

```javascript
import React, { useState, useMemo, useCallback, useEffect } from 'react';
import {
  View,
  Text,
  FlatList,
  TouchableOpacity,
  TextInput,
  StyleSheet
} from 'react-native';

const MemoCallbackExample = () => {
  const [users, setUsers] = useState([]);
  const [searchTerm, setSearchTerm] = useState('');
  const [sortBy, setSortBy] = useState('name');
  const [sortOrder, setSortOrder] = useState('asc');

  // Generate sample users
  useEffect(() => {
    const sampleUsers = Array.from({ length: 1000 }, (_, i) => ({
      id: i + 1,
      name: `User ${i + 1}`,
      email: `user${i + 1}@example.com`,
      age: Math.floor(Math.random() * 50) + 18,
      city: ['Mumbai', 'Delhi', 'Bangalore', 'Chennai', 'Kolkata'][Math.floor(Math.random() * 5)],
      salary: Math.floor(Math.random() * 100000) + 30000
    }));
    setUsers(sampleUsers);
  }, []);

  // ❌ Without useMemo - expensive calculation on every render
  const expensiveCalculationBad = () => {
    console.log('🔴 Expensive calculation running...');
    return users.reduce((sum, user) => sum + user.salary, 0);
  };

  // ✅ With useMemo - calculation only when users change
  const totalSalary = useMemo(() => {
    console.log('🟢 Memoized calculation running...');
    return users.reduce((sum, user) => sum + user.salary, 0);
  }, [users]);

  // ✅ Memoized filtered and sorted data
  const processedUsers = useMemo(() => {
    console.log('🟡 Processing users...');
    
    let filtered = users.filter(user =>
      user.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
      user.email.toLowerCase().includes(searchTerm.toLowerCase()) ||
      user.city.toLowerCase().includes(searchTerm.toLowerCase())
    );

    filtered.sort((a, b) => {
      let aValue = a[sortBy];
      let bValue = b[sortBy];
      
      if (typeof aValue === 'string') {
        aValue = aValue.toLowerCase();
        bValue = bValue.toLowerCase();
      }
      
      if (sortOrder === 'asc') {
        return aValue > bValue ? 1 : -1;
      } else {
        return aValue < bValue ? 1 : -1;
      }
    });

    return filtered;
  }, [users, searchTerm, sortBy, sortOrder]);

  // ❌ Without useCallback - new function on every render
  const handleUserPressBad = (userId) => {
    console.log('User pressed:', userId);
  };

  // ✅ With useCallback - function reference stays same
  const handleUserPress = useCallback((userId) => {
    console.log('User pressed:', userId);
  }, []);

  // ✅ Memoized render function
  const renderUser = useCallback(({ item }) => (
    <UserItem user={item} onPress={handleUserPress} />
  ), [handleUserPress]);

  const toggleSortOrder = () => {
    setSortOrder(prev => prev === 'asc' ? 'desc' : 'asc');
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>useMemo & useCallback</Text>
      
      {/* Stats */}
      <View style={styles.statsContainer}>
        <Text style={styles.statsText}>Total Users: {users.length}</Text>
        <Text style={styles.statsText}>Filtered: {processedUsers.length}</Text>
        <Text style={styles.statsText}>Total Salary: ₹{totalSalary.toLocaleString()}</Text>
      </View>

      {/* Search */}
      <TextInput
        style={styles.searchInput}
        placeholder="Search users..."
        value={searchTerm}
        onChangeText={setSearchTerm}
      />

      {/* Sort controls */}
      <View style={styles.sortControls}>
        <Text style={styles.sortLabel}>Sort by:</Text>
        {['name', 'age', 'salary', 'city'].map(field => (
          <TouchableOpacity
            key={field}
            style={[
              styles.sortButton,
              sortBy === field && styles.sortButtonActive
            ]}
            onPress={() => setSortBy(field)}
          >
            <Text style={[
              styles.sortButtonText,
              sortBy === field && styles.sortButtonTextActive
            ]}>
              {field}
            </Text>
          </TouchableOpacity>
        ))}
        
        <TouchableOpacity style={styles.orderButton} onPress={toggleSortOrder}>
          <Text style={styles.orderButtonText}>
            {sortOrder === 'asc' ? '↑' : '↓'}
          </Text>
        </TouchableOpacity>
      </View>

      {/* User list */}
      <FlatList
        data={processedUsers}
        renderItem={renderUser}
        keyExtractor={item => item.id.toString()}
        style={styles.userList}
        showsVerticalScrollIndicator={false}
        // Performance optimizations
        removeClippedSubviews={true}
        maxToRenderPerBatch={20}
        windowSize={10}
        initialNumToRender={15}
      />
    </View>
  );
};

// Memoized user item component
const UserItem = React.memo(({ user, onPress }) => {
  console.log('Rendering user item:', user.id);
  
  return (
    <TouchableOpacity
      style={styles.userItem}
      onPress={() => onPress(user.id)}
    >
      <View style={styles.userInfo}>
        <Text style={styles.userName}>{user.name}</Text>
        <Text style={styles.userEmail}>{user.email}</Text>
        <Text style={styles.userDetails}>
          {user.age} years • {user.city} • ₹{user.salary.toLocaleString()}
        </Text>
      </View>
    </TouchableOpacity>
  );
});

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    padding: 20,
    backgroundColor: 'white',
    color: '#333',
  },
  statsContainer: {
    backgroundColor: 'white',
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  statsText: {
    fontSize: 14,
    color: '#666',
    marginBottom: 5,
  },
  searchInput: {
    backgroundColor: 'white',
    margin: 15,
    paddingHorizontal: 15,
    paddingVertical: 12,
    borderRadius: 8,
    fontSize: 16,
    borderWidth: 1,
    borderColor: '#ddd',
  },
  sortControls: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: 'white',
    paddingHorizontal: 15,
    paddingVertical: 10,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  sortLabel: {
    fontSize: 14,
    fontWeight: 'bold',
    marginRight: 10,
    color: '#333',
  },
  sortButton: {
    backgroundColor: '#f8f9fa',
    paddingHorizontal: 12,
    paddingVertical: 6,
    borderRadius: 4,
    marginRight: 8,
  },
  sortButtonActive: {
    backgroundColor: '#007AFF',
  },
  sortButtonText: {
    fontSize: 12,
    color: '#666',
  },
  sortButtonTextActive: {
    color: 'white',
    fontWeight: 'bold',
  },
  orderButton: {
    backgroundColor: '#28A745',
    paddingHorizontal: 10,
    paddingVertical: 6,
    borderRadius: 4,
    marginLeft: 10,
  },
  orderButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  userList: {
    flex: 1,
  },
  userItem: {
    backgroundColor: 'white',
    marginHorizontal: 15,
    marginVertical: 5,
    padding: 15,
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.2,
    shadowRadius: 2,
    elevation: 3,
  },
  userInfo: {
    flex: 1,
  },
  userName: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 3,
  },
  userEmail: {
    fontSize: 14,
    color: '#007AFF',
    marginBottom: 3,
  },
  userDetails: {
    fontSize: 12,
    color: '#666',
  },
});

export default MemoCallbackExample;
```

---

## 🧠 Memory Management

### 1. Memory Leak Prevention

```javascript
import React, { useState, useEffect, useRef } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  Alert,
  AppState
} from 'react-native';

const MemoryManagementExample = () => {
  const [data, setData] = useState([]);
  const [isSubscribed, setIsSubscribed] = useState(false);
  const intervalRef = useRef(null);
  const timeoutRef = useRef(null);
  const subscriptionRef = useRef(null);
  const isMountedRef = useRef(true);

  // ✅ Proper cleanup of intervals
  useEffect(() => {
    if (isSubscribed) {
      intervalRef.current = setInterval(() => {
        // Only update state if component is still mounted
        if (isMountedRef.current) {
          setData(prevData => [
            ...prevData,
            {
              id: Date.now(),
              timestamp: new Date().toLocaleTimeString(),
              value: Math.random()
            }
          ]);
        }
      }, 1000);
    } else {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
        intervalRef.current = null;
      }
    }

    // Cleanup function
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, [isSubscribed]);

  // ✅ Proper cleanup of timeouts
  useEffect(() => {
    timeoutRef.current = setTimeout(() => {
      if (isMountedRef.current) {
        console.log('Delayed operation completed');
      }
    }, 5000);

    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, []);

  // ✅ Proper cleanup of event listeners
  useEffect(() => {
    const handleAppStateChange = (nextAppState) => {
      if (isMountedRef.current) {
        console.log('App state changed:', nextAppState);
      }
    };

    subscriptionRef.current = AppState.addEventListener('change', handleAppStateChange);

    return () => {
      if (subscriptionRef.current) {
        subscriptionRef.current.remove();
      }
    };
  }, []);

  // ✅ Component unmount tracking
  useEffect(() => {
    return () => {
      isMountedRef.current = false;
    };
  }, []);

  // ✅ Async operation with cleanup
  const fetchDataSafely = async () => {
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts');
      
      // Check if component is still mounted before updating state
      if (isMountedRef.current) {
        const result = await response.json();
        setData(result.slice(0, 10)); // Limit data to prevent memory issues
      }
    } catch (error) {
      if (isMountedRef.current) {
        console.error('Fetch error:', error);
        Alert.alert('Error', 'Failed to fetch data');
      }
    }
  };

  const toggleSubscription = () => {
    setIsSubscribed(!isSubscribed);
  };

  const clearData = () => {
    setData([]);
  };

  const showMemoryInfo = () => {
    const memoryInfo = {
      dataLength: data.length,
      intervalActive: !!intervalRef.current,
      timeoutActive: !!timeoutRef.current,
      subscriptionActive: !!subscriptionRef.current,
      componentMounted: isMountedRef.current
    };
    
    Alert.alert('Memory Info', JSON.stringify(memoryInfo, null, 2));
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Memory Management</Text>
      
      <View style={styles.controls}>
        <TouchableOpacity
          style={[
            styles.button,
            isSubscribed ? styles.stopButton : styles.startButton
          ]}
          onPress={toggleSubscription}
        >
          <Text style={styles.buttonText}>
            {isSubscribed ? 'Stop' : 'Start'} Data Stream
          </Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.button} onPress={fetchDataSafely}>
          <Text style={styles.buttonText}>Fetch Data</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={[styles.button, styles.clearButton]} onPress={clearData}>
          <Text style={styles.buttonText}>Clear Data</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.infoButton} onPress={showMemoryInfo}>
          <Text style={styles.buttonText}>Memory Info</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.dataInfo}>
        <Text style={styles.dataInfoText}>
          Data items: {data.length}
        </Text>
        <Text style={styles.dataInfoText}>
          Stream: {isSubscribed ? 'Active' : 'Inactive'}
        </Text>
      </View>

      <FlatList
        data={data.slice(-20)} // Show only last 20 items to prevent memory issues
        renderItem={({ item }) => (
          <View style={styles.dataItem}>
            <Text style={styles.dataItemText}>
              {item.timestamp || item.title} - {item.value?.toFixed(4) || 'API Data'}
            </Text>
          </View>
        )}
        keyExtractor={item => item.id.toString()}
        style={styles.dataList}
        showsVerticalScrollIndicator={false}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    padding: 20,
    backgroundColor: 'white',
    color: '#333',
  },
  controls: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-around',
    backgroundColor: 'white',
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 15,
    paddingVertical: 10,
    borderRadius: 6,
    marginBottom: 10,
    minWidth: 100,
    alignItems: 'center',
  },
  startButton: {
    backgroundColor: '#28A745',
  },
  stopButton: {
    backgroundColor: '#FF3B30',
  },
  clearButton: {
    backgroundColor: '#6C757D',
  },
  infoButton: {
    backgroundColor: '#17A2B8',
  },
  buttonText: {
    color: 'white',
    fontSize: 12,
    fontWeight: 'bold',
  },
  dataInfo: {
    backgroundColor: 'white',
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  dataInfoText: {
    fontSize: 14,
    color: '#666',
    marginBottom: 3,
  },
  dataList: {
    flex: 1,
  },
  dataItem: {
    backgroundColor: 'white',
    padding: 12,
    marginHorizontal: 15,
    marginVertical: 2,
    borderRadius: 6,
  },
  dataItemText: {
    fontSize: 14,
    color: '#333',
  },
});

export default MemoryManagementExample;
```

---

## 📊 Performance Monitoring

### 1. Performance Metrics

```javascript
import React, { useState, useEffect, useRef } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  ScrollView
} from 'react-native';

const PerformanceMonitor = () => {
  const [metrics, setMetrics] = useState({
    renderCount: 0,
    lastRenderTime: 0,
    averageRenderTime: 0,
    totalRenderTime: 0
  });
  
  const renderStartTime = useRef(Date.now());
  const renderTimes = useRef([]);

  // Monitor render performance
  useEffect(() => {
    const renderEndTime = Date.now();
    const renderDuration = renderEndTime - renderStartTime.current;
    
    renderTimes.current.push(renderDuration);
    
    // Keep only last 10 render times for average calculation
    if (renderTimes.current.length > 10) {
      renderTimes.current = renderTimes.current.slice(-10);
    }
    
    const averageTime = renderTimes.current.reduce((sum, time) => sum + time, 0) / renderTimes.current.length;
    const totalTime = metrics.totalRenderTime + renderDuration;
    
    setMetrics(prev => ({
      renderCount: prev.renderCount + 1,
      lastRenderTime: renderDuration,
      averageRenderTime: averageTime,
      totalRenderTime: totalTime
    }));
    
    renderStartTime.current = Date.now();
  });

  // Performance test functions
  const heavyComputation = () => {
    console.time('Heavy Computation');
    let result = 0;
    for (let i = 0; i < 1000000; i++) {
      result += Math.sqrt(i);
    }
    console.timeEnd('Heavy Computation');
    return result;
  };

  const memoryIntensiveOperation = () => {
    console.time('Memory Intensive Operation');
    const largeArray = Array.from({ length: 100000 }, (_, i) => ({
      id: i,
      data: `Item ${i}`,
      timestamp: Date.now(),
      randomValue: Math.random()
    }));
    console.timeEnd('Memory Intensive Operation');
    console.log('Created array with', largeArray.length, 'items');
  };

  const simulateMemoryLeak = () => {
    // ❌ This would cause a memory leak
    const interval = setInterval(() => {
      console.log('Memory leak interval running...');
    }, 1000);
    
    // ❌ Not cleaning up the interval
    // This is just for demonstration - don't do this!
    
    Alert.alert(
      'Memory Leak Simulated',
      'Check console for continuous logs. This interval is not cleaned up!'
    );
  };

  const resetMetrics = () => {
    setMetrics({
      renderCount: 0,
      lastRenderTime: 0,
      averageRenderTime: 0,
      totalRenderTime: 0
    });
    renderTimes.current = [];
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Performance Monitor</Text>
      
      {/* Performance Metrics */}
      <View style={styles.metricsContainer}>
        <Text style={styles.metricsTitle}>Render Metrics</Text>
        
        <View style={styles.metricRow}>
          <Text style={styles.metricLabel}>Render Count:</Text>
          <Text style={styles.metricValue}>{metrics.renderCount}</Text>
        </View>
        
        <View style={styles.metricRow}>
          <Text style={styles.metricLabel}>Last Render:</Text>
          <Text style={styles.metricValue}>{metrics.lastRenderTime.toFixed(2)}ms</Text>
        </View>
        
        <View style={styles.metricRow}>
          <Text style={styles.metricLabel}>Average Render:</Text>
          <Text style={styles.metricValue}>{metrics.averageRenderTime.toFixed(2)}ms</Text>
        </View>
        
        <View style={styles.metricRow}>
          <Text style={styles.metricLabel}>Total Render Time:</Text>
          <Text style={styles.metricValue}>{metrics.totalRenderTime.toFixed(2)}ms</Text>
        </View>
      </View>

      {/* Performance Tests */}
      <View style={styles.testsContainer}>
        <Text style={styles.testsTitle}>Performance Tests</Text>
        
        <TouchableOpacity style={styles.testButton} onPress={heavyComputation}>
          <Text style={styles.testButtonText}>Heavy Computation Test</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.testButton} onPress={memoryIntensiveOperation}>
          <Text style={styles.testButtonText}>Memory Intensive Test</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={[styles.testButton, styles.dangerButton]} onPress={simulateMemoryLeak}>
          <Text style={styles.testButtonText}>Simulate Memory Leak</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={[styles.testButton, styles.resetButton]} onPress={resetMetrics}>
          <Text style={styles.testButtonText}>Reset Metrics</Text>
        </TouchableOpacity>
      </View>

      {/* Performance Tips */}
      <View style={styles.tipsContainer}>
        <Text style={styles.tipsTitle}>Performance Tips</Text>
        <Text style={styles.tipText}>
          • Use React.memo for expensive components{'\n'}
          • Implement useCallback for event handlers{'\n'}
          • Use useMemo for expensive calculations{'\n'}
          • Clean up subscriptions and timers{'\n'}
          • Optimize FlatList with proper props{'\n'}
          • Avoid inline functions in render{'\n'}
          • Use getItemLayout for known item sizes{'\n'}
          • Implement proper key props for lists
        </Text>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    padding: 20,
    backgroundColor: 'white',
    color: '#333',
  },
  metricsContainer: {
    backgroundColor: 'white',
    margin: 15,
    padding: 20,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  metricsTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  metricRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 10,
  },
  metricLabel: {
    fontSize: 16,
    color: '#666',
  },
  metricValue: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#007AFF',
  },
  testsContainer: {
    backgroundColor: 'white',
    margin: 15,
    padding: 20,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  testsTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  testButton: {
    backgroundColor: '#007AFF',
    paddingVertical: 12,
    paddingHorizontal: 20,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 10,
  },
  dangerButton: {
    backgroundColor: '#FF3B30',
  },
  resetButton: {
    backgroundColor: '#6C757D',
  },
  testButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  tipsContainer: {
    backgroundColor: 'white',
    margin: 15,
    padding: 20,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  tipsTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  tipText: {
    fontSize: 14,
    color: '#666',
    lineHeight: 22,
  },
});

export default PerformanceMonitor;
```

---

## 📝 Assignment 12: Performance Optimization

### Task 1: Performance Audit (120 minutes)
**Objective:** Audit and optimize a slow React Native app

**Given App Issues:**
- Unnecessary re-renders
- Memory leaks
- Slow list scrolling
- Large bundle size

**Requirements:**
1. **Identify Performance Issues:**
   - Use React DevTools Profiler
   - Monitor render counts
   - Check memory usage
   - Analyze bundle size

2. **Apply Optimizations:**
   - Add React.memo where needed
   - Implement useCallback/useMemo
   - Fix memory leaks
   - Optimize FlatList performance

3. **Measure Improvements:**
   - Before/after metrics
   - Performance benchmarks
   - User experience improvements

### Task 2: Memory Management (90 minutes)
**Objective:** Implement proper memory management

**Requirements:**
1. **Cleanup Implementation:**
   - Event listeners cleanup
   - Timer cleanup
   - Subscription cleanup
   - Async operation cancellation

2. **Memory Leak Prevention:**
   - Component unmount detection
   - Safe state updates
   - Proper cleanup patterns

### Task 3: Advanced Optimization (150 minutes)
**Objective:** Implement advanced performance patterns

**Requirements:**
1. **Lazy Loading:**
   - Component lazy loading
   - Image lazy loading
   - Data lazy loading

2. **Virtualization:**
   - Large list virtualization
   - Infinite scroll optimization
   - Memory-efficient rendering

---

## 🧠 Quiz 12: Performance & Lifecycle (20 Questions)

### Multiple Choice Questions (1-12)

**Q1. useEffect mein empty dependency array ka matlab kya hai?**
a) Effect har render pe chalega  
b) Effect sirf mount pe chalega  
c) Effect kabhi nahi chalega  
d) Effect sirf unmount pe chalega

**Q2. React.memo kya karta hai?**
a) Memory manage karta hai  
b) Unnecessary re-renders prevent karta hai  
c) State manage karta hai  
d) Props validate karta hai

**Q3. useMemo hook kab use karte hain?**
a) State management ke liye  
b) Expensive calculations ke liye  
c) Event handling ke liye  
d) API calls ke liye

**Q4. useCallback hook ka purpose kya hai?**
a) API calls ke liye  
b) Function references memoize karne ke liye  
c) State updates ke liye  
d) Error handling ke liye

**Q5. Memory leak kya hai?**
a) Memory ki shortage  
b) Memory jo release nahi hoti  
c) Memory corruption  
d) Memory overflow

**Q6. Component unmount detection kaise karte hain?**
a) useEffect cleanup function  
b) componentWillUnmount  
c) useRef flag  
d) All of the above

**Q7. FlatList performance optimize karne ke liye kya use karte hain?**
a) getItemLayout  
b) removeClippedSubviews  
c) windowSize  
d) All of the above

**Q8. Bundle size reduce karne ke tarike kya hain?**
a) Code splitting  
b) Tree shaking  
c) Minification  
d) All of the above

**Q9. React DevTools Profiler kya measure karta hai?**
a) Memory usage  
b) Render performance  
c) Network requests  
d) Bundle size

**Q10. Hermes JavaScript engine ka benefit kya hai?**
a) Faster startup  
b) Lower memory usage  
c) Better performance  
d) All of the above

**Q11. Component re-render kab hota hai?**
a) State change pe  
b) Props change pe  
c) Parent re-render pe  
d) All of the above

**Q12. Performance bottleneck identify kaise karte hain?**
a) Console.log  
b) Profiling tools  
c) Manual testing  
d) Code review

### True/False Questions (13-16)

**Q13. useMemo har render pe calculation karta hai.** (True/False)

**Q14. React.memo shallow comparison karta hai.** (True/False)

**Q15. useCallback dependencies change hone pe new function return karta hai.** (True/False)

**Q16. Memory leaks sirf development mein hote hain.** (True/False)

### Short Answer Questions (17-20)

**Q17. Component lifecycle ke main phases kya hain?**

**Q18. Memory leak prevent karne ke 5 tarike batao.**

**Q19. FlatList vs ScrollView performance mein kya difference hai?**

**Q20. Production app ki performance optimize karne ke steps kya hain?**

---

## 🎯 Quiz Answers

**MCQ Answers:**
1. b) Effect sirf mount pe chalega
2. b) Unnecessary re-renders prevent karta hai
3. b) Expensive calculations ke liye
4. b) Function references memoize karne ke liye
5. b) Memory jo release nahi hoti
6. d) All of the above
7. d) All of the above
8. d) All of the above
9. b) Render performance
10. d) All of the above
11. d) All of the above
12. b) Profiling tools

**True/False Answers:**
13. False (dependencies same hain to cached value return karta hai)
14. True
15. True
16. False (production mein bhi ho sakte hain)

**Short Answers:**
17. Mount, Update, Unmount
18. Cleanup timers, remove listeners, cancel async operations, use refs for mounted state, proper useEffect dependencies
19. FlatList lazy loading karta hai, ScrollView all items render karta hai
20. Profiling, optimization, testing, monitoring, bundle analysis

---

## 🔗 Additional Resources

### Performance Tools
- [React DevTools Profiler](https://reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html)
- [Flipper Performance Plugin](https://fbflipper.com/docs/features/react-native-performance/)
- [React Native Performance](https://reactnative.dev/docs/performance)

### Optimization Libraries
- [React Native Reanimated](https://docs.swmansion.com/react-native-reanimated/)
- [React Native Fast Image](https://github.com/DylanVann/react-native-fast-image)
- [React Native Optimized FlatList](https://github.com/stoffern/react-native-super-grid)

### Best Practices
- [React Performance](https://reactjs.org/docs/optimizing-performance.html)
- [React Native Performance Best Practices](https://reactnative.dev/docs/performance)
- [Memory Management](https://reactnative.dev/docs/performance#memory-usage)

---

## 🎉 Lesson Complete!

Congratulations! Aapne successfully:
- ✅ Component lifecycle phases samjhe
- ✅ Performance optimization techniques sikhe
- ✅ Memory management implement kiya
- ✅ Performance monitoring tools use kiye
- ✅ Production-ready optimization patterns samjhe

**Next Lesson Preview:** Animations & Gestures (Reanimated) - React Native mein smooth animations aur gesture handling sikhenge.

---

*Keep Optimizing! ⚡🚀*