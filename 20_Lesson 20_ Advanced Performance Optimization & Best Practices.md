esson 20_ Advanced Performance Optimization & Best Practices.md</path>
<content">            <Text style={styles.userName}>{post.user.name}</Text>
            <Text style={styles.userHandle}>@{post.user.handle}</Text>
          </View>
        </TouchableOpacity>
        <Text style={styles.timestamp}>{post.formattedDate}</Text>
      </View>

      <Text style={styles.content}>{post.truncatedContent}</Text>

      {post.image && (
        <Image
          source={{ uri: post.image }}
          style={styles.postImage}
          resizeMode="cover"
        />
      )}

      <View style={styles.actions}>
        <TouchableOpacity style={styles.action}>
          <Text style={styles.actionText}>💬 {post.comments}</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.action}>
          <Text style={styles.actionText}>🔄 {post.retweets}</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.action}>
          <Text style={styles.actionText}>❤️ {post.likes}</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.action}>
          <Text style={styles.actionText}>📤</Text>
        </TouchableOpacity>
      </View>
    </TouchableOpacity>
  );
});

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  post: {
    backgroundColor: 'white',
    marginVertical: 5,
    marginHorizontal: 10,
    borderRadius: 12,
    padding: 15,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 10,
  },
  userInfo: {
    flexDirection: 'row',
    alignItems: 'center',
    flex: 1,
  },
  avatar: {
    width: 40,
    height: 40,
    borderRadius: 20,
    marginRight: 10,
  },
  userName: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
  },
  userHandle: {
    fontSize: 14,
    color: '#666',
  },
  timestamp: {
    fontSize: 12,
    color: '#999',
  },
  content: {
    fontSize: 16,
    color: '#333',
    lineHeight: 22,
    marginBottom: 10,
  },
  postImage: {
    width: '100%',
    height: 200,
    borderRadius: 8,
    marginBottom: 10,
  },
  actions: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    paddingTop: 10,
    borderTopWidth: 1,
    borderTopColor: '#eee',
  },
  action: {
    padding: 5,
  },
  actionText: {
    fontSize: 14,
    color: '#666',
  },
  debugInfo: {
    position: 'absolute',
    top: 10,
    right: 10,
    backgroundColor: 'rgba(0,0,0,0.8)',
    padding: 5,
    borderRadius: 5,
  },
  debugText: {
    color: 'white',
    fontSize: 10,
    fontFamily: 'monospace',
  },
});

export default OptimizedFeed;
```

---

## 📝 **Assignment 20: Advanced Performance Optimization**

### **Task 1: Performance Audit & Optimization (90 minutes)**
Conduct a comprehensive performance audit of an existing React Native app:
- Analyze bundle size and identify optimization opportunities
- Profile component render times and identify bottlenecks
- Optimize images and assets for better loading performance
- Implement proper memoization and virtualization techniques
- Set up performance monitoring and budgets

### **Task 2: Memory Management Implementation (120 minutes)**
Implement advanced memory management techniques:
- Create custom hooks for memory-efficient data fetching
- Implement proper cleanup for subscriptions and timers
- Optimize large list rendering with virtualization
- Handle memory leaks in complex component hierarchies
- Implement background processing for heavy operations

### **Task 3: Production-Ready Architecture (150 minutes)**
Build a production-ready app architecture with:
- Error boundaries and crash reporting
- Performance monitoring and analytics
- Code splitting and lazy loading
- Offline support with sync capabilities
- Security best practices and input validation

---

## 🧠 **Quiz 20: Advanced Performance Optimization**

### **Multiple Choice Questions (1-15)**

**Q1. React DevTools Profiler ka main use kya hai?**
a) UI components banana
b) Performance monitoring karna
c) Database queries run karna
d) API calls handle karna

**Q2. Code splitting ka benefit kya hai?**
a) Bundle size badhana
b) Initial load time kam karna
c) Code complexity badhana
d) Dependencies kam karna

**Q3. Tree shaking ka matlab kya hai?**
a) Unused code remove karna
b) Code structure change karna
c) Bundle size badhana
d) Dependencies add karna

**Q4. Memory leak ka main reason kya hota hai?**
a) Proper cleanup nahi karna
b) Too much code likhna
c) Large bundle size
d) Slow network

**Q5. Virtualization ka use kab hota hai?**
a) Small datasets ke liye
b) Large datasets ke liye
c) Simple components ke liye
d) Complex animations ke liye

### **True/False Questions (16-20)**

**Q16. FlatList automatically virtualization provide karta hai.** (True/False)

**Q17. Bundle size optimization sirf production mein important hai.** (True/False)

**Q18. Error boundaries child components ke errors catch karte hain.** (True/False)

**Q19. Lazy loading bundle size ko kam karta hai.** (True/False)

**Q20. Performance monitoring development mein nahi karna chahiye.** (True/False)

### **Code Output Questions (21-25)**

**Q21. Is code mein problem kya hai?**
```javascript
const Component = () => {
  const [data, setData] = useState([]);
  useEffect(() => {
    fetchData().then(setData);
  }, []); // No dependency array cleanup
};
```

**Q22. Is optimization mein kya galat hai?**
```javascript
const ListItem = ({ item, onPress }) => (
  <TouchableOpacity onPress={() => onPress(item.id)}>
    <Text>{item.name}</Text>
  </TouchableOpacity>
);
```

**Q23. Is bundle optimization mein kya missing hai?**
```javascript
import { View, Text } from 'react-native';
import _ from 'lodash'; // Entire library imported
```

**Q24. Is performance monitoring mein kya problem hai?**
```javascript
const monitor = {
  startTime: Date.now(),
  endTime: Date.now(),
  getDuration: () => this.endTime - this.startTime,
};
```

**Q25. Is error boundary mein kya galat hai?**
```javascript
class ErrorBoundary extends React.Component {
  render() {
    if (this.state.hasError) {
      return <Text>Error occurred</Text>;
    }
    return this.props.children;
  }
}
```

---

## 📚 **Answers & Explanations**

### **Multiple Choice Answers**
**Q1.** b) Performance monitoring karna  
**Q2.** b) Initial load time kam karna  
**Q3.** a) Unused code remove karna  
**Q4.** a) Proper cleanup nahi karna  
**Q5.** b) Large datasets ke liye  

### **True/False Answers**
**Q16.** True - FlatList automatically virtualization provide karta hai  
**Q17.** False - Bundle size optimization development mein bhi important hai  
**Q18.** True - Error boundaries child components ke errors catch karte hain  
**Q19.** True - Lazy loading bundle size ko kam karta hai  
**Q20.** False - Performance monitoring development mein karna chahiye  

### **Code Output Answers**
**Q21.** useEffect dependency array mein cleanup function missing hai  
**Q22.** Component memoization nahi ki gayi hai  
**Q23.** Specific lodash functions import nahi kiye gaye  
**Q24.** Performance.now() use karna chahiye Date.now() ki jagah  
**Q25.** componentDidCatch method missing hai  

---

## 🎯 **Assignment Solutions**

### **Task 1: Performance Audit Solution**

```javascript
// PerformanceAudit.js
import React, { useEffect, useState } from 'react';
import { View, Text, ScrollView, TouchableOpacity, StyleSheet, Alert } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

const PerformanceAudit = () => {
  const [auditResults, setAuditResults] = useState(null);
  const [isAuditing, setIsAuditing] = useState(false);

  const runPerformanceAudit = async () => {
    setIsAuditing(true);

    try {
      const results = {
        timestamp: new Date().toISOString(),
        bundleSize: await checkBundleSize(),
        renderPerformance: await checkRenderPerformance(),
        memoryUsage: await checkMemoryUsage(),
        networkPerformance: await checkNetworkPerformance(),
        imageOptimization: await checkImageOptimization(),
        recommendations: [],
      };

      // Generate recommendations
      results.recommendations = generateRecommendations(results);

      setAuditResults(results);

      // Save audit results
      await AsyncStorage.setItem('last_performance_audit', JSON.stringify(results));

    } catch (error) {
      console.error('Audit failed:', error);
      Alert.alert('Error', 'Performance audit failed');
    } finally {
      setIsAuditing(false);
    }
  };

  const checkBundleSize = async () => {
    // In a real app, this would analyze the actual bundle
    const mockSize = Math.random() * 10 + 5; // 5-15 MB
    return {
      size: mockSize,
      status: mockSize > 12 ? 'critical' : mockSize > 8 ? 'warning' : 'good',
    };
  };

  const checkRenderPerformance = async () => {
    const renderTimes = [];
    for (let i = 0; i < 10; i++) {
      const start = performance.now();
      await new Promise(resolve => setTimeout(resolve, 10));
      const end = performance.now();
      renderTimes.push(end - start);
    }

    const avgTime = renderTimes.reduce((a, b) => a + b, 0) / renderTimes.length;
    return {
      averageRenderTime: avgTime,
      status: avgTime > 16.67 ? 'critical' : avgTime > 8.33 ? 'warning' : 'good', // 60fps = 16.67ms
    };
  };

  const checkMemoryUsage = async () => {
    // Mock memory usage check
    const usage = Math.random() * 100 + 50; // 50-150 MB
    return {
      usage,
      status: usage > 120 ? 'critical' : usage > 80 ? 'warning' : 'good',
    };
  };

  const checkNetworkPerformance = async () => {
    const start = performance.now();
    try {
      const response = await fetch('https://httpbin.org/delay/1');
      const end = performance.now();
      return {
        responseTime: end - start,
        status: end - start > 2000 ? 'critical' : end - start > 1000 ? 'warning' : 'good',
      };
    } catch (error) {
      return {
        responseTime: -1,
        status: 'critical',
        error: error.message,
      };
    }
  };

  const checkImageOptimization = async () => {
    // Mock image optimization check
    const optimizedCount = Math.floor(Math.random() * 20);
    const totalCount = 25;
    const optimizationRate = optimizedCount / totalCount;

    return {
      optimizedImages: optimizedCount,
      totalImages: totalCount,
      optimizationRate,
      status: optimizationRate < 0.5 ? 'critical' : optimizationRate < 0.8 ? 'warning' : 'good',
    };
  };

  const generateRecommendations = (results) => {
    const recommendations = [];

    if (results.bundleSize.status === 'critical') {
      recommendations.push('Bundle size is too large. Consider code splitting and tree shaking.');
    }

    if (results.renderPerformance.status === 'critical') {
      recommendations.push('Render performance is poor. Implement memoization and optimize re-renders.');
    }

    if (results.memoryUsage.status === 'critical') {
      recommendations.push('Memory usage is high. Check for memory leaks and optimize data structures.');
    }

    if (results.networkPerformance.status === 'critical') {
      recommendations.push('Network performance is slow. Implement caching and optimize API calls.');
    }

    if (results.imageOptimization.status !== 'good') {
      recommendations.push('Images need optimization. Use appropriate sizes and formats.');
    }

    return recommendations;
  };

  const getStatusColor = (status) => {
    switch (status) {
      case 'good': return '#28a745';
      case 'warning': return '#ffc107';
      case 'critical': return '#dc3545';
      default: return '#6c757d';
    }
  };

  const getStatusText = (status) => {
    switch (status) {
      case 'good': return 'Good';
      case 'warning': return 'Needs Attention';
      case 'critical': return 'Critical';
      default: return 'Unknown';
    }
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Performance Audit</Text>

      <TouchableOpacity
        style={[styles.auditButton, isAuditing && styles.auditButtonDisabled]}
        onPress={runPerformanceAudit}
        disabled={isAuditing}
      >
        <Text style={styles.auditButtonText}>
          {isAuditing ? 'Running Audit...' : 'Run Performance Audit'}
        </Text>
      </TouchableOpacity>

      {auditResults && (
        <View style={styles.results}>
          <Text style={styles.resultsTitle}>Audit Results</Text>
          <Text style={styles.timestamp}>
            {new Date(auditResults.timestamp).toLocaleString()}
          </Text>

          <View style={styles.metric}>
            <Text style={styles.metricLabel}>Bundle Size</Text>
            <Text style={[styles.metricValue, { color: getStatusColor(auditResults.bundleSize.status) }]}>
              {auditResults.bundleSize.size.toFixed(1)} MB
            </Text>
            <Text style={[styles.metricStatus, { color: getStatusColor(auditResults.bundleSize.status) }]}>
              {getStatusText(auditResults.bundleSize.status)}
            </Text>
          </View>

          <View style={styles.metric}>
            <Text style={styles.metricLabel}>Avg Render Time</Text>
            <Text style={[styles.metricValue, { color: getStatusColor(auditResults.renderPerformance.status) }]}>
              {auditResults.renderPerformance.averageRenderTime.toFixed(1)} ms
            </Text>
            <Text style={[styles.metricStatus, { color: getStatusColor(auditResults.renderPerformance.status) }]}>
              {getStatusText(auditResults.renderPerformance.status)}
            </Text>
          </View>

          <View style={styles.metric}>
            <Text style={styles.metricLabel}>Memory Usage</Text>
            <Text style={[styles.metricValue, { color: getStatusColor(auditResults.memoryUsage.status) }]}>
              {auditResults.memoryUsage.usage.toFixed(1)} MB
            </Text>
            <Text style={[styles.metricStatus, { color: getStatusColor(auditResults.memoryUsage.status) }]}>
              {getStatusText(auditResults.memoryUsage.status)}
            </Text>
          </View>

          <View style={styles.metric}>
            <Text style={styles.metricLabel}>Network Response</Text>
            <Text style={[styles.metricValue, { color: getStatusColor(auditResults.networkPerformance.status) }]}>
              {auditResults.networkPerformance.responseTime > 0
                ? `${auditResults.networkPerformance.responseTime.toFixed(0)} ms`
                : 'Failed'
              }
            </Text>
            <Text style={[styles.metricStatus, { color: getStatusColor(auditResults.networkPerformance.status) }]}>
              {getStatusText(auditResults.networkPerformance.status)}
            </Text>
          </View>

          <View style={styles.metric}>
            <Text style={styles.metricLabel}>Image Optimization</Text>
            <Text style={[styles.metricValue, { color: getStatusColor(auditResults.imageOptimization.status) }]}>
              {Math.round(auditResults.imageOptimization.optimizationRate * 100)}%
            </Text>
            <Text style={[styles.metricStatus, { color: getStatusColor(auditResults.imageOptimization.status) }]}>
              {getStatusText(auditResults.imageOptimization.status)}
            </Text>
          </View>

          {auditResults.recommendations.length > 0 && (
            <View style={styles.recommendations}>
              <Text style={styles.recommendationsTitle}>Recommendations</Text>
              {auditResults.recommendations.map((rec, index) => (
                <Text key={index} style={styles.recommendation}>
                  • {rec}
                </Text>
              ))}
            </View>
          )}
        </View>
      )}
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
    color: '#333',
  },
  auditButton: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    alignItems: 'center',
    marginBottom: 20,
  },
  auditButtonDisabled: {
    backgroundColor: '#ccc',
  },
  auditButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  results: {
    backgroundColor: 'white',
    borderRadius: 10,
    padding: 20,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  resultsTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333',
  },
  timestamp: {
    fontSize: 14,
    color: '#666',
    marginBottom: 20,
  },
  metric: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingVertical: 10,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  metricLabel: {
    fontSize: 16,
    color: '#333',
    flex: 1,
  },
  metricValue: {
    fontSize: 16,
    fontWeight: 'bold',
    flex: 1,
    textAlign: 'center',
  },
  metricStatus: {
    fontSize: 14,
    flex: 1,
    textAlign: 'right',
  },
  recommendations: {
    marginTop: 20,
    paddingTop: 20,
    borderTopWidth: 1,
    borderTopColor: '#eee',
  },
  recommendationsTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333',
  },
  recommendation: {
    fontSize: 14,
    color: '#666',
    marginBottom: 5,
    lineHeight: 20,
  },
});

export default PerformanceAudit;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Performance Monitoring**: React DevTools, custom hooks, Flipper integration
- ✅ **Advanced Rendering**: Memoization, virtualization, chunked rendering
- ✅ **Bundle Optimization**: Code splitting, tree shaking, asset optimization
- ✅ **Memory Management**: Leak prevention, large list optimization, cleanup
- ✅ **Network Optimization**: Request caching, batching, connection monitoring
- ✅ **Image Optimization**: Smart loading, pipeline processing, lazy loading
- ✅ **Database Optimization**: SQLite best practices, indexing, query optimization
- ✅ **Architecture Patterns**: Container/presentational, custom hooks, error boundaries
- ✅ **Production Best Practices**: Error handling, monitoring, performance budgets
- ✅ **Practical Examples**: Optimized social feed, performance audit tool

### **Best Practices**
1. **Always measure performance** before and after optimizations
2. **Use React DevTools** for development performance monitoring
3. **Implement proper memoization** to prevent unnecessary re-renders
4. **Optimize bundle size** through code splitting and tree shaking
5. **Monitor memory usage** and prevent leaks with proper cleanup
6. **Cache network requests** and implement offline support
7. **Optimize images** for different screen sizes and network conditions
8. **Use virtualization** for large lists and datasets
9. **Implement error boundaries** for graceful error handling
10. **Set performance budgets** and monitor them regularly

### **Next Steps**
- Implement automated performance testing
- Set up continuous performance monitoring
- Create performance dashboards for teams
- Learn about advanced profiling techniques
- Study platform-specific optimizations

---

## 📚 **Additional Resources**
- [React Native Performance Documentation](https://reactnative.dev/docs/performance)
- [React DevTools](https://reactnative.dev/docs/debugging#react-developer-tools)
- [Flipper](https://fbflipper.com/)
- [Bundle Analyzer](https://github.com/facebook/create-react-app/blob/main/packages/react-scripts/template/README.md#analyzing-the-bundle-size)
- [Performance Best Practices](https://reactnative.dev/docs/optimizing-flatlist-configuration)

---

**Next Lesson**: [Lesson 21: Backend Fundamentals (Node.js & Express)](Lesson%2021_%20Backend%20Fundamentals%20(Node.js%20&%20Express).md)