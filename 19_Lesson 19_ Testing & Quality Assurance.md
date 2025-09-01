# Lesson 19: Testing & Quality Assurance

## 🎯 **Learning Objectives**
- Implement comprehensive testing strategies for React Native apps
- Write unit tests, integration tests, and end-to-end tests
- Set up testing frameworks and tools
- Create testable code architecture
- Implement continuous integration and deployment

## 📚 **Table of Contents**
1. [Introduction to Testing](#introduction-to-testing)
2. [Unit Testing](#unit-testing)
3. [Component Testing](#component-testing)
4. [Integration Testing](#integration-testing)
5. [End-to-End Testing](#end-to-end-testing)
6. [Mocking & Test Utilities](#mocking--test-utilities)
7. [Test-Driven Development](#test-driven-development)
8. [Continuous Integration](#continuous-integration)
9. [Performance Testing](#performance-testing)
10. [Code Quality Tools](#code-quality-tools)
11. [Practical Examples](#practical-examples)

---

## 🧪 **Introduction to Testing**

Testing is crucial for maintaining code quality and preventing regressions. A comprehensive testing strategy includes multiple types of tests working together.

### **Testing Pyramid**
```
E2E Tests (Slow, High Value)
    ↕️
Integration Tests (Medium Speed, Medium Value)
    ↕️
Unit Tests (Fast, Low Value)
```

### **Testing Types**
- **Unit Tests**: Test individual functions and components in isolation
- **Integration Tests**: Test how different parts work together
- **End-to-End Tests**: Test complete user workflows
- **Snapshot Tests**: Test UI components for visual regressions
- **Performance Tests**: Test app speed and resource usage

### **Testing Frameworks**
- **Jest**: JavaScript testing framework
- **React Native Testing Library**: Component testing utilities
- **Detox**: End-to-end testing framework
- **Maestro**: Mobile UI testing framework

---

## 🧩 **Unit Testing**

### **Jest Setup**
```javascript
// jest.config.js
module.exports = {
  preset: 'react-native',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?|react-navigation|@react-navigation/.*))',
  ],
  testMatch: [
    '**/__tests__/**/*.test.js',
    '**/?(*.)+(spec|test).js',
  ],
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
  ],
  coverageReporters: ['text', 'lcov', 'html'],
};
```

### **Basic Unit Tests**
```javascript
// __tests__/math.test.js
const { add, subtract, multiply, divide } = require('../src/utils/math');

describe('Math Utilities', () => {
  describe('add', () => {
    test('adds two positive numbers', () => {
      expect(add(2, 3)).toBe(5);
    });

    test('adds positive and negative numbers', () => {
      expect(add(5, -3)).toBe(2);
    });

    test('adds two negative numbers', () => {
      expect(add(-2, -3)).toBe(-5);
    });

    test('adds with zero', () => {
      expect(add(0, 5)).toBe(5);
      expect(add(5, 0)).toBe(5);
    });
  });

  describe('subtract', () => {
    test('subtracts two numbers', () => {
      expect(subtract(5, 3)).toBe(2);
    });

    test('subtracts negative numbers', () => {
      expect(subtract(5, -3)).toBe(8);
    });
  });

  describe('multiply', () => {
    test('multiplies two positive numbers', () => {
      expect(multiply(2, 3)).toBe(6);
    });

    test('multiplies with zero', () => {
      expect(multiply(0, 5)).toBe(0);
    });

    test('multiplies negative numbers', () => {
      expect(multiply(-2, 3)).toBe(-6);
      expect(multiply(-2, -3)).toBe(6);
    });
  });

  describe('divide', () => {
    test('divides two numbers', () => {
      expect(divide(6, 3)).toBe(2);
    });

    test('divides with decimal result', () => {
      expect(divide(5, 2)).toBe(2.5);
    });

    test('throws error when dividing by zero', () => {
      expect(() => divide(5, 0)).toThrow('Cannot divide by zero');
    });
  });
});
```

### **Async Testing**
```javascript
// __tests__/api.test.js
const api = require('../src/api');

describe('API Functions', () => {
  beforeEach(() => {
    // Reset mocks before each test
    jest.clearAllMocks();
  });

  describe('fetchUser', () => {
    test('fetches user data successfully', async () => {
      const mockUser = { id: 1, name: 'John Doe', email: 'john@example.com' };

      // Mock the fetch function
      global.fetch = jest.fn(() =>
        Promise.resolve({
          ok: true,
          json: () => Promise.resolve(mockUser),
        })
      );

      const user = await api.fetchUser(1);
      expect(user).toEqual(mockUser);
      expect(global.fetch).toHaveBeenCalledWith('/api/users/1');
    });

    test('handles fetch error', async () => {
      global.fetch = jest.fn(() =>
        Promise.reject(new Error('Network error'))
      );

      await expect(api.fetchUser(1)).rejects.toThrow('Network error');
    });

    test('handles HTTP error', async () => {
      global.fetch = jest.fn(() =>
        Promise.resolve({
          ok: false,
          status: 404,
          statusText: 'Not Found',
        })
      );

      await expect(api.fetchUser(1)).rejects.toThrow('HTTP 404: Not Found');
    });
  });

  describe('createUser', () => {
    test('creates user successfully', async () => {
      const newUser = { name: 'Jane Doe', email: 'jane@example.com' };
      const createdUser = { id: 2, ...newUser };

      global.fetch = jest.fn(() =>
        Promise.resolve({
          ok: true,
          json: () => Promise.resolve(createdUser),
        })
      );

      const result = await api.createUser(newUser);
      expect(result).toEqual(createdUser);
      expect(global.fetch).toHaveBeenCalledWith('/api/users', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(newUser),
      });
    });
  });
});
```

---

## 🧩 **Component Testing**

### **React Native Testing Library Setup**
```javascript
// jest.setup.js
import 'react-native-gesture-handler/jestSetup';
import mockAsyncStorage from '@react-native-async-storage/async-storage/jest/async-storage-mock';

jest.mock('@react-native-async-storage/async-storage', () => mockAsyncStorage);
jest.mock('react-native/Libraries/Animated/NativeAnimatedHelper');

// Mock react-native components that might cause issues
jest.mock('react-native/Libraries/Components/Switch/Switch', () => {
  const mockComponent = require('react-native/jest/mockComponent');
  return mockComponent('react-native/Libraries/Components/Switch/Switch');
});
```

### **Component Testing Examples**
```javascript
// __tests__/components/Button.test.js
import React from 'react';
import { render, fireEvent } from '@testing-library/react-native';
import Button from '../../src/components/Button';

describe('Button Component', () => {
  const mockOnPress = jest.fn();

  beforeEach(() => {
    mockOnPress.mockClear();
  });

  test('renders with title', () => {
    const { getByText } = render(<Button title="Click me" onPress={mockOnPress} />);
    expect(getByText('Click me')).toBeTruthy();
  });

  test('calls onPress when pressed', () => {
    const { getByText } = render(<Button title="Click me" onPress={mockOnPress} />);
    const button = getByText('Click me');

    fireEvent.press(button);
    expect(mockOnPress).toHaveBeenCalledTimes(1);
  });

  test('shows loading state', () => {
    const { getByText, queryByText } = render(
      <Button title="Click me" onPress={mockOnPress} loading />
    );

    expect(getByText('Loading...')).toBeTruthy();
    expect(queryByText('Click me')).toBeNull();
  });

  test('is disabled when loading', () => {
    const { getByText } = render(
      <Button title="Click me" onPress={mockOnPress} loading />
    );
    const button = getByText('Loading...');

    fireEvent.press(button);
    expect(mockOnPress).not.toHaveBeenCalled();
  });

  test('applies custom styles', () => {
    const customStyle = { backgroundColor: 'red' };
    const { getByTestId } = render(
      <Button
        title="Styled Button"
        onPress={mockOnPress}
        style={customStyle}
        testID="styled-button"
      />
    );

    const button = getByTestId('styled-button');
    expect(button.props.style).toContain(customStyle);
  });

  test('handles accessibility props', () => {
    const { getByA11yLabel } = render(
      <Button
        title="Accessible Button"
        onPress={mockOnPress}
        accessibilityLabel="Custom label"
      />
    );

    expect(getByA11yLabel('Custom label')).toBeTruthy();
  });
});
```

### **Hook Testing**
```javascript
// __tests__/hooks/useCounter.test.js
import { renderHook, act } from '@testing-library/react-hooks';
import useCounter from '../../src/hooks/useCounter';

describe('useCounter Hook', () => {
  test('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  test('initializes with custom value', () => {
    const { result } = renderHook(() => useCounter(5));
    expect(result.current.count).toBe(5);
  });

  test('increments count', () => {
    const { result } = renderHook(() => useCounter());
    act(() => {
      result.current.increment();
    });
    expect(result.current.count).toBe(1);
  });

  test('decrements count', () => {
    const { result } = renderHook(() => useCounter(5));
    act(() => {
      result.current.decrement();
    });
    expect(result.current.count).toBe(4);
  });

  test('resets count', () => {
    const { result } = renderHook(() => useCounter(10));
    act(() => {
      result.current.increment();
      result.current.reset();
    });
    expect(result.current.count).toBe(10);
  });

  test('respects minimum value', () => {
    const { result } = renderHook(() => useCounter(0, { min: 0 }));
    act(() => {
      result.current.decrement();
    });
    expect(result.current.count).toBe(0);
  });

  test('respects maximum value', () => {
    const { result } = renderHook(() => useCounter(10, { max: 10 }));
    act(() => {
      result.current.increment();
    });
    expect(result.current.count).toBe(10);
  });
});
```

---

## 🔗 **Integration Testing**

### **API Integration Testing**
```javascript
// __tests__/integration/userService.test.js
import userService from '../../src/services/userService';
import api from '../../src/api';

// Mock the API module
jest.mock('../../src/api');

describe('User Service Integration', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('getUserProfile', () => {
    test('fetches and transforms user profile', async () => {
      const mockApiResponse = {
        id: 1,
        first_name: 'John',
        last_name: 'Doe',
        email: 'john@example.com',
        avatar: 'avatar.jpg',
        created_at: '2023-01-01T00:00:00Z',
      };

      const expectedProfile = {
        id: 1,
        name: 'John Doe',
        email: 'john@example.com',
        avatar: 'avatar.jpg',
        joinDate: new Date('2023-01-01T00:00:00Z'),
      };

      api.get.mockResolvedValue(mockApiResponse);

      const profile = await userService.getUserProfile(1);

      expect(api.get).toHaveBeenCalledWith('/users/1');
      expect(profile).toEqual(expectedProfile);
    });

    test('handles API errors', async () => {
      const error = new Error('User not found');
      api.get.mockRejectedValue(error);

      await expect(userService.getUserProfile(999)).rejects.toThrow('User not found');
    });
  });

  describe('updateUserProfile', () => {
    test('updates profile and returns transformed data', async () => {
      const updateData = {
        first_name: 'Jane',
        last_name: 'Smith',
      };

      const mockApiResponse = {
        id: 1,
        first_name: 'Jane',
        last_name: 'Smith',
        email: 'jane@example.com',
        avatar: 'avatar.jpg',
        updated_at: '2023-01-02T00:00:00Z',
      };

      const expectedProfile = {
        id: 1,
        name: 'Jane Smith',
        email: 'jane@example.com',
        avatar: 'avatar.jpg',
        lastUpdated: new Date('2023-01-02T00:00:00Z'),
      };

      api.put.mockResolvedValue(mockApiResponse);

      const profile = await userService.updateUserProfile(1, updateData);

      expect(api.put).toHaveBeenCalledWith('/users/1', updateData);
      expect(profile).toEqual(expectedProfile);
    });
  });

  describe('getUserPosts', () => {
    test('fetches user posts with pagination', async () => {
      const mockPosts = [
        { id: 1, title: 'Post 1', content: 'Content 1' },
        { id: 2, title: 'Post 2', content: 'Content 2' },
      ];

      api.get.mockResolvedValue({
        posts: mockPosts,
        total: 2,
        page: 1,
        limit: 10,
      });

      const result = await userService.getUserPosts(1, { page: 1, limit: 10 });

      expect(api.get).toHaveBeenCalledWith('/users/1/posts', {
        params: { page: 1, limit: 10 },
      });

      expect(result).toEqual({
        posts: mockPosts,
        pagination: {
          total: 2,
          page: 1,
          limit: 10,
          hasNext: false,
          hasPrev: false,
        },
      });
    });
  });
});
```

### **Redux Integration Testing**
```javascript
// __tests__/integration/authSlice.test.js
import authReducer, {
  login,
  logout,
  updateProfile,
  clearError,
} from '../../src/store/slices/authSlice';
import { configureStore } from '@reduxjs/toolkit';

// Mock API calls
jest.mock('../../src/api/auth', () => ({
  loginUser: jest.fn(),
  logoutUser: jest.fn(),
  getUserProfile: jest.fn(),
  updateUserProfile: jest.fn(),
}));

import * as authApi from '../../src/api/auth';

describe('Auth Slice Integration', () => {
  let store;

  beforeEach(() => {
    store = configureStore({
      reducer: {
        auth: authReducer,
      },
    });
    jest.clearAllMocks();
  });

  describe('login', () => {
    test('handles successful login', async () => {
      const mockUser = {
        id: 1,
        name: 'John Doe',
        email: 'john@example.com',
      };

      const mockToken = 'mock-jwt-token';

      authApi.loginUser.mockResolvedValue({
        user: mockUser,
        token: mockToken,
      });

      await store.dispatch(login({
        email: 'john@example.com',
        password: 'password123',
      }));

      const state = store.getState().auth;
      expect(state.user).toEqual(mockUser);
      expect(state.token).toBe(mockToken);
      expect(state.isAuthenticated).toBe(true);
      expect(state.loading).toBe(false);
      expect(state.error).toBeNull();
    });

    test('handles login failure', async () => {
      const errorMessage = 'Invalid credentials';

      authApi.loginUser.mockRejectedValue(new Error(errorMessage));

      await store.dispatch(login({
        email: 'john@example.com',
        password: 'wrongpassword',
      }));

      const state = store.getState().auth;
      expect(state.user).toBeNull();
      expect(state.token).toBeNull();
      expect(state.isAuthenticated).toBe(false);
      expect(state.loading).toBe(false);
      expect(state.error).toBe(errorMessage);
    });
  });

  describe('logout', () => {
    test('handles logout', async () => {
      // First login
      const mockUser = { id: 1, name: 'John Doe' };
      authApi.loginUser.mockResolvedValue({
        user: mockUser,
        token: 'token',
      });

      await store.dispatch(login({
        email: 'john@example.com',
        password: 'password123',
      }));

      // Then logout
      authApi.logoutUser.mockResolvedValue();

      await store.dispatch(logout());

      const state = store.getState().auth;
      expect(state.user).toBeNull();
      expect(state.token).toBeNull();
      expect(state.isAuthenticated).toBe(false);
      expect(state.loading).toBe(false);
    });
  });

  describe('updateProfile', () => {
    test('handles profile update', async () => {
      // Setup initial state
      const initialUser = { id: 1, name: 'John Doe', email: 'john@example.com' };
      const updatedUser = { id: 1, name: 'John Smith', email: 'john@example.com' };

      store = configureStore({
        reducer: {
          auth: authReducer,
        },
        preloadedState: {
          auth: {
            user: initialUser,
            token: 'token',
            isAuthenticated: true,
            loading: false,
            error: null,
          },
        },
      });

      authApi.updateUserProfile.mockResolvedValue(updatedUser);

      await store.dispatch(updateProfile({
        name: 'John Smith',
      }));

      const state = store.getState().auth;
      expect(state.user).toEqual(updatedUser);
      expect(state.loading).toBe(false);
      expect(state.error).toBeNull();
    });
  });

  describe('clearError', () => {
    test('clears error state', () => {
      store = configureStore({
        reducer: {
          auth: authReducer,
        },
        preloadedState: {
          auth: {
            user: null,
            token: null,
            isAuthenticated: false,
            loading: false,
            error: 'Some error occurred',
          },
        },
      });

      store.dispatch(clearError());

      const state = store.getState().auth;
      expect(state.error).toBeNull();
    });
  });
});
```

---

## 📱 **End-to-End Testing**

### **Detox Setup**
```javascript
// .detoxrc.js
module.exports = {
  testRunner: 'jest',
  runnerConfig: 'e2e/config.js',
  configurations: {
    'ios.simulator': {
      type: 'ios.simulator',
      device: {
        type: 'iPhone 13',
      },
      app: 'ios.build',
    },
    'android.emulator': {
      type: 'android.emulator',
      device: {
        avdName: 'Pixel_4_API_30',
      },
      app: 'android.build',
    },
  },
};
```

### **Detox Test Examples**
```javascript
// e2e/firstTest.e2e.js
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

    // Enter credentials
    await element(by.id('email-input')).typeText('user@example.com');
    await element(by.id('password-input')).typeText('password123');

    // Submit login
    await element(by.id('submit-login')).tap();

    // Verify successful login
    await expect(element(by.id('welcome-message'))).toBeVisible();
    await expect(element(by.text('Welcome, User!'))).toBeVisible();
  });

  it('should show error for invalid credentials', async () => {
    await element(by.id('login-button')).tap();

    await element(by.id('email-input')).typeText('invalid@example.com');
    await element(by.id('password-input')).typeText('wrongpassword');

    await element(by.id('submit-login')).tap();

    // Verify error message
    await expect(element(by.text('Invalid credentials'))).toBeVisible();
  });

  it('should navigate to profile after login', async () => {
    // Login first
    await element(by.id('login-button')).tap();
    await element(by.id('email-input')).typeText('user@example.com');
    await element(by.id('password-input')).typeText('password123');
    await element(by.id('submit-login')).tap();

    // Navigate to profile
    await element(by.id('profile-tab')).tap();

    // Verify profile screen
    await expect(element(by.id('profile-screen'))).toBeVisible();
    await expect(element(by.id('user-avatar'))).toBeVisible();
  });
});
```

### **Maestro Setup**
```yaml
# .maestro/login-flow.yaml
appId: com.example.app
---
- launchApp
- tapOn: "Login"
- inputText: "user@example.com"
- tapOn: "Email"
- inputText: "password123"
- tapOn: "Password"
- tapOn: "Sign In"
- assertVisible: "Welcome, User!"
```

---

## 🎭 **Mocking & Test Utilities**

### **Custom Test Utilities**
```javascript
// __tests__/utils/testUtils.js
import React from 'react';
import { render } from '@testing-library/react-native';
import { Provider } from 'react-redux';
import { NavigationContainer } from '@react-navigation/native';
import configureStore from 'redux-mock-store';
import thunk from 'redux-thunk';

// Create mock store
const middlewares = [thunk];
const mockStore = configureStore(middlewares);

// Custom render function with providers
const AllTheProviders = ({ children, initialState = {}, store }) => {
  const mockStoreInstance = store || mockStore(initialState);

  return (
    <Provider store={mockStoreInstance}>
      <NavigationContainer>
        {children}
      </NavigationContainer>
    </Provider>
  );
};

const customRender = (ui, options = {}) => {
  const { initialState = {}, store, ...renderOptions } = options;

  const Wrapper = ({ children }) => (
    <AllTheProviders initialState={initialState} store={store}>
      {children}
    </AllTheProviders>
  );

  return render(ui, { wrapper: Wrapper, ...renderOptions });
};

// Mock API responses
export const mockApiResponse = {
  success: (data) => ({
    ok: true,
    status: 200,
    json: () => Promise.resolve(data),
  }),

  error: (message, status = 500) => ({
    ok: false,
    status,
    statusText: message,
  }),
};

// Mock navigation
export const mockNavigation = {
  navigate: jest.fn(),
  goBack: jest.fn(),
  push: jest.fn(),
  replace: jest.fn(),
  reset: jest.fn(),
  setParams: jest.fn(),
};

// Mock route
export const mockRoute = {
  params: {},
  name: 'TestScreen',
  key: 'test-key',
};

// Generate test data
export const createMockUser = (overrides = {}) => ({
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  avatar: 'avatar.jpg',
  createdAt: new Date().toISOString(),
  ...overrides,
});

export const createMockPost = (overrides = {}) => ({
  id: 1,
  title: 'Test Post',
  content: 'This is a test post content',
  author: createMockUser(),
  createdAt: new Date().toISOString(),
  likes: 10,
  comments: 5,
  ...overrides,
});

// Async utilities
export const waitForNextTick = () => new Promise(resolve => setTimeout(resolve, 0));

export const waitForMs = (ms) => new Promise(resolve => setTimeout(resolve, ms));

// Re-export everything
export * from '@testing-library/react-native';
export { customRender as render };
```

### **Mock Implementations**
```javascript
// __tests__/mocks/react-native-geolocation-service.js
const mockGeolocation = {
  getCurrentPosition: jest.fn(),
  watchPosition: jest.fn(),
  clearWatch: jest.fn(),
  stopObserving: jest.fn(),
};

mockGeolocation.getCurrentPosition.mockImplementation((success, error, options) => {
  const mockPosition = {
    coords: {
      latitude: 37.78825,
      longitude: -122.4324,
      accuracy: 5,
      altitude: 100,
      heading: 0,
      speed: 0,
    },
    timestamp: Date.now(),
  };

  if (success) {
    success(mockPosition);
  }
});

mockGeolocation.watchPosition.mockImplementation((success, error, options) => {
  const mockPosition = {
    coords: {
      latitude: 37.78825,
      longitude: -122.4324,
      accuracy: 5,
    },
    timestamp: Date.now(),
  };

  if (success) {
    success(mockPosition);
  }

  return 'mock-watch-id';
});

export default mockGeolocation;
```

---

## 🏗️ **Test-Driven Development**

### **TDD Example**
```javascript
// src/utils/stringUtils.js
// First, write the test
// __tests__/utils/stringUtils.test.js

describe('String Utils', () => {
  describe('capitalize', () => {
    test('capitalizes first letter of string', () => {
      // This will fail initially - that's expected in TDD
      expect(capitalize('hello')).toBe('Hello');
    });

    test('handles empty string', () => {
      expect(capitalize('')).toBe('');
    });

    test('handles single character', () => {
      expect(capitalize('a')).toBe('A');
    });

    test('leaves already capitalized strings unchanged', () => {
      expect(capitalize('Hello')).toBe('Hello');
    });
  });

  describe('truncate', () => {
    test('truncates string longer than max length', () => {
      expect(truncate('Hello World', 5)).toBe('Hello...');
    });

    test('returns original string if shorter than max length', () => {
      expect(truncate('Hi', 5)).toBe('Hi');
    });

    test('handles custom suffix', () => {
      expect(truncate('Hello World', 5, '***')).toBe('Hello***');
    });
  });

  describe('isEmail', () => {
    test('validates correct email format', () => {
      expect(isEmail('user@example.com')).toBe(true);
      expect(isEmail('test.email+tag@domain.co.uk')).toBe(true);
    });

    test('rejects invalid email format', () => {
      expect(isEmail('invalid-email')).toBe(false);
      expect(isEmail('@example.com')).toBe(false);
      expect(isEmail('user@')).toBe(false);
    });
  });
});

// Now implement the functions to make tests pass
// src/utils/stringUtils.js

export const capitalize = (str) => {
  if (!str) return '';
  return str.charAt(0).toUpperCase() + str.slice(1);
};

export const truncate = (str, maxLength, suffix = '...') => {
  if (str.length <= maxLength) return str;
  return str.slice(0, maxLength - suffix.length) + suffix;
};

export const isEmail = (email) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};
```

### **TDD Workflow**
```javascript
// Example: Building a Todo List with TDD

// Step 1: Write failing test
describe('TodoList', () => {
  test('starts with empty list', () => {
    const todoList = new TodoList();
    expect(todoList.getAll()).toEqual([]);
  });

  test('adds todo item', () => {
    const todoList = new TodoList();
    todoList.add('Buy groceries');
    expect(todoList.getAll()).toEqual([
      { id: 1, text: 'Buy groceries', completed: false }
    ]);
  });
});

// Step 2: Implement minimal code to pass
class TodoList {
  constructor() {
    this.todos = [];
    this.nextId = 1;
  }

  getAll() {
    return this.todos;
  }

  add(text) {
    this.todos.push({
      id: this.nextId++,
      text,
      completed: false,
    });
  }
}

// Step 3: Write next failing test
test('toggles todo completion', () => {
  const todoList = new TodoList();
  todoList.add('Buy groceries');
  todoList.toggle(1);
  expect(todoList.getAll()[0].completed).toBe(true);
  todoList.toggle(1);
  expect(todoList.getAll()[0].completed).toBe(false);
});

// Step 4: Implement toggle method
toggle(id) {
  const todo = this.todos.find(t => t.id === id);
  if (todo) {
    todo.completed = !todo.completed;
  }
}

// Continue this pattern...
```

---

## 🔄 **Continuous Integration**

### **GitHub Actions Setup**
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x, 18.x]

    steps:
    - uses: actions/checkout@v3

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linting
      run: npm run lint

    - name: Run type checking
      run: npm run type-check

    - name: Run unit tests
      run: npm run test:unit

    - name: Run integration tests
      run: npm run test:integration

    - name: Upload coverage reports
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info

  e2e-test:
    runs-on: ubuntu-latest
    needs: test

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 18.x
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Cache detox dependencies
      uses: actions/cache@v3
      with:
        path: ~/.detox
        key: detox-${{ runner.os }}-${{ hashFiles('package-lock.json') }}

    - name: Run E2E tests
      uses: wix/Detox/instruments@v1
      with:
        detox-configuration: ios.simulator.debug
        detox-logs: all
        detox-record-logs: fail
        detox-record-videos: fail
```

### **Fastlane Setup**
```ruby
# fastlane/Fastfile
platform :ios do
  desc "Run iOS tests"
  lane :test do
    run_tests(
      scheme: "MyApp",
      device: "iPhone 13",
      clean: true,
      code_coverage: true
    )
  end

  desc "Build and deploy to TestFlight"
  lane :beta do
    test
    build_app(scheme: "MyApp")
    upload_to_testflight
  end
end

platform :android do
  desc "Run Android tests"
  lane :test do
    gradle(task: "test")
  end

  desc "Build and deploy to Play Store Beta"
  lane :beta do
    test
    gradle(task: "bundle", build_type: "Release")
    upload_to_play_store(track: "beta")
  end
end
```

---

## ⚡ **Performance Testing**

### **Performance Test Setup**
```javascript
// __tests__/performance/TodoList.perf.test.js
import React from 'react';
import { render } from '@testing-library/react-native';
import { Provider } from 'react-redux';
import configureStore from 'redux-mock-store';
import TodoList from '../../src/components/TodoList';

const mockStore = configureStore([]);

describe('TodoList Performance', () => {
  test('renders large list efficiently', () => {
    const largeTodoList = Array.from({ length: 1000 }, (_, i) => ({
      id: i + 1,
      text: `Todo item ${i + 1}`,
      completed: i % 2 === 0,
    }));

    const store = mockStore({
      todos: {
        items: largeTodoList,
        loading: false,
      },
    });

    const startTime = performance.now();

    render(
      <Provider store={store}>
        <TodoList />
      </Provider>
    );

    const endTime = performance.now();
    const renderTime = endTime - startTime;

    // Assert render time is acceptable (less than 100ms)
    expect(renderTime).toBeLessThan(100);
  });

  test('handles rapid updates efficiently', async () => {
    const store = mockStore({
      todos: {
        items: [],
        loading: false,
      },
    });

    const { rerender } = render(
      <Provider store={store}>
        <TodoList />
      </Provider>
    );

    const startTime = performance.now();

    // Simulate rapid updates
    for (let i = 0; i < 100; i++) {
      const updatedStore = mockStore({
        todos: {
          items: Array.from({ length: i + 1 }, (_, j) => ({
            id: j + 1,
            text: `Updated todo ${j + 1}`,
            completed: false,
          })),
          loading: false,
        },
      });

      rerender(
        <Provider store={updatedStore}>
          <TodoList />
        </Provider>
      );
    }

    const endTime = performance.now();
    const totalTime = endTime - startTime;

    // Assert total time for 100 updates is acceptable
    expect(totalTime).toBeLessThan(500);
  });
});
```

### **Memory Leak Testing**
```javascript
// __tests__/memory/ComponentMemory.test.js
import React from 'react';
import { render, cleanup } from '@testing-library/react-native';
import { act } from 'react-test-renderer';
import TimerComponent from '../../src/components/TimerComponent';

describe('Component Memory Tests', () => {
  test('cleans up timers on unmount', () => {
    jest.useFakeTimers();

    const mockCallback = jest.fn();
    const { unmount } = render(<TimerComponent callback={mockCallback} />);

    // Fast-forward time
    act(() => {
      jest.advanceTimersByTime(1000);
    });

    expect(mockCallback).toHaveBeenCalledTimes(1);

    // Unmount component
    unmount();

    // Fast-forward time again
    act(() => {
      jest.advanceTimersByTime(1000);
    });

    // Callback should not be called again after unmount
    expect(mockCallback).toHaveBeenCalledTimes(1);

    jest.useRealTimers();
  });

  test('cleans up event listeners', () => {
    const mockEventListener = jest.fn();
    const mockRemoveListener = jest.fn();

    // Mock a component that adds event listeners
    const TestComponent = () => {
      React.useEffect(() => {
        mockEventListener();
        return () => mockRemoveListener();
      }, []);

      return null;
    };

    const { unmount } = render(<TestComponent />);

    expect(mockEventListener).toHaveBeenCalledTimes(1);
    expect(mockRemoveListener).toHaveBeenCalledTimes(0);

    unmount();

    expect(mockRemoveListener).toHaveBeenCalledTimes(1);
  });
});
```

---

## 🛠️ **Code Quality Tools**

### **ESLint Configuration**
```javascript
// .eslintrc.js
module.exports = {
  root: true,
  extends: [
    '@react-native-community',
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'prettier',
  ],
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint', 'react', 'react-native', 'jest'],
  env: {
    'jest/globals': true,
    'react-native/react-native': true,
  },
  rules: {
    // React rules
    'react/prop-types': 'off',
    'react/display-name': 'off',

    // React Native rules
    'react-native/no-unused-styles': 'error',
    'react-native/split-platform-components': 'error',
    'react-native/no-inline-styles': 'warn',
    'react-native/no-color-literals': 'warn',

    // Jest rules
    'jest/no-disabled-tests': 'warn',
    'jest/no-focused-tests': 'error',
    'jest/no-identical-title': 'error',
    'jest/prefer-to-have-length': 'warn',
    'jest/valid-expect': 'error',

    // General rules
    'no-console': 'warn',
    'no-debugger': 'error',
    'prefer-const': 'error',
    'no-var': 'error',
  },
  settings: {
    react: {
      version: 'detect',
    },
  },
};
```

### **Prettier Configuration**
```javascript
// .prettierrc.js
module.exports = {
  semi: true,
  trailingComma: 'es5',
  singleQuote: true,
  printWidth: 80,
  tabWidth: 2,
  useTabs: false,
  bracketSpacing: true,
  arrowParens: 'avoid',
  endOfLine: 'lf',
};
```

### **Husky for Git Hooks**
```javascript
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "git add"
    ],
    "*.{json,md}": [
      "prettier --write",
      "git add"
    ]
  }
}
```

---

## 🎯 **Practical Examples**

### **Complete Test Suite Example**
```javascript
// __tests__/TodoApp.test.js
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { Provider } from 'react-redux';
import configureStore from 'redux-mock-store';
import thunk from 'redux-thunk';
import TodoApp from '../src/components/TodoApp';

// Mock API
jest.mock('../src/api/todos', () => ({
  fetchTodos: jest.fn(),
  createTodo: jest.fn(),
  updateTodo: jest.fn(),
  deleteTodo: jest.fn(),
}));

import * as todoApi from '../src/api/todos';

const middlewares = [thunk];
const mockStore = configureStore(middlewares);

describe('TodoApp Integration Test', () => {
  let store;

  beforeEach(() => {
    store = mockStore({
      todos: {
        items: [],
        loading: false,
        error: null,
      },
      auth: {
        user: { id: 1, name: 'Test User' },
        isAuthenticated: true,
      },
    });
    jest.clearAllMocks();
  });

  test('loads todos on mount', async () => {
    const mockTodos = [
      { id: 1, text: 'Test todo 1', completed: false },
      { id: 2, text: 'Test todo 2', completed: true },
    ];

    todoApi.fetchTodos.mockResolvedValue(mockTodos);

    render(
      <Provider store={store}>
        <TodoApp />
      </Provider>
    );

    await waitFor(() => {
      expect(todoApi.fetchTodos).toHaveBeenCalledTimes(1);
    });
  });

  test('creates new todo', async () => {
    const newTodo = { id: 3, text: 'New todo', completed: false };
    todoApi.createTodo.mockResolvedValue(newTodo);

    const { getByPlaceholderText, getByText } = render(
      <Provider store={store}>
        <TodoApp />
      </Provider>
    );

    const input = getByPlaceholderText('Add new todo...');
    const addButton = getByText('Add');

    fireEvent.changeText(input, 'New todo');
    fireEvent.press(addButton);

    await waitFor(() => {
      expect(todoApi.createTodo).toHaveBeenCalledWith({
        text: 'New todo',
        userId: 1,
      });
    });
  });

  test('toggles todo completion', async () => {
    const mockTodos = [{ id: 1, text: 'Test todo', completed: false }];
    const updatedTodo = { id: 1, text: 'Test todo', completed: true };

    store = mockStore({
      todos: {
        items: mockTodos,
        loading: false,
        error: null,
      },
      auth: {
        user: { id: 1, name: 'Test User' },
        isAuthenticated: true,
      },
    });

    todoApi.updateTodo.mockResolvedValue(updatedTodo);

    const { getByText } = render(
      <Provider store={store}>
        <TodoApp />
      </Provider>
    );

    const todoItem = getByText('Test todo');
    fireEvent.press(todoItem);

    await waitFor(() => {
      expect(todoApi.updateTodo).toHaveBeenCalledWith(1, {
        completed: true,
      });
    });
  });

  test('deletes todo', async () => {
    const mockTodos = [{ id: 1, text: 'Test todo', completed: false }];

    store = mockStore({
      todos: {
        items: mockTodos,
        loading: false,
        error: null,
      },
      auth: {
        user: { id: 1, name: 'Test User' },
        isAuthenticated: true,
      },
    });

    todoApi.deleteTodo.mockResolvedValue();

    const { getByText } = render(
      <Provider store={store}>
        <TodoApp />
      </Provider>
    );

    const deleteButton = getByText('Delete');
    fireEvent.press(deleteButton);

    await waitFor(() => {
      expect(todoApi.deleteTodo).toHaveBeenCalledWith(1);
    });
  });

  test('handles API errors gracefully', async () => {
    todoApi.fetchTodos.mockRejectedValue(new Error('Network error'));

    const { getByText } = render(
      <Provider store={store}>
        <TodoApp />
      </Provider>
    );

    await waitFor(() => {
      expect(getByText('Failed to load todos')).toBeTruthy();
    });
  });

  test('shows loading state', () => {
    store = mockStore({
      todos: {
        items: [],
        loading: true,
        error: null,
      },
      auth: {
        user: { id: 1, name: 'Test User' },
        isAuthenticated: true,
      },
    });

    const { getByText } = render(
      <Provider store={store}>
        <TodoApp />
      </Provider>
    );

    expect(getByText('Loading todos...')).toBeTruthy();
  });
});
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Unit Testing**: Testing individual functions and components with Jest
- ✅ **Component Testing**: Testing React Native components with Testing Library
- ✅ **Integration Testing**: Testing how different parts work together
- ✅ **End-to-End Testing**: Testing complete user workflows with Detox
- ✅ **Mocking**: Creating mock implementations for external dependencies
- ✅ **Test-Driven Development**: Writing tests before implementation
- ✅ **Continuous Integration**: Automated testing with GitHub Actions
- ✅ **Performance Testing**: Testing app speed and memory usage
- ✅ **Code Quality**: Linting, formatting, and pre-commit hooks

### **Best Practices**
1. **Write tests for all new features** and bug fixes
2. **Use descriptive test names** that explain what is being tested
3. **Test both success and error scenarios**
4. **Mock external dependencies** to isolate unit tests
5. **Use test utilities** to reduce boilerplate code
6. **Run tests frequently** during development
7. **Maintain test coverage** above acceptable thresholds
8. **Test on real devices** for E2E scenarios

### **Next Steps**
- Set up CI/CD pipeline for automated testing
- Implement visual regression testing
- Add performance monitoring and alerting
- Create comprehensive test documentation
- Establish testing standards and guidelines

---

## 🎯 **Assignment**

### **Task 1: Unit Test Suite**
Create comprehensive unit tests for:
- Utility functions (string, date, math operations)
- Custom hooks (useCounter, useApi, useStorage)
- Redux actions and reducers
- API service functions

###