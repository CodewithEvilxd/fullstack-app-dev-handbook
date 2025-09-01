
# 🪝 Lesson 11: Advanced Hooks (useContext, useReducer, Custom Hooks)

## 🎯 Learning Objectives
Is lesson ke baad aap:
- useContext hook se global state manage kar sakte hain
- useReducer hook se complex state logic handle kar sakte hain
- Custom hooks create kar sakte hain
- Hook patterns aur best practices samjhenge
- Advanced state management implement kar sakte hain

---

## 📖 useContext Hook

useContext hook React Context API ko functional components mein use karne ke liye hai. Yeh prop drilling problem solve karta hai.

### 1. Basic Context Setup

```javascript
import React, { createContext, useContext, useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

// Create Context
const ThemeContext = createContext();

// Theme Provider Component
const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };
  
  const themeValue = {
    theme,
    toggleTheme,
    colors: theme === 'light' ? lightColors : darkColors
  };
  
  return (
    <ThemeContext.Provider value={themeValue}>
      {children}
    </ThemeContext.Provider>
  );
};

// Custom hook to use theme
const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};

// Theme colors
const lightColors = {
  background: '#ffffff',
  text: '#000000',
  primary: '#007AFF',
  secondary: '#f8f9fa'
};

const darkColors = {
  background: '#000000',
  text: '#ffffff',
  primary: '#0A84FF',
  secondary: '#1c1c1e'
};

// Component using theme context
const ThemedButton = ({ title, onPress }) => {
  const { colors } = useTheme();
  
  return (
    <TouchableOpacity
      style={[styles.button, { backgroundColor: colors.primary }]}
      onPress={onPress}
    >
      <Text style={[styles.buttonText, { color: colors.background }]}>
        {title}
      </Text>
    </TouchableOpacity>
  );
};

const ThemedText = ({ children, style }) => {
  const { colors } = useTheme();
  
  return (
    <Text style={[{ color: colors.text }, style]}>
      {children}
    </Text>
  );
};

// Main component
const ContextExample = () => {
  const { theme, toggleTheme, colors } = useTheme();
  
  return (
    <View style={[styles.container, { backgroundColor: colors.background }]}>
      <ThemedText style={styles.title}>
        Current Theme: {theme}
      </ThemedText>
      
      <View style={[styles.card, { backgroundColor: colors.secondary }]}>
        <ThemedText style={styles.cardTitle}>
          Theme Context Example
        </ThemedText>
        <ThemedText style={styles.cardText}>
          This component automatically adapts to the current theme using useContext hook.
        </ThemedText>
      </View>
      
      <ThemedButton
        title={`Switch to ${theme === 'light' ? 'Dark' : 'Light'} Theme`}
        onPress={toggleTheme}
      />
    </View>
  );
};

// App wrapper with provider
const App = () => {
  return (
    <ThemeProvider>
      <ContextExample />
    </ThemeProvider>
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
  card: {
    padding: 20,
    borderRadius: 12,
    marginBottom: 30,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  cardTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  cardText: {
    fontSize: 16,
    lineHeight: 24,
  },
  button: {
    paddingVertical: 15,
    paddingHorizontal: 30,
    borderRadius: 8,
    alignItems: 'center',
  },
  buttonText: {
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default App;
```

### 2. Multiple Contexts

```javascript
import React, { createContext, useContext, useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet, Alert } from 'react-native';

// User Context
const UserContext = createContext();

const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  
  const login = async (username, password) => {
    setIsLoading(true);
    try {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 1000));
      setUser({
        id: 1,
        username,
        email: `${username}@example.com`,
        role: 'user'
      });
    } catch (error) {
      Alert.alert('Login Failed', error.message);
    } finally {
      setIsLoading(false);
    }
  };
  
  const logout = () => {
    setUser(null);
  };
  
  return (
    <UserContext.Provider value={{
      user,
      isLoading,
      login,
      logout,
      isAuthenticated: !!user
    }}>
      {children}
    </UserContext.Provider>
  );
};

// Settings Context
const SettingsContext = createContext();

const SettingsProvider = ({ children }) => {
  const [settings, setSettings] = useState({
    notifications: true,
    darkMode: false,
    language: 'en'
  });
  
  const updateSetting = (key, value) => {
    setSettings(prev => ({
      ...prev,
      [key]: value
    }));
  };
  
  return (
    <SettingsContext.Provider value={{
      settings,
      updateSetting
    }}>
      {children}
    </SettingsContext.Provider>
  );
};

// Custom hooks
const useUser = () => {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within UserProvider');
  }
  return context;
};

const useSettings = () => {
  const context = useContext(SettingsContext);
  if (!context) {
    throw new Error('useSettings must be used within SettingsProvider');
  }
  return context;
};

// Components using multiple contexts
const UserProfile = () => {
  const { user, logout } = useUser();
  const { settings, updateSetting } = useSettings();
  
  if (!user) return null;
  
  return (
    <View style={styles.profileContainer}>
      <Text style={styles.profileTitle}>User Profile</Text>
      <Text style={styles.profileText}>Username: {user.username}</Text>
      <Text style={styles.profileText}>Email: {user.email}</Text>
      <Text style={styles.profileText}>Role: {user.role}</Text>
      
      <View style={styles.settingsContainer}>
        <Text style={styles.settingsTitle}>Settings</Text>
        
        <View style={styles.settingItem}>
          <Text>Notifications: {settings.notifications ? 'On' : 'Off'}</Text>
          <TouchableOpacity
            style={styles.toggleButton}
            onPress={() => updateSetting('notifications', !settings.notifications)}
          >
            <Text style={styles.toggleButtonText}>Toggle</Text>
          </TouchableOpacity>
        </View>
        
        <View style={styles.settingItem}>
          <Text>Dark Mode: {settings.darkMode ? 'On' : 'Off'}</Text>
          <TouchableOpacity
            style={styles.toggleButton}
            onPress={() => updateSetting('darkMode', !settings.darkMode)}
          >
            <Text style={styles.toggleButtonText}>Toggle</Text>
          </TouchableOpacity>
        </View>
      </View>
      
      <TouchableOpacity style={styles.logoutButton} onPress={logout}>
        <Text style={styles.logoutButtonText}>Logout</Text>
      </TouchableOpacity>
    </View>
  );
};

const LoginForm = () => {
  const { login, isLoading } = useUser();
  
  const handleLogin = () => {
    login('john_doe', 'password123');
  };
  
  return (
    <View style={styles.loginContainer}>
      <Text style={styles.loginTitle}>Please Login</Text>
      <TouchableOpacity
        style={[styles.loginButton, isLoading && styles.disabledButton]}
        onPress={handleLogin}
        disabled={isLoading}
      >
        <Text style={styles.loginButtonText}>
          {isLoading ? 'Logging in...' : 'Login'}
        </Text>
      </TouchableOpacity>
    </View>
  );
};

const MultipleContextExample = () => {
  const { isAuthenticated } = useUser();
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Multiple Context Example</Text>
      {isAuthenticated ? <UserProfile /> : <LoginForm />}
    </View>
  );
};

// App with multiple providers
const App = () => {
  return (
    <UserProvider>
      <SettingsProvider>
        <MultipleContextExample />
      </SettingsProvider>
    </UserProvider>
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
    marginBottom: 30,
    color: '#333',
  },
  profileContainer: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  profileTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  profileText: {
    fontSize: 16,
    marginBottom: 8,
    color: '#666',
  },
  settingsContainer: {
    marginTop: 20,
    paddingTop: 20,
    borderTopWidth: 1,
    borderTopColor: '#eee',
  },
  settingsTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  settingItem: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 15,
  },
  toggleButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 15,
    paddingVertical: 8,
    borderRadius: 5,
  },
  toggleButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
  logoutButton: {
    backgroundColor: '#FF3B30',
    paddingVertical: 12,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 20,
  },
  logoutButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  loginContainer: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 12,
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  loginTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 20,
    color: '#333',
  },
  loginButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 30,
    paddingVertical: 15,
    borderRadius: 8,
  },
  disabledButton: {
    backgroundColor: '#CCCCCC',
  },
  loginButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default App;
```

---

## 🔄 useReducer Hook

useReducer hook complex state logic manage karne ke liye use hota hai. Yeh Redux pattern follow karta hai.

### 1. Basic useReducer

```javascript
import React, { useReducer } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

// Action types
const ACTIONS = {
  INCREMENT: 'increment',
  DECREMENT: 'decrement',
  RESET: 'reset',
  SET_VALUE: 'set_value'
};

// Reducer function
const counterReducer = (state, action) => {
  switch (action.type) {
    case ACTIONS.INCREMENT:
      return { ...state, count: state.count + 1 };
    case ACTIONS.DECREMENT:
      return { ...state, count: state.count - 1 };
    case ACTIONS.RESET:
      return { ...state, count: 0 };
    case ACTIONS.SET_VALUE:
      return { ...state, count: action.payload };
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
};

// Initial state
const initialState = {
  count: 0
};

const BasicReducerExample = () => {
  const [state, dispatch] = useReducer(counterReducer, initialState);
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>useReducer Counter</Text>
      
      <View style={styles.counterContainer}>
        <Text style={styles.counterValue}>{state.count}</Text>
      </View>
      
      <View style={styles.buttonContainer}>
        <TouchableOpacity
          style={[styles.button, styles.decrementButton]}
          onPress={() => dispatch({ type: ACTIONS.DECREMENT })}
        >
          <Text style={styles.buttonText}>-</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.button, styles.resetButton]}
          onPress={() => dispatch({ type: ACTIONS.RESET })}
        >
          <Text style={styles.buttonText}>Reset</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.button, styles.incrementButton]}
          onPress={() => dispatch({ type: ACTIONS.INCREMENT })}
        >
          <Text style={styles.buttonText}>+</Text>
        </TouchableOpacity>
      </View>
      
      <View style={styles.quickActions}>
        <TouchableOpacity
          style={styles.quickButton}
          onPress={() => dispatch({ type: ACTIONS.SET_VALUE, payload: 10 })}
        >
          <Text style={styles.quickButtonText}>Set to 10</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={styles.quickButton}
          onPress={() => dispatch({ type: ACTIONS.SET_VALUE, payload: 100 })}
        >
          <Text style={styles.quickButtonText}>Set to 100</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
    justifyContent: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
    color: '#333',
  },
  counterContainer: {
    backgroundColor: 'white',
    padding: 40,
    borderRadius: 20,
    alignItems: 'center',
    marginBottom: 30,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  counterValue: {
    fontSize: 48,
    fontWeight: 'bold',
    color: '#007AFF',
  },
  buttonContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginBottom: 20,
  },
  button: {
    paddingHorizontal: 20,
    paddingVertical: 15,
    borderRadius: 10,
    minWidth: 80,
    alignItems: 'center',
  },
  incrementButton: {
    backgroundColor: '#28A745',
  },
  decrementButton: {
    backgroundColor: '#DC3545',
  },
  resetButton: {
    backgroundColor: '#6C757D',
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  quickActions: {
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  quickButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 20,
    paddingVertical: 10,
    borderRadius: 8,
  },
  quickButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
});

export default BasicReducerExample;
```

### 2. Complex State with useReducer

```javascript
import React, { useReducer, useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  FlatList,
  StyleSheet,
  Alert
} from 'react-native';

// Action types
const TODO_ACTIONS = {
  ADD_TODO: 'add_todo',
  TOGGLE_TODO: 'toggle_todo',
  DELETE_TODO: 'delete_todo',
  EDIT_TODO: 'edit_todo',
  SET_FILTER: 'set_filter',
  CLEAR_COMPLETED: 'clear_completed'
};

// Filter types
const FILTERS = {
  ALL: 'all',
  ACTIVE: 'active',
  COMPLETED: 'completed'
};

// Reducer function
const todoReducer = (state, action) => {
  switch (action.type) {
    case TODO_ACTIONS.ADD_TODO:
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now().toString(),
            text: action.payload,
            completed: false,
            createdAt: new Date().toISOString()
          }
        ]
      };
      
    case TODO_ACTIONS.TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
      
    case TODO_ACTIONS.DELETE_TODO:
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
      
    case TODO_ACTIONS.EDIT_TODO:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload.id
            ? { ...todo, text: action.payload.text }
            : todo
        )
      };
      
    case TODO_ACTIONS.SET_FILTER:
      return {
        ...state,
        filter: action.payload
      };
      
    case TODO_ACTIONS.CLEAR_COMPLETED:
      return {
        ...state,
        todos: state.todos.filter(todo => !todo.completed)
      };
      
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
};

// Initial state
const initialState = {
  todos: [],
  filter: FILTERS.ALL
};

const TodoApp = () => {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  const [inputText, setInputText] = useState('');
  const [editingId, setEditingId] = useState(null);
  const [editText, setEditText] = useState('');
  
  // Add new todo
  const addTodo = () => {
    if (inputText.trim()) {
      dispatch({ type: TODO_ACTIONS.ADD_TODO, payload: inputText.trim() });
      setInputText('');
    }
  };
  
  // Toggle todo completion
  const toggleTodo = (id) => {
    dispatch({ type: TODO_ACTIONS.TOGGLE_TODO, payload: id });
  };
  
  // Delete todo
  const deleteTodo = (id) => {
    Alert.alert(
      'Delete Todo',
      'Are you sure you want to delete this todo?',
      [
        { text: 'Cancel', style: 'cancel' },
        {
          text: 'Delete',
          style: 'destructive',
          onPress: () => dispatch({ type: TODO_ACTIONS.DELETE_TODO, payload: id })
        }
      ]
    );
  };
  
  // Start editing
  const startEdit = (id, text) => {
    setEditingId(id);
    setEditText(text);
  };
  
  // Save edit
  const saveEdit = () => {
    if (editText.trim()) {
      dispatch({
        type: TODO_ACTIONS.EDIT_TODO,
        payload: { id: editingId, text: editText.trim() }
      });
    }
    setEditingId(null);
    setEditText('');
  };
  
  // Cancel edit
  const cancelEdit = () => {
    setEditingId(null);
    setEditText('');
  };
  
  // Filter todos
  const filteredTodos = state.todos.filter(todo => {
    switch (state.filter) {
      case FILTERS.ACTIVE:
        return !todo.completed;
      case FILTERS.COMPLETED:
        return todo.completed;
      default:
        return true;
    }
  });
  
  // Get stats
  const stats = {
    total: state.todos.length,
    active: state.todos.filter(todo => !todo.completed).length,
    completed: state.todos.filter(todo => todo.completed).length
  };
  
  // Render todo item
  const renderTodo = ({ item }) => (
    <View style={styles.todoItem}>
      {editingId === item.id ? (
        <View style={styles.editContainer}>
          <TextInput
            style={styles.editInput}
            value={editText}
            onChangeText={setEditText}
            autoFocus
          />
          <TouchableOpacity style={styles.saveButton} onPress={saveEdit}>
            <Text style={styles.saveButtonText}>Save</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.cancelButton} onPress={cancelEdit}>
            <Text style={styles.cancelButtonText}>Cancel</Text>
          </TouchableOpacity>
        </View>
      ) : (
        <View style={styles.todoContent}>
          <TouchableOpacity
            style={styles.todoTextContainer}
            onPress={() => toggleTodo(item.id)}
          >
            <View style={[
              styles.checkbox,
              item.completed && styles.checkboxCompleted
            ]}>
              {item.completed && <Text style={styles.checkmark}>✓</Text>}
            </View>
            <Text style={[
              styles.todoText,
              item.completed && styles.todoTextCompleted
            ]}>
              {item.text}
            </Text>
          </TouchableOpacity>
          
          <View style={styles.todoActions}>
            <TouchableOpacity
              style={styles.editButton}
              onPress={() => startEdit(item.id, item.text)}
            >
              <Text style={styles.editButtonText}>Edit</Text>
            </TouchableOpacity>
            <TouchableOpacity
              style={styles.deleteButton}
              onPress={() => deleteTodo(item.id)}
            >
              <Text style={styles.deleteButtonText}>Delete</Text>
            </TouchableOpacity>
          </View>
        </View>
      )}
    </View>
  );
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Todo App with useReducer</Text>
      
      {/* Stats */}
      <View style={styles.statsContainer}>
        <Text style={styles.statsText}>
          Total: {stats.total} | Active: {stats.active} | Completed: {stats.completed}
        </Text>
      </View>
      
      {/* Add todo */}
      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          placeholder="Add a new todo..."
          value={inputText}
          onChangeText={setInputText}
          onSubmitEditing={addTodo}
        />
        <TouchableOpacity style={styles.addButton} onPress={addTodo}>
          <Text style={styles.addButtonText}>Add</Text>
        </TouchableOpacity>
      </View>
      
      {/* Filters */}
      <View style={styles.filterContainer}>
        {Object.values(FILTERS).map(filter => (
          <TouchableOpacity
            key={filter}
            style={[
              styles.filterButton,
              state.filter === filter && styles.filterButtonActive
            ]}
            onPress={() => dispatch({ type: TODO_ACTIONS.SET_FILTER, payload: filter })}
          >
            <Text style={[
              styles.filterButtonText,
              state.filter === filter && styles.filterButtonTextActive
            ]}>
              {filter.charAt(0).toUpperCase() + filter.slice(1)}
            </Text>
          </TouchableOpacity>
        ))}
      </View>
      
      {/* Clear completed */}
      {stats.completed > 0 && (
        <TouchableOpacity
          style={styles.clearButton}
          onPress={() => dispatch({ type: TODO_ACTIONS.CLEAR_COMPLETED })}
        >
          <Text style={styles.clearButtonText}>Clear Completed</Text>
        </TouchableOpacity>
      )}
      
      {/* Todo list */}
      <FlatList
        data={filteredTodos}
        renderItem={renderTodo}
        keyExtractor={item => item.id}
        style={styles.todoList}
        showsVerticalScrollIndicator={false}
      />
    </View>
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
  statsContainer: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 8,
    marginBottom: 20,
    alignItems: 'center',
  },
  statsText: {
    fontSize: 16,
    color: '#666',
  },
  inputContainer: {
    flexDirection: 'row',
    marginBottom: 20,
  },
  input: {
    flex: 1,
    backgroundColor: 'white',
    paddingHorizontal: 15,
    paddingVertical: 12,
    borderRadius: 8,
    fontSize: 16,
    marginRight: 10,
  },
  addButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 20,
    paddingVertical: 12,
    borderRadius: 8,
    justifyContent: 'center',
  },
  addButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  filterContainer: {
    flexDirection: 'row',
    marginBottom: 20,
  },
  filterButton: {
    flex: 1,
    paddingVertical: 10,
    paddingHorizontal: 15,
    backgroundColor: 'white',
    marginHorizontal: 5,
    borderRadius: 8,
    alignItems: 'center',
  },
  filterButtonActive: {
    backgroundColor: '#007AFF',
  },
  filterButtonText: {
    fontSize: 14,
    color: '#666',
  },
  filterButtonTextActive: {
    color: 'white',
    fontWeight: 'bold',
  },
  clearButton: {
    backgroundColor: '#FF3B30',
    paddingVertical: 10,
    paddingHorizontal: 20,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: 20,
  },
  clearButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  todoList: {
    flex: 1,
  },
  todoItem: {
    backgroundColor: 'white',
    marginBottom: 10,
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.2,
    shadowRadius: 2,
    elevation: 3,
  },
  todoContent: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 15,
  },
  todoTextContainer: {
    flex: 1,
    flexDirection: 'row',
    alignItems: 'center',
  },
  checkbox: {
    width: 24,
    height: 24,
    borderRadius: 12,
    borderWidth: 2,
    borderColor: '#ddd',
    marginRight: 15,
    alignItems: 'center',
    justifyContent: 'center',
  },
  checkboxCompleted: {
    backgroundColor: '#007AFF',
    borderColor: '#007AFF',
  },
  checkmark: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  todoText: {
    fontSize: 16,
    color: '#333',
    flex: 1,
  },
  todoTextCompleted: {
    textDecorationLine: 'line-through',
    color: '#999',
  },
  todoActions: {
    flexDirection: 'row',
  },
  editButton: {
    backgroundColor: '#28A745',
    paddingHorizontal: 12,
    paddingVertical: 6,
    borderRadius: 4,
    marginRight: 8,
  },
  editButtonText: {
    color: 'white',
    fontSize: 12,
    fontWeight: 'bold',
  },
  deleteButton: {
    backgroundColor: '#FF3B30',
    paddingHorizontal: 12,
    paddingVertical: 6,
    borderRadius: 4,
  },
  deleteButtonText: {
    color: 'white',
    fontSize: 12,
    fontWeight: 'bold',
  },
  editContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 15,
  },
  editInput: {
    flex: 1,
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 4,
    paddingHorizontal: 10,
    paddingVertical: 8,
    fontSize: 16,
    marginRight: 10,
  },
  saveButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 12,
    paddingVertical: 8,
    borderRadius: 4,
    marginRight: 8,
  },
  saveButtonText: {
    color: 'white',
    fontSize: 12,
    fontWeight: 'bold',
  },
  cancelButton: {
    backgroundColor: '#6C757D',
    paddingHorizontal: 12,
    paddingVertical: 8,
    borderRadius: 4,
  },
  cancelButtonText: {
    color: 'white',
    fontSize: 12,
    fontWeight: 'bold',
  },
});

export default TodoApp;
```

---

## 🎣 Custom Hooks

Custom hooks reusable logic create karne ke liye use hote hain. Ye functions hain jo dusre hooks use kar sakte hain.

### 1. Basic Custom Hooks

```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, TouchableOpacity, StyleSheet, Alert, TextInput } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Custom hook for counter with persistence
const usePersistedCounter = (key, initialValue = 0) => {
  const [count, setCount] = useState(initialValue);
  const [loading, setLoading] = useState(true);

  // Load persisted value
  useEffect(() => {
    const loadPersistedValue = async () => {
      try {
        const persistedValue = await AsyncStorage.getItem(key);
        if (persistedValue !== null) {
          setCount(parseInt(persistedValue, 10));
        }
      } catch (error) {
        console.error('Failed to load persisted value:', error);
      } finally {
        setLoading(false);
      }
    };

    loadPersistedValue();
  }, [key]);

  // Persist value when it changes
  useEffect(() => {
    if (!loading) {
      AsyncStorage.setItem(key, count.toString()).catch(error => {
        console.error('Failed to persist value:', error);
      });
    }
  }, [count, key, loading]);

  const increment = () => setCount(prev => prev + 1);
  const decrement = () => setCount(prev => prev - 1);
  const reset = () => setCount(initialValue);

  return { count, increment, decrement, reset, loading };
};

// Custom hook for API calls
const useApi = (url) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchData = async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await fetch(url);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    if (url) {
      fetchData();
    }
  }, [url]);

  const refetch = () => {
    fetchData();
  };

  return { data, loading, error, refetch };
};

// Custom hook for form handling
const useForm = (initialValues, validationRules = {}) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  const setValue = (name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };

  const setFieldTouched = (name) => {
    setTouched(prev => ({ ...prev, [name]: true }));
  };

  const validate = () => {
    const newErrors = {};
    
    Object.keys(validationRules).forEach(field => {
      const rule = validationRules[field];
      const value = values[field];
      
      if (rule.required && (!value || value.toString().trim() === '')) {
        newErrors[field] = rule.required;
      } else if (rule.pattern && value && !rule.pattern.test(value)) {
        newErrors[field] = rule.patternMessage || 'Invalid format';
      } else if (rule.minLength && value && value.length < rule.minLength) {
        newErrors[field] = `Minimum ${rule.minLength} characters required`;
      }
    });
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const reset = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  };

  return {
    values,
    errors,
    touched,
    setValue,
    setTouched: setFieldTouched,
    validate,
    reset,
    isValid: Object.keys(errors).length === 0
  };
};

// Custom hook for toggle functionality
const useToggle = (initialValue = false) => {
  const [value, setValue] = useState(initialValue);
  
  const toggle = () => setValue(prev => !prev);
  const setTrue = () => setValue(true);
  const setFalse = () => setValue(false);
  
  return [value, { toggle, setTrue, setFalse }];
};

// Component using custom hooks
const CustomHooksExample = () => {
  const { count, increment, decrement, reset, loading } = usePersistedCounter('counter', 0);
  const { data, loading: apiLoading, error, refetch } = useApi('https://jsonplaceholder.typicode.com/posts/1');
  const [showDetails, { toggle: toggleDetails }] = useToggle(false);
  
  const formValidation = {
    name: {
      required: 'Name is required',
      minLength: 2
    },
    email: {
      required: 'Email is required',
      pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      patternMessage: 'Please enter a valid email'
    }
  };
  
  const {
    values,
    errors,
    setValue,
    validate,
    reset: resetForm
  } = useForm({ name: '', email: '' }, formValidation);

  const handleSubmit = () => {
    if (validate()) {
      Alert.alert('Success', 'Form is valid!');
    } else {
      Alert.alert('Error', 'Please fix the errors');
    }
  };

  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <Text>Loading persisted data...</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Custom Hooks Example</Text>
      
      {/* Persisted Counter */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Persisted Counter</Text>
        <Text style={styles.counterValue}>{count}</Text>
        <View style={styles.buttonRow}>
          <TouchableOpacity style={styles.button} onPress={decrement}>
            <Text style={styles.buttonText}>-</Text>
          </TouchableOpacity>
          <TouchableOpacity style={styles.button} onPress={increment}>
            <Text style={styles.buttonText}>+</Text>
          </TouchableOpacity>
          <TouchableOpacity style={[styles.button, styles.resetButton]} onPress={reset}>
            <Text style={styles.buttonText}>Reset</Text>
          </TouchableOpacity>
        </View>
      </View>

      {/* API Data */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>API Data</Text>
        {apiLoading ? (
          <Text>Loading...</Text>
        ) : error ? (
          <View>
            <Text style={styles.errorText}>Error: {error}</Text>
            <TouchableOpacity style={styles.button} onPress={refetch}>
              <Text style={styles.buttonText}>Retry</Text>
            </TouchableOpacity>
          </View>
        ) : (
          <View>
            <Text style={styles.dataTitle}>{data?.title}</Text>
            <TouchableOpacity style={styles.toggleButton} onPress={toggleDetails}>
              <Text style={styles.toggleButtonText}>
                {showDetails ? 'Hide' : 'Show'} Details
              </Text>
            </TouchableOpacity>
            {showDetails && (
              <Text style={styles.dataBody}>{data?.body}</Text>
            )}
          </View>
        )}
      </View>

      {/* Form Example */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Form with Validation</Text>
        
        <View style={styles.inputContainer}>
          <Text style={styles.inputLabel}>Name:</Text>
          <TextInput
            style={[styles.input, errors.name && styles.inputError]}
            value={values.name}
            onChangeText={(text) => setValue('name', text)}
            placeholder="Enter your name"
          />
          {errors.name && <Text style={styles.errorText}>{errors.name}</Text>}
        </View>

        <View style={styles.inputContainer}>
          <Text style={styles.inputLabel}>Email:</Text>
          <TextInput
            style={[styles.input, errors.email && styles.inputError]}
            value={values.email}
            onChangeText={(text) => setValue('email', text)}
            placeholder="Enter your email"
            keyboardType="email-address"
          />
          {errors.email && <Text style={styles.errorText}>{errors.email}</Text>}
        </View>

        <View style={styles.formButtons}>
          <TouchableOpacity style={styles.button} onPress={handleSubmit}>
            <Text style={styles.buttonText}>Submit</Text>
          </TouchableOpacity>
          <TouchableOpacity style={[styles.button, styles.resetButton]} onPress={resetForm}>
            <Text style={styles.buttonText}>Reset</Text>
          </TouchableOpacity>
        </View>
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
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
    color: '#333',
  },
  section: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 12,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  counterValue: {
    fontSize: 36,
    fontWeight: 'bold',
    textAlign: 'center',
    color: '#007AFF',
    marginBottom: 15,
  },
  buttonRow: {
    flexDirection: 'row',
    justifyContent: 'space-around',
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 20,
    paddingVertical: 10,
    borderRadius: 8,
    minWidth: 60,
    alignItems: 'center',
  },
  resetButton: {
    backgroundColor: '#FF3B30',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  errorText: {
    color: '#FF3B30',
    fontSize: 14,
    marginTop: 5,
  },
  dataTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333',
  },
  dataBody: {
    fontSize: 14,
    color: '#666',
    marginTop: 10,
    lineHeight: 20,
  },
  toggleButton: {
    backgroundColor: '#28A745',
    paddingHorizontal: 15,
    paddingVertical: 8,
    borderRadius: 6,
    alignSelf: 'flex-start',
  },
  toggleButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
  inputContainer: {
    marginBottom: 15,
  },
  inputLabel: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 5,
    color: '#333',
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    paddingHorizontal: 15,
    paddingVertical: 12,
    fontSize: 16,
  },
  inputError: {
    borderColor: '#FF3B30',
    borderWidth: 2,
  },
  formButtons: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    marginTop: 10,
  },
});

export default CustomHooksExample;
```

---

## 📝 Assignment 11: Advanced Hooks Mastery

### Task 1: Global State Management (180 minutes)
**Objective:** Create a complete global state management system using Context and useReducer

**Requirements:**
1. **User Management Context:**
   - User authentication state
   - Login/logout functionality
   - User profile management
   - Role-based permissions

2. **App Settings Context:**
   - Theme management (light/dark)
   - Language preferences
   - Notification settings
   - App configuration

3. **Shopping Cart Context:**
   - Add/remove items
   - Update quantities
   - Calculate totals
   - Persist cart data

### Task 2: Custom Hooks Library (150 minutes)
**Objective:** Build a comprehensive custom hooks library

**Requirements:**
Create following custom hooks:
1. **useLocalStorage** - AsyncStorage wrapper
2. **useNetworkStatus** - Network connectivity
3. **useGeolocation** - Location tracking
4. **useCamera** - Camera functionality
5. **useKeyboard** - Keyboard height tracking
6. **useOrientation** - Device orientation
7. **useBiometric** - Biometric authentication
8. **usePermissions** - Permission management

### Task 3: Complex State Management (120 minutes)
**Objective:** Build a complex form with advanced state management

**Requirements:**
1. **Multi-step Form:**
   - Personal information
   - Address details
   - Payment information
   - Review and submit

2. **Advanced Features:**
   - Form validation
   - Progress tracking
   - Data persistence
   - Error handling
   - Conditional fields

---

## 🧠 Quiz 11: Advanced Hooks (25 Questions)

### Multiple Choice Questions (1-15)

**Q1. useContext hook ka main purpose kya hai?**
a) State management  
b) Prop drilling avoid karna  
c) Performance optimization  
d) Error handling

**Q2. useReducer kab use karte hain useState ke bajaye?**
a) Simple state ke liye  
b) Complex state logic ke liye  
c) Performance ke liye  
d) Memory optimization ke liye

**Q3. Custom hook ka naam kaise start hona chahiye?**
a) with  
b) use  
c) hook  
d) custom

**Q4. Context Provider kahan place karna chahiye?**
a) Component ke andar  
b) App ke top level par  
c) Kahi bhi  
d) useEffect ke andar

**Q5. useReducer mein dispatch function kya karta hai?**
a) State return karta hai  
b) Action trigger karta hai  
c) Component re-render karta hai  
d) Error handle karta hai

**Q6. Custom hook mein dusre hooks use kar sakte hain?**
a) Nahi  
b) Haan  
c) Sirf useState  
d) Sirf useEffect

**Q7. Context value change hone par kya hota hai?**
a) Sirf Provider re-render hota hai  
b) All consumers re-render hote hain  
c) Kuch nahi hota  
d) Error aata hai

**Q8. useDebugValue hook kahan use karte hain?**
a) Production mein  
b) Custom hooks mein  
c) Components mein  
d) Reducers mein

**Q9. Multiple contexts use karne ka best practice kya hai?**
a) Ek hi context use karo  
b) Separate providers banao  
c) Nested providers avoid karo  
d) Global state use karo

**Q10. Reducer function mein kya return karna chahiye?**
a) Action  
b) New state  
c) Previous state  
d) Dispatch function

**Q11. useContext hook kya return karta hai?**
a) Provider component  
b) Context value  
c) Consumer component  
d) Dispatch function

**Q12. Custom hook mein state share kar sakte hain?**
a) Haan, automatically  
b) Nahi, har call independent hai  
c) Sirf same component mein  
d) Sirf Context ke saath

**Q13. useReducer ka initial state kaise set karte hain?**
a) useState mein  
b) Second parameter mein  
c) Reducer function mein  
d) useEffect mein

**Q14. Context API performance issue kab hota hai?**
a) Large objects pass karne se  
b) Frequent updates se  
c) Deep nesting se  
d) All of the above

**Q15. Custom hook test kaise karte hain?**
a) Component test karo  
b) Hook testing library use karo  
c) Manual testing  
d) Console.log use karo

### True/False Questions (16-20)

**Q16. useContext sirf functional components mein use kar sakte hain.** (True/False)

**Q17. Custom hooks component lifecycle access kar sakte hain.** (True/False)

**Q18. useReducer useState se faster hai.** (True/False)

**Q19. Context value object reference change hone se all consumers re-render hote hain.** (True/False)

**Q20. Custom hooks mein JSX return kar sakte hain.** (True/False)

### Short Answer Questions (21-25)

**Q21. useContext aur prop drilling mein kya difference hai?**

**Q22. useReducer pattern Redux se kaise similar hai?**

**Q23. Custom hook banane ke benefits kya hain?**

**Q24. Context performance optimize kaise karte hain?**

**Q25. Advanced hooks ke real-world use cases batao.**

---

## 🎯 Quiz Answers

**MCQ Answers:**
1. b) Prop drilling avoid karna
2. b) Complex state logic ke liye
3. b) use
4. b) App ke top level par
5. b) Action trigger karta hai
6. b) Haan
7. b) All consumers re-render hote hain
8. b) Custom hooks mein
9. b) Separate providers banao
10. b) New state
11. b) Context value
12. b) Nahi, har call independent hai
13. b) Second parameter mein
14. d) All of the above
15. b) Hook testing library use karo

**True/False Answers:**
16. False (class components mein bhi use kar sakte hain with Consumer)
17. True (useEffect ke through)
18. False (performance similar hai)
19. True
20. False (hooks sirf logic return karte hain)

**Short Answers:**
21. useContext global state provide karta hai, prop drilling mein har level pe props pass karna padta hai
22. Dono mein actions dispatch karte hain aur reducer functions state update karte hain
23. Code reusability, logic separation, easier testing, cleaner components
24. useMemo, useCallback use karo, context value ko optimize karo
25. Authentication, theme management, API calls, form handling, local storage

---

## 🔗 Additional Resources

### Documentation
- [React Context API](https://reactjs.org/docs/context.html)
- [useReducer Hook](https://reactjs.org/docs/hooks-reference.html#usereducer)
- [Building Custom Hooks](https://reactjs.org/docs/hooks-custom.html)

### Libraries
- [React Hook Form](https://react-hook-form.com/)
- [SWR](https://swr.vercel.app/) - Data fetching hooks
- [React Query](https://tanstack.com/query) - Server state management

### Best Practices
- [Rules of Hooks](https://reactjs.org/docs/hooks-rules.html)
- [Hook Testing](https://react-hooks-testing-library.com/)
- [Performance Optimization](https://reactjs.org/docs/optimizing-performance.html)

---

## 🎉 Lesson Complete!

Congratulations! Aapne successfully:
- ✅ useContext hook se global state management sikha
- ✅ useReducer hook se complex state logic handle kiya
- ✅ Custom hooks create karne ki techniques samjhi
- ✅ Advanced hook patterns aur best practices sikhe
- ✅ Real-world applications mein hooks ka use dekha

**Next Lesson Preview:** Component Lifecycle & Performance Optimization - React Native components ki lifecycle samjhenge aur performance optimize karne ke advanced techniques sikhenge.

---

*Happy Hooking! 🪝✨*