# Lesson 56: Monitoring & Alerting

## 🎯 **Learning Objectives**
- Implement comprehensive monitoring for React Native applications
- Set up alerting systems for critical issues
- Monitor application performance and user experience
- Track errors and crashes in production
- Create dashboards for real-time insights

## 📚 **Table of Contents**
1. [Monitoring Fundamentals](#monitoring-fundamentals)
2. [Application Performance Monitoring](#application-performance-monitoring)
3. [Error Tracking & Crash Reporting](#error-tracking--crash-reporting)
4. [Real User Monitoring](#real-user-monitoring)
5. [Infrastructure Monitoring](#infrastructure-monitoring)
6. [Alerting Systems](#alerting-systems)
7. [Dashboard Creation](#dashboard-creation)
8. [Log Management](#log-management)
9. [Analytics Integration](#analytics-integration)
10. [Practical Examples](#practical-examples)

---

## 📊 **Monitoring Fundamentals**

### **Monitoring Types**
- **Application Performance Monitoring (APM)**: Track app performance metrics
- **Error Tracking**: Capture and analyze errors and crashes
- **Real User Monitoring (RUM)**: Monitor actual user experience
- **Infrastructure Monitoring**: Track server and resource usage
- **Business Metrics**: Monitor user engagement and conversion

### **Key Metrics to Monitor**
```javascript
const monitoringMetrics = {
  // Performance Metrics
  appStartTime: 'Time to start the application',
  jsBundleLoadTime: 'Time to load JavaScript bundle',
  networkRequestTime: 'API response times',
  uiRenderTime: 'Screen render times',

  // Error Metrics
  crashRate: 'Application crash frequency',
  errorRate: 'JavaScript error frequency',
  networkErrorRate: 'Network failure rate',

  // User Experience Metrics
  sessionDuration: 'Average user session length',
  screenLoadTime: 'Time to load screens',
  userRetention: 'User retention rates',

  // Business Metrics
  dailyActiveUsers: 'Daily active user count',
  conversionRate: 'User conversion rates',
  featureUsage: 'Feature adoption rates'
};
```

### **Monitoring Tools**
- **Sentry**: Error tracking and performance monitoring
- **DataDog**: Comprehensive monitoring and analytics
- **New Relic**: Application performance monitoring
- **Firebase Crashlytics**: Crash reporting for mobile apps
- **App Center Analytics**: Microsoft App Center analytics
- **Grafana**: Dashboard creation and visualization

---

## 📈 **Application Performance Monitoring**

### **React Native Performance Monitoring**
```javascript
// services/performanceMonitor.js
import { PerformanceObserver, performance } from 'react-native';

class PerformanceMonitor {
  constructor() {
    this.metrics = new Map();
    this.observers = [];
    this.isEnabled = __DEV__ ? false : true; // Disable in development
  }

  // Initialize performance monitoring
  initialize() {
    if (!this.isEnabled) return;

    // Monitor app start time
    this.monitorAppStartTime();

    // Monitor navigation performance
    this.monitorNavigationPerformance();

    // Monitor network requests
    this.monitorNetworkRequests();

    // Monitor memory usage
    this.monitorMemoryUsage();

    // Monitor JavaScript errors
    this.monitorJSErrors();
  }

  // Monitor app start time
  monitorAppStartTime() {
    const startTime = performance.now();

    // Use InteractionManager to measure when app is ready
    InteractionManager.runAfterInteractions(() => {
      const endTime = performance.now();
      const appStartTime = endTime - startTime;

      this.recordMetric('app_start_time', appStartTime);
      console.log(`App started in ${appStartTime.toFixed(2)}ms`);
    });
  }

  // Monitor navigation performance
  monitorNavigationPerformance() {
    // Monitor screen transitions
    const navigationListener = (route) => {
      const startTime = performance.now();

      // Measure screen render time
      setTimeout(() => {
        const endTime = performance.now();
        const renderTime = endTime - startTime;

        this.recordMetric('screen_render_time', renderTime, {
          screen: route.name
        });
      }, 100);
    };

    // Add navigation listener (depends on navigation library)
    // navigation.addListener('state', navigationListener);
  }

  // Monitor network requests
  monitorNetworkRequests() {
    // Intercept network requests
    const originalFetch = global.fetch;

    global.fetch = async (...args) => {
      const startTime = performance.now();
      const url = args[0];

      try {
        const response = await originalFetch(...args);
        const endTime = performance.now();
        const requestTime = endTime - startTime;

        this.recordMetric('network_request_time', requestTime, {
          url: url.toString(),
          status: response.status,
          method: args[1]?.method || 'GET'
        });

        return response;
      } catch (error) {
        const endTime = performance.now();
        const requestTime = endTime - startTime;

        this.recordMetric('network_request_error', requestTime, {
          url: url.toString(),
          error: error.message
        });

        throw error;
      }
    };
  }

  // Monitor memory usage
  monitorMemoryUsage() {
    if (Platform.OS === 'android') {
      // Android memory monitoring
      const interval = setInterval(() => {
        // Get memory info (requires native module)
        // const memoryInfo = MemoryModule.getMemoryInfo();
        // this.recordMetric('memory_usage', memoryInfo.used, {
        //   total: memoryInfo.total,
        //   available: memoryInfo.available
        // });
      }, 30000); // Every 30 seconds

      this.intervals.push(interval);
    }
  }

  // Monitor JavaScript errors
  monitorJSErrors() {
    const originalError = console.error;

    console.error = (...args) => {
      // Log original error
      originalError.apply(console, args);

      // Record error metric
      this.recordMetric('js_error', 1, {
        message: args.join(' '),
        timestamp: new Date().toISOString()
      });
    };

    // Global error handler
    ErrorUtils.setGlobalHandler((error, isFatal) => {
      this.recordMetric('js_error_fatal', 1, {
        message: error.message,
        stack: error.stack,
        isFatal
      });

      // Re-throw for default handling
      throw error;
    });
  }

  // Record metric
  recordMetric(name, value, tags = {}) {
    if (!this.isEnabled) return;

    const metric = {
      name,
      value,
      tags,
      timestamp: Date.now()
    };

    // Store locally for batch sending
    const key = `${name}_${JSON.stringify(tags)}`;
    this.metrics.set(key, metric);

    // Send to monitoring service
    this.sendToMonitoringService(metric);
  }

  // Send metrics to monitoring service
  async sendToMonitoringService(metric) {
    try {
      // Send to your monitoring service (Sentry, DataDog, etc.)
      // await monitoringService.sendMetric(metric);

      console.log('Metric recorded:', metric);
    } catch (error) {
      console.error('Failed to send metric:', error);
    }
  }

  // Get metrics summary
  getMetricsSummary() {
    const summary = {};

    for (const [key, metric] of this.metrics.entries()) {
      const baseKey = key.split('_')[0];
      if (!summary[baseKey]) {
        summary[baseKey] = [];
      }
      summary[baseKey].push(metric);
    }

    return summary;
  }

  // Clear old metrics
  clearOldMetrics(maxAge = 24 * 60 * 60 * 1000) { // 24 hours
    const cutoff = Date.now() - maxAge;

    for (const [key, metric] of this.metrics.entries()) {
      if (metric.timestamp < cutoff) {
        this.metrics.delete(key);
      }
    }
  }

  // Enable/disable monitoring
  setEnabled(enabled) {
    this.isEnabled = enabled;
  }

  // Cleanup
  destroy() {
    this.intervals.forEach(clearInterval);
    this.observers.forEach(observer => observer.disconnect());
  }
}

export const performanceMonitor = new PerformanceMonitor();
```

### **Custom Performance Hooks**
```javascript
// hooks/usePerformanceMonitor.js
import { useEffect, useRef } from 'react';
import { performanceMonitor } from '../services/performanceMonitor';

export const usePerformanceMonitor = (componentName) => {
  const startTimeRef = useRef(null);
  const renderCountRef = useRef(0);

  useEffect(() => {
    startTimeRef.current = performance.now();
    renderCountRef.current = 0;
  }, []);

  useEffect(() => {
    renderCountRef.current += 1;

    // Monitor excessive re-renders
    if (renderCountRef.current > 10) {
      performanceMonitor.recordMetric('excessive_renders', renderCountRef.current, {
        component: componentName
      });
    }
  });

  const trackInteraction = (interactionName) => {
    const startTime = performance.now();

    return () => {
      const endTime = performance.now();
      const duration = endTime - startTime;

      performanceMonitor.recordMetric('user_interaction_time', duration, {
        component: componentName,
        interaction: interactionName
      });
    };
  };

  const trackAsyncOperation = async (operationName, operation) => {
    const startTime = performance.now();

    try {
      const result = await operation();
      const endTime = performance.now();
      const duration = endTime - startTime;

      performanceMonitor.recordMetric('async_operation_time', duration, {
        component: componentName,
        operation: operationName,
        success: true
      });

      return result;
    } catch (error) {
      const endTime = performance.now();
      const duration = endTime - startTime;

      performanceMonitor.recordMetric('async_operation_time', duration, {
        component: componentName,
        operation: operationName,
        success: false,
        error: error.message
      });

      throw error;
    }
  };

  return {
    trackInteraction,
    trackAsyncOperation
  };
};
```

---

## 🚨 **Error Tracking & Crash Reporting**

### **Sentry Integration**
```javascript
// services/sentryService.js
import * as Sentry from '@sentry/react-native';

class SentryService {
  constructor() {
    this.isInitialized = false;
  }

  // Initialize Sentry
  initialize(dsn, environment = 'production') {
    if (this.isInitialized) return;

    Sentry.init({
      dsn,
      environment,
      // Enable performance monitoring
      enableTracing: true,
      // Set traces sample rate
      tracesSampleRate: environment === 'production' ? 0.1 : 1.0,
      // Capture console logs
      beforeSend: (event) => {
        console.log('Sending error to Sentry:', event);
        return event;
      },
      // Configure integrations
      integrations: [
        new Sentry.ReactNativeTracing({
          routingInstrumentation: Sentry.reactNavigationIntegration(),
        }),
      ],
    });

    this.isInitialized = true;
    console.log('Sentry initialized for environment:', environment);
  }

  // Set user context
  setUser(user) {
    Sentry.setUser({
      id: user.id,
      email: user.email,
      username: user.username,
    });
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
    Sentry.withScope((scope) => {
      // Add context
      Object.keys(context).forEach(key => {
        scope.setTag(key, context[key]);
      });

      // Add extra information
      scope.setExtra('componentStack', error.componentStack);
      scope.setExtra('errorBoundary', error.errorBoundary);

      Sentry.captureException(error);
    });
  }

  // Capture message
  captureMessage(message, level = 'info', context = {}) {
    Sentry.withScope((scope) => {
      scope.setLevel(level);

      Object.keys(context).forEach(key => {
        scope.setTag(key, context[key]);
      });

      Sentry.captureMessage(message);
    });
  }

  // Add breadcrumb
  addBreadcrumb(message, category = 'custom', level = 'info') {
    Sentry.addBreadcrumb({
      message,
      category,
      level,
    });
  }

  // Performance monitoring
  startTransaction(name, operation) {
    return Sentry.startTransaction({
      name,
      op: operation,
    });
  }

  // Flush pending events
  async flush(timeout = 2000) {
    return await Sentry.flush(timeout);
  }

  // Close Sentry
  async close(timeout = 2000) {
    return await Sentry.close(timeout);
  }
}

export const sentryService = new SentryService();
```

### **Firebase Crashlytics Integration**
```javascript
// services/crashlyticsService.js
import crashlytics from '@react-native-firebase/crashlytics';

class CrashlyticsService {
  constructor() {
    this.isEnabled = !__DEV__; // Disable in development
  }

  // Initialize Crashlytics
  initialize() {
    if (!this.isEnabled) return;

    // Enable Crashlytics
    crashlytics().setCrashlyticsCollectionEnabled(true);

    console.log('Firebase Crashlytics initialized');
  }

  // Set user identifier
  setUserId(userId) {
    if (!this.isEnabled) return;

    crashlytics().setUserId(userId);
  }

  // Set user attributes
  setAttributes(attributes) {
    if (!this.isEnabled) return;

    Object.keys(attributes).forEach(key => {
      crashlytics().setAttribute(key, attributes[key]);
    });
  }

  // Log custom events
  log(message) {
    if (!this.isEnabled) return;

    crashlytics().log(message);
  }

  // Record error
  recordError(error, context = {}) {
    if (!this.isEnabled) return;

    crashlytics().recordError(error, context);
  }

  // Simulate crash (for testing)
  crash() {
    if (!this.isEnabled) return;

    crashlytics().crash();
  }

  // Enable/disable Crashlytics
  setEnabled(enabled) {
    this.isEnabled = enabled;
    crashlytics().setCrashlyticsCollectionEnabled(enabled);
  }

  // Check if Crashlytics is enabled
  isEnabled() {
    return this.isEnabled;
  }
}

export const crashlyticsService = new CrashlyticsService();
```

### **Error Boundary Component**
```javascript
// components/ErrorBoundary.js
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { sentryService } from '../services/sentryService';
import { crashlyticsService } from '../services/crashlyticsService';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Log error to monitoring services
    sentryService.captureException(error, {
      componentStack: errorInfo.componentStack,
      errorBoundary: true
    });

    crashlyticsService.recordError(error, {
      componentStack: errorInfo.componentStack,
      errorBoundary: true
    });

    this.setState({
      error,
      errorInfo
    });

    // Call optional error handler
    if (this.props.onError) {
      this.props.onError(error, errorInfo);
    }
  }

  handleRetry = () => {
    this.setState({ hasError: false, error: null, errorInfo: null });
  };

  handleReport = () => {
    // Additional error reporting logic
    console.log('Error reported by user');
  };

  render() {
    if (this.state.hasError) {
      const { fallback: Fallback, showDetails = __DEV__ } = this.props;

      if (Fallback) {
        return <Fallback error={this.state.error} retry={this.handleRetry} />;
      }

      return (
        <View style={styles.container}>
          <Text style={styles.title}>Oops! Something went wrong</Text>
          <Text style={styles.message}>
            We're sorry for the inconvenience. Please try again.
          </Text>

          {showDetails && this.state.error && (
            <View style={styles.errorDetails}>
              <Text style={styles.errorTitle}>Error Details:</Text>
              <Text style={styles.errorText}>{this.state.error.toString()}</Text>
              {this.state.errorInfo && (
                <Text style={styles.errorStack}>{this.state.errorInfo.componentStack}</Text>
              )}
            </View>
          )}

          <View style={styles.buttonContainer}>
            <TouchableOpacity style={styles.retryButton} onPress={this.handleRetry}>
              <Text style={styles.retryButtonText}>Try Again</Text>
            </TouchableOpacity>

            <TouchableOpacity style={styles.reportButton} onPress={this.handleReport}>
              <Text style={styles.reportButtonText}>Report Issue</Text>
            </TouchableOpacity>
          </View>
        </View>
      );
    }

    return this.props.children;
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 10,
    textAlign: 'center',
  },
  message: {
    fontSize: 16,
    color: '#666',
    textAlign: 'center',
    marginBottom: 20,
  },
  errorDetails: {
    backgroundColor: '#fff',
    padding: 15,
    borderRadius: 8,
    marginBottom: 20,
    width: '100%',
  },
  errorTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#d32f2f',
    marginBottom: 10,
  },
  errorText: {
    fontSize: 14,
    color: '#333',
    fontFamily: 'monospace',
    marginBottom: 10,
  },
  errorStack: {
    fontSize: 12,
    color: '#666',
    fontFamily: 'monospace',
  },
  buttonContainer: {
    flexDirection: 'row',
    gap: 10,
  },
  retryButton: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 8,
    flex: 1,
    alignItems: 'center',
  },
  retryButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  reportButton: {
    backgroundColor: '#28a745',
    padding: 15,
    borderRadius: 8,
    flex: 1,
    alignItems: 'center',
  },
  reportButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default ErrorBoundary;
```

---

## 👥 **Real User Monitoring**

### **User Experience Tracking**
```javascript
// services/rumService.js
import analytics from '@react-native-firebase/analytics';

class RUMService {
  constructor() {
    this.sessionId = this.generateSessionId();
    this.sessionStartTime = Date.now();
    this.screenStartTime = null;
    this.currentScreen = null;
  }

  // Generate unique session ID
  generateSessionId() {
    return `session_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  // Track app launch
  trackAppLaunch() {
    analytics().logEvent('app_open', {
      session_id: this.sessionId,
      platform: Platform.OS,
      app_version: '1.0.0', // Get from package.json or native
    });
  }

  // Track screen view
  trackScreenView(screenName, screenClass = null) {
    // End previous screen tracking
    if (this.currentScreen && this.screenStartTime) {
      const duration = Date.now() - this.screenStartTime;
      analytics().logEvent('screen_view_duration', {
        screen_name: this.currentScreen,
        duration: duration,
        session_id: this.sessionId,
      });
    }

    // Start new screen tracking
    this.currentScreen = screenName;
    this.screenStartTime = Date.now();

    analytics().logEvent('screen_view', {
      screen_name: screenName,
      screen_class: screenClass,
      session_id: this.sessionId,
    });
  }

  // Track user interaction
  trackUserInteraction(interactionType, elementName, context = {}) {
    analytics().logEvent('user_interaction', {
      interaction_type: interactionType,
      element_name: elementName,
      session_id: this.sessionId,
      timestamp: Date.now(),
      ...context,
    });
  }

  // Track performance metrics
  trackPerformanceMetric(metricName, value, unit = 'ms') {
    analytics().logEvent('performance_metric', {
      metric_name: metricName,
      value: value,
      unit: unit,
      session_id: this.sessionId,
      timestamp: Date.now(),
    });
  }

  // Track error from user perspective
  trackUserError(errorType, errorMessage, context = {}) {
    analytics().logEvent('user_error', {
      error_type: errorType,
      error_message: errorMessage,
      session_id: this.sessionId,
      timestamp: Date.now(),
      ...context,
    });
  }

  // Track session end
  trackSessionEnd() {
    const sessionDuration = Date.now() - this.sessionStartTime;

    analytics().logEvent('session_end', {
      session_id: this.sessionId,
      session_duration: sessionDuration,
      final_screen: this.currentScreen,
    });
  }

  // Track custom events
  trackCustomEvent(eventName, parameters = {}) {
    analytics().logEvent(eventName, {
      session_id: this.sessionId,
      timestamp: Date.now(),
      ...parameters,
    });
  }

  // Set user properties
  setUserProperty(name, value) {
    analytics().setUserProperty(name, value);
  }

  // Set user ID
  setUserId(userId) {
    analytics().setUserId(userId);
  }

  // Get current session info
  getCurrentSession() {
    return {
      sessionId: this.sessionId,
      startTime: this.sessionStartTime,
      duration: Date.now() - this.sessionStartTime,
      currentScreen: this.currentScreen,
    };
  }
}

export const rumService = new RUMService();
```

### **Navigation Tracking Hook**
```javascript
// hooks/useNavigationTracking.js
import { useEffect } from 'react';
import { useNavigation } from '@react-navigation/native';
import { rumService } from '../services/rumService';

export const useNavigationTracking = () => {
  const navigation = useNavigation();

  useEffect(() => {
    const unsubscribe = navigation.addListener('state', (e) => {
      const currentRoute = e.data.state.routes[e.data.state.index];
      rumService.trackScreenView(currentRoute.name, currentRoute.name);
    });

    return unsubscribe;
  }, [navigation]);
};
```

---

## 🖥️ **Infrastructure Monitoring**

### **Server Monitoring**
```javascript
// services/serverMonitor.js
class ServerMonitor {
  constructor() {
    this.metrics = new Map();
    this.alerts = [];
  }

  // Monitor API endpoints
  async monitorAPIEndpoint(url, method = 'GET', expectedStatus = 200) {
    const startTime = Date.now();

    try {
      const response = await fetch(url, { method });
      const responseTime = Date.now() - startTime;

      const isHealthy = response.status === expectedStatus;

      this.recordMetric('api_response_time', responseTime, {
        endpoint: url,
        method,
        status: response.status,
        healthy: isHealthy
      });

      if (!isHealthy) {
        this.createAlert('API_UNHEALTHY', {
          endpoint: url,
          expectedStatus,
          actualStatus: response.status,
          responseTime
        });
      }

      return {
        healthy: isHealthy,
        responseTime,
        status: response.status
      };
    } catch (error) {
      const responseTime = Date.now() - startTime;

      this.recordMetric('api_error', 1, {
        endpoint: url,
        method,
        error: error.message,
        responseTime
      });

      this.createAlert('API_ERROR', {
        endpoint: url,
        error: error.message,
        responseTime
      });

      return {
        healthy: false,
        error: error.message,
        responseTime
      };
    }
  }

  // Monitor database connections
  async monitorDatabaseConnection(connectionString) {
    // Implement database connection monitoring
    // This would vary based on your database type
    console.log('Monitoring database connection:', connectionString);
  }

  // Monitor server resources
  async monitorServerResources(serverUrl) {
    try {
      // This would typically call a monitoring endpoint on your server
      const response = await fetch(`${serverUrl}/health`);
      const data = await response.json();

      this.recordMetric('server_cpu_usage', data.cpu, { server: serverUrl });
      this.recordMetric('server_memory_usage', data.memory, { server: serverUrl });
      this.recordMetric('server_disk_usage', data.disk, { server: serverUrl });

      return data;
    } catch (error) {
      this.createAlert('SERVER_MONITORING_ERROR', {
        server: serverUrl,
        error: error.message
      });
      return null;
    }
  }

  // Record metric
  recordMetric(name, value, tags = {}) {
    const metric = {
      name,
      value,
      tags,
      timestamp: Date.now()
    };

    this.metrics.set(`${name}_${JSON.stringify(tags)}`, metric);

    // Send to monitoring service
    this.sendToMonitoringService(metric);
  }

  // Create alert
  createAlert(type, data) {
    const alert = {
      id: `alert_${Date.now()}`,
      type,
      data,
      timestamp: Date.now(),
      acknowledged: false
    };

    this.alerts.push(alert);

    // Send alert notification
    this.sendAlertNotification(alert);
  }

  // Send metric to monitoring service
  async sendToMonitoringService(metric) {
    // Implement sending to your monitoring service
    console.log('Sending metric:', metric);
  }

  // Send alert notification
  async sendAlertNotification(alert) {
    // Implement alert notification (email, Slack, etc.)
    console.log('Sending alert:', alert);
  }

  // Get active alerts
  getActiveAlerts() {
    return this.alerts.filter(alert => !alert.acknowledged);
  }

  // Acknowledge alert
  acknowledgeAlert(alertId) {
    const alert = this.alerts.find(a => a.id === alertId);
    if (alert) {
      alert.acknowledged = true;
      alert.acknowledgedAt = Date.now();
    }
  }

  // Get metrics summary
  getMetricsSummary() {
    const summary = {};

    for (const [key, metric] of this.metrics.entries()) {
      const baseKey = key.split('_')[0];
      if (!summary[baseKey]) {
        summary[baseKey] = [];
      }
      summary[baseKey].push(metric);
    }

    return summary;
  }
}

export const serverMonitor = new ServerMonitor();
```

---

## 🚨 **Alerting Systems**

### **Alert Manager**
```javascript
// services/alertManager.js
class AlertManager {
  constructor() {
    this.alerts = new Map();
    this.channels = new Map();
    this.rules = [];
  }

  // Register alert channel
  registerChannel(name, handler) {
    this.channels.set(name, handler);
  }

  // Create alert rule
  createRule(name, condition, channels = ['default'], options = {}) {
    const rule = {
      name,
      condition,
      channels,
      options: {
        cooldown: 300000, // 5 minutes
        severity: 'medium',
        ...options
      },
      lastTriggered: 0
    };

    this.rules.push(rule);
    return rule;
  }

  // Trigger alert
  async triggerAlert(ruleName, data) {
    const rule = this.rules.find(r => r.name === ruleName);
    if (!rule) return;

    // Check cooldown
    const now = Date.now();
    if (now - rule.lastTriggered < rule.options.cooldown) {
      return; // Still in cooldown
    }

    // Evaluate condition
    if (typeof rule.condition === 'function') {
      if (!rule.condition(data)) return;
    }

    // Create alert
    const alert = {
      id: `alert_${Date.now()}`,
      rule: ruleName,
      data,
      severity: rule.options.severity,
      timestamp: now,
      channels: rule.channels
    };

    // Store alert
    this.alerts.set(alert.id, alert);
    rule.lastTriggered = now;

    // Send to channels
    await this.sendToChannels(alert);

    return alert;
  }

  // Send alert to channels
  async sendToChannels(alert) {
    for (const channelName of alert.channels) {
      const channel = this.channels.get(channelName);
      if (channel) {
        try {
          await channel.send(alert);
        } catch (error) {
          console.error(`Failed to send alert to channel ${channelName}:`, error);
        }
      }
    }
  }

  // Slack channel
  createSlackChannel(webhookUrl, channel = '#alerts') {
    return {
      send: async (alert) => {
        const payload = {
          channel,
          text: `🚨 Alert: ${alert.rule}`,
          attachments: [{
            color: this.getSeverityColor(alert.severity),
            fields: [
              {
                title: 'Severity',
                value: alert.severity,
                short: true
              },
              {
                title: 'Time',
                value: new Date(alert.timestamp).toISOString(),
                short: true
              }
            ]
          }]
        };

        if (alert.data) {
          payload.attachments[0].fields.push({
            title: 'Details',
            value: JSON.stringify(alert.data, null, 2),
            short: false
          });
        }

        const response = await fetch(webhookUrl, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(payload)
        });

        if (!response.ok) {
          throw new Error(`Slack API error: ${response.status}`);
        }
      }
    };
  }

  // Email channel
  createEmailChannel(smtpConfig) {
    return {
      send: async (alert) => {
        // Implement email sending logic
        console.log('Sending email alert:', alert);
      }
    };
  }

  // SMS channel
  createSMSChannel(twilioConfig) {
    return {
      send: async (alert) => {
        // Implement SMS sending logic
        console.log('Sending SMS alert:', alert);
      }
    };
  }

  // Get severity color for notifications
  getSeverityColor(severity) {
    switch (severity) {
      case 'critical': return 'danger';
      case 'high': return 'warning';
      case 'medium': return 'good';
      case 'low': return 'good';
      default: return 'good';
    }
  }

  // Get active alerts
  getActiveAlerts() {
    return Array.from(this.alerts.values()).filter(alert => {
      const age = Date.now() - alert.timestamp;
      return age < 24 * 60 * 60 * 1000; // 24 hours
    });
  }

  // Acknowledge alert
  acknowledgeAlert(alertId) {
    const alert = this.alerts.get(alertId);
    if (alert) {
      alert.acknowledged = true;
      alert.acknowledgedAt = Date.now();
    }
  }
}

export const alertManager = new AlertManager();
```

### **Alert Rules Setup**
```javascript
// alertRules.js
import { alertManager } from './services/alertManager';

// Register channels
alertManager.registerChannel('slack', alertManager.createSlackChannel(
  process.env.SLACK_WEBHOOK_URL,
  '#alerts'
));

alertManager.registerChannel('email', alertManager.createEmailChannel({
  host: process.env.SMTP_HOST,
  port: process.env.SMTP_PORT,
  secure: true,
  auth: {
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS
  }
}));

// Create alert rules
alertManager.createRule(
  'HIGH_ERROR_RATE',
  (data) => data.errorRate > 0.05, // 5% error rate
  ['slack', 'email'],
  {
    severity: 'high',
    cooldown: 600000 // 10 minutes
  }
);

alertManager.createRule(
  'SERVER_DOWN',
  (data) => !data.isHealthy,
  ['slack', 'email', 'sms'],
  {
    severity: 'critical',
    cooldown: 300000 // 5 minutes
  }
);

alertManager.createRule(
  'HIGH_CPU_USAGE',
  (data) => data.cpuUsage > 90,
  ['slack'],
  {
    severity: 'medium',
    cooldown: 900000 // 15 minutes
  }
);

alertManager.createRule(
  'LOW_DISK_SPACE',
  (data) => data.diskUsage > 85,
  ['slack', 'email'],
  {
    severity: 'high',
    cooldown: 1800000 // 30 minutes
  }
);

// Function to check and trigger alerts
export const checkAndTriggerAlerts = async (metrics) => {
  // Check error rate
  if (metrics.errorRate !== undefined) {
    await alertManager.triggerAlert('HIGH_ERROR_RATE', {
      errorRate: metrics.errorRate,
      timestamp: Date.now()
    });
  }

  // Check server health
  if (metrics.isHealthy !== undefined) {
    await alertManager.triggerAlert('SERVER_DOWN', {
      isHealthy: metrics.isHealthy,
      server: metrics.server,
      timestamp: Date.now()
    });
  }

  // Check CPU usage
  if (metrics.cpuUsage !== undefined) {
    await alertManager.triggerAlert('HIGH_CPU_USAGE', {
      cpuUsage: metrics.cpuUsage,
      server: metrics.server,
      timestamp: Date.now()
    });
  }

  // Check disk usage
  if (metrics.diskUsage !== undefined) {
    await alertManager.triggerAlert('LOW_DISK_SPACE', {
      diskUsage: metrics.diskUsage,
      server: metrics.server,
      timestamp: Date.now()
    });
  }
};
```

---

## 📊 **Dashboard Creation**

### **Monitoring Dashboard Component**
```javascript
// components/MonitoringDashboard.js
import React, { useState, useEffect } from 'react';
import { View, Text, ScrollView, RefreshControl, StyleSheet } from 'react-native';
import { LineChart, BarChart, PieChart } from 'react-native-chart-kit';
import { Dimensions } from 'react-native';
import { performanceMonitor } from '../services/performanceMonitor';
import { rumService } from '../services/rumService';

const { width } = Dimensions.get('window');

const MonitoringDashboard = () => {
  const [metrics, setMetrics] = useState({});
  const [refreshing, setRefreshing] = useState(false);

  useEffect(() => {
    loadMetrics();
  }, []);

  const loadMetrics = async () => {
    const performanceMetrics = performanceMonitor.getMetricsSummary();
    const sessionInfo = rumService.getCurrentSession();

    setMetrics({
      performance: performanceMetrics,
      session: sessionInfo
    });
  };

  const onRefresh = async () => {
    setRefreshing(true);
    await loadMetrics();
    setRefreshing(false);
  };

  const formatDuration = (ms) => {
    return `${(ms / 1000).toFixed(2)}s`;
  };

  const getChartData = (metricName) => {
    const metricData = metrics.performance?.[metricName] || [];
    return {
      labels: metricData.slice(-7).map((_, i) => `T-${6-i}`),
      datasets: [{
        data: metricData.slice(-7).map(m => m.value)
      }]
    };
  };

  return (
    <ScrollView
      style={styles.container}
      refreshControl={
        <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
      }
    >
      <Text style={styles.title}>Monitoring Dashboard</Text>

      {/* Session Info */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Current Session</Text>
        <View style={styles.sessionInfo}>
          <Text>Session ID: {metrics.session?.sessionId}</Text>
          <Text>Duration: {formatDuration(metrics.session?.duration || 0)}</Text>
          <Text>Current Screen: {metrics.session?.currentScreen || 'N/A'}</Text>
        </View>
      </View>

      {/* Performance Metrics */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Performance Metrics</Text>

        {/* App Start Time */}
        {metrics.performance?.app_start_time && (
          <View style={styles.metric}>
            <Text style={styles.metricTitle}>App Start Time</Text>
            <Text style={styles.metricValue}>
              {formatDuration(metrics.performance.app_start_time[0]?.value || 0)}
            </Text>
          </View>
        )}

        {/* Network Request Time */}
        {metrics.performance?.network_request_time && (
          <View style={styles.chartContainer}>
            <Text style={styles.chartTitle}>Network Request Times</Text>
            <LineChart
              data={getChartData('network_request_time')}
              width={width - 40}
              height={200}
              chartConfig={{
                backgroundColor: '#ffffff',
                backgroundGradientFrom: '#ffffff',
                backgroundGradientTo: '#ffffff',
                decimalPlaces: 2,
                color: (opacity = 1) => `rgba(0, 122, 255, ${opacity})`,
                style: {
                  borderRadius: 16
                }
              }}
              style={styles.chart}
            />
          </View>
        )}

        {/* Error Count */}
        {metrics.performance?.js_error && (
          <View style={styles.metric}>
            <Text style={styles.metricTitle}>JavaScript Errors</Text>
            <Text style={styles.metricValue}>
              {metrics.performance.js_error.length}
            </Text>
          </View>
        )}
      </View>

      {/* Alerts */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Active Alerts</Text>
        <Text style={styles.noAlerts}>No active alerts</Text>
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
  section: {
    backgroundColor: 'white',
    margin: 10,
    borderRadius: 10,
    padding: 15,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  sessionInfo: {
    backgroundColor: '#f8f9fa',
    padding: 10,
    borderRadius: 5,
  },
  metric: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 10,
    backgroundColor: '#f8f9fa',
    marginBottom: 10,
    borderRadius: 5,
  },
  metricTitle: {
    fontSize: 16,
    fontWeight: 'bold',
  },
  metricValue: {
    fontSize: 16,
    color: '#007AFF',
  },
  chartContainer: {
    marginBottom: 20,
  },
  chartTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  chart: {
    borderRadius: 10,
  },
  noAlerts: {
    fontSize: 14,
    color: '#666',
    fontStyle: 'italic',
  },
});

export default MonitoringDashboard;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Application Performance Monitoring**: Track app performance metrics and user experience
- ✅ **Error Tracking & Crash Reporting**: Implement Sentry and Firebase Crashlytics
- ✅ **Real User Monitoring**: Monitor actual user interactions and behavior
- ✅ **Infrastructure Monitoring**: Track server resources and health
- ✅ **Alerting Systems**: Create automated alerts for critical issues
- ✅ **Dashboard Creation**: Build monitoring dashboards with real-time data
- ✅ **Log Management**: Centralize and analyze application logs
- ✅ **Analytics Integration**: Connect with analytics platforms for insights

### **Best Practices**
1. **Implement comprehensive error boundaries** to catch and report errors
2. **Use structured logging** with consistent formats and levels
3. **Set up proper alerting thresholds** to avoid alert fatigue
4. **Monitor both technical and business metrics** for complete visibility
5. **Implement gradual rollouts** with feature flags for safer deployments
6. **Use distributed tracing** to track requests across services
7. **Regularly review and optimize monitoring setup** based on insights
8. **Ensure monitoring doesn't impact app performance**
9. **Implement proper data retention policies** for logs and metrics
10. **Train team on monitoring tools and alert response procedures**

### **Next Steps**
- Learn advanced monitoring patterns like distributed tracing
- Implement automated incident response systems
- Set up monitoring for microservices architectures
- Learn about observability best practices
- Explore AI-powered anomaly detection
- Implement monitoring for mobile-specific metrics

---

## 🎯 **Assignment**

### **Task 1: Complete Monitoring Setup**
Set up comprehensive monitoring for a React Native app:
- Configure Sentry for error tracking and performance monitoring
- Implement Firebase Crashlytics for crash reporting
- Set up real user monitoring with custom events
- Create performance monitoring for key user flows
- Implement error boundaries throughout the app

### **Task 2: Alerting System**
Build an alerting system with multiple channels:
- Create alert rules for different types of issues
- Implement Slack notifications for alerts
- Set up email alerts for critical issues
- Create SMS alerts for production outages
- Build an alert dashboard for monitoring active alerts

### **Task 3: Monitoring Dashboard**
Create a comprehensive monitoring dashboard:
- Display real-time performance metrics
- Show error rates and crash statistics
- Track user engagement and retention metrics
- Monitor server health and resource usage
- Create custom charts and visualizations
- Implement drill-down capabilities for detailed analysis

---

## 📚 **Additional Resources**
- [Sentry Documentation](https://docs.sentry.io/)
- [Firebase Crashlytics](https://firebase.google.com/docs/crashlytics)
- [DataDog Mobile Monitoring](https://docs.datadoghq.com/mobile/)
- [New Relic Mobile Monitoring](https://docs.newrelic.com/docs/mobile-monitoring/)
- [Grafana Dashboard Tutorials](https://grafana.com/tutorials/)

---

**Next Lesson**: [Lesson 57: Rollback Strategies](Lesson%2057_%20Rollback%20Strategies.md)