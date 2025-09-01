# Lesson 31: Redux Fundamentals & State Management

## 🎯 **Learning Objectives**
- Understand Redux architecture and principles
- Implement Redux store, actions, and reducers
- Create Redux middleware for async operations
- Integrate Redux with React Native applications
- Implement Redux DevTools for debugging

## 📚 **Table of Contents**
1. [Introduction to Redux](#introduction-to-redux)
2. [Redux Core Concepts](#redux-core-concepts)
3. [Setting Up Redux](#setting-up-redux)
4. [Actions and Action Creators](#actions-and-action-creators)
5. [Reducers](#reducers)
6. [Store Configuration](#store-configuration)
7. [Connecting Redux to React](#connecting-redux-to-react)
8. [Async Actions with Redux Thunk](#async-actions-with-redux-thunk)
9. [Redux DevTools](#redux-devtools)
10. [Practical Examples](#practical-examples)

---

## 🧠 **Introduction to Redux**

### **What is Redux?**
Redux is a predictable state container for JavaScript applications. It helps manage application state in a predictable way using actions and reducers.

### **Why Redux?**
- **Predictable State**: State changes are predictable and traceable
- **Centralized State**: Single source of truth for application state
- **Debugging**: Time-travel debugging with Redux DevTools
- **Testing**: Pure functions make testing easier
- **Performance**: Optimized re-rendering with selectors

### **Redux Principles**
1. **Single Source of Truth**: One store for entire application state
2. **State is Read-Only**: State can only be changed by dispatching actions
3. **Changes with Pure Functions**: Reducers are pure functions

---

## 🏗️ **Redux Core Concepts**

### **Store**
The store holds the application state and provides methods to:
- `getState()`: Get current state
- `dispatch(action)`: Dispatch actions to change state
- `subscribe(listener)`: Listen for state changes

### **Actions**
Actions are plain JavaScript objects that describe what happened:
```javascript
{
  type: 'ADD_TODO',
  payload: {
    id: 1,
    text: 'Learn Redux',
    completed: false
  }
}
```

### **Reducers**
Pure functions that take current state and action, return new state:
```javascript
function todosReducer(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.payload];
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.payload.id
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    default:
      return state;
  }
}
```

---

## ⚙️ **Setting Up Redux**

### **Installation**
```bash
npm install @reduxjs/toolkit react-redux
# OR for traditional Redux
npm install redux react-redux redux-thunk
```

### **Project Structure**
```
src/
├── store/
│   ├── index.js
│   ├── slices/
│   │   ├── authSlice.js
│   │   ├── todosSlice.js
│   │   └── userSlice.js
│   └── middleware/
│       └── logger.js
├── components/
├── screens/
└── App.js
```

---

## 🎬 **Actions and Action Creators**

### **Action Types**
```javascript
// Action Types (constants)
export const USER_LOGIN_REQUEST = 'USER_LOGIN_REQUEST';
export const USER_LOGIN_SUCCESS = 'USER_LOGIN_SUCCESS';
export const USER_LOGIN_FAILURE = 'USER_LOGIN_FAILURE';
export const USER_LOGOUT = 'USER_LOGOUT';

// Action Creators
export const loginRequest = () => ({
  type: USER_LOGIN_REQUEST,
});

export const loginSuccess = (user) => ({
  type: USER_LOGIN_SUCCESS,
  payload: user,
});

export const loginFailure = (error) => ({
  type: USER_LOGIN_FAILURE,
  payload: error,
});

export const logout = () => ({
  type: USER_LOGOUT,
});
```

### **Async Action Creators (with Thunk)**
```javascript
// Async Action Creator
export const loginUser = (credentials) => {
  return async (dispatch) => {
    try {
      dispatch(loginRequest());

      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(credentials),
      });

      if (!response.ok) {
        throw new Error('Login failed');
      }

      const user = await response.json();
      dispatch(loginSuccess(user));
    } catch (error) {
      dispatch(loginFailure(error.message));
    }
  };
};
```

---

## 🔄 **Reducers**

### **Basic Reducer**
```javascript
const initialState = {
  user: null,
  isLoading: false,
  error: null,
};

const authReducer = (state = initialState, action) => {
  switch (action.type) {
    case USER_LOGIN_REQUEST:
      return {
        ...state,
        isLoading: true,
        error: null,
      };

    case USER_LOGIN_SUCCESS:
      return {
        ...state,
        isLoading: false,
        user: action.payload,
        error: null,
      };

    case USER_LOGIN_FAILURE:
      return {
        ...state,
        isLoading: false,
        user: null,
        error: action.payload,
      };

    case USER_LOGOUT:
      return {
        ...state,
        user: null,
        error: null,
      };

    default:
      return state;
  }
};

export default authReducer;
```

### **Combining Reducers**
```javascript
import { combineReducers } from 'redux';
import authReducer from './authReducer';
import todosReducer from './todosReducer';
import settingsReducer from './settingsReducer';

const rootReducer = combineReducers({
  auth: authReducer,
  todos: todosReducer,
  settings: settingsReducer,
});

export default rootReducer;
```

---

## 🏪 **Store Configuration**

### **Creating the Store**
```javascript
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import { composeWithDevTools } from 'redux-devtools-extension';
import rootReducer from './reducers';

const middleware = [thunk];

// Add logger middleware in development
if (process.env.NODE_ENV === 'development') {
  const { createLogger } = require('redux-logger');
  const logger = createLogger({
    collapsed: true,
    diff: true,
  });
  middleware.push(logger);
}

const store = createStore(
  rootReducer,
  composeWithDevTools(applyMiddleware(...middleware))
);

export default store;
```

### **Store with Redux Toolkit**
```javascript
import { configureStore } from '@reduxjs/toolkit';
import authSlice from './slices/authSlice';
import todosSlice from './slices/todosSlice';
import settingsSlice from './slices/settingsSlice';

const store = configureStore({
  reducer: {
    auth: authSlice,
    todos: todosSlice,
    settings: settingsSlice,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST'],
      },
    }),
  devTools: process.env.NODE_ENV !== 'production',
});

export default store;
```

---

## 🔗 **Connecting Redux to React**

### **Provider Setup**
```javascript
// App.js
import React from 'react';
import { Provider } from 'react-redux';
import { PersistGate } from 'redux-persist/integration/react';
import store, { persistor } from './store';
import AppNavigator from './navigation/AppNavigator';

const App = () => {
  return (
    <Provider store={store}>
      <PersistGate loading={null} persistor={persistor}>
        <AppNavigator />
      </PersistGate>
    </Provider>
  );
};

export default App;
```

### **Using useSelector and useDispatch**
```javascript
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { loginUser, logout } from '../store/actions/authActions';

const LoginScreen = () => {
  const dispatch = useDispatch();
  const { user, isLoading, error } = useSelector(state => state.auth);

  const handleLogin = () => {
    const credentials = {
      email: 'user@example.com',
      password: 'password123',
    };
    dispatch(loginUser(credentials));
  };

  const handleLogout = () => {
    dispatch(logout());
  };

  if (user) {
    return (
      <View style={styles.container}>
        <Text style={styles.welcome}>Welcome, {user.name}!</Text>
        <TouchableOpacity style={styles.button} onPress={handleLogout}>
          <Text style={styles.buttonText}>Logout</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Login</Text>

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
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  welcome: {
    fontSize: 20,
    marginBottom: 20,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 10,
    minWidth: 150,
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
  error: {
    color: 'red',
    marginBottom: 15,
    textAlign: 'center',
  },
});

export default LoginScreen;
```

### **Custom Hooks for Redux**
```javascript
// hooks/useAuth.js
import { useSelector, useDispatch } from 'react-redux';
import { loginUser, logout, refreshToken } from '../store/actions/authActions';

export const useAuth = () => {
  const dispatch = useDispatch();
  const { user, isLoading, error } = useSelector(state => state.auth);

  const login = (credentials) => {
    dispatch(loginUser(credentials));
  };

  const logoutUser = () => {
    dispatch(logout());
  };

  const refreshUserToken = () => {
    dispatch(refreshToken());
  };

  return {
    user,
    isLoading,
    error,
    isAuthenticated: !!user,
    login,
    logout: logoutUser,
    refreshToken: refreshUserToken,
  };
};

// hooks/useTodos.js
import { useSelector, useDispatch } from 'react-redux';
import { addTodo, toggleTodo, deleteTodo, fetchTodos } from '../store/actions/todoActions';

export const useTodos = () => {
  const dispatch = useDispatch();
  const { todos, isLoading, error } = useSelector(state => state.todos);

  const createTodo = (text) => {
    dispatch(addTodo(text));
  };

  const toggleTodoStatus = (id) => {
    dispatch(toggleTodo(id));
  };

  const removeTodo = (id) => {
    dispatch(deleteTodo(id));
  };

  const loadTodos = () => {
    dispatch(fetchTodos());
  };

  return {
    todos,
    isLoading,
    error,
    createTodo,
    toggleTodo: toggleTodoStatus,
    deleteTodo: removeTodo,
    fetchTodos: loadTodos,
  };
};
```

---

## ⚡ **Async Actions with Redux Thunk**

### **Thunk Middleware Setup**
```javascript
// store/index.js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import { composeWithDevTools } from 'redux-devtools-extension';
import rootReducer from './reducers';

const middleware = [thunk];

if (process.env.NODE_ENV === 'development') {
  const { createLogger } = require('redux-logger');
  middleware.push(createLogger());
}

const store = createStore(
  rootReducer,
  composeWithDevTools(applyMiddleware(...middleware))
);

export default store;
```

### **Async Action Creators**
```javascript
// actions/userActions.js
export const FETCH_USERS_REQUEST = 'FETCH_USERS_REQUEST';
export const FETCH_USERS_SUCCESS = 'FETCH_USERS_SUCCESS';
export const FETCH_USERS_FAILURE = 'FETCH_USERS_FAILURE';

export const fetchUsersRequest = () => ({
  type: FETCH_USERS_REQUEST,
});

export const fetchUsersSuccess = (users) => ({
  type: FETCH_USERS_SUCCESS,
  payload: users,
});

export const fetchUsersFailure = (error) => ({
  type: FETCH_USERS_FAILURE,
  payload: error,
});

// Async thunk action
export const fetchUsers = () => {
  return async (dispatch, getState) => {
    try {
      dispatch(fetchUsersRequest());

      const response = await fetch('/api/users');
      if (!response.ok) {
        throw new Error('Failed to fetch users');
      }

      const users = await response.json();
      dispatch(fetchUsersSuccess(users));
    } catch (error) {
      dispatch(fetchUsersFailure(error.message));
    }
  };
};

// Conditional async action
export const fetchUsersIfNeeded = () => {
  return async (dispatch, getState) => {
    const { users } = getState();

    // Don't fetch if already loading or have data
    if (users.isLoading || users.data.length > 0) {
      return;
    }

    dispatch(fetchUsers());
  };
};
```

### **Complex Async Actions**
```javascript
// actions/orderActions.js
export const CREATE_ORDER_REQUEST = 'CREATE_ORDER_REQUEST';
export const CREATE_ORDER_SUCCESS = 'CREATE_ORDER_SUCCESS';
export const CREATE_ORDER_FAILURE = 'CREATE_ORDER_FAILURE';

export const createOrder = (orderData) => {
  return async (dispatch, getState) => {
    try {
      dispatch({ type: CREATE_ORDER_REQUEST });

      const { auth } = getState();
      if (!auth.user) {
        throw new Error('User not authenticated');
      }

      // Validate order data
      if (!orderData.items || orderData.items.length === 0) {
        throw new Error('Order must contain at least one item');
      }

      // Calculate total
      const total = orderData.items.reduce((sum, item) => sum + item.price * item.quantity, 0);

      // Create order payload
      const orderPayload = {
        ...orderData,
        total,
        userId: auth.user.id,
        status: 'pending',
      };

      const response = await fetch('/api/orders', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${auth.token}`,
        },
        body: JSON.stringify(orderPayload),
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.message || 'Failed to create order');
      }

      const order = await response.json();
      dispatch({
        type: CREATE_ORDER_SUCCESS,
        payload: order,
      });

      // Clear cart after successful order
      dispatch({ type: 'CLEAR_CART' });

      return order;
    } catch (error) {
      dispatch({
        type: CREATE_ORDER_FAILURE,
        payload: error.message,
      });
      throw error;
    }
  };
};
```

---

## 🛠️ **Redux DevTools**

### **DevTools Setup**
```javascript
// store/index.js
import { createStore, applyMiddleware } from 'redux';
import { composeWithDevTools } from 'redux-devtools-extension';
import thunk from 'redux-thunk';
import rootReducer from './reducers';

const composeEnhancers = composeWithDevTools({
  name: 'MyApp',
  realtime: true,
  trace: true,
  traceLimit: 25,
});

const store = createStore(
  rootReducer,
  composeEnhancers(applyMiddleware(thunk))
);

export default store;
```

### **Custom DevTools Configuration**
```javascript
const composeEnhancers = composeWithDevTools({
  name: 'MyApp Redux Store',
  realtime: true,
  trace: true,
  traceLimit: 25,
  actionCreators: {
    // Custom action creators for testing
    login: (credentials) => ({ type: 'USER_LOGIN_REQUEST', payload: credentials }),
    logout: () => ({ type: 'USER_LOGOUT' }),
  },
  stateSanitizer: (state) => {
    // Sanitize sensitive data
    return {
      ...state,
      auth: {
        ...state.auth,
        token: state.auth.token ? '***SANITIZED***' : null,
      },
    };
  },
  actionSanitizer: (action) => {
    // Sanitize sensitive actions
    if (action.type === 'USER_LOGIN_SUCCESS' && action.payload.token) {
      return {
        ...action,
        payload: {
          ...action.payload,
          token: '***SANITIZED***',
        },
      };
    }
    return action;
  },
});
```

---

## 🎯 **Practical Examples**

### **Todo App with Redux**
```javascript
// store/slices/todosSlice.js
import { createSlice } from '@reduxjs/toolkit';

const todosSlice = createSlice({
  name: 'todos',
  initialState: {
    items: [],
    loading: false,
    error: null,
    filter: 'all', // all, active, completed
  },
  reducers: {
    addTodo: (state, action) => {
      state.items.push({
        id: Date.now(),
        text: action.payload,
        completed: false,
        createdAt: new Date().toISOString(),
      });
    },
    toggleTodo: (state, action) => {
      const todo = state.items.find(item => item.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },
    deleteTodo: (state, action) => {
      state.items = state.items.filter(item => item.id !== action.payload);
    },
    setFilter: (state, action) => {
      state.filter = action.payload;
    },
    clearCompleted: (state) => {
      state.items = state.items.filter(item => !item.completed);
    },
    setLoading: (state, action) => {
      state.loading = action.payload;
    },
    setError: (state, action) => {
      state.error = action.payload;
      state.loading = false;
    },
  },
});

export const {
  addTodo,
  toggleTodo,
  deleteTodo,
  setFilter,
  clearCompleted,
  setLoading,
  setError,
} = todosSlice.actions;

export default todosSlice.reducer;
```

### **Todo List Component**
```javascript
import React, { useState } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  FlatList,
  StyleSheet,
} from 'react-native';
import {
  addTodo,
  toggleTodo,
  deleteTodo,
  setFilter,
  clearCompleted,
} from '../store/slices/todosSlice';

const TodoApp = () => {
  const dispatch = useDispatch();
  const { items, filter } = useSelector(state => state.todos);
  const [inputText, setInputText] = useState('');

  const filteredTodos = items.filter(todo => {
    switch (filter) {
      case 'active':
        return !todo.completed;
      case 'completed':
        return todo.completed;
      default:
        return true;
    }
  });

  const handleAddTodo = () => {
    if (inputText.trim()) {
      dispatch(addTodo(inputText.trim()));
      setInputText('');
    }
  };

  const handleToggleTodo = (id) => {
    dispatch(toggleTodo(id));
  };

  const handleDeleteTodo = (id) => {
    dispatch(deleteTodo(id));
  };

  const renderTodo = ({ item }) => (
    <View style={styles.todoItem}>
      <TouchableOpacity
        style={[styles.checkbox, item.completed && styles.checked]}
        onPress={() => handleToggleTodo(item.id)}
      >
        {item.completed && <Text style={styles.checkmark}>✓</Text>}
      </TouchableOpacity>

      <Text style={[styles.todoText, item.completed && styles.completedText]}>
        {item.text}
      </Text>

      <TouchableOpacity
        style={styles.deleteButton}
        onPress={() => handleDeleteTodo(item.id)}
      >
        <Text style={styles.deleteText}>×</Text>
      </TouchableOpacity>
    </View>
  );

  const activeCount = items.filter(todo => !todo.completed).length;
  const completedCount = items.filter(todo => todo.completed).length;

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Redux Todo App</Text>

      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Add a new todo..."
          value={inputText}
          onChangeText={setInputText}
          onSubmitEditing={handleAddTodo}
          returnKeyType="done"
        />
        <TouchableOpacity style={styles.addButton} onPress={handleAddTodo}>
          <Text style={styles.addButtonText}>Add</Text>
        </TouchableOpacity>
      </View>

      <View style={styles.filterContainer}>
        {['all', 'active', 'completed'].map(filterType => (
          <TouchableOpacity
            key={filterType}
            style={[styles.filterButton, filter === filterType && styles.activeFilter]}
            onPress={() => dispatch(setFilter(filterType))}
          >
            <Text style={[styles.filterText, filter === filterType && styles.activeFilterText]}>
              {filterType.charAt(0).toUpperCase() + filterType.slice(1)}
            </Text>
          </TouchableOpacity>
        ))}
      </View>

      <FlatList
        data={filteredTodos}
        renderItem={renderTodo}
        keyExtractor={(item) => item.id.toString()}
        style={styles.todoList}
        ListEmptyComponent={
          <Text style={styles.emptyText}>No todos found</Text>
        }
      />

      <View style={styles.footer}>
        <Text style={styles.stats}>
          {activeCount} active, {completedCount} completed
        </Text>
        {completedCount > 0 && (
          <TouchableOpacity
            style={styles.clearButton}
            onPress={() => dispatch(clearCompleted())}
          >
            <Text style={styles.clearButtonText}>Clear Completed</Text>
          </TouchableOpacity>
        )}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
  },
  inputContainer: {
    flexDirection: 'row',
    marginBottom: 20,
  },
  input: {
    flex: 1,
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    marginRight: 10,
    backgroundColor: 'white',
  },
  addButton: {
    backgroundColor: '#28a745',
    padding: 12,
    borderRadius: 8,
    justifyContent: 'center',
    minWidth: 60,
  },
  addButtonText: {
    color: 'white',
    fontWeight: 'bold',
    textAlign: 'center',
  },
  filterContainer: {
    flexDirection: 'row',
    marginBottom: 20,
  },
  filterButton: {
    flex: 1,
    padding: 10,
    alignItems: 'center',
    borderRadius: 8,
    marginHorizontal: 2,
  },
  activeFilter: {
    backgroundColor: '#007AFF',
  },
  filterText: {
    fontWeight: 'bold',
  },
  activeFilterText: {
    color: 'white',
  },
  todoList: {
    flex: 1,
  },
  todoItem: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 8,
    marginBottom: 8,
  },
  checkbox: {
    width: 24,
    height: 24,
    borderWidth: 2,
    borderColor: '#007AFF',
    borderRadius: 12,
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: 12,
  },
  checked: {
    backgroundColor: '#007AFF',
  },
  checkmark: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  todoText: {
    flex: 1,
    fontSize: 16,
  },
  completedText: {
    textDecorationLine: 'line-through',
    color: '#666',
  },
  deleteButton: {
    width: 30,
    height: 30,
    borderRadius: 15,
    backgroundColor: '#dc3545',
    justifyContent: 'center',
    alignItems: 'center',
  },
  deleteText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  emptyText: {
    textAlign: 'center',
    color: '#666',
    fontSize: 16,
    marginTop: 50,
  },
  footer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingTop: 20,
    borderTopWidth: 1,
    borderTopColor: '#ddd',
  },
  stats: {
    color: '#666',
    fontSize: 14,
  },
  clearButton: {
    backgroundColor: '#6c757d',
    padding: 8,
    borderRadius: 5,
  },
  clearButtonText: {
    color: 'white',
    fontSize: 12,
  },
});

export default TodoApp;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Redux Architecture**: Store, actions, reducers, middleware
- ✅ **Action Creators**: Synchronous and asynchronous actions
- ✅ **Reducers**: Pure functions for state updates
- ✅ **Store Configuration**: Middleware, enhancers, dev tools
- ✅ **React Integration**: Provider, useSelector, useDispatch
- ✅ **Redux Thunk**: Handling async operations
- ✅ **Redux DevTools**: Debugging and time-travel
- ✅ **Best Practices**: Code organization and patterns

### **Best Practices**
1. **Keep reducers pure**: No side effects, predictable results
2. **Normalize state shape**: Avoid deep nesting
3. **Use action creators**: Consistent action dispatching
4. **Handle async with thunks**: Clean async action handling
5. **Use selectors**: Computed values and memoization
6. **Test reducers**: Pure functions are easy to test
7. **Use Redux DevTools**: Better debugging experience

### **Next Steps**
- Learn Redux Toolkit for simplified Redux usage
- Implement Redux Saga for complex async flows
- Add Redux Persist for state persistence
- Create custom middleware for specific needs

---

## 🎯 **Assignment**

### **Task 1: Counter App with Redux**
Create a counter app with Redux that includes:
- Increment/decrement actions
- Reset functionality
- Step size configuration
- History of operations
- Undo/redo functionality

### **Task 2: Shopping Cart**
Build a shopping cart with Redux featuring:
- Add/remove items
- Update quantities
- Calculate totals
- Apply discounts
- Persist cart state

### **Task 3: Weather App**
Implement a weather app using Redux with:
- Fetch weather data from API
- Cache weather information
- Handle loading and error states
- Multiple location support
- Weather history

---

## 📚 **Additional Resources**
- [Redux Official Documentation](https://redux.js.org/)
- [Redux Toolkit Documentation](https://redux-toolkit.js.org/)
- [React Redux Documentation](https://react-redux.js.org/)
- [Redux DevTools](https://github.com/reduxjs/redux-devtools)
- [Redux Best Practices](https://redux.js.org/style-guide/style-guide)

---

**Next Lesson**: [Lesson 32: Advanced Redux Patterns & Middleware](Lesson%2032_%20Advanced%20Redux%20Patterns%20&%20Middleware.md)