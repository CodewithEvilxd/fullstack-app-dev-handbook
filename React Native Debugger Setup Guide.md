# 🔍 React Native Debugger Setup Guide
## Complete Debugging & Development Tool Manual

---

## 🎯 **Overview**
React Native Debugger is a standalone app for debugging React Native applications. It provides powerful debugging capabilities including React DevTools, Redux DevTools, and network inspection. This guide covers complete setup, configuration, and usage.

---

## 📋 **Table of Contents**
1. [What is React Native Debugger?](#what-is-react-native-debugger)
2. [Installation & Setup](#installation--setup)
3. [Basic Usage](#basic-usage)
4. [React DevTools](#react-devtools)
5. [Redux DevTools](#redux-devtools)
6. [Network Inspection](#network-inspection)
7. [Performance Monitoring](#performance-monitoring)
8. [Advanced Features](#advanced-features)
9. [Troubleshooting](#troubleshooting)
10. [Best Practices](#best-practices)

---

## 🤔 **What is React Native Debugger?**

### **Key Features:**
```bash
✅ React DevTools integration
✅ Redux DevTools support
✅ Network request inspection
✅ AsyncStorage inspection
✅ Component hierarchy visualization
✅ Hot reloading support
✅ Breakpoint debugging
✅ Performance monitoring
✅ Console logging
✅ Element inspection
```

### **Why Use React Native Debugger:**
```bash
🚀 Standalone debugging environment
🛠️ Advanced debugging tools
📊 Performance insights
🔍 Network monitoring
💾 State inspection
🎯 Component debugging
⚡ Hot reloading
📱 Device-specific debugging
```

### **Comparison with Other Debuggers:**
```bash
React Native Debugger:
✅ Standalone app
✅ React DevTools built-in
✅ Redux DevTools integrated
✅ Network inspection
❌ Browser-based

Chrome DevTools:
✅ Browser-based
✅ Familiar interface
✅ Good for web debugging
❌ Limited React Native features
❌ No Redux DevTools by default

Flipper:
✅ Facebook's official tool
✅ Plugin ecosystem
✅ Native debugging
❌ Separate installation
❌ Less React-focused
```

---

## 📥 **Installation & Setup**

### **Method 1: Standalone Installation**

#### **macOS Installation:**
```bash
# Using Homebrew
brew install --cask react-native-debugger

# Manual download
# Visit: https://github.com/jhen0409/react-native-debugger/releases
# Download .dmg file for macOS
# Mount and drag to Applications

# Verify installation
/Applications/React\ Native\ Debugger.app/Contents/MacOS/React\ Native\ Debugger --version
```

#### **Windows Installation:**
```bash
# Download from GitHub releases
https://github.com/jhen0409/react-native-debugger/releases

# Download .exe installer
# Run installer with administrator privileges

# Or using Chocolatey
choco install react-native-debugger

# Verify installation
react-native-debugger --version
```

#### **Linux Installation:**
```bash
# Download AppImage from releases
wget https://github.com/jhen0409/react-native-debugger/releases/download/v0.13.0/react-native-debugger_0.13.0_amd64.deb

# Install .deb package
sudo dpkg -i react-native-debugger_0.13.0_amd64.deb

# Or using Snap
sudo snap install react-native-debugger --edge

# Verify installation
react-native-debugger --version
```

### **Method 2: Development Installation**
```bash
# Clone repository
git clone https://github.com/jhen0409/react-native-debugger.git
cd react-native-debugger

# Install dependencies
npm install

# Build application
npm run build

# Run in development mode
npm start
```

### **Method 3: Using npx**
```bash
# Run without installation
npx react-native-debugger

# Or specify version
npx react-native-debugger@0.13.0
```

---

## 🚀 **Basic Usage**

### **Starting the Debugger**
```bash
# Method 1: Launch from applications menu
# macOS: Applications → React Native Debugger
# Windows: Start Menu → React Native Debugger
# Linux: Applications menu

# Method 2: Command line
react-native-debugger

# Method 3: From React Native project
npx react-native-debugger
```

### **Connecting to React Native App**
```bash
# Start Metro bundler
npx react-native start

# Run app on device/simulator
npx react-native run-android
# or
npx react-native run-ios

# Open React Native Debugger
# The app should auto-connect to localhost:8081

# Manual connection (if auto-connect fails)
# In debugger: Connection → localhost:8081
```

### **Basic Interface Overview**
```bash
# Main Panels:
✅ React DevTools (left panel)
✅ Console (bottom panel)
✅ Network (tab)
✅ Redux DevTools (if configured)
✅ AsyncStorage (tab)

# Toolbar:
✅ Reload app
✅ Toggle element inspector
✅ Toggle performance monitor
✅ Open dev menu
✅ Settings
```

### **First Debug Session**
```javascript
// Add debug code to your React Native app
import React, { useEffect } from 'react';
import { View, Text } from 'react-native';

const App = () => {
  useEffect(() => {
    console.log('App component mounted');
    debugger; // Add breakpoint
  }, []);

  const handlePress = () => {
    console.log('Button pressed');
    const data = { message: 'Hello Debugger' };
    console.log('Debug data:', data);
  };

  return (
    <View>
      <Text>React Native Debugger Test</Text>
    </View>
  );
};

export default App;
```

---

## ⚛️ **React DevTools**

### **Component Tree Inspection**
```bash
# Open Components tab
# View component hierarchy
# Click on components to inspect
# See component props and state
# Navigate through component tree
```

### **Props & State Inspection**
```bash
# Select component in tree
# View Props panel
# See current prop values
# View State panel (if using class components)
# Edit values in real-time
```

### **Component Search & Filtering**
```bash
# Use search bar to find components
# Filter by component name
# Filter by component type
# Highlight components in app
```

### **Performance Profiling**
```bash
# Open Profiler tab
# Start recording
# Interact with app
# Stop recording
# Analyze component render times
# Identify performance bottlenecks
```

### **React Hooks Debugging**
```bash
# For functional components with hooks
# View hooks in component details
# See useState values
# Track useEffect executions
# Debug custom hooks
```

---

## 🗂️ **Redux DevTools**

### **Redux Store Setup**
```javascript
// Install Redux DevTools integration
npm install redux-devtools-extension

// Configure store
import { createStore, applyMiddleware } from 'redux';
import { composeWithDevTools } from 'redux-devtools-extension';

const store = createStore(
  rootReducer,
  composeWithDevTools(
    applyMiddleware(thunk)
  )
);

export default store;
```

### **Time Travel Debugging**
```bash
# Open Redux DevTools tab
# View action history
# Click on past actions
# See state changes over time
# Jump to any point in history
# Reset to initial state
```

### **Action Inspection**
```bash
# View dispatched actions
# See action type and payload
# Track action creators
# Monitor async actions
# Debug action flow
```

### **State Tree Visualization**
```bash
# View current state tree
# Expand/collapse state branches
# Search through state
# Export/import state snapshots
# Compare state changes
```

### **Custom Middleware Debugging**
```javascript
// Add logging middleware
const logger = store => next => action => {
  console.log('Dispatching:', action);
  const result = next(action);
  console.log('Next state:', store.getState());
  return result;
};

// Use in store configuration
const store = createStore(
  rootReducer,
  composeWithDevTools(
    applyMiddleware(thunk, logger)
  )
);
```

---

## 🌐 **Network Inspection**

### **Network Tab Overview**
```bash
# Open Network tab
# View all network requests
# See request/response details
# Monitor request timing
# Filter requests by type
# Search through requests
```

### **Request Details**
```bash
# Click on any request
# View Headers tab
# See request/response headers
# View Payload tab
# See request body data
# View Response tab
# See response data
# View Timing tab
# See request timing breakdown
```

### **Network Filtering**
```bash
# Filter by request type:
✅ XHR/Fetch
✅ JS/CSS/HTML
✅ Images
✅ Fonts
✅ Other

# Filter by status code:
✅ 2xx (Success)
✅ 3xx (Redirect)
✅ 4xx (Client Error)
✅ 5xx (Server Error)
```

### **Network Throttling**
```bash
# Simulate network conditions:
✅ Online (no throttling)
✅ Slow 3G
✅ Fast 3G
✅ Regular 4G
✅ DSL
✅ WiFi
✅ Offline
```

### **API Debugging**
```javascript
// Add network debugging to your app
import { LogBox } from 'react-native';

LogBox.ignoreLogs(['Warning: ...']); // Ignore specific warnings

// Custom network logging
const originalFetch = global.fetch;
global.fetch = function(...args) {
  console.log('Network Request:', args[0]);
  return originalFetch.apply(this, args)
    .then(response => {
      console.log('Network Response:', response.status);
      return response;
    });
};
```

---

## 📊 **Performance Monitoring**

### **Performance Tab**
```bash
# Open Performance tab
# Start recording
# Interact with app
# Stop recording
# Analyze performance data
```

### **Performance Metrics**
```bash
# CPU Usage
# Memory Usage
# Network Activity
# Frame Rate (FPS)
# JavaScript Execution Time
# Render Time
```

### **Memory Leak Detection**
```bash
# Monitor memory usage over time
# Look for memory spikes
# Identify memory leaks
# Track object allocations
# Analyze garbage collection
```

### **Frame Rate Monitoring**
```bash
# Monitor app frame rate
# Identify dropped frames
# Analyze UI performance
# Track animation performance
# Debug scrolling performance
```

### **Bundle Analysis**
```bash
# Analyze JavaScript bundle size
# View module sizes
# Identify large dependencies
# Optimize bundle splitting
# Monitor bundle loading time
```

---

## 🚀 **Advanced Features**

### **Custom Plugins**
```javascript
// Create custom debugger plugin
const myPlugin = {
  name: 'My Custom Plugin',
  version: '1.0.0',
  tabs: [{
    name: 'Custom',
    component: MyCustomComponent
  }],
  reducers: {
    custom: (state = {}, action) => {
      switch (action.type) {
        case 'CUSTOM_ACTION':
          return { ...state, data: action.payload };
        default:
          return state;
      }
    }
  }
};

// Register plugin
window.registerPlugin(myPlugin);
```

### **Remote Debugging**
```bash
# Enable remote debugging
# Shake device or press Cmd+D (iOS) / Cmd+M (Android)
# Select "Debug"
# Open React Native Debugger
# Connect to remote device
```

### **Breakpoints & Stepping**
```bash
# Set breakpoints in code
# Use debugger; statement
# Conditional breakpoints
# Log points (log without stopping)
# Step over, step into, step out
# Continue execution
```

### **Source Maps**
```bash
# Enable source maps for debugging
// metro.config.js
module.exports = {
  transformer: {
    minifierConfig: {
      compress: {
        drop_console: false, // Keep console for debugging
      },
    },
  },
  serializer: {
    createModuleIdFactory: () => {
      return (path) => path;
    },
  },
};
```

### **AsyncStorage Inspection**
```bash
# Open AsyncStorage tab
# View all stored data
# Edit values in real-time
# Clear storage
# Export/import data
# Monitor storage changes
```

### **Custom Console Methods**
```javascript
// Enhanced console logging
const originalLog = console.log;
console.log = function(...args) {
  // Add timestamp
  const timestamp = new Date().toISOString();
  originalLog(`[${timestamp}]`, ...args);
};

// Custom debug methods
console.api = (endpoint, data) => {
  console.group(`🚀 API Call: ${endpoint}`);
  console.log('Data:', data);
  console.groupEnd();
};

console.error = (error) => {
  console.group('❌ Error');
  console.error(error);
  console.groupEnd();
};
```

---

## 🔧 **Troubleshooting**

### **Common Issues & Solutions**

#### **Issue 1: Connection Problems**
```bash
# Check Metro bundler is running
npx react-native start

# Verify port availability
lsof -i :8081

# Kill conflicting processes
kill -9 $(lsof -ti :8081)

# Check firewall settings
# Allow connections on port 8081

# Try different port
npx react-native start --port 8082
```

#### **Issue 2: Debugger Not Opening**
```bash
# Check installation
react-native-debugger --version

# Reinstall debugger
npm uninstall -g react-native-debugger
npm install -g react-native-debugger

# Clear cache
rm -rf ~/.react-native-debugger
```

#### **Issue 3: Redux DevTools Not Showing**
```bash
# Verify Redux DevTools setup
import { composeWithDevTools } from 'redux-devtools-extension';

# Check middleware order
const store = createStore(
  rootReducer,
  composeWithDevTools(applyMiddleware(thunk))
);

# Enable remote debugging
# Shake device → Debug → Open debugger
```

#### **Issue 4: Performance Issues**
```bash
# Disable unnecessary features
# Close unused tabs
# Reduce console logging
# Use production build for testing

# Clear debugger cache
# Close debugger
# Clear browser cache
# Restart debugger
```

#### **Issue 5: Source Maps Issues**
```bash
# Verify source maps are generated
// metro.config.js
module.exports = {
  transformer: {
    minifierConfig: {
      sourceMap: true,
    },
  },
};

# Check bundle generation
npx react-native bundle --platform android --dev false --entry-file index.js --bundle-output bundle.js --sourcemap-output bundle.map
```

### **Debugging Tips**
```bash
# Use descriptive console messages
console.log('🔍 Component mounted:', componentName);
console.log('📊 Data received:', data);
console.log('⚠️ Warning:', warningMessage);
console.error('❌ Error:', error);

# Group related logs
console.group('User Authentication');
console.log('Login attempt');
console.log('Validation passed');
console.log('API call successful');
console.groupEnd();

# Time operations
console.time('API Call');
await apiCall();
console.timeEnd('API Call');
```

---

## 📚 **Best Practices**

### **Debugging Workflow**
```bash
# 1. Start with console logging
console.log('Debug point reached');

# 2. Add breakpoints for complex logic
debugger;

# 3. Use React DevTools for component inspection
# Select component → View props/state

# 4. Monitor network requests
# Network tab → Inspect API calls

# 5. Profile performance
# Performance tab → Record and analyze

# 6. Use Redux DevTools for state debugging
# Action history → Time travel debugging
```

### **Code Organization for Debugging**
```javascript
// Use meaningful variable names
const userData = fetchUserData();
const processedData = processUserData(userData);

// Add debug comments
// TODO: Remove debug logging before production
console.log('Processing user data:', processedData);

// Use error boundaries
import React from 'react';

class ErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    return this.props.children;
  }
}
```

### **Performance Optimization**
```bash
# Minimize console usage in production
if (__DEV__) {
  console.log('Debug info');
}

// Use conditional debugging
const DEBUG = __DEV__;
if (DEBUG) {
  console.log('Debug mode enabled');
}

// Optimize bundle size
// Remove debug code in production builds
```

### **Team Collaboration**
```bash
# Share debugging sessions
# Document debugging procedures
# Create debugging checklists
# Use consistent logging formats
# Set up team debugging standards
```

---

## 🚀 **Advanced Configuration**

### **Custom Debugger Configuration**
```javascript
// .rndebuggerrc
{
  "port": 8081,
  "host": "localhost",
  "reduxDevTools": true,
  "networkInspect": true,
  "asyncStorage": true,
  "performanceMonitor": true,
  "sourceMaps": true,
  "breakpoints": true,
  "themes": {
    "dark": true,
    "light": false
  }
}
```

### **Integration with Development Tools**
```javascript
// VS Code launch configuration
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug React Native",
      "type": "reactnative",
      "request": "launch",
      "platform": "android",
      "target": "emulator",
      "internalConsoleOptions": "openOnSessionStart",
      "useHermesEngine": false
    }
  ]
}
```

### **Custom Redux Middleware**
```javascript
// Debug middleware
const debugMiddleware = store => next => action => {
  console.group(`Action: ${action.type}`);
  console.log('Previous state:', store.getState());
  console.log('Action:', action);

  const result = next(action);

  console.log('Next state:', store.getState());
  console.groupEnd();

  return result;
};

// Crash reporting middleware
const crashMiddleware = store => next => action => {
  try {
    return next(action);
  } catch (error) {
    console.error('Action crashed:', action, error);
    // Send to crash reporting service
    throw error;
  }
};
```

### **Automated Testing Integration**
```javascript
// Debug test helpers
global.debug = {
  log: (message, data) => {
    if (__DEV__) {
      console.log(`🐛 ${message}`, data);
    }
  },
  warn: (message, data) => {
    if (__DEV__) {
      console.warn(`⚠️ ${message}`, data);
    }
  },
  error: (message, data) => {
    if (__DEV__) {
      console.error(`❌ ${message}`, data);
    }
  }
};

// Usage in tests
import { debug } from '../debug';

describe('MyComponent', () => {
  it('should render correctly', () => {
    debug.log('Starting test');
    // Test code
    debug.log('Test completed');
  });
});
```

---

## 📊 **Monitoring & Analytics**

### **Debug Session Analytics**
```javascript
// Track debugging sessions
const debugSession = {
  startTime: Date.now(),
  actions: [],
  errors: [],
  networkRequests: [],
  performanceMetrics: {}
};

// Track actions
const actionTracker = store => next => action => {
  debugSession.actions.push({
    type: action.type,
    timestamp: Date.now(),
    payload: action.payload
  });
  return next(action);
};

// Export session data
const exportDebugSession = () => {
  const endTime = Date.now();
  const duration = endTime - debugSession.startTime;

  return {
    ...debugSession,
    endTime,
    duration,
    summary: {
      totalActions: debugSession.actions.length,
      totalErrors: debugSession.errors.length,
      totalRequests: debugSession.networkRequests.length
    }
  };
};
```

### **Performance Benchmarking**
```javascript
// Performance measurement
const performanceMonitor = {
  measures: {},

  start: (name) => {
    performanceMonitor.measures[name] = {
      startTime: performance.now(),
      startMemory: performance.memory?.usedJSHeapSize
    };
  },

  end: (name) => {
    const measure = performanceMonitor.measures[name];
    if (measure) {
      const endTime = performance.now();
      const endMemory = performance.memory?.usedJSHeapSize;

      console.log(`Performance: ${name}`);
      console.log(`Time: ${(endTime - measure.startTime).toFixed(2)}ms`);
      if (endMemory && measure.startMemory) {
        console.log(`Memory: ${((endMemory - measure.startMemory) / 1024 / 1024).toFixed(2)}MB`);
      }
    }
  }
};

// Usage
performanceMonitor.start('component-render');
renderComponent();
performanceMonitor.end('component-render');
```

---

## 🎯 **Conclusion**

React Native Debugger is an essential tool for React Native development, providing powerful debugging capabilities and development tools. Proper setup and usage can significantly improve development efficiency and app quality.

### **Key Takeaways:**
- ✅ Install React Native Debugger for your platform
- ✅ Connect debugger to React Native apps
- ✅ Use React DevTools for component inspection
- ✅ Configure Redux DevTools for state debugging
- ✅ Monitor network requests and performance
- ✅ Follow debugging best practices

### **Next Steps:**
1. **Install**: Set up React Native Debugger on your system
2. **Connect**: Link debugger to your React Native projects
3. **Explore**: Learn React DevTools and Redux DevTools
4. **Debug**: Practice debugging common React Native issues
5. **Optimize**: Use performance monitoring tools
6. **Customize**: Create custom debugging workflows

### **Additional Resources:**
- [React Native Debugger GitHub](https://github.com/jhen0409/react-native-debugger)
- [React DevTools Documentation](https://react.dev/learn/react-developer-tools)
- [Redux DevTools Documentation](https://github.com/reduxjs/redux-devtools)
- [React Native Debugging Guide](https://reactnative.dev/docs/debugging)
- [Performance Monitoring](https://reactnative.dev/docs/performance)

---

*Happy Debugging! 🔍🐛*