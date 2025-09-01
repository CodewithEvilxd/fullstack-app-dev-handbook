esson 7_ Lists, ScrollViews & FlatList.md</path>
<content"> 1000));
      const nextPage = page + 1;
      const newPosts = generatePosts(nextPage);
      setPosts(prevPosts => [...prevPosts, ...newPosts]);
      setPage(nextPage);
    } catch (error) {
      console.error('Error loading more posts:', error);
    } finally {
      setLoadingMore(false);
    }
  };

  const toggleLike = useCallback((postId) => {
    setPosts(prevPosts =>
      prevPosts.map(post =>
        post.id === postId
          ? {
              ...post,
              liked: !post.liked,
              likes: post.liked ? post.likes - 1 : post.likes + 1
            }
          : post
      )
    );
  }, []);

  const toggleBookmark = useCallback((postId) => {
    setPosts(prevPosts =>
      prevPosts.map(post =>
        post.id === postId
          ? { ...post, bookmarked: !post.bookmarked }
          : post
      )
    );
  }, []);

  const renderPost = ({ item }) => (
    <View style={styles.post}>
      {/* Post Header */}
      <View style={styles.postHeader}>
        <Image source={{ uri: item.user.avatar }} style={styles.avatar} />
        <View style={styles.userInfo}>
          <Text style={styles.userName}>{item.user.name}</Text>
          <Text style={styles.username}>{item.user.username}</Text>
        </View>
        <TouchableOpacity style={styles.moreButton}>
          <Text style={styles.moreText}>⋯</Text>
        </TouchableOpacity>
      </View>

      {/* Post Content */}
      <Text style={styles.postContent}>{item.content}</Text>

      {/* Post Image */}
      {item.image && (
        <Image source={{ uri: item.image }} style={styles.postImage} />
      )}

      {/* Post Actions */}
      <View style={styles.postActions}>
        <TouchableOpacity
          style={styles.actionButton}
          onPress={() => toggleLike(item.id)}
        >
          <Text style={[styles.actionText, item.liked && styles.likedText]}>
            {item.liked ? '❤️' : '🤍'} {item.likes}
          </Text>
        </TouchableOpacity>

        <TouchableOpacity style={styles.actionButton}>
          <Text style={styles.actionText}>💬 {item.comments}</Text>
        </TouchableOpacity>

        <TouchableOpacity style={styles.actionButton}>
          <Text style={styles.actionText}>🔄 {item.shares}</Text>
        </TouchableOpacity>

        <TouchableOpacity
          style={styles.actionButton}
          onPress={() => toggleBookmark(item.id)}
        >
          <Text style={[styles.actionText, item.bookmarked && styles.bookmarkedText]}>
            {item.bookmarked ? '🔖' : '📖'}
          </Text>
        </TouchableOpacity>
      </View>

      {/* Post Timestamp */}
      <Text style={styles.timestamp}>{item.timestamp}</Text>
    </View>
  );

  const renderFooter = () => {
    if (!loadingMore) return null;

    return (
      <View style={styles.footer}>
        <ActivityIndicator size="small" color="#007AFF" />
        <Text style={styles.footerText}>Loading more posts...</Text>
      </View>
    );
  };

  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" color="#007AFF" />
        <Text style={styles.loadingText}>Loading feed...</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Social Feed</Text>

      <FlatList
        data={posts}
        renderItem={renderPost}
        keyExtractor={item => item.id}
        refreshControl={
          <RefreshControl
            refreshing={refreshing}
            onRefresh={onRefresh}
            colors={['#007AFF']}
          />
        }
        onEndReached={loadMorePosts}
        onEndReachedThreshold={0.5}
        ListFooterComponent={renderFooter}
        showsVerticalScrollIndicator={false}
        contentContainerStyle={styles.feedContent}
        initialNumToRender={5}
        maxToRenderPerBatch={3}
        windowSize={10}
        removeClippedSubviews={true}
      />
    </View>
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
    padding: 20,
    backgroundColor: 'white',
    color: '#333',
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
  feedContent: {
    padding: 15,
  },
  post: {
    backgroundColor: 'white',
    borderRadius: 10,
    padding: 15,
    marginBottom: 15,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  postHeader: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 10,
  },
  avatar: {
    width: 40,
    height: 40,
    borderRadius: 20,
    marginRight: 10,
  },
  userInfo: {
    flex: 1,
  },
  userName: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
  },
  username: {
    fontSize: 14,
    color: '#666',
  },
  moreButton: {
    padding: 5,
  },
  moreText: {
    fontSize: 20,
    color: '#666',
  },
  postContent: {
    fontSize: 16,
    color: '#333',
    marginBottom: 10,
    lineHeight: 22,
  },
  postImage: {
    width: '100%',
    height: 200,
    borderRadius: 8,
    marginBottom: 10,
  },
  postActions: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 10,
  },
  actionButton: {
    paddingVertical: 5,
    paddingHorizontal: 10,
  },
  actionText: {
    fontSize: 14,
    color: '#666',
  },
  likedText: {
    color: '#FF3B30',
  },
  bookmarkedText: {
    color: '#007AFF',
  },
  timestamp: {
    fontSize: 12,
    color: '#999',
  },
  footer: {
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
    paddingVertical: 20,
  },
  footerText: {
    marginLeft: 10,
    fontSize: 14,
    color: '#666',
  },
});

export default SocialFeed;
```

### 2. E-commerce Product List

```javascript
import React, { useState, useEffect } from 'react';
import {
  View,
  FlatList,
  Text,
  Image,
  TouchableOpacity,
  StyleSheet,
  TextInput,
  ActivityIndicator
} from 'react-native';

const ProductList = () => {
  const [products, setProducts] = useState([]);
  const [filteredProducts, setFilteredProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [searchQuery, setSearchQuery] = useState('');
  const [selectedCategory, setSelectedCategory] = useState('All');

  const categories = ['All', 'Electronics', 'Clothing', 'Books', 'Home', 'Sports'];

  // Generate sample products
  const generateProducts = () => {
    return Array.from({ length: 50 }, (_, i) => ({
      id: `${i + 1}`,
      name: [
        'iPhone 13', 'MacBook Pro', 'AirPods', 'iPad', 'Apple Watch',
        'Nike Shoes', 'Adidas T-Shirt', 'Levi\'s Jeans', 'Samsung TV',
        'Sony Headphones', 'Dell Laptop', 'HP Printer', 'Canon Camera',
        'KitchenAid Mixer', 'Dyson Vacuum', 'Roomba', 'Instant Pot',
        'Nintendo Switch', 'Xbox Controller', 'PlayStation 5'
      ][i % 20],
      price: Math.floor(Math.random() * 50000) + 1000,
      originalPrice: Math.floor(Math.random() * 60000) + 2000,
      rating: (Math.random() * 2 + 3).toFixed(1), // 3.0 to 5.0
      reviews: Math.floor(Math.random() * 1000),
      category: categories[Math.floor(Math.random() * (categories.length - 1)) + 1],
      image: `https://picsum.photos/150/150?random=${i + 1}`,
      inStock: Math.random() > 0.2,
      discount: Math.random() > 0.7 ? Math.floor(Math.random() * 30) + 10 : 0,
    }));
  };

  useEffect(() => {
    loadProducts();
  }, []);

  useEffect(() => {
    filterProducts();
  }, [searchQuery, selectedCategory, products]);

  const loadProducts = async () => {
    setLoading(true);
    try {
      await new Promise(resolve => setTimeout(resolve, 1000));
      const productData = generateProducts();
      setProducts(productData);
    } catch (error) {
      console.error('Error loading products:', error);
    } finally {
      setLoading(false);
    }
  };

  const filterProducts = () => {
    if (!searchQuery.trim()) {
      setFilteredProducts(products);
    } else {
      const filtered = products.filter(product =>
        product.name.toLowerCase().includes(searchQuery.toLowerCase())
      );
      setFilteredProducts(filtered);
    }
  };

  const renderProduct = ({ item }) => (
    <TouchableOpacity style={styles.productCard}>
      <Image source={{ uri: item.image }} style={styles.productImage} />

      {item.discount > 0 && (
        <View style={styles.discountBadge}>
          <Text style={styles.discountText}>{item.discount}% OFF</Text>
        </View>
      )}

      <View style={styles.productInfo}>
        <Text style={styles.productName} numberOfLines={2}>
          {item.name}
        </Text>

        <View style={styles.priceContainer}>
          <Text style={styles.currentPrice}>₹{item.price.toLocaleString()}</Text>
          {item.originalPrice > item.price && (
            <Text style={styles.originalPrice}>
              ₹{item.originalPrice.toLocaleString()}
            </Text>
          )}
        </View>

        <View style={styles.ratingContainer}>
          <Text style={styles.rating}>⭐ {item.rating}</Text>
          <Text style={styles.reviews}>({item.reviews})</Text>
        </View>

        <View style={styles.stockContainer}>
          <Text style={[
            styles.stockText,
            { color: item.inStock ? '#28A745' : '#DC3545' }
          ]}>
            {item.inStock ? 'In Stock' : 'Out of Stock'}
          </Text>
        </View>
      </View>
    </TouchableOpacity>
  );

  const renderCategory = ({ item }) => (
    <TouchableOpacity
      style={[
        styles.categoryButton,
        selectedCategory === item && styles.selectedCategory
      ]}
      onPress={() => setSelectedCategory(item)}
    >
      <Text style={[
        styles.categoryText,
        selectedCategory === item && styles.selectedCategoryText
      ]}>
        {item}
      </Text>
    </TouchableOpacity>
  );

  if (loading) {
    return (
      <View style={styles.loadingContainer}>
        <ActivityIndicator size="large" color="#007AFF" />
        <Text style={styles.loadingText}>Loading products...</Text>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>E-commerce Store</Text>

      {/* Search Bar */}
      <View style={styles.searchContainer}>
        <TextInput
          style={styles.searchInput}
          placeholder="Search products..."
          value={searchQuery}
          onChangeText={setSearchQuery}
          autoCapitalize="none"
        />
      </View>

      {/* Categories */}
      <FlatList
        data={categories}
        renderItem={renderCategory}
        keyExtractor={item => item}
        horizontal
        showsHorizontalScrollIndicator={false}
        contentContainerStyle={styles.categoriesContainer}
      />

      {/* Products Grid */}
      <FlatList
        data={filteredProducts}
        renderItem={renderProduct}
        keyExtractor={item => item.id}
        numColumns={2}
        showsVerticalScrollIndicator={false}
        contentContainerStyle={styles.productsContainer}
        columnWrapperStyle={styles.productRow}
        ListEmptyComponent={
          <View style={styles.emptyContainer}>
            <Text style={styles.emptyText}>No products found</Text>
          </View>
        }
      />
    </View>
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
    padding: 20,
    backgroundColor: 'white',
    color: '#333',
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
  searchContainer: {
    backgroundColor: 'white',
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  searchInput: {
    backgroundColor: '#f8f9fa',
    borderRadius: 8,
    paddingHorizontal: 15,
    paddingVertical: 10,
    fontSize: 16,
  },
  categoriesContainer: {
    paddingHorizontal: 15,
    paddingVertical: 10,
  },
  categoryButton: {
    backgroundColor: '#f8f9fa',
    paddingHorizontal: 15,
    paddingVertical: 8,
    borderRadius: 20,
    marginRight: 10,
  },
  selectedCategory: {
    backgroundColor: '#007AFF',
  },
  categoryText: {
    fontSize: 14,
    color: '#666',
  },
  selectedCategoryText: {
    color: 'white',
  },
  productsContainer: {
    padding: 10,
  },
  productRow: {
    justifyContent: 'space-between',
  },
  productCard: {
    backgroundColor: 'white',
    borderRadius: 10,
    width: '48%',
    marginBottom: 15,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  productImage: {
    width: '100%',
    height: 120,
    borderTopLeftRadius: 10,
    borderTopRightRadius: 10,
  },
  discountBadge: {
    position: 'absolute',
    top: 10,
    right: 10,
    backgroundColor: '#FF3B30',
    paddingHorizontal: 8,
    paddingVertical: 4,
    borderRadius: 12,
  },
  discountText: {
    color: 'white',
    fontSize: 12,
    fontWeight: 'bold',
  },
  productInfo: {
    padding: 10,
  },
  productName: {
    fontSize: 14,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 5,
  },
  priceContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 5,
  },
  currentPrice: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#007AFF',
    marginRight: 5,
  },
  originalPrice: {
    fontSize: 14,
    color: '#999',
    textDecorationLine: 'line-through',
  },
  ratingContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 5,
  },
  rating: {
    fontSize: 14,
    color: '#FFA500',
    marginRight: 5,
  },
  reviews: {
    fontSize: 12,
    color: '#666',
  },
  stockContainer: {
    marginTop: 5,
  },
  stockText: {
    fontSize: 12,
    fontWeight: 'bold',
  },
  emptyContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    paddingVertical: 50,
  },
  emptyText: {
    fontSize: 16,
    color: '#999',
  },
});

export default ProductList;
```

---

## 📝 Assignment 7: Lists & Performance

### Task 1: Contact List App (90 minutes)
Create a contact list with:
- Alphabetical sections (A-Z)
- Search functionality
- Contact details view
- Add/edit/delete contacts
- Performance optimization

### Task 2: News Feed App (120 minutes)
Build a news feed with:
- Pull-to-refresh
- Infinite scrolling
- Category filtering
- Bookmark functionality
- Offline caching

### Task 3: Task Manager with Lists (150 minutes)
Implement a task manager with:
- Multiple lists (Todo, In Progress, Done)
- Drag and drop between lists
- Priority levels
- Due dates
- Search and filter

---

## 🧠 Quiz 7: Lists & Performance

### Multiple Choice Questions (1-15)

**Q1. FlatList ScrollView se better kyu hai?**
a) Better styling options
b) Performance optimization for large lists
c) Easier to implement
d) More features

**Q2. FlatList mein keyExtractor prop kya karta hai?**
a) Items ko sort karta hai
b) Unique keys provide karta hai
c) Items ko filter karta hai
d) Items ko group karta hai

**Q3. Infinite scrolling implement karne ke liye kaunsa prop use karte hain?**
a) onScroll
b) onEndReached
c) onMomentumScrollEnd
d) onScrollEndDrag

**Q4. Pull-to-refresh implement karne ke liye kaunsa component use karte hain?**
a) RefreshControl
b) ScrollView
c) FlatList
d) SectionList

**Q5. FlatList mein initialNumToRender prop kya karta hai?**
a) Total items set karta hai
b) Initial render hone wale items ki count
c) Maximum render items
d) Minimum render items

**Q6. removeClippedSubviews prop kya karta hai?**
a) Items ko remove karta hai
b) Off-screen items ko memory se remove karta hai
c) Items ko clip karta hai
d) Subviews ko hide karta hai

**Q7. SectionList FlatList se different kyu hai?**
a) Better performance
b) Grouped data display karta hai
c) Horizontal scrolling support
d) Simple implementation

**Q8. getItemLayout prop kab use karte hain?**
a) Variable height items ke liye
b) Fixed height items ke liye
c) Dynamic height items ke liye
d) No height items ke liye

**Q9. FlatList mein windowSize prop kya control karta hai?**
a) Visible items ki count
b) Memory mein rakhe gaye items ki count
c) Total items ki count
d) Rendered items ki count

**Q10. maxToRenderPerBatch prop kya karta hai?**
a) Maximum batch size set karta hai
b) Minimum batch size set karta hai
c) Batch rendering ko disable karta hai
d) Batch rendering ko enable karta hai

### True/False Questions (16-20)

**Q16. FlatList automatically virtualized hota hai.** (True/False)

**Q17. ScrollView large datasets ke liye suitable nahi hai.** (True/False)

**Q18. SectionList sirf vertical scrolling support karta hai.** (True/False)

**Q19. Pull-to-refresh native iOS/Android feature hai.** (True/False)

**Q20. FlatList mein key prop optional hai.** (True/False)

### Code Output Questions (21-25)

**Q21. Is code ka output kya hoga?**
```javascript
<FlatList
  data={[]}
  renderItem={({ item }) => <Text>{item}</Text>}
  ListEmptyComponent={<Text>No items</Text>}
/>
```

**Q22. Is FlatList mein problem kya hai?**
```javascript
<FlatList
  data={data}
  renderItem={renderItem}
  // Missing keyExtractor
/>
```

**Q23. Is code ka behavior kya hoga?**
```javascript
<FlatList
  data={data}
  renderItem={renderItem}
  onEndReached={loadMore}
  onEndReachedThreshold={0.1}
/>
```

**Q24. Is optimization ka benefit kya hai?**
```javascript
<FlatList
  data={data}
  renderItem={renderItem}
  initialNumToRender={10}
  maxToRenderPerBatch={5}
  windowSize={10}
/>
```

**Q25. Is code mein kya galat hai?**
```javascript
const renderItem = ({ item }) => (
  <TouchableOpacity onPress={() => console.log(item.id)}>
    <Text>{item.title}</Text>
  </TouchableOpacity>
);
```

---

## 📚 Answers & Explanations

### Multiple Choice Answers
**Q1.** b) Performance optimization for large lists  
**Q2.** b) Unique keys provide karta hai  
**Q3.** b) onEndReached  
**Q4.** a) RefreshControl  
**Q5.** b) Initial render hone wale items ki count  
**Q6.** b) Off-screen items ko memory se remove karta hai  
**Q7.** b) Grouped data display karta hai  
**Q8.** b) Fixed height items ke liye  
**Q9.** b) Memory mein rakhe gaye items ki count  
**Q10.** a) Maximum batch size set karta hai  

### True/False Answers
**Q16.** True - FlatList automatically virtualized hota hai  
**Q17.** True - ScrollView large datasets ke liye suitable nahi hai  
**Q18.** False - SectionList horizontal bhi support karta hai  
**Q19.** True - Pull-to-refresh native iOS/Android feature hai  
**Q20.** False - FlatList mein keyExtractor required hai  

### Code Output Answers
**Q21.** "No items" text display hoga  
**Q22.** Warning: Missing keyExtractor, performance issues  
**Q23.** Infinite scroll trigger hoga jab 10% items remaining  
**Q24.** Better performance, smooth scrolling, memory optimization  
**Q25.** No key prop in TouchableOpacity, re-render issues  

---

## 🎯 Assignment Solutions

### Task 1: Contact List Solution

```javascript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  FlatList,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  Alert,
  SectionList
} from 'react-native';

const ContactList = () => {
  const [contacts, setContacts] = useState([]);
  const [searchQuery, setSearchQuery] = useState('');
  const [filteredContacts, setFilteredContacts] = useState([]);
  const [sections, setSections] = useState([]);

  // Generate sample contacts
  const generateContacts = () => {
    const sampleContacts = [
      { id: '1', name: 'Aarav Sharma', phone: '+91 9876543210', email: 'aarav@example.com' },
      { id: '2', name: 'Ananya Patel', phone: '+91 9876543211', email: 'ananya@example.com' },
      { id: '3', name: 'Arjun Kumar', phone: '+91 9876543212', email: 'arjun@example.com' },
      { id: '4', name: 'Bhavya Singh', phone: '+91 9876543213', email: 'bhavya@example.com' },
      { id: '5', name: 'Chetan Gupta', phone: '+91 9876543214', email: 'chetan@example.com' },
      { id: '6', name: 'Divya Reddy', phone: '+91 9876543215', email: 'divya@example.com' },
      { id: '7', name: 'Esha Jain', phone: '+91 9876543216', email: 'esha@example.com' },
      { id: '8', name: 'Farhan Khan', phone: '+91 9876543217', email: 'farhan@example.com' },
    ];
    setContacts(sampleContacts);
  };

  useEffect(() => {
    generateContacts();
  }, []);

  useEffect(() => {
    filterContacts();
  }, [searchQuery, contacts]);

  useEffect(() => {
    createSections();
  }, [filteredContacts]);

  const filterContacts = () => {
    if (!searchQuery.trim()) {
      setFilteredContacts(contacts);
    } else {
      const filtered = contacts.filter(contact =>
        contact.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
        contact.phone.includes(searchQuery) ||
        contact.email.toLowerCase().includes(searchQuery.toLowerCase())
      );
      setFilteredContacts(filtered);
    }
  };

  const createSections = () => {
    const grouped = filteredContacts.reduce((acc, contact) => {
      const firstLetter = contact.name.charAt(0).toUpperCase();
      if (!acc[firstLetter]) {
        acc[firstLetter] = [];
      }
      acc[firstLetter].push(contact);
      return acc;
    }, {});

    const sectionArray = Object.keys(grouped)
      .sort()
      .map(letter => ({
        title: letter,
        data: grouped[letter].sort((a, b) => a.name.localeCompare(b.name))
      }));

    setSections(sectionArray);
  };

  const renderContact = ({ item }) => (
    <TouchableOpacity
      style={styles.contactItem}
      onPress={() => showContactDetails(item)}
    >
      <View style={styles.avatar}>
        <Text style={styles.avatarText}>
          {item.name.split(' ').map(n => n[0]).join('').toUpperCase()}
        </Text>
      </View>
      <View style={styles.contactInfo}>
        <Text style={styles.contactName}>{item.name}</Text>
        <Text style={styles.contactPhone}>{item.phone}</Text>
      </View>
    </TouchableOpacity>
  );

  const renderSectionHeader = ({ section: { title } }) => (
    <View style={styles.sectionHeader}>
      <Text style={styles.sectionHeaderText}>{title}</Text>
    </View>
  );

  const showContactDetails = (contact) => {
    Alert.alert(
      contact.name,
      `Phone: ${contact.phone}\nEmail: ${contact.email}`,
      [
        { text: 'Call', onPress: () => console.log('Call', contact.phone) },
        { text: 'Message', onPress: () => console.log('Message', contact.phone) },
        { text: 'Cancel', style: 'cancel' }
      ]
    );
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Contacts</Text>

      {/* Search Bar */}
      <View style={styles.searchContainer}>
        <TextInput
          style={styles.searchInput}
          placeholder="Search contacts..."
          value={searchQuery}
          onChangeText={setSearchQuery}
          autoCapitalize="none"
        />
      </View>

      {/* Contact List */}
      <SectionList
        sections={sections}
        renderItem={renderContact}
        renderSectionHeader={renderSectionHeader}
        keyExtractor={item => item.id}
        showsVerticalScrollIndicator={false}
        contentContainerStyle={styles.listContainer}
        ListEmptyComponent={
          <View style={styles.emptyContainer}>
            <Text style={styles.emptyText}>
              {searchQuery ? 'No contacts found' : 'No contacts available'}
            </Text>
          </View>
        }
      />
    </View>
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
    padding: 20,
    backgroundColor: 'white',
    color: '#333',
  },
  searchContainer: {
    backgroundColor: 'white',
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  searchInput: {
    backgroundColor: '#f8f9fa',
    borderRadius: 8,
    paddingHorizontal: 15,
    paddingVertical: 10,
    fontSize: 16,
  },
  listContainer: {
    padding: 15,
  },
  sectionHeader: {
    backgroundColor: '#007AFF',
    paddingVertical: 8,
    paddingHorizontal: 15,
    borderRadius: 8,
    marginTop: 15,
    marginBottom: 5,
  },
  sectionHeaderText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  contactItem: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 8,
    marginBottom: 5,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.2,
    shadowRadius: 2,
    elevation: 3,
  },
  avatar: {
    width: 50,
    height: 50,
    borderRadius: 25,
    backgroundColor: '#007AFF',
    justifyContent: 'center',
    alignItems: 'center',
    marginRight: 15,
  },
  avatarText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  contactInfo: {
    flex: 1,
  },
  contactName: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 2,
  },
  contactPhone: {
    fontSize: 14,
    color: '#666',
  },
  emptyContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    paddingVertical: 50,
  },
  emptyText: {
    fontSize: 16,
    color: '#999',
    textAlign: 'center',
  },
});

export default ContactList;
```

---

## 📝 Lesson Summary

### Key Concepts Learned
- ✅ **ScrollView Component**: Basic scrolling, horizontal scrolling, pagination
- ✅ **FlatList Component**: Basic usage, advanced features, search functionality
- ✅ **SectionList Component**: Grouped data display with sections
- ✅ **Pull-to-Refresh**: RefreshControl implementation
- ✅ **Infinite Scrolling**: Load more data dynamically
- ✅ **Performance Optimization**: Virtualized lists, memoization, getItemLayout
- ✅ **List Components**: Empty states, headers, footers, separators

### Best Practices
1. **Use FlatList for large datasets** instead of ScrollView
2. **Always provide keyExtractor** for unique item identification
3. **Implement proper loading states** for better UX
4. **Use getItemLayout** for fixed-height items
5. **Optimize with initialNumToRender** and maxToRenderPerBatch
6. **Handle empty states** appropriately
7. **Use SectionList** for grouped data
8. **Implement proper error handling** for data loading

### Next Steps
- Learn about FlashList (better FlatList alternative)
- Study advanced list animations
- Implement drag and drop functionality
- Explore list virtualization techniques

---

## 📚 Additional Resources
- [React Native FlatList Documentation](https://reactnative.dev/docs/flatlist)
- [React Native SectionList Documentation](https://reactnative.dev/docs/sectionlist)
- [React Native ScrollView Documentation](https://reactnative.dev/docs/scrollview)
- [Performance Best Practices](https://reactnative.dev/docs/optimizing-flatlist-configuration)
- [FlashList Library](https://github.com/Shopify/flash-list)

---

**Next Lesson**: [Lesson 8: Navigation in React Native](Lesson%208_%20Navigation%20in%20React%20Native.md)