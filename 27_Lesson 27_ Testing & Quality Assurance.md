# Lesson 27: Testing & Quality Assurance

## 🎯 **Learning Objectives**
- Master testing methodologies for React Native and Node.js applications
- Implement unit tests, integration tests, and end-to-end tests
- Set up automated testing pipelines and CI/CD
- Write maintainable and reliable test suites
- Implement test-driven development (TDD) practices

## 📚 **Table of Contents**
1. [Introduction to Testing](#introduction-to-testing)
2. [Unit Testing with Jest](#unit-testing-with-jest)
3. [React Native Testing](#react-native-testing)
4. [Integration Testing](#integration-testing)
5. [End-to-End Testing](#end-to-end-testing)
6. [API Testing](#api-testing)
7. [Test-Driven Development](#test-driven-development)
8. [Mocking & Stubbing](#mocking--stubbing)
9. [Test Automation & CI/CD](#test-automation--cicd)
10. [Performance Testing](#performance-testing)
11. [Code Coverage](#code-coverage)
12. [Testing Best Practices](#testing-best-practices)
13. [Practical Examples](#practical-examples)

---

## 🧪 **Introduction to Testing**

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
- **Integration Tests**: Test interaction between components/modules
- **End-to-End Tests**: Test complete user workflows
- **Performance Tests**: Test application speed and resource usage
- **Security Tests**: Test for vulnerabilities and security issues

### **Testing Benefits**
- **Quality Assurance**: Catch bugs before production
- **Refactoring Safety**: Ensure changes don't break existing functionality
- **Documentation**: Tests serve as living documentation
- **Confidence**: Deploy with assurance that code works correctly

---

## 🃏 **Unit Testing with Jest**

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
  testTimeout: 10000,
};
```

### **Jest Setup File**
```javascript
// jest.setup.js
import 'react-native-gesture-handler/jestSetup';

// Mock react-native modules
jest.mock('react-native/Libraries/Utilities/Platform', () => ({
  OS: 'ios',
  select: jest.fn(),
}));

jest.mock('react-native/Libraries/Utilities/Dimensions', () => ({
  get: jest.fn(() => ({ width: 375, height: 812 })),
  addEventListener: jest.fn(),
  removeEventListener: jest.fn(),
}));

jest.mock('@react-native-async-storage/async-storage', () => ({
  getItem: jest.fn(),
  setItem: jest.fn(),
  removeItem: jest.fn(),
  getAllKeys: jest.fn(),
  multiGet: jest.fn(),
  multiSet: jest.fn(),
  multiRemove: jest.fn(),
}));

// Mock network requests
global.fetch = jest.fn();

// Mock timers
jest.useFakeTimers();

// Global test utilities
global.testUtils = {
  flushPromises: () => new Promise(resolve => setImmediate(resolve)),
  createMockStore: (initialState) => ({
    getState: () => initialState,
    dispatch: jest.fn(),
    subscribe: jest.fn(),
  }),
};
```

### **Basic Unit Tests**
```javascript
// __tests__/utils/math.test.js
const { add, subtract, multiply, divide, fibonacci } = require('../utils/math');

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

    test('handles large numbers', () => {
      expect(fibonacci(20)).toBe(6765);
    });
  });
});
```

### **Async Testing**
```javascript
// __tests__/api/user.test.js
const { getUser, createUser, updateUser, deleteUser } = require('../api/user');

// Mock the database
jest.mock('../models/User', () => ({
  findById: jest.fn(),
  findOne: jest.fn(),
  create: jest.fn(),
  findByIdAndUpdate: jest.fn(),
  findByIdAndDelete: jest.fn(),
}));

const User = require('../models/User');

describe('User API', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('getUser', () => {
    test('returns user when found', async () => {
      const mockUser = {
        id: '1',
        name: 'John Doe',
        email: 'john@example.com',
      };

      User.findById.mockResolvedValue(mockUser);

      const result = await getUser('1');

      expect(User.findById).toHaveBeenCalledWith('1');
      expect(result).toEqual(mockUser);
    });

    test('returns null when user not found', async () => {
      User.findById.mockResolvedValue(null);

      const result = await getUser('999');

      expect(result).toBeNull();
    });

    test('throws error on database failure', async () => {
      User.findById.mockRejectedValue(new Error('Database error'));

      await expect(getUser('1')).rejects.toThrow('Database error');
    });
  });

  describe('createUser', () => {
    test('creates user successfully', async () => {
      const userData = {
        name: 'Jane Doe',
        email: 'jane@example.com',
        password: 'password123',
      };

      const createdUser = {
        id: '2',
        ...userData,
        createdAt: new Date(),
      };

      User.findOne.mockResolvedValue(null);
      User.create.mockResolvedValue(createdUser);

      const result = await createUser(userData);

      expect(User.findOne).toHaveBeenCalledWith({ email: userData.email });
      expect(User.create).toHaveBeenCalledWith(userData);
      expect(result).toEqual(createdUser);
    });

    test('throws error when email already exists', async () => {
      const userData = {
        name: 'Jane Doe',
        email: 'existing@example.com',
        password: 'password123',
      };

      User.findOne.mockResolvedValue({ id: '1', email: userData.email });

      await expect(createUser(userData)).rejects.toThrow('Email already exists');
    });
  });

  describe('updateUser', () => {
    test('updates user successfully', async () => {
      const userId = '1';
      const updateData = { name: 'Updated Name' };
      const updatedUser = {
        id: userId,
        name: 'Updated Name',
        email: 'john@example.com',
      };

      User.findByIdAndUpdate.mockResolvedValue(updatedUser);

      const result = await updateUser(userId, updateData);

      expect(User.findByIdAndUpdate).toHaveBeenCalledWith(
        userId,
        updateData,
        { new: true, runValidators: true }
      );
      expect(result).toEqual(updatedUser);
    });

    test('throws error when user not found', async () => {
      User.findByIdAndUpdate.mockResolvedValue(null);

      await expect(updateUser('999', { name: 'Test' })).rejects.toThrow('User not found');
    });
  });

  describe('deleteUser', () => {
    test('deletes user successfully', async () => {
      const userId = '1';
      const deletedUser = { id: userId, name: 'John Doe' };

      User.findByIdAndDelete.mockResolvedValue(deletedUser);

      const result = await deleteUser(userId);

      expect(User.findByIdAndDelete).toHaveBeenCalledWith(userId);
      expect(result).toEqual(deletedUser);
    });

    test('throws error when user not found', async () => {
      User.findByIdAndDelete.mockResolvedValue(null);

      await expect(deleteUser('999')).rejects.toThrow('User not found');
    });
  });
});
```

---

## 📱 **React Native Testing**

### **React Native Testing Library**
```javascript
// __tests__/components/Button.test.js
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import { Button } from '../components/Button';

describe('Button Component', () => {
  const mockOnPress = jest.fn();

  beforeEach(() => {
    mockOnPress.mockClear();
  });

  test('renders correctly with title', () => {
    const { getByText } = render(
      <Button title="Click me" onPress={mockOnPress} />
    );

    expect(getByText('Click me')).toBeTruthy();
  });

  test('calls onPress when pressed', () => {
    const { getByText } = render(
      <Button title="Click me" onPress={mockOnPress} />
    );

    fireEvent.press(getByText('Click me'));
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

    fireEvent.press(getByText('Loading...'));
    expect(mockOnPress).not.toHaveBeenCalled();
  });

  test('applies correct styles', () => {
    const { getByText } = render(
      <Button title="Primary" onPress={mockOnPress} variant="primary" />
    );

    const button = getByText('Primary');
    expect(button.props.style).toContainEqual(
      expect.objectContaining({ backgroundColor: '#007AFF' })
    );
  });

  test('handles accessibility props', () => {
    const { getByA11yLabel } = render(
      <Button
        title="Click me"
        onPress={mockOnPress}
        accessibilityLabel="Custom button"
      />
    );

    expect(getByA11yLabel('Custom button')).toBeTruthy();
  });
});
```

### **Custom Hooks Testing**
```javascript
// __tests__/hooks/useAuth.test.js
import { renderHook, act, waitFor } from '@testing-library/react-native';
import { useAuth } from '../hooks/useAuth';

// Mock AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () => ({
  getItem: jest.fn(),
  setItem: jest.fn(),
  removeItem: jest.fn(),
}));

// Mock the API
jest.mock('../api/auth', () => ({
  login: jest.fn(),
  logout: jest.fn(),
  refreshToken: jest.fn(),
}));

const AsyncStorage = require('@react-native-async-storage/async-storage');
const { login, logout, refreshToken } = require('../api/auth');

describe('useAuth Hook', () => {
  beforeEach(() => {
    jest.clearAllMocks();
    AsyncStorage.getItem.mockResolvedValue(null);
  });

  test('initializes with no user', async () => {
    const { result } = renderHook(() => useAuth());

    await waitFor(() => {
      expect(result.current.user).toBeNull();
      expect(result.current.isLoading).toBe(false);
    });
  });

  test('loads user from storage on mount', async () => {
    const storedUser = { id: '1', email: 'test@example.com' };
    AsyncStorage.getItem.mockResolvedValue(JSON.stringify(storedUser));

    const { result } = renderHook(() => useAuth());

    await waitFor(() => {
      expect(result.current.user).toEqual(storedUser);
      expect(result.current.isLoading).toBe(false);
    });
  });

  test('handles login successfully', async () => {
    const loginData = { email: 'test@example.com', password: 'password' };
    const userResponse = { id: '1', email: 'test@example.com', token: 'token123' };

    login.mockResolvedValue(userResponse);

    const { result } = renderHook(() => useAuth());

    act(() => {
      result.current.login(loginData);
    });

    await waitFor(() => {
      expect(result.current.user).toEqual(userResponse);
      expect(result.current.isLoading).toBe(false);
    });

    expect(login).toHaveBeenCalledWith(loginData);
    expect(AsyncStorage.setItem).toHaveBeenCalledWith(
      'user',
      JSON.stringify(userResponse)
    );
  });

  test('handles login error', async () => {
    const loginData = { email: 'test@example.com', password: 'wrong' };
    const error = new Error('Invalid credentials');

    login.mockRejectedValue(error);

    const { result } = renderHook(() => useAuth());

    act(() => {
      result.current.login(loginData);
    });

    await waitFor(() => {
      expect(result.current.user).toBeNull();
      expect(result.current.error).toEqual(error);
      expect(result.current.isLoading).toBe(false);
    });
  });

  test('handles logout', async () => {
    const storedUser = { id: '1', email: 'test@example.com' };
    AsyncStorage.getItem.mockResolvedValue(JSON.stringify(storedUser));

    const { result } = renderHook(() => useAuth());

    // Wait for user to be loaded
    await waitFor(() => {
      expect(result.current.user).toEqual(storedUser);
    });

    // Logout
    act(() => {
      result.current.logout();
    });

    expect(result.current.user).toBeNull();
    expect(AsyncStorage.removeItem).toHaveBeenCalledWith('user');
  });

  test('refreshes token automatically', async () => {
    const storedUser = { id: '1', email: 'test@example.com', token: 'old-token' };
    const newUserData = { id: '1', email: 'test@example.com', token: 'new-token' };

    AsyncStorage.getItem.mockResolvedValue(JSON.stringify(storedUser));
    refreshToken.mockResolvedValue(newUserData);

    const { result } = renderHook(() => useAuth());

    // Wait for user to be loaded
    await waitFor(() => {
      expect(result.current.user).toEqual(storedUser);
    });

    // Trigger token refresh
    act(() => {
      result.current.refreshToken();
    });

    await waitFor(() => {
      expect(result.current.user).toEqual(newUserData);
    });

    expect(refreshToken).toHaveBeenCalled();
    expect(AsyncStorage.setItem).toHaveBeenCalledWith(
      'user',
      JSON.stringify(newUserData)
    );
  });
});
```

---

## 🔗 **Integration Testing**

### **API Integration Tests**
```javascript
// __tests__/integration/auth.integration.test.js
const request = require('supertest');
const mongoose = require('mongoose');
const { MongoMemoryServer } = require('mongodb-memory-server');
const app = require('../app');
const User = require('../models/User');

let mongoServer;

beforeAll(async () => {
  // Start in-memory MongoDB
  mongoServer = await MongoMemoryServer.create();
  const mongoUri = mongoServer.getUri();
  await mongoose.connect(mongoUri);
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});

beforeEach(async () => {
  // Clear database before each test
  await User.deleteMany({});
});

describe('Authentication Integration Tests', () => {
  describe('POST /api/auth/register', () => {
    test('should register a new user', async () => {
      const userData = {
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        password: 'password123',
      };

      const response = await request(app)
        .post('/api/auth/register')
        .send(userData)
        .expect(201);

      expect(response.body.success).toBe(true);
      expect(response.body.data).toHaveProperty('id');
      expect(response.body.data.email).toBe(userData.email);
      expect(response.body.data).not.toHaveProperty('password');

      // Verify user was created in database
      const user = await User.findOne({ email: userData.email });
      expect(user).toBeTruthy();
      expect(user.firstName).toBe(userData.firstName);
    });

    test('should not register user with existing email', async () => {
      // Create existing user
      await User.create({
        firstName: 'Existing',
        lastName: 'User',
        email: 'existing@example.com',
        password: 'password123',
      });

      const response = await request(app)
        .post('/api/auth/register')
        .send({
          firstName: 'New',
          lastName: 'User',
          email: 'existing@example.com',
          password: 'password123',
        })
        .expect(409);

      expect(response.body.success).toBe(false);
      expect(response.body.error).toBe('Email already exists');
    });

    test('should validate required fields', async () => {
      const response = await request(app)
        .post('/api/auth/register')
        .send({
          firstName: 'John',
          // Missing required fields
        })
        .expect(400);

      expect(response.body.success).toBe(false);
      expect(response.body.error).toBe('Validation failed');
    });
  });

  describe('POST /api/auth/login', () => {
    beforeEach(async () => {
      // Create test user
      await User.create({
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        password: await require('../utils/password').hashPassword('password123'),
      });
    });

    test('should login with correct credentials', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'john@example.com',
          password: 'password123',
        })
        .expect(200);

      expect(response.body.success).toBe(true);
      expect(response.body.data).toHaveProperty('id');
      expect(response.body.data.email).toBe('john@example.com');
      expect(response.body.data).toHaveProperty('token');
      expect(response.body.data).toHaveProperty('refreshToken');
    });

    test('should not login with incorrect password', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'john@example.com',
          password: 'wrongpassword',
        })
        .expect(401);

      expect(response.body.success).toBe(false);
      expect(response.body.error).toBe('Invalid credentials');
    });

    test('should not login with non-existent email', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'nonexistent@example.com',
          password: 'password123',
        })
        .expect(401);

      expect(response.body.success).toBe(false);
      expect(response.body.error).toBe('Invalid credentials');
    });
  });

  describe('GET /api/auth/profile', () => {
    let token;
    let user;

    beforeEach(async () => {
      // Create test user and get token
      user = await User.create({
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        password: await require('../utils/password').hashPassword('password123'),
      });

      const loginResponse = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'john@example.com',
          password: 'password123',
        });

      token = loginResponse.body.data.token;
    });

    test('should get user profile with valid token', async () => {
      const response = await request(app)
        .get('/api/auth/profile')
        .set('Authorization', `Bearer ${token}`)
        .expect(200);

      expect(response.body.success).toBe(true);
      expect(response.body.data.id).toBe(user.id);
      expect(response.body.data.email).toBe(user.email);
    });

    test('should not get profile without token', async () => {
      const response = await request(app)
        .get('/api/auth/profile')
        .expect(401);

      expect(response.body.success).toBe(false);
      expect(response.body.error).toBe('Access token required');
    });

    test('should not get profile with invalid token', async () => {
      const response = await request(app)
        .get('/api/auth/profile')
        .set('Authorization', 'Bearer invalid-token')
        .expect(403);

      expect(response.body.success).toBe(false);
      expect(response.body.error).toBe('Invalid token');
    });
  });
});
```

### **Database Integration Tests**
```javascript
// __tests__/integration/database.integration.test.js
const mongoose = require('mongoose');
const { MongoMemoryServer } = require('mongodb-memory-server');
const User = require('../models/User');
const Post = require('../models/Post');

let mongoServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  const mongoUri = mongoServer.getUri();
  await mongoose.connect(mongoUri);
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});

beforeEach(async () => {
  // Clear all collections
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});

describe('Database Integration Tests', () => {
  describe('User-Post Relationship', () => {
    test('should create user with posts', async () => {
      // Create user
      const user = await User.create({
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        password: 'password123',
      });

      // Create posts for user
      const posts = await Post.create([
        {
          title: 'First Post',
          content: 'This is my first post',
          author: user._id,
        },
        {
          title: 'Second Post',
          content: 'This is my second post',
          author: user._id,
        },
      ]);

      // Populate user with posts
      const populatedUser = await User.findById(user._id).populate('posts');

      expect(populatedUser.posts).toHaveLength(2);
      expect(populatedUser.posts[0].title).toBe('First Post');
      expect(populatedUser.posts[1].title).toBe('Second Post');
    });

    test('should handle cascading deletes', async () => {
      // Create user with posts
      const user = await User.create({
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        password: 'password123',
      });

      await Post.create([
        { title: 'Post 1', content: 'Content 1', author: user._id },
        { title: 'Post 2', content: 'Content 2', author: user._id },
      ]);

      // Delete user
      await User.findByIdAndDelete(user._id);

      // Check that posts are also deleted (if cascade is implemented)
      const remainingPosts = await Post.find({ author: user._id });
      expect(remainingPosts).toHaveLength(0);
    });
  });

  describe('Data Validation', () => {
    test('should enforce unique constraints', async () => {
      // Create first user
      await User.create({
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        password: 'password123',
      });

      // Try to create user with same email
      await expect(User.create({
        firstName: 'Jane',
        lastName: 'Doe',
        email: 'john@example.com', // Same email
        password: 'password456',
      })).rejects.toThrow();
    });

    test('should validate required fields', async () => {
      await expect(User.create({
        firstName: 'John',
        // Missing required fields: lastName, email, password
      })).rejects.toThrow();
    });

    test('should validate field types', async () => {
      await expect(User.create({
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        password: 'password123',
        age: 'not-a-number', // Should be number
      })).rejects.toThrow();
    });
  });

  describe('Indexing Performance', () => {
    beforeAll(async () => {
      // Create test data
      const users = [];
      for (let i = 0; i < 1000; i++) {
        users.push({
          firstName: `User${i}`,
          lastName: `Last${i}`,
          email: `user${i}@example.com`,
          password: 'password123',
          age: Math.floor(Math.random() * 80) + 18,
        });
      }
      await User.insertMany(users);
    });

    test('should use indexes for queries', async () => {
      // Query with indexed field
      const startTime = Date.now();
      const users = await User.find({ email: 'user500@example.com' });
      const queryTime = Date.now() - startTime;

      expect(users).toHaveLength(1);
      expect(queryTime).toBeLessThan(100); // Should be fast with index
    });

    test('should handle complex queries efficiently', async () => {
      const startTime = Date.now();
      const users = await User.find({
        age: { $gte: 25, $lte: 65 },
        isActive: true,
      }).sort({ createdAt: -1 }).limit(50);
      const queryTime = Date.now() - startTime;

      expect(users.length).toBeLessThanOrEqual(50);
      expect(queryTime).toBeLessThan(500); // Should be reasonably fast
    });
  });

  describe('Transactions', () => {
    test('should handle transactions correctly', async () => {
      const session = await mongoose.startSession();

      try {
        await session.withTransaction(async () => {
          // Create user
          const user = await User.create([{
            firstName: 'Transaction',
            lastName: 'Test',
            email: 'transaction@example.com',
            password: 'password123',
          }], { session });

          // Create post for user
          await Post.create([{
            title: 'Transaction Post',
            content: 'This post was created in a transaction',
            author: user[0]._id,
          }], { session });

          // If we throw an error here, both operations should be rolled back
          // throw new Error('Test transaction rollback');
        });

        // Verify both documents were created
        const user = await User.findOne({ email: 'transaction@example.com' });
        const post = await Post.findOne({ title: 'Transaction Post' });

        expect(user).toBeTruthy();
        expect(post).toBeTruthy();
        expect(post.author.toString()).toBe(user._id.toString());

      } finally {
        await session.endSession();
      }
    });

    test('should rollback on transaction failure', async () => {
      const session = await mongoose.startSession();

      try {
        await expect(session.withTransaction(async () => {
          // Create user
          const user = await User.create([{
            firstName: 'Failed',
            lastName: 'Transaction',
            email: 'failed@example.com',
            password: 'password123',
          }], { session });

          // Create post
          await Post.create([{
            title: 'Failed Post',
            content: 'This should not exist',
            author: user[0]._id,
          }], { session });

          // Force failure
          throw new Error('Transaction failed');
        })).rejects.toThrow('Transaction failed');

        // Verify neither document exists
        const user = await User.findOne({ email: 'failed@example.com' });
        const post = await Post.findOne({ title: 'Failed Post' });

        expect(user).toBeNull();
        expect(post).toBeNull();

      } finally {
        await session.endSession();
      }
    });
  });
});
```

---

## 🤖 **End-to-End Testing**

### **Detox Setup for React Native**
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
        "type": "iPhone 12"
      }
    },
    "emulator": {
      "type": "android.emulator",
      "device": {
        "avdName": "Pixel_3_API_29"
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

### **E2E Test Examples**
```javascript
// e2e/tests/auth.test.js
const { device, expect, element, by } = require('detox');

describe('Authentication Flow', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  test('should show login screen initially', async () => {
    await expect(element(by.id('login-screen'))).toBeVisible();
    await expect(element(by.id('email-input'))).toBeVisible();
    await expect(element(by.id('password-input'))).toBeVisible();
    await expect(element(by.id('login-button'))).toBeVisible();
  });

  test('should login with valid credentials', async () => {
    // Enter credentials
    await element(by.id('email-input')).typeText('test@example.com');
    await element(by.id('password-input')).typeText('password123');

    // Tap login button
    await element(by.id('login-button')).tap();

    // Wait for navigation to home screen
    await waitFor(element(by.id('home-screen')))
      .toBeVisible()
      .withTimeout(5000);

    // Verify we're on home screen
    await expect(element(by.id('welcome-message'))).toHaveText('Welcome back!');
  });

  test('should show error for invalid credentials', async () => {
    // Enter invalid credentials
    await element(by.id('email-input')).typeText('invalid@example.com');
    await element(by.id('password-input')).typeText('wrongpassword');

    // Tap login button
    await element(by.id('login-button')).tap();

    // Wait for error message
    await waitFor(element(by.id('error-message')))
      .toBeVisible()
      .withTimeout(3000);

    // Verify error message
    await expect(element(by.id('error-message'))).toHaveText('Invalid credentials');
  });

  test('should navigate to register screen', async () => {
    await element(by.id('register-link')).tap();

    await expect(element(by.id('register-screen'))).toBeVisible();
    await expect(element(by.id('firstName-input'))).toBeVisible();
    await expect(element(by.id('lastName-input'))).toBeVisible();
    await expect(element(by.id('email-input'))).toBeVisible();
    await expect(element(by.id('password-input'))).toBeVisible();
  });

  test('should register new user', async () => {
    // Navigate to register
    await element(by.id('register-link')).tap();

    // Fill registration form
    await element(by.id('firstName-input')).typeText('John');
    await element(by.id('lastName-input')).typeText('Doe');
    await element(by.id('email-input')).typeText('john.doe@example.com');
    await element(by.id('password-input')).typeText('password123');

    // Submit registration
    await element(by.id('register-button')).tap();

    // Wait for success message
    await waitFor(element(by.id('success-message')))
      .toBeVisible()
      .withTimeout(5000);

    await expect(element(by.id('success-message'))).toHaveText('Registration successful!');
  });
});
```

---

## 🔌 **API Testing**

### **Supertest for API Testing**
```javascript
// __tests__/api/routes.test.js
const request = require('supertest');
const mongoose = require('mongoose');
const { MongoMemoryServer } = require('mongodb-memory-server');
const app = require('../../app');
const User = require('../../models/User');

let mongoServer;
let server;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  const mongoUri = mongoServer.getUri();
  await mongoose.connect(mongoUri);

  server = app.listen(0); // Random available port
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
  server.close();
});

beforeEach(async () => {
  await User.deleteMany({});
});

describe('API Routes', () => {
  describe('GET /api/health', () => {
    test('should return health status', async () => {
      const response = await request(app)
        .get('/api/health')
        .expect(200);

      expect(response.body).toHaveProperty('status', 'OK');
      expect(response.body).toHaveProperty('uptime');
      expect(response.body).toHaveProperty('timestamp');
    });
  });

  describe('User CRUD Operations', () => {
    let authToken;
    let testUser;

    beforeEach(async () => {
      // Create test user and get token
      testUser = await User.create({
        firstName: 'Test',
        lastName: 'User',
        email: 'test@example.com',
        password: 'password123',
      });

      // Login to get token
      const loginResponse = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'password123',
        });

      authToken = loginResponse.body.data.token;
    });

    test('GET /api/users should return users list', async () => {
      const response = await request(app)
        .get('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.success).toBe(true);
      expect(Array.isArray(response.body.data.users)).toBe(true);
      expect(response.body.data.users.length).toBeGreaterThan(0);
    });

    test('GET /api/users/:id should return specific user', async () => {
      const response = await request(app)
        .get(`/api/users/${testUser.id}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.success).toBe(true);
      expect(response.body.data.id).toBe(testUser.id);
      expect(response.body.data.email).toBe(testUser.email);
    });

    test('POST /api/users should create new user', async () => {
      const newUser = {
        firstName: 'New',
        lastName: 'User',
        email: 'newuser@example.com',
        password: 'password123',
      };

      const response = await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send(newUser)
        .expect(201);

      expect(response.body.success).toBe(true);
      expect(response.body.data).toHaveProperty('id');
      expect(response.body.data.email).toBe(newUser.email);
    });

    test('PUT /api/users/:id should update user', async () => {
      const updates = {
        firstName: 'Updated',
        lastName: 'Name',
      };

      const response = await request(app)
        .put(`/api/users/${testUser.id}`)
        .set('Authorization', `Bearer ${authToken}`)
        .send(updates)
        .expect(200);

      expect(response.body.success).toBe(true);
      expect(response.body.data.firstName).toBe(updates.firstName);
      expect(response.body.data.lastName).toBe(updates.lastName);
    });

    test('DELETE /api/users/:id should delete user', async () => {
      const response = await request(app)
        .delete(`/api/users/${testUser.id}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.success).toBe(true);
      expect(response.body.message).toBe('User deleted successfully');

      // Verify user is deleted
      const deletedUser = await User.findById(testUser.id);
      expect(deletedUser).toBeNull();
    });
  });

  describe('Error Handling', () => {
    test('should handle 404 for non-existent routes', async () => {
      const response = await request(app)
        .get('/api/non-existent-route')
        .expect(404);

      expect(response.body.success).toBe(false);
      expect(response.body.error).toBe('Not Found');
    });

    test('should handle unauthorized access', async () => {
      const response = await request(app)
        .get('/api/users')
        .expect(401);

      expect(response.body.success).toBe(false);
      expect(response.body.error).toBe('Access token required');
    });

    test('should handle invalid tokens', async () => {
      const response = await request(app)
        .get('/api/users')
        .set('Authorization', 'Bearer invalid-token')
        .expect(403);

      expect(response.body.success).toBe(false);
      expect(response.body.error).toBe('Invalid token');
    });
  });

  describe('Rate Limiting', () => {
    test('should handle rate limiting', async () => {
      // Make multiple requests quickly
      const promises = [];
      for (let i = 0; i < 15; i++) {
        promises.push(
          request(app)
            .get('/api/health')
            .expect(200)
        );
      }

      const responses = await Promise.all(promises);

      // At least one should be rate limited (429)
      const rateLimitedResponse = responses.find(r => r.status === 429);
      expect(rateLimitedResponse).toBeDefined();
      expect(rateLimitedResponse.body.error).toContain('Too many requests');
    });
  });
});
```

---

## 🧪 **Test-Driven Development**

### **TDD Example: User Registration**
```javascript
// Step 1: Write failing test first
// __tests__/features/userRegistration.test.js
const request = require('supertest');
const app = require('../../app');
const User = require('../../models/User');

describe('User Registration Feature', () => {
  beforeEach(async () => {
    await User.deleteMany({});
  });

  describe('POST /api/auth/register', () => {
    test('should register a new user with valid data', async () => {
      const userData = {
        firstName: 'John',
        lastName: 'Doe',
        email: 'john.doe@example.com',
        password: 'SecurePass123!',
      };

      const response = await request(app)
        .post('/api/auth/register')
        .send(userData)
        .expect(201);

      expect(response.body.success).toBe(true);
      expect(response.body.data).toHaveProperty('id');
      expect(response.body.data.email).toBe(userData.email);
      expect(response.body.data.firstName).toBe(userData.firstName);
      expect(response.body.data.lastName).toBe(userData.lastName);
      expect(response.body.data).not.toHaveProperty('password');

      // Verify user exists in database
      const userInDb = await User.findOne({ email: userData.email });
      expect(userInDb).toBeTruthy();
      expect(userInDb.firstName).toBe(userData.firstName);
    });

    test('should hash the password before storing', async () => {
      const userData = {
        firstName: 'Jane',
        lastName: 'Smith',
        email: 'jane.smith@example.com',
        password: 'MyPassword123!',
      };

      await request(app)
        .post('/api/auth/register')
        .send(userData)
        .expect(201);

      const userInDb = await User.findOne({ email: userData.email });
      expect(userInDb.password).not.toBe(userData.password);
      expect(userInDb.password).toHaveLength(60); // bcrypt hash length
    });

    test('should not register user with existing email', async () => {
      // Create existing user
      await User.create({
        firstName: 'Existing',
        lastName: 'User',
        email: 'existing@example.com',
        password: 'password123',
      });

      const response = await request(app)
        .post('/api/auth/register')
        .send({
          firstName: 'New',
          lastName: 'User',
          email: 'existing@example.com',
          password: 'password456',
        })
        .expect(409);

      expect(response.body.success).toBe(false);
      expect(response.body.error).toBe('Email already exists');
    });

    test('should validate required fields', async () => {
      const response = await request(app)
        .post('/api/auth/register')
        .send({
          firstName: 'John',
          // Missing lastName, email, password
        })
        .expect(400);

      expect(response.body.success).toBe(false);
      expect(response.body.error).toBe('Validation failed');
      expect(response.body.details).toContainEqual(
        expect.objectContaining({ field: 'lastName' })
      );
      expect(response.body.details).toContainEqual(
        expect.objectContaining({ field: 'email' })
      );
      expect(response.body.details).toContainEqual(
        expect.objectContaining({ field: 'password' })
      );
    });

    test('should validate email format', async () => {
      const response = await request(app)
        .post('/api/auth/register')
        .send({
          firstName: 'John',
          lastName: 'Doe',
          email: 'invalid-email',
          password: 'password123',
        })
        .expect(400);

      expect(response.body.success).toBe(false);
      expect(response.body.details).toContainEqual(
        expect.objectContaining({
          field: 'email',
          message: expect.stringContaining('valid email'),
        })
      );
    });

    test('should validate password strength', async () => {
      const response = await request(app)
        .post('/api/auth/register')
        .send({
          firstName: 'John',
          lastName: 'Doe',
          email: 'john@example.com',
          password: 'weak', // Too short and weak
        })
        .expect(400);

      expect(response.body.success).toBe(false);
      expect(response.body.details).toContainEqual(
        expect.objectContaining({
          field: 'password',
          message: expect.stringContaining('at least 8 characters'),
        })
      );
    });

    test('should send welcome email after registration', async () => {
      const userData = {
        firstName: 'Welcome',
        lastName: 'User',
        email: 'welcome@example.com',
        password: 'Welcome123!',
      };

      const response = await request(app)
        .post('/api/auth/register')
        .send(userData)
        .expect(201);

      expect(response.body.success).toBe(true);
      // Note: Email sending would be tested with mocked email service
    });
  });
});
---

## 🎭 **Mocking & Stubbing**

### **Mock Functions with Jest**
```javascript
// __tests__/utils/api.test.js
const axios = require('axios');
const { fetchUserData, fetchPosts } = require('../utils/api');

// Mock axios
jest.mock('axios');
const mockedAxios = axios;

describe('API Utils', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('fetchUserData', () => {
    test('should fetch user data successfully', async () => {
      const mockUser = {
        id: 1,
        name: 'John Doe',
        email: 'john@example.com',
      };

      mockedAxios.get.mockResolvedValueOnce({
        data: mockUser,
        status: 200,
      });

      const result = await fetchUserData(1);

      expect(mockedAxios.get).toHaveBeenCalledWith('/api/users/1');
      expect(result).toEqual(mockUser);
    });

    test('should handle API errors', async () => {
      const errorMessage = 'User not found';
      mockedAxios.get.mockRejectedValueOnce({
        response: {
          status: 404,
          data: { error: errorMessage },
        },
      });

      await expect(fetchUserData(999)).rejects.toThrow(errorMessage);
    });

    test('should handle network errors', async () => {
      mockedAxios.get.mockRejectedValueOnce(new Error('Network Error'));

      await expect(fetchUserData(1)).rejects.toThrow('Network Error');
    });
  });

  describe('fetchPosts', () => {
    test('should fetch posts with pagination', async () => {
      const mockPosts = [
        { id: 1, title: 'Post 1', content: 'Content 1' },
        { id: 2, title: 'Post 2', content: 'Content 2' },
      ];

      const mockResponse = {
        data: {
          posts: mockPosts,
          total: 25,
          page: 1,
          limit: 10,
        },
        status: 200,
      };

      mockedAxios.get.mockResolvedValueOnce(mockResponse);

      const result = await fetchPosts({ page: 1, limit: 10 });

      expect(mockedAxios.get).toHaveBeenCalledWith('/api/posts', {
        params: { page: 1, limit: 10 },
      });
      expect(result.posts).toEqual(mockPosts);
      expect(result.total).toBe(25);
    });
  });
});
```

### **Mocking React Native Modules**
```javascript
// __tests__/components/ImageUploader.test.js
import React from 'react';
import { render, fireEvent, waitFor } from '@testing-library/react-native';
import ImageUploader from '../components/ImageUploader';

// Mock React Native Image Picker
jest.mock('react-native-image-picker', () => ({
  showImagePicker: jest.fn(),
  launchCamera: jest.fn(),
  launchImageLibrary: jest.fn(),
}));

// Mock React Native FS
jest.mock('react-native-fs', () => ({
  readFile: jest.fn(),
  writeFile: jest.fn(),
  unlink: jest.fn(),
  exists: jest.fn(),
}));

// Mock React Native Permissions
jest.mock('react-native-permissions', () => ({
  request: jest.fn(),
  check: jest.fn(),
  RESULTS: {
    GRANTED: 'granted',
    DENIED: 'denied',
    BLOCKED: 'blocked',
  },
}));

const ImagePicker = require('react-native-image-picker');
const RNFS = require('react-native-fs');
const { request, RESULTS } = require('react-native-permissions');

describe('ImageUploader Component', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('should request camera permission on mount', async () => {
    request.mockResolvedValueOnce(RESULTS.GRANTED);

    render(<ImageUploader />);

    await waitFor(() => {
      expect(request).toHaveBeenCalledWith('ios.permission.CAMERA');
    });
  });

  test('should handle successful image selection', async () => {
    const mockImage = {
      uri: 'file://test/image.jpg',
      type: 'image/jpeg',
      fileName: 'test.jpg',
      fileSize: 1024000,
    };

    ImagePicker.showImagePicker.mockImplementationOnce((options, callback) => {
      callback({
        didCancel: false,
        error: null,
        data: mockImage,
      });
    });

    const { getByText } = render(<ImageUploader />);
    fireEvent.press(getByText('Select Image'));

    await waitFor(() => {
      expect(getByText('test.jpg')).toBeTruthy();
    });
  });

  test('should handle camera permission denied', async () => {
    request.mockResolvedValueOnce(RESULTS.DENIED);

    const { getByText } = render(<ImageUploader />);
    fireEvent.press(getByText('Take Photo'));

    await waitFor(() => {
      expect(getByText('Camera permission is required')).toBeTruthy();
    });
  });
});
```

### **Custom Mock Utilities**
```javascript
// __tests__/mocks/mockServer.js
const express = require('express');
const { createServer } = require('http');

class MockServer {
  constructor() {
    this.app = express();
    this.server = null;
    this.port = 0; // Use random available port
    this.setupRoutes();
  }

  setupRoutes() {
    // Mock API routes
    this.app.get('/api/users', (req, res) => {
      res.json({
        users: [
          { id: 1, name: 'John Doe', email: 'john@example.com' },
          { id: 2, name: 'Jane Smith', email: 'jane@example.com' },
        ],
        total: 2,
      });
    });

    this.app.get('/api/users/:id', (req, res) => {
      const userId = parseInt(req.params.id);
      if (userId === 1) {
        res.json({
          id: 1,
          name: 'John Doe',
          email: 'john@example.com',
        });
      } else {
        res.status(404).json({ error: 'User not found' });
      }
    });

    this.app.post('/api/users', (req, res) => {
      const newUser = {
        id: Date.now(),
        ...req.body,
        createdAt: new Date(),
      };
      res.status(201).json(newUser);
    });
  }

  async start() {
    return new Promise((resolve) => {
      this.server = createServer(this.app);
      this.server.listen(this.port, () => {
        this.port = this.server.address().port;
        resolve(`http://localhost:${this.port}`);
      });
    });
  }

  async stop() {
    return new Promise((resolve) => {
      if (this.server) {
        this.server.close(resolve);
      } else {
        resolve();
      }
    });
  }

  getUrl() {
    return `http://localhost:${this.port}`;
  }
}

module.exports = MockServer;
```

---

## 🤖 **Test Automation & CI/CD**

### **GitHub Actions CI/CD Pipeline**
```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

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
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linting
      run: npm run lint

    - name: Run unit tests
      run: npm run test:unit

    - name: Run integration tests
      run: npm run test:integration

    - name: Generate coverage report
      run: npm run test:coverage

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info

  e2e-test:
    runs-on: ubuntu-latest
    needs: test

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 18.x
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Setup MongoDB
      uses: supercharge/mongodb-github-action@1.8.0
      with:
        mongodb-version: 5.0

    - name: Run E2E tests
      run: npm run test:e2e

  deploy:
    runs-on: ubuntu-latest
    needs: [test, e2e-test]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 18.x
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Build application
      run: npm run build

    - name: Deploy to staging
      run: |
        echo "Deploying to staging environment"
        # Add your deployment commands here

    - name: Run smoke tests
      run: npm run test:smoke

    - name: Deploy to production
      if: github.event.inputs.deploy_to_prod == 'true'
      run: |
        echo "Deploying to production environment"
        # Add your production deployment commands here
```

### **Jest Configuration for CI/CD**
```javascript
// jest.config.ci.js
module.exports = {
  preset: 'react-native',
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js',
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html', 'json'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  testTimeout: 30000,
  maxWorkers: '50%',
  bail: true, // Stop on first failure in CI
  verbose: false,
  silent: true,
};
```

### **Parallel Test Execution**
```javascript
// scripts/run-tests-parallel.js
const { execSync } = require('child_process');
const fs = require('fs');
const path = require('path');

function runParallelTests() {
  const testDirs = [
    'src/components',
    'src/hooks',
    'src/utils',
    'src/services',
  ];

  const testCommands = testDirs.map(dir => {
    const testFiles = findTestFiles(dir);
    if (testFiles.length > 0) {
      return `npm test -- ${testFiles.join(' ')}`;
    }
    return null;
  }).filter(Boolean);

  // Run tests in parallel
  const promises = testCommands.map(command => {
    return new Promise((resolve, reject) => {
      try {
        execSync(command, { stdio: 'inherit' });
        resolve();
      } catch (error) {
        reject(error);
      }
    });
  });

  return Promise.all(promises);
}

function findTestFiles(dir) {
  const files = [];
  const items = fs.readdirSync(dir);

  for (const item of items) {
    const fullPath = path.join(dir, item);
    const stat = fs.statSync(fullPath);

    if (stat.isDirectory()) {
      files.push(...findTestFiles(fullPath));
    } else if (item.endsWith('.test.js') || item.endsWith('.spec.js')) {
      files.push(fullPath);
    }
  }

  return files;
}

if (require.main === module) {
  runParallelTests()
    .then(() => {
      console.log('All tests passed!');
      process.exit(0);
    })
    .catch((error) => {
      console.error('Tests failed:', error);
      process.exit(1);
    });
}

module.exports = { runParallelTests };
```

---

## ⚡ **Performance Testing**

### **Load Testing with Artillery**
```javascript
// performance/load-test.yml
config:
  target: 'http://localhost:3000'
  phases:
    - duration: 60
      arrivalRate: 5
      name: "Warm up phase"
    - duration: 120
      arrivalRate: 5
      rampTo: 50
      name: "Ramp up load"
    - duration: 60
      arrivalRate: 50
      name: "Sustained load"

scenarios:
  - name: "User registration and login"
    weight: 30
    flow:
      - post:
          url: "/api/auth/register"
          json:
            firstName: "Load"
            lastName: "Test"
            email: "loadtest{{ $randomInt }}@example.com"
            password: "password123"
          expect:
            - statusCode: 201

  - name: "User login"
    weight: 40
    flow:
      - post:
          url: "/api/auth/login"
          json:
            email: "test@example.com"
            password: "password123"
          expect:
            - statusCode: 200
          capture:
            json: "$.data.token"
            as: "token"

      - get:
          url: "/api/auth/profile"
          headers:
            Authorization: "Bearer {{ token }}"
          expect:
            - statusCode: 200

  - name: "API endpoints"
    weight: 30
    flow:
      - get:
          url: "/api/health"
          expect:
            - statusCode: 200

      - get:
          url: "/api/posts?page=1&limit=10"
          expect:
            - statusCode: 200
```

### **Performance Testing with k6**
```javascript
// performance/k6-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

// Custom metrics
const errorRate = new Rate('errors');
const responseTime = new Trend('response_time');

export const options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp up to 100 users over 2 minutes
    { duration: '5m', target: 100 }, // Stay at 100 users for 5 minutes
    { duration: '2m', target: 200 }, // Ramp up to 200 users over 2 minutes
    { duration: '5m', target: 200 }, // Stay at 200 users for 5 minutes
    { duration: '2m', target: 0 },   // Ramp down to 0 users
  ],
  thresholds: {
    http_req_duration: ['p(99)<500'], // 99% of requests should be below 500ms
    http_req_failed: ['rate<0.1'],    // Error rate should be below 10%
  },
};

const BASE_URL = __ENV.BASE_URL || 'http://localhost:3000';

export default function () {
  // Test user registration
  const registerPayload = {
    firstName: 'Performance',
    lastName: 'Test',
    email: `perf${__VU}_${Date.now()}@example.com`,
    password: 'password123',
  };

  const registerResponse = http.post(
    `${BASE_URL}/api/auth/register`,
    JSON.stringify(registerPayload),
    {
      headers: {
        'Content-Type': 'application/json',
      },
    }
  );

  check(registerResponse, {
    'registration successful': (r) => r.status === 201,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });

  errorRate.add(registerResponse.status !== 201);
  responseTime.add(registerResponse.timings.duration);

  sleep(1);

  // Test login
  const loginPayload = {
    email: 'test@example.com',
    password: 'password123',
  };

  const loginResponse = http.post(
    `${BASE_URL}/api/auth/login`,
    JSON.stringify(loginPayload),
    {
      headers: {
        'Content-Type': 'application/json',
      },
    }
  );

  check(loginResponse, {
    'login successful': (r) => r.status === 200,
    'response time < 300ms': (r) => r.timings.duration < 300,
  });

  errorRate.add(loginResponse.status !== 200);
  responseTime.add(loginResponse.timings.duration);

  sleep(1);

  // Test API endpoints
  const healthResponse = http.get(`${BASE_URL}/api/health`);

  check(healthResponse, {
    'health check successful': (r) => r.status === 200,
    'response time < 100ms': (r) => r.timings.duration < 100,
  });

  errorRate.add(healthResponse.status !== 200);
  responseTime.add(healthResponse.timings.duration);

  sleep(2);
}

export function handleSummary(data) {
  return {
    'stdout': textSummary(data, { indent: ' ', enableColors: true }),
    'performance-report.json': JSON.stringify(data),
  };
}
```

### **Memory Leak Testing**
```javascript
// __tests__/performance/memory-leak.test.js
const { render } = require('@testing-library/react-native');
const React from 'react';

describe('Memory Leak Tests', () => {
  test('should not leak memory in component with timers', async () => {
    let component;

    // Mock performance.memory
    const originalMemory = performance.memory;
    performance.memory = {
      usedJSHeapSize: 0,
      totalJSHeapSize: 0,
      jsHeapSizeLimit: 0,
    };

    const TestComponent = () => {
      const [count, setCount] = React.useState(0);

      React.useEffect(() => {
        const interval = setInterval(() => {
          setCount(c => c + 1);
        }, 100);

        return () => clearInterval(interval);
      }, []);

      return React.createElement('Text', null, `Count: ${count}`);
    };

    // Render component
    component = render(React.createElement(TestComponent));

    // Wait for some updates
    await new Promise(resolve => setTimeout(resolve, 1000));

    // Unmount component
    component.unmount();

    // Force garbage collection (if available)
    if (global.gc) {
      global.gc();
    }

    // Check memory usage
    const memoryUsage = performance.memory.usedJSHeapSize;
    expect(memoryUsage).toBeLessThan(50 * 1024 * 1024); // Less than 50MB

    // Restore original memory
    performance.memory = originalMemory;
  });

  test('should clean up event listeners', () => {
    const mockRemoveListener = jest.fn();
    const mockAddListener = jest.fn(() => mockRemoveListener);

    // Mock Dimensions
    jest.mock('react-native/Libraries/Utilities/Dimensions', () => ({
      addEventListener: mockAddListener,
      removeEventListener: mockRemoveListener,
    }));

    const TestComponent = () => {
      React.useEffect(() => {
        const subscription = require('react-native/Libraries/Utilities/Dimensions')
          .addEventListener('change', () => {});

        return () => subscription();
      }, []);

      return React.createElement('View', null);
    };

    const component = render(React.createElement(TestComponent));
    component.unmount();

    expect(mockRemoveListener).toHaveBeenCalled();
  });
});
```

---

## 📊 **Code Coverage**

### **Coverage Configuration**
```javascript
// jest.config.coverage.js
module.exports = {
  preset: 'react-native',
  collectCoverage: true,
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js',
    '!src/constants/**',
    '!src/types/**',
  ],
  coverageDirectory: 'coverage',
  coverageReporters: [
    'text',
    'lcov',
    'html',
    'json-summary',
    'cobertura',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    './src/components/': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90,
    },
    './src/utils/': {
      branches: 95,
      functions: 95,
      lines: 95,
      statements: 95,
    },
  },
  coveragePathIgnorePatterns: [
    '/node_modules/',
    '/__tests__/',
    '/coverage/',
    '/.jest/',
  ],
};
```

### **Coverage Analysis Script**
```javascript
// scripts/analyze-coverage.js
const fs = require('fs');
const path = require('path');

function analyzeCoverage() {
  const coveragePath = path.join(__dirname, '..', 'coverage', 'coverage-summary.json');

  if (!fs.existsSync(coveragePath)) {
    console.error('Coverage report not found. Run tests with coverage first.');
    process.exit(1);
  }

  const coverage = JSON.parse(fs.readFileSync(coveragePath, 'utf8'));
  const { total, ...files } = coverage;

  console.log('📊 Code Coverage Analysis');
  console.log('========================\n');

  console.log('Global Coverage:');
  console.log(`  Statements: ${total.statements.pct}%`);
  console.log(`  Branches:   ${total.branches.pct}%`);
  console.log(`  Functions:  ${total.functions.pct}%`);
  console.log(`  Lines:      ${total.lines.pct}%\n`);

  // Find files with low coverage
  const lowCoverageFiles = Object.entries(files)
    .filter(([file, data]) => {
      return data.lines.pct < 80 ||
             data.functions.pct < 80 ||
             data.branches.pct < 80 ||
             data.statements.pct < 80;
    })
    .sort(([, a], [, b]) => a.lines.pct - b.lines.pct);

  if (lowCoverageFiles.length > 0) {
    console.log('Files with Low Coverage (< 80%):');
    console.log('================================');

    lowCoverageFiles.forEach(([file, data]) => {
      console.log(`\n${file}:`);
      console.log(`  Statements: ${data.statements.pct}%`);
      console.log(`  Branches:   ${data.branches.pct}%`);
      console.log(`  Functions:  ${data.functions.pct}%`);
      console.log(`  Lines:      ${data.lines.pct}%`);
    });
  } else {
    console.log('🎉 All files meet coverage requirements!');
  }

  // Check if thresholds are met
  const thresholds = {
    statements: 80,
    branches: 80,
    functions: 80,
    lines: 80,
  };

  const failedThresholds = Object.entries(thresholds)
    .filter(([metric]) => total[metric].pct < thresholds[metric]);

  if (failedThresholds.length > 0) {
    console.log('\n❌ Coverage thresholds not met:');
    failedThresholds.forEach(([metric, threshold]) => {
      console.log(`  ${metric}: ${total[metric].pct}% (required: ${threshold}%)`);
    });
    process.exit(1);
  } else {
    console.log('\n✅ All coverage thresholds met!');
  }
}

if (require.main === module) {
  analyzeCoverage();
}

module.exports = { analyzeCoverage };
```

### **Coverage Badge Generation**
```javascript
// scripts/generate-coverage-badge.js
const fs = require('fs');
const path = require('path');

function generateCoverageBadge() {
  const coveragePath = path.join(__dirname, '..', 'coverage', 'coverage-summary.json');

  if (!fs.existsSync(coveragePath)) {
    console.error('Coverage report not found.');
    return;
  }

  const coverage = JSON.parse(fs.readFileSync(coveragePath, 'utf8'));
  const percentage = Math.round(coverage.total.lines.pct);

  // Determine color based on coverage
  let color;
  if (percentage >= 90) {
    color = 'brightgreen';
  } else if (percentage >= 80) {
    color = 'green';
  } else if (percentage >= 70) {
    color = 'yellow';
  } else if (percentage >= 60) {
    color = 'orange';
  } else {
    color = 'red';
  }

  const badgeUrl = `https://img.shields.io/badge/coverage-${percentage}%25-${color}`;

  // Generate README badge
  const badgeMarkdown = `![Coverage](${badgeUrl})`;

  console.log('Coverage Badge:');
  console.log(badgeMarkdown);
  console.log('\nAdd this to your README.md:');
  console.log(`## Coverage\n\n${badgeMarkdown}`);

  // Save to file
  const outputPath = path.join(__dirname, '..', 'coverage', 'badge.md');
  fs.writeFileSync(outputPath, badgeMarkdown);

  return badgeMarkdown;
}

if (require.main === module) {
  generateCoverageBadge();
}

module.exports = { generateCoverageBadge };
```

---

## 🏆 **Testing Best Practices**

### **Test Organization**
```javascript
// __tests__/setup/test-setup.js
const mongoose = require('mongoose');
const { MongoMemoryServer } = require('mongodb-memory-server');

let mongoServer;

// Global test setup
beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  const mongoUri = mongoServer.getUri();
  await mongoose.connect(mongoUri);
});

// Global test teardown
afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});

// Clean up after each test
afterEach(async () => {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});

// Mock external services
jest.mock('../services/emailService', () => ({
  sendWelcomeEmail: jest.fn().mockResolvedValue(true),
  sendPasswordResetEmail: jest.fn().mockResolvedValue(true),
}));

jest.mock('../services/paymentService', () => ({
  processPayment: jest.fn().mockResolvedValue({ success: true, transactionId: 'txn_123' }),
  refundPayment: jest.fn().mockResolvedValue({ success: true, refundId: 'ref_123' }),
}));
```

### **Test Utilities**
```javascript
// __tests__/utils/test-helpers.js
const User = require('../../models/User');
const Post = require('../../models/Post');

/**
 * Create a test user
 */
async function createTestUser(overrides = {}) {
  const defaultUser = {
    firstName: 'Test',
    lastName: 'User',
    email: `test${Date.now()}@example.com`,
    password: 'password123',
  };

  return await User.create({ ...defaultUser, ...overrides });
}

/**
 * Create a test post
 */
async function createTestPost(authorId, overrides = {}) {
  const defaultPost = {
    title: 'Test Post',
    content: 'This is a test post content',
    author: authorId,
  };

  return await Post.create({ ...defaultPost, ...overrides });
}

/**
 * Generate authentication token for test user
 */
function generateTestToken(user) {
  const jwt = require('jsonwebtoken');
  return jwt.sign(
    { id: user.id, email: user.email },
    process.env.JWT_SECRET || 'test-secret',
    { expiresIn: '1h' }
  );
}

/**
 * Create authenticated request
 */
function createAuthenticatedRequest(app, token) {
  const request = require('supertest')(app);
  return {
    get: (url) => request.get(url).set('Authorization', `Bearer ${token}`),
    post: (url) => request.post(url).set('Authorization', `Bearer ${token}`),
    put: (url) => request.put(url).set('Authorization', `Bearer ${token}`),
    delete: (url) => request.delete(url).set('Authorization', `Bearer ${token}`),
  };
}

/**
 * Wait for async operations
 */
function flushPromises() {
  return new Promise(resolve => setImmediate(resolve));
}

/**
 * Mock console methods to reduce noise
 */
function mockConsole() {
  const originalConsole = { ...console };
  beforeAll(() => {
    console.log = jest.fn();
    console.error = jest.fn();
    console.warn = jest.fn();
  });

  afterAll(() => {
    Object.assign(console, originalConsole);
  });
}

module.exports = {
  createTestUser,
  createTestPost,
  generateTestToken,
  createAuthenticatedRequest,
  flushPromises,
  mockConsole,
};
```

### **Test Naming Conventions**
```javascript
// ✅ Good test names
describe('User Registration', () => {
  test('should create user with valid data', () => {});
  test('should hash password before storing', () => {});
  test('should not create user with existing email', () => {});
  test('should validate required fields', () => {});
  test('should validate email format', () => {});
  test('should validate password strength', () => {});
  test('should send welcome email after registration', () => {});
});

// ❌ Bad test names
describe('User tests', () => {
  test('test user creation', () => {});
  test('user password', () => {});
  test('user email exists', () => {});
  test('validation', () => {});
  test('email validation', () => {});
  test('password validation', () => {});
  test('email sending', () => {});
});
```

### **Test Data Management**
```javascript
// __tests__/fixtures/test-data.js
const users = {
  admin: {
    firstName: 'Admin',
    lastName: 'User',
    email: 'admin@example.com',
    password: 'admin123',
    role: 'admin',
  },

  moderator: {
    firstName: 'Moderator',
    lastName: 'User',
    email: 'moderator@example.com',
    password: 'mod123',
    role: 'moderator',
  },

  regular: {
    firstName: 'Regular',
    lastName: 'User',
    email: 'user@example.com',
    password: 'user123',
    role: 'user',
  },
};

const posts = {
  published: {
    title: 'Published Post',
    content: 'This is a published post',
    status: 'published',
  },

  draft: {
    title: 'Draft Post',
    content: 'This is a draft post',
    status: 'draft',
  },

  archived: {
    title: 'Archived Post',
    content: 'This is an archived post',
    status: 'archived',
  },
};

const categories = {
  technology: {
    name: 'Technology',
    slug: 'technology',
    description: 'Posts about technology',
  },

  lifestyle: {
    name: 'Lifestyle',
    slug: 'lifestyle',
    description: 'Posts about lifestyle',
  },
};

module.exports = {
  users,
  posts,
  categories,
};
```

---

## 🛠️ **Practical Examples**

### **Complete Testing Setup**
```javascript
// package.json test scripts
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:unit": "jest --testPathPattern=unit",
    "test:integration": "jest --testPathPattern=integration",
    "test:e2e": "jest --testPathPattern=e2e",
    "test:smoke": "jest --testPathPattern=smoke",
    "test:performance": "artillery run performance/load-test.yml",
    "lint": "eslint src --ext .js,.jsx,.ts,.tsx",
    "lint:fix": "eslint src --ext .js,.jsx,.ts,.tsx --fix",
    "type-check": "tsc --noEmit"
  }
}
```

### **Testing Workflow**
```javascript
// scripts/test-workflow.js
const { execSync } = require('child_process');

function runTestWorkflow() {
  const steps = [
    {
      name: 'Linting',
      command: 'npm run lint',
      description: 'Check code style and potential issues',
    },
    {
      name: 'Type Checking',
      command: 'npm run type-check',
      description: 'Verify TypeScript types',
    },
    {
      name: 'Unit Tests',
      command: 'npm run test:unit',
      description: 'Run unit tests',
    },
    {
      name: 'Integration Tests',
      command: 'npm run test:integration',
      description: 'Run integration tests',
    },
    {
      name: 'Coverage Analysis',
      command: 'node scripts/analyze-coverage.js',
      description: 'Analyze test coverage',
    },
    {
      name: 'E2E Tests',
      command: 'npm run test:e2e',
      description: 'Run end-to-end tests',
    },
  ];

  console.log('🚀 Starting Test Workflow\n');

  for (const step of steps) {
    console.log(`📋 ${step.name}`);
    console.log(`   ${step.description}`);

    try {
      execSync(step.command, { stdio: 'inherit' });
      console.log(`✅ ${step.name} passed\n`);
    } catch (error) {
      console.log(`❌ ${step.name} failed\n`);
      console.error(error.message);
      process.exit(1);
    }
  }

  console.log('🎉 All tests passed!');
}

if (require.main === module) {
  runTestWorkflow();
}

module.exports = { runTestWorkflow };
```

### **Test Reporting**
```javascript
// scripts/generate-test-report.js
const fs = require('fs');
const path = require('path');

function generateTestReport() {
  const coveragePath = path.join(__dirname, '..', 'coverage', 'coverage-summary.json');
  const testResultsPath = path.join(__dirname, '..', 'test-results.json');

  const report = {
    timestamp: new Date().toISOString(),
    summary: {},
    details: {},
  };

  // Read coverage data
  if (fs.existsSync(coveragePath)) {
    const coverage = JSON.parse(fs.readFileSync(coveragePath, 'utf8'));
    report.summary.coverage = {
      statements: coverage.total.statements.pct,
      branches: coverage.total.branches.pct,
      functions: coverage.total.functions.pct,
      lines: coverage.total.lines.pct,
    };
  }

  // Read test results
  if (fs.existsSync(testResultsPath)) {
    const testResults = JSON.parse(fs.readFileSync(testResultsPath, 'utf8'));
    report.summary.tests = {
      total: testResults.numTotalTests,
      passed: testResults.numPassedTests,
      failed: testResults.numFailedTests,
      duration: testResults.testResults[0]?.perfStats?.runtime,
    };
  }

  // Generate HTML report
  const htmlReport = generateHTMLReport(report);

  // Save reports
  fs.writeFileSync(
    path.join(__dirname, '..', 'test-report.json'),
    JSON.stringify(report, null, 2)
  );

  fs.writeFileSync(
    path.join(__dirname, '..', 'test-report.html'),
    htmlReport
  );

  console.log('📊 Test report generated:');
  console.log('  - test-report.json');
  console.log('  - test-report.html');

  return report;
}

function generateHTMLReport(report) {
  return `
<!DOCTYPE html>
<html>
<head>
  <title>Test Report</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    .summary { background: #f5f5f5; padding: 20px; border-radius: 5px; }
    .metric { margin: 10px 0; }
    .passed { color: green; }
    .failed { color: red; }
  </style>
</head>
<body>
  <h1>Test Report</h1>
  <p>Generated: ${report.timestamp}</p>

  <div class="summary">
    <h2>Summary</h2>

    ${report.summary.coverage ? `
    <h3>Coverage</h3>
    <div class="metric">Statements: ${report.summary.coverage.statements}%</div>
    <div class="metric">Branches: ${report.summary.coverage.branches}%</div>
    <div class="metric">Functions: ${report.summary.coverage.functions}%</div>
    <div class="metric">Lines: ${report.summary.coverage.lines}%</div>
    ` : ''}

    ${report.summary.tests ? `
    <h3>Tests</h3>
    <div class="metric">Total: ${report.summary.tests.total}</div>
    <div class="metric class="passed"">Passed: ${report.summary.tests.passed}</div>
    <div class="metric class="failed"">Failed: ${report.summary.tests.failed}</div>
    <div class="metric">Duration: ${report.summary.tests.duration}ms</div>
    ` : ''}
  </div>
</body>
</html>
  `;
}

if (require.main === module) {
  generateTestReport();
}

module.exports = { generateTestReport };
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Testing Pyramid**: Unit, integration, and end-to-end tests
- ✅ **Jest Framework**: Configuration, mocking, and assertions
- ✅ **React Native Testing**: Component and hook testing
- ✅ **Integration Testing**: API and database testing
- ✅ **End-to-End Testing**: Detox for mobile app testing
- ✅ **API Testing**: Supertest for backend API testing
- ✅ **Test-Driven Development**: Writing tests before code
- ✅ **Mocking & Stubbing**: Isolating code for testing
- ✅ **CI/CD Integration**: Automated testing pipelines
- ✅ **Performance Testing**: Load and memory leak testing
- ✅ **Code Coverage**: Measuring test effectiveness
- ✅ **Testing Best Practices**: Organization and maintenance

### **Best Practices**
1. **Write tests first** - Follow TDD principles
2. **Keep tests isolated** - Each test should be independent
3. **Use descriptive names** - Tests should explain what they're testing
4. **Mock external dependencies** - Focus on testing your code
5. **Maintain test coverage** - Aim for 80%+ coverage
6. **Run tests frequently** - Integrate into CI/CD pipeline
7. **Test edge cases** - Don't just test happy paths
8. **Keep tests fast** - Slow tests discourage running them
9. **Use test utilities** - Create reusable test helpers
10. **Review test failures** - Learn from test failures

### **Next Steps**
- Implement advanced mocking techniques
- Set up visual regression testing
- Learn about property-based testing
- Explore test automation frameworks
- Study performance monitoring tools
- Implement chaos engineering practices

---

## 🎯 **Assignment**

### **Task 1: Unit Testing Suite**
Create a comprehensive unit testing suite for a React Native app with:
- Component tests for all UI components
- Custom hook tests with various scenarios
- Utility function tests with edge cases
- Redux action and reducer tests
- Service layer tests with mocking
- 80%+ code coverage achievement

### **Task 2: Integration Testing**
Build integration tests for a Node.js API including:
- Database integration tests
- API endpoint integration tests
- Authentication flow tests
- File upload integration tests
- Email service integration tests
- Payment processing integration tests

### **Task 3: E2E Testing Setup**
Implement end-to-end testing for a mobile app featuring:
- User registration and login flows
- Navigation between screens
- Form submissions and validations
- Data synchronization tests
- Offline functionality tests
- Performance testing scenarios

### **Task 4: CI/CD Pipeline**
Create a complete CI/CD pipeline that includes:
- Automated linting and type checking
- Parallel test execution
- Code coverage reporting
- Performance regression testing
- Automated deployment to staging
- Manual approval for production deployment

---

## 📚 **Additional Resources**
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Detox Documentation](https://wix.github.io/Detox/)
- [Supertest Documentation](https://github.com/visionmedia/supertest)
- [Artillery Documentation](https://artillery.io/docs/)
- [k6 Documentation](https://k6.io/docs/)

---

**Next Lesson**: [Lesson 28: Advanced React Native Patterns](Lesson%2028_%20Advanced%20React%20Native%20Patterns.md)