# Lesson 40: Testing & Quality Assurance

## 🎯 **Learning Objectives**
- Master testing strategies for React Native applications
- Implement unit tests, integration tests, and end-to-end tests
- Set up automated testing pipelines
- Write testable code and maintain high code quality
- Use testing utilities and mocking frameworks

## 📚 **Table of Contents**
1. [Testing Fundamentals](#testing-fundamentals)
2. [Unit Testing](#unit-testing)
3. [Component Testing](#component-testing)
4. [Integration Testing](#integration-testing)
5. [End-to-End Testing](#end-to-end-testing)
6. [Mocking & Test Utilities](#mocking--test-utilities)
7. [Test-Driven Development](#test-driven-development)
8. [Automated Testing](#automated-testing)
9. [Code Quality Tools](#code-quality-tools)
10. [Practical Examples](#practical-examples)

---

## 🧪 **Testing Fundamentals**

### **Testing Pyramid**
```
End-to-End Tests (E2E)
    ↕️
Integration Tests
    ↕️
Unit Tests
```

### **Types of Tests**
- **Unit Tests**: Test individual functions/components in isolation
- **Integration Tests**: Test how components work together
- **End-to-End Tests**: Test complete user workflows
- **Snapshot Tests**: Test UI components for visual regressions
- **Performance Tests**: Test app performance under load

### **Testing Best Practices**
- ✅ **Write tests first** (TDD approach)
- ✅ **Test behavior, not implementation**
- ✅ **Keep tests fast and reliable**
- ✅ **Use descriptive test names**
- ✅ **Test edge cases and error conditions**
- ✅ **Maintain test coverage above 80%**

---

## 🧩 **Unit Testing**

### **Jest Setup**
```javascript
// __tests__/setup.js
import 'react-native-gesture-handler/jestSetup';

// Mock AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () =>
  require('@react-native-async-storage/async-storage/jest/async-storage-mock')
);

// Mock react-native-reanimated
jest.mock('react-native-reanimated', () => {
  const Reanimated = require('react-native-reanimated/mock');
  return Reanimated;
});

// Mock react-native-vector-icons
jest.mock('react-native-vector-icons', () => ({
  createIconSet: () => 'IconMock',
}));

// Mock react-navigation
jest.mock('@react-navigation/native', () => ({
  useNavigation: () => ({
    navigate: jest.fn(),
    goBack: jest.fn(),
  }),
  useRoute: () => ({
    params: {},
  }),
}));

// Global test setup
beforeEach(() => {
  jest.clearAllMocks();
});

// Mock timers
jest.useFakeTimers();
```

### **Basic Unit Tests**
```javascript
// __tests__/utils/math.test.js
import { add, subtract, multiply, divide, fibonacci } from '../../utils/math';

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
    test('multiplies two numbers', () => {
      expect(multiply(3, 4)).toBe(12);
    });

    test('multiplies with zero', () => {
      expect(multiply(0, 5)).toBe(0);
    });

    test('multiplies negative numbers', () => {
      expect(multiply(-2, 3)).toBe(-6);
    });
  });

  describe('divide', () => {
    test('divides two numbers', () => {
      expect(divide(10, 2)).toBe(5);
    });

    test('divides with decimal result', () => {
      expect(divide(5, 2)).toBe(2.5);
    });

    test('throws error when dividing by zero', () => {
      expect(() => divide(10, 0)).toThrow('Cannot divide by zero');
    });
  });

  describe('fibonacci', () => {
    test('returns correct fibonacci sequence', () => {
      expect(fibonacci(0)).toBe(0);
      expect(fibonacci(1)).toBe(1);
      expect(fibonacci(2)).toBe(1);
      expect(fibonacci(3)).toBe(2);
      expect(fibonacci(4)).toBe(3);
      expect(fibonacci(5)).toBe(5);
      expect(fibonacci(6)).toBe(8);
    });

    test('throws error for negative numbers', () => {
      expect(() => fibonacci(-1)).toThrow('Input must be a non-negative integer');
    });
  });
});
```

### **Async Function Testing**
```javascript
// __tests__/services/api.test.js
import apiService from '../../services/apiService';

// Mock fetch
global.fetch = jest.fn();

describe('API Service', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('getUser', () => {
    test('fetches user successfully', async () => {
      const mockUser = {
        id: 1,
        name: 'John Doe',
        email: 'john@example.com',
      };

      fetch.mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve(mockUser),
      });

      const result = await apiService.getUser(1);

      expect(fetch).toHaveBeenCalledWith('/api/users/1', {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
        },
      });

      expect(result).toEqual(mockUser);
    });

    test('handles network error', async () => {
      fetch.mockRejectedValueOnce(new Error('Network error'));

      await expect(apiService.getUser(1)).rejects.toThrow('Network error');
    });

    test('handles 404 error', async () => {
      fetch.mockResolvedValueOnce({
        ok: false,
        status: 404,
        json: () => Promise.resolve({ message: 'User not found' }),
      });

      await expect(apiService.getUser(999)).rejects.toThrow('User not found');
    });
  });

  describe('createUser', () => {
    test('creates user successfully', async () => {
      const newUser = {
        name: 'Jane Doe',
        email: 'jane@example.com',
      };

      const createdUser = {
        id: 2,
        ...newUser,
        createdAt: '2023-01-01T00:00:00Z',
      };

      fetch.mockResolvedValueOnce({
        ok: true,
        json: () => Promise.resolve(createdUser),
      });

      const result = await apiService.createUser(newUser);

      expect(fetch).toHaveBeenCalledWith('/api/users', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(newUser),
      });

      expect(result).toEqual(createdUser);
    });
  });
});
```

---

## 🧩 **Component Testing**

### **React Native Testing Library Setup**
```javascript
// __tests__/components/Button.test.js
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import Button from '../../components/Button';

describe('Button Component', () => {
  const defaultProps = {
    title: 'Click me',
    onPress: jest.fn(),
  };

  test('renders correctly', () => {
    const { getByText } = render(<Button {...defaultProps} />);
    expect(getByText('Click me')).toBeTruthy();
  });

  test('calls onPress when pressed', () => {
    const mockOnPress = jest.fn();
    const { getByText } = render(
      <Button {...defaultProps} onPress={mockOnPress} />
    );

    fireEvent.press(getByText('Click me'));
    expect(mockOnPress).toHaveBeenCalledTimes(1);
  });

  test('shows loading state', () => {
    const { getByText, queryByText } = render(
      <Button {...defaultProps} loading />
    );

    expect(getByText('Loading...')).toBeTruthy();
    expect(queryByText('Click me')).toBeNull();
  });

  test('is disabled when loading', () => {
    const mockOnPress = jest.fn();
    const { getByText } = render(
      <Button {...defaultProps} onPress={mockOnPress} loading />
    );

    fireEvent.press(getByText('Loading...'));
    expect(mockOnPress).not.toHaveBeenCalled();
  });

  test('applies custom styles', () => {
    const customStyle = { backgroundColor: 'red' };
    const { getByTestId } = render(
      <Button {...defaultProps} style={customStyle} testID="custom-button" />
    );

    const button = getByTestId('custom-button');
    expect(button.props.style).toContain(customStyle);
  });

  test('handles different variants', () => {
    const { getByText, rerender } = render(
      <Button {...defaultProps} variant="primary" />
    );

    expect(getByText('Click me')).toBeTruthy();

    rerender(<Button {...defaultProps} variant="secondary" />);
    expect(getByText('Click me')).toBeTruthy();
  });
});
```

### **Hook Testing**
```javascript
// __tests__/hooks/useCounter.test.js
import { renderHook, act } from '@testing-library/react-native';
import { useCounter } from '../../hooks/useCounter';

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
      result.current.increment();
    });

    expect(result.current.count).toBe(12);

    act(() => {
      result.current.reset();
    });

    expect(result.current.count).toBe(10);
  });

  test('respects min and max bounds', () => {
    const { result } = renderHook(() => useCounter(0, { min: 0, max: 5 }));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(0); // Should not go below min

    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.increment();
      result.current.increment();
      result.current.increment();
      result.current.increment();
    });

    expect(result.current.count).toBe(5); // Should not go above max
  });
});
```

### **Snapshot Testing**
```javascript
// __tests__/components/Card.test.js
import React from 'react';
import { render } from '@testing-library/react-native';
import Card from '../../components/Card';

describe('Card Component', () => {
  test('renders correctly', () => {
    const { toJSON } = render(
      <Card title="Test Card" description="This is a test card" />
    );

    expect(toJSON()).toMatchSnapshot();
  });

  test('renders with custom content', () => {
    const { toJSON } = render(
      <Card title="Custom Card">
        <Text>Custom content</Text>
      </Card>
    );

    expect(toJSON()).toMatchSnapshot();
  });

  test('renders in loading state', () => {
    const { toJSON } = render(
      <Card title="Loading Card" loading />
    );

    expect(toJSON()).toMatchSnapshot();
  });
});
```

---

## 🔗 **Integration Testing**

### **API Integration Testing**
```javascript
// __tests__/integration/apiIntegration.test.js
import { renderHook, waitFor } from '@testing-library/react-native';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useUser } from '../../hooks/useUser';

// Mock the API service
jest.mock('../../services/apiService', () => ({
  getUser: jest.fn(),
}));

const apiService = require('../../services/apiService');

describe('API Integration', () => {
  let queryClient;

  beforeEach(() => {
    queryClient = new QueryClient({
      defaultOptions: {
        queries: {
          retry: false,
        },
      },
    });
    jest.clearAllMocks();
  });

  const wrapper = ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );

  test('fetches user data successfully', async () => {
    const mockUser = {
      id: 1,
      name: 'John Doe',
      email: 'john@example.com',
    };

    apiService.getUser.mockResolvedValue(mockUser);

    const { result } = renderHook(() => useUser(1), { wrapper });

    expect(result.current.isLoading).toBe(true);

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });

    expect(result.current.data).toEqual(mockUser);
    expect(apiService.getUser).toHaveBeenCalledWith(1);
  });

  test('handles API error', async () => {
    const errorMessage = 'User not found';
    apiService.getUser.mockRejectedValue(new Error(errorMessage));

    const { result } = renderHook(() => useUser(999), { wrapper });

    await waitFor(() => {
      expect(result.current.isError).toBe(true);
    });

    expect(result.current.error.message).toBe(errorMessage);
  });

  test('caches user data', async () => {
    const mockUser = {
      id: 1,
      name: 'John Doe',
      email: 'john@example.com',
    };

    apiService.getUser.mockResolvedValue(mockUser);

    // First call
    const { result: result1, rerender } = renderHook(() => useUser(1), { wrapper });

    await waitFor(() => {
      expect(result1.current.isLoading).toBe(false);
    });

    expect(apiService.getUser).toHaveBeenCalledTimes(1);

    // Second call with same ID should use cache
    const { result: result2 } = renderHook(() => useUser(1), { wrapper });

    expect(result2.current.data).toEqual(mockUser);
    expect(apiService.getUser).toHaveBeenCalledTimes(1); // Still 1 call
  });
});
```

### **Navigation Integration Testing**
```javascript
// __tests__/integration/navigation.test.js
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';

// Mock screens
const HomeScreen = ({ navigation }) => (
  <View>
    <Text>Home Screen</Text>
    <TouchableOpacity onPress={() => navigation.navigate('Profile')}>
      <Text>Go to Profile</Text>
    </TouchableOpacity>
  </View>
);

const ProfileScreen = ({ navigation }) => (
  <View>
    <Text>Profile Screen</Text>
    <TouchableOpacity onPress={() => navigation.goBack()}>
      <Text>Go Back</Text>
    </TouchableOpacity>
  </View>
);

const Stack = createStackNavigator();

const AppNavigator = () => (
  <NavigationContainer>
    <Stack.Navigator>
      <Stack.Screen name="Home" component={HomeScreen} />
      <Stack.Screen name="Profile" component={ProfileScreen} />
    </Stack.Navigator>
  </NavigationContainer>
);

describe('Navigation Integration', () => {
  test('navigates between screens', async () => {
    const { getByText, queryByText } = render(<AppNavigator />);

    // Should start on Home screen
    expect(getByText('Home Screen')).toBeTruthy();
    expect(queryByText('Profile Screen')).toBeNull();

    // Navigate to Profile
    fireEvent.press(getByText('Go to Profile'));

    await waitFor(() => {
      expect(getByText('Profile Screen')).toBeTruthy();
      expect(queryByText('Home Screen')).toBeNull();
    });

    // Go back to Home
    fireEvent.press(getByText('Go Back'));

    await waitFor(() => {
      expect(getByText('Home Screen')).toBeTruthy();
      expect(queryByText('Profile Screen')).toBeNull();
    });
  });
});
```

---

## 🤖 **End-to-End Testing**

### **Detox Setup**
```javascript
// e2e/firstTest.e2e.js
import { device, expect, element, by } from 'detox';

describe('Example App', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  test('should show welcome message', async () => {
    await expect(element(by.text('Welcome to MyApp'))).toBeVisible();
  });

  test('should navigate to profile screen', async () => {
    await element(by.text('Go to Profile')).tap();
    await expect(element(by.text('Profile Screen'))).toBeVisible();
  });

  test('should handle user login', async () => {
    // Navigate to login
    await element(by.text('Login')).tap();

    // Fill login form
    await element(by.id('email-input')).typeText('test@example.com');
    await element(by.id('password-input')).typeText('password123');

    // Submit form
    await element(by.text('Sign In')).tap();

    // Verify login success
    await expect(element(by.text('Welcome back!'))).toBeVisible();
  });

  test('should add item to cart', async () => {
    // Navigate to products
    await element(by.text('Products')).tap();

    // Select first product
    await element(by.id('product-1')).tap();

    // Add to cart
    await element(by.text('Add to Cart')).tap();

    // Verify cart badge
    await expect(element(by.text('1'))).toBeVisible();
  });

  test('should complete checkout flow', async () => {
    // Assume user is logged in and has items in cart
    await element(by.text('Checkout')).tap();

    // Fill shipping info
    await element(by.id('name-input')).typeText('John Doe');
    await element(by.id('address-input')).typeText('123 Main St');

    // Continue to payment
    await element(by.text('Continue')).tap();

    // Select payment method
    await element(by.text('Credit Card')).tap();

    // Complete purchase
    await element(by.text('Complete Purchase')).tap();

    // Verify success
    await expect(element(by.text('Order placed successfully!'))).toBeVisible();
  });
});
```

### **Detox Configuration**
```javascript
// .detoxrc.js
module.exports = {
  testRunner: {
    args: {
      $0: 'jest',
      config: 'e2e/jest.config.js',
    },
    jest: {
      setupTimeout: 120000,
    },
  },
  apps: {
    'ios.debug': {
      type: 'ios.app',
      binaryPath: 'ios/build/Build/Products/Debug-iphonesimulator/MyApp.app',
      build: 'xcodebuild -workspace ios/MyApp.xcworkspace -scheme MyApp -configuration Debug -sdk iphonesimulator -derivedDataPath ios/build',
    },
    'ios.release': {
      type: 'ios.app',
      binaryPath: 'ios/build/Build/Products/Release-iphonesimulator/MyApp.app',
      build: 'xcodebuild -workspace ios/MyApp.xcworkspace -scheme MyApp -configuration Release -sdk iphonesimulator -derivedDataPath ios/build',
    },
    'android.debug': {
      type: 'android.apk',
      binaryPath: 'android/app/build/outputs/apk/debug/app-debug.apk',
      build: 'cd android && ./gradlew assembleDebug assembleAndroidTest -DtestBuildType=debug',
      reversePorts: [8081],
    },
    'android.release': {
      type: 'android.apk',
      binaryPath: 'android/app/build/outputs/apk/release/app-release.apk',
      build: 'cd android && ./gradlew assembleRelease assembleAndroidTest -DtestBuildType=release',
    },
  },
  devices: {
    simulator: {
      type: 'ios.simulator',
      device: {
        type: 'iPhone 14',
      },
    },
    emulator: {
      type: 'android.emulator',
      device: {
        avdName: 'Pixel_4_API_30',
      },
    },
  },
  configurations: {
    'ios.sim.debug': {
      device: 'simulator',
      app: 'ios.debug',
    },
    'ios.sim.release': {
      device: 'simulator',
      app: 'ios.release',
    },
    'android.emu.debug': {
      device: 'emulator',
      app: 'android.debug',
    },
    'android.emu.release': {
      device: 'emulator',
      app: 'android.release',
    },
  },
};
```

---

## 🎭 **Mocking & Test Utilities**

### **Custom Test Utilities**
```javascript
// __tests__/testUtils.js
import React from 'react';
import { render } from '@testing-library/react-native';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { NavigationContainer } from '@react-navigation/native';

// Custom render function with providers
const AllTheProviders = ({ children }) => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
      },
    },
  });

  return (
    <QueryClientProvider client={queryClient}>
      <NavigationContainer>
        {children}
      </NavigationContainer>
    </QueryClientProvider>
  );
};

const customRender = (ui, options) =>
  render(ui, { wrapper: AllTheProviders, ...options });

// Re-export everything
export * from '@testing-library/react-native';

// Override render method
export { customRender as render };

// Custom test utilities
export const createMockNavigation = () => ({
  navigate: jest.fn(),
  goBack: jest.fn(),
  reset: jest.fn(),
  setParams: jest.fn(),
});

export const createMockRoute = (params = {}) => ({
  params,
  name: 'MockScreen',
  key: 'mock-key',
});

export const waitForNextTick = () => new Promise(resolve => setTimeout(resolve, 0));

export const mockApiResponse = (data, options = {}) => ({
  data,
  status: options.status || 200,
  statusText: options.statusText || 'OK',
  headers: options.headers || {},
  config: options.config || {},
});

export const mockApiError = (message, status = 500) => {
  const error = new Error(message);
  error.response = {
    status,
    data: { message },
  };
  return error;
};

// User event utilities
export const userEvent = {
  type: async (element, text) => {
    const { fireEvent } = await import('@testing-library/react-native');
    fireEvent.changeText(element, text);
  },

  press: async (element) => {
    const { fireEvent } = await import('@testing-library/react-native');
    fireEvent.press(element);
  },

  scroll: async (element, options = {}) => {
    const { fireEvent } = await import('@testing-library/react-native');
    fireEvent.scroll(element, {
      nativeEvent: {
        contentOffset: { x: 0, y: 100 },
        ...options,
      },
    });
  },
};
```

### **Mock Data Generators**
```javascript
// __tests__/mocks/userMocks.js
export const createMockUser = (overrides = {}) => ({
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  avatar: 'https://example.com/avatar.jpg',
  createdAt: '2023-01-01T00:00:00Z',
  ...overrides,
});

export const createMockUsers = (count = 5) => {
  return Array.from({ length: count }, (_, index) =>
    createMockUser({
      id: index + 1,
      name: `User ${index + 1}`,
      email: `user${index + 1}@example.com`,
    })
  );
};

export const createMockPost = (overrides = {}) => ({
  id: 1,
  title: 'Sample Post',
  content: 'This is a sample post content',
  authorId: 1,
  createdAt: '2023-01-01T00:00:00Z',
  updatedAt: '2023-01-01T00:00:00Z',
  likes: 10,
  comments: 5,
  ...overrides,
});

export const createMockPosts = (count = 10) => {
  return Array.from({ length: count }, (_, index) =>
    createMockPost({
      id: index + 1,
      title: `Post ${index + 1}`,
      content: `Content for post ${index + 1}`,
      authorId: (index % 3) + 1,
    })
  );
};

// API response mocks
export const mockApiResponses = {
  success: (data) => ({
    data,
    status: 200,
    statusText: 'OK',
  }),

  error: (message, status = 500) => ({
    response: {
      data: { message },
      status,
      statusText: 'Error',
    },
  }),

  pagination: (items, page = 1, limit = 10, total = 100) => ({
    data: items,
    status: 200,
    statusText: 'OK',
    headers: {
      'x-total-count': total.toString(),
      'x-page': page.toString(),
      'x-limit': limit.toString(),
    },
  }),
};
```

---

## 🧪 **Test-Driven Development**

### **TDD Example**
```javascript
// __tests__/utils/calculator.test.js
import Calculator from '../../utils/calculator';

// Red: Write failing test first
describe('Calculator', () => {
  let calculator;

  beforeEach(() => {
    calculator = new Calculator();
  });

  describe('add', () => {
    test('should add two positive numbers', () => {
      // This test will fail initially
      expect(calculator.add(2, 3)).toBe(5);
    });

    test('should add positive and negative numbers', () => {
      expect(calculator.add(5, -3)).toBe(2);
    });
  });

  describe('subtract', () => {
    test('should subtract two numbers', () => {
      expect(calculator.subtract(5, 3)).toBe(2);
    });
  });

  describe('multiply', () => {
    test('should multiply two numbers', () => {
      expect(calculator.multiply(3, 4)).toBe(12);
    });
  });

  describe('divide', () => {
    test('should divide two numbers', () => {
      expect(calculator.divide(10, 2)).toBe(5);
    });

    test('should throw error when dividing by zero', () => {
      expect(() => calculator.divide(10, 0)).toThrow('Cannot divide by zero');
    });
  });

  describe('history', () => {
    test('should keep track of operations', () => {
      calculator.add(2, 3);
      calculator.subtract(5, 2);

      expect(calculator.getHistory()).toEqual([
        { operation: 'add', a: 2, b: 3, result: 5 },
        { operation: 'subtract', a: 5, b: 2, result: 3 },
      ]);
    });

    test('should limit history to last 10 operations', () => {
      for (let i = 0; i < 12; i++) {
        calculator.add(i, 1);
      }

      expect(calculator.getHistory()).toHaveLength(10);
    });
  });
});

// Green: Implement minimal code to pass tests
// utils/calculator.js
class Calculator {
  constructor() {
    this.history = [];
    this.maxHistorySize = 10;
  }

  add(a, b) {
    const result = a + b;
    this.addToHistory('add', a, b, result);
    return result;
  }

  subtract(a, b) {
    const result = a - b;
    this.addToHistory('subtract', a, b, result);
    return result;
  }

  multiply(a, b) {
    const result = a * b;
    this.addToHistory('multiply', a, b, result);
    return result;
  }

  divide(a, b) {
    if (b === 0) {
      throw new Error('Cannot divide by zero');
    }
    const result = a / b;
    this.addToHistory('divide', a, b, result);
    return result;
  }

  addToHistory(operation, a, b, result) {
    this.history.push({ operation, a, b, result });

    if (this.history.length > this.maxHistorySize) {
      this.history.shift();
    }
  }

  getHistory() {
    return [...this.history];
  }

  clearHistory() {
    this.history = [];
  }
}

export default Calculator;
```

### **TDD Workflow**
```javascript
// Example TDD workflow for a new feature
describe('ShoppingCart', () => {
  let cart;

  beforeEach(() => {
    cart = new ShoppingCart();
  });

  // 1. Red: Write failing test
  test('should add item to cart', () => {
    const item = { id: 1, name: 'Product A', price: 10 };
    cart.addItem(item);

    expect(cart.getItems()).toContain(item);
  });

  // 2. Green: Implement minimal code
  // class ShoppingCart { addItem(item) { this.items.push(item); } }

  // 3. Refactor: Improve implementation
  // Add validation, error handling, etc.

  test('should calculate total price', () => {
    cart.addItem({ id: 1, name: 'Product A', price: 10 });
    cart.addItem({ id: 2, name: 'Product B', price: 20 });

    expect(cart.getTotal()).toBe(30);
  });

  test('should apply discount', () => {
    cart.addItem({ id: 1, name: 'Product A', price: 100 });
    cart.applyDiscount(10); // 10% discount

    expect(cart.getTotal()).toBe(90);
  });

  test('should handle invalid discount', () => {
    cart.addItem({ id: 1, name: 'Product A', price: 100 });

    expect(() => cart.applyDiscount(150)).toThrow('Invalid discount percentage');
  });
});
```

---

## 🤖 **Automated Testing**

### **GitHub Actions CI/CD**
```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

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

      - name: Build app
        run: npm run build

      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info

  e2e-test:
    runs-on: ubuntu-latest
    needs: test

    steps:
      - uses: actions/checkout@v3

      - name: Use Node.js 18.x
        uses: actions/setup-node@v3
        with:
          node-version: 18.x
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Cache Detox dependencies
        uses: actions/cache@v3
        with:
          path: ~/.cache/detox
          key: detox-${{ runner.os }}-${{ hashFiles('package-lock.json') }}

      - name: Run E2E tests
        uses: wix/Detox/instruments@v1
        with:
          detox-configuration: android.emu.debug
          detox-artifacts-location: artifacts
          working-directory: .

      - name: Upload E2E artifacts
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: e2e-artifacts
          path: artifacts
```

### **Jest Configuration**
```javascript
// jest.config.js
module.exports = {
  preset: 'react-native',
  setupFilesAfterEnv: ['<rootDir>/__tests__/setup.js'],
  testMatch: [
    '<rootDir>/__tests__/**/*.test.js',
    '<rootDir>/src/**/*.test.js',
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
    '^@screens/(.*)$': '<rootDir>/src/screens/$1',
    '^@services/(.*)$': '<rootDir>/src/services/$1',
    '^@utils/(.*)$': '<rootDir>/src/utils/$1',
  },
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?|react-navigation|@react-navigation/.*))',
  ],
  testTimeout: 10000,
};
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
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:jest/recommended',
  ],
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint', 'react', 'react-hooks', 'jest'],
  rules: {
    // React rules
    'react/prop-types': 'off',
    'react/display-name': 'off',

    // React Hooks rules
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',

    // Jest rules
    'jest/no-disabled-tests': 'warn',
    'jest/no-focused-tests': 'error',
    'jest/no-identical-title': 'error',
    'jest/prefer-to-have-length': 'warn',
    'jest/valid-expect': 'error',

    // General rules
    'no-console': 'warn',
    'no-debugger': 'error',
    'no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    'prefer-const': 'error',
    'no-var': 'error',

    // React Native specific
    'react-native/no-unused-styles': 'error',
    'react-native/split-platform-components': 'error',
    'react-native/no-inline-styles': 'warn',
    'react-native/no-color-literals': 'warn',
  },
  settings: {
    react: {
      version: 'detect',
    },
  },
  env: {
    'jest/globals': true,
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

### **Husky & Lint-Staged**
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
      "jest --findRelatedTests --passWithNoTests"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

---

## 🎯 **Practical Examples**

### **Complete Testing Suite**
```javascript
// __tests__/integration/userJourney.test.js
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';

// Mock services
jest.mock('../../services/authService');
jest.mock('../../services/apiService');

const authService = require('../../services/authService');
const apiService = require('../../services/apiService');

// Test components
const LoginScreen = ({ navigation }) => {
  const [email, setEmail] = React.useState('');
  const [password, setPassword] = React.useState('');

  const handleLogin = async () => {
    try {
      await authService.login({ email, password });
      navigation.replace('Home');
    } catch (error) {
      alert('Login failed');
    }
  };

  return (
    <View>
      <TextInput
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        testID="email-input"
      />
      <TextInput
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
        testID="password-input"
      />
      <TouchableOpacity onPress={handleLogin} testID="login-button">
        <Text>Login</Text>
      </TouchableOpacity>
    </View>
  );
};

const HomeScreen = ({ navigation }) => {
  const [posts, setPosts] = React.useState([]);

  React.useEffect(() => {
    const loadPosts = async () => {
      const data = await apiService.getPosts();
      setPosts(data);
    };
    loadPosts();
  }, []);

  return (
    <View>
      <Text testID="welcome-text">Welcome!</Text>
      {posts.map(post => (
        <Text key={post.id} testID={`post-${post.id}`}>
          {post.title}
        </Text>
      ))}
      <TouchableOpacity
        onPress={() => navigation.navigate('Profile')}
        testID="profile-button"
      >
        <Text>Go to Profile</Text>
      </TouchableOpacity>
    </View>
  );
};

const Stack = createStackNavigator();

const App = () => (
  <NavigationContainer>
    <Stack.Navigator>
      <Stack.Screen name="Login" component={LoginScreen} />
      <Stack.Screen name="Home" component={HomeScreen} />
      <Stack.Screen name="Profile" component={() => <Text>Profile</Text>} />
    </Stack.Navigator>
  </NavigationContainer>
);

describe('User Journey Integration Test', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('complete user login and navigation flow', async () => {
    // Mock successful login
    authService.login.mockResolvedValue({
      user: { id: 1, email: 'test@example.com' },
      token: 'mock-token',
    });

    // Mock posts data
    apiService.getPosts.mockResolvedValue([
      { id: 1, title: 'Post 1' },
      { id: 2, title: 'Post 2' },
    ]);

    const { getByTestId, queryByTestId } = render(<App />);

    // Should start on login screen
    expect(getByTestId('email-input')).toBeTruthy();
    expect(getByTestId('password-input')).toBeTruthy();

    // Fill login form
    fireEvent.changeText(getByTestId('email-input'), 'test@example.com');
    fireEvent.changeText(getByTestId('password-input'), 'password123');

    // Submit login
    fireEvent.press(getByTestId('login-button'));

    // Should navigate to home screen
    await waitFor(() => {
      expect(getByTestId('welcome-text')).toBeTruthy();
    });

    // Should load and display posts
    await waitFor(() => {
      expect(getByTestId('post-1')).toBeTruthy();
      expect(getByTestId('post-2')).toBeTruthy();
    });

    // Navigate to profile
    fireEvent.press(getByTestId('profile-button'));

    // Should show profile screen
    await waitFor(() => {
      expect(queryByTestId('welcome-text')).toBeNull();
    });
  });

  test('handles login failure', async () => {
    // Mock failed login
    authService.login.mockRejectedValue(new Error('Invalid credentials'));

    const { getByTestId } = render(<App />);

    // Fill and submit login form
    fireEvent.changeText(getByTestId('email-input'), 'wrong@example.com');
    fireEvent.changeText(getByTestId('password-input'), 'wrongpassword');
    fireEvent.press(getByTestId('login-button'));

    // Should stay on login screen
    await waitFor(() => {
      expect(getByTestId('email-input')).toBeTruthy();
    });
  });
});
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Unit Testing**: Testing individual functions and components
- ✅ **Component Testing**: Testing React Native components with RTL
- ✅ **Integration Testing**: Testing component interactions and API calls
- ✅ **End-to-End Testing**: Testing complete user workflows with Detox
- ✅ **Mocking**: Creating mocks for external dependencies
- ✅ **Test-Driven Development**: Writing tests before implementation
- ✅ **Automated Testing**: CI/CD pipelines and test automation
- ✅ **Code Quality**: ESLint, Prettier, and code quality tools

### **Best Practices**
1. **Write tests for all new code** and maintain existing tests
2. **Use descriptive test names** that explain what is being tested
3. **Test behavior, not implementation** details
4. **Keep tests fast and reliable** to encourage frequent running
5. **Use appropriate mocking** to isolate units under test
6. **Maintain high test coverage** (aim for 80%+)
7. **Run tests in CI/CD** to catch issues early
8. **Test edge cases and error conditions**
9. **Use snapshot testing** for UI components
10. **Keep test files organized** and well-structured

### **Next Steps**
- Learn advanced mocking techniques
- Implement visual regression testing
- Set up performance testing
- Create custom test utilities
- Implement automated deployment with testing

---

## 🎯 **Assignment**

### **Task 1: Unit Testing Suite**
Create comprehensive unit tests for:
- Utility functions (math, string manipulation, date formatting)
- Custom hooks (useCounter, useApi, useAuth)
- Business logic classes (User, Product, Cart)
- Error handling and edge cases

### **Task 2: Component Testing**
Build component tests for:
- Form components (login, registration, checkout)
- List components with pagination
- Modal and overlay components
- Navigation components
- Loading and error states

### **Task 3: Integration Testing**
Implement integration tests for:
- User authentication flow
- Shopping cart functionality
- API data fetching and caching
- Navigation between screens
- Form submission and validation

### **Task 4: E2E Testing**
Create end-to-end tests for:
- Complete user registration and login
- Product browsing and purchasing
- Profile management
- Settings configuration
- Error recovery scenarios

---

## 📚 **Additional Resources**
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Detox Documentation](https://wix.github.io/Detox/)
- [Testing Library](https://testing-library.com/docs/react-native-testing-library/intro/)
- [React Native Testing Best Practices](https://reactnative.dev/docs/testing-overview)

---

**Next Lesson**: [Lesson 41: Advanced Backend Integration Patterns](Lesson%2041_%20Advanced%20Backend%20Integration%20Patterns.md)