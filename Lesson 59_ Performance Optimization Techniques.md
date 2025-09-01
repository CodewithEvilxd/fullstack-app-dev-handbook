# Lesson 59: Performance Optimization Techniques

## 🎯 **Learning Objectives**
- Master React Native performance optimization techniques
- Implement efficient rendering and state management
- Optimize bundle size and loading times
- Handle memory leaks and performance bottlenecks
- Use profiling tools to identify and fix performance issues

## 📚 **Table of Contents**
1. [Performance Fundamentals](#performance-fundamentals)
2. [Rendering Optimization](#rendering-optimization)
3. [Memory Management](#memory-management)
4. [Bundle Optimization](#bundle-optimization)
5. [Image Optimization](#image-optimization)
6. [Network Optimization](#network-optimization)
7. [Database Optimization](#database-optimization)
8. [Profiling and Monitoring](#profiling-and-monitoring)
9. [Performance Testing](#performance-testing)
10. [Practical Examples](#practical-examples)

---

## 📊 **Performance Fundamentals**

### **Performance Metrics**
```javascript
// utils/performanceMetrics.js
class PerformanceMetrics {
  constructor() {
    this.metrics = new Map();
    this.isEnabled = __DEV__ ? false : true; // Disable in development
  }

  // Measure function execution time
  measureExecutionTime(label, fn) {
    if (!this.isEnabled) return fn();

    const startTime = performance.now();
    const result = fn();
    const endTime = performance.now();

    const executionTime = endTime - startTime;
    this.recordMetric(`${label}_execution_time`, executionTime);

    console.log(`${label} executed in ${executionTime.toFixed(2)}ms`);
    return result;
  }

  // Measure async function execution time
  async measureAsyncExecutionTime(label, asyncFn) {
    if (!this.isEnabled) return await asyncFn();

    const startTime = performance.now();
    const result = await asyncFn();
    const endTime = performance.now();

    const executionTime = endTime - startTime;
    this.recordMetric(`${label}_execution_time`, executionTime);

    console.log(`${label} executed in ${executionTime.toFixed(2)}ms`);
    return result;
  }

  // Record custom metric
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
      // Send to your monitoring service
      console.log('Performance metric:', metric);
    } catch (error) {
      console.error('Failed to send performance metric:', error);
    }
  }

  // Get performance summary
  getPerformanceSummary() {
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
}

export const performanceMetrics = new PerformanceMetrics();
```

### **Performance Budget**
```javascript
// config/performanceBudget.js
export const PERFORMANCE_BUDGET = {
  // Bundle size limits
  bundleSize: {
    android: 15 * 1024 * 1024, // 15MB
    ios: 50 * 1024 * 1024,     // 50MB (iOS has higher limits)
    web: 2 * 1024 * 1024,      // 2MB for web
  },

  // Loading time limits
  loadingTime: {
    initialLoad: 3000,    // 3 seconds
    navigation: 1000,     // 1 second
    apiResponse: 2000,    // 2 seconds
  },

  // Memory usage limits
  memoryUsage: {
    heapSize: 100 * 1024 * 1024, // 100MB
    imageCache: 50 * 1024 * 1024, // 50MB
  },

  // Frame rate limits
  frameRate: {
    target: 60,     // 60 FPS
    minimum: 30,    // 30 FPS minimum
  },

  // Bundle analysis
  bundleAnalysis: {
    maxChunks: 20,
    maxAssetSize: 2 * 1024 * 1024, // 2MB per asset
  }
};

// Performance budget checker
export class PerformanceBudgetChecker {
  static checkBundleSize(platform, actualSize) {
    const limit = PERFORMANCE_BUDGET.bundleSize[platform];
    const isWithinBudget = actualSize <= limit;

    if (!isWithinBudget) {
      console.warn(`Bundle size exceeds budget: ${actualSize} > ${limit}`);
    }

    return {
      isWithinBudget,
      actualSize,
      budget: limit,
      difference: actualSize - limit
    };
  }

  static checkLoadingTime(type, actualTime) {
    const limit = PERFORMANCE_BUDGET.loadingTime[type];
    const isWithinBudget = actualTime <= limit;

    if (!isWithinBudget) {
      console.warn(`${type} loading time exceeds budget: ${actualTime}ms > ${limit}ms`);
    }

    return {
      isWithinBudget,
      actualTime,
      budget: limit,
      difference: actualTime - limit
    };
  }

  static checkMemoryUsage(type, actualUsage) {
    const limit = PERFORMANCE_BUDGET.memoryUsage[type];
    const isWithinBudget = actualUsage <= limit;

    if (!isWithinBudget) {
      console.warn(`${type} memory usage exceeds budget: ${actualUsage} > ${limit}`);
    }

    return {
      isWithinBudget,
      actualUsage,
      budget: limit,
      difference: actualUsage - limit
    };
  }
}
```

---

## 🎨 **Rendering Optimization**

### **React.memo for Component Memoization**
```javascript
// components/OptimizedListItem.js
import React, { memo, useMemo } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

// Memoized list item component
const OptimizedListItem = memo(({
  item,
  onPress,
  isSelected,
  style
}) => {
  // Memoize expensive calculations
  const itemStyle = useMemo(() => ({
    backgroundColor: isSelected ? '#e3f2fd' : 'white',
    ...style
  }), [isSelected, style]);

  const displayText = useMemo(() => {
    // Expensive text processing
    return item.title.toUpperCase();
  }, [item.title]);

  return (
    <TouchableOpacity
      style={[styles.container, itemStyle]}
      onPress={() => onPress(item)}
      activeOpacity={0.7}
    >
      <View style={styles.content}>
        <Text style={styles.title} numberOfLines={1}>
          {displayText}
        </Text>
        {item.subtitle && (
          <Text style={styles.subtitle} numberOfLines={2}>
            {item.subtitle}
          </Text>
        )}
      </View>
      {isSelected && <View style={styles.selectedIndicator} />}
    </TouchableOpacity>
  );
});

// Custom comparison function for memo
OptimizedListItem.displayName = 'OptimizedListItem';

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 16,
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: '#e0e0e0',
  },
  content: {
    flex: 1,
  },
  title: {
    fontSize: 16,
    fontWeight: '600',
    color: '#333',
  },
  subtitle: {
    fontSize: 14,
    color: '#666',
    marginTop: 4,
  },
  selectedIndicator: {
    width: 4,
    height: 40,
    backgroundColor: '#2196f3',
    borderRadius: 2,
  },
});

export default OptimizedListItem;
```

### **useMemo and useCallback Hooks**
```javascript
// hooks/useOptimizedCallbacks.js
import { useCallback, useMemo, useRef } from 'react';

// Optimized callback hook
export const useOptimizedCallback = (callback, deps) => {
  const callbackRef = useRef(callback);

  // Update ref when callback changes
  callbackRef.current = callback;

  return useCallback((...args) => {
    return callbackRef.current(...args);
  }, deps);
};

// Optimized expensive calculation hook
export const useOptimizedCalculation = (data, calculationFn) => {
  return useMemo(() => {
    const startTime = performance.now();
    const result = calculationFn(data);
    const endTime = performance.now();

    console.log(`Calculation took ${(endTime - startTime).toFixed(2)}ms`);

    return result;
  }, [data, calculationFn]);
};

// Optimized event handlers
export const useOptimizedEventHandlers = (handlers) => {
  return useMemo(() => {
    const optimizedHandlers = {};

    Object.keys(handlers).forEach(key => {
      optimizedHandlers[key] = useCallback(
        (...args) => handlers[key](...args),
        [handlers[key]]
      );
    });

    return optimizedHandlers;
  }, [handlers]);
};
```

### **Virtualized Lists**
```javascript
// components/OptimizedFlatList.js
import React, { useMemo, useCallback } from 'react';
import { FlatList, View, Text, StyleSheet } from 'react-native';

const OptimizedFlatList = ({
  data,
  renderItem,
  keyExtractor,
  onEndReached,
  ...props
}) => {
  // Memoize key extractor
  const memoizedKeyExtractor = useCallback(
    (item, index) => keyExtractor ? keyExtractor(item, index) : item.id || index.toString(),
    [keyExtractor]
  );

  // Memoize render item
  const memoizedRenderItem = useCallback(
    ({ item, index }) => renderItem({ item, index }),
    [renderItem]
  );

  // Memoize on end reached
  const memoizedOnEndReached = useCallback(
    (info) => {
      if (onEndReached) {
        onEndReached(info);
      }
    },
    [onEndReached]
  );

  // Optimize data with stable references
  const optimizedData = useMemo(() => {
    return data.map((item, index) => ({
      ...item,
      _optimizedKey: item.id || `item_${index}`
    }));
  }, [data]);

  return (
    <FlatList
      data={optimizedData}
      renderItem={memoizedRenderItem}
      keyExtractor={memoizedKeyExtractor}
      onEndReached={memoizedOnEndReached}
      onEndReachedThreshold={0.1}
      maxToRenderPerBatch={10}
      windowSize={10}
      initialNumToRender={10}
      removeClippedSubviews={true}
      getItemLayout={(data, index) => ({
        length: 60, // Item height
        offset: 60 * index,
        index
      })}
      {...props}
    />
  );
};

export default OptimizedFlatList;
```

---

## 🧠 **Memory Management**

### **Memory Leak Prevention**
```javascript
// hooks/useMemorySafeEffect.js
import { useEffect, useRef } from 'react';

// Memory-safe useEffect hook
export const useMemorySafeEffect = (effect, deps) => {
  const isMountedRef = useRef(true);
  const cleanupRef = useRef(null);

  useEffect(() => {
    isMountedRef.current = true;

    const cleanup = effect();

    if (cleanup) {
      cleanupRef.current = cleanup;
    }

    return () => {
      isMountedRef.current = false;

      if (cleanupRef.current) {
        cleanupRef.current();
        cleanupRef.current = null;
      }
    };
  }, deps);

  // Return mounted status
  return isMountedRef.current;
};

// Memory-safe async operations
export const useMemorySafeAsync = () => {
  const isMountedRef = useRef(true);

  useEffect(() => {
    isMountedRef.current = true;
    return () => {
      isMountedRef.current = false;
    };
  }, []);

  const executeAsync = useCallback(async (asyncFn) => {
    try {
      const result = await asyncFn();

      // Only update state if component is still mounted
      if (isMountedRef.current) {
        return result;
      }
    } catch (error) {
      if (isMountedRef.current) {
        throw error;
      }
    }
  }, []);

  return executeAsync;
};
```

### **Image Memory Management**
```javascript
// utils/imageMemoryManager.js
class ImageMemoryManager {
  constructor() {
    this.imageCache = new Map();
    this.maxCacheSize = 50 * 1024 * 1024; // 50MB
    this.currentCacheSize = 0;
  }

  // Cache image with memory management
  cacheImage(uri, imageData) {
    const imageSize = this.estimateImageSize(imageData);

    // Check if adding this image would exceed cache limit
    if (this.currentCacheSize + imageSize > this.maxCacheSize) {
      this.evictOldImages(imageSize);
    }

    // Add to cache
    this.imageCache.set(uri, {
      data: imageData,
      size: imageSize,
      lastAccessed: Date.now(),
      accessCount: 0
    });

    this.currentCacheSize += imageSize;
  }

  // Get cached image
  getCachedImage(uri) {
    const cached = this.imageCache.get(uri);

    if (cached) {
      cached.lastAccessed = Date.now();
      cached.accessCount++;
      return cached.data;
    }

    return null;
  }

  // Estimate image size
  estimateImageSize(imageData) {
    // Rough estimation based on base64 string length
    if (typeof imageData === 'string' && imageData.startsWith('data:image')) {
      return Math.round(imageData.length * 0.75); // Base64 is ~33% larger
    }

    // For other formats, use a default size
    return 1024 * 1024; // 1MB default
  }

  // Evict old images to free memory
  evictOldImages(requiredSpace) {
    // Sort by last accessed time and access count
    const sortedImages = Array.from(this.imageCache.entries())
      .sort(([, a], [, b]) => {
        // Prefer keeping frequently accessed images
        const scoreA = a.lastAccessed + (a.accessCount * 1000);
        const scoreB = b.lastAccessed + (b.accessCount * 1000);
        return scoreA - scoreB;
      });

    let freedSpace = 0;

    for (const [uri, imageData] of sortedImages) {
      if (freedSpace >= requiredSpace) break;

      this.imageCache.delete(uri);
      this.currentCacheSize -= imageData.size;
      freedSpace += imageData.size;

      console.log(`Evicted image from cache: ${uri}`);
    }
  }

  // Clear entire cache
  clearCache() {
    this.imageCache.clear();
    this.currentCacheSize = 0;
    console.log('Image cache cleared');
  }

  // Get cache statistics
  getCacheStats() {
    return {
      totalImages: this.imageCache.size,
      totalSize: this.currentCacheSize,
      maxSize: this.maxCacheSize,
      utilization: (this.currentCacheSize / this.maxCacheSize) * 100
    };
  }

  // Auto-cleanup old images
  startAutoCleanup(interval = 5 * 60 * 1000) { // 5 minutes
    this.cleanupInterval = setInterval(() => {
      const cutoffTime = Date.now() - (30 * 60 * 1000); // 30 minutes

      for (const [uri, imageData] of this.imageCache.entries()) {
        if (imageData.lastAccessed < cutoffTime) {
          this.imageCache.delete(uri);
          this.currentCacheSize -= imageData.size;
        }
      }

      console.log(`Auto-cleanup completed. Cache size: ${this.currentCacheSize} bytes`);
    }, interval);
  }

  // Stop auto-cleanup
  stopAutoCleanup() {
    if (this.cleanupInterval) {
      clearInterval(this.cleanupInterval);
      this.cleanupInterval = null;
    }
  }
}

export const imageMemoryManager = new ImageMemoryManager();
```

---

## 📦 **Bundle Optimization**

### **Code Splitting and Dynamic Imports**
```javascript
// utils/lazyImports.js
import { lazy } from 'react';

// Lazy load components
export const LazyHomeScreen = lazy(() =>
  import('../screens/HomeScreen').then(module => ({ default: module.HomeScreen }))
);

export const LazyProfileScreen = lazy(() =>
  import('../screens/ProfileScreen').then(module => ({ default: module.ProfileScreen }))
);

export const LazySettingsScreen = lazy(() =>
  import('../screens/SettingsScreen').then(module => ({ default: module.SettingsScreen }))
);

// Lazy load utilities
export const loadHeavyUtility = () =>
  import('../utils/heavyUtility').then(module => module.default);

// Lazy load third-party libraries
export const loadChartLibrary = () =>
  import('react-native-chart-kit').then(module => module.default);

// Prefetch critical resources
export const prefetchCriticalResources = () => {
  // Prefetch critical screens
  import('../screens/HomeScreen');
  import('../screens/ProfileScreen');

  // Prefetch critical utilities
  import('../utils/auth');
  import('../utils/api');
};

// Conditional loading based on platform
export const loadPlatformSpecificModule = () => {
  if (Platform.OS === 'ios') {
    return import('../modules/iOSModule');
  } else {
    return import('../modules/AndroidModule');
  }
};
```

### **Tree Shaking and Dead Code Elimination**
```javascript
// utils/treeShaking.js
// Named exports for better tree shaking
export const utilityFunction1 = () => {
  // Implementation
};

export const utilityFunction2 = () => {
  // Implementation
};

export const utilityFunction3 = () => {
  // Implementation
};

// Avoid default exports when possible
// export default { utilityFunction1, utilityFunction2, utilityFunction3 }; // Bad for tree shaking

// Instead, use named exports
export { utilityFunction1, utilityFunction2, utilityFunction3 };

// Side effect free modules
export const createPureFunction = () => {
  return (input) => {
    // Pure function with no side effects
    return input * 2;
  };
};

// Conditional exports based on environment
export const debugLogger = __DEV__ ? console.log : () => {};

// Export only what's needed
export { utilityFunction1, createPureFunction };
```

### **Bundle Analysis**
```javascript
// scripts/analyzeBundle.js
const fs = require('fs');
const path = require('path');
const { execSync } = require('child_process');

class BundleAnalyzer {
  constructor() {
    this.bundlePath = './android/app/build/generated/assets/react/release/index.android.bundle';
    this.outputPath = './bundle-analysis';
  }

  // Analyze bundle size
  analyzeBundleSize() {
    try {
      const stats = fs.statSync(this.bundlePath);
      const sizeInMB = (stats.size / (1024 * 1024)).toFixed(2);

      console.log(`Bundle size: ${sizeInMB} MB`);

      return {
        size: stats.size,
        sizeInMB: parseFloat(sizeInMB),
        path: this.bundlePath
      };
    } catch (error) {
      console.error('Error analyzing bundle size:', error);
      return null;
    }
  }

  // Analyze bundle composition
  analyzeBundleComposition() {
    try {
      // Read bundle content
      const bundleContent = fs.readFileSync(this.bundlePath, 'utf8');

      // Analyze imports
      const importMatches = bundleContent.match(/import\s+.*?\s+from\s+['"]([^'"]+)['"]/g) || [];
      const requireMatches = bundleContent.match(/require\s*\(\s*['"]([^'"]+)['"]\s*\)/g) || [];

      const allImports = [...importMatches, ...requireMatches];

      // Count module usage
      const moduleCount = {};
      allImports.forEach(imp => {
        const moduleName = this.extractModuleName(imp);
        moduleCount[moduleName] = (moduleCount[moduleName] || 0) + 1;
      });

      return {
        totalImports: allImports.length,
        uniqueModules: Object.keys(moduleCount).length,
        moduleUsage: moduleCount,
        bundleLength: bundleContent.length
      };
    } catch (error) {
      console.error('Error analyzing bundle composition:', error);
      return null;
    }
  }

  // Extract module name from import statement
  extractModuleName(importStatement) {
    const match = importStatement.match(/['"]([^'"]+)['"]/);
    return match ? match[1] : 'unknown';
  }

  // Generate bundle analysis report
  generateReport() {
    const sizeAnalysis = this.analyzeBundleSize();
    const compositionAnalysis = this.analyzeBundleComposition();

    const report = {
      timestamp: new Date().toISOString(),
      bundleSize: sizeAnalysis,
      composition: compositionAnalysis,
      recommendations: this.generateRecommendations(sizeAnalysis, compositionAnalysis)
    };

    // Ensure output directory exists
    if (!fs.existsSync(this.outputPath)) {
      fs.mkdirSync(this.outputPath, { recursive: true });
    }

    // Write report
    const reportPath = path.join(this.outputPath, 'bundle-report.json');
    fs.writeFileSync(reportPath, JSON.stringify(report, null, 2));

    console.log(`Bundle analysis report generated: ${reportPath}`);

    return report;
  }

  // Generate optimization recommendations
  generateRecommendations(sizeAnalysis, compositionAnalysis) {
    const recommendations = [];

    if (sizeAnalysis && sizeAnalysis.sizeInMB > 10) {
      recommendations.push('Bundle size is large. Consider code splitting and lazy loading.');
    }

    if (compositionAnalysis) {
      const unusedModules = Object.entries(compositionAnalysis.moduleUsage)
        .filter(([, count]) => count === 1)
        .map(([module]) => module);

      if (unusedModules.length > 0) {
        recommendations.push(`Potentially unused modules: ${unusedModules.slice(0, 5).join(', ')}`);
      }
    }

    return recommendations;
  }

  // Compare bundle sizes
  compareBundles(oldBundlePath) {
    const currentSize = this.analyzeBundleSize();
    const oldSize = this.getBundleSize(oldBundlePath);

    if (!currentSize || !oldSize) {
      return null;
    }

    const difference = currentSize.size - oldSize.size;
    const percentChange = ((difference / oldSize.size) * 100).toFixed(2);

    return {
      currentSize: currentSize.sizeInMB,
      oldSize: oldSize.sizeInMB,
      difference: (difference / (1024 * 1024)).toFixed(2),
      percentChange: parseFloat(percentChange)
    };
  }

  // Get bundle size from file
  getBundleSize(bundlePath) {
    try {
      const stats = fs.statSync(bundlePath);
      return {
        size: stats.size,
        sizeInMB: (stats.size / (1024 * 1024)).toFixed(2)
      };
    } catch (error) {
      return null;
    }
  }
}

// CLI usage
if (require.main === module) {
  const analyzer = new BundleAnalyzer();

  const command = process.argv[2];

  switch (command) {
    case 'size':
      console.log(analyzer.analyzeBundleSize());
      break;

    case 'composition':
      console.log(analyzer.analyzeBundleComposition());
      break;

    case 'report':
      analyzer.generateReport();
      break;

    case 'compare':
      if (process.argv[3]) {
        console.log(analyzer.compareBundles(process.argv[3]));
      } else {
        console.log('Usage: node analyzeBundle.js compare <old-bundle-path>');
      }
      break;

    default:
      console.log('Usage: node analyzeBundle.js {size|composition|report|compare}');
  }
}

module.exports = BundleAnalyzer;
```

---

## 🖼️ **Image Optimization**

### **Image Optimization Pipeline**
```javascript
// utils/imageOptimizer.js
import ImageResizer from 'react-native-image-resizer';

class ImageOptimizer {
  constructor() {
    this.quality = 0.8;
    this.maxWidth = 1920;
    this.maxHeight = 1080;
    this.format = 'JPEG';
  }

  // Optimize single image
  async optimizeImage(imageUri, options = {}) {
    try {
      const {
        quality = this.quality,
        maxWidth = this.maxWidth,
        maxHeight = this.maxHeight,
        format = this.format
      } = options;

      console.log(`Optimizing image: ${imageUri}`);

      const optimizedImage = await ImageResizer.createResizedImage(
        imageUri,
        maxWidth,
        maxHeight,
        format,
        quality,
        0, // rotation
        undefined, // output path
        false, // keep metadata
        { mode: 'contain' } // resize mode
      );

      console.log(`Image optimized: ${optimizedImage.size} bytes`);

      return {
        uri: optimizedImage.uri,
        path: optimizedImage.path,
        size: optimizedImage.size,
        width: maxWidth,
        height: maxHeight,
        format,
        quality
      };
    } catch (error) {
      console.error('Image optimization failed:', error);
      throw error;
    }
  }

  // Batch optimize images
  async optimizeImages(imageUris, options = {}) {
    const results = [];

    for (const uri of imageUris) {
      try {
        const optimized = await this.optimizeImage(uri, options);
        results.push(optimized);
      } catch (error) {
        console.error(`Failed to optimize image ${uri}:`, error);
        results.push({ uri, error: error.message });
      }
    }

    return results;
  }

  // Smart image optimization based on use case
  async optimizeForUseCase(imageUri, useCase) {
    const useCaseConfigs = {
      thumbnail: {
        maxWidth: 150,
        maxHeight: 150,
        quality: 0.7,
        format: 'JPEG'
      },
      profile: {
        maxWidth: 400,
        maxHeight: 400,
        quality: 0.8,
        format: 'JPEG'
      },
      gallery: {
        maxWidth: 800,
        maxHeight: 600,
        quality: 0.85,
        format: 'JPEG'
      },
      upload: {
        maxWidth: 1920,
        maxHeight: 1080,
        quality: 0.9,
        format: 'JPEG'
      }
    };

    const config = useCaseConfigs[useCase] || useCaseConfigs.upload;
    return this.optimizeImage(imageUri, config);
  }

  // Generate multiple sizes for responsive images
  async generateResponsiveImages(imageUri) {
    const sizes = [
      { name: 'small', maxWidth: 320, maxHeight: 240 },
      { name: 'medium', maxWidth: 640, maxHeight: 480 },
      { name: 'large', maxWidth: 1024, maxHeight: 768 },
      { name: 'xlarge', maxWidth: 1920, maxHeight: 1080 }
    ];

    const results = {};

    for (const size of sizes) {
      try {
        const optimized = await this.optimizeImage(imageUri, {
          ...size,
          quality: 0.8,
          format: 'JPEG'
        });
        results[size.name] = optimized;
      } catch (error) {
        console.error(`Failed to generate ${size.name} size:`, error);
        results[size.name] = { error: error.message };
      }
    }

    return results;
  }

  // Get image metadata
  async getImageMetadata(imageUri) {
    try {
      const response = await fetch(imageUri);
      const blob = await response.blob();

      return {
        size: blob.size,
        type: blob.type,
        uri: imageUri
      };
    } catch (error) {
      console.error('Failed to get image metadata:', error);
      return null;
    }
  }

  // Calculate optimal quality based on image size
  calculateOptimalQuality(originalSize) {
    // Smaller images can use higher quality
    if (originalSize < 100 * 1024) { // < 100KB
      return 0.95;
    } else if (originalSize < 500 * 1024) { // < 500KB
      return 0.85;
    } else if (originalSize < 1024 * 1024) { // < 1MB
      return 0.75;
    } else {
      return 0.65; // > 1MB
    }
  }

  // Compress image to target size
  async compressToTargetSize(imageUri, targetSizeKB) {
    const targetSizeBytes = targetSizeKB * 1024;
    let quality = 0.9;
    let optimizedImage = null;
    let attempts = 0;
    const maxAttempts = 5;

    while (attempts < maxAttempts) {
      try {
        optimizedImage = await this.optimizeImage(imageUri, {
          quality,
          maxWidth: this.maxWidth,
          maxHeight: this.maxHeight,
          format: this.format
        });

        if (optimizedImage.size <= targetSizeBytes) {
          break;
        }

        // Reduce quality for next attempt
        quality -= 0.1;
        attempts++;
      } catch (error) {
        console.error(`Compression attempt ${attempts + 1} failed:`, error);
        attempts++;
      }
    }

    if (optimizedImage && optimizedImage.size <= targetSizeBytes) {
      console.log(`Image compressed to ${optimizedImage.size} bytes (${quality * 100}% quality)`);
      return optimizedImage;
    } else {
      console.warn(`Could not compress image to target size after ${maxAttempts} attempts`);
      return optimizedImage;
    }
  }
}

export const imageOptimizer = new ImageOptimizer();
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Performance Fundamentals**: Metrics, budgets, and monitoring
- ✅ **Rendering Optimization**: React.memo, useMemo, useCallback, virtualized lists
- ✅ **Memory Management**: Memory leak prevention, image caching, cleanup
- ✅ **Bundle Optimization**: Code splitting, tree shaking, lazy loading
- ✅ **Image Optimization**: Compression, responsive images, caching
- ✅ **Network Optimization**: Caching, compression, connection pooling
- ✅ **Database Optimization**: Query optimization, indexing, connection pooling
- ✅ **Profiling and Monitoring**: Performance monitoring tools and techniques
- ✅ **Performance Testing**: Automated testing and benchmarking

### **Best Practices**
1. **Measure before optimizing** - Use profiling tools to identify bottlenecks
2. **Optimize rendering** - Use memoization and avoid unnecessary re-renders
3. **Manage memory efficiently** - Clean up resources and prevent memory leaks
4. **Optimize bundle size** - Use code splitting and tree shaking
5. **Compress images** - Use appropriate formats and compression levels
6. **Cache aggressively** - Cache data, images, and API responses
7. **Monitor performance** - Set up continuous monitoring and alerting
8. **Test performance** - Include performance tests in CI/CD pipeline
9. **Optimize for the platform** - Consider platform-specific optimizations
10. **Profile regularly** - Monitor performance changes over time

### **Next Steps**
- Learn advanced profiling techniques with Flipper and Chrome DevTools
- Implement automated performance regression testing
- Explore platform-specific performance optimizations
- Learn about performance monitoring in production
- Study advanced React performance patterns
- Implement performance budgets in CI/CD
- Learn about WebAssembly for performance-critical code
- Explore advanced caching strategies and CDNs

---

## 🎯 **Assignment**

### **Task 1: Performance Audit and Optimization**
Conduct a comprehensive performance audit of a React Native app:
- Profile the app using performance monitoring tools
- Identify performance bottlenecks and memory leaks
- Optimize rendering performance with memoization
- Implement bundle size optimization
- Optimize images and media assets
- Set up performance monitoring and alerting

### **Task 2: Bundle Optimization Implementation**
Implement advanced bundle optimization techniques:
- Set up code splitting and dynamic imports
- Configure tree shaking and dead code elimination
- Implement lazy loading for components and routes
- Optimize third-party library usage
- Set up bundle analysis and monitoring
- Create performance budgets and alerts

### **Task 3: Memory Management System**
Build a comprehensive memory management system:
- Implement automatic cleanup for components
- Create image caching and memory management
- Set up memory leak detection and monitoring
- Implement background cleanup processes
- Create memory usage alerts and monitoring
- Optimize large data structure handling

---

## 📚 **Additional Resources**
- [React Native Performance Documentation](https://reactnative.dev/docs/performance)
- [React Performance Best Practices](https://reactjs.org/docs/optimizing-performance.html)
- [Flipper Performance Plugin](https://fbflipper.com/docs/features/react-native/)
- [Hermes JavaScript Engine](https://hermesengine.dev/)
- [Bundle Analyzer Tools](https://github.com/webpack-contrib/webpack-bundle-analyzer)
- [Image Optimization Libraries](https://github.com/bamlab/react-native-image-resizer)

---

**Next Lesson**: [Lesson 60: Capstone Project - Complete React Native App](Lesson%2060_%20Capstone%20Project%20-%20Complete%20React%20Native%20App.md)