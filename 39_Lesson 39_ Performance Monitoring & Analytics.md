# Lesson 39: Performance Monitoring & Analytics

## 🎯 **Learning Objectives**
- Implement comprehensive performance monitoring
- Set up analytics tracking and reporting
- Monitor app crashes and errors
- Track user behavior and engagement
- Optimize app performance based on metrics

## 📚 **Table of Contents**
1. [Performance Monitoring Setup](#performance-monitoring-setup)
2. [Crash Reporting & Error Tracking](#crash-reporting--error-tracking)
3. [Analytics Implementation](#analytics-implementation)
4. [User Behavior Tracking](#user-behavior-tracking)
5. [Performance Metrics](#performance-metrics)
6. [A/B Testing](#ab-testing)
7. [Real-time Monitoring](#real-time-monitoring)
8. [Custom Dashboards](#custom-dashboards)
9. [Alerting & Notifications](#alerting--notifications)
10. [Practical Examples](#practical-examples)

---

## 📊 **Performance Monitoring Setup**

### **React Native Performance Monitor**
```javascript
// services/performanceMonitor.js
import { PerformanceObserver, performance } from 'react-native';

class PerformanceMonitor {
  constructor() {
    this.metrics = new Map();
    this.observers = new Map();
    this.initializeObservers();
  }

  initializeObservers() {
    // Observe navigation performance
    const navObserver = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        this.recordMetric('navigation', {
          name: entry.name,
          duration: entry.duration,
          startTime: entry.startTime,
        });
      }
    });

    navObserver.observe({ entryTypes: ['navigation'] });

    // Observe resource loading
    const resourceObserver = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (entry.duration > 1000) { // Log slow resources
          this.recordMetric('slow_resource', {
            name: entry.name,
            duration: entry.duration,
            size: entry.transferSize,
          });
        }
      }
    });

    resourceObserver.observe({ entryTypes: ['resource'] });

    this.observers.set('navigation', navObserver);
    this.observers.set('resource', resourceObserver);
  }

  // Record custom metrics
  recordMetric(name, data) {
    const timestamp = Date.now();
    const metric = {
      name,
      data,
      timestamp,
      sessionId: this.getSessionId(),
    };

    // Store locally
    this.storeMetric(metric);

    // Send to analytics service
    this.sendToAnalytics(metric);
  }

  // Measure function execution time
  measureExecutionTime(fn, name) {
    return async (...args) => {
      const startTime = performance.now();
      try {
        const result = await fn(...args);
        const duration = performance.now() - startTime;

        this.recordMetric('function_execution', {
          name,
          duration,
          success: true,
        });

        return result;
      } catch (error) {
        const duration = performance.now() - startTime;

        this.recordMetric('function_execution', {
          name,
          duration,
          success: false,
          error: error.message,
        });

        throw error;
      }
    };
  }

  // Track component render time
  trackComponentRender(Component, componentName) {
    return React.forwardRef((props, ref) => {
      const renderStart = performance.now();

      React.useEffect(() => {
        const renderTime = performance.now() - renderStart;

        if (renderTime > 16.67) { // Slower than 60fps
          this.recordMetric('slow_render', {
            component: componentName,
            renderTime,
          });
        }
      });

      return <Component {...props} ref={ref} />;
    });
  }

  // Memory usage tracking
  trackMemoryUsage() {
    if (global.gc) {
      global.gc();
    }

    // Note: Memory tracking is limited in React Native
    // Use native modules for more detailed memory tracking
    this.recordMetric('memory_check', {
      timestamp: Date.now(),
    });
  }

  // Network request tracking
  trackNetworkRequest(url, method, duration, status) {
    this.recordMetric('network_request', {
      url,
      method,
      duration,
      status,
      timestamp: Date.now(),
    });
  }

  getSessionId() {
    // Generate or retrieve session ID
    return 'session_' + Date.now();
  }

  storeMetric(metric) {
    // Store in local storage for offline sync
    const metrics = JSON.parse(AsyncStorage.getItem('performance_metrics') || '[]');
    metrics.push(metric);

    // Keep only last 1000 metrics
    if (metrics.length > 1000) {
      metrics.splice(0, metrics.length - 1000);
    }

    AsyncStorage.setItem('performance_metrics', JSON.stringify(metrics));
  }

  sendToAnalytics(metric) {
    // Send to your analytics service
    // Example: Firebase Analytics, Mixpanel, etc.
    console.log('Sending metric to analytics:', metric);
  }

  getMetricsReport() {
    const metrics = JSON.parse(AsyncStorage.getItem('performance_metrics') || '[]');

    return {
      totalMetrics: metrics.length,
      slowRenders: metrics.filter(m => m.name === 'slow_render').length,
      networkErrors: metrics.filter(m => m.name === 'network_request' && m.data.status >= 400).length,
      averageRenderTime: this.calculateAverageRenderTime(metrics),
    };
  }

  calculateAverageRenderTime(metrics) {
    const renderMetrics = metrics.filter(m => m.name === 'slow_render');
    if (renderMetrics.length === 0) return 0;

    const totalTime = renderMetrics.reduce((sum, m) => sum + m.data.renderTime, 0);
    return totalTime / renderMetrics.length;
  }
}

export default new PerformanceMonitor();
```

### **Custom Performance Hook**
```javascript
// hooks/usePerformance.js
import { useEffect, useRef, useCallback } from 'react';
import performanceMonitor from '../services/performanceMonitor';

export const usePerformance = (componentName) => {
  const renderCount = useRef(0);
  const mountTime = useRef(performance.now());

  useEffect(() => {
    renderCount.current += 1;

    // Track component mount
    if (renderCount.current === 1) {
      performanceMonitor.recordMetric('component_mount', {
        component: componentName,
        timestamp: Date.now(),
      });
    }

    // Track re-renders
    if (renderCount.current > 1) {
      performanceMonitor.recordMetric('component_rerender', {
        component: componentName,
        renderCount: renderCount.current,
      });
    }
  });

  // Track function execution
  const trackFunction = useCallback((fn, functionName) => {
    return performanceMonitor.measureExecutionTime(fn, `${componentName}.${functionName}`);
  }, [componentName]);

  // Track user interactions
  const trackInteraction = useCallback((interactionType, data = {}) => {
    performanceMonitor.recordMetric('user_interaction', {
      component: componentName,
      type: interactionType,
      ...data,
    });
  }, [componentName]);

  useEffect(() => {
    return () => {
      // Track component unmount
      const lifetime = performance.now() - mountTime.current;
      performanceMonitor.recordMetric('component_unmount', {
        component: componentName,
        lifetime,
      });
    };
  }, [componentName]);

  return {
    trackFunction,
    trackInteraction,
    renderCount: renderCount.current,
  };
};
```

---

## 🚨 **Crash Reporting & Error Tracking**

### **Sentry Integration**
```javascript
// services/sentry.js
import * as Sentry from '@sentry/react-native';

const initializeSentry = () => {
  Sentry.init({
    dsn: 'your-sentry-dsn',
    environment: __DEV__ ? 'development' : 'production',
    debug: __DEV__,
    enableAutoSessionTracking: true,
    sessionTrackingIntervalMillis: 30000,
    enableAutoPerformanceTracking: true,
    enableTracing: true,
    tracesSampleRate: __DEV__ ? 1.0 : 0.1,
    integrations: [
      new Sentry.ReactNativeTracing({
        routingInstrumentation: Sentry.reactNavigationIntegration(),
      }),
    ],
  });

  // Set user context
  Sentry.setUser({
    id: 'user-id',
    email: 'user@example.com',
  });

  // Set tags
  Sentry.setTag('app_version', '1.0.0');
  Sentry.setTag('platform', Platform.OS);
};

// Error boundary for React components
class ErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    Sentry.captureException(error, {
      contexts: {
        react: {
          componentStack: errorInfo.componentStack,
        },
      },
    });
  }

  render() {
    return this.props.children;
  }
}

// Custom error handler
const handleError = (error, context = {}) => {
  Sentry.captureException(error, {
    tags: {
      context: context.name || 'unknown',
    },
    extra: context,
  });
};

// Performance monitoring
const startTransaction = (name, op) => {
  return Sentry.startTransaction({
    name,
    op,
  });
};

// Network request monitoring
const trackNetworkRequest = (url, method, status, duration) => {
  Sentry.addBreadcrumb({
    category: 'http',
    message: `${method} ${url}`,
    level: status >= 400 ? 'error' : 'info',
    data: {
      status,
      duration,
    },
  });

  if (status >= 500) {
    Sentry.captureMessage(`Server error: ${status} ${method} ${url}`, 'error');
  }
};

export {
  initializeSentry,
  ErrorBoundary,
  handleError,
  startTransaction,
  trackNetworkRequest,
};
```

### **Firebase Crashlytics**
```javascript
// services/crashlytics.js
import crashlytics from '@react-native-firebase/crashlytics';

class CrashReporter {
  static initialize() {
    // Enable crashlytics collection
    crashlytics().setCrashlyticsCollectionEnabled(true);
  }

  static setUserId(userId) {
    crashlytics().setUserId(userId);
  }

  static setUserEmail(email) {
    crashlytics().setAttribute('email', email);
  }

  static setUserName(name) {
    crashlytics().setAttribute('name', name);
  }

  static log(message) {
    crashlytics().log(message);
  }

  static recordError(error, context) {
    crashlytics().recordError(error, context);
  }

  static setAttribute(key, value) {
    crashlytics().setAttribute(key, value);
  }

  static setAttributes(attributes) {
    Object.entries(attributes).forEach(([key, value]) => {
      crashlytics().setAttribute(key, value);
    });
  }

  // Track custom events
  static logEvent(eventName, parameters = {}) {
    crashlytics().log(`${eventName}: ${JSON.stringify(parameters)}`);
  }

  // Track navigation
  static logNavigation(screenName, parameters = {}) {
    this.logEvent('navigation', {
      screen: screenName,
      ...parameters,
    });
  }

  // Track user actions
  static logUserAction(action, parameters = {}) {
    this.logEvent('user_action', {
      action,
      ...parameters,
    });
  }

  // Track performance
  static logPerformance(metricName, value, unit = 'ms') {
    crashlytics().setAttribute(`perf_${metricName}`, `${value}${unit}`);
  }
}

// Error boundary with Crashlytics
class CrashlyticsErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    CrashReporter.recordError(error, {
      componentStack: errorInfo.componentStack,
      timestamp: new Date().toISOString(),
    });
  }

  render() {
    return this.props.children;
  }
}

export { CrashReporter, CrashlyticsErrorBoundary };
```

---

## 📈 **Analytics Implementation**

### **Firebase Analytics**
```javascript
// services/analytics.js
import analytics from '@react-native-firebase/analytics';
import { Platform } from 'react-native';

class AnalyticsService {
  static async initialize() {
    await analytics().setAnalyticsCollectionEnabled(true);
  }

  // User properties
  static async setUserId(userId) {
    await analytics().setUserId(userId);
  }

  static async setUserProperty(name, value) {
    await analytics().setUserProperty(name, value);
  }

  static async setUserProperties(properties) {
    for (const [key, value] of Object.entries(properties)) {
      await this.setUserProperty(key, value);
    }
  }

  // Events
  static async logEvent(eventName, parameters = {}) {
    try {
      await analytics().logEvent(eventName, {
        platform: Platform.OS,
        timestamp: Date.now(),
        ...parameters,
      });
    } catch (error) {
      console.error('Analytics error:', error);
    }
  }

  // Screen tracking
  static async logScreenView(screenName, screenClass = screenName) {
    await analytics().logScreenView({
      screen_name: screenName,
      screen_class: screenClass,
    });
  }

  // E-commerce events
  static async logPurchase(transactionId, value, currency = 'USD', items = []) {
    await analytics().logPurchase({
      transaction_id: transactionId,
      value,
      currency,
      items,
    });
  }

  static async logAddToCart(item) {
    await analytics().logAddToCart({
      items: [item],
    });
  }

  static async logBeginCheckout(value, currency = 'USD', items = []) {
    await analytics().logBeginCheckout({
      value,
      currency,
      items,
    });
  }

  // Custom events
  static async logCustomEvent(eventName, parameters = {}) {
    await this.logEvent(eventName, parameters);
  }

  // App lifecycle events
  static async logAppOpen() {
    await analytics().logAppOpen();
  }

  static async logAppUpdate(previousVersion, currentVersion) {
    await this.logEvent('app_update', {
      previous_version: previousVersion,
      current_version: currentVersion,
    });
  }

  // Error tracking
  static async logError(error, context = {}) {
    await this.logEvent('error', {
      error_message: error.message,
      error_stack: error.stack,
      context: JSON.stringify(context),
    });
  }

  // Performance tracking
  static async logPerformance(metricName, value, unit = 'ms') {
    await this.logEvent('performance', {
      metric_name: metricName,
      value,
      unit,
    });
  }

  // User engagement
  static async logUserEngagement(action, parameters = {}) {
    await this.logEvent('user_engagement', {
      action,
      ...parameters,
    });
  }

  // Search tracking
  static async logSearch(searchTerm, resultsCount = 0) {
    await analytics().logSearch({
      search_term: searchTerm,
    });

    await this.logEvent('search_performed', {
      search_term: searchTerm,
      results_count: resultsCount,
    });
  }
}

export default AnalyticsService;
```

### **Mixpanel Integration**
```javascript
// services/mixpanel.js
import { Mixpanel } from 'mixpanel-react-native';

class MixpanelService {
  constructor() {
    this.mixpanel = null;
    this.initialized = false;
  }

  async initialize(token) {
    if (this.initialized) return;

    this.mixpanel = new Mixpanel(token, true);
    this.mixpanel.init();

    this.initialized = true;
  }

  // User identification
  identify(userId) {
    if (!this.mixpanel) return;
    this.mixpanel.identify(userId);
  }

  setUserProperties(properties) {
    if (!this.mixpanel) return;
    this.mixpanel.getPeople().set(properties);
  }

  // Event tracking
  track(eventName, properties = {}) {
    if (!this.mixpanel) return;

    this.mixpanel.track(eventName, {
      timestamp: new Date().toISOString(),
      platform: Platform.OS,
      ...properties,
    });
  }

  // Screen tracking
  trackScreen(screenName, properties = {}) {
    this.track('Screen View', {
      screen_name: screenName,
      ...properties,
    });
  }

  // User actions
  trackUserAction(action, properties = {}) {
    this.track('User Action', {
      action,
      ...properties,
    });
  }

  // E-commerce events
  trackPurchase(productId, price, currency = 'USD') {
    this.track('Purchase', {
      product_id: productId,
      price,
      currency,
    });
  }

  trackAddToCart(productId, quantity = 1) {
    this.track('Add to Cart', {
      product_id: productId,
      quantity,
    });
  }

  // Error tracking
  trackError(error, context = {}) {
    this.track('Error', {
      error_message: error.message,
      error_stack: error.stack,
      context: JSON.stringify(context),
    });
  }

  // Performance tracking
  trackPerformance(metricName, value, unit = 'ms') {
    this.track('Performance', {
      metric_name: metricName,
      value,
      unit,
    });
  }

  // Funnel analysis
  trackFunnelStep(stepName, funnelName, stepNumber) {
    this.track('Funnel Step', {
      funnel_name: funnelName,
      step_name: stepName,
      step_number: stepNumber,
    });
  }

  // A/B Testing
  trackABTest(experimentName, variantName) {
    this.track('A/B Test', {
      experiment_name: experimentName,
      variant_name: variantName,
    });
  }

  // Revenue tracking
  trackRevenue(amount, currency = 'USD', properties = {}) {
    if (!this.mixpanel) return;

    this.mixpanel.getPeople().trackCharge(amount, {
      currency,
      ...properties,
    });
  }

  // Push notification tracking
  trackPushNotification(action, campaignId, properties = {}) {
    this.track('Push Notification', {
      action, // received, opened, dismissed
      campaign_id: campaignId,
      ...properties,
    });
  }

  // Time tracking
  timeEvent(eventName) {
    if (!this.mixpanel) return;
    this.mixpanel.timeEvent(eventName);
  }

  // Flush events
  flush() {
    if (!this.mixpanel) return;
    this.mixpanel.flush();
  }

  // Reset
  reset() {
    if (!this.mixpanel) return;
    this.mixpanel.reset();
  }
}

export default new MixpanelService();
```

---

## 👤 **User Behavior Tracking**

### **Navigation Tracking**
```javascript
// services/navigationTracker.js
import AnalyticsService from './analytics';

class NavigationTracker {
  constructor() {
    this.currentScreen = null;
    this.screenStartTime = null;
    this.sessionStartTime = Date.now();
    this.screenHistory = [];
  }

  // Track screen view
  trackScreenView(screenName, parameters = {}) {
    // Track previous screen duration
    if (this.currentScreen && this.screenStartTime) {
      const duration = Date.now() - this.screenStartTime;
      AnalyticsService.logEvent('screen_duration', {
        screen_name: this.currentScreen,
        duration,
      });
    }

    // Update current screen
    this.currentScreen = screenName;
    this.screenStartTime = Date.now();

    // Track screen view
    AnalyticsService.logScreenView(screenName, parameters);

    // Add to history
    this.screenHistory.push({
      screen: screenName,
      timestamp: Date.now(),
      parameters,
    });

    // Keep only last 50 screens
    if (this.screenHistory.length > 50) {
      this.screenHistory.shift();
    }
  }

  // Track navigation action
  trackNavigation(fromScreen, toScreen, method = 'push', parameters = {}) {
    AnalyticsService.logEvent('navigation', {
      from_screen: fromScreen,
      to_screen: toScreen,
      method,
      ...parameters,
    });
  }

  // Track user flow
  trackUserFlow(flowName, step, totalSteps) {
    AnalyticsService.logEvent('user_flow', {
      flow_name: flowName,
      step,
      total_steps: totalSteps,
      progress_percentage: (step / totalSteps) * 100,
    });
  }

  // Track session
  trackSessionEnd() {
    const sessionDuration = Date.now() - this.sessionStartTime;

    AnalyticsService.logEvent('session_end', {
      session_duration: sessionDuration,
      screens_viewed: this.screenHistory.length,
      last_screen: this.currentScreen,
    });
  }

  // Get navigation insights
  getNavigationInsights() {
    const screenCounts = {};
    this.screenHistory.forEach(entry => {
      screenCounts[entry.screen] = (screenCounts[entry.screen] || 0) + 1;
    });

    return {
      totalScreensViewed: this.screenHistory.length,
      uniqueScreens: Object.keys(screenCounts).length,
      mostViewedScreen: Object.entries(screenCounts).reduce((a, b) =>
        screenCounts[a[0]] > screenCounts[b[0]] ? a : b
      )[0],
      screenCounts,
      sessionDuration: Date.now() - this.sessionStartTime,
    };
  }
}

export default new NavigationTracker();
```

### **Interaction Tracking**
```javascript
// hooks/useInteractionTracking.js
import { useCallback, useEffect } from 'react';
import AnalyticsService from '../services/analytics';

export const useInteractionTracking = (componentName) => {
  // Track button clicks
  const trackButtonClick = useCallback((buttonName, parameters = {}) => {
    AnalyticsService.logEvent('button_click', {
      component: componentName,
      button_name: buttonName,
      ...parameters,
    });
  }, [componentName]);

  // Track form interactions
  const trackFormInteraction = useCallback((action, fieldName, parameters = {}) => {
    AnalyticsService.logEvent('form_interaction', {
      component: componentName,
      action, // focus, blur, change, submit
      field_name: fieldName,
      ...parameters,
    });
  }, [componentName]);

  // Track scroll events
  const trackScroll = useCallback((scrollPosition, parameters = {}) => {
    AnalyticsService.logEvent('scroll', {
      component: componentName,
      scroll_position: scrollPosition,
      ...parameters,
    });
  }, [componentName]);

  // Track gestures
  const trackGesture = useCallback((gestureType, parameters = {}) => {
    AnalyticsService.logEvent('gesture', {
      component: componentName,
      gesture_type: gestureType,
      ...parameters,
    });
  }, [componentName]);

  // Track time spent
  useEffect(() => {
    const startTime = Date.now();

    return () => {
      const timeSpent = Date.now() - startTime;
      AnalyticsService.logEvent('time_spent', {
        component: componentName,
        time_spent: timeSpent,
      });
    };
  }, [componentName]);

  return {
    trackButtonClick,
    trackFormInteraction,
    trackScroll,
    trackGesture,
  };
};
```

---

## 📊 **Performance Metrics**

### **Custom Performance Metrics**
```javascript
// services/performanceMetrics.js
import AnalyticsService from './analytics';

class PerformanceMetrics {
  constructor() {
    this.metrics = new Map();
  }

  // App startup time
  trackAppStartup(startTime) {
    const startupTime = Date.now() - startTime;

    AnalyticsService.logPerformance('app_startup', startupTime);

    this.storeMetric('app_startup', startupTime);
  }

  // Screen load time
  trackScreenLoad(screenName, loadTime) {
    AnalyticsService.logPerformance('screen_load', loadTime, {
      screen_name: screenName,
    });

    this.storeMetric(`screen_load_${screenName}`, loadTime);
  }

  // API response time
  trackApiResponse(endpoint, method, responseTime, status) {
    AnalyticsService.logPerformance('api_response', responseTime, {
      endpoint,
      method,
      status,
    });

    this.storeMetric(`api_${endpoint}_${method}`, responseTime);
  }

  // Image load time
  trackImageLoad(imageUrl, loadTime) {
    AnalyticsService.logPerformance('image_load', loadTime, {
      image_url: imageUrl,
    });
  }

  // Memory usage
  trackMemoryUsage(usage) {
    AnalyticsService.logEvent('memory_usage', {
      usage_mb: usage,
      timestamp: Date.now(),
    });
  }

  // Battery level
  trackBatteryLevel(level, isCharging) {
    AnalyticsService.logEvent('battery_level', {
      level_percentage: level,
      is_charging: isCharging,
    });
  }

  // Network quality
  trackNetworkQuality(type, speed) {
    AnalyticsService.logEvent('network_quality', {
      type,
      speed_mbps: speed,
    });
  }

  // FPS tracking
  trackFPS(fps) {
    AnalyticsService.logEvent('fps', {
      fps,
      timestamp: Date.now(),
    });
  }

  // Custom metric
  trackCustomMetric(name, value, unit = '', metadata = {}) {
    AnalyticsService.logEvent('custom_metric', {
      metric_name: name,
      value,
      unit,
      ...metadata,
    });

    this.storeMetric(name, value, metadata);
  }

  // Store metric locally
  storeMetric(name, value, metadata = {}) {
    const metric = {
      name,
      value,
      timestamp: Date.now(),
      ...metadata,
    };

    this.metrics.set(name, metric);

    // Keep only last 100 metrics per type
    if (this.metrics.size > 100) {
      const firstKey = this.metrics.keys().next().value;
      this.metrics.delete(firstKey);
    }
  }

  // Get metrics report
  getMetricsReport() {
    const report = {
      totalMetrics: this.metrics.size,
      averageValues: {},
      peakValues: {},
    };

    // Calculate averages and peaks
    const metricGroups = {};
    this.metrics.forEach((metric, name) => {
      if (!metricGroups[name]) {
        metricGroups[name] = [];
      }
      metricGroups[name].push(metric.value);
    });

    Object.entries(metricGroups).forEach(([name, values]) => {
      report.averageValues[name] = values.reduce((a, b) => a + b, 0) / values.length;
      report.peakValues[name] = Math.max(...values);
    });

    return report;
  }

  // Export metrics
  exportMetrics() {
    return Array.from(this.metrics.values());
  }
}

export default new PerformanceMetrics();
```

### **Real-time Performance Monitoring**
```javascript
// services/realtimeMonitor.js
import { DeviceEventEmitter } from 'react-native';

class RealtimeMonitor {
  constructor() {
    this.listeners = new Set();
    this.isMonitoring = false;
    this.monitoringInterval = null;
  }

  startMonitoring(interval = 5000) {
    if (this.isMonitoring) return;

    this.isMonitoring = true;

    this.monitoringInterval = setInterval(() => {
      this.collectMetrics();
    }, interval);
  }

  stopMonitoring() {
    this.isMonitoring = false;

    if (this.monitoringInterval) {
      clearInterval(this.monitoringInterval);
      this.monitoringInterval = null;
    }
  }

  addListener(callback) {
    this.listeners.add(callback);

    return () => {
      this.listeners.delete(callback);
    };
  }

  notifyListeners(metrics) {
    this.listeners.forEach(callback => {
      try {
        callback(metrics);
      } catch (error) {
        console.error('Error in metrics listener:', error);
      }
    });
  }

  async collectMetrics() {
    const metrics = {
      timestamp: Date.now(),
      memory: await this.getMemoryUsage(),
      battery: await this.getBatteryLevel(),
      network: await this.getNetworkInfo(),
      fps: this.getCurrentFPS(),
    };

    this.notifyListeners(metrics);

    // Send to analytics if thresholds exceeded
    this.checkThresholds(metrics);
  }

  async getMemoryUsage() {
    // Use native module for memory info
    return {
      used: 0, // Implement with native module
      total: 0,
      percentage: 0,
    };
  }

  async getBatteryLevel() {
    // Use BatteryManager or native module
    return {
      level: 0,
      isCharging: false,
    };
  }

  async getNetworkInfo() {
    // Use NetInfo
    return {
      type: 'unknown',
      isConnected: true,
    };
  }

  getCurrentFPS() {
    // Calculate FPS based on frame timing
    return 60; // Placeholder
  }

  checkThresholds(metrics) {
    // Check memory usage
    if (metrics.memory.percentage > 80) {
      AnalyticsService.logEvent('high_memory_usage', {
        percentage: metrics.memory.percentage,
      });
    }

    // Check battery level
    if (metrics.battery.level < 20 && !metrics.battery.isCharging) {
      AnalyticsService.logEvent('low_battery', {
        level: metrics.battery.level,
      });
    }

    // Check FPS
    if (metrics.fps < 30) {
      AnalyticsService.logEvent('low_fps', {
        fps: metrics.fps,
      });
    }
  }

  // Emergency monitoring
  startEmergencyMonitoring() {
    // Monitor for critical issues
    const emergencyListener = (metrics) => {
      if (metrics.memory.percentage > 90) {
        // Trigger emergency cleanup
        this.triggerEmergencyCleanup();
      }
    };

    return this.addListener(emergencyListener);
  }

  triggerEmergencyCleanup() {
    console.warn('Emergency cleanup triggered');

    // Clear caches
    // Force garbage collection
    // Reduce image quality
    // Disable non-essential features

    AnalyticsService.logEvent('emergency_cleanup', {
      timestamp: Date.now(),
    });
  }
}

export default new RealtimeMonitor();
```

---

## 🧪 **A/B Testing**

### **A/B Testing Framework**
```javascript
// services/abTesting.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import AnalyticsService from './analytics';

class ABTesting {
  constructor() {
    this.experiments = new Map();
    this.userVariants = new Map();
  }

  // Define experiment
  defineExperiment(experimentName, variants, weights = null) {
    this.experiments.set(experimentName, {
      variants,
      weights: weights || variants.map(() => 1), // Equal weights by default
    });
  }

  // Get variant for user
  getVariant(experimentName, userId = null) {
    // Check if variant already assigned
    const cacheKey = `${experimentName}_${userId || 'anonymous'}`;
    let assignedVariant = this.userVariants.get(cacheKey);

    if (!assignedVariant) {
      assignedVariant = this.assignVariant(experimentName, userId);
      this.userVariants.set(cacheKey, assignedVariant);

      // Store assignment
      this.storeVariantAssignment(cacheKey, assignedVariant);
    }

    return assignedVariant;
  }

  // Assign variant based on weights
  assignVariant(experimentName, userId) {
    const experiment = this.experiments.get(experimentName);
    if (!experiment) {
      console.warn(`Experiment ${experimentName} not defined`);
      return experiment.variants[0]; // Return first variant as fallback
    }

    const { variants, weights } = experiment;

    // Use user ID for consistent assignment
    const hash = this.hashString(userId || 'anonymous');
    const totalWeight = weights.reduce((sum, weight) => sum + weight, 0);
    const normalizedHash = (hash % totalWeight);

    let cumulativeWeight = 0;
    for (let i = 0; i < variants.length; i++) {
      cumulativeWeight += weights[i];
      if (normalizedHash < cumulativeWeight) {
        return variants[i];
      }
    }

    return variants[0]; // Fallback
  }

  // Hash function for consistent assignment
  hashString(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convert to 32-bit integer
    }
    return Math.abs(hash);
  }

  // Store variant assignment
  async storeVariantAssignment(cacheKey, variant) {
    try {
      const assignments = JSON.parse(
        await AsyncStorage.getItem('ab_test_assignments') || '{}'
      );
      assignments[cacheKey] = variant;
      await AsyncStorage.setItem('ab_test_assignments', JSON.stringify(assignments));
    } catch (error) {
      console.error('Error storing variant assignment:', error);
    }
  }

  // Load stored assignments
  async loadStoredAssignments() {
    try {
      const assignments = JSON.parse(
        await AsyncStorage.getItem('ab_test_assignments') || '{}'
      );

      Object.entries(assignments).forEach(([cacheKey, variant]) => {
        this.userVariants.set(cacheKey, variant);
      });
    } catch (error) {
      console.error('Error loading stored assignments:', error);
    }
  }

  // Track experiment events
  trackEvent(experimentName, eventName, properties = {}) {
    const variant = this.getVariant(experimentName);

    AnalyticsService.logEvent(`ab_test_${eventName}`, {
      experiment_name: experimentName,
      variant,
      ...properties,
    });
  }

  // Get experiment results
  getExperimentResults(experimentName) {
    // This would typically fetch from analytics service
    // For now, return mock data
    return {
      experimentName,
      variants: this.experiments.get(experimentName)?.variants || [],
      winner: null, // Would be determined by analytics
      confidence: 0,
    };
  }

  // End experiment
  endExperiment(experimentName, winnerVariant) {
    AnalyticsService.logEvent('experiment_ended', {
      experiment_name: experimentName,
      winner_variant: winnerVariant,
    });

    // Remove from active experiments
    this.experiments.delete(experimentName);
  }
}

// Usage example
const abTesting = new ABTesting();

// Define experiments
abTesting.defineExperiment('button_color', ['blue', 'green', 'red'], [0.5, 0.3, 0.2]);
abTesting.defineExperiment('checkout_flow', ['single_step', 'multi_step']);

// Get variant for user
const buttonColor = abTesting.getVariant('button_color', userId);

// Track events
abTesting.trackEvent('button_color', 'button_click', { button_color: buttonColor });

export default abTesting;
```

---

## 📊 **Custom Dashboards**

### **Analytics Dashboard Component**
```javascript
// components/AnalyticsDashboard.js
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, StyleSheet, RefreshControl } from 'react-native';
import AnalyticsService from '../services/analytics';
import PerformanceMetrics from '../services/performanceMetrics';

const AnalyticsDashboard = () => {
  const [metrics, setMetrics] = useState({});
  const [isLoading, setIsLoading] = useState(true);
  const [lastUpdated, setLastUpdated] = useState(null);

  useEffect(() => {
    loadDashboardData();
  }, []);

  const loadDashboardData = async () => {
    setIsLoading(true);

    try {
      // Load local metrics
      const performanceReport = PerformanceMetrics.getMetricsReport();

      // Load analytics data (would come from your analytics service)
      const analyticsData = await loadAnalyticsData();

      setMetrics({
        performance: performanceReport,
        analytics: analyticsData,
      });

      setLastUpdated(new Date());
    } catch (error) {
      console.error('Error loading dashboard data:', error);
    } finally {
      setIsLoading(false);
    }
  };

  const loadAnalyticsData = async () => {
    // This would fetch from your analytics service
    return {
      totalUsers: 1250,
      activeUsers: 890,
      averageSessionDuration: 245, // seconds
      topScreens: [
        { name: 'Home', views: 1250 },
        { name: 'Profile', views: 890 },
        { name: 'Settings', views: 456 },
      ],
      conversionRate: 3.2,
      crashRate: 0.1,
    };
  };

  const formatDuration = (seconds) => {
    const minutes = Math.floor(seconds / 60);
    const remainingSeconds = seconds % 60;
    return `${minutes}m ${remainingSeconds}s`;
  };

  const formatPercentage = (value) => {
    return `${value.toFixed(1)}%`;
  };

  return (
    <ScrollView
      style={styles.container}
      refreshControl={
        <RefreshControl refreshing={isLoading} onRefresh={loadDashboardData} />
      }
    >
      <Text style={styles.title}>Analytics Dashboard</Text>

      {lastUpdated && (
        <Text style={styles.lastUpdated}>
          Last updated: {lastUpdated.toLocaleString()}
        </Text>
      )}

      {/* Performance Metrics */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Performance Metrics</Text>

        <View style={styles.metricGrid}>
          <View style={styles.metricCard}>
            <Text style={styles.metricValue}>
              {metrics.performance?.totalMetrics || 0}
            </Text>
            <Text style={styles.metricLabel}>Total Metrics</Text>
          </View>

          <View style={styles.metricCard}>
            <Text style={styles.metricValue}>
              {metrics.performance?.slowRenders || 0}
            </Text>
            <Text style={styles.metricLabel}>Slow Renders</Text>
          </View>

          <View style={styles.metricCard}>
            <Text style={styles.metricValue}>
              {formatDuration(metrics.analytics?.averageSessionDuration || 0)}
            </Text>
            <Text style={styles.metricLabel}>Avg Session</Text>
          </View>

          <View style={styles.metricCard}>
            <Text style={styles.metricValue}>
              {formatPercentage(metrics.analytics?.crashRate || 0)}
            </Text>
            <Text style={styles.metricLabel}>Crash Rate</Text>
          </View>
        </View>
      </View>

      {/* User Analytics */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>User Analytics</Text>

        <View style={styles.metricGrid}>
          <View style={styles.metricCard}>
            <Text style={styles.metricValue}>
              {metrics.analytics?.totalUsers || 0}
            </Text>
            <Text style={styles.metricLabel}>Total Users</Text>
          </View>

          <View style={styles.metricCard}>
            <Text style={styles.metricValue}>
              {metrics.analytics?.activeUsers || 0}
            </Text>
            <Text style={styles.metricLabel}>Active Users</Text>
          </View>

          <View style={styles.metricCard}>
            <Text style={styles.metricValue}>
              {formatPercentage(metrics.analytics?.conversionRate || 0)}
            </Text>
            <Text style={styles.metricLabel}>Conversion</Text>
          </View>
        </View>
      </View>

      {/* Top Screens */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Top Screens</Text>

        {metrics.analytics?.topScreens?.map((screen, index) => (
          <View key={screen.name} style={styles.screenItem}>
            <Text style={styles.screenRank}>#{index + 1}</Text>
            <Text style={styles.screenName}>{screen.name}</Text>
            <Text style={styles.screenViews}>{screen.views} views</Text>
          </View>
        ))}
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
    margin: 20,
  },
  lastUpdated: {
    textAlign: 'center',
    color: '#666',
    marginBottom: 20,
  },
  section: {
    backgroundColor: 'white',
    margin: 10,
    borderRadius: 10,
    padding: 15,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
  },
  metricGrid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-between',
  },
  metricCard: {
    width: '48%',
    backgroundColor: '#f8f9fa',
    padding: 15,
    borderRadius: 8,
    marginBottom: 10,
    alignItems: 'center',
  },
  metricValue: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#007AFF',
  },
  metricLabel: {
    fontSize: 12,
    color: '#666',
    marginTop: 5,
  },
  screenItem: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 10,
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  screenRank: {
    fontSize: 16,
    fontWeight: 'bold',
    width: 30,
  },
  screenName: {
    flex: 1,
    fontSize: 16,
  },
  screenViews: {
    fontSize: 14,
    color: '#666',
  },
});

export default AnalyticsDashboard;
```

---

## 🚨 **Alerting & Notifications**

### **Alert System**
```javascript
// services/alertSystem.js
import PushNotification from 'react-native-push-notification';
import AnalyticsService from './analytics';

class AlertSystem {
  constructor() {
    this.alerts = new Map();
    this.thresholds = new Map();
  }

  // Define alert thresholds
  setThreshold(metricName, threshold, condition = 'greater') {
    this.thresholds.set(metricName, { threshold, condition });
  }

  // Check metric against thresholds
  checkThreshold(metricName, value) {
    const threshold = this.thresholds.get(metricName);
    if (!threshold) return false;

    const { threshold: limit, condition } = threshold;

    switch (condition) {
      case 'greater':
        return value > limit;
      case 'less':
        return value < limit;
      case 'equal':
        return value === limit;
      default:
        return false;
    }
  }

  // Trigger alert
  triggerAlert(alertType, data = {}) {
    const alertId = `${alertType}_${Date.now()}`;

    const alert = {
      id: alertId,
      type: alertType,
      data,
      timestamp: new Date(),
      status: 'active',
    };

    this.alerts.set(alertId, alert);

    // Send notification
    this.sendAlertNotification(alert);

    // Log to analytics
    AnalyticsService.logEvent('alert_triggered', {
      alert_type: alertType,
      alert_id: alertId,
      ...data,
    });

    return alertId;
  }

  // Send alert notification
  sendAlertNotification(alert) {
    const { type, data } = alert;

    let title, message;

    switch (type) {
      case 'high_memory':
        title = 'High Memory Usage';
        message = `Memory usage is ${data.percentage}%`;
        break;
      case 'low_battery':
        title = 'Low Battery';
        message = `Battery level is ${data.level}%`;
        break;
      case 'slow_performance':
        title = 'Performance Issue';
        message = `App is running slowly (${data.fps} FPS)`;
        break;
      case 'high_error_rate':
        title = 'High Error Rate';
        message = `Error rate is ${data.rate}%`;
        break;
      default:
        title = 'Alert';
        message = 'An alert has been triggered';
    }

    PushNotification.localNotification({
      id: alert.id,
      title,
      message,
      playSound: true,
      soundName: 'default',
      vibrate: true,
      vibration: 1000,
    });
  }

  // Resolve alert
  resolveAlert(alertId) {
    const alert = this.alerts.get(alertId);
    if (alert) {
      alert.status = 'resolved';
      alert.resolvedAt = new Date();

      AnalyticsService.logEvent('alert_resolved', {
        alert_id: alertId,
        alert_type: alert.type,
        duration: alert.resolvedAt - alert.timestamp,
      });
    }
  }

  // Get active alerts
  getActiveAlerts() {
    return Array.from(this.alerts.values()).filter(alert => alert.status === 'active');
  }

  // Performance monitoring alerts
  monitorPerformance(metrics) {
    // Memory alert
    if (this.checkThreshold('memory_usage', metrics.memory.percentage)) {
      this.triggerAlert('high_memory', {
        percentage: metrics.memory.percentage,
      });
    }

    // Battery alert
    if (this.checkThreshold('battery_level', metrics.battery.level)) {
      this.triggerAlert('low_battery', {
        level: metrics.battery.level,
      });
    }

    // FPS alert
    if (this.checkThreshold('fps', metrics.fps)) {
      this.triggerAlert('slow_performance', {
        fps: metrics.fps,
      });
    }
  }

  // Error rate monitoring
  monitorErrorRate(errorCount, totalRequests) {
    const errorRate = (errorCount / totalRequests) * 100;

    if (this.checkThreshold('error_rate', errorRate)) {
      this.triggerAlert('high_error_rate', {
        rate: errorRate.toFixed(2),
        errors: errorCount,
        total: totalRequests,
      });
    }
  }

  // Auto-resolve alerts
  setupAutoResolve() {
    // Check for alerts that can be auto-resolved
    setInterval(() => {
      const activeAlerts = this.getActiveAlerts();

      activeAlerts.forEach(alert => {
        const age = Date.now() - alert.timestamp;

        // Auto-resolve old alerts
        if (age > 24 * 60 * 60 * 1000) { // 24 hours
          this.resolveAlert(alert.id);
        }
      });
    }, 60 * 1000); // Check every minute
  }

  // Get alert statistics
  getAlertStatistics() {
    const alerts = Array.from(this.alerts.values());
    const active = alerts.filter(a => a.status === 'active');
    const resolved = alerts.filter(a => a.status === 'resolved');

    return {
      total: alerts.length,
      active: active.length,
      resolved: resolved.length,
      byType: alerts.reduce((acc, alert) => {
        acc[alert.type] = (acc[alert.type] || 0) + 1;
        return acc;
      }, {}),
    };
  }
}

export default new AlertSystem();

---

## 🎯 **Practical Examples**

### **Complete Performance Monitoring Setup**
```javascript
// App.js
import React, { useEffect } from 'react';
import { AppState, Platform } from 'react-native';
import { NavigationContainer } from '@react-navigation/native';

// Services
import performanceMonitor from './services/performanceMonitor';
import { initializeSentry } from './services/sentry';
import AnalyticsService from './services/analytics';
import realtimeMonitor from './services/realtimeMonitor';
import alertSystem from './services/alertSystem';

// Components
import AppNavigator from './navigation/AppNavigator';
import ErrorBoundary from './components/ErrorBoundary';

const App = () => {
  useEffect(() => {
    initializeApp();
    setupAppStateListener();

    return () => {
      cleanupApp();
    };
  }, []);

  const initializeApp = async () => {
    try {
      // Initialize monitoring services
      await initializeSentry();
      await AnalyticsService.initialize();

      // Start performance monitoring
      performanceMonitor.trackMemoryUsage();

      // Start real-time monitoring
      realtimeMonitor.startMonitoring(10000); // Every 10 seconds

      // Setup alert thresholds
      alertSystem.setThreshold('memory_usage', 80, 'greater');
      alertSystem.setThreshold('battery_level', 20, 'less');
      alertSystem.setThreshold('fps', 30, 'less');
      alertSystem.setThreshold('error_rate', 5, 'greater');

      // Setup auto-resolve for alerts
      alertSystem.setupAutoResolve();

      // Track app open
      AnalyticsService.logAppOpen();

      console.log('App monitoring initialized');
    } catch (error) {
      console.error('Failed to initialize monitoring:', error);
    }
  };

  const setupAppStateListener = () => {
    const subscription = AppState.addEventListener('change', (nextAppState) => {
      if (nextAppState === 'active') {
        // App came to foreground
        AnalyticsService.logEvent('app_foreground');
        realtimeMonitor.startMonitoring();
      } else if (nextAppState === 'background') {
        // App went to background
        AnalyticsService.logEvent('app_background');
        realtimeMonitor.stopMonitoring();
      }
    });

    return subscription;
  };

  const cleanupApp = () => {
    realtimeMonitor.stopMonitoring();
  };

  return (
    <ErrorBoundary>
      <NavigationContainer
        onStateChange={(state) => {
          // Track navigation changes
          const currentRoute = state.routes[state.index];
          AnalyticsService.logScreenView(currentRoute.name);
        }}
      >
        <AppNavigator />
      </NavigationContainer>
    </ErrorBoundary>
  );
};

export default App;
```

### **Complete Analytics Dashboard**
```javascript
// screens/AnalyticsDashboardScreen.js
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  ScrollView,
  StyleSheet,
  TouchableOpacity,
  Alert,
  RefreshControl,
} from 'react-native';
import { useFocusEffect } from '@react-navigation/native';

// Services
import AnalyticsService from '../services/analytics';
import PerformanceMetrics from '../services/performanceMetrics';
import alertSystem from '../services/alertSystem';

// Components
import MetricCard from '../components/MetricCard';
import AlertList from '../components/AlertList';
import PerformanceChart from '../components/PerformanceChart';

const AnalyticsDashboardScreen = () => {
  const [dashboardData, setDashboardData] = useState({
    performance: {},
    analytics: {},
    alerts: [],
  });
  const [isLoading, setIsLoading] = useState(true);
  const [lastUpdated, setLastUpdated] = useState(null);

  useFocusEffect(
    React.useCallback(() => {
      loadDashboardData();
    }, [])
  );

  const loadDashboardData = async () => {
    setIsLoading(true);

    try {
      // Load performance metrics
      const performanceReport = PerformanceMetrics.getMetricsReport();

      // Load analytics data
      const analyticsData = await loadAnalyticsData();

      // Load active alerts
      const alerts = alertSystem.getActiveAlerts();

      setDashboardData({
        performance: performanceReport,
        analytics: analyticsData,
        alerts,
      });

      setLastUpdated(new Date());
    } catch (error) {
      console.error('Error loading dashboard data:', error);
      Alert.alert('Error', 'Failed to load dashboard data');
    } finally {
      setIsLoading(false);
    }
  };

  const loadAnalyticsData = async () => {
    // In a real app, this would fetch from your analytics service
    return {
      totalUsers: 15420,
      activeUsers: 8920,
      averageSessionDuration: 285,
      topScreens: [
        { name: 'Home', views: 45230, percentage: 35 },
        { name: 'Profile', views: 28910, percentage: 22 },
        { name: 'Search', views: 18750, percentage: 14 },
        { name: 'Settings', views: 12450, percentage: 9 },
        { name: 'Notifications', views: 9870, percentage: 8 },
      ],
      conversionRate: 4.2,
      crashRate: 0.08,
      retentionRate: 68,
      revenue: 15420,
    };
  };

  const handleRefresh = () => {
    loadDashboardData();
  };

  const handleClearAlerts = () => {
    Alert.alert(
      'Clear All Alerts',
      'Are you sure you want to clear all active alerts?',
      [
        { text: 'Cancel', style: 'cancel' },
        {
          text: 'Clear',
          onPress: () => {
            dashboardData.alerts.forEach(alert => {
              alertSystem.resolveAlert(alert.id);
            });
            loadDashboardData();
          },
        },
      ]
    );
  };

  const formatDuration = (seconds) => {
    const minutes = Math.floor(seconds / 60);
    const remainingSeconds = seconds % 60;
    return `${minutes}m ${remainingSeconds}s`;
  };

  const formatCurrency = (amount) => {
    return `$${amount.toLocaleString()}`;
  };

  return (
    <ScrollView
      style={styles.container}
      refreshControl={
        <RefreshControl refreshing={isLoading} onRefresh={handleRefresh} />
      }
    >
      <View style={styles.header}>
        <Text style={styles.title}>Analytics Dashboard</Text>
        {lastUpdated && (
          <Text style={styles.lastUpdated}>
            Last updated: {lastUpdated.toLocaleTimeString()}
          </Text>
        )}
      </View>

      {/* Key Metrics */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Key Metrics</Text>
        <View style={styles.metricsGrid}>
          <MetricCard
            title="Total Users"
            value={dashboardData.analytics.totalUsers?.toLocaleString() || '0'}
            subtitle="Registered users"
            icon="👥"
          />
          <MetricCard
            title="Active Users"
            value={dashboardData.analytics.activeUsers?.toLocaleString() || '0'}
            subtitle="Last 30 days"
            icon="🔥"
          />
          <MetricCard
            title="Avg Session"
            value={formatDuration(dashboardData.analytics.averageSessionDuration || 0)}
            subtitle="Session duration"
            icon="⏱️"
          />
          <MetricCard
            title="Revenue"
            value={formatCurrency(dashboardData.analytics.revenue || 0)}
            subtitle="Total revenue"
            icon="💰"
          />
        </View>
      </View>

      {/* Performance Metrics */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Performance</Text>
        <View style={styles.metricsGrid}>
          <MetricCard
            title="Crash Rate"
            value={`${dashboardData.analytics.crashRate || 0}%`}
            subtitle="App crashes"
            icon="💥"
            trend={dashboardData.analytics.crashRate < 0.1 ? 'down' : 'up'}
          />
          <MetricCard
            title="Conversion"
            value={`${dashboardData.analytics.conversionRate || 0}%`}
            subtitle="Conversion rate"
            icon="📈"
            trend="up"
          />
          <MetricCard
            title="Retention"
            value={`${dashboardData.analytics.retentionRate || 0}%`}
            subtitle="User retention"
            icon="🔄"
            trend="up"
          />
          <MetricCard
            title="Slow Renders"
            value={dashboardData.performance.slowRenders || 0}
            subtitle="Performance issues"
            icon="🐌"
            trend={dashboardData.performance.slowRenders < 10 ? 'down' : 'up'}
          />
        </View>
      </View>

      {/* Top Screens */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Top Screens</Text>
        {dashboardData.analytics.topScreens?.map((screen, index) => (
          <View key={screen.name} style={styles.screenItem}>
            <View style={styles.screenRank}>
              <Text style={styles.rankText}>#{index + 1}</Text>
            </View>
            <View style={styles.screenInfo}>
              <Text style={styles.screenName}>{screen.name}</Text>
              <Text style={styles.screenStats}>
                {screen.views.toLocaleString()} views ({screen.percentage}%)
              </Text>
            </View>
            <View style={styles.screenBar}>
              <View
                style={[
                  styles.screenBarFill,
                  { width: `${screen.percentage}%` },
                ]}
              />
            </View>
          </View>
        ))}
      </View>

      {/* Active Alerts */}
      {dashboardData.alerts.length > 0 && (
        <View style={styles.section}>
          <View style={styles.sectionHeader}>
            <Text style={styles.sectionTitle}>Active Alerts</Text>
            <TouchableOpacity onPress={handleClearAlerts}>
              <Text style={styles.clearButton}>Clear All</Text>
            </TouchableOpacity>
          </View>
          <AlertList alerts={dashboardData.alerts} />
        </View>
      )}

      {/* Performance Chart */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Performance Trends</Text>
        <PerformanceChart data={dashboardData.performance} />
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  header: {
    padding: 20,
    backgroundColor: '#fff',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
  },
  lastUpdated: {
    fontSize: 12,
    color: '#666',
    marginTop: 5,
  },
  section: {
    backgroundColor: '#fff',
    margin: 10,
    borderRadius: 12,
    padding: 15,
  },
  sectionHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 15,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
  },
  clearButton: {
    color: '#007AFF',
    fontSize: 14,
    fontWeight: '600',
  },
  metricsGrid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-between',
  },
  screenItem: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingVertical: 12,
    borderBottomWidth: 1,
    borderBottomColor: '#f0f0f0',
  },
  screenRank: {
    width: 40,
    height: 40,
    borderRadius: 20,
    backgroundColor: '#007AFF',
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: 15,
  },
  rankText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
  screenInfo: {
    flex: 1,
  },
  screenName: {
    fontSize: 16,
    fontWeight: '600',
    color: '#333',
  },
  screenStats: {
    fontSize: 14,
    color: '#666',
    marginTop: 2,
  },
  screenBar: {
    flex: 1,
    height: 8,
    backgroundColor: '#f0f0f0',
    borderRadius: 4,
    marginLeft: 15,
    overflow: 'hidden',
  },
  screenBarFill: {
    height: '100%',
    backgroundColor: '#007AFF',
    borderRadius: 4,
  },
});

export default AnalyticsDashboardScreen;
```

### **A/B Testing Implementation**
```javascript
// components/ABTestButton.js
import React from 'react';
import { TouchableOpacity, Text, StyleSheet } from 'react-native';
import abTesting from '../services/abTesting';

const ABTestButton = ({ userId, onPress, children, ...props }) => {
  // Get button color variant
  const buttonColor = abTesting.getVariant('button_color', userId);

  // Track button click
  const handlePress = () => {
    abTesting.trackEvent('button_color', 'click', {
      button_color: buttonColor,
      user_id: userId,
    });

    if (onPress) {
      onPress();
    }
  };

  const getButtonStyle = () => {
    switch (buttonColor) {
      case 'blue':
        return styles.blueButton;
      case 'green':
        return styles.greenButton;
      case 'red':
        return styles.redButton;
      default:
        return styles.defaultButton;
    }
  };

  return (
    <TouchableOpacity
      style={[styles.button, getButtonStyle()]}
      onPress={handlePress}
      {...props}
    >
      <Text style={styles.buttonText}>{children}</Text>
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  button: {
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    justifyContent: 'center',
  },
  defaultButton: {
    backgroundColor: '#007AFF',
  },
  blueButton: {
    backgroundColor: '#007AFF',
  },
  greenButton: {
    backgroundColor: '#28a745',
  },
  redButton: {
    backgroundColor: '#dc3545',
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default ABTestButton;
```

### **Real-time Monitoring Component**
```javascript
// components/RealtimeMonitor.js
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet, ActivityIndicator } from 'react-native';
import realtimeMonitor from '../services/realtimeMonitor';
import alertSystem from '../services/alertSystem';

const RealtimeMonitor = () => {
  const [metrics, setMetrics] = useState({});
  const [isMonitoring, setIsMonitoring] = useState(false);

  useEffect(() => {
    const listener = realtimeMonitor.addListener((newMetrics) => {
      setMetrics(newMetrics);
      alertSystem.monitorPerformance(newMetrics);
    });

    setIsMonitoring(realtimeMonitor.isMonitoring);

    return () => {
      listener?.();
    };
  }, []);

  const getStatusColor = (value, thresholds) => {
    if (value >= thresholds.high) return '#dc3545';
    if (value >= thresholds.medium) return '#ffc107';
    return '#28a745';
  };

  const formatBytes = (bytes) => {
    if (bytes === 0) return '0 B';
    const k = 1024;
    const sizes = ['B', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  };

  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Real-time Monitor</Text>
        <View style={[styles.statusIndicator, { backgroundColor: isMonitoring ? '#28a745' : '#dc3545' }]}>
          <Text style={styles.statusText}>
            {isMonitoring ? 'Active' : 'Inactive'}
          </Text>
        </View>
      </View>

      <View style={styles.metricsGrid}>
        {/* Memory Usage */}
        <View style={styles.metricCard}>
          <Text style={styles.metricLabel}>Memory</Text>
          <Text
            style={[
              styles.metricValue,
              {
                color: getStatusColor(metrics.memory?.percentage || 0, {
                  medium: 60,
                  high: 80,
                }),
              },
            ]}
          >
            {metrics.memory?.percentage || 0}%
          </Text>
          <Text style={styles.metricSubtext}>
            {formatBytes(metrics.memory?.used || 0)} / {formatBytes(metrics.memory?.total || 0)}
          </Text>
        </View>

        {/* Battery Level */}
        <View style={styles.metricCard}>
          <Text style={styles.metricLabel}>Battery</Text>
          <Text
            style={[
              styles.metricValue,
              {
                color: getStatusColor(100 - (metrics.battery?.level || 100), {
                  medium: 30,
                  high: 50,
                }),
              },
            ]}
          >
            {metrics.battery?.level || 0}%
          </Text>
          <Text style={styles.metricSubtext}>
            {metrics.battery?.isCharging ? 'Charging' : 'Discharging'}
          </Text>
        </View>

        {/* Network */}
        <View style={styles.metricCard}>
          <Text style={styles.metricLabel}>Network</Text>
          <Text style={styles.metricValue}>
            {metrics.network?.type || 'Unknown'}
          </Text>
          <Text style={styles.metricSubtext}>
            {metrics.network?.isConnected ? 'Connected' : 'Disconnected'}
          </Text>
        </View>

        {/* FPS */}
        <View style={styles.metricCard}>
          <Text style={styles.metricLabel}>FPS</Text>
          <Text
            style={[
              styles.metricValue,
              {
                color: getStatusColor(60 - (metrics.fps || 60), {
                  medium: 20,
                  high: 30,
                }),
              },
            ]}
          >
            {metrics.fps || 60}
          </Text>
          <Text style={styles.metricSubtext}>Frames per second</Text>
        </View>
      </View>

      {metrics.timestamp && (
        <Text style={styles.lastUpdate}>
          Last update: {new Date(metrics.timestamp).toLocaleTimeString()}
        </Text>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 15,
    margin: 10,
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 15,
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
  },
  statusIndicator: {
    paddingHorizontal: 8,
    paddingVertical: 4,
    borderRadius: 12,
  },
  statusText: {
    color: '#fff',
    fontSize: 12,
    fontWeight: 'bold',
  },
  metricsGrid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-between',
  },
  metricCard: {
    width: '48%',
    backgroundColor: '#f8f9fa',
    padding: 12,
    borderRadius: 8,
    marginBottom: 10,
    alignItems: 'center',
  },
  metricLabel: {
    fontSize: 12,
    color: '#666',
    marginBottom: 4,
  },
  metricValue: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 2,
  },
  metricSubtext: {
    fontSize: 10,
    color: '#999',
    textAlign: 'center',
  },
  lastUpdate: {
    fontSize: 10,
    color: '#999',
    textAlign: 'center',
    marginTop: 10,
  },
});

export default RealtimeMonitor;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Performance Monitoring Setup**: React Native performance monitor with custom metrics
- ✅ **Crash Reporting & Error Tracking**: Sentry and Firebase Crashlytics integration
- ✅ **Analytics Implementation**: Firebase Analytics and Mixpanel integration
- ✅ **User Behavior Tracking**: Navigation tracking and interaction monitoring
- ✅ **Performance Metrics**: Custom metrics and real-time monitoring
- ✅ **A/B Testing**: Framework for testing different app variants
- ✅ **Real-time Monitoring**: Live performance and system metrics
- ✅ **Custom Dashboards**: Analytics dashboard with real-time data
- ✅ **Alerting & Notifications**: Alert system with push notifications
- ✅ **Practical Implementation**: Complete monitoring and analytics setup

### **Best Practices**
1. **Implement comprehensive monitoring** - Track performance, errors, and user behavior
2. **Use multiple analytics services** - Combine Firebase, Mixpanel, and custom tracking
3. **Set up proper error boundaries** - Catch and report React errors
4. **Monitor real-time metrics** - Track memory, battery, network, and FPS
5. **Implement A/B testing** - Test different features and UI variants
6. **Create custom dashboards** - Visualize analytics data effectively
7. **Set up alerting systems** - Get notified of critical issues
8. **Track user journeys** - Understand how users navigate your app
9. **Monitor API performance** - Track response times and error rates
10. **Respect user privacy** - Implement proper data collection consent

### **Next Steps**
- Learn about advanced analytics platforms (Amplitude, Segment)
- Implement server-side analytics and monitoring
- Set up CI/CD with automated testing and monitoring
- Learn about performance profiling tools
- Study data visualization and dashboard creation
- Implement predictive analytics
- Learn about privacy regulations (GDPR, CCPA)
- Study conversion rate optimization
- Implement user feedback systems
- Learn about heatmaps and session recordings

---

## 🎯 **Assignment**

### **Task 1: Complete Analytics Implementation**
Implement a comprehensive analytics system with:
- Firebase Analytics for basic tracking
- Mixpanel for advanced user behavior analysis
- Custom event tracking for business metrics
- User segmentation and cohort analysis
- Funnel analysis for user conversion
- Revenue tracking and monetization metrics
- Real-time dashboard with key performance indicators

### **Task 2: Performance Monitoring System**
Create a complete performance monitoring solution:
- Real-time performance metrics collection
- Memory usage and leak detection
- Network request monitoring and optimization
- Battery and device performance tracking
- FPS monitoring and frame drop detection
- Custom performance alerts and notifications
- Performance trend analysis and reporting

### **Task 3: Crash Reporting & Error Handling**
Implement robust error tracking and reporting:
- Sentry integration with custom error boundaries
- Firebase Crashlytics for native crash reporting
- Custom error logging and categorization
- Error trend analysis and alerting
- User impact assessment for errors
- Error reproduction and debugging tools
- Automated error reporting and ticketing

### **Task 4: A/B Testing Framework**
Build a complete A/B testing system:
- Experiment definition and variant management
- User segmentation and targeting
- Statistical significance calculation
- Real-time results monitoring
- Automated winner determination
- Multi-armed bandit algorithms
- Integration with feature flags
- Results visualization and reporting

---

## 📚 **Additional Resources**
- [Firebase Analytics Documentation](https://firebase.google.com/docs/analytics)
- [Sentry Documentation](https://docs.sentry.io/)
- [Mixpanel Documentation](https://docs.mixpanel.com/)
- [React Native Performance](https://reactnative.dev/docs/performance)
- [A/B Testing Guide](https://www.optimizely.com/optimization-glossary/ab-testing/)

---

**Next Lesson**: [Lesson 40: Testing & Quality Assurance](Lesson%2040_%20Testing%20&%20Quality%20Assurance.md)