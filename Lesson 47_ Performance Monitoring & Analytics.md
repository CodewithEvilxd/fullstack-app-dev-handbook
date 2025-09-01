# Lesson 47: Performance Monitoring & Analytics

## 🎯 **Learning Objectives**
- Implement comprehensive performance monitoring for React Native applications
- Set up analytics tracking and user behavior analysis
- Create performance dashboards and alerting systems
- Optimize app performance based on real user data
- Implement A/B testing and feature experimentation

## 📚 **Table of Contents**
1. [Performance Monitoring Fundamentals](#performance-monitoring-fundamentals)
2. [Analytics Implementation](#analytics-implementation)
3. [Real User Monitoring](#real-user-monitoring)
4. [Performance Metrics](#performance-metrics)
5. [Crash Reporting](#crash-reporting)
6. [A/B Testing](#ab-testing)
7. [Performance Dashboards](#performance-dashboards)
8. [Alerting & Notifications](#alerting--notifications)
9. [Data Analysis & Insights](#data-analysis--insights)
10. [Practical Examples](#practical-examples)

---

## 📊 **Performance Monitoring Fundamentals**

### **Key Performance Indicators (KPIs)**
- **App Start Time**: Time to interactive
- **Frame Rate**: 60 FPS target
- **Memory Usage**: Heap size and leaks
- **Network Requests**: Response times and success rates
- **Crash Rate**: Application stability
- **User Engagement**: Session duration and retention

### **Monitoring Tools**
- **Firebase Performance Monitoring**: Real-time performance data
- **Sentry**: Error tracking and performance monitoring
- **DataDog**: Comprehensive monitoring and analytics
- **New Relic**: Application performance monitoring
- **App Center**: Microsoft crash reporting and analytics

---

## 📈 **Analytics Implementation**

### **Firebase Analytics Setup**
```javascript
// services/analyticsService.js
import analytics from '@react-native-firebase/analytics';
import { Platform } from 'react-native';

class AnalyticsService {
  constructor() {
    this.isEnabled = true;
    this.userProperties = {};
  }

  // Initialize analytics
  async initialize() {
    try {
      await analytics().setAnalyticsCollectionEnabled(this.isEnabled);

      // Set default user properties
      await this.setUserProperty('platform', Platform.OS);
      await this.setUserProperty('app_version', '1.0.0');

      console.log('Analytics initialized');
    } catch (error) {
      console.error('Analytics initialization error:', error);
    }
  }

  // Track screen views
  async trackScreen(screenName, screenClass = null) {
    try {
      await analytics().logScreenView({
        screen_name: screenName,
        screen_class: screenClass || screenName,
      });

      console.log(`Screen tracked: ${screenName}`);
    } catch (error) {
      console.error('Screen tracking error:', error);
    }
  }

  // Track custom events
  async trackEvent(eventName, parameters = {}) {
    try {
      // Validate event name
      if (!this.isValidEventName(eventName)) {
        console.warn(`Invalid event name: ${eventName}`);
        return;
      }

      // Sanitize parameters
      const sanitizedParams = this.sanitizeParameters(parameters);

      await analytics().logEvent(eventName, sanitizedParams);

      console.log(`Event tracked: ${eventName}`, sanitizedParams);
    } catch (error) {
      console.error('Event tracking error:', error);
    }
  }

  // Track user actions
  async trackUserAction(action, context = {}) {
    const eventName = `user_action_${action}`;
    const parameters = {
      action,
      timestamp: new Date().toISOString(),
      ...context,
    };

    await this.trackEvent(eventName, parameters);
  }

  // Track business events
  async trackBusinessEvent(eventType, data = {}) {
    const eventName = `business_${eventType}`;
    const parameters = {
      event_type: eventType,
      value: data.value || 0,
      currency: data.currency || 'USD',
      ...data,
    };

    await this.trackEvent(eventName, parameters);
  }

  // Track errors
  async trackError(error, context = {}) {
    const parameters = {
      error_message: error.message,
      error_stack: error.stack,
      error_type: error.name,
      timestamp: new Date().toISOString(),
      ...context,
    };

    await this.trackEvent('app_error', parameters);
  }

  // Track performance metrics
  async trackPerformance(metricName, value, unit = 'ms') {
    const parameters = {
      metric_name: metricName,
      metric_value: value,
      metric_unit: unit,
      timestamp: new Date().toISOString(),
    };

    await this.trackEvent('performance_metric', parameters);
  }

  // Set user properties
  async setUserProperty(name, value) {
    try {
      if (!this.isValidUserProperty(name, value)) {
        console.warn(`Invalid user property: ${name}`);
        return;
      }

      await analytics().setUserProperty(name, String(value));
      this.userProperties[name] = value;

      console.log(`User property set: ${name} = ${value}`);
    } catch (error) {
      console.error('User property error:', error);
    }
  }

  // Set user ID
  async setUserId(userId) {
    try {
      await analytics().setUserId(userId);
      await this.setUserProperty('user_id', userId);
      console.log(`User ID set: ${userId}`);
    } catch (error) {
      console.error('User ID error:', error);
    }
  }

  // Track session
  async trackSessionStart() {
    await this.trackEvent('session_start', {
      timestamp: new Date().toISOString(),
    });
  }

  async trackSessionEnd(duration) {
    await this.trackEvent('session_end', {
      duration,
      timestamp: new Date().toISOString(),
    });
  }

  // Track purchases
  async trackPurchase(productId, price, currency = 'USD', quantity = 1) {
    try {
      await analytics().logPurchase({
        currency,
        value: price * quantity,
        items: [{
          item_id: productId,
          item_name: productId,
          quantity,
          price,
        }],
      });

      await this.trackBusinessEvent('purchase', {
        product_id: productId,
        price,
        currency,
        quantity,
        total_value: price * quantity,
      });

      console.log(`Purchase tracked: ${productId}`);
    } catch (error) {
      console.error('Purchase tracking error:', error);
    }
  }

  // Validation helpers
  isValidEventName(name) {
    // Firebase event name rules
    return /^[a-zA-Z][a-zA-Z0-9_]*$/.test(name) && name.length <= 40;
  }

  isValidUserProperty(name, value) {
    // Firebase user property rules
    return name.length <= 24 && String(value).length <= 36;
  }

  sanitizeParameters(params) {
    const sanitized = {};

    for (const [key, value] of Object.entries(params)) {
      if (key.length <= 40) {
        if (typeof value === 'string' && value.length > 100) {
          sanitized[key] = value.substring(0, 100);
        } else {
          sanitized[key] = value;
        }
      }
    }

    return sanitized;
  }

  // Enable/disable analytics
  async setEnabled(enabled) {
    try {
      this.isEnabled = enabled;
      await analytics().setAnalyticsCollectionEnabled(enabled);
      console.log(`Analytics ${enabled ? 'enabled' : 'disabled'}`);
    } catch (error) {
      console.error('Analytics toggle error:', error);
    }
  }

  // Reset analytics data
  async reset() {
    try {
      await analytics().resetAnalyticsData();
      this.userProperties = {};
      console.log('Analytics data reset');
    } catch (error) {
      console.error('Analytics reset error:', error);
    }
  }

  // Get analytics instance ID
  async getInstanceId() {
    try {
      return await analytics().getAppInstanceId();
    } catch (error) {
      console.error('Instance ID error:', error);
      return null;
    }
  }
}

export default new AnalyticsService();
```

### **Custom Analytics Events**
```javascript
// constants/analyticsEvents.js
export const ANALYTICS_EVENTS = {
  // User actions
  USER_LOGIN: 'user_login',
  USER_LOGOUT: 'user_logout',
  USER_REGISTER: 'user_register',
  USER_PROFILE_UPDATE: 'user_profile_update',

  // App interactions
  APP_OPEN: 'app_open',
  APP_CLOSE: 'app_close',
  SCREEN_VIEW: 'screen_view',
  BUTTON_CLICK: 'button_click',
  FORM_SUBMIT: 'form_submit',

  // Business events
  PURCHASE_START: 'purchase_start',
  PURCHASE_COMPLETE: 'purchase_complete',
  SUBSCRIPTION_START: 'subscription_start',
  SUBSCRIPTION_CANCEL: 'subscription_cancel',

  // Content interactions
  CONTENT_VIEW: 'content_view',
  CONTENT_SHARE: 'content_share',
  CONTENT_LIKE: 'content_like',
  SEARCH_QUERY: 'search_query',

  // Error events
  ERROR_OCCURRED: 'error_occurred',
  NETWORK_ERROR: 'network_error',
  API_ERROR: 'api_error',

  // Performance events
  APP_START_TIME: 'app_start_time',
  SCREEN_LOAD_TIME: 'screen_load_time',
  API_RESPONSE_TIME: 'api_response_time',
};

export const ANALYTICS_PROPERTIES = {
  // User properties
  USER_TYPE: 'user_type',
  SUBSCRIPTION_STATUS: 'subscription_status',
  USER_COHORT: 'user_cohort',

  // Content properties
  CONTENT_TYPE: 'content_type',
  CONTENT_CATEGORY: 'content_category',
  SEARCH_TERM: 'search_term',

  // Performance properties
  LOAD_TIME: 'load_time',
  RESPONSE_TIME: 'response_time',
  ERROR_CODE: 'error_code',
};
```

---

## 👥 **Real User Monitoring**

### **Firebase Performance Monitoring**
```javascript
// services/performanceMonitoring.js
import perf from '@react-native-firebase/perf';
import analytics from '../services/analyticsService';

class PerformanceMonitoring {
  constructor() {
    this.traces = new Map();
    this.isEnabled = true;
  }

  // Start performance trace
  async startTrace(traceName) {
    if (!this.isEnabled) return null;

    try {
      const trace = await perf().startTrace(traceName);
      this.traces.set(traceName, trace);

      console.log(`Performance trace started: ${traceName}`);
      return trace;
    } catch (error) {
      console.error('Start trace error:', error);
      return null;
    }
  }

  // Stop performance trace
  async stopTrace(traceName) {
    const trace = this.traces.get(traceName);
    if (!trace) return;

    try {
      await trace.stop();
      this.traces.delete(traceName);

      console.log(`Performance trace stopped: ${traceName}`);
    } catch (error) {
      console.error('Stop trace error:', error);
    }
  }

  // Measure function execution time
  async measureExecutionTime(traceName, fn) {
    const trace = await this.startTrace(traceName);

    try {
      const startTime = Date.now();
      const result = await fn();
      const executionTime = Date.now() - startTime;

      // Track performance metric
      await analytics.trackPerformance(traceName, executionTime);

      return result;
    } finally {
      if (trace) {
        await this.stopTrace(traceName);
      }
    }
  }

  // Monitor network requests
  async monitorNetworkRequest(url, method = 'GET') {
    const traceName = `network_${method}_${url.replace(/[^a-zA-Z0-9]/g, '_')}`;

    return this.measureExecutionTime(traceName, async () => {
      const startTime = Date.now();

      // This would be integrated with your HTTP client
      const response = await fetch(url, { method });

      const responseTime = Date.now() - startTime;

      // Track network performance
      await analytics.trackEvent('network_request', {
        url,
        method,
        response_time: responseTime,
        status_code: response.status,
        success: response.ok,
      });

      return response;
    });
  }

  // Monitor screen loading
  async monitorScreenLoad(screenName) {
    const traceName = `screen_load_${screenName}`;

    return this.measureExecutionTime(traceName, async () => {
      const startTime = Date.now();

      // Screen loading logic would go here
      // This is a placeholder for the actual screen loading

      const loadTime = Date.now() - startTime;

      await analytics.trackEvent('screen_load', {
        screen_name: screenName,
        load_time: loadTime,
      });

      return loadTime;
    });
  }

  // Monitor app startup
  async monitorAppStartup() {
    const traceName = 'app_startup';

    return this.measureExecutionTime(traceName, async () => {
      const startTime = Date.now();

      // App initialization logic
      await this.initializeServices();

      const startupTime = Date.now() - startTime;

      await analytics.trackEvent('app_startup', {
        startup_time: startupTime,
        timestamp: new Date().toISOString(),
      });

      return startupTime;
    });
  }

  // Monitor memory usage
  async monitorMemoryUsage() {
    // Note: Memory monitoring is limited in React Native
    // This is a basic implementation
    if (global.gc) {
      const startMemory = this.getMemoryUsage();
      global.gc();
      const endMemory = this.getMemoryUsage();

      const freedMemory = startMemory - endMemory;

      await analytics.trackEvent('memory_cleanup', {
        memory_freed: freedMemory,
        memory_before: startMemory,
        memory_after: endMemory,
      });
    }
  }

  // Get memory usage (limited in RN)
  getMemoryUsage() {
    // This is a placeholder - actual memory monitoring
    // would require native modules or third-party libraries
    return 0;
  }

  // Monitor custom metrics
  async monitorCustomMetric(metricName, value, unit = 'count') {
    const traceName = `custom_metric_${metricName}`;

    const trace = await this.startTrace(traceName);
    if (trace) {
      trace.putMetric(metricName, value);
      await this.stopTrace(traceName);
    }

    await analytics.trackPerformance(metricName, value, unit);
  }

  // Enable/disable monitoring
  setEnabled(enabled) {
    this.isEnabled = enabled;
    console.log(`Performance monitoring ${enabled ? 'enabled' : 'disabled'}`);
  }

  // Initialize services (placeholder)
  async initializeServices() {
    // Initialize various services
    await new Promise(resolve => setTimeout(resolve, 100));
  }

  // Get performance metrics
  async getPerformanceMetrics() {
    try {
      const metrics = await perf().getTraceDuration('app_startup');
      return {
        app_startup_time: metrics,
        // Add more metrics as needed
      };
    } catch (error) {
      console.error('Get metrics error:', error);
      return {};
    }
  }
}

export default new PerformanceMonitoring();
```

---

## 📏 **Performance Metrics**

### **Core Web Vitals for Mobile**
```javascript
// services/coreWebVitals.js
import analytics from '../services/analyticsService';

class CoreWebVitals {
  constructor() {
    this.metrics = {};
    this.observers = [];
  }

  // Measure First Contentful Paint (FCP)
  measureFCP() {
    // In React Native, FCP is when the first component renders
    const startTime = Date.now();

    // This would be called when the first component mounts
    const measureFCP = () => {
      const fcp = Date.now() - startTime;

      this.metrics.fcp = fcp;
      this.notifyObservers('fcp', fcp);

      analytics.trackPerformance('first_contentful_paint', fcp);

      console.log(`FCP: ${fcp}ms`);
    };

    return measureFCP;
  }

  // Measure Largest Contentful Paint (LCP)
  measureLCP() {
    let largestPaint = 0;
    let largestElement = null;

    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (entry.size > largestPaint) {
          largestPaint = entry.size;
          largestElement = entry.element;
        }
      }
    });

    observer.observe({ entryTypes: ['largest-contentful-paint'] });

    // In React Native context, this would be adapted
    // to measure when the largest content element renders
    const measureLCP = (elementSize) => {
      if (elementSize > largestPaint) {
        largestPaint = elementSize;
        this.metrics.lcp = elementSize;
        this.notifyObservers('lcp', elementSize);

        analytics.trackPerformance('largest_contentful_paint', elementSize);
      }
    };

    return measureLCP;
  }

  // Measure First Input Delay (FID)
  measureFID() {
    let firstInput = null;

    const measureFID = (inputDelay) => {
      if (!firstInput) {
        firstInput = inputDelay;
        this.metrics.fid = inputDelay;
        this.notifyObservers('fid', inputDelay);

        analytics.trackPerformance('first_input_delay', inputDelay);

        console.log(`FID: ${inputDelay}ms`);
      }
    };

    return measureFID;
  }

  // Measure Cumulative Layout Shift (CLS)
  measureCLS() {
    let clsValue = 0;
    let sessionEntries = [];

    const measureCLS = (layoutShift) => {
      sessionEntries.push(layoutShift);

      // Calculate CLS for the current session
      clsValue = sessionEntries.reduce((sum, entry) => sum + entry, 0);

      this.metrics.cls = clsValue;
      this.notifyObservers('cls', clsValue);

      analytics.trackPerformance('cumulative_layout_shift', clsValue);
    };

    return measureCLS;
  }

  // Get all metrics
  getMetrics() {
    return { ...this.metrics };
  }

  // Check if metrics are good
  getMetricsAssessment() {
    const assessment = {
      fcp: this.assessFCP(this.metrics.fcp),
      lcp: this.assessLCP(this.metrics.lcp),
      fid: this.assessFID(this.metrics.fid),
      cls: this.assessCLS(this.metrics.cls),
    };

    return assessment;
  }

  // Assess FCP
  assessFCP(fcp) {
    if (!fcp) return 'unknown';
    if (fcp <= 1800) return 'good';
    if (fcp <= 3000) return 'needs-improvement';
    return 'poor';
  }

  // Assess LCP
  assessLCP(lcp) {
    if (!lcp) return 'unknown';
    if (lcp <= 2500) return 'good';
    if (lcp <= 4000) return 'needs-improvement';
    return 'poor';
  }

  // Assess FID
  assessFID(fid) {
    if (!fid) return 'unknown';
    if (fid <= 100) return 'good';
    if (fid <= 300) return 'needs-improvement';
    return 'poor';
  }

  // Assess CLS
  assessCLS(cls) {
    if (!cls) return 'unknown';
    if (cls <= 0.1) return 'good';
    if (cls <= 0.25) return 'needs-improvement';
    return 'poor';
  }

  // Observer pattern for real-time monitoring
  addObserver(callback) {
    this.observers.push(callback);
  }

  removeObserver(callback) {
    const index = this.observers.indexOf(callback);
    if (index > -1) {
      this.observers.splice(index, 1);
    }
  }

  notifyObservers(metric, value) {
    this.observers.forEach(callback => {
      callback(metric, value);
    });
  }

  // Reset metrics
  reset() {
    this.metrics = {};
    console.log('Core Web Vitals metrics reset');
  }
}

export default new CoreWebVitals();
```

---

## 🚨 **Crash Reporting**

### **Sentry Integration**
```javascript
// services/crashReporting.js
import * as Sentry from '@sentry/react-native';
import analytics from '../services/analyticsService';

class CrashReporting {
  constructor() {
    this.isEnabled = true;
    this.userContext = {};
  }

  // Initialize Sentry
  initialize(dsn, environment = 'development') {
    Sentry.init({
      dsn,
      environment,
      enableAutoSessionTracking: true,
      sessionTrackingIntervalMillis: 30000,
      enableTracing: true,
      tracesSampleRate: environment === 'production' ? 0.1 : 1.0,
      integrations: [
        new Sentry.ReactNativeTracing({
          routingInstrumentation: null, // Will be set later
        }),
      ],
      beforeSend: this.beforeSend.bind(this),
      beforeBreadcrumb: this.beforeBreadcrumb.bind(this),
    });

    console.log('Crash reporting initialized');
  }

  // Set user context
  setUserContext(user) {
    this.userContext = {
      id: user.id,
      email: user.email,
      username: user.username,
    };

    Sentry.setUser(this.userContext);
  }

  // Set tags
  setTag(key, value) {
    Sentry.setTag(key, value);
  }

  // Set extra context
  setExtra(key, value) {
    Sentry.setExtra(key, value);
  }

  // Capture exception
  captureException(error, context = {}) {
    if (!this.isEnabled) return;

    console.error('Capturing exception:', error);

    // Track error in analytics
    analytics.trackError(error, context);

    Sentry.withScope((scope) => {
      // Add context
      Object.keys(context).forEach(key => {
        scope.setExtra(key, context[key]);
      });

      // Add tags
      if (context.tags) {
        Object.keys(context.tags).forEach(key => {
          scope.setTag(key, context.tags[key]);
        });
      }

      Sentry.captureException(error);
    });
  }

  // Capture message
  captureMessage(message, level = 'info', context = {}) {
    if (!this.isEnabled) return;

    Sentry.withScope((scope) => {
      scope.setLevel(level);

      Object.keys(context).forEach(key => {
        scope.setExtra(key, context[key]);
      });

      Sentry.captureMessage(message);
    });
  }

  // Add breadcrumb
  addBreadcrumb(message, category = 'custom', level = 'info', data = {}) {
    Sentry.addBreadcrumb({
      message,
      category,
      level,
      data,
    });
  }

  // Before send hook
  beforeSend(event, hint) {
    // Filter out development errors
    if (__DEV__ && event.exception) {
      console.log('Development error filtered out:', event.exception);
      return null;
    }

    // Add custom data
    event.extra = {
      ...event.extra,
      appVersion: '1.0.0',
      userAgent: 'React Native App',
    };

    return event;
  }

  // Before breadcrumb hook
  beforeBreadcrumb(breadcrumb, hint) {
    // Filter out sensitive breadcrumbs
    if (breadcrumb.category === 'http' && breadcrumb.data?.url?.includes('password')) {
      return null;
    }

    return breadcrumb;
  }

  // Performance monitoring
  startTransaction(name, operation) {
    return Sentry.startTransaction({
      name,
      op: operation,
    });
  }

  // Manual crash
  crash() {
    throw new Error('Manual crash for testing');
  }

  // Enable/disable reporting
  setEnabled(enabled) {
    this.isEnabled = enabled;
    console.log(`Crash reporting ${enabled ? 'enabled' : 'disabled'}`);
  }

  // Flush pending events
  async flush(timeout = 2000) {
    return await Sentry.flush(timeout);
  }

  // Close connection
  async close() {
    return await Sentry.close();
  }
}

export default new CrashReporting();
```

---

## 🧪 **A/B Testing**

### **Firebase Remote Config**
```javascript
// services/abTesting.js
import remoteConfig from '@react-native-firebase/remote-config';
import analytics from '../services/analyticsService';

class ABTesting {
  constructor() {
    this.config = {};
    this.experiments = new Map();
  }

  // Initialize remote config
  async initialize() {
    try {
      await remoteConfig().setDefaults({
        // Default values
        feature_enabled: false,
        button_color: 'blue',
        max_items: 10,
        experiment_variant: 'control',
      });

      await remoteConfig().fetchAndActivate();

      // Get all config values
      this.config = remoteConfig().getAll();

      console.log('A/B Testing initialized');
    } catch (error) {
      console.error('A/B Testing initialization error:', error);
    }
  }

  // Get config value
  getValue(key) {
    try {
      return this.config[key]?._value || null;
    } catch (error) {
      console.error(`Error getting config value for ${key}:`, error);
      return null;
    }
  }

  // Get boolean value
  getBoolean(key) {
    const value = this.getValue(key);
    return value === 'true' || value === true;
  }

  // Get string value
  getString(key) {
    return String(this.getValue(key) || '');
  }

  // Get number value
  getNumber(key) {
    const value = this.getValue(key);
    const num = Number(value);
    return isNaN(num) ? 0 : num;
  }

  // Start experiment
  startExperiment(experimentName, variants) {
    const experiment = {
      name: experimentName,
      variants,
      startTime: new Date().toISOString(),
      participants: 0,
    };

    this.experiments.set(experimentName, experiment);

    // Track experiment start
    analytics.trackEvent('experiment_started', {
      experiment_name: experimentName,
      variants_count: variants.length,
    });

    console.log(`Experiment started: ${experimentName}`);
  }

  // Assign user to variant
  assignUserToVariant(experimentName, userId) {
    const experiment = this.experiments.get(experimentName);
    if (!experiment) return null;

    // Simple hash-based assignment for consistency
    const hash = this.hashString(userId + experimentName);
    const variantIndex = Math.abs(hash) % experiment.variants.length;
    const assignedVariant = experiment.variants[variantIndex];

    experiment.participants++;

    // Track assignment
    analytics.trackEvent('experiment_assignment', {
      experiment_name: experimentName,
      variant: assignedVariant,
      user_id: userId,
    });

    return assignedVariant;
  }

  // Track experiment event
  trackExperimentEvent(experimentName, eventName, variant, data = {}) {
    analytics.trackEvent(`experiment_${eventName}`, {
      experiment_name: experimentName,
      variant,
      ...data,
    });
  }

  // Get experiment results
  getExperimentResults(experimentName) {
    const experiment = this.experiments.get(experimentName);
    if (!experiment) return null;

    // This would typically fetch results from analytics
    // For now, return basic info
    return {
      name: experiment.name,
      participants: experiment.participants,
      startTime: experiment.startTime,
      variants: experiment.variants,
    };
  }

  // End experiment
  endExperiment(experimentName) {
    const experiment = this.experiments.get(experimentName);
    if (!experiment) return;

    experiment.endTime = new Date().toISOString();

    // Track experiment end
    analytics.trackEvent('experiment_ended', {
      experiment_name: experimentName,
      duration: new Date(experiment.endTime) - new Date(experiment.startTime),
      participants: experiment.participants,
    });

    console.log(`Experiment ended: ${experimentName}`);
  }

  // Simple string hash function
  hashString(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convert to 32-bit integer
    }
    return hash;
  }

  // Feature flag
  isFeatureEnabled(featureName) {
    return this.getBoolean(`${featureName}_enabled`);
  }

  // Get feature variant
  getFeatureVariant(featureName, userId) {
    const variants = this.getString(`${featureName}_variants`);
    if (!variants) return 'control';

    const variantList = variants.split(',');
    return this.assignUserToVariant(featureName, userId) || variantList[0];
  }

  // Refresh config
  async refreshConfig() {
    try {
      const activated = await remoteConfig().fetchAndActivate();
      if (activated) {
        this.config = remoteConfig().getAll();
        console.log('Remote config refreshed');
      }
    } catch (error) {
      console.error('Config refresh error:', error);
    }
  }
}

export default new ABTesting();
```

---

## 📊 **Performance Dashboards**

### **Custom Dashboard Component**
```javascript
// components/PerformanceDashboard.js
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, RefreshControl, StyleSheet } from 'react-native';
import analytics from '../services/analyticsService';
import performanceMonitoring from '../services/performanceMonitoring';
import coreWebVitals from '../services/coreWebVitals';

const PerformanceDashboard = () => {
  const [metrics, setMetrics] = useState({});
  const [refreshing, setRefreshing] = useState(false);
  const [lastUpdate, setLastUpdate] = useState(new Date());

  useEffect(() => {
    loadMetrics();
  }, []);

  const loadMetrics = async () => {
    try {
      const [analyticsData, performanceData, vitalsData] = await Promise.all([
        getAnalyticsMetrics(),
        performanceMonitoring.getPerformanceMetrics(),
        getVitalsMetrics(),
      ]);

      setMetrics({
        analytics: analyticsData,
        performance: performanceData,
        vitals: vitalsData,
      });

      setLastUpdate(new Date());
    } catch (error) {
      console.error('Error loading metrics:', error);
    }
  };

  const getAnalyticsMetrics = async () => {
    // This would fetch real analytics data
    return {
      totalUsers: 1250,
      activeUsers: 890,
      sessionDuration: 245, // seconds
      bounceRate: 0.32,
      topScreens: [
        { name: 'Home', views: 1250 },
        { name: 'Profile', views: 890 },
        { name: 'Settings', views: 456 },
      ],
    };
  };

  const getVitalsMetrics = () => {
    const vitals = coreWebVitals.getMetrics();
    const assessment = coreWebVitals.getMetricsAssessment();

    return {
      fcp: vitals.fcp,
      lcp: vitals.lcp,
      fid: vitals.fid,
      cls: vitals.cls,
      assessment,
    };
  };

  const onRefresh = async () => {
    setRefreshing(true);
    await loadMetrics();
    setRefreshing(false);
  };

  const formatTime = (ms) => {
    return ms ? `${ms.toFixed(0)}ms` : 'N/A';
  };

  const formatPercentage = (value) => {
    return value ? `${(value * 100).toFixed(1)}%` : 'N/A';
  };

  const getAssessmentColor = (assessment) => {
    switch (assessment) {
      case 'good': return '#28a745';
      case 'needs-improvement': return '#ffc107';
      case 'poor': return '#dc3545';
      default: return '#6c757d';
    }
  };

  return (
    <ScrollView
      style={styles.container}
      refreshControl={
        <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
      }
    >
      <Text style={styles.title}>Performance Dashboard</Text>
      <Text style={styles.lastUpdate}>
        Last updated: {lastUpdate.toLocaleString()}
      </Text>

      {/* Analytics Metrics */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Analytics</Text>

        <View style={styles.metricGrid}>
          <View style={styles.metric}>
            <Text style={styles.metricValue}>{metrics.analytics?.totalUsers || 0}</Text>
            <Text style={styles.metricLabel}>Total Users</Text>
          </View>

          <View style={styles.metric}>
            <Text style={styles.metricValue}>{metrics.analytics?.activeUsers || 0}</Text>
            <Text style={styles.metricLabel}>Active Users</Text>
          </View>

          <View style={styles.metric}>
            <Text style={styles.metricValue}>
              {metrics.analytics?.sessionDuration ?
                `${Math.round(metrics.analytics.sessionDuration / 60)}m` : '0m'}
            </Text>
            <Text style={styles.metricLabel}>Avg Session</Text>
          </View>

          <View style={styles.metric}>
            <Text style={styles.metricValue}>
              {formatPercentage(metrics.analytics?.bounceRate)}
            </Text>
            <Text style={styles.metricLabel}>Bounce Rate</Text>
          </View>
        </View>

        {/* Top Screens */}
        <View style={styles.subSection}>
          <Text style={styles.subSectionTitle}>Top Screens</Text>
          {metrics.analytics?.topScreens?.map((screen, index) => (
            <View key={index} style={styles.screenItem}>
              <Text style={styles.screenName}>{screen.name}</Text>
              <Text style={styles.screenViews}>{screen.views} views</Text>
            </View>
          ))}
        </View>
      </View>

      {/* Performance Metrics */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Performance</Text>

        <View style={styles.metricGrid}>
          <View style={styles.metric}>
            <Text style={styles.metricValue}>
              {formatTime(metrics.performance?.app_startup_time)}
            </Text>
            <Text style={styles.metricLabel}>App Startup</Text>
          </View>
        </View>
      </View>

      {/* Core Web Vitals */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Core Web Vitals</Text>

        <View style={styles.vitalsGrid}>
          <View style={styles.vital}>
            <Text style={styles.vitalLabel}>FCP</Text>
            <Text style={styles.vitalValue}>{formatTime(metrics.vitals?.fcp)}</Text>
            <View
              style={[
                styles.assessmentIndicator,
                { backgroundColor: getAssessmentColor(metrics.vitals?.assessment?.fcp) }
              ]}
            />
          </View>

          <View style={styles.vital}>
            <Text style={styles.vitalLabel}>LCP</Text>
            <Text style={styles.vitalValue}>{formatTime(metrics.vitals?.lcp)}</Text>
            <View
              style={[
                styles.assessmentIndicator,
                { backgroundColor: getAssessmentColor(metrics.vitals?.assessment?.lcp) }
              ]}
            />
          </View>

          <View style={styles.vital}>
            <Text style={styles.vitalLabel}>FID</Text>
            <Text style={styles.vitalValue}>{formatTime(metrics.vitals?.fid)}</Text>
            <View
              style={[
                styles.assessmentIndicator,
                { backgroundColor: getAssessmentColor(metrics.vitals?.assessment?.fid) }
              ]}
            />
          </View>

          <View style={styles.vital}>
            <Text style={styles.vitalLabel}>CLS</Text>
            <Text style={styles.vitalValue}>{metrics.vitals?.cls?.toFixed(3) || 'N/A'}</Text>
            <View
              style={[
                styles.assessmentIndicator,
                { backgroundColor: getAssessmentColor(metrics.vitals?.assessment?.cls) }
              ]}
            />
          </View>
        </View>

        <View style={styles.legend}>
          <View style={styles.legendItem}>
            <View style={[styles.legendColor, { backgroundColor: '#28a745' }]} />
            <Text style={styles.legendText}>Good</Text>
          </View>
          <View style={styles.legendItem}>
            <View style={[styles.legendColor, { backgroundColor: '#ffc107' }]} />
            <Text style={styles.legendText}>Needs Work</Text>
          </View>
          <View style={styles.legendItem}>
            <View style={[styles.legendColor, { backgroundColor: '#dc3545' }]} />
            <Text style={styles.legendText}>Poor</Text>
          </View>
        </View>
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
  lastUpdate: {
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
  metric: {
    width: '48%',
    alignItems: 'center',
    marginBottom: 15,
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
  subSection: {
    marginTop: 15,
  },
  subSectionTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  screenItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    paddingVertical: 5,
  },
  screenName: {
    fontSize: 14,
  },
  screenViews: {
    fontSize: 14,
    color: '#666',
  },
  vitalsGrid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-between',
  },
  vital: {
    width: '48%',
    alignItems: 'center',
    marginBottom: 15,
  },
  vitalLabel: {
    fontSize: 12,
    color: '#666',
    marginBottom: 5,
  },
  vitalValue: {
    fontSize: 18,
    fontWeight: 'bold',
  },
  assessmentIndicator: {
    width: 20,
    height: 20,
    borderRadius: 10,
    marginTop: 5,
  },
  legend: {
    flexDirection: 'row',
    justifyContent: 'center',
    marginTop: 10,
  },
  legendItem: {
    flexDirection: 'row',
    alignItems: 'center',
    marginHorizontal: 10,
  },
  legendColor: {
    width: 12,
    height: 12,
    borderRadius: 6,
    marginRight: 5,
  },
  legendText: {
    fontSize: 12,
    color: '#666',
  },
});

export default PerformanceDashboard;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Analytics Implementation**: Firebase Analytics setup and custom events
- ✅ **Real User Monitoring**: Performance monitoring and error tracking
- ✅ **Performance Metrics**: Core Web Vitals and custom metrics
- ✅ **Crash Reporting**: Sentry integration and error handling
- ✅ **A/B Testing**: Firebase Remote Config and experiment management
- ✅ **Performance Dashboards**: Real-time metrics visualization
- ✅ **Alerting & Notifications**: Automated alerts for performance issues
- ✅ **Data Analysis & Insights**: User behavior analysis and insights

### **Best Practices**
1. **Implement comprehensive analytics** from the start of development
2. **Monitor key performance indicators** regularly
3. **Set up proper error tracking** and crash reporting
4. **Use A/B testing** for data-driven feature decisions
5. **Create performance dashboards** for real-time monitoring
6. **Implement alerting** for critical performance issues
7. **Analyze user behavior** to improve app experience
8. **Regularly review and optimize** based on analytics data
9. **Ensure data privacy** and compliance in analytics
10. **Use performance budgets** to maintain app quality

### **Next Steps**
- Learn advanced analytics techniques
- Implement custom performance monitoring
- Set up automated alerting systems
- Create detailed user journey analytics
- Implement predictive analytics
- Learn about heatmaps and user recordings

---

## 🎯 **Assignment**

### **Task 1: Analytics Implementation**
Create a comprehensive analytics system with:
- User registration and login tracking
- Screen navigation analytics
- Feature usage tracking
- Error and crash reporting
- Performance metrics collection
- Custom event tracking for business metrics

### **Task 2: Performance Monitoring**
Implement performance monitoring for:
- App startup time
- Screen loading times
- Network request performance
- Memory usage tracking
- Frame rate monitoring
- Core Web Vitals measurement

### **Task 3: A/B Testing Framework**
Build an A/B testing system that:
- Supports multiple concurrent experiments
- Provides variant assignment logic
- Tracks experiment metrics
- Generates experiment reports
- Integrates with analytics
- Supports feature flags

---

## 📚 **Additional Resources**
- [Firebase Analytics Documentation](https://firebase.google.com/docs/analytics)
- [Firebase Performance Monitoring](https://firebase.google.com/docs/perf-mon)
- [Sentry Documentation](https://docs.sentry.io/)
- [Google Analytics for Firebase](https://firebase.google.com/docs/analytics)
- [Core Web Vitals](https://web.dev/vitals/)
- [A/B Testing Best Practices](https://www.optimizely.com/optimization-glossary/ab-testing/)

---

**Next Lesson**: [Lesson 48: Advanced State Management Patterns](Lesson%2048_%20Advanced%20State%20Management%20Patterns.md)