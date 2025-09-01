---

## 📝 **Assignment 5: State Management Practice**

### **Task 1: Shopping Cart Application (60 minutes)**
Create a shopping cart with:
- Product listing with add to cart functionality
- Cart management (add, remove, update quantities)
- Total calculation with tax
- Persistent cart state using AsyncStorage
- Real-time cart updates

### **Task 2: User Authentication Flow (45 minutes)**
Build a login/signup system with:
- Form validation with custom hooks
- Authentication state management
- Protected routes simulation
- Remember me functionality
- Logout with confirmation

### **Task 3: Real-time Chat Application (60 minutes)**
Implement a chat app with:
- Message state management
- Real-time message updates (simulated)
- Message persistence
- Typing indicators
- Message status (sent, delivered, read)

---

## 🧠 **Quiz 5: State Management with Hooks**

### **Multiple Choice Questions**

**Q1. useState hook ka purpose kya hai?**
a) Side effects handle karna
b) Component state manage karna
c) API calls karna
d) Routing handle karna

**Q2. useEffect hook kab run hota hai?**
a) Sirf component mount pe
b) Sirf component unmount pe
c) Dependencies change hone pe
d) Kabhi nahi

**Q3. Functional updates ka use kab karte hain?**
a) Jab state simple ho
b) Jab state complex ho aur previous state ki zaroorat ho
c) Jab state change nahi karna ho
d) Jab component unmount ho raha ho

**Q4. useEffect cleanup function kya karta hai?**
a) Component ko destroy karta hai
b) Memory leaks prevent karta hai
c) State ko reset karta hai
d) Component ko re-render karta hai

**Q5. Custom hooks ka benefit kya hai?**
a) Code reusability badhati hain
b) Performance kam karte hain
c) Bundle size badhate hain
d) Complexity badhate hain

### **True/False Questions**

**Q6. useState asynchronous hai.** (True/False)

**Q7. useEffect dependencies array optional hai.** (True/False)

**Q8. Functional updates stale closures se bachate hain.** (True/False)

**Q9. useEffect cleanup function return statement ke baad execute hoti hai.** (True/False)

**Q10. Custom hooks 'use' se start hone chahiye.** (True/False)

### **Code Output Questions**

**Q11. Is code ka output kya hoga?**
```javascript
const [count, setCount] = useState(0);

useEffect(() => {
  console.log('Count changed:', count);
}, [count]);

setCount(1);
setCount(2);
setCount(3);
```

**Q12. Is useEffect kab run hoga?**
```javascript
useEffect(() => {
  console.log('Effect ran');
}, []);
```

**Q13. Is code mein problem kya hai?**
```javascript
const [count, setCount] = useState(0);

const increment = () => {
  setTimeout(() => {
    setCount(count + 1);
  }, 1000);
};
```

---

## 📚 **Answers & Explanations**

### **Multiple Choice Answers**
**Q1.** b) Component state manage karna  
**Q2.** c) Dependencies change hone pe  
**Q3.** b) Jab state complex ho aur previous state ki zaroorat ho  
**Q4.** b) Memory leaks prevent karta hai  
**Q5.** a) Code reusability badhati hain  

### **True/False Answers**
**Q6.** True  
**Q7.** True  
**Q8.** True  
**Q9.** False - Component unmount hone pe execute hoti hai  
**Q10.** True  

### **Code Output Answers**
**Q11.** 'Count changed: 1', 'Count changed: 2', 'Count changed: 3'  
**Q12.** Sirf component mount hone pe ek baar  
**Q13.** Stale closure - count ki purani value use hogi  

---

## 🎯 **Assignment Solutions**

### **Task 1: Shopping Cart Application Solution**

```javascript
import React, { useState, useEffect, useCallback } from 'react';
import {
  View,
  Text,
  FlatList,
  TouchableOpacity,
  StyleSheet,
  SafeAreaView,
  Alert
} from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

const ShoppingCartApp = () => {
  const [products] = useState([
    { id: 1, name: 'iPhone 13', price: 79900, image: '📱' },
    { id: 2, name: 'MacBook Pro', price: 199900, image: '💻' },
    { id: 3, name: 'AirPods', price: 24900, image: '🎧' },
    { id: 4, name: 'iPad', price: 49900, image: '📱' },
  ]);

  const [cart, setCart] = useState([]);

  // Load cart from storage
  useEffect(() => {
    const loadCart = async () => {
      try {
        const storedCart = await AsyncStorage.getItem('cart');
        if (storedCart) {
          setCart(JSON.parse(storedCart));
        }
      } catch (error) {
        console.error('Error loading cart:', error);
      }
    };
    loadCart();
  }, []);

  // Save cart to storage
  useEffect(() => {
    const saveCart = async () => {
      try {
        await AsyncStorage.setItem('cart', JSON.stringify(cart));
      } catch (error) {
        console.error('Error saving cart:', error);
      }
    };
    saveCart();
  }, [cart]);

  // Add item to cart
  const addToCart = useCallback((product) => {
    setCart(prevCart => {
      const existingItem = prevCart.find(item => item.id === product.id);
      if (existingItem) {
        return prevCart.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      } else {
        return [...prevCart, { ...product, quantity: 1 }];
      }
    });
  }, []);

  // Remove item from cart
  const removeFromCart = useCallback((productId) => {
    setCart(prevCart => prevCart.filter(item => item.id !== productId));
  }, []);

  // Update item quantity
  const updateQuantity = useCallback((productId, newQuantity) => {
    if (newQuantity <= 0) {
      removeFromCart(productId);
      return;
    }

    setCart(prevCart =>
      prevCart.map(item =>
        item.id === productId
          ? { ...item, quantity: newQuantity }
          : item
      )
    );
  }, [removeFromCart]);

  // Calculate totals
  const cartTotal = cart.reduce((total, item) => total + (item.price * item.quantity), 0);
  const tax = cartTotal * 0.18; // 18% GST
  const finalTotal = cartTotal + tax;

  // Render product item
  const renderProduct = ({ item }) => (
    <View style={styles.productItem}>
      <Text style={styles.productImage}>{item.image}</Text>
      <View style={styles.productInfo}>
        <Text style={styles.productName}>{item.name}</Text>
        <Text style={styles.productPrice}>₹{item.price.toLocaleString()}</Text>
      </View>
      <TouchableOpacity
        style={styles.addButton}
        onPress={() => addToCart(item)}
      >
        <Text style={styles.addButtonText}>Add to Cart</Text>
      </TouchableOpacity>
    </View>
  );

  // Render cart item
  const renderCartItem = ({ item }) => (
    <View style={styles.cartItem}>
      <Text style={styles.cartItemImage}>{item.image}</Text>
      <View style={styles.cartItemInfo}>
        <Text style={styles.cartItemName}>{item.name}</Text>
        <Text style={styles.cartItemPrice}>₹{item.price.toLocaleString()}</Text>
        <View style={styles.quantityContainer}>
          <TouchableOpacity
            style={styles.quantityButton}
            onPress={() => updateQuantity(item.id, item.quantity - 1)}
          >
            <Text style={styles.quantityButtonText}>-</Text>
          </TouchableOpacity>
          <Text style={styles.quantityText}>{item.quantity}</Text>
          <TouchableOpacity
            style={styles.quantityButton}
            onPress={() => updateQuantity(item.id, item.quantity + 1)}
          >
            <Text style={styles.quantityButtonText}>+</Text>
          </TouchableOpacity>
        </View>
      </View>
      <TouchableOpacity
        style={styles.removeButton}
        onPress={() => removeFromCart(item.id)}
      >
        <Text style={styles.removeButtonText}>×</Text>
      </TouchableOpacity>
    </View>
  );

  return (
    <SafeAreaView style={styles.container}>
      <Text style={styles.title}>Shopping Cart</Text>

      <View style={styles.content}>
        {/* Products List */}
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Products</Text>
          <FlatList
            data={products}
            renderItem={renderProduct}
            keyExtractor={item => item.id.toString()}
            style={styles.productsList}
            showsVerticalScrollIndicator={false}
          />
        </View>

        {/* Cart */}
        <View style={styles.section}>
          <View style={styles.cartHeader}>
            <Text style={styles.sectionTitle}>Cart ({cart.length} items)</Text>
            {cart.length > 0 && (
              <TouchableOpacity style={styles.clearButton} onPress={() => setCart([])}>
                <Text style={styles.clearButtonText}>Clear All</Text>
              </TouchableOpacity>
            )}
          </View>

          {cart.length === 0 ? (
            <Text style={styles.emptyCart}>Your cart is empty</Text>
          ) : (
            <>
              <FlatList
                data={cart}
                renderItem={renderCartItem}
                keyExtractor={item => item.id.toString()}
                style={styles.cartList}
                showsVerticalScrollIndicator={false}
              />

              {/* Cart Summary */}
              <View style={styles.cartSummary}>
                <View style={styles.summaryRow}>
                  <Text style={styles.summaryLabel}>Subtotal:</Text>
                  <Text style={styles.summaryValue}>₹{cartTotal.toLocaleString()}</Text>
                </View>
                <View style={styles.summaryRow}>
                  <Text style={styles.summaryLabel}>Tax (18%):</Text>
                  <Text style={styles.summaryValue}>₹{tax.toLocaleString()}</Text>
                </View>
                <View style={[styles.summaryRow, styles.totalRow]}>
                  <Text style={styles.totalLabel}>Total:</Text>
                  <Text style={styles.totalValue}>₹{finalTotal.toLocaleString()}</Text>
                </View>
              </View>

              <TouchableOpacity style={styles.checkoutButton}>
                <Text style={styles.checkoutButtonText}>Proceed to Checkout</Text>
              </TouchableOpacity>
            </>
          )}
        </View>
      </View>
    </SafeAreaView>
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
    marginVertical: 20,
    color: '#333',
  },
  content: {
    flex: 1,
  },
  section: {
    flex: 1,
    margin: 15,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  productsList: {
    flex: 1,
  },
  productItem: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 10,
    marginBottom: 10,
    flexDirection: 'row',
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.1,
    shadowRadius: 2,
    elevation: 2,
  },
  productImage: {
    fontSize: 30,
    marginRight: 15,
  },
  productInfo: {
    flex: 1,
  },
  productName: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 5,
  },
  productPrice: {
    fontSize: 14,
    color: '#007AFF',
    fontWeight: 'bold',
  },
  addButton: {
    backgroundColor: '#28A745',
    paddingHorizontal: 15,
    paddingVertical: 8,
    borderRadius: 20,
  },
  addButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
  cartHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 15,
  },
  clearButton: {
    backgroundColor: '#FF3B30',
    paddingHorizontal: 12,
    paddingVertical: 6,
    borderRadius: 15,
  },
  clearButtonText: {
    color: 'white',
    fontSize: 12,
    fontWeight: 'bold',
  },
  emptyCart: {
    textAlign: 'center',
    fontSize: 16,
    color: '#666',
    marginTop: 50,
  },
  cartList: {
    flex: 1,
  },
  cartItem: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 10,
    marginBottom: 10,
    flexDirection: 'row',
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.1,
    shadowRadius: 2,
    elevation: 2,
  },
  cartItemImage: {
    fontSize: 24,
    marginRight: 15,
  },
  cartItemInfo: {
    flex: 1,
  },
  cartItemName: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 5,
  },
  cartItemPrice: {
    fontSize: 14,
    color: '#007AFF',
    fontWeight: 'bold',
    marginBottom: 10,
  },
  quantityContainer: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  quantityButton: {
    backgroundColor: '#007AFF',
    width: 30,
    height: 30,
    borderRadius: 15,
    justifyContent: 'center',
    alignItems: 'center',
  },
  quantityButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  quantityText: {
    fontSize: 16,
    fontWeight: 'bold',
    marginHorizontal: 15,
    minWidth: 30,
    textAlign: 'center',
  },
  removeButton: {
    backgroundColor: '#FF3B30',
    width: 30,
    height: 30,
    borderRadius: 15,
    justifyContent: 'center',
    alignItems: 'center',
  },
  removeButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  cartSummary: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 10,
    marginTop: 10,
  },
  summaryRow: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    paddingVertical: 5,
  },
  summaryLabel: {
    fontSize: 16,
    color: '#666',
  },
  summaryValue: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
  },
  totalRow: {
    borderTopWidth: 1,
    borderTopColor: '#eee',
    paddingTop: 10,
    marginTop: 5,
  },
  totalLabel: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
  },
  totalValue: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#007AFF',
  },
  checkoutButton: {
    backgroundColor: '#007AFF',
    marginTop: 15,
    paddingVertical: 15,
    borderRadius: 10,
    alignItems: 'center',
  },
  checkoutButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default ShoppingCartApp;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **useState Hook**: Basic state management with primitives, objects, and arrays
- ✅ **useEffect Hook**: Side effects, lifecycle management, and cleanup functions
- ✅ **Functional Updates**: Preventing stale closures with updater functions
- ✅ **Complex State**: Managing nested objects and arrays immutably
- ✅ **Data Fetching**: API calls with loading, error, and success states
- ✅ **Custom Hooks**: Reusable logic extraction for better code organization
- ✅ **Performance Optimization**: useCallback and useMemo for expensive operations
- ✅ **AsyncStorage**: Persistent local storage for React Native apps

### **Best Practices**
1. **Always use functional updates** when new state depends on previous state
2. **Include dependency arrays** in useEffect to prevent unnecessary re-runs
3. **Clean up effects** to prevent memory leaks (timers, subscriptions, etc.)
4. **Use useCallback** for functions passed to child components
5. **Store primitive values** directly, use objects for related data
6. **Handle loading and error states** in data fetching operations
7. **Use AsyncStorage** for simple key-value storage, SQLite for complex data
8. **Extract custom hooks** for reusable stateful logic

### **Next Steps**
- Learn about Context API for global state management
- Study Redux and Redux Toolkit for complex applications
- Explore React Navigation for multi-screen apps
- Practice building real-world applications with proper state management

---

## 📚 **Additional Resources**
- [React Hooks Documentation](https://reactjs.org/docs/hooks-intro.html)
- [useState Hook Guide](https://reactjs.org/docs/hooks-state.html)
- [useEffect Hook Guide](https://reactjs.org/docs/hooks-effect.html)
- [AsyncStorage Documentation](https://react-native-async-storage.github.io/async-storage/)
- [React Native State Management](https://reactnative.dev/docs/state)

---

**Next Lesson**: [Lesson 6: Handling User Input & Forms](Lesson%206_%20Handling%20User%20Input%20&%20Forms.md)