# Lesson 49: Testing Strategies

## 🎯 **Learning Objectives**
- Master comprehensive testing strategies for React Native applications
- Implement unit, integration, and end-to-end tests
- Set up automated testing pipelines
- Write testable code with proper architecture
- Debug and maintain test suites effectively

## 📚 **Table of Contents**
1. [Testing Fundamentals](#testing-fundamentals)
2. [Unit Testing](#unit-testing)
3. [Component Testing](#component-testing)
4. [Integration Testing](#integration-testing)
5. [End-to-End Testing](#end-to-end-testing)
6. [Mocking and Stubbing](#mocking-and-stubbing)
7. [Test-Driven Development](#test-driven-development)
8. [Testing Best Practices](#testing-best-practices)
9. [CI/CD Integration](#cicd-integration)
10. [Practical Examples](#practical-examples)

---

## 🧪 **Testing Fundamentals**

### **Testing Pyramid**
```
E2E Tests (Slow, Expensive)
    ↕️
Integration Tests (Medium)
    ↕️
Unit Tests (Fast, Cheap)
```

### **Testing Types**
- **Unit Tests**: Test individual functions/components
- **Integration Tests**: Test component interactions
- **End-to-End Tests**: Test complete user workflows
- **Snapshot Tests**: Test UI consistency
- **Performance Tests**: Test app performance
- **Accessibility Tests**: Test app accessibility

### **Testing Tools**
- **Jest**: JavaScript testing framework
- **React Native Testing Library**: Component testing
- **Detox**: End-to-end testing
- **Mock Service Worker**: API mocking
- **Jest Mocks**: Function and module mocking

---

## 🧩 **Unit Testing**

### **Jest Configuration**
```javascript
// jest.config.js
module.exports = {
  preset: 'react-native',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  testEnvironment: 'jsdom',
  testMatch: [
    '**/__tests__/**/*.test.js',
    '**/?(*.)+(spec|test).js',
  ],
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js',
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^@components/(.*)$': '<rootDir>/src/components/$1',
    '^@utils/(.*)$': '<rootDir>/src/utils/$1',
  },
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?|react-navigation|@react-navigation/.*))',
  ],
};
```

```javascript
// jest.setup.js
import 'react-native-gesture-handler/jestSetup';
import mockAsyncStorage from '@react-native-async-storage/async-storage/jest/async-storage-mock';

// Mock AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () => mockAsyncStorage);

// Mock react-native-reanimated
jest.mock('react-native-reanimated', () => {
  const Reanimated = require('react-native-reanimated/mock');
  return Reanimated;
});

// Mock react-native-vector-icons
jest.mock('react-native-vector-icons', () => ({
  createIconSet: () => 'IconMock',
}));

// Mock react-native-permissions
jest.mock('react-native-permissions', () => ({
  check: jest.fn(),
  request: jest.fn(),
}));

// Global test utilities
global.fetch = jest.fn();

// Mock timers
jest.useFakeTimers();

// Silence console warnings during tests
const originalWarn = console.warn;
console.warn = (...args) => {
  if (args[0]?.includes?.('Warning:')) return;
  originalWarn.call(console, ...args);
};
```

### **Utility Function Testing**
```javascript
// src/utils/dateUtils.js
export const formatDate = (date, format = 'MM/DD/YYYY') => {
  if (!date) return '';

  const d = new Date(date);

  if (isNaN(d.getTime())) return '';

  const year = d.getFullYear();
  const month = String(d.getMonth() + 1).padStart(2, '0');
  const day = String(d.getDate()).padStart(2, '0');

  switch (format) {
    case 'MM/DD/YYYY':
      return `${month}/${day}/${year}`;
    case 'DD/MM/YYYY':
      return `${day}/${month}/${year}`;
    case 'YYYY-MM-DD':
      return `${year}-${month}-${day}`;
    default:
      return `${month}/${day}/${year}`;
  }
};

export const isToday = (date) => {
  if (!date) return false;

  const today = new Date();
  const d = new Date(date);

  return d.getDate() === today.getDate() &&
         d.getMonth() === today.getMonth() &&
         d.getFullYear() === today.getFullYear();
};

export const getRelativeTime = (date) => {
  if (!date) return '';

  const now = new Date();
  const d = new Date(date);
  const diffInMs = now - d;
  const diffInMinutes = Math.floor(diffInMs / (1000 * 60));
  const diffInHours = Math.floor(diffInMinutes / 60);
  const diffInDays = Math.floor(diffInHours / 24);

  if (diffInMinutes < 1) return 'Just now';
  if (diffInMinutes < 60) return `${diffInMinutes}m ago`;
  if (diffInHours < 24) return `${diffInHours}h ago`;
  if (diffInDays < 7) return `${diffInDays}d ago`;

  return formatDate(d);
};
```

```javascript
// src/utils/__tests__/dateUtils.test.js
import { formatDate, isToday, getRelativeTime } from '../dateUtils';

describe('Date Utils', () => {
  describe('formatDate', () => {
    it('should format date in MM/DD/YYYY format by default', () => {
      const date = new Date('2023-12-25');
      expect(formatDate(date)).toBe('12/25/2023');
    });

    it('should format date in DD/MM/YYYY format', () => {
      const date = new Date('2023-12-25');
      expect(formatDate(date, 'DD/MM/YYYY')).toBe('25/12/2023');
    });

    it('should format date in YYYY-MM-DD format', () => {
      const date = new Date('2023-12-25');
      expect(formatDate(date, 'YYYY-MM-DD')).toBe('2023-12-25');
    });

    it('should return empty string for invalid date', () => {
      expect(formatDate('invalid')).toBe('');
      expect(formatDate(null)).toBe('');
      expect(formatDate(undefined)).toBe('');
    });
  });

  describe('isToday', () => {
    beforeEach(() => {
      // Mock Date to return a fixed date
      const mockDate = new Date('2023-12-25T10:00:00Z');
      jest.spyOn(global, 'Date').mockImplementation(() => mockDate);
    });

    afterEach(() => {
      jest.restoreAllMocks();
    });

    it('should return true for today\'s date', () => {
      const today = new Date('2023-12-25');
      expect(isToday(today)).toBe(true);
    });

    it('should return false for different date', () => {
      const yesterday = new Date('2023-12-24');
      expect(isToday(yesterday)).toBe(false);
    });

    it('should return false for invalid date', () => {
      expect(isToday('invalid')).toBe(false);
      expect(isToday(null)).toBe(false);
    });
  });

  describe('getRelativeTime', () => {
    beforeEach(() => {
      // Mock Date.now to return a fixed time
      const mockNow = new Date('2023-12-25T10:00:00Z').getTime();
      jest.spyOn(Date, 'now').mockReturnValue(mockNow);
    });

    afterEach(() => {
      jest.restoreAllMocks();
    });

    it('should return "Just now" for very recent dates', () => {
      const recent = new Date('2023-12-25T09:59:30Z');
      expect(getRelativeTime(recent)).toBe('Just now');
    });

    it('should return minutes ago for recent dates', () => {
      const fiveMinAgo = new Date('2023-12-25T09:55:00Z');
      expect(getRelativeTime(fiveMinAgo)).toBe('5m ago');
    });

    it('should return hours ago for older dates', () => {
      const twoHoursAgo = new Date('2023-12-25T08:00:00Z');
      expect(getRelativeTime(twoHoursAgo)).toBe('2h ago');
    });

    it('should return days ago for much older dates', () => {
      const threeDaysAgo = new Date('2023-12-22T10:00:00Z');
      expect(getRelativeTime(threeDaysAgo)).toBe('3d ago');
    });

    it('should return formatted date for very old dates', () => {
      const oldDate = new Date('2023-12-15T10:00:00Z');
      expect(getRelativeTime(oldDate)).toBe('12/15/2023');
    });

    it('should return empty string for invalid date', () => {
      expect(getRelativeTime('invalid')).toBe('');
      expect(getRelativeTime(null)).toBe('');
    });
  });
});
```

---

## 🧩 **Component Testing**

### **React Native Testing Library Setup**
```javascript
// src/components/__tests__/Button.test.js
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { Button } from '../Button';

describe('Button', () => {
  const defaultProps = {
    title: 'Click me',
    onPress: jest.fn(),
  };

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should render correctly', () => {
    const { getByText } = render(<Button {...defaultProps} />);
    expect(getByText('Click me')).toBeTruthy();
  });

  it('should call onPress when pressed', () => {
    const { getByText } = render(<Button {...defaultProps} />);
    const button = getByText('Click me');

    fireEvent.press(button);
    expect(defaultProps.onPress).toHaveBeenCalledTimes(1);
  });

  it('should show loading state', () => {
    const { getByText, queryByText } = render(
      <Button {...defaultProps} loading={true} />
    );

    expect(getByText('Loading...')).toBeTruthy();
    expect(queryByText('Click me')).toBeNull();
  });

  it('should be disabled when loading', () => {
    const { getByText } = render(
      <Button {...defaultProps} loading={true} />
    );
    const button = getByText('Loading...');

    fireEvent.press(button);
    expect(defaultProps.onPress).not.toHaveBeenCalled();
  });

  it('should apply custom styles', () => {
    const customStyle = { backgroundColor: 'red' };
    const { getByTestId } = render(
      <Button {...defaultProps} style={customStyle} testID="custom-button" />
    );
    const button = getByTestId('custom-button');

    expect(button.props.style).toContain(customStyle);
  });

  it('should handle disabled state', () => {
    const { getByText } = render(
      <Button {...defaultProps} disabled={true} />
    );
    const button = getByText('Click me');

    fireEvent.press(button);
    expect(defaultProps.onPress).not.toHaveBeenCalled();
  });
});
```

### **Async Component Testing**
```javascript
// src/components/__tests__/UserProfile.test.js
import React from 'react';
import { render, waitFor, screen } from '@testing-library/react-native';
import { UserProfile } from '../UserProfile';
import * as api from '../../services/api';

// Mock the API
jest.mock('../../services/api');

describe('UserProfile', () => {
  const mockUser = {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
    avatar: 'https://example.com/avatar.jpg',
  };

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should show loading state initially', () => {
    api.get.mockImplementation(() => new Promise(() => {})); // Never resolves

    render(<UserProfile userId={1} />);

    expect(screen.getByText('Loading...')).toBeTruthy();
  });

  it('should display user data when loaded', async () => {
    api.get.mockResolvedValue({ data: mockUser });

    render(<UserProfile userId={1} />);

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeTruthy();
      expect(screen.getByText('john@example.com')).toBeTruthy();
    });

    expect(api.get).toHaveBeenCalledWith('/users/1');
  });

  it('should handle API errors', async () => {
    const errorMessage = 'User not found';
    api.get.mockRejectedValue(new Error(errorMessage));

    render(<UserProfile userId={1} />);

    await waitFor(() => {
      expect(screen.getByText(`Error: ${errorMessage}`)).toBeTruthy();
    });
  });

  it('should retry on error when retry button is pressed', async () => {
    api.get
      .mockRejectedValueOnce(new Error('Network error'))
      .mockResolvedValueOnce({ data: mockUser });

    render(<UserProfile userId={1} />);

    await waitFor(() => {
      expect(screen.getByText('Error: Network error')).toBeTruthy();
    });

    const retryButton = screen.getByText('Retry');
    fireEvent.press(retryButton);

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeTruthy();
    });

    expect(api.get).toHaveBeenCalledTimes(2);
  });

  it('should call onUserUpdate when user data changes', async () => {
    const onUserUpdate = jest.fn();
    api.get.mockResolvedValue({ data: mockUser });

    render(<UserProfile userId={1} onUserUpdate={onUserUpdate} />);

    await waitFor(() => {
      expect(onUserUpdate).toHaveBeenCalledWith(mockUser);
    });
  });
});
```

### **Hook Testing**
```javascript
// src/hooks/__tests__/useAuth.test.js
import { renderHook, act, waitFor } from '@testing-library/react-native';
import { useAuth } from '../useAuth';
import * as api from '../../services/api';

// Mock the API
jest.mock('../../services/api');

describe('useAuth', () => {
  const mockUser = {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
  };

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should initialize with default state', () => {
    const { result } = renderHook(() => useAuth());

    expect(result.current.user).toBeNull();
    expect(result.current.loading).toBe(false);
    expect(result.current.error).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
  });

  it('should handle successful login', async () => {
    api.post.mockResolvedValue({
      data: { user: mockUser, token: 'mock-token' }
    });

    const { result } = renderHook(() => useAuth());

    act(() => {
      result.current.login({ email: 'john@example.com', password: 'password' });
    });

    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
      expect(result.current.user).toEqual(mockUser);
      expect(result.current.isAuthenticated).toBe(true);
      expect(result.current.error).toBeNull();
    });

    expect(api.post).toHaveBeenCalledWith('/auth/login', {
      email: 'john@example.com',
      password: 'password',
    });
  });

  it('should handle login error', async () => {
    const errorMessage = 'Invalid credentials';
    api.post.mockRejectedValue(new Error(errorMessage));

    const { result } = renderHook(() => useAuth());

    act(() => {
      result.current.login({ email: 'john@example.com', password: 'wrong' });
    });

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
      expect(result.current.user).toBeNull();
      expect(result.current.isAuthenticated).toBe(false);
      expect(result.current.error).toBe(errorMessage);
    });
  });

  it('should handle logout', async () => {
    api.post.mockResolvedValue({
      data: { user: mockUser, token: 'mock-token' }
    });

    const { result } = renderHook(() => useAuth());

    // Login first
    act(() => {
      result.current.login({ email: 'john@example.com', password: 'password' });
    });

    await waitFor(() => {
      expect(result.current.isAuthenticated).toBe(true);
    });

    // Logout
    act(() => {
      result.current.logout();
    });

    expect(result.current.user).toBeNull();
    expect(result.current.isAuthenticated).toBe(false);
    expect(result.current.error).toBeNull();
  });

  it('should handle profile update', async () => {
    const updatedUser = { ...mockUser, name: 'John Smith' };
    api.post.mockResolvedValueOnce({
      data: { user: mockUser, token: 'mock-token' }
    });
    api.put.mockResolvedValueOnce({ data: updatedUser });

    const { result } = renderHook(() => useAuth());

    // Login first
    act(() => {
      result.current.login({ email: 'john@example.com', password: 'password' });
    });

    await waitFor(() => {
      expect(result.current.isAuthenticated).toBe(true);
    });

    // Update profile
    act(() => {
      result.current.updateProfile({ name: 'John Smith' });
    });

    await waitFor(() => {
      expect(result.current.user.name).toBe('John Smith');
    });

    expect(api.put).toHaveBeenCalledWith('/users/1', { name: 'John Smith' });
  });
});
```

---

## 🔗 **Integration Testing**

### **Redux Store Testing**
```javascript
// src/store/__tests__/userSlice.test.js
import userSlice, {
  loginUser,
  fetchUserProfile,
  updateUserProfile,
  logout,
  clearError,
} from '../slices/userSlice';
import { configureStore } from '@reduxjs/toolkit';

// Mock API
jest.mock('../../services/api');

describe('User Slice', () => {
  let store;

  beforeEach(() => {
    store = configureStore({
      reducer: {
        user: userSlice,
      },
    });
  });

  it('should handle initial state', () => {
    expect(store.getState().user).toEqual({
      currentUser: null,
      profile: null,
      loading: false,
      error: null,
      isAuthenticated: false,
      lastLogin: null,
    });
  });

  it('should handle logout', () => {
    // Set some initial state
    store.dispatch({
      type: 'user/loginUser/fulfilled',
      payload: { user: { id: 1, name: 'John' } },
    });

    store.dispatch(logout());

    expect(store.getState().user).toEqual({
      currentUser: null,
      profile: null,
      loading: false,
      error: null,
      isAuthenticated: false,
      lastLogin: null,
    });
  });

  it('should handle clearError', () => {
    // Set an error
    store.dispatch({
      type: 'user/loginUser/rejected',
      payload: 'Login failed',
    });

    store.dispatch(clearError());

    expect(store.getState().user.error).toBeNull();
  });

  it('should handle loginUser.pending', () => {
    store.dispatch(loginUser.pending());

    expect(store.getState().user.loading).toBe(true);
    expect(store.getState().user.error).toBeNull();
  });

  it('should handle loginUser.fulfilled', () => {
    const mockUser = { id: 1, name: 'John Doe' };
    const mockToken = 'mock-jwt-token';

    store.dispatch(loginUser.fulfilled({
      user: mockUser,
      token: mockToken,
    }));

    const state = store.getState().user;
    expect(state.loading).toBe(false);
    expect(state.currentUser).toEqual(mockUser);
    expect(state.isAuthenticated).toBe(true);
    expect(state.error).toBeNull();
    expect(state.lastLogin).toBeTruthy();
  });

  it('should handle loginUser.rejected', () => {
    const errorMessage = 'Invalid credentials';

    store.dispatch(loginUser.rejected(errorMessage));

    const state = store.getState().user;
    expect(state.loading).toBe(false);
    expect(state.error).toBe(errorMessage);
    expect(state.isAuthenticated).toBe(false);
  });
});
```

### **API Integration Testing**
```javascript
// src/services/__tests__/api.test.js
import MockAdapter from 'axios-mock-adapter';
import api from '../api';

describe('API Service', () => {
  let mock;

  beforeEach(() => {
    mock = new MockAdapter(api);
  });

  afterEach(() => {
    mock.restore();
  });

  describe('GET /users', () => {
    it('should fetch users successfully', async () => {
      const mockUsers = [
        { id: 1, name: 'John Doe' },
        { id: 2, name: 'Jane Smith' },
      ];

      mock.onGet('/users').reply(200, mockUsers);

      const response = await api.get('/users');

      expect(response.status).toBe(200);
      expect(response.data).toEqual(mockUsers);
    });

    it('should handle network errors', async () => {
      mock.onGet('/users').networkError();

      await expect(api.get('/users')).rejects.toThrow();
    });

    it('should handle 404 errors', async () => {
      mock.onGet('/users/999').reply(404, { message: 'User not found' });

      try {
        await api.get('/users/999');
        fail('Should have thrown an error');
      } catch (error) {
        expect(error.response.status).toBe(404);
        expect(error.response.data.message).toBe('User not found');
      }
    });
  });

  describe('POST /users', () => {
    it('should create user successfully', async () => {
      const newUser = { name: 'John Doe', email: 'john@example.com' };
      const createdUser = { id: 1, ...newUser };

      mock.onPost('/users').reply(201, createdUser);

      const response = await api.post('/users', newUser);

      expect(response.status).toBe(201);
      expect(response.data).toEqual(createdUser);
    });

    it('should handle validation errors', async () => {
      const invalidUser = { name: '', email: 'invalid-email' };
      const validationErrors = {
        name: ['Name is required'],
        email: ['Email is invalid'],
      };

      mock.onPost('/users').reply(422, validationErrors);

      try {
        await api.post('/users', invalidUser);
        fail('Should have thrown an error');
      } catch (error) {
        expect(error.response.status).toBe(422);
        expect(error.response.data).toEqual(validationErrors);
      }
    });
  });

  describe('Authentication', () => {
    it('should include auth token in requests', async () => {
      const mockUser = { id: 1, name: 'John Doe' };
      mock.onGet('/users/profile').reply(200, mockUser);

      // Mock the auth token retrieval
      jest.mock('../auth', () => ({
        getAuthToken: () => 'mock-token',
      }));

      const response = await api.get('/users/profile');

      expect(response.config.headers.Authorization).toBe('Bearer mock-token');
    });

    it('should handle token refresh', async () => {
      // First request fails with 401
      mock.onGet('/users/profile').replyOnce(401, { message: 'Token expired' });

      // Token refresh succeeds
      mock.onPost('/auth/refresh').reply(200, { token: 'new-token' });

      // Second request succeeds
      mock.onGet('/users/profile').reply(200, { id: 1, name: 'John' });

      const response = await api.get('/users/profile');

      expect(response.status).toBe(200);
      expect(response.data.name).toBe('John');
    });
  });
});
```

---

## 🤖 **End-to-End Testing**

### **Detox Setup**
```javascript
// e2e/config.json
{
  "testRunner": "jest",
  "runnerConfig": "e2e/config.js",
  "skipLegacyWorkersInjection": true,
  "apps": {
    "ios": {
      "type": "ios.app",
      "binaryPath": "ios/build/Build/Products/Debug-iphonesimulator/ReactNativeTesting.app",
      "build": "xcodebuild -workspace ios/ReactNativeTesting.xcworkspace -scheme ReactNativeTesting -configuration Debug -sdk iphonesimulator -derivedDataPath ios/build"
    },
    "android": {
      "type": "android.apk",
      "binaryPath": "android/app/build/outputs/apk/debug/app-debug.apk",
      "build": "cd android && ./gradlew assembleDebug assembleAndroidTest -DtestBuildType=debug && cd .."
    }
  },
  "devices": {
    "simulator": {
      "type": "ios.simulator",
      "device": {
        "type": "iPhone 13"
      }
    },
    "emulator": {
      "type": "android.emulator",
      "device": {
        "avdName": "Pixel_4_API_29"
      }
    }
  },
  "configurations": {
    "ios": {
      "device": "simulator",
      "app": "ios"
    },
    "android": {
      "device": "emulator",
      "app": "android"
    }
  }
}
```

```javascript
// e2e/config.js
module.exports = {
  testEnvironment: 'node',
  rootDir: '..',
  testMatch: ['<rootDir>/e2e/**/*.test.js'],
  verbose: true,
  setupFilesAfterEnv: ['<rootDir>/e2e/init.js'],
  maxWorkers: 1,
  globalSetup: '<rootDir>/e2e/global-setup.js',
  globalTeardown: '<rootDir>/e2e/global-teardown.js',
};
```

```javascript
// e2e/init.js
const detox = require('detox');
const config = require('./config.json');

beforeAll(async () => {
  await detox.init(config);
});

afterAll(async () => {
  await detox.cleanup();
});
```

### **E2E Test Examples**
```javascript
// e2e/tests/login.test.js
import { device, expect, element, by } from 'detox';

describe('Login Flow', () => {
  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it('should login successfully with valid credentials', async () => {
    // Navigate to login screen
    await expect(element(by.id('login-screen'))).toBeVisible();

    // Enter credentials
    await element(by.id('email-input')).typeText('john@example.com');
    await element(by.id('password-input')).typeText('password123');

    // Tap login button
    await element(by.id('login-button')).tap();

    // Wait for success and navigation
    await waitFor(element(by.id('home-screen')))
      .toBeVisible()
      .withTimeout(5000);

    // Verify user is logged in
    await expect(element(by.id('welcome-message'))).toHaveText('Welcome, John!');
  });

  it('should show error for invalid credentials', async () => {
    await element(by.id('email-input')).typeText('invalid@example.com');
    await element(by.id('password-input')).typeText('wrongpassword');
    await element(by.id('login-button')).tap();

    // Wait for error message
    await waitFor(element(by.id('error-message')))
      .toBeVisible()
      .withTimeout(3000);

    await expect(element(by.id('error-message'))).toHaveText('Invalid credentials');
  });

  it('should handle network errors gracefully', async () => {
    // Disable network
    await device.setURLBlacklist(['https://api.example.com/**']);

    await element(by.id('email-input')).typeText('john@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('login-button')).tap();

    await waitFor(element(by.id('error-message')))
      .toBeVisible()
      .withTimeout(3000);

    await expect(element(by.id('error-message'))).toHaveText('Network error');
  });
});
```

```javascript
// e2e/tests/navigation.test.js
import { device, expect, element, by, waitFor } from 'detox';

describe('Navigation', () => {
  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it('should navigate between screens', async () => {
    // Start at home screen
    await expect(element(by.id('home-screen'))).toBeVisible();

    // Navigate to profile
    await element(by.id('profile-tab')).tap();
    await expect(element(by.id('profile-screen'))).toBeVisible();

    // Navigate to settings
    await element(by.id('settings-button')).tap();
    await expect(element(by.id('settings-screen'))).toBeVisible();

    // Go back
    await element(by.id('back-button')).tap();
    await expect(element(by.id('profile-screen'))).toBeVisible();

    // Go back to home
    await device.pressBack();
    await expect(element(by.id('home-screen'))).toBeVisible();
  });

  it('should handle deep linking', async () => {
    // Simulate deep link
    await device.launchApp({
      newInstance: true,
      url: 'myapp://profile/123',
    });

    // Should navigate to profile screen
    await waitFor(element(by.id('profile-screen')))
      .toBeVisible()
      .withTimeout(5000);

    // Should show correct user profile
    await expect(element(by.id('user-name'))).toHaveText('John Doe');
  });

  it('should handle push notifications', async () => {
    // Simulate push notification
    await device.launchApp({
      newInstance: true,
      notification: {
        title: 'New Message',
        body: 'You have a new message from Jane',
        payload: { userId: 456 },
      },
    });

    // Should navigate to messages screen
    await waitFor(element(by.id('messages-screen')))
      .toBeVisible()
      .withTimeout(5000);

    // Should show the message
    await expect(element(by.id('message-text'))).toHaveText('You have a new message from Jane');
  });
});
```

---

## 🎭 **Mocking and Stubbing**

### **API Mocking with MSW**
```javascript
// src/mocks/handlers.js
import { rest } from 'msw';

export const handlers = [
  // Mock user authentication
  rest.post('/api/auth/login', (req, res, ctx) => {
    const { email, password } = req.body;

    if (email === 'john@example.com' && password === 'password123') {
      return res(
        ctx.status(200),
        ctx.json({
          user: {
            id: 1,
            name: 'John Doe',
            email: 'john@example.com',
          },
          token: 'mock-jwt-token',
        })
      );
    }

    return res(
      ctx.status(401),
      ctx.json({ message: 'Invalid credentials' })
    );
  }),

  // Mock user profile
  rest.get('/api/users/:userId', (req, res, ctx) => {
    const { userId } = req.params;

    if (userId === '1') {
      return res(
        ctx.status(200),
        ctx.json({
          id: 1,
          name: 'John Doe',
          email: 'john@example.com',
          avatar: 'https://example.com/avatar.jpg',
          bio: 'Software developer',
        })
      );
    }

    return res(
      ctx.status(404),
      ctx.json({ message: 'User not found' })
    );
  }),

  // Mock posts
  rest.get('/api/posts', (req, res, ctx) => {
    const page = parseInt(req.url.searchParams.get('page') || '1');
    const limit = parseInt(req.url.searchParams.get('limit') || '10');

    const mockPosts = Array.from({ length: limit }, (_, i) => ({
      id: (page - 1) * limit + i + 1,
      title: `Post ${(page - 1) * limit + i + 1}`,
      content: `This is the content of post ${(page - 1) * limit + i + 1}`,
      author: 'John Doe',
      createdAt: new Date().toISOString(),
    }));

    return res(
      ctx.status(200),
      ctx.json({
        posts: mockPosts,
        total: 100,
        page,
        limit,
        hasMore: page * limit < 100,
      })
    );
  }),

  // Mock network delay
  rest.get('/api/slow-endpoint', (req, res, ctx) => {
    return res(
      ctx.delay(2000), // 2 second delay
      ctx.status(200),
      ctx.json({ message: 'Slow response' })
    );
  }),

  // Mock error responses
  rest.get('/api/error-endpoint', (req, res, ctx) => {
    return res(
      ctx.status(500),
      ctx.json({ message: 'Internal server error' })
    );
  }),
];
```

```javascript
// src/mocks/server.js
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

// Setup requests interception using the given handlers
export const server = setupServer(...handlers);
```

```javascript
// src/mocks/browser.js
import { setupWorker } from 'msw';
import { handlers } from './handlers';

// Setup requests interception using the given handlers
export const worker = setupWorker(...handlers);
```

### **Component Mocking**
```javascript
// src/components/__mocks__/react-native-maps.js
const React = require('react');
const { View, Text } = require('react-native');

const MapView = ({ children, ...props }) => (
  <View {...props} testID="mock-map-view">
    <Text>Map View Mock</Text>
    {children}
  </View>
);

const Marker = ({ children, ...props }) => (
  <View {...props} testID="mock-marker">
    <Text>Marker Mock</Text>
    {children}
  </View>
);

const Polyline = (props) => (
  <View {...props} testID="mock-polyline">
    <Text>Polyline Mock</Text>
  </View>
);

module.exports = {
  MapView,
  Marker,
  Polyline,
  PROVIDER_GOOGLE: 'google',
  PROVIDER_DEFAULT: 'default',
};
```

---

## 🧪 **Test-Driven Development**

### **TDD Example**
```javascript
// src/utils/__tests__/calculator.test.js
import { add, subtract, multiply, divide } from '../calculator';

// Red: Write failing test first
describe('Calculator', () => {
  describe('add', () => {
    it('should add two positive numbers', () => {
      expect(add(2, 3)).toBe(5);
    });

    it('should add negative numbers', () => {
      expect(add(-2, -3)).toBe(-5);
    });

    it('should add zero', () => {
      expect(add(5, 0)).toBe(5);
    });
  });

  describe('subtract', () => {
    it('should subtract two numbers', () => {
      expect(subtract(5, 3)).toBe(2);
    });

    it('should handle negative results', () => {
      expect(subtract(3, 5)).toBe(-2);
    });
  });

  describe('multiply', () => {
    it('should multiply two numbers', () => {
      expect(multiply(3, 4)).toBe(12);
    });

    it('should handle zero', () => {
      expect(multiply(5, 0)).toBe(0);
    });
  });

  describe('divide', () => {
    it('should divide two numbers', () => {
      expect(divide(10, 2)).toBe(5);
    });

    it('should handle decimal results', () => {
      expect(divide(5, 2)).toBe(2.5);
    });

    it('should throw error for division by zero', () => {
      expect(() => divide(5, 0)).toThrow('Division by zero');
    });
  });
});
```

```javascript
// src/utils/calculator.js
// Green: Write minimal code to pass tests
export const add = (a, b) => {
  return a + b;
};

export const subtract = (a, b) => {
  return a - b;
};

export const multiply = (a, b) => {
  return a * b;
};

export const divide = (a, b) => {
  if (b === 0) {
    throw new Error('Division by zero');
  }
  return a / b;
};
```

### **Refactor: Improve Code Quality**
```javascript
// src/utils/calculator.js
// Refactor: Add input validation and improve error handling
export const add = (a, b) => {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw new Error('Both arguments must be numbers');
  }
  return a + b;
};

export const subtract = (a, b) => {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw new Error('Both arguments must be numbers');
  }
  return a - b;
};

export const multiply = (a, b) => {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw new Error('Both arguments must be numbers');
  }
  return a * b;
};

export const divide = (a, b) => {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw new Error('Both arguments must be numbers');
  }
  if (b === 0) {
    throw new Error('Division by zero');
  }
  return a / b;
};
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Unit Testing**: Testing individual functions and utilities
- ✅ **Component Testing**: Testing React Native components
- ✅ **Integration Testing**: Testing component interactions
- ✅ **End-to-End Testing**: Testing complete user workflows
- ✅ **Mocking**: Mocking APIs, components, and external dependencies
- ✅ **Test-Driven Development**: Writing tests before implementation
- ✅ **Testing Best Practices**: Code coverage, test organization
- ✅ **CI/CD Integration**: Automated testing in deployment pipelines

### **Best Practices**
1. **Write tests for all new code** and maintain existing tests
2. **Use descriptive test names** that explain what is being tested
3. **Test both positive and negative scenarios**
4. **Mock external dependencies** to isolate code under test
5. **Aim for high code coverage** but prioritize important paths
6. **Run tests frequently** during development
7. **Use test doubles** (mocks, stubs, spies) appropriately
8. **Keep tests fast and reliable**
9. **Test edge cases and error conditions**
10. **Maintain test code quality** alongside production code

### **Next Steps**
- Learn advanced testing patterns and techniques
- Implement visual regression testing
- Set up performance testing
- Learn about property-based testing
- Explore testing in different environments

---

## 🎯 **Assignment**

### **Task 1: Unit Testing Suite**
Create comprehensive unit tests for utility functions:
- Date formatting and manipulation functions
- String processing and validation utilities
- Mathematical calculation functions
- Data transformation helpers
- API response parsers

### **Task 2: Component Testing**
Build component tests for common UI elements:
- Form inputs with validation
- List components with pagination
- Modal dialogs with user interactions
- Navigation components
- Error boundary components

### **Task 3: Integration Testing**
Implement integration tests for key user flows:
- User authentication and registration
- Data fetching and display
- Form submission and validation
- Navigation between screens
- Offline functionality

### **Task 4: E2E Testing Setup**
Create end-to-end tests for critical user journeys:
- Complete user onboarding flow
- Product purchase/checkout flow
- User profile management
- Settings and preferences
- Error recovery scenarios

---

## 📚 **Additional Resources**
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Detox Documentation](https://wix.github.io/Detox/)
- [MSW Documentation](https://mswjs.io/docs/)
- [Testing Best Practices](https://testing-library.com/docs/react-native-testing-library/intro/)

---

**Next Lesson**: [Lesson 50: Deployment & DevOps](Lesson%2050_%20Deployment%20&%20DevOps.md)