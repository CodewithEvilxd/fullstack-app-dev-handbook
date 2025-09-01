
# 🐛 Lesson 10: Debugging & Development Tools

## 🎯 Learning Objectives
Is lesson ke baad aap:
- React Native debugging tools use kar sakte hain
- Common errors identify aur fix kar sakte hain
- Performance optimization techniques samjhenge
- Development workflow improve kar sakte hain
- Production-ready apps build kar sakte hain

---

## 📖 Development Environment Setup

React Native development ke liye proper debugging setup zaroori hai.

### 1. Developer Menu Access

```javascript
// Enable Developer Menu
// iOS Simulator: Cmd + D
// Android Emulator: Cmd + M (Mac) or Ctrl + M (Windows/Linux)
// Physical Device: Shake the device

// Programmatically open developer menu
import { NativeModules } from 'react-native';

const openDeveloperMenu = () => {
  if (__DEV__) {
    NativeModules.DevMenu.show();
  }
};
```

### 2. Debug Mode Configuration

```javascript
// App.js - Debug Configuration
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

const DebugApp = () => {
  // Check if running in debug mode
  const isDebugMode = __DEV__;
  
  // Debug information
  const debugInfo = {
    platform: Platform.OS,
    version: Platform.Version,
    debugMode: __DEV__,
    timestamp: new Date().toISOString()
  };

  const logDebugInfo = () => {
    console.log('Debug Info:', debugInfo);
    console.warn('This is a warning message');
    console.error('This is an error message');
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Debug Mode: {isDebugMode ? 'ON' : 'OFF'}</Text>
      
      {isDebugMode && (
        <View style={styles.debugPanel}>
          <Text style={styles.debugTitle}>Debug Information</Text>
          <Text style={styles.debugText}>Platform: {debugInfo.platform}</Text>
          <Text style={styles.debugText}>Version: {debugInfo.version}</Text>
          <Text style={styles.debugText}>Timestamp: {debugInfo.timestamp}</Text>
          
          <TouchableOpacity style={styles.debugButton} onPress={logDebugInfo}>
            <Text style={styles.debugButtonText}>Log Debug Info</Text>
          </TouchableOpacity>
        </View>
      )}
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
  debugPanel: {
    backgroundColor: '#fff3cd',
    padding: 20,
    borderRadius: 8,
    borderWidth: 1,
    borderColor: '#ffeaa7',
  },
  debugTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#856404',
    marginBottom: 10,
  },
  debugText: {
    fontSize: 14,
    color: '#856404',
    marginBottom: 5,
  },
  debugButton: {
    backgroundColor: '#007AFF',
    paddingVertical: 10,
    paddingHorizontal: 15,
    borderRadius: 5,
    marginTop: 10,
  },
  debugButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
});

export default DebugApp;
```

---

## 🔍 Console Debugging

### 1. Console Methods

```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

const ConsoleDebuggingExample = () => {
  const [counter, setCounter] = useState(0);
  const [users, setUsers] = useState([]);

  useEffect(() => {
    // Basic console logging
    console.log('Component mounted');
    
    // Console with styling (works in Chrome DevTools)
    console.log('%c Component Mounted', 'color: green; font-weight: bold;');
    
    // Cleanup function
    return () => {
      console.log('Component will unmount');
    };
  }, []);

  useEffect(() => {
    // Log state changes
    console.log('Counter updated:', counter);
    
    // Conditional logging
    if (counter > 5) {
      console.warn('Counter is getting high:', counter);
    }
    
    if (counter > 10) {
      console.error('Counter exceeded limit!');
    }
  }, [counter]);

  const demonstrateConsoleMethods = () => {
    // Different console methods
    console.log('Regular log message');
    console.info('Info message');
    console.warn('Warning message');
    console.error('Error message');
    
    // Console table (great for arrays/objects)
    const sampleData = [
      { id: 1, name: 'John', age: 25 },
      { id: 2, name: 'Jane', age: 30 },
      { id: 3, name: 'Bob', age: 35 }
    ];
    console.table(sampleData);
    
    // Console group (organize related logs)
    console.group('User Data Processing');
    console.log('Processing user 1...');
    console.log('Processing user 2...');
    console.log('Processing user 3...');
    console.groupEnd();
    
    // Console time (measure performance)
    console.time('Data Processing');
    // Simulate some processing
    setTimeout(() => {
      console.timeEnd('Data Processing');
    }, 1000);
    
    // Console count (count occurrences)
    console.count('Button clicked');
    
    // Console assert (conditional logging)
    console.assert(counter < 100, 'Counter should be less than 100');
    
    // Console trace (show call stack)
    console.trace('Trace from button click');
  };

  const simulateError = () => {
    try {
      // Intentional error for demonstration
      const result = JSON.parse('invalid json');
    } catch (error) {
      console.error('JSON Parse Error:', error);
      console.error('Error stack:', error.stack);
    }
  };

  const logComplexData = () => {
    const complexObject = {
      user: {
        id: 1,
        name: 'John Doe',
        preferences: {
          theme: 'dark',
          notifications: true,
          languages: ['en', 'es', 'fr']
        }
      },
      metadata: {
        lastLogin: new Date(),
        sessionCount: 42
      }
    };
    
    // Log complex objects
    console.log('Complex Object:', complexObject);
    console.dir(complexObject); // Alternative way to log objects
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Console Debugging</Text>
      
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Counter: {counter}</Text>
        <View style={styles.buttonRow}>
          <TouchableOpacity
            style={styles.button}
            onPress={() => setCounter(counter + 1)}
          >
            <Text style={styles.buttonText}>Increment</Text>
          </TouchableOpacity>
          
          <TouchableOpacity
            style={[styles.button, styles.resetButton]}
            onPress={() => setCounter(0)}
          >
            <Text style={styles.buttonText}>Reset</Text>
          </TouchableOpacity>
        </View>
      </View>

      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Debug Actions</Text>
        
        <TouchableOpacity
          style={styles.debugButton}
          onPress={demonstrateConsoleMethods}
        >
          <Text style={styles.debugButtonText}>Demo Console Methods</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={styles.debugButton}
          onPress={simulateError}
        >
          <Text style={styles.debugButtonText}>Simulate Error</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={styles.debugButton}
          onPress={logComplexData}
        >
          <Text style={styles.debugButtonText}>Log Complex Data</Text>
        </TouchableOpacity>
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
    color: '#333',
    marginBottom: 15,
    textAlign: 'center',
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
  },
  resetButton: {
    backgroundColor: '#FF3B30',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  debugButton: {
    backgroundColor: '#28A745',
    paddingVertical: 12,
    paddingHorizontal: 20,
    borderRadius: 8,
    marginBottom: 10,
    alignItems: 'center',
  },
  debugButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
});

export default ConsoleDebuggingExample;
```

---

## 🛠️ React Developer Tools

### 1. React DevTools Setup

```bash
# Install React DevTools globally
npm install -g react-devtools

# Run React DevTools
react-devtools

# Or use the browser extension
# Chrome: React Developer Tools extension
# Firefox: React Developer Tools extension
```

### 2. Component Debugging

```javascript
import React, { useState, useEffect, useDebugValue } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

// Custom hook with debug value
const useCounter = (initialValue = 0) => {
  const [count, setCount] = useState(initialValue);
  
  // This will show in React DevTools
  useDebugValue(count > 5 ? 'High' : 'Low');
  
  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);
  const reset = () => setCount(initialValue);
  
  return { count, increment, decrement, reset };
};

// Component with display name for better debugging
const CounterComponent = ({ title }) => {
  const { count, increment, decrement, reset } = useCounter(0);
  
  useEffect(() => {
    // This effect will be visible in React DevTools
    console.log(`${title} count changed:`, count);
  }, [count, title]);

  return (
    <View style={styles.counterContainer}>
      <Text style={styles.counterTitle}>{title}</Text>
      <Text style={styles.counterValue}>{count}</Text>
      
      <View style={styles.counterButtons}>
        <TouchableOpacity style={styles.counterButton} onPress={decrement}>
          <Text style={styles.counterButtonText}>-</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.counterButton} onPress={increment}>
          <Text style={styles.counterButtonText}>+</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={[styles.counterButton, styles.resetButton]} onPress={reset}>
          <Text style={styles.counterButtonText}>Reset</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

// Set display name for better debugging
CounterComponent.displayName = 'CounterComponent';

const ReactDevToolsExample = () => {
  const [showSecondCounter, setShowSecondCounter] = useState(false);
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>React DevTools Example</Text>
      
      <CounterComponent title="Primary Counter" />
      
      <TouchableOpacity
        style={styles.toggleButton}
        onPress={() => setShowSecondCounter(!showSecondCounter)}
      >
        <Text style={styles.toggleButtonText}>
          {showSecondCounter ? 'Hide' : 'Show'} Second Counter
        </Text>
      </TouchableOpacity>
      
      {showSecondCounter && (
        <CounterComponent title="Secondary Counter" />
      )}
      
      <View style={styles.debugInfo}>
        <Text style={styles.debugInfoTitle}>Debug Tips:</Text>
        <Text style={styles.debugInfoText}>
          • Open React DevTools to inspect component tree{'\n'}
          • Check component props and state{'\n'}
          • Monitor hook values and effects{'\n'}
          • Use profiler to measure performance
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
  counterContainer: {
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
  counterTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    textAlign: 'center',
    marginBottom: 10,
  },
  counterValue: {
    fontSize: 48,
    fontWeight: 'bold',
    color: '#007AFF',
    textAlign: 'center',
    marginBottom: 20,
  },
  counterButtons: {
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  counterButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 20,
    paddingVertical: 12,
    borderRadius: 8,
    minWidth: 60,
    alignItems: 'center',
  },
  resetButton: {
    backgroundColor: '#FF3B30',
  },
  counterButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  toggleButton: {
    backgroundColor: '#28A745',
    paddingVertical: 15,
    paddingHorizontal: 20,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 20,
  },
  toggleButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  debugInfo: {
    backgroundColor: '#fff3cd',
    padding: 20,
    borderRadius: 8,
    borderWidth: 1,
    borderColor: '#ffeaa7',
  },
  debugInfoTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#856404',
    marginBottom: 10,
  },
  debugInfoText: {
    fontSize: 14,
    color: '#856404',
    lineHeight: 20,
  },
});

export default ReactDevToolsExample;
```

---

## 🚀 Performance Debugging

### 1. Performance Monitoring

```javascript
import React, { useState, useEffect, useMemo, useCallback } from 'react';
import {
  View,
  Text,
  FlatList,
  TouchableOpacity,
  StyleSheet,
  Alert
} from 'react-native';

const PerformanceExample = () => {
  const [data, setData] = useState([]);
  const [filter, setFilter] = useState('');
  const [renderCount, setRenderCount] = useState(0);

  // Performance monitoring
  useEffect(() => {
    setRenderCount(prev => prev + 1);
    console.log('Component rendered:', renderCount + 1);
  });

  // Generate large dataset
  const generateData = useCallback(() => {
    console.time('Data Generation');
    const newData = Array.from({ length: 1000 }, (_, i) => ({
      id: i,
      name: `Item ${i}`,
      value: Math.random() * 100,
      category: ['A', 'B', 'C'][Math.floor(Math.random() * 3)]
    }));
    setData(newData);
    console.timeEnd('Data Generation');
  }, []);

  // Memoized filtered data (performance optimization)
  const filteredData = useMemo(() => {
    console.time('Data Filtering');
    const result = data.filter(item =>
      item.name.toLowerCase().includes(filter.toLowerCase()) ||
      item.category.toLowerCase().includes(filter.toLowerCase())
    );
    console.timeEnd('Data Filtering');
    return result;
  }, [data, filter]);

  // Memoized render item (prevents unnecessary re-renders)
  const renderItem = useCallback(({ item }) => (
    <ItemComponent item={item} />
  ), []);

  const measurePerformance = () => {
    console.time('Performance Test');
    
    // Simulate heavy computation
    let result = 0;
    for (let i = 0; i < 1000000; i++) {
      result += Math.random();
    }
    
    console.timeEnd('Performance Test');
    Alert.alert('Performance Test', `Result: ${result.toFixed(2)}`);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Performance Debugging</Text>
      
      <View style={styles.stats}>
        <Text style={styles.statsText}>Renders: {renderCount}</Text>
        <Text style={styles.statsText}>Items: {filteredData.length}</Text>
      </View>

      <View style={styles.controls}>
        <TouchableOpacity style={styles.button} onPress={generateData}>
          <Text style={styles.buttonText}>Generate Data</Text>
        </TouchableOpacity>
        
        <TouchableOpacity style={styles.button} onPress={measurePerformance}>
          <Text style={styles.buttonText}>Measure Performance</Text>
        </TouchableOpacity>
      </View>

      <FlatList
        data={filteredData}
        renderItem={renderItem}
        keyExtractor={item => item.id.toString()}
        style={styles.list}
        getItemLayout={(data, index) => ({
          length: 60,
          offset: 60 * index,
          index,
        })}
        removeClippedSubviews={true}
        maxToRenderPerBatch={10}
        windowSize={10}
      />
    </View>
  );
};

// Memoized item component to prevent unnecessary re-renders
const ItemComponent = React.memo(({ item }) => {
  console.log('Rendering item:', item.id);
  
  return (
    <View style={styles.item}>
      <Text style={styles.itemName}>{item.name}</Text>
      <Text style={styles.itemValue}>{item.value.toFixed(2)}</Text>
      <Text style={styles.itemCategory}>{item.category}</Text>
    </View>
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
  stats: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    backgroundColor: 'white',
    paddingVertical: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  statsText: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#007AFF',
  },
  controls: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    padding: 15,
    backgroundColor: 'white',
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 20,
    paddingVertical: 12,
    borderRadius: 8,
  },
  buttonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
  list: {
    flex: 1,
  },
  item: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    backgroundColor: 'white',
    padding: 15,
    marginHorizontal: 15,
    marginVertical: 2,
    borderRadius: 8,
  },
  itemName: {
    fontSize: 16,
    color: '#333',
    flex: 1,
  },
  itemValue: {
    fontSize: 14,
    color: '#666',
    marginHorizontal: 10,
  },
  itemCategory: {
    fontSize: 14,
    color: '#007AFF',
    fontWeight: 'bold',
  },
});

export default PerformanceExample;
```

---

## 🔧 Network Debugging

### 1. Network Request Monitoring

```javascript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  ScrollView,
  StyleSheet,
  Alert
} from 'react-native';

const NetworkDebuggingExample = () => {
  const [requests, setRequests] = useState([]);
  const [loading, setLoading] = useState(false);

  // Network request interceptor for debugging
  const originalFetch = global.fetch;
  
  useEffect(() => {
    // Intercept fetch requests for debugging
    global.fetch = async (...args) => {
      const startTime = Date.now();
      const [url, options = {}] = args;
      
      console.group(`🌐 Network Request: ${options.method || 'GET'} ${url}`);
      console.log('Request URL:', url);
      console.log('Request Options:', options);
      
      try {
        const response = await originalFetch(...args);
        const endTime = Date.now();
        const duration = endTime - startTime;
        
        console.log('Response Status:', response.status);
        console.log('Response Headers:', Object.fromEntries(response.headers.entries()));
        console.log('Duration:', `${duration}ms`);
        console.groupEnd();
        
        // Log request for UI display
        setRequests(prev => [...prev, {
          id: Date.now(),
          url: url.toString(),
          method: options.method || 'GET',
          status: response.status,
          duration,
          timestamp: new Date().toLocaleTimeString()
        }]);
        
        return response;
      } catch (error) {
        const endTime = Date.now();
        const duration = endTime - startTime;
        
        console.error('Request Failed:', error);
        console.log('Duration:', `${duration}ms`);
        console.groupEnd();
        
        // Log failed request
        setRequests(prev => [...prev, {
          id: Date.now(),
          url: url.toString(),
          method: options.method || 'GET',
          status: 'ERROR',
          duration,
          timestamp: new Date().toLocaleTimeString(),
          error: error.message
        }]);
        
        throw error;
      }
    };
    
    // Cleanup
    return () => {
      global.fetch = originalFetch;
    };
  }, []);

  const makeSuccessfulRequest = async () => {
    setLoading(true);
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts/1');
      const data = await response.json();
      console.log('Successful Response:', data);
      Alert.alert('Success', 'Request completed successfully');
    } catch (error) {
      console.error('Request Error:', error);
      Alert.alert('Error', error.message);
    } finally {
      setLoading(false);
    }
  };

  const makeFailedRequest = async () => {
    setLoading(true);
    try {
      const response = await fetch('https://invalid-url-that-will-fail.com/api');
      const data = await response.json();
    } catch (error) {
      console.error('Expected Error:', error);
      Alert.alert('Expected Error', 'This request was supposed to fail');
    } finally {
      setLoading(false);
    }
  };

  const makeSlowRequest = async () => {
    setLoading(true);
    try {
      // Simulate slow request
      const response = await fetch('https://httpbin.org/delay/3');
      const data = await response.json();
      console.log('Slow Response:', data);
      Alert.alert('Success', 'Slow request completed');
    } catch (error) {
      console.error('Slow Request Error:', error);
      Alert.alert('Error', error.message);
    } finally {
      setLoading(false);
    }
  };

  const clearRequests = () => {
    setRequests([]);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Network Debugging</Text>
      
      <View style={styles.controls}>
        <TouchableOpacity
          style={[styles.button, loading && styles.buttonDisabled]}
          onPress={makeSuccessfulRequest}
          disabled={loading}
        >
          <Text style={styles.buttonText}>Successful Request</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.button, styles.errorButton, loading && styles.buttonDisabled]}
          onPress={makeFailedRequest}
          disabled={loading}
        >
          <Text style={styles.buttonText}>Failed Request</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.button, styles.slowButton, loading && styles.buttonDisabled]}
          onPress={makeSlowRequest}
          disabled={loading}
        >
          <Text style={styles.buttonText}>Slow Request</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.button, styles.clearButton]}
          onPress={clearRequests}
        >
          <Text style={styles.buttonText}>Clear Log</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.requestsHeader}>
        <Text style={styles.requestsTitle}>Network Requests ({requests.length})</Text>
      </View>

      <ScrollView style={styles.requestsList}>
        {requests.map(request => (
          <View key={request.id} style={styles.requestItem}>
            <View style={styles.requestHeader}>
              <Text style={styles.requestMethod}>{request.method}</Text>
              <Text style={[
                styles.requestStatus,
                request.status === 'ERROR' ? styles.errorStatus : 
                request.status >= 400 ? styles.errorStatus :
                request.status >= 300 ? styles.warningStatus :
                styles.successStatus
              ]}>
                {request.status}
              </Text>
              <Text style={styles.requestDuration}>{request.duration}ms</Text>
            </View>
            
            <Text style={styles.requestUrl} numberOfLines={2}>
              {request.url}
            </Text>
            
            <Text style={styles.requestTime}>{request.timestamp}</Text>
            
            {request.error && (
              <Text style={styles.requestError}>Error: {request.error}</Text>
            )}
          </View>
        ))}
        
        {requests.length === 0 && (
          <View style={styles.emptyState}>
            <Text style={styles.emptyStateText}>No network requests yet</Text>
            <Text style={styles.emptyStateSubtext}>
              Make some requests to see them logged here
            </Text>
          </View>
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
    padding: 15,
    backgroundColor: 'white',
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 15,
    paddingVertical: 10,
    borderRadius: 6,
    marginBottom: 10,
    minWidth: 120,
    alignItems: 'center',
  },
  errorButton: {
    backgroundColor: '#FF3B30',
  },
  slowButton: {
    backgroundColor: '#FF9500',
  },
  clearButton: {
    backgroundColor: '#6C757D',
  },
  buttonDisabled: {
    opacity: 0.5,
  },
  buttonText: {
    color: 'white',
    fontSize: 12,
    fontWeight: 'bold',
  },
  requestsHeader: {
    backgroundColor: 'white',
    paddingHorizontal: 15,
    paddingVertical: 10,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  requestsTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
  },
  requestsList: {
    flex: 1,
  },
  requestItem: {
    backgroundColor: 'white',
    margin: 10,
    padding: 15,
    borderRadius: 8,
    borderLeftWidth: 4,
    borderLeftColor: '#007AFF',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.2,
    shadowRadius: 2,
    elevation: 3,
  },
  requestHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 8,
  },
  requestMethod: {
    fontSize: 14,
    fontWeight: 'bold',
    color: '#007AFF',
    backgroundColor: '#f0f8ff',
    paddingHorizontal: 8,
    paddingVertical: 4,
    borderRadius: 4,
  },
  requestStatus: {
    fontSize: 14,
    fontWeight: 'bold',
    paddingHorizontal: 8,
    paddingVertical: 4,
    borderRadius: 4,
  },
  successStatus: {
    backgroundColor: '#d4edda',
    color: '#155724',
  },
  warningStatus: {
    backgroundColor: '#fff3cd',
    color: '#856404',
  },
  errorStatus: {
    backgroundColor: '#f8d7da',
    color: '#721c24',
  },
  requestDuration: {
    fontSize: 12,
    color: '#666',
    fontWeight: 'bold',
  },
  requestUrl: {
    fontSize: 14,
    color: '#333',
    marginBottom: 5,
  },
  requestTime: {
    fontSize: 12,
    color: '#999',
    marginBottom: 5,
  },
  requestError: {
    fontSize: 12,
    color: '#FF3B30',
    backgroundColor: '#fff5f5',
    padding: 8,
    borderRadius: 4,
    marginTop: 5,
  },
  emptyState: {
    alignItems: 'center',
    padding: 50,
    backgroundColor: 'white',
    margin: 15,
    borderRadius: 8,
  },
  emptyStateText: {
    fontSize: 16,
    color: '#666',
    marginBottom: 5,
  },
  emptyStateSubtext: {
    fontSize: 14,
    color: '#999',
    textAlign: 'center',
  },
});

export default NetworkDebuggingExample;
```

---

## 🔍 Error Boundary Implementation

### 1. Error Boundary Component

```javascript
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Log error details
    console.error('Error Boundary caught an error:', error);
    console.error('Error Info:', errorInfo);
    
    // Update state with error details
    this.setState({
      error: error,
      errorInfo: errorInfo
    });
    
    // You can also log the error to an error reporting service here
    // logErrorToService(error, errorInfo);
  }

  handleRetry = () => {
    this.setState({ hasError: false, error: null, errorInfo: null });
  };

  render() {
    if (this.state.hasError) {
      // Custom error UI
      return (
        <View style={styles.errorContainer}>
          <Text style={styles.errorTitle}>Oops! Something went wrong</Text>
          <Text style={styles.errorMessage}>
            {this.state.error && this.state.error.toString()}
          </Text>
          
          {__DEV__ && (
            <View style={styles.errorDetails}>
              <Text style={styles.errorDetailsTitle}>Error Details:</Text>
              <Text style={styles.errorDetailsText}>
                {this.state.errorInfo.componentStack}
              </Text>
            </View>
          )}
          
          <TouchableOpacity style={styles.retryButton} onPress={this.handleRetry}>
            <Text style={styles.retryButtonText}>Try Again</Text>
          </TouchableOpacity>
        </View>
      );
    }

    return this.props.children;
  }
}

const styles = StyleSheet.create({
  errorContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#f8f9fa',
  },
  errorTitle: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#dc3545',
    marginBottom: 15,
    textAlign: 'center',
  },
  errorMessage: {
    fontSize: 16,
    color: '#6c757d',
    marginBottom: 20,
    textAlign: 'center',
  },
  errorDetails: {
    backgroundColor: '#f8d7da',
    padding: 15,
    borderRadius: 8,
    marginBottom: 20,
    width: '100%',
  },
  errorDetailsTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#721c24',
    marginBottom: 10,
  },
  errorDetailsText: {
    fontSize: 12,
    color: '#721c24',
    fontFamily: 'monospace',
  },
  retryButton: {
    backgroundColor: '#007bff',
    paddingHorizontal: 30,
    paddingVertical: 12,
    borderRadius: 8,
  },
  retryButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default ErrorBoundary;
```

### 2. Using Error Boundary

```javascript
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import ErrorBoundary from './ErrorBoundary';

// Component that might throw an error
const ProblematicComponent = ({ shouldThrowError }) => {
  if (shouldThrowError) {
    throw new Error('This is a test error!');
  }
  
  return (
    <View style={styles.component}>
      <Text style={styles.componentText}>This component is working fine!</Text>
    </View>
  );
};

const ErrorBoundaryExample = () => {
  const [shouldThrowError, setShouldThrowError] = useState(false);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Error Boundary Example</Text>
      
      <TouchableOpacity
        style={styles.button}
        onPress={() => setShouldThrowError(!shouldThrowError)}
      >
        <Text style={styles.buttonText}>
          {shouldThrowError ? 'Fix Component' : 'Break Component'}
        </Text>
      </TouchableOpacity>

      <ErrorBoundary>
        <ProblematicComponent shouldThrowError={shouldThrowError} />
      </ErrorBoundary>
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
  button: {
    backgroundColor: '#dc3545',
    paddingVertical: 15,
    paddingHorizontal: 20,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 30,
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  component: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 8,
    alignItems: 'center',
  },
  componentText: {
    fontSize: 18,
    color: '#28a745',
  },
});

export default ErrorBoundaryExample;
```

---

## 📝 Assignment 10: Debugging Mastery

### Task 1: Debug Dashboard (180 minutes)
**Objective:** Create a comprehensive debugging dashboard

**Requirements:**
1. **Performance Monitor:**
   - Render count tracking
   - Memory usage display
   - FPS monitoring
   - Bundle size information

2. **Network Monitor:**
   - Request/response logging
   - Error tracking
   - Performance metrics
   - Request replay functionality

3. **Error Tracking:**
   - Error boundary implementation
   - Crash reporting
   - Error categorization
   - Stack trace analysis

4. **Development Tools:**
   - Console log viewer
   - State inspector
   - Props debugger
   - Navigation tracker

### Task 2: Performance Optimization (150 minutes)
**Objective:** Optimize a slow React Native app

**Given:** A poorly performing app with:
- Unnecessary re-renders
- Large bundle size
- Slow list scrolling
- Memory leaks

**Requirements:**
1. **Identify Issues:**
   - Use React DevTools Profiler
   - Analyze bundle with Metro
   - Monitor memory usage
   - Track render performance

2. **Apply Optimizations:**
   - Implement React.memo
   - Use useCallback/useMemo
   - Optimize FlatList
   - Fix memory leaks

3. **Measure Improvements:**
   - Before/after metrics
   - Performance benchmarks
   - User experience improvements

### Task 3: Error Handling System (120 minutes)
**Objective:** Build robust error handling

**Requirements:**
1. **Error Boundaries:**
   - Global error boundary
   - Component-specific boundaries
   - Error recovery mechanisms
   - User-friendly error messages

2. **Crash Reporting:**
   - Automatic crash detection
   - Error logging service
   - User feedback collection
   - Error categorization

3. **Debugging Tools:**
   - Custom debug panel
   - Log level filtering
   - Remote debugging support
   - Error reproduction tools

---

## 🧠 Quiz 10: Debugging & Development (25 Questions)

### Multiple Choice Questions (1-15)

**Q1. Developer menu access karne ka shortcut kya hai iOS simulator mein?**
a) Cmd + R
b) Cmd + D
c) Cmd + M
d) Cmd + Shift + D

**Q2. __DEV__ variable kya indicate karta hai?**
a) Device type
b) Debug mode status
c) Development server
d) Developer name

**Q3. React DevTools install karne ka command kya hai?**
a) npm install react-devtools
b) npm install -g react-devtools
c) npm install --save-dev react-devtools
d) npm install @react-devtools/core

**Q4. console.time() aur console.timeEnd() ka use kya hai?**
a) Current time display
b) Performance measurement
c) Timer functionality
d) Time formatting

**Q5. useDebugValue hook ka purpose kya hai?**
a) Debug mode enable karna
b) Custom hooks mein debug info show karna
c) Error debugging
d) Performance debugging

**Q6. React.memo() kab use karte hain?**
a) Memory management ke liye
b) Unnecessary re-renders prevent karne ke liye
c) State management ke liye
d) Error handling ke liye

**Q7. FlatList performance optimize karne ke liye kya use karte hain?**
a) getItemLayout
b) removeClippedSubviews
c) windowSize
d) All of the above

**Q8. Error Boundary sirf kahan errors catch kar sakta hai?**
a) Render methods mein
b) Event handlers mein
c) useEffect mein
d) Async functions mein

**Q9. Network requests debug karne ka best way kya hai?**
a) console.log use karna
b) Fetch interceptor use karna
c) Alert use karna
d) File logging

**Q10. componentDidCatch method kya return karta hai?**
a) Error object
b) Boolean value
c) Nothing (void)
d) Error message

**Q11. Performance profiling ke liye kaunsa tool use karte hain?**
a) React DevTools Profiler
b) Chrome DevTools
c) Flipper
d) All of the above

**Q12. Memory leaks prevent karne ke liye kya karna chahiye?**
a) useEffect cleanup
b) Event listeners remove karna
c) Timers clear karna
d) All of the above

**Q13. Bundle size analyze karne ke liye kya use karte hain?**
a) Metro bundler
b) Webpack analyzer
c) Bundle analyzer
d) React DevTools

**Q14. Remote debugging enable karne ke liye kya karna hota hai?**
a) Developer menu se "Debug JS Remotely"
b) Chrome DevTools open karna
c) React DevTools connect karna
d) All of the above

**Q15. Production build mein debugging disable karne ke liye kya check karte hain?**
a) __DEV__ flag
b) NODE_ENV
c) DEBUG flag
d) PRODUCTION flag

### True/False Questions (16-20)

**Q16. Error boundaries async errors catch kar sakte hain.** (True/False)

**Q17. console.table() arrays aur objects ko table format mein display karta hai.** (True/False)

**Q18. React DevTools sirf development mode mein kaam karta hai.** (True/False)

**Q19. useCallback har render pe new function create karta hai.** (True/False)

**Q20. Flipper sirf Android debugging ke liye hai.** (True/False)

### Short Answer Questions (21-25)

**Q21. React Native app ki performance optimize karne ke 5 tarike batao.**

**Q22. Error boundary implement karne ke steps kya hain?**

**Q23. Network debugging ke liye kaunse tools available hain?**

**Q24. Memory leaks identify karne ke methods kya hain?**

**Q25. Production debugging ke best practices kya hain?**

---

## 🎯 Quiz Answers

**MCQ Answers:**
1. b) Cmd + D
2. b) Debug mode status
3. b) npm install -g react-devtools
4. b) Performance measurement
5. b) Custom hooks mein debug info show karna
6. b) Unnecessary re-renders prevent karne ke liye
7. d) All of the above
8. a) Render methods mein
9. b) Fetch interceptor use karna
10. c) Nothing (void)
11. d) All of the above
12. d) All of the above
13. a) Metro bundler
14. d) All of the above
15. a) __DEV__ flag

**True/False Answers:**
16. False (sirf render errors catch karte hain)
17. True
18. False (production mein bhi kaam karta hai)
19. False (dependencies same hain to same function return karta hai)
20. False (iOS aur Android dono ke liye hai)

**Short Answers:**
21. React.memo, useCallback, useMemo, FlatList optimization, bundle splitting
22. Class component banao, componentDidCatch implement karo, error UI return karo
23. Chrome DevTools, React DevTools, Flipper, Network interceptors
24. Memory profiling, heap snapshots, component unmount tracking
25. Error boundaries, crash reporting, performance monitoring, user feedback

---

## 🔗 Additional Resources

### Debugging Tools
- [React DevTools](https://github.com/facebook/react/tree/main/packages/react-devtools)
- [Flipper](https://fbflipper.com/)
- [Reactotron](https://github.com/infinitered/reactotron)
- [React Native Debugger](https://github.com/jhen0409/react-native-debugger)

### Performance Tools
- [React Native Performance](https://reactnative.dev/docs/performance)
- [Metro Bundler](https://facebook.github.io/metro/)
- [Hermes JavaScript Engine](https://hermesengine.dev/)

### Error Tracking
- [Sentry](https://sentry.io/)
- [Bugsnag](https://www.bugsnag.com/)
- [Crashlytics](https://firebase.google.com/products/crashlytics)

---

## 🎉 Lesson Complete!

Congratulations! Aapne successfully:
- ✅ React Native debugging tools master kiye
- ✅ Performance optimization techniques sikhe
- ✅ Error handling aur boundaries implement kiye
- ✅ Network debugging aur monitoring sikha
- ✅ Production-ready debugging practices samjhe

**Course Section Complete!** React Native Fundamentals (Lessons 1-10) successfully complete ho gaye hain. Ab aap advanced concepts ke liye ready hain!

---

*Happy Debugging! 🐛➡️✨*