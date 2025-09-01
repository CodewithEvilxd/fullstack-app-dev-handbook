# Lesson 34: Context API & Global State Management

## 🎯 **Learning Objectives**
- Master React Context API for state management
- Implement global state with useContext and useReducer
- Create custom hooks for state management
- Handle complex state scenarios with Context
- Optimize Context performance with memoization

## 📚 **Table of Contents**
1. [Introduction to Context API](#introduction-to-context-api)
2. [Creating Context Providers](#creating-context-providers)
3. [useContext Hook](#usecontext-hook)
4. [useReducer with Context](#usereducer-with-context)
5. [Custom Hooks for Context](#custom-hooks-for-context)
6. [Context Composition](#context-composition)
7. [Performance Optimization](#performance-optimization)
8. [Error Boundaries with Context](#error-boundaries-with-context)
9. [Testing Context](#testing-context)
10. [Practical Examples](#practical-examples)

---

## 🌐 **Introduction to Context API**

### **What is Context API?**
Context API is a React feature that allows you to share state across the component tree without passing props down manually at every level.

### **When to Use Context**
- **Global state**: User authentication, theme, language
- **Shared state**: Shopping cart, notifications, app settings
- **Deep prop drilling**: When props are passed through many levels
- **Theme/state management**: App-wide configuration

### **Context vs Redux**
```
Context API:
✅ Simple to set up
✅ Built into React
✅ Good for small to medium apps
✅ Less boilerplate
❌ Can cause unnecessary re-renders
❌ No built-in debugging tools

Redux:
✅ Predictable state updates
✅ Powerful debugging tools
✅ Large ecosystem
✅ Better performance for complex apps
❌ More boilerplate
❌ Steeper learning curve
```

---

## 🏗️ **Creating Context Providers**

### **Basic Context Setup**
```javascript
// contexts/AuthContext.js
import React, { createContext, useState, useContext } from 'react';

// Create context
const AuthContext = createContext();

// Create provider component
export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  // Authentication methods
  const login = async (email, password) => {
    setIsLoading(true);
    setError(null);

    try {
      // Simulate API call
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });

      if (!response.ok) {
        throw new Error('Login failed');
      }

      const userData = await response.json();
      setUser(userData);
    } catch (error) {
      setError(error.message);
    } finally {
      setIsLoading(false);
    }
  };

  const logout = () => {
    setUser(null);
    setError(null);
  };

  const register = async (userData) => {
    setIsLoading(true);
    setError(null);

    try {
      const response = await fetch('/api/auth/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData),
      });

      if (!response.ok) {
        throw new Error('Registration failed');
      }

      const newUser = await response.json();
      setUser(newUser);
    } catch (error) {
      setError(error.message);
    } finally {
      setIsLoading(false);
    }
  };

  // Context value
  const value = {
    user,
    isLoading,
    error,
    login,
    logout,
    register,
    isAuthenticated: !!user,
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};

// Custom hook to use auth context
export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

### **Theme Context**
```javascript
// contexts/ThemeContext.js
import React, { createContext, useState, useContext, useEffect } from 'react';
import { Appearance, ColorSchemeName } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

const THEME_STORAGE_KEY = '@app_theme';

const ThemeContext = createContext();

export const themes = {
  light: {
    name: 'light',
    colors: {
      primary: '#007AFF',
      secondary: '#5856D6',
      background: '#FFFFFF',
      surface: '#F2F2F7',
      text: '#000000',
      textSecondary: '#3C3C43',
      border: '#C6C6C8',
      error: '#FF3B30',
      success: '#34C759',
    },
  },
  dark: {
    name: 'dark',
    colors: {
      primary: '#0A84FF',
      secondary: '#5E5CE6',
      background: '#000000',
      surface: '#1C1C1E',
      text: '#FFFFFF',
      textSecondary: '#EBEBF5',
      border: '#38383A',
      error: '#FF453A',
      success: '#30D158',
    },
  },
};

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState(themes.light);
  const [isLoading, setIsLoading] = useState(true);

  // Load saved theme on mount
  useEffect(() => {
    loadSavedTheme();
  }, []);

  // Listen to system theme changes
  useEffect(() => {
    const subscription = Appearance.addChangeListener(({ colorScheme }) => {
      if (theme.name === 'system') {
        setTheme(colorScheme === 'dark' ? themes.dark : themes.light);
      }
    });

    return () => subscription?.remove();
  }, [theme.name]);

  const loadSavedTheme = async () => {
    try {
      const savedTheme = await AsyncStorage.getItem(THEME_STORAGE_KEY);
      if (savedTheme) {
        if (savedTheme === 'system') {
          const colorScheme = Appearance.getColorScheme();
          setTheme(colorScheme === 'dark' ? themes.dark : themes.light);
        } else {
          setTheme(themes[savedTheme] || themes.light);
        }
      } else {
        // Use system theme by default
        const colorScheme = Appearance.getColorScheme();
        setTheme(colorScheme === 'dark' ? themes.dark : themes.light);
      }
    } catch (error) {
      console.error('Error loading theme:', error);
    } finally {
      setIsLoading(false);
    }
  };

  const setThemeMode = async (themeName) => {
    try {
      let newTheme;
      if (themeName === 'system') {
        const colorScheme = Appearance.getColorScheme();
        newTheme = colorScheme === 'dark' ? themes.dark : themes.light;
        newTheme.name = 'system';
      } else {
        newTheme = themes[themeName] || themes.light;
      }

      setTheme(newTheme);
      await AsyncStorage.setItem(THEME_STORAGE_KEY, themeName);
    } catch (error) {
      console.error('Error saving theme:', error);
    }
  };

  const toggleTheme = () => {
    const newThemeName = theme.name === 'light' ? 'dark' : 'light';
    setThemeMode(newThemeName);
  };

  const value = {
    theme,
    setThemeMode,
    toggleTheme,
    isLoading,
  };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

---

## 🎣 **useContext Hook**

### **Basic Usage**
```javascript
// components/LoginForm.js
import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, StyleSheet } from 'react-native';
import { useAuth } from '../contexts/AuthContext';

const LoginForm = () => {
  const { login, isLoading, error } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = async () => {
    if (!email || !password) {
      alert('Please fill in all fields');
      return;
    }

    try {
      await login(email, password);
    } catch (error) {
      // Error is handled by context
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Login</Text>

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

      {error && (
        <Text style={styles.error}>{error}</Text>
      )}

      <TouchableOpacity
        style={[styles.button, isLoading && styles.disabledButton]}
        onPress={handleLogin}
        disabled={isLoading}
      >
        <Text style={styles.buttonText}>
          {isLoading ? 'Logging in...' : 'Login'}
        </Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    justifyContent: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 15,
    marginBottom: 15,
    fontSize: 16,
  },
  error: {
    color: 'red',
    marginBottom: 15,
    textAlign: 'center',
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  disabledButton: {
    backgroundColor: '#ccc',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default LoginForm;
```

### **Multiple Contexts Usage**
```javascript
// components/UserProfile.js
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { useAuth } from '../contexts/AuthContext';
import { useTheme } from '../contexts/ThemeContext';

const UserProfile = () => {
  const { user, logout } = useAuth();
  const { theme, toggleTheme } = useTheme();

  if (!user) {
    return (
      <View style={styles.container}>
        <Text style={styles.message}>Please login to view your profile</Text>
      </View>
    );
  }

  return (
    <View style={[styles.container, { backgroundColor: theme.colors.background }]}>
      <View style={[styles.profileCard, { backgroundColor: theme.colors.surface }]}>
        <Text style={[styles.name, { color: theme.colors.text }]}>
          {user.firstName} {user.lastName}
        </Text>
        <Text style={[styles.email, { color: theme.colors.textSecondary }]}>
          {user.email}
        </Text>
        <Text style={[styles.role, { color: theme.colors.primary }]}>
          {user.role || 'User'}
        </Text>
      </View>

      <View style={styles.actions}>
        <TouchableOpacity
          style={[styles.button, { backgroundColor: theme.colors.primary }]}
          onPress={toggleTheme}
        >
          <Text style={[styles.buttonText, { color: theme.colors.background }]}>
            Toggle Theme
          </Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={[styles.button, { backgroundColor: theme.colors.error }]}
          onPress={logout}
        >
          <Text style={[styles.buttonText, { color: theme.colors.background }]}>
            Logout
          </Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  message: {
    fontSize: 16,
    textAlign: 'center',
    marginTop: 50,
  },
  profileCard: {
    padding: 20,
    borderRadius: 12,
    marginBottom: 30,
    alignItems: 'center',
  },
  name: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  email: {
    fontSize: 16,
    marginBottom: 8,
  },
  role: {
    fontSize: 14,
    fontWeight: 'bold',
  },
  actions: {
    gap: 15,
  },
  button: {
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  buttonText: {
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default UserProfile;
```

---

## 🔄 **useReducer with Context**

### **Complex State Management**
```javascript
// contexts/TodoContext.js
import React, { createContext, useContext, useReducer, useEffect } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

const TodoContext = createContext();

// Action types
const ADD_TODO = 'ADD_TODO';
const TOGGLE_TODO = 'TOGGLE_TODO';
const DELETE_TODO = 'DELETE_TODO';
const SET_FILTER = 'SET_FILTER';
const SET_LOADING = 'SET_LOADING';
const SET_ERROR = 'SET_ERROR';
const LOAD_TODOS = 'LOAD_TODOS';

// Initial state
const initialState = {
  todos: [],
  filter: 'all', // all, active, completed
  isLoading: false,
  error: null,
};

// Reducer
const todoReducer = (state, action) => {
  switch (action.type) {
    case LOAD_TODOS:
      return {
        ...state,
        todos: action.payload,
        isLoading: false,
        error: null,
      };

    case ADD_TODO:
      return {
        ...state,
        todos: [...state.todos, {
          id: Date.now().toString(),
          text: action.payload,
          completed: false,
          createdAt: new Date().toISOString(),
        }],
      };

    case TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        ),
      };

    case DELETE_TODO:
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload),
      };

    case SET_FILTER:
      return {
        ...state,
        filter: action.payload,
      };

    case SET_LOADING:
      return {
        ...state,
        isLoading: action.payload,
      };

    case SET_ERROR:
      return {
        ...state,
        error: action.payload,
        isLoading: false,
      };

    default:
      return state;
  }
};

// Actions
const loadTodos = (todos) => ({ type: LOAD_TODOS, payload: todos });
const addTodo = (text) => ({ type: ADD_TODO, payload: text });
const toggleTodo = (id) => ({ type: TOGGLE_TODO, payload: id });
const deleteTodo = (id) => ({ type: DELETE_TODO, payload: id });
const setFilter = (filter) => ({ type: SET_FILTER, payload: filter });
const setLoading = (loading) => ({ type: SET_LOADING, payload: loading });
const setError = (error) => ({ type: SET_ERROR, payload: error });

export const TodoProvider = ({ children }) => {
  const [state, dispatch] = useReducer(todoReducer, initialState);

  // Load todos from storage on mount
  useEffect(() => {
    const loadStoredTodos = async () => {
      try {
        dispatch(setLoading(true));
        const storedTodos = await AsyncStorage.getItem('@todos');
        if (storedTodos) {
          const todos = JSON.parse(storedTodos);
          dispatch(loadTodos(todos));
        }
      } catch (error) {
        dispatch(setError('Failed to load todos'));
      }
    };

    loadStoredTodos();
  }, []);

  // Save todos to storage whenever todos change
  useEffect(() => {
    const saveTodos = async () => {
      try {
        await AsyncStorage.setItem('@todos', JSON.stringify(state.todos));
      } catch (error) {
        console.error('Failed to save todos:', error);
      }
    };

    if (state.todos.length > 0) {
      saveTodos();
    }
  }, [state.todos]);

  // Computed values
  const filteredTodos = state.todos.filter(todo => {
    switch (state.filter) {
      case 'active':
        return !todo.completed;
      case 'completed':
        return todo.completed;
      default:
        return true;
    }
  });

  const stats = {
    total: state.todos.length,
    active: state.todos.filter(todo => !todo.completed).length,
    completed: state.todos.filter(todo => todo.completed).length,
  };

  const value = {
    // State
    todos: filteredTodos,
    filter: state.filter,
    isLoading: state.isLoading,
    error: state.error,
    stats,

    // Actions
    addTodo: (text) => dispatch(addTodo(text)),
    toggleTodo: (id) => dispatch(toggleTodo(id)),
    deleteTodo: (id) => dispatch(deleteTodo(id)),
    setFilter: (filter) => dispatch(setFilter(filter)),
    clearError: () => dispatch(setError(null)),
  };

  return (
    <TodoContext.Provider value={value}>
      {children}
    </TodoContext.Provider>
  );
};

export const useTodos = () => {
  const context = useContext(TodoContext);
  if (!context) {
    throw new Error('useTodos must be used within a TodoProvider');
  }
  return context;
};
```

### **Async Actions with useReducer**
```javascript
// contexts/ShoppingCartContext.js
import React, { createContext, useContext, useReducer, useEffect } from 'react';

// Action types
const ADD_TO_CART = 'ADD_TO_CART';
const REMOVE_FROM_CART = 'REMOVE_FROM_CART';
const UPDATE_QUANTITY = 'UPDATE_QUANTITY';
const CLEAR_CART = 'CLEAR_CART';
const SET_LOADING = 'SET_LOADING';
const SET_ERROR = 'SET_ERROR';
const LOAD_CART = 'LOAD_CART';

// Initial state
const initialState = {
  items: [],
  total: 0,
  isLoading: false,
  error: null,
};

// Reducer
const cartReducer = (state, action) => {
  switch (action.type) {
    case LOAD_CART:
      return {
        ...state,
        items: action.payload.items,
        total: action.payload.total,
        isLoading: false,
      };

    case ADD_TO_CART:
      const existingItem = state.items.find(item => item.id === action.payload.id);
      let newItems;

      if (existingItem) {
        newItems = state.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      } else {
        newItems = [...state.items, { ...action.payload, quantity: 1 }];
      }

      return {
        ...state,
        items: newItems,
        total: calculateTotal(newItems),
      };

    case REMOVE_FROM_CART:
      const filteredItems = state.items.filter(item => item.id !== action.payload);
      return {
        ...state,
        items: filteredItems,
        total: calculateTotal(filteredItems),
      };

    case UPDATE_QUANTITY:
      const updatedItems = state.items.map(item =>
        item.id === action.payload.id
          ? { ...item, quantity: Math.max(0, action.payload.quantity) }
          : item
      ).filter(item => item.quantity > 0);

      return {
        ...state,
        items: updatedItems,
        total: calculateTotal(updatedItems),
      };

    case CLEAR_CART:
      return {
        ...state,
        items: [],
        total: 0,
      };

    case SET_LOADING:
      return {
        ...state,
        isLoading: action.payload,
      };

    case SET_ERROR:
      return {
        ...state,
        error: action.payload,
        isLoading: false,
      };

    default:
      return state;
  }
};

// Helper function
const calculateTotal = (items) => {
  return items.reduce((total, item) => total + item.price * item.quantity, 0);
};

// Actions
const loadCart = (cartData) => ({ type: LOAD_CART, payload: cartData });
const addToCart = (product) => ({ type: ADD_TO_CART, payload: product });
const removeFromCart = (productId) => ({ type: REMOVE_FROM_CART, payload: productId });
const updateQuantity = (productId, quantity) => ({
  type: UPDATE_QUANTITY,
  payload: { id: productId, quantity },
});
const clearCart = () => ({ type: CLEAR_CART });
const setLoading = (loading) => ({ type: SET_LOADING, payload: loading });
const setError = (error) => ({ type: SET_ERROR, payload: error });

const ShoppingCartContext = createContext();

export const ShoppingCartProvider = ({ children }) => {
  const [state, dispatch] = useReducer(cartReducer, initialState);

  // Load cart from API on mount
  useEffect(() => {
    const loadUserCart = async () => {
      try {
        dispatch(setLoading(true));
        // Simulate API call
        const cartData = await new Promise(resolve =>
          setTimeout(() => resolve({ items: [], total: 0 }), 1000)
        );
        dispatch(loadCart(cartData));
      } catch (error) {
        dispatch(setError('Failed to load cart'));
      }
    };

    loadUserCart();
  }, []);

  // Sync cart with server when items change
  useEffect(() => {
    const syncCart = async () => {
      try {
        // Simulate API call to sync cart
        await new Promise(resolve => setTimeout(resolve, 500));
        console.log('Cart synced with server');
      } catch (error) {
        console.error('Failed to sync cart:', error);
      }
    };

    if (state.items.length > 0) {
      syncCart();
    }
  }, [state.items]);

  const value = {
    // State
    items: state.items,
    total: state.total,
    itemCount: state.items.reduce((count, item) => count + item.quantity, 0),
    isLoading: state.isLoading,
    error: state.error,

    // Actions
    addToCart: (product) => dispatch(addToCart(product)),
    removeFromCart: (productId) => dispatch(removeFromCart(productId)),
    updateQuantity: (productId, quantity) => dispatch(updateQuantity(productId, quantity)),
    clearCart: () => dispatch(clearCart()),
    clearError: () => dispatch(setError(null)),
  };

  return (
    <ShoppingCartContext.Provider value={value}>
      {children}
    </ShoppingCartContext.Provider>
  );
};

export const useShoppingCart = () => {
  const context = useContext(ShoppingCartContext);
  if (!context) {
    throw new Error('useShoppingCart must be used within a ShoppingCartProvider');
  }
  return context;
};
```

---

## 🎣 **Custom Hooks for Context**

### **Advanced Custom Hooks**
```javascript
// hooks/useLocalStorage.js
import { useState, useEffect } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

export const useLocalStorage = (key, initialValue) => {
  const [storedValue, setStoredValue] = useState(initialValue);
  const [isLoading, setIsLoading] = useState(true);

  // Load from storage on mount
  useEffect(() => {
    const loadValue = async () => {
      try {
        const item = await AsyncStorage.getItem(key);
        if (item !== null) {
          setStoredValue(JSON.parse(item));
        }
      } catch (error) {
        console.error(`Error loading ${key} from storage:`, error);
      } finally {
        setIsLoading(false);
      }
    };

    loadValue();
  }, [key]);

  // Save to storage when value changes
  const setValue = async (value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      await AsyncStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(`Error saving ${key} to storage:`, error);
    }
  };

  const removeValue = async () => {
    try {
      setStoredValue(initialValue);
      await AsyncStorage.removeItem(key);
    } catch (error) {
      console.error(`Error removing ${key} from storage:`, error);
    }
  };

  return [storedValue, setValue, removeValue, isLoading];
};

// hooks/useAsync.js
import { useState, useEffect, useCallback } from 'react';

export const useAsync = (asyncFunction, immediate = true) => {
  const [status, setStatus] = useState('idle');
  const [value, setValue] = useState(null);
  const [error, setError] = useState(null);

  const execute = useCallback(async (...args) => {
    setStatus('pending');
    setValue(null);
    setError(null);

    try {
      const response = await asyncFunction(...args);
      setValue(response);
      setStatus('success');
      return response;
    } catch (error) {
      setError(error);
      setStatus('error');
      throw error;
    }
  }, [asyncFunction]);

  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);

  const reset = useCallback(() => {
    setStatus('idle');
    setValue(null);
    setError(null);
  }, []);

  return { execute, status, value, error, reset };
};

// hooks/useDebounce.js
import { useState, useEffect } from 'react';

export const useDebounce = (value, delay) => {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
};
```

### **Context-Aware Hooks**
```javascript
// hooks/useAuthActions.js
import { useAuth } from '../contexts/AuthContext';
import { useAsync } from './useAsync';

export const useAuthActions = () => {
  const { login, register, logout } = useAuth();

  const loginAsync = useAsync(login, false);
  const registerAsync = useAsync(register, false);

  const handleLogin = async (credentials) => {
    try {
      await loginAsync.execute(credentials);
      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  };

  const handleRegister = async (userData) => {
    try {
      await registerAsync.execute(userData);
      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  };

  const handleLogout = () => {
    logout();
  };

  return {
    handleLogin,
    handleRegister,
    handleLogout,
    isLoggingIn: loginAsync.status === 'pending',
    isRegistering: registerAsync.status === 'pending',
    loginError: loginAsync.error,
    registerError: registerAsync.error,
  };
};

// hooks/useForm.js
import { useState, useCallback } from 'react';

export const useForm = (initialValues = {}, validate) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const setValue = useCallback((field, value) => {
    setValues(prev => ({ ...prev, [field]: value }));

    // Clear error when user starts typing
    if (errors[field]) {
      setErrors(prev => ({ ...prev, [field]: null }));
    }
  }, [errors]);

  const setTouched = useCallback((field) => {
    setTouched(prev => ({ ...prev, [field]: true }));
  }, []);

  const validateForm = useCallback(() => {
    if (!validate) return true;

    const validationErrors = validate(values);
    setErrors(validationErrors);

    return Object.keys(validationErrors).length === 0;
  }, [values, validate]);

  const handleSubmit = useCallback(async (onSubmit) => {
    setIsSubmitting(true);

    const isValid = validateForm();
    if (!isValid) {
      setIsSubmitting(false);
      return;
    }

    try {
      await onSubmit(values);
    } catch (error) {
      console.error('Form submission error:', error);
    } finally {
      setIsSubmitting(false);
    }
  }, [values, validateForm]);

  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  }, [initialValues]);

  const isValid = Object.keys(errors).length === 0;
  const isDirty = JSON.stringify(values) !== JSON.stringify(initialValues);

  return {
    values,
    errors,
    touched,
    isSubmitting,
    isValid,
    isDirty,
    setValue,
    setTouched,
    handleSubmit,
    reset,
    validateForm,
  };
};
```

---

## 🧩 **Context Composition**

### **Context Composition Pattern**
```javascript
// contexts/AppProviders.js
import React from 'react';
import { AuthProvider } from './AuthContext';
import { ThemeProvider } from './ThemeContext';
import { TodoProvider } from './TodoContext';
import { ShoppingCartProvider } from './ShoppingCartContext';
import { NetworkProvider } from './NetworkContext';

export const AppProviders = ({ children }) => {
  return (
    <NetworkProvider>
      <ThemeProvider>
        <AuthProvider>
          <TodoProvider>
            <ShoppingCartProvider>
              {children}
            </ShoppingCartProvider>
          </TodoProvider>
        </AuthProvider>
      </ThemeProvider>
    </NetworkProvider>
  );
};

// App.js
import React from 'react';
import { AppProviders } from './contexts/AppProviders';
import AppNavigator from './navigation/AppNavigator';
const App = () => {
  return (
    <AppProviders>
      <NavigationContainer>
        <AppNavigator />
      </NavigationContainer>
    </AppProviders>
  );
};

export default App;

// contexts/AppProviders.js
import React from 'react';
import { AuthProvider } from './AuthContext';
import { ThemeProvider } from './ThemeContext';
import { ShoppingCartProvider } from './ShoppingCartContext';
import { NetworkProvider } from './NetworkContext';
import { ErrorBoundaryProvider } from './ErrorBoundaryContext';

export const AppProviders = ({ children }) => {
  return (
    <ErrorBoundaryProvider>
      <NetworkProvider>
        <ThemeProvider>
          <AuthProvider>
            <ShoppingCartProvider>
              {children}
            </ShoppingCartProvider>
          </AuthProvider>
        </ThemeProvider>
      </NetworkProvider>
    </ErrorBoundaryProvider>
  );
};

// contexts/NetworkContext.js
import React, { createContext, useContext, useState, useEffect } from 'react';
import NetInfo from '@react-native-community/netinfo';

const NetworkContext = createContext();

export const NetworkProvider = ({ children }) => {
  const [isConnected, setIsConnected] = useState(true);
  const [connectionType, setConnectionType] = useState('unknown');

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(state => {
      setIsConnected(state.isConnected);
      setConnectionType(state.type);
    });

    return () => unsubscribe();
  }, []);

  const value = {
    isConnected,
    connectionType,
    isOnline: isConnected && connectionType !== 'none',
  };

  return (
    <NetworkContext.Provider value={value}>
      {children}
    </NetworkContext.Provider>
  );
};

export const useNetwork = () => {
  const context = useContext(NetworkContext);
  if (!context) {
    throw new Error('useNetwork must be used within a NetworkProvider');
  }
  return context;
};
  return (
    <AppProviders>
      <AppNavigator />
    </AppProviders>
  );
};

export default App;
```

### **Selective Context Consumption**
```javascript
// components/Providers.js
import React, { createContext, useContext } from 'react';

// Create a context for selective consumption
const SelectiveContext = createContext();

export const SelectiveProvider = ({ contexts, children }) => {
  const contextValues = {};

  // Collect values from specified contexts
  contexts.forEach(contextName => {
    switch (contextName) {
      case 'auth':
        contextValues.auth = useAuth();
        break;
      case 'theme':
        contextValues.theme = useTheme();
        break;
      case 'todos':
        contextValues.todos = useTodos();
        break;
      case 'cart':
        contextValues.cart = useShoppingCart();
        break;
    }
  });

  return (
    <SelectiveContext.Provider value={contextValues}>
      {children}
    </SelectiveContext.Provider>
  );
};

export const useSelectiveContext = () => {
  return useContext(SelectiveContext);
};

// Usage
const SomeComponent = () => {
  const { auth, theme } = useSelectiveContext();

  return (
    <View style={{ backgroundColor: theme.colors.background }}>
      {auth.isAuthenticated && <Text>Welcome!</Text>}
    </View>
  );
};

// In App.js
<SelectiveProvider contexts={['auth', 'theme']}>
  <SomeComponent />
</SelectiveProvider>
```

---

## ⚡ **Performance Optimization**

### **Memoization with Context**
```javascript
// contexts/OptimizedContext.js
import React, { createContext, useContext, useMemo, useCallback } from 'react';

const OptimizedContext = createContext();

export const OptimizedProvider = ({ children }) => {
  const [count, setCount] = useState(0);
  const [user, setUser] = useState({ name: 'John', age: 30 });

  // Memoize expensive calculations
  const expensiveValue = useMemo(() => {
    console.log('Computing expensive value...');
    return count * 2 + user.age;
  }, [count, user.age]);

  // Memoize functions to prevent unnecessary re-renders
  const increment = useCallback(() => {
    setCount(c => c + 1);
  }, []);

  const updateUser = useCallback((updates) => {
    setUser(u => ({ ...u, ...updates }));
  }, []);

  // Memoize context value
  const value = useMemo(() => ({
    count,
    user,
    expensiveValue,
    increment,
    updateUser,
  }), [count, user, expensiveValue, increment, updateUser]);

  return (
    <OptimizedContext.Provider value={value}>
      {children}
    </OptimizedContext.Provider>
  );
};

export const useOptimized = () => {
  const context = useContext(OptimizedContext);
  if (!context) {
    throw new Error('useOptimized must be used within an OptimizedProvider');
  }
  return context;
};
```

### **Context Splitting**
```javascript
// contexts/SplitContexts.js
import React, { createContext, useContext, useState } from 'react';

// Split contexts to avoid unnecessary re-renders
const UserContext = createContext();
const SettingsContext = createContext();
const NotificationsContext = createContext();

export const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [profile, setProfile] = useState({});

  return (
    <UserContext.Provider value={{ user, setUser, profile, setProfile }}>
      {children}
    </UserContext.Provider>
  );
};

export const SettingsProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  const [language, setLanguage] = useState('en');

  return (
    <SettingsContext.Provider value={{ theme, setTheme, language, setLanguage }}>
      {children}
    </SettingsContext.Provider>
  );
};

export const NotificationsProvider = ({ children }) => {
  const [notifications, setNotifications] = useState([]);
  const [unreadCount, setUnreadCount] = useState(0);

  return (
    <NotificationsContext.Provider value={{
      notifications,
      setNotifications,
      unreadCount,
      setUnreadCount
    }}>
      {children}
    </NotificationsContext.Provider>
  );
};

// Combined provider
export const AppContextProvider = ({ children }) => {
  return (
    <UserProvider>
      <SettingsProvider>
        <NotificationsProvider>
          {children}
        </NotificationsProvider>
      </SettingsProvider>
    </UserProvider>
  );
};

// Custom hooks
export const useUser = () => useContext(UserContext);
export const useSettings = () => useContext(SettingsContext);
export const useNotifications = () => useContext(NotificationsContext);
```

---

## 🚨 **Error Boundaries with Context**

### **Error Boundary Context**
```javascript
// contexts/ErrorBoundaryContext.js
import React, { createContext, useContext, useState, useCallback } from 'react';

const ErrorBoundaryContext = createContext();

export const ErrorBoundaryProvider = ({ children }) => {
  const [errors, setErrors] = useState([]);
  const [hasError, setHasError] = useState(false);

  const addError = useCallback((error, errorInfo) => {
    const errorEntry = {
      id: Date.now(),
      error: error.toString(),
      stack: error.stack,
      componentStack: errorInfo?.componentStack,
      timestamp: new Date().toISOString(),
    };

    setErrors(prev => [errorEntry, ...prev].slice(0, 10)); // Keep last 10 errors
    setHasError(true);

    // Log to external service
    console.error('Error caught by boundary:', errorEntry);
  }, []);

  const clearErrors = useCallback(() => {
    setErrors([]);
    setHasError(false);
  }, []);

  const resetError = useCallback(() => {
    setHasError(false);
  }, []);

  const value = {
    errors,
    hasError,
    addError,
    clearErrors,
    resetError,
  };

  return (
    <ErrorBoundaryContext.Provider value={value}>
      {children}
    </ErrorBoundaryContext.Provider>
  );
};

export const useErrorBoundary = () => {
  const context = useContext(ErrorBoundaryContext);
  if (!context) {
    throw new Error('useErrorBoundary must be used within an ErrorBoundaryProvider');
  }
  return context;
};
```

### **Error Boundary Component**
```javascript
// components/ErrorBoundary.js
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet, ScrollView } from 'react-native';
import { useErrorBoundary } from '../contexts/ErrorBoundaryContext';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({
      error,
      errorInfo,
    });

    // Report error to context
    if (this.props.onError) {
      this.props.onError(error, errorInfo);
    }
  }

  handleReset = () => {
    this.setState({ hasError: false, error: null, errorInfo: null });
    if (this.props.onReset) {
      this.props.onReset();
    }
  };

  render() {
    if (this.state.hasError) {
      return (
        <View style={styles.container}>
          <Text style={styles.title}>Oops! Something went wrong</Text>
          <Text style={styles.message}>
            We're sorry, but something unexpected happened.
          </Text>

          {__DEV__ && (
            <ScrollView style={styles.errorDetails}>
              <Text style={styles.errorText}>
                {this.state.error && this.state.error.toString()}
              </Text>
              <Text style={styles.stackTrace}>
                {this.state.errorInfo?.componentStack}
              </Text>
            </ScrollView>
          )}

          <View style={styles.actions}>
            <TouchableOpacity style={styles.retryButton} onPress={this.handleReset}>
              <Text style={styles.retryButtonText}>Try Again</Text>
            </TouchableOpacity>

            <TouchableOpacity style={styles.reportButton} onPress={this.props.onReport}>
              <Text style={styles.reportButtonText}>Report Issue</Text>
            </TouchableOpacity>
          </View>
        </View>
      );
    }

    return this.props.children;
  }
}

// Wrapped Error Boundary with Context
export const ContextErrorBoundary = ({ children }) => {
  const { addError, resetError } = useErrorBoundary();

  return (
    <ErrorBoundary
      onError={addError}
      onReset={resetError}
      onReport={() => {
        // Handle error reporting
        console.log('Reporting error...');
      }}
    >
      {children}
    </ErrorBoundary>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    justifyContent: 'center',
    alignItems: 'center',
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
    marginBottom: 20,
    color: '#666',
  },
  errorDetails: {
    maxHeight: 200,
    backgroundColor: '#fff',
    padding: 10,
    borderRadius: 8,
    marginBottom: 20,
  },
  errorText: {
    fontSize: 14,
    color: '#d32f2f',
    fontFamily: 'monospace',
  },
  stackTrace: {
    fontSize: 12,
    color: '#666',
    fontFamily: 'monospace',
    marginTop: 10,
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

export default ErrorBoundary;
```

---

## 🧪 **Testing Context**

### **Testing Context Providers**
```javascript
// __tests__/contexts/AuthContext.test.js
import React from 'react';
import { render, waitFor } from '@testing-library/react-native';
import { AuthProvider, useAuth } from '../../contexts/AuthContext';

// Mock AsyncStorage
jest.mock('@react-native-async-storage/async-storage', () => ({
  getItem: jest.fn(),
  setItem: jest.fn(),
  removeItem: jest.fn(),
}));

const AsyncStorage = require('@react-native-async-storage/async-storage');

// Test component that uses auth context
const TestComponent = () => {
  const { user, login, logout, isAuthenticated } = useAuth();

  return (
    <View>
      <Text testID="auth-status">
        {isAuthenticated ? 'Authenticated' : 'Not Authenticated'}
      </Text>
      <Text testID="user-name">{user?.name || 'No user'}</Text>
      <TouchableOpacity testID="login-btn" onPress={() => login('test@test.com', 'password')} />
      <TouchableOpacity testID="logout-btn" onPress={logout} />
    </View>
  );
};

describe('AuthContext', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  test('should provide initial auth state', () => {
    const { getByTestId } = render(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );

    expect(getByTestId('auth-status')).toHaveTextContent('Not Authenticated');
    expect(getByTestId('user-name')).toHaveTextContent('No user');
  });

  test('should handle successful login', async () => {
    // Mock successful API response
    global.fetch = jest.fn(() =>
      Promise.resolve({
        ok: true,
        json: () => Promise.resolve({
          id: 1,
          name: 'Test User',
          email: 'test@test.com',
        }),
      })
    );

    const { getByTestId } = render(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );

    // Trigger login
    fireEvent.press(getByTestId('login-btn'));

    // Wait for login to complete
    await waitFor(() => {
      expect(getByTestId('auth-status')).toHaveTextContent('Authenticated');
      expect(getByTestId('user-name')).toHaveTextContent('Test User');
    });

    // Verify AsyncStorage was called
    expect(AsyncStorage.setItem).toHaveBeenCalledWith(
      'user',
      JSON.stringify({
        id: 1,
        name: 'Test User',
        email: 'test@test.com',
      })
    );
  });

  test('should handle login error', async () => {
    // Mock failed API response
    global.fetch = jest.fn(() =>
      Promise.resolve({
        ok: false,
        json: () => Promise.resolve({ message: 'Invalid credentials' }),
      })
    );

    const { getByTestId } = render(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );

    // Trigger login
    fireEvent.press(getByTestId('login-btn'));

    // Wait for error
    await waitFor(() => {
      expect(getByTestId('auth-status')).toHaveTextContent('Not Authenticated');
    });

    // Verify AsyncStorage was not called
    expect(AsyncStorage.setItem).not.toHaveBeenCalled();
  });

  test('should handle logout', () => {
    // Set up initial authenticated state
    AsyncStorage.getItem.mockResolvedValue(JSON.stringify({
      id: 1,
      name: 'Test User',
      email: 'test@test.com',
    }));

    const { getByTestId } = render(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );

    // Wait for user to be loaded
    waitFor(() => {
      expect(getByTestId('auth-status')).toHaveTextContent('Authenticated');
    });

    // Trigger logout
    fireEvent.press(getByTestId('logout-btn'));

    // Verify logout
    expect(getByTestId('auth-status')).toHaveTextContent('Not Authenticated');
    expect(AsyncStorage.removeItem).toHaveBeenCalledWith('user');
  });
});
```

---

## 🎯 **Practical Examples**

### **E-commerce App with Context**
```javascript
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { AppProviders } from './contexts/AppProviders';
import AppNavigator from './navigation/AppNavigator';

const App = () => {
  return (
    <AppProviders>
      <NavigationContainer>
        <AppNavigator />
      </NavigationContainer>
    </AppProviders>
  );
};

export default App;

// components/ProductList.js
import React from 'react';
import { View, Text, FlatList, TouchableOpacity, StyleSheet } from 'react-native';
import { useShoppingCart } from '../contexts/ShoppingCartContext';
import { useTheme } from '../contexts/ThemeContext';

const ProductList = () => {
  const { addToCart } = useShoppingCart();
  const { theme } = useTheme();

  const products = [
    { id: 1, name: 'iPhone 15', price: 999, category: 'Electronics' },
    { id: 2, name: 'MacBook Pro', price: 1999, category: 'Electronics' },
    { id: 3, name: 'Nike Air Max', price: 129, category: 'Footwear' },
    { id: 4, name: 'Coffee Maker', price: 89, category: 'Appliances' },
  ];

  const renderProduct = ({ item }) => (
    <View style={[styles.productCard, { backgroundColor: theme.colors.surface }]} key={item.id}>
      <Text style={[styles.productName, { color: theme.colors.text }]}>{item.name}</Text>
      <Text style={[styles.productPrice, { color: theme.colors.primary }]}>${item.price}</Text>
      <Text style={[styles.productCategory, { color: theme.colors.textSecondary }]}>
        {item.category}
      </Text>
      <TouchableOpacity
        style={[styles.addButton, { backgroundColor: theme.colors.primary }]}
        onPress={() => addToCart(item)}
      >
        <Text style={[styles.addButtonText, { color: theme.colors.background }]}>Add to Cart</Text>
      </TouchableOpacity>
    </View>
  );

  return (
    <View style={[styles.container, { backgroundColor: theme.colors.background }]}>
      <Text style={[styles.title, { color: theme.colors.text }]}>Products</Text>
      <FlatList
        data={products}
        renderItem={renderProduct}
        keyExtractor={(item) => item.id.toString()}
        numColumns={2}
        contentContainerStyle={styles.productGrid}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  productGrid: {
    paddingBottom: 20,
  },
  productCard: {
    flex: 1,
    margin: 10,
    padding: 15,
    borderRadius: 12,
    alignItems: 'center',
  },
  productName: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 8,
    textAlign: 'center',
  },
  productPrice: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 4,
  },
  productCategory: {
    fontSize: 12,
    marginBottom: 12,
    textTransform: 'uppercase',
  },
  addButton: {
    padding: 10,
    borderRadius: 8,
    width: '100%',
    alignItems: 'center',
  },
  addButtonText: {
    fontSize: 14,
    fontWeight: 'bold',
  },
});

export default ProductList;

// components/CartSummary.js
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet, Alert } from 'react-native';
import { useShoppingCart } from '../contexts/ShoppingCartContext';
import { useTheme } from '../contexts/ThemeContext';

const CartSummary = () => {
  const { items, total, itemCount, clearCart } = useShoppingCart();
  const { theme } = useTheme();

  const handleClearCart = () => {
    Alert.alert(
      'Clear Cart',
      'Are you sure you want to clear all items from your cart?',
      [
        { text: 'Cancel', style: 'cancel' },
        { text: 'Clear', onPress: clearCart, style: 'destructive' },
      ]
    );
  };

  if (itemCount === 0) {
    return (
      <View style={[styles.emptyContainer, { backgroundColor: theme.colors.surface }]}>
        <Text style={[styles.emptyText, { color: theme.colors.textSecondary }]}>
          Your cart is empty
        </Text>
      </View>
    );
  }

  return (
    <View style={[styles.container, { backgroundColor: theme.colors.surface }]}>
      <Text style={[styles.title, { color: theme.colors.text }]}>Cart Summary</Text>

      <View style={styles.summary}>
        <Text style={[styles.itemCount, { color: theme.colors.textSecondary }]}>
          {itemCount} item{itemCount !== 1 ? 's' : ''}
        </Text>
        <Text style={[styles.total, { color: theme.colors.primary }]}>${total.toFixed(2)}</Text>
      </View>

      <View style={styles.items}>
        {items.map((item) => (
          <View key={item.id} style={styles.item}>
            <Text style={[styles.itemName, { color: theme.colors.text }]}>{item.name}</Text>
            <Text style={[styles.itemDetails, { color: theme.colors.textSecondary }]}>
              Qty: {item.quantity} × ${item.price}
            </Text>
            <Text style={[styles.itemSubtotal, { color: theme.colors.primary }]}>
              ${(item.price * item.quantity).toFixed(2)}
            </Text>
          </View>
        ))}
      </View>

      <View style={styles.actions}>
        <TouchableOpacity
          style={[styles.clearButton, { borderColor: theme.colors.error }]}
          onPress={handleClearCart}
        >
          <Text style={[styles.clearButtonText, { color: theme.colors.error }]}>Clear Cart</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={[styles.checkoutButton, { backgroundColor: theme.colors.primary }]}
          onPress={() => Alert.alert('Checkout', 'Checkout functionality would be implemented here')}
        >
          <Text style={[styles.checkoutButtonText, { color: theme.colors.background }]}>Checkout</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    margin: 20,
    padding: 20,
    borderRadius: 12,
  },
  emptyContainer: {
    margin: 20,
    padding: 40,
    borderRadius: 12,
    alignItems: 'center',
  },
  emptyText: {
    fontSize: 16,
    textAlign: 'center',
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 15,
  },
  summary: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 15,
  },
  itemCount: {
    fontSize: 14,
  },
  total: {
    fontSize: 18,
    fontWeight: 'bold',
  },
  items: {
    marginBottom: 20,
  },
  item: {
    marginBottom: 10,
    paddingBottom: 10,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  itemName: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 4,
  },
  itemDetails: {
    fontSize: 14,
    marginBottom: 2,
  },
  itemSubtotal: {
    fontSize: 14,
    fontWeight: 'bold',
  },
  actions: {
    flexDirection: 'row',
    gap: 10,
  },
  clearButton: {
    flex: 1,
    padding: 12,
    borderRadius: 8,
    borderWidth: 1,
    alignItems: 'center',
  },
  clearButtonText: {
    fontSize: 14,
    fontWeight: 'bold',
  },
  checkoutButton: {
    flex: 2,
    padding: 12,
    borderRadius: 8,
    alignItems: 'center',
  },
  checkoutButtonText: {
    fontSize: 14,
    fontWeight: 'bold',
  },
});

export default CartSummary;

// components/TodoApp.js
import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, FlatList, StyleSheet } from 'react-native';
import { useTodos } from '../contexts/TodoContext';
import { useTheme } from '../contexts/ThemeContext';

const TodoApp = () => {
  const { todos, addTodo, toggleTodo, deleteTodo, setFilter, filter, stats } = useTodos();
  const { theme } = useTheme();
  const [newTodo, setNewTodo] = useState('');

  const handleAddTodo = () => {
    if (newTodo.trim()) {
      addTodo(newTodo.trim());
      setNewTodo('');
    }
  };

  const renderTodo = ({ item }) => (
    <View style={[styles.todoItem, { backgroundColor: theme.colors.surface }]}>
      <TouchableOpacity
        style={styles.todoContent}
        onPress={() => toggleTodo(item.id)}
      >
        <Text
          style={[
            styles.todoText,
            { color: theme.colors.text },
            item.completed && styles.completedText,
          ]}
        >
          {item.text}
        </Text>
        <Text style={[styles.todoDate, { color: theme.colors.textSecondary }]}>
          {new Date(item.createdAt).toLocaleDateString()}
        </Text>
      </TouchableOpacity>

      <TouchableOpacity
        style={[styles.deleteButton, { backgroundColor: theme.colors.error }]}
        onPress={() => deleteTodo(item.id)}
      >
        <Text style={[styles.deleteButtonText, { color: theme.colors.background }]}>×</Text>
      </TouchableOpacity>
    </View>
  );

  const filterButtons = [
    { key: 'all', label: 'All' },
    { key: 'active', label: 'Active' },
    { key: 'completed', label: 'Completed' },
  ];

  return (
    <View style={[styles.container, { backgroundColor: theme.colors.background }]}>
      <Text style={[styles.title, { color: theme.colors.text }]}>Todo App</Text>

      {/* Add Todo Input */}
      <View style={styles.inputContainer}>
        <TextInput
          style={[styles.input, { backgroundColor: theme.colors.surface, color: theme.colors.text }]}
          placeholder="Add a new todo..."
          placeholderTextColor={theme.colors.textSecondary}
          value={newTodo}
          onChangeText={setNewTodo}
          onSubmitEditing={handleAddTodo}
        />
        <TouchableOpacity
          style={[styles.addButton, { backgroundColor: theme.colors.primary }]}
          onPress={handleAddTodo}
        >
          <Text style={[styles.addButtonText, { color: theme.colors.background }]}>Add</Text>
        </TouchableOpacity>
      </View>

      {/* Filter Buttons */}
      <View style={styles.filterContainer}>
        {filterButtons.map((button) => (
          <TouchableOpacity
            key={button.key}
            style={[
              styles.filterButton,
              filter === button.key && { backgroundColor: theme.colors.primary },
            ]}
            onPress={() => setFilter(button.key)}
          >
            <Text
              style={[
                styles.filterButtonText,
                { color: filter === button.key ? theme.colors.background : theme.colors.text },
              ]}
            >
              {button.label}
            </Text>
          </TouchableOpacity>
        ))}
      </View>

      {/* Stats */}
      <View style={styles.statsContainer}>
        <Text style={[styles.statsText, { color: theme.colors.textSecondary }]}>
          Total: {stats.total} | Active: {stats.active} | Completed: {stats.completed}
        </Text>
      </View>

      {/* Todo List */}
      <FlatList
        data={todos}
        renderItem={renderTodo}
        keyExtractor={(item) => item.id}
        style={styles.todoList}
        showsVerticalScrollIndicator={false}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  inputContainer: {
    flexDirection: 'row',
    marginBottom: 20,
    gap: 10,
  },
  input: {
    flex: 1,
    padding: 12,
    borderRadius: 8,
    fontSize: 16,
  },
  addButton: {
    padding: 12,
    borderRadius: 8,
    justifyContent: 'center',
    minWidth: 60,
  },
  addButtonText: {
    fontSize: 16,
    fontWeight: 'bold',
    textAlign: 'center',
  },
  filterContainer: {
    flexDirection: 'row',
    marginBottom: 15,
    gap: 10,
  },
  filterButton: {
    flex: 1,
    padding: 10,
    borderRadius: 8,
    alignItems: 'center',
  },
  filterButtonText: {
    fontSize: 14,
    fontWeight: 'bold',
  },
  statsContainer: {
    marginBottom: 15,
    alignItems: 'center',
  },
  statsText: {
    fontSize: 14,
  },
  todoList: {
    flex: 1,
  },
  todoItem: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 15,
    marginBottom: 10,
    borderRadius: 8,
  },
  todoContent: {
    flex: 1,
  },
  todoText: {
    fontSize: 16,
    marginBottom: 4,
  },
  completedText: {
    textDecorationLine: 'line-through',
    opacity: 0.6,
  },
  todoDate: {
    fontSize: 12,
  },
  deleteButton: {
    width: 30,
    height: 30,
    borderRadius: 15,
    alignItems: 'center',
    justifyContent: 'center',
    marginLeft: 10,
  },
  deleteButtonText: {
    fontSize: 18,
    fontWeight: 'bold',
  },
});

export default TodoApp;

---

## 📝 **Lesson Summary**

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Context API Basics**: Creating and using React Context for state management
- ✅ **Context Providers**: Building provider components with state and actions
- ✅ **useContext Hook**: Consuming context values in functional components
- ✅ **useReducer with Context**: Complex state management with reducers
- ✅ **Custom Hooks**: Creating reusable hooks for context operations
- ✅ **Context Composition**: Combining multiple contexts efficiently
- ✅ **Performance Optimization**: Memoization and context splitting techniques
- ✅ **Error Boundaries**: Handling errors with context integration
- ✅ **Testing Context**: Unit and integration testing for context providers
- ✅ **Practical Implementation**: Real-world e-commerce app with multiple contexts

### **Best Practices**
1. **Keep contexts focused** - Each context should manage one concern
2. **Use memoization** - Memoize context values to prevent unnecessary re-renders
3. **Split large contexts** - Break down complex contexts into smaller ones
4. **Provide custom hooks** - Create useContext hooks for better error handling
5. **Handle loading states** - Include loading and error states in context
6. **Use TypeScript** - Add type safety to context values and actions
7. **Test thoroughly** - Test context providers and consumers separately
8. **Document context APIs** - Clearly document what each context provides
9. **Avoid deep nesting** - Use composition patterns to avoid prop drilling
10. **Optimize re-renders** - Use selectors and memoization for performance

### **Next Steps**
- Learn Redux Toolkit for more complex state management
- Explore Zustand as an alternative to Context API
- Implement context with TypeScript for better type safety
- Learn about state machines and XState
- Study advanced patterns like render props and compound components
- Explore server state management with React Query/SWR

---

## 🎯 **Assignment**

### **Task 1: Social Media App Context**
Create a comprehensive context system for a social media application with:
- User authentication and profile management
- Posts, comments, and likes functionality
- Real-time notifications system
- Theme switching (light/dark mode)
- Network status monitoring
- Error handling and offline support

### **Task 2: E-commerce Shopping Cart**
Build a shopping cart context with:
- Product catalog management
- Shopping cart operations (add, remove, update quantity)
- Wishlist functionality
- Order processing and checkout flow
- Payment integration context
- Order history and tracking
- Inventory management

### **Task 3: Task Management App**
Implement a task management system using Context API with:
- Project and task CRUD operations
- User assignment and permissions
- Time tracking and reporting
- File attachments and comments
- Real-time collaboration features
- Data export and import functionality

### **Task 4: Advanced Context Patterns**
Create advanced context patterns including:
- Context composition with higher-order components
- Render props pattern with context
- Compound component pattern
- Context with error boundaries
- Performance monitoring for context operations
- Automated testing for complex context interactions

---

## 📚 **Additional Resources**
- [React Context Documentation](https://reactjs.org/docs/context.html)
- [useContext Hook Documentation](https://reactjs.org/docs/hooks-reference.html#usecontext)
- [useReducer Hook Documentation](https://reactjs.org/docs/hooks-reference.html#usereducer)
- [React Performance Optimization](https://reactjs.org/docs/optimizing-performance.html)
- [Context vs Redux Comparison](https://blog.logrocket.com/context-api-vs-redux/)
- [Advanced React Patterns](https://kentcdodds.com/blog/application-state-management)

---

**Next Lesson**: [Lesson 35: React Query & Data Fetching](Lesson%2035_%20React%20Query%20&%20Data%20Fetching.md)