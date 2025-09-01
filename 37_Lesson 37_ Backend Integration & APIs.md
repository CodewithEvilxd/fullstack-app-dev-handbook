# Lesson 37: Backend Integration & APIs

## 🎯 **Learning Objectives**
- Master backend integration patterns in React Native
- Implement RESTful API communication
- Handle authentication and authorization
- Create robust API service layers
- Implement error handling and retry mechanisms

## 📚 **Table of Contents**
1. [API Integration Fundamentals](#api-integration-fundamentals)
2. [HTTP Client Setup](#http-client-setup)
3. [Authentication & Authorization](#authentication--authorization)
4. [API Service Layer](#api-service-layer)
5. [Error Handling](#error-handling)
6. [Request/Response Interceptors](#requestresponse-interceptors)
7. [Caching Strategies](#caching-strategies)
8. [Real-time Communication](#real-time-communication)
9. [API Testing](#api-testing)
10. [Practical Examples](#practical-examples)

---

## 🌐 **API Integration Fundamentals**

### **RESTful API Principles**
- **GET**: Retrieve data
- **POST**: Create new resources
- **PUT/PATCH**: Update existing resources
- **DELETE**: Remove resources

### **HTTP Status Codes**
```javascript
const HTTP_STATUS = {
  OK: 200,
  CREATED: 201,
  NO_CONTENT: 204,
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  FORBIDDEN: 403,
  NOT_FOUND: 404,
  INTERNAL_SERVER_ERROR: 500,
};
```

### **API Response Structure**
```javascript
// Standard API response format
const apiResponse = {
  success: true,
  data: {}, // Actual data
  message: 'Operation successful',
  errors: [], // Validation errors
  meta: {
    page: 1,
    limit: 10,
    total: 100,
  },
};
```

---

## 🔧 **HTTP Client Setup**

### **Axios Configuration**
```javascript
// services/apiClient.js
import axios from 'axios';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Create axios instance
const apiClient = axios.create({
  baseURL: 'https://api.myapp.com',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  },
});

// Request interceptor
apiClient.interceptors.request.use(
  async (config) => {
    // Add auth token
    const token = await AsyncStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    // Add request ID for tracking
    config.metadata = {
      requestId: Date.now(),
      startTime: new Date(),
    };

    console.log(`🚀 API Request: ${config.method?.toUpperCase()} ${config.url}`);
    return config;
  },
  (error) => {
    console.error('Request error:', error);
    return Promise.reject(error);
  }
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => {
    const duration = new Date() - response.config.metadata.startTime;
    console.log(`✅ API Response: ${response.status} (${duration}ms)`);

    return response;
  },
  async (error) => {
    const { response, config } = error;

    if (response) {
      // Server responded with error status
      const { status, data } = response;

      console.error(`❌ API Error: ${status} - ${data?.message || 'Unknown error'}`);

      // Handle specific error codes
      if (status === 401) {
        // Token expired, redirect to login
        await AsyncStorage.removeItem('authToken');
        // navigation.navigate('Login');
      } else if (status === 403) {
        // Forbidden, show error message
        console.error('Access forbidden');
      } else if (status >= 500) {
        // Server error, retry logic could be implemented here
        console.error('Server error');
      }
    } else if (error.code === 'NETWORK_ERROR') {
      // Network error
      console.error('Network error - check internet connection');
    } else {
      // Other error
      console.error('Unknown error:', error.message);
    }

    return Promise.reject(error);
  }
);

export default apiClient;
```

### **Fetch API Alternative**
```javascript
// services/fetchClient.js
class FetchClient {
  constructor(baseURL = '', defaultHeaders = {}) {
    this.baseURL = baseURL;
    this.defaultHeaders = {
      'Content-Type': 'application/json',
      ...defaultHeaders,
    };
  }

  async request(endpoint, options = {}) {
    const url = this.baseURL + endpoint;
    const config = {
      headers: { ...this.defaultHeaders, ...options.headers },
      ...options,
    };

    // Add auth token
    const token = await AsyncStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    try {
      const response = await fetch(url, config);
      return await this.handleResponse(response);
    } catch (error) {
      return this.handleError(error);
    }
  }

  async handleResponse(response) {
    const { status, statusText } = response;

    if (status >= 200 && status < 300) {
      const contentType = response.headers.get('content-type');
      if (contentType && contentType.includes('application/json')) {
        return await response.json();
      }
      return await response.text();
    }

    // Handle error responses
    let errorMessage = statusText;
    try {
      const errorData = await response.json();
      errorMessage = errorData.message || errorMessage;
    } catch (e) {
      // Response is not JSON
    }

    throw new Error(`${status}: ${errorMessage}`);
  }

  handleError(error) {
    if (error.name === 'TypeError' && error.message.includes('fetch')) {
      throw new Error('Network error - check your internet connection');
    }
    throw error;
  }

  // HTTP methods
  get(endpoint, options = {}) {
    return this.request(endpoint, { ...options, method: 'GET' });
  }

  post(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  put(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }

  patch(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'PATCH',
      body: JSON.stringify(data),
    });
  }

  delete(endpoint, options = {}) {
    return this.request(endpoint, { ...options, method: 'DELETE' });
  }
}

// Create instance
const apiClient = new FetchClient('https://api.myapp.com');

export default apiClient;
```

---

## 🔐 **Authentication & Authorization**

### **JWT Token Management**
```javascript
// services/authService.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import apiClient from './apiClient';

class AuthService {
  constructor() {
    this.tokenKey = 'authToken';
    this.refreshTokenKey = 'refreshToken';
    this.userKey = 'userData';
  }

  // Login
  async login(credentials) {
    try {
      const response = await apiClient.post('/auth/login', credentials);

      const { token, refreshToken, user } = response.data;

      // Store tokens and user data
      await AsyncStorage.multiSet([
        [this.tokenKey, token],
        [this.refreshTokenKey, refreshToken],
        [this.userKey, JSON.stringify(user)],
      ]);

      return { token, user };
    } catch (error) {
      throw new Error(error.response?.data?.message || 'Login failed');
    }
  }

  // Register
  async register(userData) {
    try {
      const response = await apiClient.post('/auth/register', userData);

      const { token, refreshToken, user } = response.data;

      await AsyncStorage.multiSet([
        [this.tokenKey, token],
        [this.refreshTokenKey, refreshToken],
        [this.userKey, JSON.stringify(user)],
      ]);

      return { token, user };
    } catch (error) {
      throw new Error(error.response?.data?.message || 'Registration failed');
    }
  }

  // Logout
  async logout() {
    try {
      // Call logout endpoint
      await apiClient.post('/auth/logout');
    } catch (error) {
      console.error('Logout API error:', error);
    } finally {
      // Clear local storage regardless of API response
      await AsyncStorage.multiRemove([
        this.tokenKey,
        this.refreshTokenKey,
        this.userKey,
      ]);
    }
  }

  // Get stored token
  async getToken() {
    return await AsyncStorage.getItem(this.tokenKey);
  }

  // Get stored user
  async getUser() {
    const userData = await AsyncStorage.getItem(this.userKey);
    return userData ? JSON.parse(userData) : null;
  }

  // Check if user is authenticated
  async isAuthenticated() {
    const token = await this.getToken();
    return !!token;
  }

  // Refresh token
  async refreshToken() {
    try {
      const refreshToken = await AsyncStorage.getItem(this.refreshTokenKey);

      if (!refreshToken) {
        throw new Error('No refresh token available');
      }

      const response = await apiClient.post('/auth/refresh', {
        refreshToken,
      });

      const { token, refreshToken: newRefreshToken } = response.data;

      // Update stored tokens
      await AsyncStorage.multiSet([
        [this.tokenKey, token],
        [this.refreshTokenKey, newRefreshToken],
      ]);

      return token;
    } catch (error) {
      // Refresh failed, logout user
      await this.logout();
      throw new Error('Session expired');
    }
  }

  // Auto refresh token before expiry
  startTokenRefreshTimer() {
    // Refresh token 5 minutes before expiry
    const refreshInterval = (55 * 60 * 1000); // 55 minutes

    this.refreshTimer = setInterval(async () => {
      try {
        await this.refreshToken();
      } catch (error) {
        console.error('Auto token refresh failed:', error);
      }
    }, refreshInterval);
  }

  // Stop token refresh timer
  stopTokenRefreshTimer() {
    if (this.refreshTimer) {
      clearInterval(this.refreshTimer);
      this.refreshTimer = null;
    }
  }
}

export default new AuthService();
```

### **OAuth Integration**
```javascript
// services/oauthService.js
import { authorize } from 'react-native-app-auth';
import AsyncStorage from '@react-native-async-storage/async-storage';

const oauthConfig = {
  issuer: 'https://accounts.google.com',
  clientId: 'your-client-id',
  redirectUrl: 'com.yourapp:/oauth2redirect',
  scopes: ['openid', 'profile', 'email'],
  serviceConfiguration: {
    authorizationEndpoint: 'https://accounts.google.com/o/oauth2/v2/auth',
    tokenEndpoint: 'https://oauth2.googleapis.com/token',
    revocationEndpoint: 'https://oauth2.googleapis.com/revoke',
  },
};

class OAuthService {
  async signInWithGoogle() {
    try {
      const result = await authorize(oauthConfig);

      // Store tokens
      await AsyncStorage.setItem('oauthToken', result.accessToken);
      await AsyncStorage.setItem('oauthRefreshToken', result.refreshToken);

      // Get user info
      const userInfo = await this.getUserInfo(result.accessToken);

      return {
        token: result.accessToken,
        user: userInfo,
      };
    } catch (error) {
      console.error('OAuth error:', error);
      throw new Error('Google sign-in failed');
    }
  }

  async getUserInfo(accessToken) {
    try {
      const response = await fetch('https://www.googleapis.com/oauth2/v2/userinfo', {
        headers: {
          Authorization: `Bearer ${accessToken}`,
        },
      });

      if (!response.ok) {
        throw new Error('Failed to get user info');
      }

      return await response.json();
    } catch (error) {
      console.error('Get user info error:', error);
      throw error;
    }
  }

  async refreshToken() {
    try {
      const refreshToken = await AsyncStorage.getItem('oauthRefreshToken');

      if (!refreshToken) {
        throw new Error('No refresh token available');
      }

      const result = await authorize({
        ...oauthConfig,
        additionalParameters: {
          grant_type: 'refresh_token',
          refresh_token: refreshToken,
        },
      });

      await AsyncStorage.setItem('oauthToken', result.accessToken);
      return result.accessToken;
    } catch (error) {
      console.error('Token refresh error:', error);
      throw new Error('Token refresh failed');
    }
  }

  async signOut() {
    try {
      const token = await AsyncStorage.getItem('oauthToken');

      if (token) {
        // Revoke token
        await fetch(oauthConfig.serviceConfiguration.revocationEndpoint, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
          },
          body: `token=${token}`,
        });
      }
    } catch (error) {
      console.error('Sign out error:', error);
    } finally {
      // Clear local storage
      await AsyncStorage.multiRemove(['oauthToken', 'oauthRefreshToken']);
    }
  }
}

export default new OAuthService();
```

---

## 🏗️ **API Service Layer**

### **Base API Service**
```javascript
// services/baseApiService.js
class BaseApiService {
  constructor(baseURL = '') {
    this.baseURL = baseURL;
    this.defaultHeaders = {
      'Content-Type': 'application/json',
    };
  }

  async request(endpoint, options = {}) {
    const url = this.baseURL + endpoint;
    const config = {
      headers: { ...this.defaultHeaders, ...options.headers },
      ...options,
    };

    // Add auth token if available
    const token = await this.getAuthToken();
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    const startTime = Date.now();

    try {
      const response = await fetch(url, config);
      const duration = Date.now() - startTime;

      return await this.handleResponse(response, duration);
    } catch (error) {
      return this.handleError(error);
    }
  }

  async getAuthToken() {
    // Override in subclasses to get token from storage/context
    return null;
  }

  async handleResponse(response, duration) {
    const { status } = response;

    if (status >= 200 && status < 300) {
      const contentType = response.headers.get('content-type');

      if (contentType && contentType.includes('application/json')) {
        const data = await response.json();
        console.log(`✅ ${response.url} - ${status} (${duration}ms)`);
        return data;
      }

      const text = await response.text();
      console.log(`✅ ${response.url} - ${status} (${duration}ms)`);
      return text;
    }

    // Handle error responses
    let errorMessage = response.statusText;
    try {
      const errorData = await response.json();
      errorMessage = errorData.message || errorMessage;
    } catch (e) {
      // Response is not JSON
    }

    console.error(`❌ ${response.url} - ${status} (${duration}ms): ${errorMessage}`);
    throw new ApiError(status, errorMessage);
  }

  handleError(error) {
    if (error.name === 'TypeError' && error.message.includes('fetch')) {
      throw new NetworkError('Network error - check your internet connection');
    }
    throw error;
  }

  // HTTP methods
  get(endpoint, params = {}, options = {}) {
    const queryString = new URLSearchParams(params).toString();
    const url = queryString ? `${endpoint}?${queryString}` : endpoint;
    return this.request(url, { ...options, method: 'GET' });
  }

  post(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  put(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }

  patch(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'PATCH',
      body: JSON.stringify(data),
    });
  }

  delete(endpoint, options = {}) {
    return this.request(endpoint, { ...options, method: 'DELETE' });
  }
}

// Custom error classes
export class ApiError extends Error {
  constructor(status, message) {
    super(message);
    this.name = 'ApiError';
    this.status = status;
  }
}

export class NetworkError extends Error {
  constructor(message) {
    super(message);
    this.name = 'NetworkError';
  }
}

export default BaseApiService;
```

### **Specific API Services**
```javascript
// services/userService.js
import BaseApiService from './baseApiService';

class UserService extends BaseApiService {
  constructor() {
    super('https://api.myapp.com/users');
  }

  async getAuthToken() {
    // Get token from AsyncStorage or context
    return await AsyncStorage.getItem('authToken');
  }

  // User-specific methods
  async getProfile(userId) {
    return this.get(`/${userId}`);
  }

  async updateProfile(userId, userData) {
    return this.put(`/${userId}`, userData);
  }

  async updateAvatar(userId, avatarUri) {
    const formData = new FormData();
    formData.append('avatar', {
      uri: avatarUri,
      type: 'image/jpeg',
      name: 'avatar.jpg',
    });

    return this.request(`/${userId}/avatar`, {
      method: 'POST',
      body: formData,
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    });
  }

  async getFollowers(userId, page = 1, limit = 20) {
    return this.get(`/${userId}/followers`, { page, limit });
  }

  async getFollowing(userId, page = 1, limit = 20) {
    return this.get(`/${userId}/following`, { page, limit });
  }

  async followUser(userId) {
    return this.post(`/${userId}/follow`);
  }

  async unfollowUser(userId) {
    return this.delete(`/${userId}/follow`);
  }

  async searchUsers(query, page = 1, limit = 20) {
    return this.get('/search', { q: query, page, limit });
  }
}

export default new UserService();

// services/postService.js
import BaseApiService from './baseApiService';

class PostService extends BaseApiService {
  constructor() {
    super('https://api.myapp.com/posts');
  }

  async getAuthToken() {
    return await AsyncStorage.getItem('authToken');
  }

  // Post-specific methods
  async getPosts(page = 1, limit = 10, filters = {}) {
    const params = { page, limit, ...filters };
    return this.get('/', params);
  }

  async getPost(postId) {
    return this.get(`/${postId}`);
  }

  async createPost(postData) {
    return this.post('/', postData);
  }

  async updatePost(postId, postData) {
    return this.put(`/${postId}`, postData);
  }

  async deletePost(postId) {
    return this.delete(`/${postId}`);
  }

  async likePost(postId) {
    return this.post(`/${postId}/like`);
  }

  async unlikePost(postId) {
    return this.delete(`/${postId}/like`);
  }

  async getComments(postId, page = 1, limit = 20) {
    return this.get(`/${postId}/comments`, { page, limit });
  }

  async addComment(postId, commentData) {
    return this.post(`/${postId}/comments`, commentData);
  }

  async updateComment(postId, commentId, commentData) {
    return this.put(`/${postId}/comments/${commentId}`, commentData);
  }

  async deleteComment(postId, commentId) {
    return this.delete(`/${postId}/comments/${commentId}`);
  }
}

export default new PostService();
```

---

## 🚨 **Error Handling**

### **Global Error Handler**
```javascript
// services/errorHandler.js
import { Alert } from 'react-native';

class ErrorHandler {
  static handle(error, context = '') {
    console.error(`Error in ${context}:`, error);

    // Handle different error types
    if (error.name === 'NetworkError') {
      this.handleNetworkError(error);
    } else if (error.name === 'ApiError') {
      this.handleApiError(error);
    } else if (error.name === 'ValidationError') {
      this.handleValidationError(error);
    } else {
      this.handleGenericError(error);
    }
  }

  static handleNetworkError(error) {
    Alert.alert(
      'Connection Error',
      'Please check your internet connection and try again.',
      [{ text: 'OK' }]
    );
  }

  static handleApiError(error) {
    const { status } = error;

    switch (status) {
      case 400:
        Alert.alert('Bad Request', 'Please check your input and try again.');
        break;
      case 401:
        Alert.alert('Unauthorized', 'Please log in to continue.');
        // Redirect to login
        break;
      case 403:
        Alert.alert('Forbidden', 'You do not have permission to perform this action.');
        break;
      case 404:
        Alert.alert('Not Found', 'The requested resource was not found.');
        break;
      case 422:
        Alert.alert('Validation Error', error.message);
        break;
      case 500:
        Alert.alert('Server Error', 'Something went wrong on our end. Please try again later.');
        break;
      default:
        Alert.alert('Error', 'An unexpected error occurred. Please try again.');
    }
  }

  static handleValidationError(error) {
    const messages = error.errors?.map(err => `• ${err.message}`).join('\n') || error.message;

    Alert.alert(
      'Validation Error',
      messages,
      [{ text: 'OK' }]
    );
  }

  static handleGenericError(error) {
    Alert.alert(
      'Error',
      error.message || 'An unexpected error occurred.',
      [{ text: 'OK' }]
    );
  }

  // Retry mechanism
  static async withRetry(fn, maxRetries = 3, delay = 1000) {
    let lastError;

    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return await fn();
      } catch (error) {
        lastError = error;

        if (attempt === maxRetries) {
          break;
        }

        // Don't retry on client errors (4xx)
        if (error.status >= 400 && error.status < 500) {
          break;
        }

        // Wait before retrying
        await new Promise(resolve => setTimeout(resolve, delay * attempt));
      }
    }

    throw lastError;
  }
}

export default ErrorHandler;
```

### **Error Boundary for API Calls**
```javascript
// components/ApiErrorBoundary.js
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import ErrorHandler from '../services/errorHandler';

class ApiErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    ErrorHandler.handle(error, 'ApiErrorBoundary');
  }

  handleRetry = () => {
    this.setState({ hasError: false, error: null });

    // Call retry function if provided
    if (this.props.onRetry) {
      this.props.onRetry();
    }
  };

  render() {
    if (this.state.hasError) {
      return (
        <View style={styles.container}>
          <Text style={styles.title}>Oops! Something went wrong</Text>
          <Text style={styles.message}>
            {this.state.error?.message || 'An unexpected error occurred'}
          </Text>

          <View style={styles.actions}>
            <TouchableOpacity style={styles.retryButton} onPress={this.handleRetry}>
              <Text style={styles.retryButtonText}>Try Again</Text>
            </TouchableOpacity>

            <TouchableOpacity
              style={styles.reportButton}
              onPress={() => ErrorHandler.handle(this.state.error, 'User reported')}
            >
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
    marginBottom: 10,
    textAlign: 'center',
  },
  message: {
    fontSize: 16,
    textAlign: 'center',
    marginBottom: 30,
    color: '#666',
  },
  actions: {
    flexDirection: 'row',
    gap: 15,
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
    backgroundColor: '#FF9500',
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

export default ApiErrorBoundary;
```

---

## 🔄 **Request/Response Interceptors**

### **Advanced Interceptors**
```javascript
// services/interceptors.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import NetInfo from '@react-native-community/netinfo';

// Request interceptor
export const requestInterceptor = async (config) => {
  // Add timestamp
  config.metadata = {
    ...config.metadata,
    requestStartTime: Date.now(),
  };

  // Add request ID for tracking
  config.requestId = Date.now().toString();

  // Check network connectivity
  const networkState = await NetInfo.fetch();
  if (!networkState.isConnected) {
    throw new Error('No internet connection');
  }

  // Add auth token
  const token = await AsyncStorage.getItem('authToken');
  if (token) {
    config.headers = {
      ...config.headers,
      Authorization: `Bearer ${token}`,
    };
  }

  // Add device info
  config.headers = {
    ...config.headers,
    'X-Device-Type': 'mobile',
    'X-App-Version': '1.0.0',
    'X-Request-ID': config.requestId,
  };

  // Log request
  console.log(`🚀 [${config.requestId}] ${config.method?.toUpperCase()} ${config.url}`);

  return config;
};

// Response interceptor
export const responseInterceptor = (response) => {
  const { config, status } = response;
  const duration = Date.now() - config.metadata?.requestStartTime;

  // Log successful response
  console.log(`✅ [${config.requestId}] ${status} (${duration}ms)`);

  // Add response metadata
  response.metadata = {
    duration,
    requestId: config.requestId,
  };

  return response;
};

// Error interceptor
export const errorInterceptor = async (error) => {
  const { config, response } = error;

  if (config) {
    const duration = Date.now() - config.metadata?.requestStartTime;
    console.error(`❌ [${config.requestId}] ${error.message} (${duration}ms)`);
  }

  // Handle token refresh
  if (response?.status === 401 && !config._retry) {
    config._retry = true;

    try {
      // Attempt to refresh token
      const refreshToken = await AsyncStorage.getItem('refreshToken');
      if (refreshToken) {
        const refreshResponse = await fetch('/api/auth/refresh', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ refreshToken }),
        });

        if (refreshResponse.ok) {
          const { token } = await refreshResponse.json();
          await AsyncStorage.setItem('authToken', token);

          // Retry original request with new token
          config.headers.Authorization = `Bearer ${token}`;
          return axios(config);
        }
      }
    } catch (refreshError) {
      console.error('Token refresh failed:', refreshError);
    }

    // Token refresh failed, redirect to login
    await AsyncStorage.multiRemove(['authToken', 'refreshToken']);
    // navigation.navigate('Login');
  }

  // Handle network errors
  if (!response) {
    console.error('Network error - no response received');
  }

  return Promise.reject(error);
};

// Setup interceptors
export const setupInterceptors = (axiosInstance) => {
  axiosInstance.interceptors.request.use(requestInterceptor, errorInterceptor);
  axiosInstance.interceptors.response.use(responseInterceptor, errorInterceptor);
};
```

---

## 💾 **Caching Strategies**

### **HTTP Caching**
```javascript
// services/cache.js
import AsyncStorage from '@react-native-async-storage/async-storage';

class ApiCache {
  constructor() {
    this.cachePrefix = '@api_cache_';
    this.defaultTTL = 5 * 60 * 1000; // 5 minutes
  }

  generateKey(endpoint, params = {}) {
    const sortedParams = Object.keys(params)
      .sort()
      .reduce((result, key) => {
        result[key] = params[key];
        return result;
      }, {});

    return this.cachePrefix + btoa(JSON.stringify({ endpoint, params: sortedParams }));
  }

  async get(endpoint, params = {}) {
    try {
      const key = this.generateKey(endpoint, params);
      const cachedData = await AsyncStorage.getItem(key);

      if (!cachedData) {
        return null;
      }

      const { data, timestamp, ttl } = JSON.parse(cachedData);
      const now = Date.now();
      const expiryTime = timestamp + (ttl || this.defaultTTL);

      if (now > expiryTime) {
        // Cache expired, remove it
        await AsyncStorage.removeItem(key);
        return null;
      }

      console.log(`📦 Cache hit for ${endpoint}`);
      return data;
    } catch (error) {
      console.error('Cache read error:', error);
      return null;
    }
  }

  async set(endpoint, params = {}, data, ttl = null) {
    try {
      const key = this.generateKey(endpoint, params);
      const cacheData = {
        data,
        timestamp: Date.now(),
        ttl: ttl || this.defaultTTL,
      };

      await AsyncStorage.setItem(key, JSON.stringify(cacheData));
      console.log(`💾 Cached data for ${endpoint}`);
    } catch (error) {
      console.error('Cache write error:', error);
    }
  }

  async invalidate(endpoint, params = {}) {
    try {
      const key = this.generateKey(endpoint, params);
      await AsyncStorage.removeItem(key);
      console.log(`🗑️ Invalidated cache for ${endpoint}`);
    } catch (error) {
      console.error('Cache invalidation error:', error);
    }
  }

  async invalidatePattern(pattern) {
    try {
      const keys = await AsyncStorage.getAllKeys();
      const cacheKeys = keys.filter(key => key.startsWith(this.cachePrefix));

      const keysToRemove = cacheKeys.filter(key => {
        const decodedKey = JSON.parse(atob(key.replace(this.cachePrefix, '')));
        return decodedKey.endpoint.includes(pattern);
      });

      await AsyncStorage.multiRemove(keysToRemove);
      console.log(`🗑️ Invalidated ${keysToRemove.length} cache entries matching ${pattern}`);
    } catch (error) {
      console.error('Pattern invalidation error:', error);
    }
  }

  async clear() {
    try {
      const keys = await AsyncStorage.getAllKeys();
      const cacheKeys = keys.filter(key => key.startsWith(this.cachePrefix));
      await AsyncStorage.multiRemove(cacheKeys);
      console.log(`🗑️ Cleared all cache (${cacheKeys.length} entries)`);
    } catch (error) {
      console.error('Cache clear error:', error);
    }
  }
}

export default new ApiCache();
```

### **Cache-Aware API Client**
```javascript
// services/cachedApiClient.js
import apiClient from './apiClient';
import cache from './cache';

class CachedApiClient {
  constructor() {
    this.client = apiClient;
  }

  async get(endpoint, params = {}, options = {}) {
    const { useCache = true, cacheTTL } = options;

    // Try to get from cache first
    if (useCache) {
      const cachedData = await cache.get(endpoint, params);
      if (cachedData) {
        return cachedData;
      }
    }

    // Fetch from API
    try {
      const response = await this.client.get(endpoint, params);

      // Cache the response
      if (useCache && response.data) {
        await cache.set(endpoint, params, response.data, cacheTTL);
      }

      return response.data;
    } catch (error) {
      // If API fails and we have cache, return cached data
      if (useCache && error.response?.status >= 500) {
        const cachedData = await cache.get(endpoint, params);
        if (cachedData) {
          console.log('🚨 API failed, returning cached data');
          return cachedData;
        }
      }

      throw error;
    }
  }

  // Other methods without caching
  post(endpoint, data, options = {}) {
    return this.client.post(endpoint, data, options);
  }

  put(endpoint, data, options = {}) {
    return this.client.put(endpoint, data, options);
  }

  patch(endpoint, data, options = {}) {
    return this.client.patch(endpoint, data, options);
  }

  delete(endpoint, options = {}) {
    return this.client.delete(endpoint, options);
  }

  // Cache management methods
  async invalidateCache(endpoint, params = {}) {
    await cache.invalidate(endpoint, params);
  }

  async clearCache() {
    await cache.clear();
  }
}

export default new CachedApiClient();
```

---

## 🔄 **Real-time Communication**

### **WebSocket Integration**
```javascript
// services/websocketService.js
import AsyncStorage from '@react-native-async-storage/async-storage';

class WebSocketService {
  constructor() {
    this.ws = null;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
    this.reconnectInterval = 1000;
    this.listeners = new Map();
  }

  async connect() {
    try {
      const token = await AsyncStorage.getItem('authToken');
      const wsUrl = `wss://api.myapp.com/ws?token=${token}`;

      this.ws = new WebSocket(wsUrl);

      this.ws.onopen = this.onOpen.bind(this);
      this.ws.onmessage = this.onMessage.bind(this);
      this.ws.onclose = this.onClose.bind(this);
      this.ws.onerror = this.onError.bind(this);

    } catch (error) {
      console.error('WebSocket connection error:', error);
      this.handleReconnect();
    }
  }

  onOpen() {
    console.log('WebSocket connected');
    this.reconnectAttempts = 0;

    // Send authentication message
    this.send({
      type: 'authenticate',
      token: await AsyncStorage.getItem('authToken'),
    });
  }

  onMessage(event) {
    try {
      const message = JSON.parse(event.data);
      console.log('WebSocket message received:', message);

      // Notify listeners
      const listeners = this.listeners.get(message.type) || [];
      listeners.forEach(callback => callback(message.data));

    } catch (error) {
      console.error('WebSocket message parse error:', error);
    }
  }

  onClose(event) {
    console.log('WebSocket disconnected:', event.code, event.reason);
    this.handleReconnect();
  }

  onError(error) {
    console.error('WebSocket error:', error);
  }

  handleReconnect() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      console.log(`Attempting to reconnect (${this.reconnectAttempts}/${this.maxReconnectAttempts})`);

      setTimeout(() => {
        this.connect();
      }, this.reconnectInterval * this.reconnectAttempts);
    } else {
      console.error('Max reconnection attempts reached');
    }
  }

  send(message) {
    if (this.ws && this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(message));
    } else {
      console.error('WebSocket is not connected');
    }
  }

  subscribe(eventType, callback) {
    if (!this.listeners.has(eventType)) {
      this.listeners.set(eventType, []);
    }
    this.listeners.get(eventType).push(callback);

    // Return unsubscribe function
    return () => {
      const listeners = this.listeners.get(eventType);
      const index = listeners.indexOf(callback);
      if (index > -1) {
        listeners.splice(index, 1);
      }
    };
  }

  disconnect() {
    if (this.ws) {
      this.ws.close();
      this.ws = null;
    }
    this.listeners.clear();
  }
}

export default new WebSocketService();
```

### **Server-Sent Events (SSE)**
```javascript
// services/sseService.js
import EventSource from 'react-native-sse';

class SSEService {
  constructor() {
    this.eventSource = null;
    this.listeners = new Map();
  }

  async connect() {
    try {
      const token = await AsyncStorage.getItem('authToken');
      const url = `https://api.myapp.com/events?token=${token}`;

      this.eventSource = new EventSource(url, {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });

      this.eventSource.addEventListener('open', this.onOpen.bind(this));
      this.eventSource.addEventListener('message', this.onMessage.bind(this));
      this.eventSource.addEventListener('error', this.onError.bind(this));

      // Listen for specific events
      this.eventSource.addEventListener('notification', this.onNotification.bind(this));
      this.eventSource.addEventListener('update', this.onUpdate.bind(this));

    } catch (error) {
      console.error('SSE connection error:', error);
    }
  }

  onOpen() {
    console.log('SSE connection opened');
  }

  onMessage(event) {
    try {
      const data = JSON.parse(event.data);
      console.log('SSE message:', data);

      // Notify listeners
      const listeners = this.listeners.get(data.type) || [];
      listeners.forEach(callback => callback(data));

    } catch (error) {
      console.error('SSE message parse error:', error);
    }
  }

  onNotification(event) {
    const notification = JSON.parse(event.data);
    console.log('New notification:', notification);

    // Show local notification
    // PushNotification.localNotification({...});
  }

  onUpdate(event) {
    const update = JSON.parse(event.data);
    console.log('Data update:', update);

    // Update local data
    // queryClient.invalidateQueries(['posts']);
  }

  onError(error) {
    console.error('SSE error:', error);
  }

  subscribe(eventType, callback) {
    if (!this.listeners.has(eventType)) {
      this.listeners.set(eventType, []);
    }
    this.listeners.get(eventType).push(callback);

    return () => {
      const listeners = this.listeners.get(eventType);
      const index = listeners.indexOf(callback);
      if (index > -1) {
        listeners.splice(index, 1);
      }
    };
  }

  disconnect() {
    if (this.eventSource) {
      this.eventSource.close();
      this.eventSource = null;
    }
    this.listeners.clear();
  }
}

export default new SSEService();
```

---

## 🧪 **API Testing**

### **API Testing Utilities**
```javascript
// __tests__/api/userService.test.js
import userService from '../../services/userService';
import { ApiError, NetworkError } from '../../services/baseApiService';

// Mock fetch
global.fetch = jest.fn();

describe('UserService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('getProfile', () => {
    test('should return user profile on success', async () => {
      const mockUser = {
        id: 1,
        name: 'John Doe',
        email: 'john@example.com',
      };

      fetch.mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve(mockUser),
        headers: new Headers({ 'content-type': 'application/json' }),
      });

      const result = await userService.getProfile(1);

      expect(fetch).toHaveBeenCalledWith(
        'https://api.myapp.com/users/1',
        expect.objectContaining({
          method: 'GET',
          headers: expect.objectContaining({
            'Content-Type': 'application/json',
          }),
        })
      );

      expect(result).toEqual(mockUser);
    });

    test('should throw ApiError on 404', async () => {
      fetch.mockResolvedValueOnce({
        ok: false,
        status: 404,
        statusText: 'Not Found',
        json: () => Promise.resolve({ message: 'User not found' }),
      });

      await expect(userService.getProfile(999)).rejects.toThrow(ApiError);
    });

    test('should throw NetworkError on network failure', async () => {
      fetch.mockRejectedValueOnce(new TypeError('Failed to fetch'));

      await expect(userService.getProfile(1)).rejects.toThrow(NetworkError);
    });
  });

  describe('updateProfile', () => {
    test('should update user profile successfully', async () => {
      const updateData = { name: 'Jane Doe' };
      const updatedUser = {
        id: 1,
        name: 'Jane Doe',
        email: 'john@example.com',
      };

      fetch.mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve(updatedUser),
      });

      const result = await userService.updateProfile(1, updateData);

      expect(fetch).toHaveBeenCalledWith(
        'https://api.myapp.com/users/1',
        expect.objectContaining({
          method: 'PUT',
          body: JSON.stringify(updateData),
        })
      );

      expect(result).toEqual(updatedUser);
    });
  });
});

// __tests__/api/authService.test.js
import authService from '../../services/authService';

// Mock AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () => ({
  getItem: jest.fn(),
  setItem: jest.fn(),
  multiSet: jest.fn(),
  multiRemove: jest.fn(),
}));

const AsyncStorage = require('@react-native-async-storage/async-storage');

describe('AuthService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('login', () => {
    test('should login successfully and store tokens', async () => {
      const credentials = { email: 'test@example.com', password: 'password' };
      const mockResponse = {
        data: {
          token: 'jwt-token',
          refreshToken: 'refresh-token',
          user: { id: 1, email: 'test@example.com' },
        },
      };

      global.fetch = jest.fn().mockResolvedValue({
        ok: true,
        json: () => Promise.resolve(mockResponse),
      });

      const result = await authService.login(credentials);

      expect(fetch).toHaveBeenCalledWith('/api/auth/login', expect.any(Object));
      expect(AsyncStorage.multiSet).toHaveBeenCalledWith([
        ['authToken', 'jwt-token'],
        ['refreshToken', 'refresh-token'],
        ['userData', JSON.stringify({ id: 1, email: 'test@example.com' })],
      ]);
      expect(result).toEqual({
        token: 'jwt-token',
        user: { id: 1, email: 'test@example.com' },
      });
    });

    test('should handle login failure', async () => {
      global.fetch = jest.fn().mockResolvedValue({
        ok: false,
        json: () => Promise.resolve({ message: 'Invalid credentials' }),
      });

      await expect(
        authService.login({ email: 'wrong@example.com', password: 'wrong' })
      ).rejects.toThrow('Invalid credentials');

      expect(AsyncStorage.multiSet).not.toHaveBeenCalled();
    });
  });

  describe('logout', () => {
    test('should logout and clear storage', async () => {
      global.fetch = jest.fn().mockResolvedValue({ ok: true });

      await authService.logout();

      expect(fetch).toHaveBeenCalledWith('/api/auth/logout', expect.any(Object));
      expect(AsyncStorage.multiRemove).toHaveBeenCalledWith([
        'authToken',
        'refreshToken',
        'userData',
      ]);
    });
  });
});
```

---

## 🎯 **Practical Examples**

### **Complete API Integration Example**
```javascript
// App.js
import React from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';

// Contexts
import { AuthProvider } from './contexts/AuthContext';
import { ThemeProvider } from './contexts/ThemeContext';

// Services
import { setupInterceptors } from './services/interceptors';
import apiClient from './services/apiClient';

// Screens
import LoginScreen from './screens/LoginScreen';
import HomeScreen from './screens/HomeScreen';
import ProfileScreen from './screens/ProfileScreen';

// Setup API interceptors
setupInterceptors(apiClient);

// Query client
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      cacheTime: 1000 * 60 * 10, // 10 minutes
      retry: (failureCount, error) => {
        // Don't retry on 4xx errors
        if (error?.status >= 400 && error?.status < 500) {
          return false;
        }
        return failureCount < 3;
      },
      refetchOnWindowFocus: false,
    },
    mutations: {
      retry: 1,
    },
  },
});

const Stack = createStackNavigator();

const App = () => {
  return (
    <QueryClientProvider client={queryClient}>
      <AuthProvider>
        <ThemeProvider>
          <NavigationContainer>
            <Stack.Navigator
              initialRouteName="Login"
              screenOptions={{
                headerStyle: {
                  backgroundColor: '#007AFF',
                },
                headerTintColor: '#fff',
                headerTitleStyle: {
                  fontWeight: 'bold',
                },
              }}
            >
              <Stack.Screen
                name="Login"
                component={LoginScreen}
                options={{ headerShown: false }}
              />
              <Stack.Screen
                name="Home"
                component={HomeScreen}
                options={{ title: 'My App' }}
              />
              <Stack.Screen
                name="Profile"
                component={ProfileScreen}
                options={{ title: 'Profile' }}
              />
            </Stack.Navigator>
          </NavigationContainer>
        </ThemeProvider>
      </AuthProvider>
      {__DEV__ && <ReactQueryDevtools initialIsOpen={false} />}
    </QueryClientProvider>
  );
};

export default App;

// screens/LoginScreen.js
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  Alert,
  ActivityIndicator,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { useAuth } from '../contexts/AuthContext';

const LoginScreen = () => {
  const navigation = useNavigation();
  const { login, isLoading } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = async () => {
    if (!email || !password) {
      Alert.alert('Error', 'Please fill in all fields');
      return;
    }

    try {
      await login(email, password);
      navigation.replace('Home');
    } catch (error) {
      Alert.alert('Login Failed', error.message);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome Back</Text>
      <Text style={styles.subtitle}>Sign in to your account</Text>

      <View style={styles.form}>
        <TextInput
          style={styles.input}
          placeholder="Email"
          value={email}
          onChangeText={setEmail}
          keyboardType="email-address"
          autoCapitalize="none"
        />

        <TextInput
          style={styles.input}
          placeholder="Password"
          value={password}
          onChangeText={setPassword}
          secureTextEntry
        />

        <TouchableOpacity
          style={[styles.button, isLoading && styles.buttonDisabled]}
          onPress={handleLogin}
          disabled={isLoading}
        >
          {isLoading ? (
            <ActivityIndicator color="#fff" />
          ) : (
            <Text style={styles.buttonText}>Sign In</Text>
          )}
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    justifyContent: 'center',
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 8,
    color: '#333',
  },
  subtitle: {
    fontSize: 16,
    textAlign: 'center',
    marginBottom: 40,
    color: '#666',
  },
  form: {
    backgroundColor: '#fff',
    padding: 20,
    borderRadius: 12,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 15,
    marginBottom: 15,
    fontSize: 16,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  buttonDisabled: {
    backgroundColor: '#ccc',
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default LoginScreen;

// screens/HomeScreen.js
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet, FlatList } from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { useQuery } from '@tanstack/react-query';
import { useAuth } from '../contexts/AuthContext';
import userService from '../services/userService';

const HomeScreen = () => {
  const navigation = useNavigation();
  const { user, logout } = useAuth();

  // Fetch user posts using React Query
  const {
    data: posts,
    isLoading,
    error,
    refetch,
  } = useQuery({
    queryKey: ['posts', user?.id],
    queryFn: () => userService.getPosts(user.id),
    enabled: !!user?.id,
  });

  const handleLogout = async () => {
    await logout();
    navigation.replace('Login');
  };

  const renderPost = ({ item }) => (
    <View style={styles.postCard}>
      <Text style={styles.postTitle}>{item.title}</Text>
      <Text style={styles.postContent} numberOfLines={3}>
        {item.content}
      </Text>
      <Text style={styles.postDate}>
        {new Date(item.createdAt).toLocaleDateString()}
      </Text>
    </View>
  );

  if (isLoading) {
    return (
      <View style={styles.center}>
        <Text>Loading posts...</Text>
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.center}>
        <Text style={styles.errorText}>Error loading posts</Text>
        <TouchableOpacity style={styles.retryButton} onPress={refetch}>
          <Text style={styles.retryButtonText}>Retry</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.welcome}>Welcome, {user?.name}!</Text>
        <TouchableOpacity style={styles.profileButton} onPress={() => navigation.navigate('Profile')}>
          <Text style={styles.profileButtonText}>Profile</Text>
        </TouchableOpacity>
      </View>

      <Text style={styles.sectionTitle}>Your Posts</Text>

      <FlatList
        data={posts}
        renderItem={renderPost}
        keyExtractor={(item) => item.id.toString()}
        style={styles.postsList}
        ListEmptyComponent={
          <View style={styles.emptyState}>
            <Text style={styles.emptyText}>No posts yet</Text>
            <Text style={styles.emptySubtext}>Create your first post!</Text>
          </View>
        }
      />

      <TouchableOpacity style={styles.logoutButton} onPress={handleLogout}>
        <Text style={styles.logoutButtonText}>Logout</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 20,
  },
  welcome: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#333',
  },
  profileButton: {
    backgroundColor: '#007AFF',
    padding: 8,
    borderRadius: 6,
  },
  profileButtonText: {
    color: '#fff',
    fontSize: 14,
    fontWeight: 'bold',
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  postsList: {
    flex: 1,
  },
  postCard: {
    backgroundColor: '#fff',
    padding: 15,
    marginBottom: 10,
    borderRadius: 8,
  },
  postTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 8,
    color: '#333',
  },
  postContent: {
    fontSize: 14,
    color: '#666',
    marginBottom: 8,
    lineHeight: 20,
  },
  postDate: {
    fontSize: 12,
    color: '#999',
  },
  center: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  errorText: {
    color: '#e74c3c',
    marginBottom: 15,
  },
  retryButton: {
    backgroundColor: '#007AFF',
    padding: 10,
    borderRadius: 6,
  },
  retryButtonText: {
    color: '#fff',
    fontSize: 14,
    fontWeight: 'bold',
  },
  emptyState: {
    alignItems: 'center',
    padding: 40,
  },
  emptyText: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#666',
    marginBottom: 8,
  },
  emptySubtext: {
    fontSize: 14,
    color: '#999',
  },
  logoutButton: {
    backgroundColor: '#e74c3c',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 20,
  },
  logoutButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default HomeScreen;

// screens/ProfileScreen.js
import React, { useState } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
  Alert,
  ActivityIndicator,
  ScrollView,
} from 'react-native';
import { useNavigation } from '@react-navigation/native';
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { useAuth } from '../contexts/AuthContext';
import userService from '../services/userService';

const ProfileScreen = () => {
  const navigation = useNavigation();
  const { user, logout } = useAuth();
  const queryClient = useQueryClient();
  const [isEditing, setIsEditing] = useState(false);
  const [editData, setEditData] = useState({
    name: user?.name || '',
    email: user?.email || '',
  });

  // Update profile mutation
  const updateMutation = useMutation({
    mutationFn: (data) => userService.updateProfile(user.id, data),
    onSuccess: (updatedUser) => {
      queryClient.invalidateQueries(['user', user.id]);
      Alert.alert('Success', 'Profile updated successfully');
      setIsEditing(false);
    },
    onError: (error) => {
      Alert.alert('Error', error.message || 'Failed to update profile');
    },
  });

  const handleSave = () => {
    if (!editData.name.trim() || !editData.email.trim()) {
      Alert.alert('Error', 'Name and email are required');
      return;
    }

    updateMutation.mutate(editData);
  };

  const handleLogout = async () => {
    Alert.alert(
      'Logout',
      'Are you sure you want to logout?',
      [
        { text: 'Cancel', style: 'cancel' },
        {
          text: 'Logout',
          style: 'destructive',
          onPress: async () => {
            await logout();
            navigation.replace('Login');
          },
        },
      ]
    );
  };

  if (!user) {
    return (
      <View style={styles.center}>
        <Text>Loading profile...</Text>
      </View>
    );
  }

  return (
    <ScrollView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Profile</Text>
        <TouchableOpacity
          style={styles.editButton}
          onPress={() => setIsEditing(!isEditing)}
        >
          <Text style={styles.editButtonText}>
            {isEditing ? 'Cancel' : 'Edit'}
          </Text>
        </TouchableOpacity>
      </View>

      <View style={styles.profileCard}>
        <View style={styles.avatar}>
          <Text style={styles.avatarText}>
            {user.name?.charAt(0)?.toUpperCase() || 'U'}
          </Text>
        </View>

        <Text style={styles.name}>{user.name}</Text>
        <Text style={styles.email}>{user.email}</Text>
        <Text style={styles.role}>{user.role || 'User'}</Text>
      </View>

      {isEditing && (
        <View style={styles.editForm}>
          <TextInput
            style={styles.input}
            placeholder="Name"
            value={editData.name}
            onChangeText={(value) => setEditData(prev => ({ ...prev, name: value }))}
          />

          <TextInput
            style={styles.input}
            placeholder="Email"
            value={editData.email}
            onChangeText={(value) => setEditData(prev => ({ ...prev, email: value }))}
            keyboardType="email-address"
            autoCapitalize="none"
          />

          <TouchableOpacity
            style={[styles.saveButton, updateMutation.isLoading && styles.buttonDisabled]}
            onPress={handleSave}
            disabled={updateMutation.isLoading}
          >
            {updateMutation.isLoading ? (
              <ActivityIndicator color="#fff" />
            ) : (
              <Text style={styles.saveButtonText}>Save Changes</Text>
            )}
          </TouchableOpacity>
        </View>
      )}

      <View style={styles.actions}>
        <TouchableOpacity style={styles.actionButton} onPress={() => navigation.goBack()}>
          <Text style={styles.actionButtonText}>Back to Home</Text>
        </TouchableOpacity>

        <TouchableOpacity style={styles.logoutButton} onPress={handleLogout}>
          <Text style={styles.logoutButtonText}>Logout</Text>
        </TouchableOpacity>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  center: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
  },
  editButton: {
    backgroundColor: '#007AFF',
    padding: 8,
    borderRadius: 6,
  },
  editButtonText: {
    color: '#fff',
    fontSize: 14,
    fontWeight: 'bold',
  },
  profileCard: {
    backgroundColor: '#fff',
    margin: 20,
    padding: 20,
    borderRadius: 12,
    alignItems: 'center',
  },
  avatar: {
    width: 80,
    height: 80,
    borderRadius: 40,
    backgroundColor: '#007AFF',
    justifyContent: 'center',
    alignItems: 'center',
    marginBottom: 15,
  },
  avatarText: {
    fontSize: 32,
    fontWeight: 'bold',
    color: '#fff',
  },
  name: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 8,
  },
  email: {
    fontSize: 16,
    color: '#666',
    marginBottom: 8,
  },
  role: {
    fontSize: 14,
    color: '#007AFF',
    fontWeight: 'bold',
  },
  editForm: {
    backgroundColor: '#fff',
    margin: 20,
    padding: 20,
    borderRadius: 12,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 15,
    marginBottom: 15,
    fontSize: 16,
    backgroundColor: '#fafafa',
  },
  saveButton: {
    backgroundColor: '#28a745',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  buttonDisabled: {
    backgroundColor: '#ccc',
  },
  saveButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
  actions: {
    padding: 20,
    gap: 10,
  },
  actionButton: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  actionButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
  logoutButton: {
    backgroundColor: '#e74c3c',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  logoutButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default ProfileScreen;

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **API Integration Fundamentals**: REST principles, HTTP methods, status codes
- ✅ **HTTP Client Setup**: Axios and Fetch API configuration with interceptors
- ✅ **Authentication & Authorization**: JWT token management, OAuth integration
- ✅ **API Service Layer**: Base service class, specific service implementations
- ✅ **Error Handling**: Global error handler, retry mechanisms, error boundaries
- ✅ **Request/Response Interceptors**: Advanced interceptors for logging and auth
- ✅ **Caching Strategies**: HTTP caching, cache-aware API client
- ✅ **Real-time Communication**: WebSocket and SSE integration
- ✅ **API Testing**: Comprehensive testing utilities and examples
- ✅ **Practical Implementation**: Complete app with authentication, API calls, and React Query

### **Best Practices**
1. **Use consistent API patterns** - Follow REST conventions and proper HTTP methods
2. **Implement proper error handling** - Handle different error types appropriately
3. **Add request/response interceptors** - For logging, authentication, and error handling
4. **Implement caching** - Use appropriate caching strategies for better performance
5. **Handle authentication properly** - JWT tokens, refresh tokens, and secure storage
6. **Use service layers** - Separate API logic from components
7. **Implement retry mechanisms** - For network failures and server errors
8. **Test API integrations** - Unit tests and integration tests
9. **Monitor API performance** - Track response times and error rates
10. **Handle offline scenarios** - Cache data and sync when online

### **Next Steps**
- Learn about GraphQL and Apollo Client
- Implement API rate limiting and throttling
- Set up API monitoring and analytics
- Learn about API gateways and microservices
- Explore serverless functions and edge computing
- Study API security best practices
- Implement API versioning strategies
- Learn about real-time databases (Firebase, Supabase)

---

## 🎯 **Assignment**

### **Task 1: Complete Social Media API**
Build a comprehensive social media API integration with:
- User authentication and profile management
- Posts creation, editing, and deletion
- Like/unlike functionality with real-time updates
- Comments system with nested replies
- Follow/unfollow users with activity feeds
- Image upload and media management
- Real-time notifications using WebSocket
- Offline support with data synchronization

### **Task 2: E-commerce API Integration**
Create a complete e-commerce API system including:
- Product catalog with search and filtering
- Shopping cart management with persistence
- Order processing and payment integration
- User reviews and ratings system
- Inventory management and stock tracking
- Wishlist functionality
- Order history and tracking
- Admin dashboard for product management

### **Task 3: Real-time Chat Application**
Implement a real-time chat application with:
- User authentication and online status
- One-on-one and group chat functionality
- Message encryption and security
- File sharing and media messages
- Typing indicators and read receipts
- Message search and history
- Push notifications for new messages
- Offline message queuing

### **Task 4: Advanced API Patterns**
Create advanced API integration patterns including:
- GraphQL integration with Apollo Client
- API composition and aggregation
- Request batching and deduplication
- Advanced caching strategies
- API mocking and testing utilities
- Performance monitoring and analytics
- Error tracking and reporting
- API documentation generation

---

## 📚 **Additional Resources**
- [Axios Documentation](https://axios-http.com/docs/intro)
- [React Query Documentation](https://tanstack.com/query/latest)
- [JWT Authentication](https://jwt.io/)
- [WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [REST API Design Best Practices](https://restfulapi.net/)
- [API Testing with Jest](https://jestjs.io/docs/asynchronous)

---

**Next Lesson**: [Lesson 38: Advanced Navigation Patterns](Lesson%2038_%20Advanced%20Navigation%20Patterns.md)