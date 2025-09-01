# Lesson 29: Performance Monitoring & Optimization

## 🎯 **Learning Objectives**
- Implement comprehensive performance monitoring for React Native and Node.js applications
- Optimize application performance using profiling tools and best practices
- Set up real-time performance tracking and alerting
- Implement caching strategies and database optimization
- Monitor and optimize memory usage and CPU performance

## 📚 **Table of Contents**
1. [Performance Monitoring Setup](#performance-monitoring-setup)
2. [React Native Performance Profiling](#react-native-performance-profiling)
3. [Backend Performance Monitoring](#backend-performance-monitoring)
4. [Database Performance Optimization](#database-performance-optimization)
5. [Caching Strategies](#caching-strategies)
6. [Memory Management](#memory-management)
7. [Network Optimization](#network-optimization)
8. [Real-time Performance Tracking](#real-time-performance-tracking)
9. [Performance Alerting](#performance-alerting)
10. [Practical Examples](#practical-examples)

---

## 📊 **Performance Monitoring Setup**

### **Application Performance Monitoring (APM)**
```javascript
// server.js - APM Setup
const express = require('express');
const newrelic = require('newrelic'); // or datadog, appdynamics, etc.
const app = express();

// New Relic configuration (newrelic.js)
// module.exports = {
//   app_name: ['MyApp'],
//   license_key: process.env.NEW_RELIC_LICENSE_KEY,
//   logging: {
//     level: 'info'
//   },
//   allow_all_headers: true,
//   attributes: {
//     enabled: true,
//     include: ['request.parameters.*']
//   }
// };

// Custom performance monitoring middleware
const performanceMiddleware = (req, res, next) => {
  const start = process.hrtime.bigint();
  const startMemory = process.memoryUsage();

  res.on('finish', () => {
    const end = process.hrtime.bigint();
    const endMemory = process.memoryUsage();

    const duration = Number(end - start) / 1000000; // Convert to milliseconds
    const memoryDelta = endMemory.heapUsed - startMemory.heapUsed;

    // Log performance metrics
    console.log(`[${new Date().toISOString()}] ${req.method} ${req.originalUrl}`);
    console.log(`  Duration: ${duration.toFixed(2)}ms`);
    console.log(`  Memory Delta: ${(memoryDelta / 1024 / 1024).toFixed(2)}MB`);
    console.log(`  Status: ${res.statusCode}`);

    // Send metrics to monitoring service
    if (global.monitoringService) {
      global.monitoringService.recordMetric('http_request_duration', duration, {
        method: req.method,
        route: req.originalUrl,
        status: res.statusCode,
      });

      global.monitoringService.recordMetric('memory_usage', endMemory.heapUsed, {
        type: 'heap_used',
      });
    }
  });

  next();
};

app.use(performanceMiddleware);
```

### **Custom Monitoring Service**
```javascript
class MonitoringService {
  constructor() {
    this.metrics = new Map();
    this.alerts = [];
    this.thresholds = {
      responseTime: 1000, // 1 second
      memoryUsage: 500 * 1024 * 1024, // 500MB
      errorRate: 0.05, // 5%
    };
  }

  // Record metric
  recordMetric(name, value, tags = {}) {
    const key = `${name}:${JSON.stringify(tags)}`;
    const timestamp = Date.now();

    if (!this.metrics.has(key)) {
      this.metrics.set(key, []);
    }

    const metricData = this.metrics.get(key);
    metricData.push({ value, timestamp });

    // Keep only last 1000 data points
    if (metricData.length > 1000) {
      metricData.shift();
    }

    // Check thresholds
    this.checkThresholds(name, value, tags);
  }

  // Check performance thresholds
  checkThresholds(metricName, value, tags) {
    let alertTriggered = false;

    switch (metricName) {
      case 'http_request_duration':
        if (value > this.thresholds.responseTime) {
          alertTriggered = true;
          this.createAlert('High Response Time', `Response time ${value}ms exceeds threshold`, {
            metric: metricName,
            value,
            threshold: this.thresholds.responseTime,
            ...tags,
          });
        }
        break;

      case 'memory_usage':
        if (tags.type === 'heap_used' && value > this.thresholds.memoryUsage) {
          alertTriggered = true;
          this.createAlert('High Memory Usage', `Memory usage ${(value / 1024 / 1024).toFixed(2)}MB exceeds threshold`, {
            metric: metricName,
            value,
            threshold: this.thresholds.memoryUsage,
            ...tags,
          });
        }
        break;
    }

    return alertTriggered;
  }

  // Create alert
  createAlert(title, message, data) {
    const alert = {
      id: Date.now().toString(),
      title,
      message,
      data,
      timestamp: new Date().toISOString(),
      acknowledged: false,
    };

    this.alerts.unshift(alert);

    // Keep only last 100 alerts
    if (this.alerts.length > 100) {
      this.alerts = this.alerts.slice(0, 100);
    }

    // Send alert notification (email, Slack, etc.)
    this.sendAlertNotification(alert);

    console.error(`🚨 ALERT: ${title} - ${message}`);
  }

  // Send alert notification
  async sendAlertNotification(alert) {
    // Implement notification logic (email, Slack, etc.)
    try {
      // Example: Send to Slack
      if (process.env.SLACK_WEBHOOK_URL) {
        await fetch(process.env.SLACK_WEBHOOK_URL, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            text: `🚨 *${alert.title}*\n${alert.message}`,
            attachments: [{
              fields: Object.entries(alert.data).map(([key, value]) => ({
                title: key,
                value: String(value),
                short: true,
              })),
            }],
          }),
        });
      }
    } catch (error) {
      console.error('Failed to send alert notification:', error);
    }
  }

  // Get metrics summary
  getMetricsSummary() {
    const summary = {
      totalMetrics: this.metrics.size,
      activeAlerts: this.alerts.filter(a => !a.acknowledged).length,
      recentAlerts: this.alerts.slice(0, 5),
      systemInfo: {
        uptime: process.uptime(),
        memory: process.memoryUsage(),
        cpu: process.cpuUsage(),
      },
    };

    return summary;
  }

  // Get metric data
  getMetricData(name, tags = {}, timeRange = 3600000) { // 1 hour default
    const key = `${name}:${JSON.stringify(tags)}`;
    const metricData = this.metrics.get(key) || [];

    const cutoff = Date.now() - timeRange;
    const filteredData = metricData.filter(point => point.timestamp > cutoff);

    return {
      name,
      tags,
      data: filteredData,
      count: filteredData.length,
      average: filteredData.length > 0
        ? filteredData.reduce((sum, point) => sum + point.value, 0) / filteredData.length
        : 0,
      min: filteredData.length > 0 ? Math.min(...filteredData.map(p => p.value)) : 0,
      max: filteredData.length > 0 ? Math.max(...filteredData.map(p => p.value)) : 0,
    };
  }

  // Acknowledge alert
  acknowledgeAlert(alertId) {
    const alert = this.alerts.find(a => a.id === alertId);
    if (alert) {
      alert.acknowledged = true;
      alert.acknowledgedAt = new Date().toISOString();
    }
  }
}

// Global monitoring instance
global.monitoringService = new MonitoringService();

// Health check endpoint with metrics
app.get('/health/metrics', (req, res) => {
  const summary = global.monitoringService.getMetricsSummary();
  res.json(summary);
});

// Metrics endpoint
app.get('/metrics/:name', (req, res) => {
  const { name } = req.params;
  const tags = req.query;
  const timeRange = parseInt(req.query.timeRange) || 3600000;

  const metricData = global.monitoringService.getMetricData(name, tags, timeRange);
  res.json(metricData);
});

module.exports = MonitoringService;
```

---

## 📱 **React Native Performance Profiling**

### **Flipper Integration**
```javascript
// metro.config.js
const { getDefaultConfig } = require('metro-config');

module.exports = (async () => {
  const {
    resolver: { sourceExts, assetExts },
  } = await getDefaultConfig();

  return {
    transformer: {
      getTransformOptions: async () => ({
        transform: {
          experimentalImportSupport: false,
          inlineRequires: true,
        },
      }),
    },
    resolver: {
      sourceExts,
      assetExts,
    },
  };
})();
```

### **Performance Monitoring Hook**
```javascript
import { useEffect, useRef } from 'react';
import { PerformanceMonitor } from 'react-native/Libraries/Performance/Systrace';

const usePerformanceMonitor = (componentName) => {
  const startTime = useRef(null);
  const renderCount = useRef(0);

  useEffect(() => {
    // Start monitoring
    PerformanceMonitor.start();

    return () => {
      // Stop monitoring
      PerformanceMonitor.stop();
    };
  }, []);

  useEffect(() => {
    // Track renders
    renderCount.current += 1;

    if (__DEV__) {
      console.log(`${componentName} rendered ${renderCount.current} times`);
    }
  });

  const startTiming = (operation) => {
    startTime.current = performance.now();
    if (__DEV__) {
      console.log(`Starting ${operation} in ${componentName}`);
    }
  };

  const endTiming = (operation) => {
    if (startTime.current) {
      const duration = performance.now() - startTime.current;
      if (__DEV__) {
        console.log(`${operation} took ${duration.toFixed(2)}ms in ${componentName}`);
      }

      // Send to monitoring service
      if (global.monitoringService) {
        global.monitoringService.recordMetric('component_operation_duration', duration, {
          component: componentName,
          operation,
        });
      }

      startTime.current = null;
    }
  };

  return { startTiming, endTiming, renderCount: renderCount.current };
};
```

### **Memory Leak Detection**
```javascript
import { useEffect, useRef } from 'react';

const useMemoryMonitor = () => {
  const intervalRef = useRef(null);
  const memoryHistory = useRef([]);

  useEffect(() => {
    // Start memory monitoring
    intervalRef.current = setInterval(() => {
      // Note: This is a simplified example
      // In production, use more sophisticated memory monitoring
      const memoryInfo = {
        timestamp: Date.now(),
        used: performance.memory?.usedJSHeapSize || 0,
        total: performance.memory?.totalJSHeapSize || 0,
        limit: performance.memory?.jsHeapSizeLimit || 0,
      };

      memoryHistory.current.push(memoryInfo);

      // Keep only last 100 readings
      if (memoryHistory.current.length > 100) {
        memoryHistory.current.shift();
      }

      // Check for memory leaks
      if (memoryHistory.current.length >= 10) {
        const recent = memoryHistory.current.slice(-10);
        const avgUsed = recent.reduce((sum, m) => sum + m.used, 0) / recent.length;
        const trend = recent[recent.length - 1].used - recent[0].used;

        if (trend > 10 * 1024 * 1024) { // 10MB increase
          console.warn('Potential memory leak detected');
          if (global.monitoringService) {
            global.monitoringService.recordMetric('memory_leak_detected', trend, {
              type: 'memory_trend',
            });
          }
        }
      }

      if (__DEV__) {
        console.log('Memory usage:', {
          used: (memoryInfo.used / 1024 / 1024).toFixed(2) + 'MB',
          total: (memoryInfo.total / 1024 / 1024).toFixed(2) + 'MB',
          limit: (memoryInfo.limit / 1024 / 1024).toFixed(2) + 'MB',
        });
      }
    }, 5000); // Check every 5 seconds

    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);

  const getMemoryStats = () => {
    return {
      current: memoryHistory.current[memoryHistory.current.length - 1],
      history: memoryHistory.current,
      average: memoryHistory.current.length > 0
        ? memoryHistory.current.reduce((sum, m) => sum + m.used, 0) / memoryHistory.current.length
        : 0,
    };
  };

  return { getMemoryStats };
};
```

---

## 🖥️ **Backend Performance Monitoring**

### **Response Time Monitoring**
```javascript
const responseTime = require('response-time');

app.use(responseTime((req, res, time) => {
  // Log response time
  console.log(`${req.method} ${req.originalUrl} - ${time.toFixed(2)}ms`);

  // Send to monitoring service
  if (global.monitoringService) {
    global.monitoringService.recordMetric('http_response_time', time, {
      method: req.method,
      route: req.originalUrl,
      status: res.statusCode,
    });
  }
}));
```

### **Database Query Performance**
```javascript
const mongoose = require('mongoose');

// Enable mongoose debugging
if (process.env.NODE_ENV === 'development') {
  mongoose.set('debug', (collectionName, method, query, doc) => {
    console.log(`${collectionName}.${method}`, JSON.stringify(query, null, 2));
  });
}

// Query performance middleware
const queryPerformanceMiddleware = (schema) => {
  schema.pre('find', function() {
    this._startTime = Date.now();
  });

  schema.post('find', function() {
    if (this._startTime) {
      const duration = Date.now() - this._startTime;
      console.log(`Query took ${duration}ms`);

      if (global.monitoringService) {
        global.monitoringService.recordMetric('database_query_duration', duration, {
          collection: this.model.collection.name,
          operation: 'find',
        });
      }
    }
  });

  // Add to other operations
  ['findOne', 'findOneAndUpdate', 'findOneAndDelete', 'updateMany', 'deleteMany'].forEach(method => {
    schema.pre(method, function() {
      this._startTime = Date.now();
    });

    schema.post(method, function() {
      if (this._startTime) {
        const duration = Date.now() - this._startTime;
        if (global.monitoringService) {
          global.monitoringService.recordMetric('database_query_duration', duration, {
            collection: this.model.collection.name,
            operation: method,
          });
        }
      }
    });
  });
};

// Apply to all schemas
mongoose.plugin(queryPerformanceMiddleware);
```

### **CPU and Memory Monitoring**
```javascript
const os = require('os');
const v8 = require('v8');

class SystemMonitor {
  constructor() {
    this.metrics = {
      cpu: [],
      memory: [],
      heap: [],
    };
    this.interval = null;
  }

  start(interval = 30000) { // 30 seconds
    this.interval = setInterval(() => {
      this.collectMetrics();
    }, interval);
  }

  stop() {
    if (this.interval) {
      clearInterval(this.interval);
      this.interval = null;
    }
  }

  collectMetrics() {
    const timestamp = Date.now();

    // CPU usage
    const cpuUsage = process.cpuUsage();
    const cpuPercent = (cpuUsage.user + cpuUsage.system) / 1000000; // Convert to seconds

    this.metrics.cpu.push({
      timestamp,
      usage: cpuPercent,
      cores: os.cpus().length,
    });

    // Memory usage
    const memUsage = process.memoryUsage();
    this.metrics.memory.push({
      timestamp,
      rss: memUsage.rss,
      heapTotal: memUsage.heapTotal,
      heapUsed: memUsage.heapUsed,
      external: memUsage.external,
    });

    // V8 heap statistics
    const heapStats = v8.getHeapStatistics();
    this.metrics.heap.push({
      timestamp,
      totalHeapSize: heapStats.total_heap_size,
      usedHeapSize: heapStats.used_heap_size,
      heapSizeLimit: heapStats.heap_size_limit,
      numberOfNativeContexts: heapStats.number_of_native_contexts,
      numberOfDetachedContexts: heapStats.number_of_detached_contexts,
    });

    // Keep only last 100 readings
    Object.keys(this.metrics).forEach(key => {
      if (this.metrics[key].length > 100) {
        this.metrics[key] = this.metrics[key].slice(-100);
      }
    });

    // Send to monitoring service
    if (global.monitoringService) {
      global.monitoringService.recordMetric('cpu_usage', cpuPercent, {
        type: 'process_cpu',
      });

      global.monitoringService.recordMetric('memory_usage', memUsage.heapUsed, {
        type: 'heap_used',
      });

      global.monitoringService.recordMetric('memory_usage', memUsage.rss, {
        type: 'rss',
      });
    }

    // Log in development
    if (process.env.NODE_ENV === 'development') {
      console.log('System Metrics:', {
        cpu: `${cpuPercent.toFixed(2)}%`,
        memory: `${(memUsage.heapUsed / 1024 / 1024).toFixed(2)}MB heap used`,
        rss: `${(memUsage.rss / 1024 / 1024).toFixed(2)}MB RSS`,
      });
    }
  }

  getMetrics() {
    return {
      cpu: this.calculateStats(this.metrics.cpu, 'usage'),
      memory: {
        rss: this.calculateStats(this.metrics.memory, 'rss'),
        heapTotal: this.calculateStats(this.metrics.memory, 'heapTotal'),
        heapUsed: this.calculateStats(this.metrics.memory, 'heapUsed'),
        external: this.calculateStats(this.metrics.memory, 'external'),
      },
      heap: {
        totalHeapSize: this.calculateStats(this.metrics.heap, 'totalHeapSize'),
        usedHeapSize: this.calculateStats(this.metrics.heap, 'usedHeapSize'),
        heapSizeLimit: this.calculateStats(this.metrics.heap, 'heapSizeLimit'),
      },
    };
  }

  calculateStats(data, field) {
    if (data.length === 0) return null;

    const values = data.map(item => item[field]);
    const sum = values.reduce((a, b) => a + b, 0);

    return {
      current: values[values.length - 1],
      average: sum / values.length,
      min: Math.min(...values),
      max: Math.max(...values),
      count: values.length,
    };
  }
}

// Global system monitor
const systemMonitor = new SystemMonitor();

// Start monitoring
if (process.env.NODE_ENV === 'production') {
  systemMonitor.start();
}

// Graceful shutdown
process.on('SIGINT', () => {
  systemMonitor.stop();
  process.exit(0);
});

process.on('SIGTERM', () => {
  systemMonitor.stop();
  process.exit(0);
});

module.exports = systemMonitor;
```

---

## 🗄️ **Database Performance Optimization**

### **Query Optimization**
```javascript
const mongoose = require('mongoose');

// User model with indexes
const UserSchema = new mongoose.Schema({
  email: { type: String, unique: true, index: true },
  firstName: { type: String, index: true },
  lastName: { type: String, index: true },
  role: { type: String, index: true },
  createdAt: { type: Date, index: true },
  lastLogin: { type: Date, index: true },
  isActive: { type: Boolean, index: true },
});

// Compound indexes for common queries
UserSchema.index({ role: 1, isActive: 1 });
UserSchema.index({ createdAt: -1, lastLogin: -1 });
UserSchema.index({ firstName: 1, lastName: 1 });

// Post model with optimized indexes
const PostSchema = new mongoose.Schema({
  title: { type: String, index: 'text' }, // Text index for search
  content: String,
  author: { type: mongoose.Schema.Types.ObjectId, ref: 'User', index: true },
  category: { type: mongoose.Schema.Types.ObjectId, ref: 'Category', index: true },
  tags: [{ type: String, index: true }],
  status: { type: String, enum: ['draft', 'published', 'archived'], index: true },
  publishedAt: { type: Date, index: true },
  createdAt: { type: Date, index: true },
  updatedAt: { type: Date, index: true },
  viewCount: { type: Number, default: 0 },
  likeCount: { type: Number, default: 0 },
});

// Compound indexes
PostSchema.index({ status: 1, publishedAt: -1 });
PostSchema.index({ author: 1, createdAt: -1 });
PostSchema.index({ category: 1, publishedAt: -1 });
PostSchema.index({ tags: 1, publishedAt: -1 });

// Optimized query methods
UserSchema.statics.findActiveUsers = function(limit = 50) {
  return this.find({ isActive: true })
    .select('firstName lastName email role lastLogin')
    .sort({ lastLogin: -1 })
    .limit(limit)
    .lean(); // Use lean() for better performance
};

PostSchema.statics.findPublishedPosts = function(page = 1, limit = 10) {
  return this.find({ status: 'published' })
    .populate('author', 'firstName lastName')
    .populate('category', 'name')
    .sort({ publishedAt: -1 })
    .skip((page - 1) * limit)
    .limit(limit)
    .lean();
};

PostSchema.statics.searchPosts = function(query, page = 1, limit = 10) {
  return this.find(
    { $text: { $search: query } },
    { score: { $meta: 'textScore' } }
  )
    .populate('author', 'firstName lastName')
    .sort({ score: { $meta: 'textScore' } })
    .skip((page - 1) * limit)
    .limit(limit);
};

// Aggregation pipeline for analytics
PostSchema.statics.getPostAnalytics = function() {
  return this.aggregate([
    {
      $match: { status: 'published' }
    },
    {
      $group: {
        _id: {
          year: { $year: '$publishedAt' },
          month: { $month: '$publishedAt' },
        },
        count: { $sum: 1 },
        totalViews: { $sum: '$viewCount' },
        totalLikes: { $sum: '$likeCount' },
        avgViews: { $avg: '$viewCount' },
        avgLikes: { $avg: '$likeCount' },
      }
    },
    {
      $sort: { '_id.year': -1, '_id.month': -1 }
    }
  ]);
};
```

### **Connection Pooling**
```javascript
const mongoose = require('mongoose');

// Optimized connection options
const connectionOptions = {
  maxPoolSize: 10, // Maintain up to 10 socket connections
  serverSelectionTimeoutMS: 5000, // Keep trying to send operations for 5 seconds
  socketTimeoutMS: 45000, // Close sockets after 45 seconds of inactivity
  bufferCommands: false, // Disable mongoose buffering
  bufferMaxEntries: 0, // Disable mongoose buffering
  maxIdleTimeMS: 30000, // Close connections after 30 seconds of inactivity
};

// Connect with optimized settings
mongoose.connect(process.env.MONGODB_URI, connectionOptions)
  .then(() => {
    console.log('Connected to MongoDB with optimized settings');
  })
  .catch((error) => {
    console.error('MongoDB connection error:', error);
  });

// Connection event handlers
mongoose.connection.on('connected', () => {
  console.log('Mongoose connected to MongoDB');
});

mongoose.connection.on('error', (error) => {
  console.error('Mongoose connection error:', error);
});

mongoose.connection.on('disconnected', () => {
  console.log('Mongoose disconnected');
});

// Graceful shutdown
process.on('SIGINT', async () => {
  await mongoose.connection.close();
  console.log('MongoDB connection closed through app termination');
  process.exit(0);
});
```

---

## 💾 **Caching Strategies**

### **Multi-Level Caching**
```javascript
const redis = require('redis');
const NodeCache = require('node-cache');

// In-memory cache (L1)
const memoryCache = new NodeCache({
  stdTTL: 300, // 5 minutes
  checkperiod: 60, // Check for expired keys every 60 seconds
  maxKeys: 1000, // Maximum number of keys
});

// Redis cache (L2)
const redisClient = redis.createClient({
  host: process.env.REDIS_HOST || 'localhost',
  port: process.env.REDIS_PORT || 6379,
  password: process.env.REDIS_PASSWORD,
});

redisClient.on('error', (error) => {
  console.error('Redis error:', error);
});

redisClient.on('connect', () => {
  console.log('Connected to Redis');
});

// Promisify Redis methods
const { promisify } = require('util');
const redisGet = promisify(redisClient.get).bind(redisClient);
const redisSet = promisify(redisClient.set).bind(redisClient);
const redisDel = promisify(redisClient.del).bind(redisClient);
const redisExpire = promisify(redisClient.expire).bind(redisClient);

class CacheManager {
  constructor() {
    this.defaultTTL = 300; // 5 minutes
  }

  // Multi-level cache get
  async get(key) {
    // Try L1 cache first
    let data = memoryCache.get(key);
    if (data !== undefined) {
      console.log(`Cache hit (L1): ${key}`);
      return data;
    }

    // Try L2 cache
    try {
      const redisData = await redisGet(key);
      if (redisData) {
        console.log(`Cache hit (L2): ${key}`);
        data = JSON.parse(redisData);

        // Populate L1 cache
        memoryCache.set(key, data, this.defaultTTL);
        return data;
      }
    } catch (error) {
      console.error('Redis get error:', error);
    }

    console.log(`Cache miss: ${key}`);
    return null;
  }

  // Multi-level cache set
  async set(key, data, ttl = this.defaultTTL) {
    // Set in L1 cache
    memoryCache.set(key, data, ttl);

    // Set in L2 cache
    try {
      await redisSet(key, JSON.stringify(data));
      await redisExpire(key, ttl);
    } catch (error) {
      console.error('Redis set error:', error);
    }
  }

  // Delete from both caches
  async del(key) {
    // Delete from L1
    memoryCache.del(key);

    // Delete from L2
    try {
      await redisDel(key);
    } catch (error) {
      console.error('Redis del error:', error);
    }
  }

  // Clear all caches
  async clear() {
    // Clear L1
    memoryCache.flushAll();

    // Clear L2 (be careful with this in production)
    try {
      await promisify(redisClient.flushall).bind(redisClient)();
    } catch (error) {
      console.error('Redis clear error:', error);
    }
  }

  // Cache middleware for Express
  middleware(ttl = this.defaultTTL) {
    return async (req, res, next) => {
      // Only cache GET requests
      if (req.method !== 'GET') {
        return next();
      }

      const key = `http:${req.originalUrl}`;

      // Try to get from cache
      const cachedResponse = await this.get(key);
      if (cachedResponse) {
        return res.json(cachedResponse);
      }

      // Store original json method
      const originalJson = res.json;

      // Override json method to cache response
      res.json = function(data) {
        // Cache the response
        this.cacheManager.set(key, data, ttl);

        // Call original method
        originalJson.call(this, data);
      }.bind({ cacheManager: this });

      next();
    };
  }

  // Cache statistics
  getStats() {
    const memoryStats = memoryCache.getStats();
    const redisStats = {
      connected: redisClient.connected,
      // Add more Redis stats as needed
    };

    return {
      memory: memoryStats,
      redis: redisStats,
    };
  }
}

// Global cache manager
const cacheManager = new CacheManager();

// Export for use in other modules
module.exports = cacheManager;

// Cache invalidation helpers
const invalidateUserCache = async (userId) => {
  const keys = [
    `user:${userId}`,
    `user:${userId}:posts`,
    `user:${userId}:profile`,
  ];

  for (const key of keys) {
    await cacheManager.del(key);
  }
};

const invalidatePostCache = async (postId) => {
  const keys = [
    `post:${postId}`,
    `posts:recent`,
    `posts:popular`,
  ];

  for (const key of keys) {
    await cacheManager.del(key);
  }
};

module.exports = {
  cacheManager,
  invalidateUserCache,
  invalidatePostCache,
};
```

---

## 🧠 **Memory Management**

### **Memory Leak Detection**
```javascript
const memwatch = require('memwatch-next');

class MemoryLeakDetector {
  constructor() {
    this.leaks = [];
    this.gcStats = [];

    // Start memory monitoring
    this.startMonitoring();
  }

  startMonitoring() {
    // Monitor garbage collection
    memwatch.on('stats', (stats) => {
      this.gcStats.push({
        timestamp: new Date(),
        ...stats,
      });

      // Keep only last 100 GC stats
      if (this.gcStats.length > 100) {
        this.gcStats.shift();
      }

      console.log('GC Stats:', {
        num_full_gc: stats.num_full_gc,
        num_inc_gc: stats.num_inc_gc,
        heap_compactions: stats.heap_compactions,
        usage_trend: stats.usage_trend,
      });
    });

    // Detect memory leaks
    memwatch.on('leak', (info) => {
      const leak = {
        timestamp: new Date(),
        growth: info.growth,
        reason: info.reason,
        ...info,
      };

      this.leaks.push(leak);

      console.error('🚨 MEMORY LEAK DETECTED:', leak);

      // Send alert
      if (global.monitoringService) {
        global.monitoringService.recordMetric('memory_leak', info.growth, {
          reason: info.reason,
          type: 'leak_detected',
        });
      }
    });
  }

  getMemoryStats() {
    const memUsage = process.memoryUsage();

    return {
      rss: memUsage.rss,
      heapTotal: memUsage.heapTotal,
      heapUsed: memUsage.heapUsed,
      external: memUsage.external,
      arrayBuffers: memUsage.arrayBuffers,
      leaks: this.leaks.slice(-10), // Last 10 leaks
      gcStats: this.gcStats.slice(-10), // Last 10 GC stats
    };
  }

  // Force garbage collection (use with caution)
  forceGC() {
    if (global.gc) {
      global.gc();
      console.log('Forced garbage collection');
    } else {
      console.warn('Garbage collection not available. Run with --expose-gc flag');
    }
  }
}

// Memory optimization helpers
class MemoryOptimizer {
  // Clear unused references
  static clearReferences(obj) {
    if (obj && typeof obj === 'object') {
      Object.keys(obj).forEach(key => {
        if (obj[key] && typeof obj[key] === 'object') {
          this.clearReferences(obj[key]);
        }
        obj[key] = null;
      });
    }
  }

  // Optimize array operations
  static optimizeArray(array) {
    // Pre-allocate array size if known
    if (array.length > 1000) {
      array.length = 0; // Clear array efficiently
    }
  }

  // Memory-efficient string operations
  static processLargeString(str) {
    // Process string in chunks to avoid memory spikes
    const chunkSize = 1024 * 1024; // 1MB chunks
    const chunks = [];

    for (let i = 0; i < str.length; i += chunkSize) {
      chunks.push(str.slice(i, i + chunkSize));
    }

    return chunks;
  }
}

// Memory monitoring middleware
const memoryMonitoringMiddleware = (req, res, next) => {
  const startMemory = process.memoryUsage();

  res.on('finish', () => {
    const endMemory = process.memoryUsage();
    const memoryDelta = endMemory.heapUsed - startMemory.heapUsed;

    // Log significant memory changes
    if (Math.abs(memoryDelta) > 10 * 1024 * 1024) { // 10MB
      console.log(`Memory change: ${(memoryDelta / 1024 / 1024).toFixed(2)}MB for ${req.method} ${req.originalUrl}`);
    }
  });

  next();
};

module.exports = {
  MemoryLeakDetector,
  MemoryOptimizer,
  memoryMonitoringMiddleware,
};
```

---

## 🌐 **Network Optimization**

### **HTTP/2 and Connection Pooling**
```javascript
const http2 = require('http2');
const https = require('https');

// HTTP/2 client for API calls
class Http2Client {
  constructor(baseUrl, options = {}) {
    this.baseUrl = baseUrl;
    this.client = http2.connect(baseUrl, {
      rejectUnauthorized: process.env.NODE_ENV === 'production',
      ...options,
    });

    this.client.on('error', (error) => {
      console.error('HTTP/2 client error:', error);
    });

    this.client.on('close', () => {
      console.log('HTTP/2 client closed');
    });
  }

  async request(method, path, data = null, headers = {}) {
    return new Promise((resolve, reject) => {
      const req = this.client.request({
        ':method': method,
        ':path': path,
        ...headers,
      });

      let responseData = '';

      req.on('response', (headers) => {
        const status = headers[':status'];
        if (status >= 400) {
          reject(new Error(`HTTP ${status}`));
        }
      });

      req.on('data', (chunk) => {
        responseData += chunk;
      });

      req.on('end', () => {
        try {
          const parsed = JSON.parse(responseData);
          resolve(parsed);
        } catch (error) {
          resolve(responseData);
        }
      });

      req.on('error', reject);

      // Send data if provided
      if (data) {
        if (typeof data === 'object') {
          req.write(JSON.stringify(data));
        } else {
          req.write(data);
        }
      }

      req.end();
    });
  }

  async get(path, headers = {}) {
    return this.request('GET', path, null, headers);
  }

  async post(path, data, headers = {}) {
    return this.request('POST', path, data, {
      'content-type': 'application/json',
      ...headers,
    });
  }

  close() {
    this.client.close();
  }
}

// Connection pooling for regular HTTP
const httpAgent = new https.Agent({
  keepAlive: true,
  maxSockets: 100,
  maxFreeSockets: 10,
  timeout: 60000,
  keepAliveMsecs: 30000,
});

// Optimized fetch with connection pooling
const optimizedFetch = async (url, options = {}) => {
  const startTime = Date.now();

  try {
    const response = await fetch(url, {
      agent: httpAgent,
      timeout: 30000,
      ...options,
    });

    const duration = Date.now() - startTime;

    // Log slow requests
    if (duration > 5000) {
      console.warn(`Slow request: ${url} took ${duration}ms`);
    }

    return response;
  } catch (error) {
    const duration = Date.now() - startTime;
    console.error(`Request failed: ${url} after ${duration}ms`, error);
    throw error;
  }
};
```

### **Response Compression and Caching**
```javascript
const compression = require('compression');
const express = require('express');

const app = express();

// Compression middleware
app.use(compression({
  level: 6, // Compression level (1-9)
  threshold: 1024, // Only compress responses larger than 1KB
  filter: (req, res) => {
    // Don't compress if client doesn't support it
    if (req.headers['x-no-compression']) {
      return false;
    }

    // Use default filter
    return compression.filter(req, res);
  },
}));

// Cache headers middleware
const cacheControl = (maxAge = 300) => {
  return (req, res, next) => {
    // Set cache headers
    res.set({
      'Cache-Control': `public, max-age=${maxAge}`,
      'Expires': new Date(Date.now() + maxAge * 1000).toUTCString(),
    });

    // Add ETag for conditional requests
    const etag = require('crypto').createHash('md5')
      .update(req.originalUrl + JSON.stringify(req.query))
      .digest('hex');

    res.set('ETag', etag);

    next();
  };
};

// Apply cache control to static files
app.use('/static', cacheControl(86400)); // 24 hours for static files
app.use('/api/public', cacheControl(3600)); // 1 hour for public API data

// Conditional requests middleware
const conditionalRequests = (req, res, next) => {
  const ifNoneMatch = req.headers['if-none-match'];
  const etag = res.get('ETag');

  if (ifNoneMatch && etag && ifNoneMatch === etag) {
    res.status(304).end(); // Not Modified
    return;
  }

  next();
};

// Apply to API routes
app.use('/api', conditionalRequests);
```

---

## 📈 **Real-time Performance Tracking**

### **Performance Dashboard**
```javascript
const express = require('express');
const router = express.Router();

// Performance metrics endpoint
router.get('/metrics', (req, res) => {
  const metrics = {
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    cpu: process.cpuUsage(),
    system: {
      platform: process.platform,
      arch: process.arch,
      nodeVersion: process.version,
      totalMemory: require('os').totalmem(),
      freeMemory: require('os').freemem(),
    },
  };

  // Add custom metrics if monitoring service exists
  if (global.monitoringService) {
    metrics.custom = global.monitoringService.getMetricsSummary();
  }

  res.json(metrics);
});

// Real-time metrics WebSocket
const setupMetricsWebSocket = (io) => {
  const metricsNamespace = io.of('/metrics');

  metricsNamespace.on('connection', (socket) => {
    console.log('Metrics client connected');

    // Send initial metrics
    const sendMetrics = () => {
      const metrics = {
        timestamp: new Date().toISOString(),
        memory: process.memoryUsage(),
        uptime: process.uptime(),
      };

      socket.emit('metrics', metrics);
    };

    // Send metrics every 5 seconds
    const interval = setInterval(sendMetrics, 5000);

    socket.on('disconnect', () => {
      clearInterval(interval);
      console.log('Metrics client disconnected');
    });
  });
};

// Performance profiling endpoint
router.post('/profile/start', (req, res) => {
  console.log('Starting performance profiling...');

  // Start CPU profiling
  require('v8-profiler-next').startProfiling('cpu-profile', true);

  // Start memory profiling
  if (global.gc) {
    global.gc();
  }

  res.json({ status: 'profiling_started' });
});

router.post('/profile/stop', (req, res) => {
  console.log('Stopping performance profiling...');

  // Stop CPU profiling
  const cpuProfile = require('v8-profiler-next').stopProfiling('cpu-profile');

  // Generate heap snapshot
  const heapSnapshot = require('v8-profiler-next').takeSnapshot();

  res.json({
    status: 'profiling_stopped',
    cpuProfile: cpuProfile.export(),
    heapSnapshot: heapSnapshot.export(),
  });
});

module.exports = {
  router,
  setupMetricsWebSocket,
};
```

---

## 🚨 **Performance Alerting**

### **Automated Alerting System**
```javascript
class AlertManager {
  constructor() {
    this.alerts = new Map();
    this.thresholds = {
      responseTime: 2000, // 2 seconds
      memoryUsage: 500 * 1024 * 1024, // 500MB
      errorRate: 0.05, // 5%
      cpuUsage: 80, // 80%
    };

    this.cooldowns = new Map(); // Prevent alert spam
  }

  // Check metric against threshold
  checkThreshold(metricName, value, metadata = {}) {
    const threshold = this.thresholds[metricName];
    if (!threshold) return false;

    let alertTriggered = false;

    switch (metricName) {
      case 'responseTime':
        alertTriggered = value > threshold;
        break;
      case 'memoryUsage':
        alertTriggered = value > threshold;
        break;
      case 'errorRate':
        alertTriggered = value > threshold;
        break;
      case 'cpuUsage':
        alertTriggered = value > threshold;
        break;
    }

    if (alertTriggered) {
      this.triggerAlert(metricName, value, threshold, metadata);
    }

    return alertTriggered;
  }

  // Trigger alert with cooldown
  triggerAlert(metricName, value, threshold, metadata) {
    const alertKey = `${metricName}:${JSON.stringify(metadata)}`;
    const lastAlert = this.cooldowns.get(alertKey);

    // Cooldown period: 5 minutes
    if (lastAlert && Date.now() - lastAlert < 5 * 60 * 1000) {
      return; // Skip alert due to cooldown
    }

    const alert = {
      id: Date.now().toString(),
      metric: metricName,
      value,
      threshold,
      metadata,
      timestamp: new Date().toISOString(),
      severity: this.getSeverity(metricName, value, threshold),
    };

    this.alerts.set(alert.id, alert);
    this.cooldowns.set(alertKey, Date.now());

    // Send alert notifications
    this.sendAlertNotifications(alert);

    console.error(`🚨 ALERT: ${metricName} exceeded threshold`, {
      value,
      threshold,
      severity: alert.severity,
    });
  }

  // Determine alert severity
  getSeverity(metricName, value, threshold) {
    const ratio = value / threshold;

    if (ratio >= 2) return 'critical';
    if (ratio >= 1.5) return 'high';
    if (ratio >= 1.2) return 'medium';
    return 'low';
  }

  // Send alert notifications
  async sendAlertNotifications(alert) {
    const notificationPromises = [
      this.sendSlackNotification(alert),
      this.sendEmailNotification(alert),
      this.sendWebhookNotification(alert),
    ];

    try {
      await Promise.allSettled(notificationPromises);
    } catch (error) {
      console.error('Failed to send some alert notifications:', error);
    }
  }

  // Slack notification
  async sendSlackNotification(alert) {
    if (!process.env.SLACK_WEBHOOK_URL) return;

    const color = {
      critical: '#dc3545',
      high: '#fd7e14',
      medium: '#ffc107',
      low: '#28a745',
    }[alert.severity];

    const payload = {
      attachments: [{
        color,
        title: `🚨 Performance Alert: ${alert.metric}`,
        text: `Value: ${alert.value}\nThreshold: ${alert.threshold}\nSeverity: ${alert.severity}`,
        fields: Object.entries(alert.metadata).map(([key, value]) => ({
          title: key,
          value: String(value),
          short: true,
        })),
        ts: Math.floor(new Date(alert.timestamp).getTime() / 1000),
      }],
    };

    await fetch(process.env.SLACK_WEBHOOK_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });
  }

  // Email notification
  async sendEmailNotification(alert) {
    if (!process.env.SMTP_HOST) return;

    const nodemailer = require('nodemailer');
    const transporter = nodemailer.createTransporter({
      host: process.env.SMTP_HOST,
      port: process.env.SMTP_PORT,
      secure: false,
      auth: {
        user: process.env.SMTP_USER,
        pass: process.env.SMTP_PASS,
      },
    });

    const mailOptions = {
      from: process.env.ALERT_EMAIL_FROM,
      to: process.env.ALERT_EMAIL_TO,
      subject: `🚨 Performance Alert: ${alert.metric}`,
      html: `
        <h2>Performance Alert</h2>
        <p><strong>Metric:</strong> ${alert.metric}</p>
        <p><strong>Value:</strong> ${alert.value}</p>
        <p><strong>Threshold:</strong> ${alert.threshold}</p>
        <p><strong>Severity:</strong> ${alert.severity}</p>
        <p><strong>Timestamp:</strong> ${alert.timestamp}</p>
        <pre>${JSON.stringify(alert.metadata, null, 2)}</pre>
      `,
    };

    await transporter.sendMail(mailOptions);
  }

  // Webhook notification
  async sendWebhookNotification(alert) {
    if (!process.env.ALERT_WEBHOOK_URL) return;

    await fetch(process.env.ALERT_WEBHOOK_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        event: 'performance_alert',
        alert,
      }),
    });
  }

  // Get active alerts
  getActiveAlerts() {
    return Array.from(this.alerts.values())
      .filter(alert => !alert.acknowledged)
      .sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));
  }

  // Acknowledge alert
  acknowledgeAlert(alertId) {
    const alert = this.alerts.get(alertId);
    if (alert) {
      alert.acknowledged = true;
      alert.acknowledgedAt = new Date().toISOString();
    }
  }

  // Clean old alerts (keep last 1000)
  cleanupOldAlerts() {
    const alertsArray = Array.from(this.alerts.values());
    if (alertsArray.length > 1000) {
      alertsArray.sort((a, b) => new Date(a.timestamp) - new Date(b.timestamp));
      const toDelete = alertsArray.slice(0, alertsArray.length - 1000);
      toDelete.forEach(alert => this.alerts.delete(alert.id));
    }
  }
}

// Global alert manager
const alertManager = new AlertManager();

// Periodic cleanup
setInterval(() => {
  alertManager.cleanupOldAlerts();
}, 60 * 60 * 1000); // Every hour

module.exports = alertManager;

---

## 🛠️ **Practical Examples**

### **Complete Performance Monitoring Setup**
```javascript
// server.js - Complete setup with all monitoring features
const express = require('express');
const mongoose = require('mongoose');
const monitoringService = require('./services/monitoring');
const alertManager = require('./services/alerts');
const cacheManager = require('./services/cache');
const systemMonitor = require('./services/systemMonitor');

const app = express();

// Apply performance monitoring middleware
app.use(monitoringService.performanceMiddleware);

// Apply caching middleware
app.use(cacheManager.middleware());

// Body parsing
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// Routes
app.use('/api/auth', require('./routes/auth'));
app.use('/api/users', require('./routes/users'));
app.use('/api/posts', require('./routes/posts'));

// Health check with detailed metrics
app.get('/health', (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    database: mongoose.connection.readyState === 1 ? 'connected' : 'disconnected',
    cache: cacheManager.getStats(),
    alerts: alertManager.getActiveAlerts().length,
  };

  // Check if any critical alerts exist
  const criticalAlerts = alertManager.getActiveAlerts()
    .filter(alert => alert.severity === 'critical');

  if (criticalAlerts.length > 0) {
    health.status = 'degraded';
    health.criticalAlerts = criticalAlerts.length;
  }

  const statusCode = health.status === 'healthy' ? 200 : 503;
  res.status(statusCode).json(health);
});

// Metrics dashboard endpoint
app.get('/metrics/dashboard', (req, res) => {
  const dashboard = {
    system: systemMonitor.getMetrics(),
    monitoring: monitoringService.getMetricsSummary(),
    cache: cacheManager.getStats(),
    alerts: alertManager.getActiveAlerts(),
    timestamp: new Date().toISOString(),
  };

  res.json(dashboard);
});

// Acknowledge alert endpoint
app.post('/alerts/:alertId/acknowledge', (req, res) => {
  const { alertId } = req.params;
  alertManager.acknowledgeAlert(alertId);
  res.json({ success: true, message: 'Alert acknowledged' });
});

// Start system monitoring
if (process.env.NODE_ENV === 'production') {
  systemMonitor.start();
}

// Graceful shutdown
process.on('SIGINT', async () => {
  console.log('Shutting down gracefully...');

  systemMonitor.stop();
  await mongoose.connection.close();

  console.log('Shutdown complete');
  process.exit(0);
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🚀 Server running on port ${PORT}`);
  console.log(`📊 Performance monitoring: http://localhost:${PORT}/metrics/dashboard`);
  console.log(`❤️  Health check: http://localhost:${PORT}/health`);
});

module.exports = app;
```

### **React Native Performance Optimization**
```javascript
// App.js - Complete performance-optimized React Native app
import React, { useEffect } from 'react';
import { StatusBar, StyleSheet, View } from 'react-native';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { Provider } from 'react-redux';
import { PersistGate } from 'redux-persist/integration/react';
import { store, persistor } from './src/redux/store';
import { monitoringService } from './src/services/monitoring';

// Screens
import HomeScreen from './src/screens/HomeScreen';
import ProfileScreen from './src/screens/ProfileScreen';
import SettingsScreen from './src/screens/SettingsScreen';

const Stack = createStackNavigator();

const App = () => {
  useEffect(() => {
    // Initialize performance monitoring
    monitoringService.initialize();

    // Enable layout animation for smooth transitions
    const { LayoutAnimation } = require('react-native');
    LayoutAnimation.configureNext(LayoutAnimation.Presets.easeInEaseOut);

    return () => {
      monitoringService.cleanup();
    };
  }, []);

  return (
    <Provider store={store}>
      <PersistGate loading={null} persistor={persistor}>
        <NavigationContainer>
          <View style={styles.container}>
            <StatusBar barStyle="light-content" backgroundColor="#007AFF" />

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
                // Optimize navigation performance
                animationEnabled: true,
                gestureEnabled: true,
                cardStyle: { backgroundColor: '#f5f5f5' },
              }}
            >
              <Stack.Screen
                name="Home"
                component={HomeScreen}
                options={{
                  title: 'Home',
                  // Optimize header rendering
                  headerTransparent: false,
                }}
              />
              <Stack.Screen
                name="Profile"
                component={ProfileScreen}
                options={{
                  title: 'Profile',
                }}
              />
              <Stack.Screen
                name="Settings"
                component={SettingsScreen}
                options={{
                  title: 'Settings',
                }}
              />
            </Stack.Navigator>
          </View>
        </NavigationContainer>
      </PersistGate>
    </Provider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    // Optimize rendering
    backgroundColor: '#f5f5f5',
  },
});

export default App;
```

### **Database Optimization Script**
```javascript
// scripts/optimize-database.js
const mongoose = require('mongoose');
require('dotenv').config();

async function optimizeDatabase() {
  try {
    // Connect to MongoDB
    await mongoose.connect(process.env.MONGODB_URI);

    console.log('🔧 Starting database optimization...\n');

    // Get database and collections
    const db = mongoose.connection.db;
    const collections = await db.listCollections().toArray();

    for (const collection of collections) {
      const collectionName = collection.name;
      console.log(`Optimizing collection: ${collectionName}`);

      // Analyze collection statistics
      const stats = await db.collection(collectionName).stats();
      console.log(`  Documents: ${stats.count}`);
      console.log(`  Size: ${(stats.size / 1024 / 1024).toFixed(2)} MB`);
      console.log(`  Indexes: ${stats.nindexes}`);

      // Check for missing indexes based on collection name
      if (collectionName === 'users') {
        await optimizeUsersCollection(db);
      } else if (collectionName === 'posts') {
        await optimizePostsCollection(db);
      } else if (collectionName === 'comments') {
        await optimizeCommentsCollection(db);
      }

      // Analyze slow queries (if available)
      try {
        const slowQueries = await db.collection('system.profile')
          .find({ millis: { $gt: 100 } }) // Queries taking more than 100ms
          .sort({ ts: -1 })
          .limit(5)
          .toArray();

        if (slowQueries.length > 0) {
          console.log(`  ⚠️  Found ${slowQueries.length} slow queries`);
          slowQueries.forEach((query, index) => {
            console.log(`    ${index + 1}. ${query.millis}ms - ${JSON.stringify(query.command)}`);
          });
        }
      } catch (error) {
        // Profiling might not be enabled
      }

      console.log('');
    }

    // Database-wide optimizations
    console.log('📊 Database-wide optimizations:');

    // Check database statistics
    const dbStats = await db.stats();
    console.log(`Total size: ${(dbStats.dataSize / 1024 / 1024).toFixed(2)} MB`);
    console.log(`Storage size: ${(dbStats.storageSize / 1024 / 1024).toFixed(2)} MB`);
    console.log(`Index size: ${(dbStats.indexSize / 1024 / 1024).toFixed(2)} MB`);

    // Suggest optimizations
    if (dbStats.dataSize > dbStats.storageSize * 1.5) {
      console.log('💡 Consider running: db.repairDatabase()');
    }

    console.log('\n✅ Database optimization complete!');

  } catch (error) {
    console.error('❌ Database optimization failed:', error);
  } finally {
    await mongoose.disconnect();
  }
}

async function optimizeUsersCollection(db) {
  const collection = db.collection('users');

  // Check existing indexes
  const indexes = await collection.listIndexes().toArray();
  const indexNames = indexes.map(idx => idx.name);

  // Create missing indexes
  if (!indexNames.includes('email_1')) {
    await collection.createIndex({ email: 1 }, { unique: true });
    console.log('  ✓ Created email index');
  }

  if (!indexNames.includes('role_1')) {
    await collection.createIndex({ role: 1 });
    console.log('  ✓ Created role index');
  }

  if (!indexNames.includes('createdAt_1')) {
    await collection.createIndex({ createdAt: 1 });
    console.log('  ✓ Created createdAt index');
  }

  if (!indexNames.includes('lastLogin_1')) {
    await collection.createIndex({ lastLogin: 1 });
    console.log('  ✓ Created lastLogin index');
  }
}

async function optimizePostsCollection(db) {
  const collection = db.collection('posts');

  const indexes = await collection.listIndexes().toArray();
  const indexNames = indexes.map(idx => idx.name);

  if (!indexNames.includes('author_1')) {
    await collection.createIndex({ author: 1 });
    console.log('  ✓ Created author index');
  }

  if (!indexNames.includes('status_1')) {
    await collection.createIndex({ status: 1 });
    console.log('  ✓ Created status index');
  }

  if (!indexNames.includes('publishedAt_1')) {
    await collection.createIndex({ publishedAt: 1 });
    console.log('  ✓ Created publishedAt index');
  }

  if (!indexNames.includes('category_1')) {
    await collection.createIndex({ category: 1 });
    console.log('  ✓ Created category index');
  }

  // Compound indexes
  if (!indexNames.includes('status_1_publishedAt_-1')) {
    await collection.createIndex({ status: 1, publishedAt: -1 });
    console.log('  ✓ Created status + publishedAt compound index');
  }

  if (!indexNames.includes('author_1_createdAt_-1')) {
    await collection.createIndex({ author: 1, createdAt: -1 });
    console.log('  ✓ Created author + createdAt compound index');
  }
}

async function optimizeCommentsCollection(db) {
  const collection = db.collection('comments');

  const indexes = await collection.listIndexes().toArray();
  const indexNames = indexes.map(idx => idx.name);

  if (!indexNames.includes('post_1')) {
    await collection.createIndex({ post: 1 });
    console.log('  ✓ Created post index');
  }

  if (!indexNames.includes('author_1')) {
    await collection.createIndex({ author: 1 });
    console.log('  ✓ Created author index');
  }

  if (!indexNames.includes('createdAt_1')) {
    await collection.createIndex({ createdAt: 1 });
    console.log('  ✓ Created createdAt index');
  }
}

// Run optimization if called directly
if (require.main === module) {
  optimizeDatabase();
}

module.exports = { optimizeDatabase };
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Performance Monitoring Setup**: APM tools, custom monitoring services, and alerting
- ✅ **React Native Profiling**: Flipper integration, performance hooks, memory monitoring
- ✅ **Backend Monitoring**: Response time tracking, database query monitoring, system metrics
- ✅ **Database Optimization**: Query optimization, indexing strategies, connection pooling
- ✅ **Caching Strategies**: Multi-level caching, Redis integration, cache invalidation
- ✅ **Memory Management**: Leak detection, memory optimization, garbage collection
- ✅ **Network Optimization**: HTTP/2, connection pooling, response compression
- ✅ **Real-time Tracking**: Performance dashboards, WebSocket metrics, profiling
- ✅ **Alerting System**: Automated alerts, multi-channel notifications, alert management
- ✅ **Practical Implementation**: Complete monitoring setup, optimization scripts

### **Best Practices**
1. **Monitor Everything**: Track response times, memory usage, CPU, and error rates
2. **Set Appropriate Thresholds**: Configure alerts for critical performance issues
3. **Use Multi-level Caching**: Implement L1 (memory) and L2 (Redis) caching
4. **Optimize Database Queries**: Use proper indexing and avoid N+1 query problems
5. **Implement Connection Pooling**: Reuse database and HTTP connections
6. **Monitor Memory Usage**: Detect and fix memory leaks early
7. **Use Compression**: Compress responses to reduce bandwidth
8. **Profile Regularly**: Use profiling tools to identify bottlenecks
9. **Set Up Automated Alerts**: Get notified of performance issues immediately
10. **Test Performance**: Include performance tests in CI/CD pipeline

### **Next Steps**
- Implement distributed tracing across microservices
- Set up log aggregation and analysis
- Create performance budgets for frontend assets
- Implement automated scaling based on metrics
- Learn about chaos engineering and resilience testing
- Explore machine learning for anomaly detection

---

## 🎯 **Assignment**

### **Task 1: Performance Monitoring Dashboard**
Create a comprehensive performance monitoring dashboard that includes:
- Real-time system metrics (CPU, memory, disk usage)
- Application performance metrics (response times, throughput)
- Database performance metrics (query times, connection pool status)
- Cache hit rates and performance
- Error rates and alerting system
- Historical performance trends and charts

### **Task 2: Database Optimization**
Optimize a MongoDB database for a blog application by:
- Analyzing slow queries and creating appropriate indexes
- Implementing query optimization techniques
- Setting up connection pooling and monitoring
- Creating aggregation pipelines for analytics
- Implementing database sharding if needed
- Setting up automated index optimization

### **Task 3: Caching Implementation**
Implement a multi-level caching system that includes:
- In-memory caching for frequently accessed data
- Redis caching for session data and API responses
- CDN caching for static assets
- Database query result caching
- Cache invalidation strategies
- Cache performance monitoring and metrics

### **Task 4: Performance Testing Suite**
Create a comprehensive performance testing suite including:
- Load testing with Artillery for API endpoints
- Stress testing for database operations
- Memory leak detection and monitoring
- Network performance testing
- Mobile app performance profiling
- Automated performance regression testing

---

## 📚 **Additional Resources**
- [New Relic Documentation](https://docs.newrelic.com/)
- [DataDog Documentation](https://docs.datadoghq.com/)
- [Flipper Documentation](https://fbflipper.com/)
- [Redis Documentation](https://redis.io/documentation)
- [MongoDB Performance Best Practices](https://docs.mongodb.com/manual/core/query-optimization/)
- [Node.js Performance Guide](https://nodejs.org/en/docs/guides/anatomy-of-an-async-function/)

---

**Next Lesson**: [Lesson 30: Security Best Practices & Implementation](Lesson%2030_%20Security%20Best%20Practices%20&%20Implementation.md)
