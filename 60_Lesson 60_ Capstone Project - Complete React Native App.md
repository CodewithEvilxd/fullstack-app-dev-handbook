# Lesson 60: Capstone Project - Complete React Native App

## 🎯 **Learning Objectives**
- Apply all learned concepts in a comprehensive React Native application
- Build a production-ready app with advanced features
- Implement best practices for code organization and architecture
- Create a scalable and maintainable codebase
- Deploy the application to app stores

## 📚 **Table of Contents**
1. [Project Overview](#project-overview)
2. [Architecture Design](#architecture-design)
3. [Core Features Implementation](#core-features-implementation)
4. [Advanced Features](#advanced-features)
5. [Testing Strategy](#testing-strategy)
6. [Deployment Preparation](#deployment-preparation)
7. [Performance Optimization](#performance-optimization)
8. [Documentation](#documentation)
9. [Project Showcase](#project-showcase)

---

## 📱 **Project Overview**

### **App Concept: SocialConnect**
A comprehensive social networking application that combines multiple features learned throughout the course:

**Core Features:**
- User authentication and profiles
- Real-time messaging and notifications
- Photo/video sharing with camera integration
- Location-based content discovery
- Offline support and data synchronization
- Push notifications and alerts

**Advanced Features:**
- AI-powered content recommendations
- Live streaming capabilities
- E-commerce integration
- Multi-language support
- Dark mode and accessibility
- Advanced analytics and reporting

### **Technical Stack**
```
Frontend: React Native + TypeScript
Backend: Node.js + Express + Socket.io
Database: PostgreSQL + Redis
Authentication: JWT + OAuth
File Storage: AWS S3
Real-time: Socket.io
Deployment: Docker + Kubernetes
Monitoring: DataDog + Sentry
```

---

## 🏗️ **Architecture Design**

### **Project Structure**
```
SocialConnect/
├── android/                    # Android native code
├── ios/                       # iOS native code
├── src/
│   ├── components/            # Reusable UI components
│   │   ├── common/           # Generic components
│   │   ├── forms/            # Form components
│   │   ├── media/            # Media components
│   │   └── navigation/       # Navigation components
│   ├── screens/              # Screen components
│   │   ├── auth/            # Authentication screens
│   │   ├── home/            # Home feed screens
│   │   ├── profile/         # Profile screens
│   │   ├── messaging/       # Chat screens
│   │   └── settings/        # Settings screens
│   ├── services/            # Business logic services
│   │   ├── api/            # API services
│   │   ├── auth/           # Authentication services
│   │   ├── storage/        # Data storage services
│   │   ├── notifications/  # Notification services
│   │   └── analytics/      # Analytics services
│   ├── utils/              # Utility functions
│   │   ├── constants/     # App constants
│   │   ├── helpers/       # Helper functions
│   │   ├── validators/    # Validation functions
│   │   └── formatters/    # Data formatters
│   ├── hooks/             # Custom React hooks
│   ├── contexts/          # React contexts
│   ├── types/            # TypeScript type definitions
│   ├── assets/           # Images, fonts, icons
│   └── config/           # Configuration files
├── __tests__/            # Test files
├── e2e/                 # End-to-end tests
├── docs/               # Documentation
├── scripts/           # Build and deployment scripts
└── docker/            # Docker configuration
```

### **State Management Architecture**
```typescript
// src/contexts/AppContext.tsx
import React, { createContext, useContext, useReducer, ReactNode } from 'react';

interface AppState {
  user: User | null;
  isAuthenticated: boolean;
  isOnline: boolean;
  theme: 'light' | 'dark';
  language: string;
  notifications: Notification[];
  loading: boolean;
  error: string | null;
}

type AppAction =
  | { type: 'SET_USER'; payload: User }
  | { type: 'SET_AUTHENTICATED'; payload: boolean }
  | { type: 'SET_ONLINE'; payload: boolean }
  | { type: 'SET_THEME'; payload: 'light' | 'dark' }
  | { type: 'SET_LANGUAGE'; payload: string }
  | { type: 'ADD_NOTIFICATION'; payload: Notification }
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'SET_ERROR'; payload: string | null };

const initialState: AppState = {
  user: null,
  isAuthenticated: false,
  isOnline: true,
  theme: 'light',
  language: 'en',
  notifications: [],
  loading: false,
  error: null,
};

function appReducer(state: AppState, action: AppAction): AppState {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, user: action.payload };
    case 'SET_AUTHENTICATED':
      return { ...state, isAuthenticated: action.payload };
    case 'SET_ONLINE':
      return { ...state, isOnline: action.payload };
    case 'SET_THEME':
      return { ...state, theme: action.payload };
    case 'SET_LANGUAGE':
      return { ...state, language: action.payload };
    case 'ADD_NOTIFICATION':
      return { ...state, notifications: [...state.notifications, action.payload] };
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    case 'SET_ERROR':
      return { ...state, error: action.payload };
    default:
      return state;
  }
}

const AppContext = createContext<{
  state: AppState;
  dispatch: React.Dispatch<AppAction>;
} | null>(null);

export const AppProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [state, dispatch] = useReducer(appReducer, initialState);

  return (
    <AppContext.Provider value={{ state, dispatch }}>
      {children}
    </AppContext.Provider>
  );
};

export const useApp = () => {
  const context = useContext(AppContext);
  if (!context) {
    throw new Error('useApp must be used within an AppProvider');
  }
  return context;
};
```

---

## 🚀 **Core Features Implementation**

### **Authentication System**
```typescript
// src/services/auth/AuthService.ts
import AsyncStorage from '@react-native-async-storage/async-storage';
import { apiClient } from '../api/ApiClient';
import { secureStorage } from '../storage/SecureStorage';
import { biometricService } from '../biometric/BiometricService';

export interface User {
  id: string;
  email: string;
  username: string;
  firstName: string;
  lastName: string;
  avatar?: string;
  bio?: string;
  verified: boolean;
  createdAt: string;
  updatedAt: string;
}

export interface LoginCredentials {
  email: string;
  password: string;
  rememberMe?: boolean;
}

export interface RegisterData {
  email: string;
  username: string;
  password: string;
  firstName: string;
  lastName: string;
}

class AuthService {
  private readonly TOKEN_KEY = 'auth_tokens';
  private readonly USER_KEY = 'user_data';
  private refreshPromise: Promise<any> | null = null;

  // Login user
  async login(credentials: LoginCredentials): Promise<{ user: User; tokens: any }> {
    try {
      const response = await apiClient.post('/auth/login', credentials);
      const { user, accessToken, refreshToken } = response.data;

      // Store tokens securely
      await secureStorage.storeAuthTokens(accessToken, refreshToken);

      // Store user data
      await AsyncStorage.setItem(this.USER_KEY, JSON.stringify(user));

      return { user, tokens: { accessToken, refreshToken } };
    } catch (error) {
      throw new Error('Login failed. Please check your credentials.');
    }
  }

  // Register new user
  async register(data: RegisterData): Promise<{ user: User; tokens: any }> {
    try {
      const response = await apiClient.post('/auth/register', data);
      const { user, accessToken, refreshToken } = response.data;

      // Store tokens securely
      await secureStorage.storeAuthTokens(accessToken, refreshToken);

      // Store user data
      await AsyncStorage.setItem(this.USER_KEY, JSON.stringify(user));

      return { user, tokens: { accessToken, refreshToken } };
    } catch (error) {
      if (error.response?.status === 409) {
        throw new Error('User with this email or username already exists.');
      }
      throw new Error('Registration failed. Please try again.');
    }
  }

  // Logout user
  async logout(): Promise<void> {
    try {
      const tokens = await secureStorage.getAuthTokens();
      if (tokens.success) {
        await apiClient.post('/auth/logout', {
          refreshToken: tokens.data.refreshToken
        });
      }
    } catch (error) {
      console.error('Logout API call failed:', error);
    } finally {
      // Clear local data
      await secureStorage.removeSecureData(this.TOKEN_KEY);
      await AsyncStorage.removeItem(this.USER_KEY);
    }
  }

  // Get current user
  async getCurrentUser(): Promise<User | null> {
    try {
      const userData = await AsyncStorage.getItem(this.USER_KEY);
      return userData ? JSON.parse(userData) : null;
    } catch (error) {
      console.error('Failed to get current user:', error);
      return null;
    }
  }

  // Refresh access token
  async refreshToken(): Promise<any> {
    if (this.refreshPromise) {
      return this.refreshPromise;
    }

    this.refreshPromise = this.performTokenRefresh();

    try {
      return await this.refreshPromise;
    } finally {
      this.refreshPromise = null;
    }
  }

  private async performTokenRefresh(): Promise<any> {
    try {
      const tokens = await secureStorage.getAuthTokens();
      if (!tokens.success) {
        throw new Error('No refresh token available');
      }

      const response = await apiClient.post('/auth/refresh', {
        refreshToken: tokens.data.refreshToken
      });

      const { accessToken, refreshToken: newRefreshToken } = response.data;

      // Store new tokens
      await secureStorage.storeAuthTokens(accessToken, newRefreshToken);

      return { accessToken, refreshToken: newRefreshToken };
    } catch (error) {
      // Clear invalid tokens
      await secureStorage.removeSecureData(this.TOKEN_KEY);
      throw error;
    }
  }

  // Check if user is authenticated
  async isAuthenticated(): Promise<boolean> {
    const tokens = await secureStorage.getAuthTokens();
    if (!tokens.success) return false;

    // Check if token is expired
    try {
      const payload = JSON.parse(atob(tokens.data.accessToken.split('.')[1]));
      const currentTime = Date.now() / 1000;
      return payload.exp > currentTime;
    } catch (error) {
      return false;
    }
  }

  // Enable biometric authentication
  async enableBiometric(password: string): Promise<void> {
    const user = await this.getCurrentUser();
    if (!user) throw new Error('No user found');

    await biometricService.storeBiometricCredentials(user.email, password);
  }

  // Authenticate with biometrics
  async authenticateWithBiometrics(): Promise<User> {
    const result = await biometricService.authenticateWithBiometrics();
    if (!result.success) {
      throw new Error(result.error || 'Biometric authentication failed');
    }

    // Verify credentials with server
    const loginResult = await this.login({
      email: result.username,
      password: result.password
    });

    return loginResult.user;
  }

  // Change password
  async changePassword(currentPassword: string, newPassword: string): Promise<void> {
    try {
      await apiClient.post('/auth/change-password', {
        currentPassword,
        newPassword
      });

      // Update biometric credentials if enabled
      const user = await this.getCurrentUser();
      if (user && await biometricService.authenticateWithBiometrics()) {
        await biometricService.storeBiometricCredentials(user.email, newPassword);
      }
    } catch (error) {
      throw new Error('Failed to change password');
    }
  }

  // Reset password
  async resetPassword(email: string): Promise<void> {
    try {
      await apiClient.post('/auth/reset-password', { email });
    } catch (error) {
      throw new Error('Failed to send password reset email');
    }
  }

  // Verify email
  async verifyEmail(token: string): Promise<void> {
    try {
      await apiClient.post('/auth/verify-email', { token });
    } catch (error) {
      throw new Error('Email verification failed');
    }
  }
}

export const authService = new AuthService();
```

### **Real-time Messaging System**
```typescript
// src/services/messaging/MessagingService.ts
import { io, Socket } from 'socket.io-client';
import { apiClient } from '../api/ApiClient';
import AsyncStorage from '@react-native-async-storage/async-storage';

export interface Message {
  id: string;
  senderId: string;
  receiverId: string;
  content: string;
  type: 'text' | 'image' | 'video' | 'file';
  metadata?: any;
  timestamp: string;
  read: boolean;
  delivered: boolean;
}

export interface Conversation {
  id: string;
  participants: string[];
  lastMessage?: Message;
  unreadCount: number;
  updatedAt: string;
}

class MessagingService {
  private socket: Socket | null = null;
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 5;
  private messageListeners: Map<string, Function> = new Map();
  private typingListeners: Map<string, Function> = new Map();

  // Initialize messaging service
  async initialize(userId: string): Promise<void> {
    try {
      const token = await this.getAuthToken();

      this.socket = io('wss://api.socialconnect.com', {
        auth: { token, userId },
        transports: ['websocket'],
        timeout: 5000,
        reconnection: true,
        reconnectionAttempts: this.maxReconnectAttempts,
        reconnectionDelay: 1000,
      });

      this.setupSocketListeners();
    } catch (error) {
      console.error('Failed to initialize messaging service:', error);
      throw error;
    }
  }

  // Setup socket event listeners
  private setupSocketListeners(): void {
    if (!this.socket) return;

    this.socket.on('connect', () => {
      console.log('Connected to messaging server');
      this.reconnectAttempts = 0;
    });

    this.socket.on('disconnect', (reason) => {
      console.log('Disconnected from messaging server:', reason);
    });

    this.socket.on('connect_error', (error) => {
      console.error('Connection error:', error);
      this.reconnectAttempts++;

      if (this.reconnectAttempts >= this.maxReconnectAttempts) {
        console.error('Max reconnection attempts reached');
      }
    });

    this.socket.on('message', (message: Message) => {
      this.handleIncomingMessage(message);
    });

    this.socket.on('typing', (data: { userId: string; conversationId: string }) => {
      this.handleTypingIndicator(data);
    });

    this.socket.on('message_read', (data: { messageId: string; userId: string }) => {
      this.handleMessageRead(data);
    });
  }

  // Send message
  async sendMessage(conversationId: string, content: string, type: Message['type'] = 'text'): Promise<Message> {
    try {
      const messageData = {
        conversationId,
        content,
        type,
        timestamp: new Date().toISOString()
      };

      const response = await apiClient.post('/messages', messageData);
      const message: Message = response.data;

      // Emit via socket for real-time delivery
      if (this.socket) {
        this.socket.emit('message', message);
      }

      return message;
    } catch (error) {
      console.error('Failed to send message:', error);
      throw new Error('Failed to send message');
    }
  }

  // Send file message
  async sendFileMessage(conversationId: string, fileUri: string, fileType: string): Promise<Message> {
    try {
      // Upload file first
      const uploadResponse = await apiClient.uploadFile('/upload', fileUri, 'file');
      const fileUrl = uploadResponse.data.url;

      // Send message with file URL
      return await this.sendMessage(conversationId, fileUrl, fileType as Message['type']);
    } catch (error) {
      console.error('Failed to send file message:', error);
      throw new Error('Failed to send file');
    }
  }

  // Get conversation messages
  async getConversationMessages(conversationId: string, page: number = 1, limit: number = 50): Promise<Message[]> {
    try {
      const response = await apiClient.get(`/conversations/${conversationId}/messages`, {
        params: { page, limit }
      });
      return response.data.messages;
    } catch (error) {
      console.error('Failed to get conversation messages:', error);
      return [];
    }
  }

  // Get user conversations
  async getUserConversations(): Promise<Conversation[]> {
    try {
      const response = await apiClient.get('/conversations');
      return response.data.conversations;
    } catch (error) {
      console.error('Failed to get user conversations:', error);
      return [];
    }
  }

  // Create new conversation
  async createConversation(participantIds: string[]): Promise<Conversation> {
    try {
      const response = await apiClient.post('/conversations', {
        participantIds
      });
      return response.data.conversation;
    } catch (error) {
      console.error('Failed to create conversation:', error);
      throw new Error('Failed to create conversation');
    }
  }

  // Mark messages as read
  async markMessagesAsRead(conversationId: string, messageIds: string[]): Promise<void> {
    try {
      await apiClient.post(`/conversations/${conversationId}/read`, {
        messageIds
      });

      // Emit via socket
      if (this.socket) {
        this.socket.emit('mark_read', { conversationId, messageIds });
      }
    } catch (error) {
      console.error('Failed to mark messages as read:', error);
    }
  }

  // Send typing indicator
  sendTypingIndicator(conversationId: string, isTyping: boolean): void {
    if (this.socket) {
      this.socket.emit('typing', { conversationId, isTyping });
    }
  }

  // Add message listener
  addMessageListener(conversationId: string, listener: Function): void {
    this.messageListeners.set(conversationId, listener);
  }

  // Remove message listener
  removeMessageListener(conversationId: string): void {
    this.messageListeners.delete(conversationId);
  }

  // Add typing listener
  addTypingListener(conversationId: string, listener: Function): void {
    this.typingListeners.set(conversationId, listener);
  }

  // Remove typing listener
  removeTypingListener(conversationId: string): void {
    this.typingListeners.delete(conversationId);
  }

  // Handle incoming message
  private handleIncomingMessage(message: Message): void {
    // Store message locally
    this.storeMessageLocally(message);

    // Notify listeners
    const listener = this.messageListeners.get(message.conversationId);
    if (listener) {
      listener(message);
    }

    // Show notification if app is in background
    this.showMessageNotification(message);
  }

  // Handle typing indicator
  private handleTypingIndicator(data: { userId: string; conversationId: string }): void {
    const listener = this.typingListeners.get(data.conversationId);
    if (listener) {
      listener(data);
    }
  }

  // Handle message read
  private handleMessageRead(data: { messageId: string; userId: string }): void {
    // Update local message status
    this.updateMessageReadStatus(data.messageId, data.userId);
  }

  // Store message locally for offline access
  private async storeMessageLocally(message: Message): Promise<void> {
    try {
      const key = `messages_${message.conversationId}`;
      const stored = await AsyncStorage.getItem(key);
      const messages = stored ? JSON.parse(stored) : [];

      messages.push(message);

      // Keep only last 100 messages per conversation
      if (messages.length > 100) {
        messages.splice(0, messages.length - 100);
      }

      await AsyncStorage.setItem(key, JSON.stringify(messages));
    } catch (error) {
      console.error('Failed to store message locally:', error);
    }
  }

  // Show message notification
  private showMessageNotification(message: Message): void {
    // Implementation depends on notification service
    // This would integrate with the notification system
    console.log('Showing message notification:', message);
  }

  // Update message read status
  private async updateMessageReadStatus(messageId: string, userId: string): Promise<void> {
    // Update local message status
    console.log('Updating message read status:', messageId, userId);
  }

  // Get authentication token
  private async getAuthToken(): Promise<string> {
    // Get token from secure storage
    const tokens = await secureStorage.getAuthTokens();
    if (!tokens.success) {
      throw new Error('No authentication token available');
    }
    return tokens.data.accessToken;
  }

  // Disconnect from messaging server
  disconnect(): void {
    if (this.socket) {
      this.socket.disconnect();
      this.socket = null;
    }
  }

  // Check connection status
  isConnected(): boolean {
    return this.socket?.connected || false;
  }

  // Reconnect to messaging server
  reconnect(): void {
    if (this.socket && !this.socket.connected) {
      this.socket.connect();
    }
  }
}

export const messagingService = new MessagingService();
```

---

## 🧪 **Testing Strategy**

### **Unit Testing Setup**
```typescript
// __tests__/services/AuthService.test.ts
import { authService } from '../../src/services/auth/AuthService';
import { apiClient } from '../../src/services/api/ApiClient';

// Mock dependencies
jest.mock('../../src/services/api/ApiClient');
jest.mock('../../src/services/storage/SecureStorage');

const mockApiClient = apiClient as jest.Mocked<typeof apiClient>;
const mockSecureStorage = {
  storeAuthTokens: jest.fn(),
  getAuthTokens: jest.fn(),
  removeSecureData: jest.fn(),
};

describe('AuthService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('login', () => {
    it('should login user successfully', async () => {
      const mockResponse = {
        data: {
          user: { id: '1', email: 'test@example.com' },
          accessToken: 'access-token',
          refreshToken: 'refresh-token'
        }
      };

      mockApiClient.post.mockResolvedValue(mockResponse);
      mockSecureStorage.storeAuthTokens.mockResolvedValue({ success: true });

      const result = await authService.login({
        email: 'test@example.com',
        password: 'password'
      });

      expect(result.user.email).toBe('test@example.com');
      expect(mockApiClient.post).toHaveBeenCalledWith('/auth/login', {
        email: 'test@example.com',
        password: 'password'
      });
      expect(mockSecureStorage.storeAuthTokens).toHaveBeenCalledWith(
        'access-token',
        'refresh-token'
      );
    });

    it('should handle login failure', async () => {
      mockApiClient.post.mockRejectedValue(new Error('Invalid credentials'));

      await expect(authService.login({
        email: 'test@example.com',
        password: 'wrong-password'
      })).rejects.toThrow('Login failed. Please check your credentials.');
    });
  });

  describe('isAuthenticated', () => {
    it('should return true for valid token', async () => {
      const validToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxIiwiaWF0IjoxNjE2MjM5MDIyLCJleHAiOjE2MTYyNDI2MjJ9.example';
      mockSecureStorage.getAuthTokens.mockResolvedValue({
        success: true,
        data: { accessToken: validToken }
      });

      const result = await authService.isAuthenticated();
      expect(result).toBe(true);
    });

    it('should return false for expired token', async () => {
      const expiredToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxIiwiaWF0IjoxNTE2MjM5MDIyLCJleHAiOjE1MTYyMzkwMjJ9.example';
      mockSecureStorage.getAuthTokens.mockResolvedValue({
        success: true,
        data: { accessToken: expiredToken }
      });

      const result = await authService.isAuthenticated();
      expect(result).toBe(false);
    });
  });
});
```

### **Integration Testing**
```typescript
// __tests__/screens/LoginScreen.test.tsx
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { LoginScreen } from '../../src/screens/auth/LoginScreen';
import { authService } from '../../src/services/auth/AuthService';

// Mock navigation
const mockNavigate = jest.fn();
jest.mock('@react-navigation/native', () => ({
  useNavigation: () => ({
    navigate: mockNavigate
  })
}));

// Mock auth service
jest.mock('../../src/services/auth/AuthService');

describe('LoginScreen', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should render login form', () => {
    const { getByPlaceholderText, getByText } = render(<LoginScreen />);

    expect(getByPlaceholderText('Email')).toBeTruthy();
    expect(getByPlaceholderText('Password')).toBeTruthy();
    expect(getByText('Login')).toBeTruthy();
  });

  it('should handle successful login', async () => {
    const mockUser = { id: '1', email: 'test@example.com' };
    (authService.login as jest.Mock).mockResolvedValue({
      user: mockUser,
      tokens: { accessToken: 'token', refreshToken: 'refresh' }
    });

    const { getByPlaceholderText, getByText } = render(<LoginScreen />);

    fireEvent.changeText(getByPlaceholderText('Email'), 'test@example.com');
    fireEvent.changeText(getByPlaceholderText('Password'), 'password');
    fireEvent.press(getByText('Login'));

    await waitFor(() => {
      expect(authService.login).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password'
      });
      expect(mockNavigate).toHaveBeenCalledWith('Home');
    });
  });

  it('should handle login error', async () => {
    (authService.login as jest.Mock).mockRejectedValue(
      new Error('Invalid credentials')
    );

    const { getByPlaceholderText, getByText, findByText } = render(<LoginScreen />);

    fireEvent.changeText(getByPlaceholderText('Email'), 'test@example.com');
    fireEvent.changeText(getByPlaceholderText('Password'), 'wrong-password');
    fireEvent.press(getByText('Login'));

    expect(await findByText('Invalid credentials')).toBeTruthy();
  });
});
```

### **E2E Testing**
```typescript
// e2e/LoginFlow.test.ts
import { device, expect, element, by } from 'detox';

describe('Login Flow', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it('should login successfully', async () => {
    // Navigate to login screen
    await element(by.id('login-button')).tap();

    // Fill login form
    await element(by.id('email-input')).typeText('test@example.com');
    await element(by.id('password-input')).typeText('password123');

    // Submit form
    await element(by.id('submit-login')).tap();

    // Wait for navigation to home screen
    await expect(element(by.id('home-screen'))).toBeVisible();
    await expect(element(by.text('Welcome back!'))).toBeVisible();
  });

  it('should show error for invalid credentials', async () => {
    await element(by.id('login-button')).tap();

    await element(by.id('email-input')).typeText('invalid@example.com');
    await element(by.id('password-input')).typeText('wrongpassword');

    await element(by.id('submit-login')).tap();

    // Check for error message
    await expect(element(by.text('Invalid email or password'))).toBeVisible();
  });

  it('should navigate to register screen', async () => {
    await element(by.id('login-button')).tap();
    await element(by.id('register-link')).tap();

    await expect(element(by.id('register-screen'))).toBeVisible();
  });
});
```

---

## 🚀 **Deployment Preparation**

### **Build Configuration**
```javascript
// scripts/build.js
const { execSync } = require('child_process');
const fs = require('fs');
const path = require('path');

class BuildManager {
  constructor() {
    this.platforms = ['android', 'ios'];
    this.environments = ['development', 'staging', 'production'];
  }

  // Build Android APK
  buildAndroid(env = 'production') {
    console.log(`Building Android APK for ${env}...`);

    try {
      // Set environment variables
      process.env.NODE_ENV = env;

      // Clean previous build
      execSync('cd android && ./gradlew clean', { stdio: 'inherit' });

      // Build APK
      const buildCommand = env === 'production'
        ? 'cd android && ./gradlew assembleRelease'
        : 'cd android && ./gradlew assembleDebug';

      execSync(buildCommand, { stdio: 'inherit' });

      // Get APK path
      const apkPath = env === 'production'
        ? './android/app/build/outputs/apk/release/app-release.apk'
        : './android/app/build/outputs/apk/debug/app-debug.apk';

      console.log(`✅ Android APK built successfully: ${apkPath}`);
      return apkPath;
    } catch (error) {
      console.error('❌ Android build failed:', error.message);
      throw error;
    }
  }

  // Build iOS app
  buildIOS(env = 'production') {
    console.log(`Building iOS app for ${env}...`);

    try {
      // Set environment variables
      process.env.NODE_ENV = env;

      // Install CocoaPods
      execSync('cd ios && pod install', { stdio: 'inherit' });

      // Build iOS app
      const buildCommand = env === 'production'
        ? 'cd ios && xcodebuild -workspace SocialConnect.xcworkspace -scheme SocialConnect -configuration Release -destination generic/platform=iOS build'
        : 'cd ios && xcodebuild -workspace SocialConnect.xcworkspace -scheme SocialConnect -configuration Debug -destination generic/platform=iOS build';

      execSync(buildCommand, { stdio: 'inherit' });

      console.log(`✅ iOS app built successfully`);
      return './ios/build';
    } catch (error) {
      console.error('❌ iOS build failed:', error.message);
      throw error;
    }
  }

  // Build both platforms
  buildAll(env = 'production') {
    console.log(`Building all platforms for ${env}...`);

    const results = {};

    try {
      results.android = this.buildAndroid(env);
      console.log('Android build completed');
    } catch (error) {
      console.error('Android build failed:', error);
      results.android = null;
    }

    try {
      results.ios = this.buildIOS(env);
      console.log('iOS build completed');
    } catch (error) {
      console.error('iOS build failed:', error);
      results.ios = null;
    }

    return results;
  }

  // Generate build info
  generateBuildInfo(env = 'production') {
    const buildInfo = {
      version: require('../package.json').version,
      buildNumber: Date.now().toString(),
      environment: env,
      timestamp: new Date().toISOString(),
      gitCommit: execSync('git rev-parse HEAD').toString().trim(),
      gitBranch: execSync('git rev-parse --abbrev-ref HEAD').toString().trim(),
      nodeVersion: process.version,
      platform: process.platform
    };

    // Write build info
    const buildInfoPath = './build-info.json';
    fs.writeFileSync(buildInfoPath, JSON.stringify(buildInfo, null, 2));

    console.log(`Build info generated: ${buildInfoPath}`);
    return buildInfo;
  }

  // Validate build
  validateBuild(platform, buildPath) {
    console.log(`Validating ${platform} build...`);

    const validations = {
      android: () => {
        if (!fs.existsSync(buildPath)) {
          throw new Error('APK file not found');
        }

        const stats = fs.statSync(buildPath);
        if (stats.size < 10 * 1024 * 1024) { // 10MB minimum
          throw new Error('APK file seems too small');
        }

        console.log(`✅ Android APK validation passed (${(stats.size / 1024 / 1024).toFixed(2)}MB)`);
      },

      ios: () => {
        if (!fs.existsSync(buildPath)) {
          throw new Error('iOS build not found');
        }

        // Additional iOS-specific validations can be added here
        console.log('✅ iOS build validation passed');
      }
    };

    if (validations[platform]) {
      validations[platform]();
    }
  }

  // Create release notes
  createReleaseNotes(version, changes) {
    const releaseNotes = `# Release ${version}

## Changes
${changes.map(change => `- ${change}`).join('\n')}

## Build Information
- Build Date: ${new Date().toISOString()}
- Commit: ${execSync('git rev-parse HEAD').toString().trim()}
- Branch: ${execSync('git rev-parse --abbrev-ref HEAD').toString().trim()}

## Installation
- Download the latest APK from the releases page
- Install on your Android device
- Grant necessary permissions when prompted

## Known Issues
- None at this time

## Support
For support, please contact support@socialconnect.com
`;

    const releaseNotesPath = `./RELEASE_NOTES_v${version}.md`;
    fs.writeFileSync(releaseNotesPath, releaseNotes);

    console.log(`Release notes created: ${releaseNotesPath}`);
    return releaseNotesPath;
  }
}

// CLI usage
if (require.main === module) {
  const buildManager = new BuildManager();
  const [,, command, env] = process.argv;

  switch (command) {
    case 'android':
      buildManager.buildAndroid(env);
      break;

    case 'ios':
      buildManager.buildIOS(env);
      break;

    case 'all':
      buildManager.buildAll(env);
      break;

    case 'info':
      buildManager.generateBuildInfo(env);
      break;

    default:
      console.log('Usage: node build.js {android|ios|all|info} [environment]');
      console.log('Environments: development, staging, production');
  }
}

module.exports = BuildManager;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Full-Stack Architecture**: Complete React Native application with backend integration
- ✅ **Advanced State Management**: Complex state management with Context API and custom hooks
- ✅ **Real-time Features**: WebSocket integration for real-time messaging
- ✅ **Authentication & Security**: JWT authentication with biometric support
- ✅ **Testing Strategy**: Unit tests, integration tests, and E2E tests
- ✅ **Build & Deployment**: Automated build pipelines and deployment scripts
- ✅ **Performance Optimization**: Bundle optimization and performance monitoring
- ✅ **Code Quality**: TypeScript, linting, and code organization
- ✅ **Documentation**: Comprehensive documentation and release notes

### **Project Features Implemented**
- **User Authentication**: Login, registration, biometric auth, password reset
- **Real-time Messaging**: WebSocket-based chat with file sharing
- **Social Features**: Posts, comments, likes, user profiles
- **Media Handling**: Camera integration, image/video upload
- **Offline Support**: Local data storage and sync
- **Push Notifications**: Local and remote notifications
- **Location Services**: Maps integration and geolocation
- **Settings & Preferences**: Theme switching, language selection
- **Analytics**: User behavior tracking and reporting

### **Best Practices Applied**
1. **TypeScript**: Full type safety throughout the application
2. **Component Architecture**: Reusable, maintainable component structure
3. **State Management**: Efficient state management with Context API
4. **Error Handling**: Comprehensive error boundaries and handling
5. **Security**: Input validation, secure storage, authentication
6. **Performance**: Memoization, lazy loading, bundle optimization
7. **Testing**: Comprehensive test coverage with multiple testing types
8. **Documentation**: Inline documentation and API documentation
9. **Version Control**: Proper Git workflow and release management
10. **CI/CD**: Automated build, test, and deployment pipelines

### **Next Steps**
- **App Store Deployment**: Submit to Apple App Store and Google Play Store
- **Beta Testing**: Conduct beta testing with real users
- **Feature Expansion**: Add new features based on user feedback
- **Performance Monitoring**: Set up production monitoring and alerting
- **User Support**: Implement in-app support and feedback systems
- **Analytics Review**: Analyze user behavior and optimize features
- **Security Audits**: Regular security audits and updates
- **Scalability**: Plan for future growth and scaling requirements

---

## 🎯 **Final Assignment**

### **Capstone Project Requirements**
Build a complete React Native application that demonstrates mastery of all course concepts:

#### **Core Requirements**
- **User Authentication**: Complete auth flow with registration, login, logout
- **Main App Features**: At least 3 major features (e.g., social feed, messaging, profile)
- **Backend Integration**: RESTful API or GraphQL integration
- **Database**: Local storage with sync capabilities
- **Real-time Features**: WebSocket or similar real-time communication
- **Offline Support**: App works without internet connection

#### **Advanced Requirements**
- **TypeScript**: Full TypeScript implementation
- **Testing**: Unit tests, integration tests, E2E tests
- **Performance**: Optimized rendering and bundle size
- **Security**: Input validation, secure storage, authentication
- **UI/UX**: Modern, responsive design with accessibility
- **Error Handling**: Comprehensive error boundaries and user feedback

#### **Technical Requirements**
- **Code Quality**: ESLint, Prettier, proper code organization
- **Documentation**: README, API docs, inline comments
- **Build System**: Automated build and deployment scripts
- **Version Control**: Proper Git workflow with feature branches
- **Environment Config**: Separate configs for dev/staging/production

#### **Bonus Features**
- **Push Notifications**: Local and remote notifications
- **Camera/Media**: Photo/video capture and processing
- **Maps/Location**: Location services and maps integration
- **Biometric Auth**: Fingerprint/Face ID authentication
- **Multi-language**: Internationalization support
- **Dark Mode**: Theme switching capability
- **Analytics**: User behavior tracking
- **Deep Linking**: URL-based navigation

### **Submission Requirements**
1. **GitHub Repository**: Complete codebase with proper documentation
2. **Demo Video**: 5-10 minute video demonstrating all features
3. **Technical Documentation**: Architecture, API docs, setup instructions
4. **Test Results**: Test coverage reports and test execution
5. **Performance Report**: Bundle analysis and performance metrics
6. **Deployment Guide**: Step-by-step deployment instructions

### **Evaluation Criteria**
- **Functionality** (40%): All required features work correctly
- **Code Quality** (25%): Clean, maintainable, well-documented code
- **User Experience** (15%): Intuitive, responsive, accessible UI
- **Testing** (10%): Comprehensive test coverage and quality
- **Documentation** (10%): Clear, complete documentation

---

## 📚 **Additional Resources**
- [React Native Documentation](https://reactnative.dev/docs/getting-started)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Jest Testing Framework](https://jestjs.io/docs/getting-started)
- [Detox E2E Testing](https://wix.github.io/Detox/)
- [Fastlane for Deployment](https://fastlane.tools/)
- [App Store Connect](https://developer.apple.com/support/app-store-connect/)
- [Google Play Console](https://play.google.com/console/)

---

**Congratulations!** 🎉

You have successfully completed the comprehensive React Native Full-Stack Development Course. This capstone project demonstrates your mastery of mobile app development, from concept to deployment. The skills you've acquired will enable you to build production-ready React Native applications that can compete with any commercial app in the market.

**Keep Learning and Building!** 🚀