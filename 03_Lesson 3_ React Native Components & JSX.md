
# 🧩 Lesson 3: React Native Components & JSX

## 🎯 Learning Objectives
Is lesson ke baad aap:
- JSX syntax aur rules samjhenge
- React Native core components use kar sakte hain
- Custom components create kar sakte hain
- Props aur component composition samjhenge
- Component lifecycle basics samjhenge

---

## 📖 What is JSX?

JSX (JavaScript XML) ek syntax extension hai JavaScript ke liye jo HTML-like syntax provide karta hai React components banane ke liye.

### JSX Rules

```javascript
// ✅ Valid JSX
const MyComponent = () => {
  return (
    <View>
      <Text>Hello World!</Text>
    </View>
  );
};

// ❌ Invalid JSX - Multiple root elements
const InvalidComponent = () => {
  return (
    <Text>First element</Text>
    <Text>Second element</Text>  // Error!
  );
};

// ✅ Valid - Single root element
const ValidComponent = () => {
  return (
    <View>
      <Text>First element</Text>
      <Text>Second element</Text>
    </View>
  );
};

// ✅ Valid - React Fragment
const FragmentComponent = () => {
  return (
    <>
      <Text>First element</Text>
      <Text>Second element</Text>
    </>
  );
};
```

### JSX Expressions

```javascript
const UserGreeting = () => {
  const userName = 'John';
  const userAge = 25;
  const isLoggedIn = true;
  const hobbies = ['reading', 'coding', 'gaming'];

  return (
    <View>
      {/* String interpolation */}
      <Text>Hello, {userName}!</Text>
      
      {/* Mathematical expressions */}
      <Text>You are {userAge} years old</Text>
      <Text>Next year you'll be {userAge + 1}</Text>
      
      {/* Conditional rendering */}
      {isLoggedIn ? (
        <Text>Welcome back!</Text>
      ) : (
        <Text>Please log in</Text>
      )}
      
      {/* Logical AND operator */}
      {isLoggedIn && <Text>You have access to premium features</Text>}
      
      {/* Array rendering */}
      {hobbies.map((hobby, index) => (
        <Text key={index}>Hobby: {hobby}</Text>
      ))}
      
      {/* Function calls */}
      <Text>{new Date().toLocaleDateString()}</Text>
    </View>
  );
};
```

---

## 🧱 Core React Native Components

### 1. View Component
```javascript
import React from 'react';
import { View, StyleSheet } from 'react-native';

const ViewExample = () => {
  return (
    <View style={styles.container}>
      <View style={styles.box1}>
        <View style={styles.innerBox} />
      </View>
      <View style={styles.box2} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f0f0f0',
  },
  box1: {
    width: 100,
    height: 100,
    backgroundColor: 'red',
    justifyContent: 'center',
    alignItems: 'center',
    marginBottom: 20,
  },
  innerBox: {
    width: 50,
    height: 50,
    backgroundColor: 'white',
  },
  box2: {
    width: 150,
    height: 80,
    backgroundColor: 'blue',
    borderRadius: 10,
  },
});
```

### 2. Text Component
```javascript
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

const TextExample = () => {
  return (
    <View style={styles.container}>
      {/* Basic text */}
      <Text style={styles.title}>Main Title</Text>
      
      {/* Nested text with different styles */}
      <Text style={styles.paragraph}>
        This is a paragraph with{' '}
        <Text style={styles.bold}>bold text</Text>
        {' '}and{' '}
        <Text style={styles.italic}>italic text</Text>.
      </Text>
      
      {/* Multiline text */}
      <Text style={styles.multiline}>
        This is a very long text that will wrap to multiple lines
        when it exceeds the width of its container.
      </Text>
      
      {/* Text with numberOfLines */}
      <Text style={styles.truncated} numberOfLines={2}>
        This text will be truncated after two lines no matter how long
        the content is. It will show ellipsis at the end.
      </Text>
      
      {/* Selectable text */}
      <Text selectable style={styles.selectable}>
        This text can be selected and copied by the user.
      </Text>
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
    color: '#333',
    marginBottom: 15,
  },
  paragraph: {
    fontSize: 16,
    lineHeight: 24,
    color: '#666',
    marginBottom: 15,
  },
  bold: {
    fontWeight: 'bold',
    color: '#000',
  },
  italic: {
    fontStyle: 'italic',
    color: '#007AFF',
  },
  multiline: {
    fontSize: 14,
    color: '#888',
    marginBottom: 15,
  },
  truncated: {
    fontSize: 14,
    color: '#555',
    marginBottom: 15,
  },
  selectable: {
    fontSize: 16,
    color: '#007AFF',
    backgroundColor: '#f0f8ff',
    padding: 10,
    borderRadius: 5,
  },
});
```

### 3. Image Component
```javascript
import React from 'react';
import { View, Image, StyleSheet } from 'react-native';

const ImageExample = () => {
  return (
    <View style={styles.container}>
      {/* Local image */}
      <Image
        source={require('../assets/logo.png')}
        style={styles.localImage}
      />
      
      {/* Network image */}
      <Image
        source={{
          uri: 'https://picsum.photos/200/200',
        }}
        style={styles.networkImage}
      />
      
      {/* Image with resizeMode */}
      <Image
        source={{
          uri: 'https://picsum.photos/300/200',
        }}
        style={styles.resizedImage}
        resizeMode="cover"
      />
      
      {/* Image with loading indicator */}
      <Image
        source={{
          uri: 'https://picsum.photos/250/150',
        }}
        style={styles.loadingImage}
        loadingIndicatorSource={require('../assets/loading.gif')}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    padding: 20,
  },
  localImage: {
    width: 100,
    height: 100,
    marginBottom: 20,
  },
  networkImage: {
    width: 200,
    height: 200,
    borderRadius: 100,
    marginBottom: 20,
  },
  resizedImage: {
    width: 300,
    height: 150,
    borderRadius: 10,
    marginBottom: 20,
  },
  loadingImage: {
    width: 250,
    height: 150,
    borderRadius: 8,
  },
});
```

### 4. ScrollView Component
```javascript
import React from 'react';
import { ScrollView, View, Text, StyleSheet } from 'react-native';

const ScrollViewExample = () => {
  const items = Array.from({ length: 50 }, (_, i) => `Item ${i + 1}`);

  return (
    <ScrollView
      style={styles.container}
      showsVerticalScrollIndicator={false}
      bounces={true}
      refreshControl={
        <RefreshControl
          refreshing={false}
          onRefresh={() => console.log('Refreshing...')}
        />
      }
    >
      <Text style={styles.title}>Scrollable Content</Text>
      
      {items.map((item, index) => (
        <View key={index} style={styles.item}>
          <Text style={styles.itemText}>{item}</Text>
        </View>
      ))}
      
      {/* Horizontal ScrollView */}
      <Text style={styles.subtitle}>Horizontal Scroll</Text>
      <ScrollView
        horizontal
        showsHorizontalScrollIndicator={false}
        style={styles.horizontalScroll}
      >
        {Array.from({ length: 10 }, (_, i) => (
          <View key={i} style={styles.horizontalItem}>
            <Text style={styles.horizontalItemText}>{i + 1}</Text>
          </View>
        ))}
      </ScrollView>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    margin: 20,
  },
  item: {
    backgroundColor: 'white',
    padding: 20,
    marginHorizontal: 15,
    marginVertical: 5,
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.2,
    shadowRadius: 2,
    elevation: 3,
  },
  itemText: {
    fontSize: 16,
    color: '#333',
  },
  subtitle: {
    fontSize: 18,
    fontWeight: 'bold',
    margin: 20,
  },
  horizontalScroll: {
    paddingLeft: 15,
  },
  horizontalItem: {
    width: 80,
    height: 80,
    backgroundColor: '#007AFF',
    borderRadius: 40,
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: 15,
  },
  horizontalItemText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
});
```

### 5. TouchableOpacity Component
```javascript
import React, { useState } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  Alert,
  StyleSheet
} from 'react-native';

const TouchableExample = () => {
  const [count, setCount] = useState(0);
  const [isPressed, setIsPressed] = useState(false);

  const handlePress = () => {
    setCount(count + 1);
    Alert.alert('Button Pressed!', `Count: ${count + 1}`);
  };

  const handleLongPress = () => {
    Alert.alert('Long Press!', 'You held the button');
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Touchable Components</Text>
      
      {/* Basic TouchableOpacity */}
      <TouchableOpacity
        style={styles.button}
        onPress={handlePress}
        activeOpacity={0.7}
      >
        <Text style={styles.buttonText}>Press Me ({count})</Text>
      </TouchableOpacity>
      
      {/* TouchableOpacity with long press */}
      <TouchableOpacity
        style={styles.longPressButton}
        onPress={() => console.log('Short press')}
        onLongPress={handleLongPress}
        delayLongPress={1000}
      >
        <Text style={styles.buttonText}>Long Press Me</Text>
      </TouchableOpacity>
      
      {/* Dynamic styling based on press state */}
      <TouchableOpacity
        style={[
          styles.dynamicButton,
          isPressed && styles.dynamicButtonPressed
        ]}
        onPressIn={() => setIsPressed(true)}
        onPressOut={() => setIsPressed(false)}
        onPress={() => console.log('Dynamic button pressed')}
      >
        <Text style={styles.buttonText}>
          {isPressed ? 'Pressed!' : 'Dynamic Button'}
        </Text>
      </TouchableOpacity>
      
      {/* Disabled button */}
      <TouchableOpacity
        style={[styles.button, styles.disabledButton]}
        disabled={true}
        onPress={() => console.log('This won\'t work')}
      >
        <Text style={[styles.buttonText, styles.disabledText]}>
          Disabled Button
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
    marginBottom: 30,
  },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 30,
    paddingVertical: 15,
    borderRadius: 25,
    marginBottom: 20,
    minWidth: 200,
    alignItems: 'center',
  },
  longPressButton: {
    backgroundColor: '#FF6B35',
    paddingHorizontal: 30,
    paddingVertical: 15,
    borderRadius: 25,
    marginBottom: 20,
    minWidth: 200,
    alignItems: 'center',
  },
  dynamicButton: {
    backgroundColor: '#28A745',
    paddingHorizontal: 30,
    paddingVertical: 15,
    borderRadius: 25,
    marginBottom: 20,
    minWidth: 200,
    alignItems: 'center',
  },
  dynamicButtonPressed: {
    backgroundColor: '#1E7E34',
    transform: [{ scale: 0.95 }],
  },
  disabledButton: {
    backgroundColor: '#CCCCCC',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  disabledText: {
    color: '#999999',
  },
});
```

---

## 🏗️ Creating Custom Components

### 1. Functional Components

```javascript
// Simple functional component
const Greeting = () => {
  return (
    <View>
      <Text>Hello, World!</Text>
    </View>
  );
};

// Component with props
const UserCard = ({ name, age, email, avatar }) => {
  return (
    <View style={styles.card}>
      <Image source={{ uri: avatar }} style={styles.avatar} />
      <View style={styles.info}>
        <Text style={styles.name}>{name}</Text>
        <Text style={styles.age}>Age: {age}</Text>
        <Text style={styles.email}>{email}</Text>
      </View>
    </View>
  );
};

// Component with default props
const Button = ({ 
  title = 'Click Me', 
  onPress = () => {}, 
  backgroundColor = '#007AFF',
  textColor = 'white',
  disabled = false 
}) => {
  return (
    <TouchableOpacity
      style={[
        styles.button,
        { backgroundColor: disabled ? '#CCCCCC' : backgroundColor }
      ]}
      onPress={onPress}
      disabled={disabled}
    >
      <Text style={[styles.buttonText, { color: textColor }]}>
        {title}
      </Text>
    </TouchableOpacity>
  );
};

// Usage
const App = () => {
  const handleButtonPress = () => {
    Alert.alert('Button Pressed!');
  };

  return (
    <View style={styles.container}>
      <Greeting />
      
      <UserCard
        name="John Doe"
        age={28}
        email="john@example.com"
        avatar="https://picsum.photos/100/100"
      />
      
      <Button
        title="Primary Button"
        onPress={handleButtonPress}
      />
      
      <Button
        title="Secondary Button"
        onPress={handleButtonPress}
        backgroundColor="#6C757D"
      />
      
      <Button
        title="Disabled Button"
        disabled={true}
      />
    </View>
  );
};
```

### 2. Component Composition

```javascript
// Card component
const Card = ({ children, style }) => {
  return (
    <View style={[styles.card, style]}>
      {children}
    </View>
  );
};

// Card Header component
const CardHeader = ({ title, subtitle }) => {
  return (
    <View style={styles.cardHeader}>
      <Text style={styles.cardTitle}>{title}</Text>
      {subtitle && <Text style={styles.cardSubtitle}>{subtitle}</Text>}
    </View>
  );
};

// Card Body component
const CardBody = ({ children }) => {
  return (
    <View style={styles.cardBody}>
      {children}
    </View>
  );
};

// Card Footer component
const CardFooter = ({ children }) => {
  return (
    <View style={styles.cardFooter}>
      {children}
    </View>
  );
};

// Usage - Composed components
const ProductCard = ({ product }) => {
  return (
    <Card style={styles.productCard}>
      <CardHeader
        title={product.name}
        subtitle={`₹${product.price}`}
      />
      <CardBody>
        <Image source={{ uri: product.image }} style={styles.productImage} />
        <Text style={styles.productDescription}>
          {product.description}
        </Text>
      </CardBody>
      <CardFooter>
        <Button
          title="Add to Cart"
          onPress={() => console.log('Added to cart:', product.id)}
        />
      </CardFooter>
    </Card>
  );
};

// App using composed components
const ShoppingApp = () => {
  const products = [
    {
      id: 1,
      name: 'iPhone 13',
      price: 79900,
      image: 'https://picsum.photos/200/150',
      description: 'Latest iPhone with amazing features'
    },
    {
      id: 2,
      name: 'MacBook Pro',
      price: 199900,
      image: 'https://picsum.photos/200/150',
      description: 'Powerful laptop for professionals'
    }
  ];

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Our Products</Text>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </ScrollView>
  );
};
```

### 3. Higher-Order Components (HOC)

```javascript
// HOC for loading state
const withLoading = (WrappedComponent) => {
  return ({ isLoading, ...props }) => {
    if (isLoading) {
      return (
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color="#007AFF" />
          <Text style={styles.loadingText}>Loading...</Text>
        </View>
      );
    }
    
    return <WrappedComponent {...props} />;
  };
};

// HOC for error handling
const withErrorHandling = (WrappedComponent) => {
  return ({ error, onRetry, ...props }) => {
    if (error) {
      return (
        <View style={styles.errorContainer}>
          <Text style={styles.errorText}>Something went wrong!</Text>
          <Text style={styles.errorMessage}>{error.message}</Text>
          <Button title="Retry" onPress={onRetry} />
        </View>
      );
    }
    
    return <WrappedComponent {...props} />;
  };
};

// Base component
const UserList = ({ users }) => {
  return (
    <FlatList
      data={users}
      renderItem={({ item }) => (
        <UserCard
          name={item.name}
          age={item.age}
          email={item.email}
          avatar={item.avatar}
        />
      )}
      keyExtractor={item => item.id.toString()}
    />
  );
};

// Enhanced component with HOCs
const EnhancedUserList = withErrorHandling(withLoading(UserList));

// Usage
const App = () => {
  const [users, setUsers] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchUsers = async () => {
    try {
      setIsLoading(true);
      setError(null);
      const response = await fetch('https://jsonplaceholder.typicode.com/users');
      const userData = await response.json();
      setUsers(userData);
    } catch (err) {
      setError(err);
    } finally {
      setIsLoading(false);
    }
  };

  useEffect(() => {
    fetchUsers();
  }, []);

  return (
    <View style={styles.container}>
      <EnhancedUserList
        users={users}
        isLoading={isLoading}
        error={error}
        onRetry={fetchUsers}
      />
    </View>
  );
};
```

---

## 🔄 Component Lifecycle Basics

### 1. useEffect Hook

```javascript
import React, { useState, useEffect } from 'react';

const LifecycleExample = () => {
  const [count, setCount] = useState(0);
  const [data, setData] = useState(null);

  // Component Did Mount (runs once after first render)
  useEffect(() => {
    console.log('Component mounted');
    
    // Cleanup function (Component Will Unmount)
    return () => {
      console.log('Component will unmount');
    };
  }, []); // Empty dependency array

  // Component Did Update (runs after every render)
  useEffect(() => {
    console.log('Component updated');
  }); // No dependency array

  // Specific state change (runs when count changes)
  useEffect(() => {
    console.log('Count changed:', count);
    
    if (count > 0) {
      document.title = `Count: ${count}`;
    }
  }, [count]); // Dependency array with count

  // Data fetching effect
  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/posts/1');
        const result = await response.json();
        setData(result);
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    };

    fetchData();
  }, []);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Lifecycle Example</Text>
      <Text style={styles.count}>Count: {count}</Text>
      
      <TouchableOpacity
        style={styles.button}
        onPress={() => setCount(count + 1)}
      >
        <Text style={styles.buttonText}>Increment</Text>
      </TouchableOpacity>
      
      {data && (
        <View style={styles.dataContainer}>
          <Text style={styles.dataTitle}>{data.title}</Text>
          <Text style={styles.dataBody}>{data.body}</Text>
        </View>
      )}
    </View>
  );
};
```

---

## 🎯 Complete Example: Contact List App

```javascript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  FlatList,
  TouchableOpacity,
  TextInput,
  Alert,
  StyleSheet,
  SafeAreaView
} from 'react-native';

// Contact Item Component
const ContactItem = ({ contact, onEdit, onDelete }) => {
  return (
    <View style={styles.contactItem}>
      <View style={styles.contactInfo}>
        <Text style={styles.contactName}>{contact.name}</Text>
        <Text style={styles.contactPhone}>{contact.phone}</Text>
        <Text style={styles.contactEmail}>{contact.email}</Text>
      </View>
      <View style={styles.contactActions}>
        <TouchableOpacity
          style={[styles.actionButton, styles.editButton]}
          onPress={() => onEdit(contact)}
        >
          <Text style={styles.actionButtonText}>Edit</Text>
        </TouchableOpacity>
        <TouchableOpacity
          style={[styles.actionButton, styles.deleteButton]}
          onPress={() => onDelete(contact.id)}
        >
          <Text style={styles.actionButtonText}>Delete</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

// Add/Edit Contact Form Component
const ContactForm = ({ contact, onSave, onCancel }) => {
  const [name, setName] = useState(contact?.name || '');
  const [phone, setPhone] = useState(contact?.phone || '');
  const [email, setEmail] = useState(contact?.email || '');

  const handleSave = () => {
    if (!name.trim() || !phone.trim()) {
      Alert.alert('Error', 'Name and phone are required');
      return;
    }

    const contactData = {
      id: contact?.id || Date.now().toString(),
      name: name.trim(),
      phone: phone.trim(),
      email: email.trim()
    };

    onSave(contactData);
  };

  return (
    <View style={styles.formContainer}>
      <Text style={styles.formTitle}>
        {contact ? 'Edit Contact' : 'Add New Contact'}
      </Text>
      
      <TextInput
        style={styles.input}
        placeholder="Name *"
        value={name}
        onChangeText={setName}
      />
      
      <TextInput
        style={styles.input}
        placeholder="Phone *"
        value={phone}
        onChangeText={setPhone}
        keyboardType="phone-pad"
      />
      
      <TextInput
        style={styles.input}
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
      />
      
      <View style={styles.formActions}>
        <TouchableOpacity
          style={[styles.formButton, styles.cancelButton]}
          onPress={onCancel}
        >
          <Text style={styles.formButtonText}>Cancel</Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[styles.formButton, styles.saveButton]}
          onPress={handleSave}
        >
          <Text style={styles.formButtonText}>Save</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

// Main App Component
const ContactListApp = () => {
  const [contacts, setContacts] = useState([]);
  const [showForm, setShowForm] = useState(false);
  const [editingContact, setEditingContact] = useState(null);
  const [searchTerm, setSearchTerm] = useState('');

  // Load initial data
  useEffect(() => {
    const initialContacts = [
      {
        id: '1',
        name: 'John Doe',
        phone: '+91 9876543210',
        email: 'john@example.com'
      },
      {
        id: '2',
        name: 'Jane Smith',
        phone: '+91 8765432109',
        email: 'jane@example.com'
      }
    ];
    setContacts(initialContacts);
  }, []);

  // Filter contacts based on search term
  const filteredContacts = contacts.filter(contact =>
    contact.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
    contact.phone.includes(searchTerm) ||
    contact.email.toLowerCase().includes(searchTerm.toLowerCase())
  );

  const handleAddContact = () => {
    setEditingContact(null);
    setShowForm(true);
  };

  const handleEditContact = (contact) => {
    setEditingContact(contact);
    setShowForm(true);
  };

  const handleSaveContact = (contactData) => {
    if (editingContact) {
      // Update existing contact
      setContacts(prevContacts =>
        prevContacts.map(contact =>
          contact.id === contactData.id ? contactData : contact
        )
      );
    } else {
      // Add new contact
      setContacts(prevContacts => [...prevContacts, contactData]);
    }
    
    setShowForm(false);
    setEditingContact(null);
  };

  const handleDeleteContact = (contactId) => {
    Alert.alert(
      'Delete Contact',
      'Are you sure you want to delete this contact?',
      [
        { text: 'Cancel', style: 'cancel' },
        {
          text: 'Delete',
          style: 'destructive',
          onPress: () => {
            setContacts(prevContacts =>
              prevContacts.filter(contact => contact.id !== contactId)
            );
          }
        }
      ]
    );
  };

  const handleCancelForm = () => {
    setShowForm(false);
    setEditingContact(null);
  };

  if (showForm) {
    return (
      <SafeAreaView style={styles.container}>
        <ContactForm
          contact={editingContact}
          onSave={handleSaveContact}
          onCancel={handleCancelForm}
        />
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>My Contacts</Text>
        <TouchableOpacity
          style={styles.addButton}
          onPress={handleAddContact}
        >
          <Text style={styles.addButtonText}>+ Add</Text>
        </TouchableOpacity>
      </View>

      <TextInput
        style={styles.searchInput}
        placeholder="Search contacts..."
        value={searchTerm}
        onChangeText={setSearchTerm}
      />

      <FlatList
        data={filteredContacts}
        renderItem={({ item }) => (
          <ContactItem
            contact={item}
            onEdit={handleEditContact}
            onDelete={handleDeleteContact}
          />
        )}
        keyExtractor={item => item.id}
        style={styles.contactList}
        showsVerticalScrollIndicator={false}
      />

      {filteredContacts.length === 0 && (
        <View style={styles.emptyContainer}>
          <Text style={styles.emptyText}>
            {searchTerm ? 'No contacts found' : 'No contacts yet'}
          </Text>
        </View>
      )}
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    padding: 20,
    backgroundColor: 'white',
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
  },
  addButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 15,
    paddingVertical: 8,
    borderRadius: 20,
  },
  addButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
  searchInput: {
    backgroundColor: 'white',
    paddingHorizontal: 15,
    paddingVertical: 12,
    margin: 15,
    borderRadius: 10,
    fontSize: 16,
  },
  contactList: {
    flex: 1,
    paddingHorizontal: 15,
  },
  contactItem: {
    backgroundColor: 'white',
    padding: 15,
    marginVertical: 5,
    borderRadius: 10,
    flexDirection: 'row',
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.1,
    shadowRadius: 2,
    elevation: 2,
  },
  contactInfo: {
    flex: 1,
  },
  contactName: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 5,
  },
  contactPhone: {
    fontSize: 16,
    color: '#007AFF',
    marginBottom: 3,
  },
  contactEmail: {
    fontSize: 14,
    color: '#666',
  },
  contactActions: {
    flexDirection: 'row',
  },
  actionButton: {
    paddingHorizontal: 12,
    paddingVertical: 6,
    borderRadius: 15,
    marginLeft: 8,
  },
  editButton: {
    backgroundColor: '#28a745',
  },
  deleteButton: {
    backgroundColor: '#dc3545',
  },
  actionButtonText: {
    color: 'white',
    fontSize: 12,
    fontWeight: 'bold',
  },
  formContainer: {
    flex: 1,
    backgroundColor: '#f5f5f5',
    padding: 20,
  },
  formTitle: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
    color: '#333',
  },
  input: {
    backgroundColor: 'white',
    paddingHorizontal: 15,
    paddingVertical: 12,
    borderRadius: 10,
    fontSize: 16,
    marginBottom: 15,
  },
  formActions: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginTop: 20,
  },
  formButton: {
    flex: 1,
    paddingVertical: 15,
    borderRadius: 10,
    alignItems: 'center',
    marginHorizontal: 5,
  },
  cancelButton: {
    backgroundColor: '#6c757d',
  },
  saveButton: {
    backgroundColor: '#007AFF',
  },
  formButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  emptyContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  emptyText: {
    fontSize: 18,
    color: '#666',
    textAlign: 'center',
  },
});

export default ContactListApp;
```

---

## 📝 **Assignment 3: Component Creation Practice**

### **Task 1: Custom Button Component (30 minutes)**
Create a reusable Button component with the following features:
- Different variants (primary, secondary, danger, success)
- Different sizes (small, medium, large)
- Loading state with spinner
- Disabled state
- Icon support (left and right)
- Custom styling support

### **Task 2: Card Component System (45 minutes)**
Build a flexible Card component system:
- Card container with shadow/elevation
- CardHeader, CardBody, CardFooter sub-components
- Support for different layouts
- Custom styling and theming
- Responsive design considerations

### **Task 3: Form Components (60 minutes)**
Create a comprehensive form component library:
- TextInput with validation
- Select/Dropdown component
- Checkbox and Radio button groups
- Date/Time picker wrapper
- Form validation system
- Error handling and display

---

## 🧠 **Quiz 3: React Native Components & JSX**

### **Multiple Choice Questions**

**Q1. JSX mein single root element ka rule kyun important hai?**
a) Performance ke liye
b) Memory management ke liye
c) React's virtual DOM structure ke liye
d) Styling ke liye

**Q2. TouchableOpacity component ka primary use case kya hai?**
a) Text display
b) Image display
c) Touch interactions
d) Layout containers

**Q3. React Native mein custom components banane ka best practice kya hai?**
a) Class components use karo
b) Functional components with hooks use karo
c) Mix of both approaches
d) Avoid custom components

**Q4. useEffect hook ka purpose kya hai?**
a) State management
b) Side effects handling
c) Component rendering
d) Event handling

**Q5. Component composition ka benefit kya hai?**
a) Performance improvement
b) Code reusability aur maintainability
c) Bundle size reduction
d) Memory optimization

### **True/False Questions**

**Q6. JSX mein JavaScript expressions {} ke andar use kar sakte hain.** (True/False)

**Q7. View component React Native ka fundamental building block hai.** (True/False)

**Q8. Higher-Order Components (HOC) functional programming concept hai.** (True/False)

**Q9. ScrollView automatically provides pull-to-refresh functionality.** (True/False)

**Q10. React Fragments (<>) multiple root elements return karne ke liye use hote hain.** (True/False)

### **Code Output Questions**

**Q11. Is code ka output kya hoga?**
```javascript
const MyComponent = () => {
  const items = ['Apple', 'Banana', 'Orange'];
  return (
    <View>
      {items.map((item, index) => (
        <Text key={index}>{item}</Text>
      ))}
    </View>
  );
};
```

**Q12. Is code mein error kya hai?**
```javascript
const MyComponent = () => {
  return (
    <Text>Hello</Text>
    <Text>World</Text>
  );
};
```

**Q13. Is code ka correct version kya hoga?**
```javascript
const MyComponent = ({ user }) => {
  return (
    <View>
      <Text>Welcome {user.name}</Text>
      <Text>Age: {user.age}</Text>
    </View>
  );
};
```

---

## 📚 **Answers & Explanations**

### **Multiple Choice Answers**
**Q1.** c) React's virtual DOM structure ke liye
**Q2.** c) Touch interactions
**Q3.** b) Functional components with hooks use karo
**Q4.** b) Side effects handling
**Q5.** b) Code reusability aur maintainability

### **True/False Answers**
**Q6.** True
**Q7.** True
**Q8.** True
**Q9.** False - Manual implementation required
**Q10.** True

### **Code Output Answers**
**Q11.** Three Text components with 'Apple', 'Banana', 'Orange'
**Q12.** Multiple root elements without wrapper
**Q13.** Props destructuring: `const MyComponent = ({ user }) => { ... }`

---

## 🎯 **Assignment Solutions**

### **Task 1: Custom Button Component Solution**

```javascript
import React from 'react';
import { TouchableOpacity, Text, ActivityIndicator, View, StyleSheet } from 'react-native';

const CustomButton = ({
  title,
  onPress,
  variant = 'primary',
  size = 'medium',
  loading = false,
  disabled = false,
  leftIcon,
  rightIcon,
  style,
  textStyle,
  ...props
}) => {
  const getVariantStyles = () => {
    const variants = {
      primary: {
        backgroundColor: '#007AFF',
        textColor: 'white',
      },
      secondary: {
        backgroundColor: '#6c757d',
        textColor: 'white',
      },
      danger: {
        backgroundColor: '#dc3545',
        textColor: 'white',
      },
      success: {
        backgroundColor: '#28a745',
        textColor: 'white',
      },
      outline: {
        backgroundColor: 'transparent',
        borderColor: '#007AFF',
        borderWidth: 1,
        textColor: '#007AFF',
      },
    };
    return variants[variant] || variants.primary;
  };

  const getSizeStyles = () => {
    const sizes = {
      small: {
        paddingHorizontal: 12,
        paddingVertical: 8,
        fontSize: 14,
      },
      medium: {
        paddingHorizontal: 16,
        paddingVertical: 12,
        fontSize: 16,
      },
      large: {
        paddingHorizontal: 24,
        paddingVertical: 16,
        fontSize: 18,
      },
    };
    return sizes[size] || sizes.medium;
  };

  const variantStyles = getVariantStyles();
  const sizeStyles = getSizeStyles();

  const isDisabled = disabled || loading;

  return (
    <TouchableOpacity
      style={[
        styles.button,
        {
          backgroundColor: isDisabled ? '#cccccc' : variantStyles.backgroundColor,
          borderColor: variantStyles.borderColor,
          borderWidth: variantStyles.borderWidth,
          ...sizeStyles,
        },
        style,
      ]}
      onPress={isDisabled ? undefined : onPress}
      disabled={isDisabled}
      activeOpacity={0.8}
      {...props}
    >
      <View style={styles.content}>
        {leftIcon && !loading && (
          <View style={styles.leftIcon}>
            {leftIcon}
          </View>
        )}

        {loading ? (
          <ActivityIndicator
            size="small"
            color={variantStyles.textColor}
          />
        ) : (
          <Text
            style={[
              styles.text,
              {
                color: isDisabled ? '#999999' : variantStyles.textColor,
                fontSize: sizeStyles.fontSize,
              },
              textStyle,
            ]}
          >
            {title}
          </Text>
        )}

        {rightIcon && !loading && (
          <View style={styles.rightIcon}>
            {rightIcon}
          </View>
        )}
      </View>
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  button: {
    borderRadius: 8,
    alignItems: 'center',
    justifyContent: 'center',
    flexDirection: 'row',
  },
  content: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
  },
  text: {
    fontWeight: '600',
    textAlign: 'center',
  },
  leftIcon: {
    marginRight: 8,
  },
  rightIcon: {
    marginLeft: 8,
  },
});

export default CustomButton;
```

### **Task 2: Card Component System Solution**

```javascript
import React from 'react';
import { View, Text, StyleSheet, Platform } from 'react-native';

const Card = ({
  children,
  style,
  shadow = true,
  borderRadius = 12,
  padding = 16,
  margin = 8,
  backgroundColor = 'white',
  ...props
}) => {
  const cardStyles = [
    styles.card,
    {
      borderRadius,
      padding,
      margin,
      backgroundColor,
    },
    shadow && styles.shadow,
    style,
  ];

  return (
    <View style={cardStyles} {...props}>
      {children}
    </View>
  );
};

const CardHeader = ({
  title,
  subtitle,
  leftComponent,
  rightComponent,
  style,
  titleStyle,
  subtitleStyle,
  ...props
}) => {
  return (
    <View style={[styles.header, style]} {...props}>
      {leftComponent && (
        <View style={styles.leftComponent}>
          {leftComponent}
        </View>
      )}

      <View style={styles.headerContent}>
        {title && (
          <Text style={[styles.headerTitle, titleStyle]}>
            {title}
          </Text>
        )}
        {subtitle && (
          <Text style={[styles.headerSubtitle, subtitleStyle]}>
            {subtitle}
          </Text>
        )}
      </View>

      {rightComponent && (
        <View style={styles.rightComponent}>
          {rightComponent}
        </View>
      )}
    </View>
  );
};

const CardBody = ({
  children,
  style,
  scrollable = false,
  ...props
}) => {
  const bodyStyles = [
    styles.body,
    style,
  ];

  if (scrollable) {
    return (
      <ScrollView style={bodyStyles} {...props}>
        {children}
      </ScrollView>
    );
  }

  return (
    <View style={bodyStyles} {...props}>
      {children}
    </View>
  );
};

const CardFooter = ({
  children,
  style,
  align = 'center',
  ...props
}) => {
  const footerStyles = [
    styles.footer,
    styles[`footer${align.charAt(0).toUpperCase() + align.slice(1)}`],
    style,
  ];

  return (
    <View style={footerStyles} {...props}>
      {children}
    </View>
  );
};

// Usage example
const ProductCard = ({ product }) => {
  return (
    <Card>
      <CardHeader
        title={product.name}
        subtitle={`$${product.price}`}
        rightComponent={
          <TouchableOpacity>
            <Text>❤️</Text>
          </TouchableOpacity>
        }
      />

      <CardBody>
        <Image
          source={{ uri: product.image }}
          style={styles.productImage}
        />
        <Text style={styles.productDescription}>
          {product.description}
        </Text>
      </CardBody>

      <CardFooter align="space-between">
        <CustomButton
          title="Add to Cart"
          variant="primary"
          size="small"
        />
        <CustomButton
          title="Buy Now"
          variant="success"
          size="small"
        />
      </CardFooter>
    </Card>
  );
};

const styles = StyleSheet.create({
  card: {
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.1,
        shadowRadius: 4,
      },
      android: {
        elevation: 4,
      },
    }),
  },
  shadow: Platform.select({
    ios: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 2 },
      shadowOpacity: 0.1,
      shadowRadius: 4,
    },
    android: {
      elevation: 4,
    },
  }),
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 12,
  },
  leftComponent: {
    marginRight: 12,
  },
  headerContent: {
    flex: 1,
  },
  headerTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
  },
  headerSubtitle: {
    fontSize: 14,
    color: '#666',
    marginTop: 2,
  },
  rightComponent: {
    marginLeft: 12,
  },
  body: {
    flex: 1,
  },
  footer: {
    marginTop: 12,
    paddingTop: 12,
    borderTopWidth: 1,
    borderTopColor: '#e0e0e0',
  },
  footerCenter: {
    alignItems: 'center',
  },
  footerLeft: {
    alignItems: 'flex-start',
  },
  footerRight: {
    alignItems: 'flex-end',
  },
  footerSpaceBetween: {
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
  productImage: {
    width: '100%',
    height: 200,
    borderRadius: 8,
    marginBottom: 12,
  },
  productDescription: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
  },
});

export { Card, CardHeader, CardBody, CardFooter };
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **JSX Syntax**: Rules, expressions, and best practices
- ✅ **Core Components**: View, Text, Image, ScrollView, TouchableOpacity
- ✅ **Custom Components**: Functional components with props and state
- ✅ **Component Composition**: Building complex UIs from smaller components
- ✅ **Higher-Order Components**: Reusable component logic patterns
- ✅ **Component Lifecycle**: useEffect hook for side effects
- ✅ **Complete App Example**: Contact list app with full CRUD functionality

### **Best Practices**
1. **Always use a single root element** in JSX return statements
2. **Use meaningful component names** that describe their purpose
3. **Pass data through props** rather than hardcoding values
4. **Use destructuring** for cleaner prop handling
5. **Implement proper key props** in lists for performance
6. **Handle loading and error states** in your components
7. **Use StyleSheet.create** for consistent styling
8. **Keep components small and focused** on single responsibilities

### **Next Steps**
- Learn about React Navigation for multi-screen apps
- Study state management with Context API and Redux
- Explore advanced component patterns and hooks
- Practice building more complex component systems

---

## 📚 **Additional Resources**
- [React Native Components Documentation](https://reactnative.dev/docs/components-and-apis)
- [JSX in Depth](https://reactjs.org/docs/jsx-in-depth.html)
- [React Component Patterns](https://www.patterns.dev/posts/react-component-patterns)
- [React Native Styling Guide](https://reactnative.dev/docs/style)
- [Component Lifecycle Methods](https://reactjs.org/docs/state-and-lifecycle.html)

---

**Next Lesson**: [Lesson 4: Styling in React Native (Flexbox & StyleSheet)](Lesson%204_%20Styling%20in%20React%20Native%20(Flexbox%20&%20StyleSheet).md)