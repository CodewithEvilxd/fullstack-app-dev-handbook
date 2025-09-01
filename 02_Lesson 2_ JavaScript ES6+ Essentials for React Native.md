
# 🔥 Lesson 2: JavaScript ES6+ Essentials for React Native

## 🎯 Learning Objectives
Is lesson ke baad aap:
- Modern JavaScript (ES6+) features samjhenge
- Arrow functions, destructuring, spread operator use kar sakte hain
- Promises, async/await master kar sakte hain
- Array methods aur object manipulation kar sakte hain
- React Native mein modern JS patterns apply kar sakte hain

---

## 📖 Why Modern JavaScript?

React Native modern JavaScript features heavily use karta hai. ES6+ features code ko:
- **Cleaner** banate hain
- **More readable** banate hain
- **Less verbose** banate hain
- **More powerful** banate hain

---

## 🔧 ES6+ Features Essential for React Native

### 1. Let & Const vs Var

```javascript
// ❌ Old way (var)
var name = 'John';
var age = 25;
age = 26; // Can be reassigned

// ✅ Modern way
const name = 'John';    // Cannot be reassigned
let age = 25;           // Can be reassigned
age = 26;

// Block scope example
function example() {
  if (true) {
    var varVariable = 'I am var';
    let letVariable = 'I am let';
    const constVariable = 'I am const';
  }
  
  console.log(varVariable);    // ✅ Works
  // console.log(letVariable); // ❌ Error
  // console.log(constVariable); // ❌ Error
}
```

### 2. Arrow Functions

```javascript
// ❌ Traditional function
function add(a, b) {
  return a + b;
}

// ✅ Arrow function
const add = (a, b) => {
  return a + b;
};

// ✅ Shorter arrow function
const add = (a, b) => a + b;

// ✅ Single parameter (no parentheses needed)
const square = x => x * x;

// React Native example
const Button = ({ title, onPress }) => (
  <TouchableOpacity onPress={onPress}>
    <Text>{title}</Text>
  </TouchableOpacity>
);

// Event handlers
const handlePress = () => {
  console.log('Button pressed!');
};

// Array methods with arrow functions
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(num => num * 2);
const evens = numbers.filter(num => num % 2 === 0);
```

### 3. Template Literals

```javascript
// ❌ Old string concatenation
const name = 'John';
const age = 25;
const message = 'Hello, my name is ' + name + ' and I am ' + age + ' years old.';

// ✅ Template literals
const message = `Hello, my name is ${name} and I am ${age} years old.`;

// Multi-line strings
const multiLine = `
  This is a
  multi-line
  string
`;

// React Native example
const WelcomeText = ({ userName, userAge }) => (
  <Text style={styles.welcome}>
    {`Welcome back, ${userName}! You are ${userAge} years old.`}
  </Text>
);

// Dynamic styling
const dynamicStyle = (isActive) => `
  background-color: ${isActive ? '#007AFF' : '#gray'};
  padding: 10px;
`;
```

### 4. Destructuring

#### Object Destructuring
```javascript
// ❌ Old way
const user = {
  name: 'John',
  age: 25,
  email: 'john@example.com',
  address: {
    city: 'Mumbai',
    state: 'Maharashtra'
  }
};

const name = user.name;
const age = user.age;
const email = user.email;

// ✅ Object destructuring
const { name, age, email } = user;

// With default values
const { name, age, phone = 'Not provided' } = user;

// Nested destructuring
const { address: { city, state } } = user;

// Renaming variables
const { name: userName, age: userAge } = user;

// React Native component props
const UserProfile = ({ name, age, email, onEdit }) => {
  return (
    <View>
      <Text>{name}</Text>
      <Text>{age}</Text>
      <Text>{email}</Text>
      <TouchableOpacity onPress={onEdit}>
        <Text>Edit</Text>
      </TouchableOpacity>
    </View>
  );
};

// Function parameters destructuring
const handleUserUpdate = ({ id, name, email }) => {
  // Update user logic
  console.log(`Updating user ${id}: ${name} - ${email}`);
};
```

#### Array Destructuring
```javascript
// ❌ Old way
const colors = ['red', 'green', 'blue'];
const firstColor = colors[0];
const secondColor = colors[1];

// ✅ Array destructuring
const [firstColor, secondColor, thirdColor] = colors;

// Skip elements
const [first, , third] = colors;

// With default values
const [first, second, third, fourth = 'yellow'] = colors;

// React Native useState example
import { useState } from 'react';

const Counter = () => {
  const [count, setCount] = useState(0);
  
  return (
    <View>
      <Text>{count}</Text>
      <TouchableOpacity onPress={() => setCount(count + 1)}>
        <Text>Increment</Text>
      </TouchableOpacity>
    </View>
  );
};
```

### 5. Spread Operator (...)

```javascript
// Array spreading
const numbers1 = [1, 2, 3];
const numbers2 = [4, 5, 6];

// ❌ Old way
const combined = numbers1.concat(numbers2);

// ✅ Spread operator
const combined = [...numbers1, ...numbers2];

// Adding elements
const newNumbers = [...numbers1, 4, 5, ...numbers2];

// Object spreading
const user = {
  name: 'John',
  age: 25
};

// ❌ Old way
const updatedUser = Object.assign({}, user, { age: 26 });

// ✅ Spread operator
const updatedUser = { ...user, age: 26 };

// React Native state updates
const [user, setUser] = useState({
  name: 'John',
  age: 25,
  email: 'john@example.com'
});

const updateUserAge = (newAge) => {
  setUser({ ...user, age: newAge });
};

// Function arguments
const sum = (a, b, c) => a + b + c;
const numbers = [1, 2, 3];
const result = sum(...numbers);

// React Native component props
const defaultProps = {
  backgroundColor: '#fff',
  padding: 10,
  borderRadius: 5
};

const CustomButton = (props) => (
  <TouchableOpacity style={{ ...defaultProps, ...props.style }}>
    <Text>{props.title}</Text>
  </TouchableOpacity>
);
```

### 6. Rest Parameters

```javascript
// Function with rest parameters
const sum = (...numbers) => {
  return numbers.reduce((total, num) => total + num, 0);
};

console.log(sum(1, 2, 3, 4, 5)); // 15

// Object rest
const user = {
  name: 'John',
  age: 25,
  email: 'john@example.com',
  phone: '123-456-7890'
};

const { name, age, ...otherDetails } = user;
console.log(otherDetails); // { email: '...', phone: '...' }

// React Native component
const CustomInput = ({ label, error, ...inputProps }) => (
  <View>
    <Text>{label}</Text>
    <TextInput {...inputProps} />
    {error && <Text style={styles.error}>{error}</Text>}
  </View>
);
```

### 7. Enhanced Object Literals

```javascript
const name = 'John';
const age = 25;

// ❌ Old way
const user = {
  name: name,
  age: age,
  greet: function() {
    return 'Hello!';
  }
};

// ✅ Enhanced object literals
const user = {
  name,           // Shorthand property
  age,
  greet() {       // Method shorthand
    return 'Hello!';
  },
  [`full_${name}`]: true  // Computed property names
};

// React Native styles
const createStyles = (theme) => ({
  container: {
    flex: 1,
    backgroundColor: theme.backgroundColor
  },
  [`${theme.mode}Text`]: {
    color: theme.textColor
  }
});
```

---

## 🔄 Asynchronous JavaScript

### 8. Promises

```javascript
// Creating a Promise
const fetchUserData = (userId) => {
  return new Promise((resolve, reject) => {
    // Simulate API call
    setTimeout(() => {
      if (userId) {
        resolve({
          id: userId,
          name: 'John Doe',
          email: 'john@example.com'
        });
      } else {
        reject(new Error('User ID is required'));
      }
    }, 1000);
  });
};

// Using Promises
fetchUserData(123)
  .then(user => {
    console.log('User data:', user);
    return user.email;
  })
  .then(email => {
    console.log('User email:', email);
  })
  .catch(error => {
    console.error('Error:', error.message);
  })
  .finally(() => {
    console.log('Request completed');
  });

// React Native example
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchUserData(userId)
      .then(userData => {
        setUser(userData);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]);

  if (loading) return <Text>Loading...</Text>;
  if (error) return <Text>Error: {error}</Text>;
  
  return (
    <View>
      <Text>{user.name}</Text>
      <Text>{user.email}</Text>
    </View>
  );
};
```

### 9. Async/Await

```javascript
// ❌ Promise chains
const getUserAndPosts = (userId) => {
  return fetchUserData(userId)
    .then(user => {
      return fetchUserPosts(user.id)
        .then(posts => {
          return { user, posts };
        });
    });
};

// ✅ Async/Await
const getUserAndPosts = async (userId) => {
  try {
    const user = await fetchUserData(userId);
    const posts = await fetchUserPosts(user.id);
    return { user, posts };
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
};

// React Native component with async/await
const UserDashboard = ({ userId }) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const loadUserData = async () => {
      try {
        setLoading(true);
        const userData = await getUserAndPosts(userId);
        setData(userData);
      } catch (error) {
        console.error('Failed to load user data:', error);
      } finally {
        setLoading(false);
      }
    };

    loadUserData();
  }, [userId]);

  const handleRefresh = async () => {
    try {
      setLoading(true);
      const freshData = await getUserAndPosts(userId);
      setData(freshData);
    } catch (error) {
      Alert.alert('Error', 'Failed to refresh data');
    } finally {
      setLoading(false);
    }
  };

  return (
    <ScrollView>
      {loading ? (
        <ActivityIndicator size="large" />
      ) : (
        <View>
          <Text>{data?.user?.name}</Text>
          <TouchableOpacity onPress={handleRefresh}>
            <Text>Refresh</Text>
          </TouchableOpacity>
        </View>
      )}
    </ScrollView>
  );
};
```

---

## 🔄 Array Methods

### 10. Essential Array Methods

```javascript
const users = [
  { id: 1, name: 'John', age: 25, active: true },
  { id: 2, name: 'Jane', age: 30, active: false },
  { id: 3, name: 'Bob', age: 35, active: true },
  { id: 4, name: 'Alice', age: 28, active: true }
];

// map() - Transform array elements
const userNames = users.map(user => user.name);
const userCards = users.map(user => ({
  ...user,
  displayName: `${user.name} (${user.age})`
}));

// filter() - Filter array elements
const activeUsers = users.filter(user => user.active);
const youngUsers = users.filter(user => user.age < 30);

// find() - Find single element
const johnUser = users.find(user => user.name === 'John');
const userById = users.find(user => user.id === 2);

// reduce() - Reduce array to single value
const totalAge = users.reduce((sum, user) => sum + user.age, 0);
const usersByAge = users.reduce((acc, user) => {
  acc[user.age] = user;
  return acc;
}, {});

// some() & every()
const hasActiveUsers = users.some(user => user.active);
const allUsersActive = users.every(user => user.active);

// forEach() - Execute function for each element
users.forEach(user => {
  console.log(`${user.name} is ${user.age} years old`);
});

// React Native FlatList example
const UserList = () => {
  const [users, setUsers] = useState([]);
  const [searchTerm, setSearchTerm] = useState('');

  const filteredUsers = users.filter(user =>
    user.name.toLowerCase().includes(searchTerm.toLowerCase())
  );

  const renderUser = ({ item }) => (
    <TouchableOpacity style={styles.userItem}>
      <Text style={styles.userName}>{item.name}</Text>
      <Text style={styles.userAge}>Age: {item.age}</Text>
      <View style={[
        styles.statusBadge,
        { backgroundColor: item.active ? 'green' : 'red' }
      ]}>
        <Text style={styles.statusText}>
          {item.active ? 'Active' : 'Inactive'}
        </Text>
      </View>
    </TouchableOpacity>
  );

  return (
    <View style={styles.container}>
      <TextInput
        style={styles.searchInput}
        placeholder="Search users..."
        value={searchTerm}
        onChangeText={setSearchTerm}
      />
      <FlatList
        data={filteredUsers}
        renderItem={renderUser}
        keyExtractor={item => item.id.toString()}
      />
    </View>
  );
};
```

---

## 🏗️ Modules (Import/Export)

### 11. ES6 Modules

```javascript
// utils.js - Named exports
export const formatCurrency = (amount) => {
  return `₹${amount.toLocaleString('en-IN')}`;
};

export const formatDate = (date) => {
  return new Date(date).toLocaleDateString('en-IN');
};

export const validateEmail = (email) => {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
};

// Default export
const API_BASE_URL = 'https://api.example.com';
export default API_BASE_URL;

// userService.js - Service module
import API_BASE_URL from './utils';

class UserService {
  static async getUsers() {
    const response = await fetch(`${API_BASE_URL}/users`);
    return response.json();
  }

  static async createUser(userData) {
    const response = await fetch(`${API_BASE_URL}/users`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(userData),
    });
    return response.json();
  }
}

export default UserService;

// App.js - Using imports
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList } from 'react-native';
import UserService from './services/userService';
import { formatCurrency, formatDate } from './utils/utils';

const App = () => {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    const loadUsers = async () => {
      try {
        const userData = await UserService.getUsers();
        setUsers(userData);
      } catch (error) {
        console.error('Failed to load users:', error);
      }
    };

    loadUsers();
  }, []);

  return (
    <View>
      <FlatList
        data={users}
        renderItem={({ item }) => (
          <View>
            <Text>{item.name}</Text>
            <Text>{formatCurrency(item.salary)}</Text>
            <Text>{formatDate(item.joinDate)}</Text>
          </View>
        )}
        keyExtractor={item => item.id.toString()}
      />
    </View>
  );
};

export default App;
```

---

## 🎯 Practical React Native Examples

### 12. Complete Component Example

```javascript
// components/TodoApp.js
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  FlatList,
  Alert,
  StyleSheet
} from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

const TodoApp = () => {
  const [todos, setTodos] = useState([]);
  const [inputText, setInputText] = useState('');
  const [filter, setFilter] = useState('all'); // all, active, completed

  // Load todos from storage
  useEffect(() => {
    const loadTodos = async () => {
      try {
        const storedTodos = await AsyncStorage.getItem('todos');
        if (storedTodos) {
          setTodos(JSON.parse(storedTodos));
        }
      } catch (error) {
        console.error('Failed to load todos:', error);
      }
    };

    loadTodos();
  }, []);

  // Save todos to storage
  useEffect(() => {
    const saveTodos = async () => {
      try {
        await AsyncStorage.setItem('todos', JSON.stringify(todos));
      } catch (error) {
        console.error('Failed to save todos:', error);
      }
    };

    if (todos.length > 0) {
      saveTodos();
    }
  }, [todos]);

  // Add new todo
  const addTodo = () => {
    if (!inputText.trim()) {
      Alert.alert('Error', 'Please enter a todo item');
      return;
    }

    const newTodo = {
      id: Date.now().toString(),
      text: inputText.trim(),
      completed: false,
      createdAt: new Date().toISOString()
    };

    setTodos(prevTodos => [...prevTodos, newTodo]);
    setInputText('');
  };

  // Toggle todo completion
  const toggleTodo = (id) => {
    setTodos(prevTodos =>
      prevTodos.map(todo =>
        todo.id === id
          ? { ...todo, completed: !todo.completed }
          : todo
      )
    );
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
          onPress: () => {
            setTodos(prevTodos => prevTodos.filter(todo => todo.id !== id));
          }
        }
      ]
    );
  };

  // Filter todos
  const filteredTodos = todos.filter(todo => {
    switch (filter) {
      case 'active':
        return !todo.completed;
      case 'completed':
        return todo.completed;
      default:
        return true;
    }
  });

  // Get statistics
  const stats = {
    total: todos.length,
    active: todos.filter(todo => !todo.completed).length,
    completed: todos.filter(todo => todo.completed).length
  };

  // Render todo item
  const renderTodoItem = ({ item }) => (
    <View style={styles.todoItem}>
      <TouchableOpacity
        style={styles.todoContent}
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
      <TouchableOpacity
        style={styles.deleteButton}
        onPress={() => deleteTodo(item.id)}
      >
        <Text style={styles.deleteButtonText}>🗑️</Text>
      </TouchableOpacity>
    </View>
  );

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Todo App</Text>
      
      {/* Stats */}
      <View style={styles.statsContainer}>
        <Text style={styles.statsText}>
          Total: {stats.total} | Active: {stats.active} | Completed: {stats.completed}
        </Text>
      </View>

      {/* Input */}
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

      {/* Filter buttons */}
      <View style={styles.filterContainer}>
        {['all', 'active', 'completed'].map(filterType => (
          <TouchableOpacity
            key={filterType}
            style={[
              styles.filterButton,
              filter === filterType && styles.filterButtonActive
            ]}
            onPress={() => setFilter(filterType)}
          >
            <Text style={[
              styles.filterButtonText,
              filter === filterType && styles.filterButtonTextActive
            ]}>
              {filterType.charAt(0).toUpperCase() + filterType.slice(1)}
            </Text>
          </TouchableOpacity>
        ))}
      </View>

      {/* Todo list */}
      <FlatList
        data={filteredTodos}
        renderItem={renderTodoItem}
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
    fontSize: 28,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
    color: '#333',
  },
  statsContainer: {
    backgroundColor: '#fff',
    padding: 15,
    borderRadius: 10,
    marginBottom: 20,
  },
  statsText: {
    textAlign: 'center',
    fontSize: 16,
    color: '#666',
  },
  inputContainer: {
    flexDirection: 'row',
    marginBottom: 20,
  },
  input: {
    flex: 1,
    backgroundColor: '#fff',
    paddingHorizontal: 15,
    paddingVertical: 12,
    borderRadius: 10,
    fontSize: 16,
    marginRight: 10,
  },
  addButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 20,
    paddingVertical: 12,
    borderRadius: 10,
    justifyContent: 'center',
  },
  addButtonText: {
    color: '#fff',
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
    backgroundColor: '#fff',
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
    color: '#fff',
    fontWeight: 'bold',
  },
  todoList: {
    flex: 1,
  },
  todoItem: {
    flexDirection: 'row',
    backgroundColor: '#fff',
    padding: 15,
    borderRadius: 10,
    marginBottom: 10,
    alignItems: 'center',
  },
  todoContent: {
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
    color: '#fff',
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
  deleteButton: {
    padding: 10,
  },
  deleteButtonText: {
    fontSize: 18,
  },
});

export default TodoApp;
```

---

## 📝 Assignment 2: Modern JavaScript Practice

### Task 1: Array Methods Practice (45 minutes)
```javascript
// Given data
const products = [
  { id: 1, name: 'iPhone', price: 80000, category: 'electronics', inStock: true },
  { id: 2, name: 'Shirt', price: 1500, category: 'clothing', inStock: false },
  { id: 3, name: 'Laptop', price: 60000, category: 'electronics', inStock: true },
  { id: 4, name: 'Jeans', price: 2500, category: 'clothing', inStock: true },
  { id: 5, name: 'Watch', price: 15000, category: 'accessories', inStock: false }
];

// Complete these tasks:
// 1. Get all product names
// 2. Filter products under ₹20,000
// 3. Find the most expensive product
// 4. Calculate total value of in-stock products
// 5. Group products by category
// 6. Check if any electronics are out of stock
```

### Task 2: Async/Await Practice (60 minutes)
```javascript
// Create a weather app component that:
// 1. Fetches weather data from API
// 2. Shows loading state
// 3. Handles errors gracefully
// 4. Allows refresh functionality
// 5. Uses async/await properly

const WeatherApp = () => {
  // Your implementation here
};
```

### Task 3: Destructuring & Spread Practice (30 minutes)
```javascript
// Given user data
const userData = {
  personal: {
    name: 'John Doe',
    age: 28,
    email: 'john@example.com'
  },
  preferences: {
    theme: 'dark',
    language: 'en',
    notifications: true
  },
  address: {
    street: '123 Main St',
    city: 'Mumbai',
    state: 'Maharashtra',
    pincode: '400001'
  }
};

// Tasks:
// 1. Extract name, age, and email using destructuring
// 2. Extract city and state from nested address
// 3. Create a new user object with updated age
// 4. Merge preferences with new settings
// 5. Create a function that accepts destructured parameters
```

---

## 🧠 Quiz 2: Modern JavaScript

### Multiple Choice Questions

**Q1. Arrow functions mein 'this' keyword ka behavior kya hai?**
a) Regular functions ke jaisa
b) Lexical binding (parent scope se inherit)
c) Always undefined
d) Global object ko point karta hai

**Q2. Template literals mein expressions embed karne ke liye kya use karte hain?**
a) ${expression}
b) {expression}
c) {{expression}}
d) #expression#

**Q3. Spread operator ka symbol kya hai?**
a) ***
b) ...
c) +++
d) <<<

**Q4. Array.map() method kya return karta hai?**
a) Original array
b) Boolean value
c) New array with transformed elements
d) Single value

**Q5. async/await kya hai?**
a) Promise-based code likhne ka syntactic sugar
b) New type of function
c) Error handling mechanism
d) Loop construct

### True/False Questions

**Q6. const variables ko reassign kar sakte hain.** (True/False)

**Q7. Arrow functions ko constructor ke roop mein use kar sakte hain.** (True/False)

**Q8. Destructuring sirf objects ke saath kaam karta hai.** (True/False)

**Q9. Array.filter() original array ko modify karta hai.** (True/False)

**Q10. Template literals multi-line strings support karte hain.** (True/False)

### Code Output Questions

**Q11. Is code ka output kya hoga?**
```javascript
const arr = [1, 2, 3];
const newArr = [...arr, 4, 5];
console.log(newArr);
```

**Q12. Is code ka output kya hoga?**
```javascript
const user = { name: 'John', age: 25 };
const { name, age, city = 'Mumbai' } = user;
console.log(city);
```

**Q13. Is code ka output kya hoga?**
```javascript
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(num => num * 2);
console.log(doubled);
```

---

## 📚 Answers & Explanations

### Multiple Choice Answers
**Q1.** b) Lexical binding (parent scope se inherit)
**Q2.** a) ${expression}
**Q3.** b) ...
**Q4.** c) New array with transformed elements
**Q5.** a) Promise-based code likhne ka syntactic sugar

### True/False Answers
**Q6.** False - const variables ko reassign nahi kar sakte
**Q7.** False - Arrow functions ko constructor ke roop mein use nahi kar sakte
**Q8.** False - Destructuring arrays aur objects dono ke saath kaam karta hai
**Q9.** False - Array.filter() original array ko modify nahi karta
**Q10.** True - Template literals multi-line strings support karte hain

### Code Output Answers
**Q11.** `[1, 2, 3, 4, 5]` - Spread operator array elements ko expand karta hai
**Q12.** `'Mumbai'` - Default value use hoti hai jab property exist nahi karti
**Q13.** `[2, 4, 6, 8, 10]` - map() method har element ko double karta hai

---

## 🎯 **Assignment Solutions**

### **Task 1: Array Methods Practice Solutions**

```javascript
// Given data
const products = [
  { id: 1, name: 'iPhone', price: 80000, category: 'electronics', inStock: true },
  { id: 2, name: 'Shirt', price: 1500, category: 'clothing', inStock: false },
  { id: 3, name: 'Laptop', price: 60000, category: 'electronics', inStock: true },
  { id: 4, name: 'Jeans', price: 2500, category: 'clothing', inStock: true },
  { id: 5, name: 'Watch', price: 15000, category: 'accessories', inStock: false }
];

// 1. Get all product names
const productNames = products.map(product => product.name);
console.log('Product Names:', productNames);
// Output: ['iPhone', 'Shirt', 'Laptop', 'Jeans', 'Watch']

// 2. Filter products under ₹20,000
const affordableProducts = products.filter(product => product.price < 20000);
console.log('Affordable Products:', affordableProducts);
// Output: Products with price < 20000

// 3. Find the most expensive product
const mostExpensive = products.reduce((max, product) =>
  product.price > max.price ? product : max
);
console.log('Most Expensive:', mostExpensive);
// Output: iPhone object

// 4. Calculate total value of in-stock products
const totalValue = products
  .filter(product => product.inStock)
  .reduce((total, product) => total + product.price, 0);
console.log('Total Value of In-stock Products:', totalValue);
// Output: 140000

// 5. Group products by category
const productsByCategory = products.reduce((groups, product) => {
  if (!groups[product.category]) {
    groups[product.category] = [];
  }
  groups[product.category].push(product);
  return groups;
}, {});
console.log('Products by Category:', productsByCategory);

// 6. Check if any electronics are out of stock
const hasOutOfStockElectronics = products
  .filter(product => product.category === 'electronics')
  .some(product => !product.inStock);
console.log('Any Electronics Out of Stock:', hasOutOfStockElectronics);
// Output: false
```

### **Task 2: Async/Await Practice Solution**

```javascript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  ActivityIndicator,
  Alert,
  StyleSheet
} from 'react-native';

const WeatherApp = () => {
  const [city, setCity] = useState('Mumbai');
  const [weather, setWeather] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // API key (in production, store securely)
  const API_KEY = 'your_weather_api_key';
  const API_URL = 'https://api.openweathermap.org/data/2.5/weather';

  // Fetch weather data
  const fetchWeather = async (cityName) => {
    try {
      setLoading(true);
      setError(null);

      const response = await fetch(
        `${API_URL}?q=${cityName}&appid=${API_KEY}&units=metric`
      );

      if (!response.ok) {
        throw new Error(`Weather API error: ${response.status}`);
      }

      const data = await response.json();
      setWeather(data);
    } catch (err) {
      setError(err.message);
      setWeather(null);
    } finally {
      setLoading(false);
    }
  };

  // Handle search
  const handleSearch = async () => {
    if (!city.trim()) {
      Alert.alert('Error', 'Please enter a city name');
      return;
    }

    await fetchWeather(city.trim());
  };

  // Handle refresh
  const handleRefresh = async () => {
    if (city.trim()) {
      await fetchWeather(city.trim());
    }
  };

  // Load default city weather on mount
  useEffect(() => {
    fetchWeather('Mumbai');
  }, []);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Weather App</Text>

      {/* Search Input */}
      <View style={styles.searchContainer}>
        <TextInput
          style={styles.input}
          placeholder="Enter city name..."
          value={city}
          onChangeText={setCity}
          onSubmitEditing={handleSearch}
        />
        <TouchableOpacity style={styles.searchButton} onPress={handleSearch}>
          <Text style={styles.searchButtonText}>Search</Text>
        </TouchableOpacity>
      </View>

      {/* Loading State */}
      {loading && (
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color="#007AFF" />
          <Text style={styles.loadingText}>Loading weather data...</Text>
        </View>
      )}

      {/* Error State */}
      {error && (
        <View style={styles.errorContainer}>
          <Text style={styles.errorText}>❌ {error}</Text>
          <TouchableOpacity style={styles.retryButton} onPress={handleRefresh}>
            <Text style={styles.retryButtonText}>Retry</Text>
          </TouchableOpacity>
        </View>
      )}

      {/* Weather Data */}
      {weather && !loading && (
        <View style={styles.weatherContainer}>
          <Text style={styles.cityName}>{weather.name}, {weather.sys.country}</Text>
          <Text style={styles.temperature}>{Math.round(weather.main.temp)}°C</Text>
          <Text style={styles.description}>{weather.weather[0].description}</Text>

          <View style={styles.detailsContainer}>
            <View style={styles.detail}>
              <Text style={styles.detailLabel}>Humidity</Text>
              <Text style={styles.detailValue}>{weather.main.humidity}%</Text>
            </View>
            <View style={styles.detail}>
              <Text style={styles.detailLabel}>Wind Speed</Text>
              <Text style={styles.detailValue}>{weather.wind.speed} m/s</Text>
            </View>
            <View style={styles.detail}>
              <Text style={styles.detailLabel}>Pressure</Text>
              <Text style={styles.detailValue}>{weather.main.pressure} hPa</Text>
            </View>
          </View>

          <TouchableOpacity style={styles.refreshButton} onPress={handleRefresh}>
            <Text style={styles.refreshButtonText}>Refresh</Text>
          </TouchableOpacity>
        </View>
      )}
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
    fontSize: 28,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
    color: '#333',
  },
  searchContainer: {
    flexDirection: 'row',
    marginBottom: 20,
  },
  input: {
    flex: 1,
    backgroundColor: '#fff',
    paddingHorizontal: 15,
    paddingVertical: 12,
    borderRadius: 10,
    fontSize: 16,
    marginRight: 10,
  },
  searchButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 20,
    paddingVertical: 12,
    borderRadius: 10,
    justifyContent: 'center',
  },
  searchButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  loadingText: {
    marginTop: 10,
    fontSize: 16,
    color: '#666',
  },
  errorContainer: {
    backgroundColor: '#ffebee',
    padding: 20,
    borderRadius: 10,
    marginBottom: 20,
  },
  errorText: {
    fontSize: 16,
    color: '#c62828',
    textAlign: 'center',
    marginBottom: 15,
  },
  retryButton: {
    backgroundColor: '#c62828',
    padding: 10,
    borderRadius: 5,
    alignItems: 'center',
  },
  retryButtonText: {
    color: '#fff',
    fontSize: 14,
    fontWeight: 'bold',
  },
  weatherContainer: {
    backgroundColor: '#fff',
    padding: 20,
    borderRadius: 15,
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  cityName: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 10,
  },
  temperature: {
    fontSize: 48,
    fontWeight: 'bold',
    color: '#007AFF',
    marginBottom: 5,
  },
  description: {
    fontSize: 18,
    color: '#666',
    textTransform: 'capitalize',
    marginBottom: 20,
  },
  detailsContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    width: '100%',
    marginBottom: 20,
  },
  detail: {
    alignItems: 'center',
  },
  detailLabel: {
    fontSize: 14,
    color: '#666',
    marginBottom: 5,
  },
  detailValue: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
  },
  refreshButton: {
    backgroundColor: '#28a745',
    paddingHorizontal: 30,
    paddingVertical: 12,
    borderRadius: 25,
  },
  refreshButtonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default WeatherApp;
```

### **Task 3: Destructuring & Spread Practice Solutions**

```javascript
// Given user data
const userData = {
  personal: {
    name: 'John Doe',
    age: 28,
    email: 'john@example.com'
  },
  preferences: {
    theme: 'dark',
    language: 'en',
    notifications: true
  },
  address: {
    street: '123 Main St',
    city: 'Mumbai',
    state: 'Maharashtra',
    pincode: '400001'
  }
};

// 1. Extract name, age, and email using destructuring
const { personal: { name, age, email } } = userData;
console.log('Name:', name);     // 'John Doe'
console.log('Age:', age);       // 28
console.log('Email:', email);   // 'john@example.com'

// 2. Extract city and state from nested address
const { address: { city, state } } = userData;
console.log('City:', city);     // 'Mumbai'
console.log('State:', state);   // 'Maharashtra'

// 3. Create a new user object with updated age
const updatedUser = {
  ...userData,
  personal: {
    ...userData.personal,
    age: 29
  }
};
console.log('Updated age:', updatedUser.personal.age); // 29

// 4. Merge preferences with new settings
const newPreferences = {
  theme: 'light',
  language: 'hi',
  notifications: false,
  autoSave: true
};

const mergedPreferences = {
  ...userData.preferences,
  ...newPreferences
};
console.log('Merged preferences:', mergedPreferences);

// 5. Create a function that accepts destructured parameters
const createUserProfile = ({
  personal: { name, age, email },
  preferences: { theme, language },
  address: { city, state }
}) => {
  return {
    profile: `${name} (${age} years old)`,
    contact: `${email} from ${city}, ${state}`,
    settings: `Theme: ${theme}, Language: ${language}`
  };
};

const profile = createUserProfile(userData);
console.log('User Profile:', profile);
```

---

## 🎯 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Let & Const**: Block-scoped variables with proper scoping rules
- ✅ **Arrow Functions**: Modern function syntax with lexical `this`
- ✅ **Template Literals**: String interpolation and multi-line strings
- ✅ **Destructuring**: Object and array destructuring for cleaner code
- ✅ **Spread Operator**: Array and object spreading for immutability
- ✅ **Rest Parameters**: Variable number of function arguments
- ✅ **Enhanced Object Literals**: Shorthand properties and methods
- ✅ **Promises**: Asynchronous programming with promise chains
- ✅ **Async/Await**: Synchronous-style async code
- ✅ **Array Methods**: map, filter, reduce, find, and more
- ✅ **ES6 Modules**: Import/export for modular code organization

### **Best Practices**
1. **Use const by default**, let only when reassignment is needed
2. **Prefer arrow functions** for callbacks and short functions
3. **Use template literals** instead of string concatenation
4. **Leverage destructuring** to extract values cleanly
5. **Use spread operator** for immutable updates
6. **Handle async operations** with async/await for readability
7. **Use array methods** instead of traditional loops when possible
8. **Organize code** with ES6 modules for maintainability

### **Next Steps**
- Practice these concepts in React Native components
- Learn about advanced ES6+ features like generators and proxies
- Study functional programming concepts
- Explore modern JavaScript frameworks and libraries

---

## 📚 **Additional Resources**
- [MDN JavaScript ES6+ Documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- [JavaScript ES6+ Features Overview](https://babeljs.io/docs/en/learn)
- [React Native JavaScript Environment](https://reactnative.dev/docs/javascript-environment)
- [JavaScript Array Methods Guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [Async/Await Best Practices](https://javascript.info/async-await)

---

**Next Lesson**: [Lesson 3: React Native Components & JSX](Lesson%203_%20React%20Native%20Components%20&%20JSX.md)