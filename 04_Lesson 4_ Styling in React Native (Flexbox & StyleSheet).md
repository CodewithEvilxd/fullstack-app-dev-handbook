
# 🎨 Lesson 4: Styling in React Native (Flexbox & StyleSheet)

## 🎯 Learning Objectives
Is lesson ke baad aap:
- React Native styling system samjhenge
- Flexbox layout master kar sakte hain
- StyleSheet API efficiently use kar sakte hain
- Responsive designs create kar sakte hain
- Platform-specific styling kar sakte hain

---

## 📖 React Native Styling Basics

React Native mein styling CSS ke similar hai lekin kuch differences hain:
- **No CSS files** - JavaScript objects use karte hain
- **Camel case properties** - background-color → backgroundColor
- **Limited CSS properties** - Sirf mobile-relevant properties
- **Flexbox by default** - Har View component flex container hai

---

## 🎨 StyleSheet API

### 1. Basic Styling

```javascript
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

const BasicStyling = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Hello World!</Text>
      <Text style={styles.subtitle}>Welcome to React Native</Text>
      
      {/* Inline styling (not recommended for complex styles) */}
      <Text style={{ color: 'red', fontSize: 16 }}>
        Inline styled text
      </Text>
      
      {/* Multiple styles */}
      <Text style={[styles.text, styles.boldText]}>
        Multiple styles applied
      </Text>
      
      {/* Conditional styling */}
      <Text style={[
        styles.text,
        { color: true ? 'green' : 'red' }
      ]}>
        Conditional styling
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f0f0f0',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 18,
    color: '#666',
    marginBottom: 20,
  },
  text: {
    fontSize: 16,
    color: '#333',
    marginBottom: 10,
  },
  boldText: {
    fontWeight: 'bold',
  },
});

export default BasicStyling;
```

### 2. StyleSheet Benefits

```javascript
// ❌ Without StyleSheet (inefficient)
const BadExample = () => {
  return (
    <View style={{
      flex: 1,
      backgroundColor: '#fff',
      padding: 20,
      justifyContent: 'center',
      alignItems: 'center'
    }}>
      <Text style={{
        fontSize: 24,
        fontWeight: 'bold',
        color: '#333',
        textAlign: 'center'
      }}>
        Title
      </Text>
    </View>
  );
};

// ✅ With StyleSheet (efficient)
const GoodExample = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Title</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    padding: 20,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    textAlign: 'center',
  },
});

// StyleSheet benefits:
// 1. Performance optimization
// 2. Code reusability
// 3. Better organization
// 4. Validation in development
// 5. Style caching
```

---

## 📐 Flexbox Layout System

### 1. Flex Direction

```javascript
const FlexDirectionExample = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.sectionTitle}>Flex Direction Examples</Text>
      
      {/* Row (default for web, but column for React Native) */}
      <View style={styles.section}>
        <Text style={styles.label}>flexDirection: 'row'</Text>
        <View style={[styles.flexContainer, { flexDirection: 'row' }]}>
          <View style={[styles.box, { backgroundColor: 'red' }]} />
          <View style={[styles.box, { backgroundColor: 'green' }]} />
          <View style={[styles.box, { backgroundColor: 'blue' }]} />
        </View>
      </View>
      
      {/* Column (default in React Native) */}
      <View style={styles.section}>
        <Text style={styles.label}>flexDirection: 'column'</Text>
        <View style={[styles.flexContainer, { flexDirection: 'column' }]}>
          <View style={[styles.box, { backgroundColor: 'red' }]} />
          <View style={[styles.box, { backgroundColor: 'green' }]} />
          <View style={[styles.box, { backgroundColor: 'blue' }]} />
        </View>
      </View>
      
      {/* Row Reverse */}
      <View style={styles.section}>
        <Text style={styles.label}>flexDirection: 'row-reverse'</Text>
        <View style={[styles.flexContainer, { flexDirection: 'row-reverse' }]}>
          <View style={[styles.box, { backgroundColor: 'red' }]} />
          <View style={[styles.box, { backgroundColor: 'green' }]} />
          <View style={[styles.box, { backgroundColor: 'blue' }]} />
        </View>
      </View>
      
      {/* Column Reverse */}
      <View style={styles.section}>
        <Text style={styles.label}>flexDirection: 'column-reverse'</Text>
        <View style={[styles.flexContainer, { flexDirection: 'column-reverse' }]}>
          <View style={[styles.box, { backgroundColor: 'red' }]} />
          <View style={[styles.box, { backgroundColor: 'green' }]} />
          <View style={[styles.box, { backgroundColor: 'blue' }]} />
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
  sectionTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  section: {
    marginBottom: 30,
  },
  label: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333',
  },
  flexContainer: {
    backgroundColor: '#fff',
    padding: 10,
    borderRadius: 8,
    minHeight: 100,
  },
  box: {
    width: 50,
    height: 50,
    margin: 5,
    borderRadius: 4,
  },
});
```

### 2. Justify Content (Main Axis)

```javascript
const JustifyContentExample = () => {
  const justifyOptions = [
    'flex-start',
    'flex-end',
    'center',
    'space-between',
    'space-around',
    'space-evenly'
  ];

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Justify Content Examples</Text>
      
      {justifyOptions.map((option, index) => (
        <View key={index} style={styles.section}>
          <Text style={styles.label}>justifyContent: '{option}'</Text>
          <View style={[
            styles.flexContainer,
            {
              flexDirection: 'row',
              justifyContent: option
            }
          ]}>
            <View style={[styles.smallBox, { backgroundColor: 'red' }]} />
            <View style={[styles.smallBox, { backgroundColor: 'green' }]} />
            <View style={[styles.smallBox, { backgroundColor: 'blue' }]} />
          </View>
        </View>
      ))}
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  section: {
    marginBottom: 25,
  },
  label: {
    fontSize: 14,
    fontWeight: 'bold',
    marginBottom: 8,
    color: '#333',
  },
  flexContainer: {
    backgroundColor: '#fff',
    padding: 10,
    borderRadius: 8,
    height: 80,
    borderWidth: 1,
    borderColor: '#ddd',
  },
  smallBox: {
    width: 40,
    height: 40,
    borderRadius: 4,
  },
});
```

### 3. Align Items (Cross Axis)

```javascript
const AlignItemsExample = () => {
  const alignOptions = [
    'flex-start',
    'flex-end',
    'center',
    'stretch',
    'baseline'
  ];

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Align Items Examples</Text>
      
      {alignOptions.map((option, index) => (
        <View key={index} style={styles.section}>
          <Text style={styles.label}>alignItems: '{option}'</Text>
          <View style={[
            styles.flexContainer,
            {
              flexDirection: 'row',
              alignItems: option
            }
          ]}>
            <View style={[styles.box1, { backgroundColor: 'red' }]} />
            <View style={[styles.box2, { backgroundColor: 'green' }]} />
            <View style={[styles.box3, { backgroundColor: 'blue' }]} />
          </View>
        </View>
      ))}
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  section: {
    marginBottom: 25,
  },
  label: {
    fontSize: 14,
    fontWeight: 'bold',
    marginBottom: 8,
    color: '#333',
  },
  flexContainer: {
    backgroundColor: '#fff',
    padding: 10,
    borderRadius: 8,
    height: 120,
    borderWidth: 1,
    borderColor: '#ddd',
  },
  box1: {
    width: 40,
    height: 40,
    marginRight: 10,
    borderRadius: 4,
  },
  box2: {
    width: 40,
    height: 60,
    marginRight: 10,
    borderRadius: 4,
  },
  box3: {
    width: 40,
    height: 80,
    borderRadius: 4,
  },
});
```

### 4. Flex Property

```javascript
const FlexPropertyExample = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Flex Property Examples</Text>
      
      {/* Equal flex */}
      <View style={styles.section}>
        <Text style={styles.label}>Equal Flex (flex: 1)</Text>
        <View style={styles.flexContainer}>
          <View style={[styles.flexBox, { flex: 1, backgroundColor: 'red' }]}>
            <Text style={styles.boxText}>flex: 1</Text>
          </View>
          <View style={[styles.flexBox, { flex: 1, backgroundColor: 'green' }]}>
            <Text style={styles.boxText}>flex: 1</Text>
          </View>
          <View style={[styles.flexBox, { flex: 1, backgroundColor: 'blue' }]}>
            <Text style={styles.boxText}>flex: 1</Text>
          </View>
        </View>
      </View>
      
      {/* Different flex ratios */}
      <View style={styles.section}>
        <Text style={styles.label}>Different Flex Ratios</Text>
        <View style={styles.flexContainer}>
          <View style={[styles.flexBox, { flex: 1, backgroundColor: 'red' }]}>
            <Text style={styles.boxText}>flex: 1</Text>
          </View>
          <View style={[styles.flexBox, { flex: 2, backgroundColor: 'green' }]}>
            <Text style={styles.boxText}>flex: 2</Text>
          </View>
          <View style={[styles.flexBox, { flex: 3, backgroundColor: 'blue' }]}>
            <Text style={styles.boxText}>flex: 3</Text>
          </View>
        </View>
      </View>
      
      {/* Fixed width with flex */}
      <View style={styles.section}>
        <Text style={styles.label}>Fixed Width + Flex</Text>
        <View style={styles.flexContainer}>
          <View style={[styles.fixedBox, { backgroundColor: 'red' }]}>
            <Text style={styles.boxText}>Fixed 80px</Text>
          </View>
          <View style={[styles.flexBox, { flex: 1, backgroundColor: 'green' }]}>
            <Text style={styles.boxText}>flex: 1</Text>
          </View>
          <View style={[styles.fixedBox, { backgroundColor: 'blue' }]}>
            <Text style={styles.boxText}>Fixed 80px</Text>
          </View>
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
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  section: {
    marginBottom: 30,
  },
  label: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 10,
    color: '#333',
  },
  flexContainer: {
    flexDirection: 'row',
    backgroundColor: '#fff',
    borderRadius: 8,
    overflow: 'hidden',
    height: 80,
  },
  flexBox: {
    justifyContent: 'center',
    alignItems: 'center',
    margin: 2,
    borderRadius: 4,
  },
  fixedBox: {
    width: 80,
    justifyContent: 'center',
    alignItems: 'center',
    margin: 2,
    borderRadius: 4,
  },
  boxText: {
    color: 'white',
    fontWeight: 'bold',
    fontSize: 12,
    textAlign: 'center',
  },
});
```

---

## 🎯 Common Layout Patterns

### 1. Header, Content, Footer Layout

```javascript
const HeaderContentFooterLayout = () => {
  return (
    <View style={styles.container}>
      {/* Header */}
      <View style={styles.header}>
        <Text style={styles.headerText}>Header</Text>
      </View>
      
      {/* Content (scrollable) */}
      <ScrollView style={styles.content} contentContainerStyle={styles.contentContainer}>
        {Array.from({ length: 20 }, (_, i) => (
          <View key={i} style={styles.contentItem}>
            <Text style={styles.contentText}>Content Item {i + 1}</Text>
          </View>
        ))}
      </ScrollView>
      
      {/* Footer */}
      <View style={styles.footer}>
        <TouchableOpacity style={styles.footerButton}>
          <Text style={styles.footerButtonText}>Action 1</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.footerButton}>
          <Text style={styles.footerButtonText}>Action 2</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  header: {
    height: 60,
    backgroundColor: '#007AFF',
    justifyContent: 'center',
    alignItems: 'center',
    paddingTop: 20,
  },
  headerText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
  content: {
    flex: 1,
  },
  contentContainer: {
    padding: 20,
  },
  contentItem: {
    backgroundColor: 'white',
    padding: 15,
    marginBottom: 10,
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.2,
    shadowRadius: 2,
    elevation: 3,
  },
  contentText: {
    fontSize: 16,
    color: '#333',
  },
  footer: {
    height: 60,
    backgroundColor: 'white',
    flexDirection: 'row',
    borderTopWidth: 1,
    borderTopColor: '#ddd',
  },
  footerButton: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    borderRightWidth: 1,
    borderRightColor: '#ddd',
  },
  footerButtonText: {
    fontSize: 16,
    color: '#007AFF',
    fontWeight: 'bold',
  },
});
```

### 2. Card Layout

```javascript
const CardLayout = () => {
  const cardData = [
    {
      id: 1,
      title: 'Beautiful Sunset',
      description: 'A stunning sunset over the mountains',
      image: 'https://picsum.photos/300/200?random=1',
      likes: 42,
      comments: 8
    },
    {
      id: 2,
      title: 'City Lights',
      description: 'Night view of the bustling city',
      image: 'https://picsum.photos/300/200?random=2',
      likes: 38,
      comments: 12
    }
  ];

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Card Layout Example</Text>
      
      {cardData.map(card => (
        <View key={card.id} style={styles.card}>
          <Image source={{ uri: card.image }} style={styles.cardImage} />
          
          <View style={styles.cardContent}>
            <Text style={styles.cardTitle}>{card.title}</Text>
            <Text style={styles.cardDescription}>{card.description}</Text>
            
            <View style={styles.cardActions}>
              <View style={styles.cardStats}>
                <View style={styles.statItem}>
                  <Text style={styles.statNumber}>{card.likes}</Text>
                  <Text style={styles.statLabel}>Likes</Text>
                </View>
                <View style={styles.statItem}>
                  <Text style={styles.statNumber}>{card.comments}</Text>
                  <Text style={styles.statLabel}>Comments</Text>
                </View>
              </View>
              
              <View style={styles.cardButtons}>
                <TouchableOpacity style={styles.actionButton}>
                  <Text style={styles.actionButtonText}>Like</Text>
                </TouchableOpacity>
                <TouchableOpacity style={styles.actionButton}>
                  <Text style={styles.actionButtonText}>Share</Text>
                </TouchableOpacity>
              </View>
            </View>
          </View>
        </View>
      ))}
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
    padding: 15,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
    color: '#333',
  },
  card: {
    backgroundColor: 'white',
    borderRadius: 12,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 5,
    overflow: 'hidden',
  },
  cardImage: {
    width: '100%',
    height: 200,
  },
  cardContent: {
    padding: 15,
  },
  cardTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 8,
  },
  cardDescription: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
    marginBottom: 15,
  },
  cardActions: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  },
  cardStats: {
    flexDirection: 'row',
  },
  statItem: {
    alignItems: 'center',
    marginRight: 20,
  },
  statNumber: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
  },
  statLabel: {
    fontSize: 12,
    color: '#666',
  },
  cardButtons: {
    flexDirection: 'row',
  },
  actionButton: {
    paddingHorizontal: 15,
    paddingVertical: 8,
    backgroundColor: '#007AFF',
    borderRadius: 20,
    marginLeft: 10,
  },
  actionButtonText: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
});
```

### 3. Grid Layout

```javascript
const GridLayout = () => {
  const gridData = Array.from({ length: 20 }, (_, i) => ({
    id: i + 1,
    title: `Item ${i + 1}`,
    color: `hsl(${(i * 30) % 360}, 70%, 60%)`
  }));

  const renderGridItem = (item, index) => (
    <TouchableOpacity
      key={item.id}
      style={[styles.gridItem, { backgroundColor: item.color }]}
      onPress={() => Alert.alert('Pressed', `You pressed ${item.title}`)}
    >
      <Text style={styles.gridItemText}>{item.title}</Text>
    </TouchableOpacity>
  );

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Grid Layout Example</Text>
      
      {/* 2 Column Grid */}
      <Text style={styles.sectionTitle}>2 Column Grid</Text>
      <View style={styles.grid2}>
        {gridData.slice(0, 8).map(renderGridItem)}
      </View>
      
      {/* 3 Column Grid */}
      <Text style={styles.sectionTitle}>3 Column Grid</Text>
      <View style={styles.grid3}>
        {gridData.slice(8, 17).map(renderGridItem)}
      </View>
      
      {/* Mixed Grid */}
      <Text style={styles.sectionTitle}>Mixed Grid</Text>
      <View style={styles.mixedGrid}>
        <TouchableOpacity style={[styles.largeGridItem, { backgroundColor: '#FF6B6B' }]}>
          <Text style={styles.gridItemText}>Large Item</Text>
        </TouchableOpacity>
        <View style={styles.smallGridContainer}>
          <TouchableOpacity style={[styles.smallGridItem, { backgroundColor: '#4ECDC4' }]}>
            <Text style={styles.gridItemText}>Small 1</Text>
          </TouchableOpacity>
          <TouchableOpacity style={[styles.smallGridItem, { backgroundColor: '#45B7D1' }]}>
            <Text style={styles.gridItemText}>Small 2</Text>
          </TouchableOpacity>
          <TouchableOpacity style={[styles.smallGridItem, { backgroundColor: '#96CEB4' }]}>
            <Text style={styles.gridItemText}>Small 3</Text>
          </TouchableOpacity>
          <TouchableOpacity style={[styles.smallGridItem, { backgroundColor: '#FFEAA7' }]}>
            <Text style={styles.gridItemText}>Small 4</Text>
          </TouchableOpacity>
        </View>
      </View>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
    padding: 15,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 20,
    color: '#333',
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginTop: 20,
    marginBottom: 15,
    color: '#333',
  },
  grid2: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-between',
  },
  grid3: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-between',
  },
  gridItem: {
    width: '48%',
    height: 100,
    marginBottom: 15,
    borderRadius: 8,
    justifyContent: 'center',
    alignItems: 'center',
  },
  gridItemText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  mixedGrid: {
    flexDirection: 'row',
    height: 200,
  },
  largeGridItem: {
    flex: 1,
    marginRight: 10,
    borderRadius: 8,
    justifyContent: 'center',
    alignItems: 'center',
  },
  smallGridContainer: {
    flex: 1,
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-between',
  },
  smallGridItem: {
    width: '48%',
    height: '48%',
    borderRadius: 8,
    justifyContent: 'center',
    alignItems: 'center',
    marginBottom: '4%',
  },
});

// For 3 column grid, modify gridItem width
StyleSheet.create({
  ...styles,
  grid3: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-between',
  },
  gridItem3: {
    width: '31%',
    height: 80,
    marginBottom: 15,
    borderRadius: 8,
    justifyContent: 'center',
    alignItems: 'center',
  },
});
```

---

## 📱 Responsive Design

### 1. Dimensions API

```javascript
import { Dimensions } from 'react-native';

const ResponsiveExample = () => {
  const { width, height } = Dimensions.get('window');
  const isTablet = width > 768;
  const isLandscape = width > height;

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Responsive Design</Text>
      
      <View style={styles.infoContainer}>
        <Text style={styles.infoText}>Screen Width: {width.toFixed(0)}px</Text>
        <Text style={styles.infoText}>Screen Height: {height.toFixed(0)}px</Text>
        <Text style={styles.infoText}>Device Type: {isTablet ? 'Tablet' : 'Phone'}</Text>
        <Text style={styles.infoText}>Orientation: {isLandscape ? 'Landscape' : 'Portrait'}</Text>
      </View>
      
      {/* Responsive grid */}
      <View style={[
        styles.responsiveGrid,
        {
          flexDirection: isLandscape ? 'row' : 'column',
        }
      ]}>
        <View style={[
          styles.gridItem,
          {
            width: isLandscape ? width * 0.3 : width * 0.8,
            height: isTablet ? 150 : 100,
          }
        ]}>
          <Text style={styles.gridText}>Responsive Item 1</Text>
        </View>
        
        <View style={[
          styles.gridItem,
          {
            width: isLandscape ? width * 0.3 : width * 0.8,
            height: isTablet ? 150 : 100,
          }
        ]}>
          <Text style={styles.gridText}>Responsive Item 2</Text>
        </View>
      </View>
      
      {/* Responsive text */}
      <Text style={[
        styles.responsiveText,
        {
          fontSize: isTablet ? 24 : 18,
          padding: isTablet ? 30 : 20,
        }
      ]}>
        This text adapts to device size
      </Text>
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
    color: '#333',
  },
  infoContainer: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 8,
    marginBottom: 20,
  },
  infoText: {
    fontSize: 16,
    color: '#333',
    marginBottom: 5,
  },
  responsiveGrid: {
    justifyContent: 'space-around',
    alignItems: 'center',
    marginBottom: 20,
  },
  gridItem: {
    backgroundColor: '#007AFF',
    borderRadius: 8,
    justifyContent: 'center',
    alignItems: 'center',
    margin: 10,
  },
  gridText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
    textAlign: 'center',
  },
  responsiveText: {
    backgroundColor: 'white',
    borderRadius: 8,
    textAlign: 'center',
    color: '#333',
  },
});
```

### 2. Platform-Specific Styling

```javascript
import { Platform } from 'react-native';

const PlatformSpecificStyling = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Platform-Specific Styling</Text>

      {/* Platform-specific components */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Platform-Specific Components</Text>

        {Platform.OS === 'ios' ? (
          <View style={styles.iosComponent}>
            <Text style={styles.componentText}>iOS Specific Component</Text>
            <Text style={styles.componentText}>Uses iOS design patterns</Text>
          </View>
        ) : (
          <View style={styles.androidComponent}>
            <Text style={styles.componentText}>Android Specific Component</Text>
            <Text style={styles.componentText}>Uses Material Design</Text>
          </View>
        )}
      </View>

      {/* Platform-specific styles */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Platform-Specific Styles</Text>

        <View style={styles.demoContainer}>
          <Text style={[styles.demoText, styles.platformText]}>
            This text has platform-specific styling
          </Text>

          <View style={[styles.demoBox, styles.platformBox]}>
            <Text style={styles.boxText}>Platform Box</Text>
          </View>
        </View>
      </View>

      {/* Device info */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Device Information</Text>
        <View style={styles.infoContainer}>
          <Text style={styles.infoText}>Platform: {Platform.OS}</Text>
          <Text style={styles.infoText}>Version: {Platform.Version}</Text>
          <Text style={styles.infoText}>isPad: {Platform.isPad ? 'Yes' : 'No'}</Text>
          <Text style={styles.infoText}>isTV: {Platform.isTV ? 'Yes' : 'No'}</Text>
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
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 30,
    color: '#333',
  },
  section: {
    marginBottom: 30,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  iosComponent: {
    backgroundColor: '#f8f9fa',
    padding: 20,
    borderRadius: 10,
    borderWidth: 1,
    borderColor: '#007AFF',
  },
  androidComponent: {
    backgroundColor: '#f8f9fa',
    padding: 20,
    borderRadius: 10,
    borderWidth: 1,
    borderColor: '#4CAF50',
    ...Platform.select({
      android: {
        elevation: 4,
      },
    }),
  },
  componentText: {
    fontSize: 16,
    color: '#333',
    marginBottom: 5,
  },
  demoContainer: {
    alignItems: 'center',
  },
  demoText: {
    fontSize: 16,
    marginBottom: 20,
    textAlign: 'center',
  },
  platformText: Platform.select({
    ios: {
      fontFamily: 'System',
      color: '#007AFF',
    },
    android: {
      fontFamily: 'Roboto',
      color: '#4CAF50',
    },
  }),
  demoBox: {
    width: 150,
    height: 100,
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 10,
  },
  platformBox: Platform.select({
    ios: {
      backgroundColor: '#007AFF',
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 2 },
      shadowOpacity: 0.25,
      shadowRadius: 4,
    },
    android: {
      backgroundColor: '#4CAF50',
      elevation: 4,
    },
  }),
  boxText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  infoContainer: {
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 10,
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 1 },
        shadowOpacity: 0.2,
        shadowRadius: 2,
      },
      android: {
        elevation: 2,
      },
    }),
  },
  infoText: {
    fontSize: 14,
    color: '#666',
    marginBottom: 5,
  },
});
```

---

## 🎨 **Advanced Styling Techniques**

### **1. Dynamic Styles with Theme Support**

```javascript
// Theme configuration
const themes = {
  light: {
    backgroundColor: '#ffffff',
    textColor: '#333333',
    primaryColor: '#007AFF',
    secondaryColor: '#6c757d',
    borderColor: '#e0e0e0',
  },
  dark: {
    backgroundColor: '#1a1a1a',
    textColor: '#ffffff',
    primaryColor: '#0A84FF',
    secondaryColor: '#8e8e93',
    borderColor: '#38383a',
  },
  colorful: {
    backgroundColor: '#f8f9fa',
    textColor: '#2d3748',
    primaryColor: '#e53e3e',
    secondaryColor: '#38a169',
    borderColor: '#e2e8f0',
  },
};

// Theme context
const ThemeContext = React.createContext(themes.light);

const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    const themeKeys = Object.keys(themes);
    const currentIndex = themeKeys.indexOf(theme);
    const nextIndex = (currentIndex + 1) % themeKeys.length;
    setTheme(themeKeys[nextIndex]);
  };

  return (
    <ThemeContext.Provider value={{ theme: themes[theme], toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// Hook to use theme
const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};

// Themed component
const ThemedComponent = () => {
  const { theme, toggleTheme } = useTheme();

  return (
    <View style={[styles.container, { backgroundColor: theme.backgroundColor }]}>
      <Text style={[styles.title, { color: theme.textColor }]}>
        Themed Component
      </Text>

      <Text style={[styles.subtitle, { color: theme.secondaryColor }]}>
        Current theme: {Object.keys(themes).find(key => themes[key] === theme)}
      </Text>

      <TouchableOpacity
        style={[styles.button, { backgroundColor: theme.primaryColor }]}
        onPress={toggleTheme}
      >
        <Text style={[styles.buttonText, { color: theme.backgroundColor }]}>
          Toggle Theme
        </Text>
      </TouchableOpacity>

      <View style={[styles.card, {
        backgroundColor: theme.backgroundColor,
        borderColor: theme.borderColor
      }]}>
        <Text style={[styles.cardText, { color: theme.textColor }]}>
          This card adapts to the current theme
        </Text>
      </View>
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
    textAlign: 'center',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 16,
    textAlign: 'center',
    marginBottom: 30,
  },
  button: {
    paddingHorizontal: 30,
    paddingVertical: 15,
    borderRadius: 25,
    alignItems: 'center',
    marginBottom: 20,
  },
  buttonText: {
    fontSize: 16,
    fontWeight: 'bold',
  },
  card: {
    padding: 20,
    borderRadius: 10,
    borderWidth: 1,
    marginTop: 20,
  },
  cardText: {
    fontSize: 16,
    textAlign: 'center',
  },
});

// App with theme provider
const ThemedApp = () => {
  return (
    <ThemeProvider>
      <ThemedComponent />
    </ThemeProvider>
  );
};
```

### **2. Style Composition and Inheritance**

```javascript
// Base styles
const baseStyles = StyleSheet.create({
  text: {
    fontSize: 16,
    color: '#333',
  },
  button: {
    paddingHorizontal: 20,
    paddingVertical: 12,
    borderRadius: 8,
    alignItems: 'center',
    justifyContent: 'center',
  },
  input: {
    borderWidth: 1,
    borderRadius: 8,
    paddingHorizontal: 12,
    paddingVertical: 10,
    fontSize: 16,
  },
});

// Variant styles
const variantStyles = StyleSheet.create({
  // Text variants
  textLarge: {
    fontSize: 20,
    fontWeight: 'bold',
  },
  textSmall: {
    fontSize: 14,
    color: '#666',
  },
  textError: {
    color: '#e74c3c',
  },
  textSuccess: {
    color: '#27ae60',
  },

  // Button variants
  buttonPrimary: {
    backgroundColor: '#007AFF',
  },
  buttonSecondary: {
    backgroundColor: '#6c757d',
  },
  buttonDanger: {
    backgroundColor: '#e74c3c',
  },
  buttonSuccess: {
    backgroundColor: '#27ae60',
  },
  buttonOutline: {
    backgroundColor: 'transparent',
    borderWidth: 1,
    borderColor: '#007AFF',
  },

  // Input variants
  inputError: {
    borderColor: '#e74c3c',
  },
  inputSuccess: {
    borderColor: '#27ae60',
  },
  inputDisabled: {
    backgroundColor: '#f8f9fa',
    color: '#6c757d',
  },
});

// Size variants
const sizeStyles = StyleSheet.create({
  buttonSmall: {
    paddingHorizontal: 12,
    paddingVertical: 8,
  },
  buttonMedium: {
    paddingHorizontal: 20,
    paddingVertical: 12,
  },
  buttonLarge: {
    paddingHorizontal: 32,
    paddingVertical: 16,
  },
});

// Utility function for style composition
const composeStyles = (...styles) => {
  return StyleSheet.flatten(styles);
};

// Styled components using composition
const StyledComponents = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Style Composition Example</Text>

      {/* Text variants */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Text Variants</Text>
        <Text style={composeStyles(baseStyles.text, variantStyles.textLarge)}>
          Large Text
        </Text>
        <Text style={composeStyles(baseStyles.text, variantStyles.textSmall)}>
          Small Text
        </Text>
        <Text style={composeStyles(baseStyles.text, variantStyles.textError)}>
          Error Text
        </Text>
        <Text style={composeStyles(baseStyles.text, variantStyles.textSuccess)}>
          Success Text
        </Text>
      </View>

      {/* Button variants */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Button Variants</Text>

        <TouchableOpacity style={composeStyles(
          baseStyles.button,
          variantStyles.buttonPrimary,
          sizeStyles.buttonMedium
        )}>
          <Text style={styles.buttonText}>Primary Button</Text>
        </TouchableOpacity>

        <TouchableOpacity style={composeStyles(
          baseStyles.button,
          variantStyles.buttonSecondary,
          sizeStyles.buttonMedium
        )}>
          <Text style={styles.buttonText}>Secondary Button</Text>
        </TouchableOpacity>

        <TouchableOpacity style={composeStyles(
          baseStyles.button,
          variantStyles.buttonOutline,
          sizeStyles.buttonMedium
        )}>
          <Text style={[styles.buttonText, { color: '#007AFF' }]}>
            Outline Button
          </Text>
        </TouchableOpacity>
      </View>

      {/* Input variants */}
      <View style={styles.section}>
        <Text style={styles.sectionTitle}>Input Variants</Text>

        <TextInput
          style={composeStyles(baseStyles.input, variantStyles.inputSuccess)}
          placeholder="Success input"
        />

        <TextInput
          style={composeStyles(baseStyles.input, variantStyles.inputError)}
          placeholder="Error input"
        />

        <TextInput
          style={composeStyles(baseStyles.input, variantStyles.inputDisabled)}
          placeholder="Disabled input"
          editable={false}
        />
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
    marginBottom: 30,
    color: '#333',
  },
  section: {
    marginBottom: 30,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
    color: '#333',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});
```

---

## 📝 **Assignment 4: Flexbox & Styling Practice**

### **Task 1: Complex Layout Challenge (60 minutes)**
Create a social media profile screen with:
- Header with profile picture and stats
- Tab navigation (Posts, Photos, Videos)
- Grid layout for content
- Floating action button
- Responsive design for different screen sizes

### **Task 2: Theme System Implementation (45 minutes)**
Build a complete theme system with:
- Light and dark themes
- Custom color palettes
- Typography scales
- Component theming
- Theme persistence

### **Task 3: Responsive Card Grid (30 minutes)**
Create a responsive card grid that:
- Adapts to screen size (1-4 columns)
- Handles different card sizes
- Supports infinite scroll
- Has smooth animations
- Works in both orientations

---

## 🧠 **Quiz 4: Flexbox & Styling**

### **Multiple Choice Questions**

**Q1. Flexbox mein main axis ka direction kya hota hai?**
a) Upar niche
b) Left to right (default)
c) Diagonal
d) Circular

**Q2. React Native mein StyleSheet.create() ka main benefit kya hai?**
a) Code readability
b) Performance optimization
c) Bundle size reduction
d) All of the above

**Q3. Platform-specific styling ke liye kya use karte hain?**
a) Platform.select()
b) Platform.OS
c) Dimensions.get()
d) StyleSheet.compose()

**Q4. Flex: 1 ka matlab kya hai?**
a) Component ka size fixed hai
b) Component available space ko occupy karta hai
c) Component hide ho jata hai
d) Component center mein aa jata hai

**Q5. justifyContent: 'space-between' kya karta hai?**
a) Items ko center mein place karta hai
b) Items ke beech equal space daalta hai
c) Items ko end mein place karta hai
d) Items ko stretch karta hai

### **True/False Questions**

**Q6. React Native mein har View component by default flex container hai.** (True/False)

**Q7. StyleSheet.create() development mein validation provide karta hai.** (True/False)

**Q8. Platform.OS 'ios' ya 'android' return karta hai.** (True/False)

**Q9. alignItems cross axis ke liye use hota hai.** (True/False)

**Q10. Inline styles performance ke liye best practice hain.** (True/False)

### **Code Output Questions**

**Q11. Is code ka output kya hoga?**
```javascript
const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'row',
    justifyContent: 'space-around',
    alignItems: 'center'
  }
});
```

**Q12. Is StyleSheet ka correct syntax kya hai?**
```javascript
// Which one is correct?
// A)
const styles = {
  container: { flex: 1 }
};

// B)
const styles = StyleSheet.create({
  container: { flex: 1 }
});
```

**Q13. Platform-specific styling ka correct syntax kya hai?**
```javascript
// Which one is correct?
// A)
style={{ color: Platform.OS === 'ios' ? 'blue' : 'red' }}

// B)
style={Platform.select({
  ios: { color: 'blue' },
  android: { color: 'red' }
})}
```

---

## 📚 **Answers & Explanations**

### **Multiple Choice Answers**
**Q1.** b) Left to right (default)
**Q2.** d) All of the above
**Q3.** a) Platform.select()
**Q4.** b) Component available space ko occupy karta hai
**Q5.** b) Items ke beech equal space daalta hai

### **True/False Answers**
**Q6.** True
**Q7.** True
**Q8.** True
**Q9.** True
**Q10.** False

### **Code Output Answers**
**Q11.** Row layout with equal spacing around items, centered vertically
**Q12.** B) StyleSheet.create()
**Q13.** B) Platform.select()

---

## 🎯 **Assignment Solutions**

### **Task 1: Social Media Profile Screen Solution**

```javascript
import React, { useState } from 'react';
import {
  View,
  Text,
  Image,
  TouchableOpacity,
  ScrollView,
  Dimensions,
  StyleSheet
} from 'react-native';

const { width } = Dimensions.get('window');
const numColumns = width > 600 ? 3 : 2;

const SocialProfileScreen = () => {
  const [activeTab, setActiveTab] = useState('posts');

  const profile = {
    name: 'John Doe',
    username: '@johndoe',
    avatar: 'https://picsum.photos/150/150',
    bio: 'Mobile app developer passionate about React Native',
    stats: {
      posts: 42,
      followers: 1250,
      following: 380
    }
  };

  const posts = Array.from({ length: 20 }, (_, i) => ({
    id: i + 1,
    image: `https://picsum.photos/200/200?random=${i}`,
    likes: Math.floor(Math.random() * 100),
    comments: Math.floor(Math.random() * 20)
  }));

  const photos = posts.slice(0, 12);
  const videos = posts.slice(12, 18);

  const renderTabContent = () => {
    let data = [];
    switch (activeTab) {
      case 'posts':
        data = posts;
        break;
      case 'photos':
        data = photos;
        break;
      case 'videos':
        data = videos;
        break;
      default:
        data = posts;
    }

    return (
      <View style={styles.grid}>
        {data.map((item) => (
          <TouchableOpacity key={item.id} style={styles.gridItem}>
            <Image source={{ uri: item.image }} style={styles.gridImage} />
            <View style={styles.overlay}>
              <Text style={styles.overlayText}>❤️ {item.likes}</Text>
              <Text style={styles.overlayText}>💬 {item.comments}</Text>
            </View>
          </TouchableOpacity>
        ))}
      </View>
    );
  };

  return (
    <View style={styles.container}>
      {/* Header */}
      <View style={styles.header}>
        <Image source={{ uri: profile.avatar }} style={styles.avatar} />
        <View style={styles.profileInfo}>
          <Text style={styles.name}>{profile.name}</Text>
          <Text style={styles.username}>{profile.username}</Text>
          <Text style={styles.bio}>{profile.bio}</Text>
        </View>
      </View>

      {/* Stats */}
      <View style={styles.stats}>
        <View style={styles.stat}>
          <Text style={styles.statNumber}>{profile.stats.posts}</Text>
          <Text style={styles.statLabel}>Posts</Text>
        </View>
        <View style={styles.stat}>
          <Text style={styles.statNumber}>{profile.stats.followers}</Text>
          <Text style={styles.statLabel}>Followers</Text>
        </View>
        <View style={styles.stat}>
          <Text style={styles.statNumber}>{profile.stats.following}</Text>
          <Text style={styles.statLabel}>Following</Text>
        </View>
      </View>

      {/* Action Buttons */}
      <View style={styles.actions}>
        <TouchableOpacity style={styles.followButton}>
          <Text style={styles.followButtonText}>Follow</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.messageButton}>
          <Text style={styles.messageButtonText}>Message</Text>
        </TouchableOpacity>
      </View>

      {/* Tabs */}
      <View style={styles.tabs}>
        {['posts', 'photos', 'videos'].map((tab) => (
          <TouchableOpacity
            key={tab}
            style={[styles.tab, activeTab === tab && styles.activeTab]}
            onPress={() => setActiveTab(tab)}
          >
            <Text style={[styles.tabText, activeTab === tab && styles.activeTabText]}>
              {tab.charAt(0).toUpperCase() + tab.slice(1)}
            </Text>
          </TouchableOpacity>
        ))}
      </View>

      {/* Content */}
      <ScrollView style={styles.content}>
        {renderTabContent()}
      </ScrollView>

      {/* Floating Action Button */}
      <TouchableOpacity style={styles.fab}>
        <Text style={styles.fabText}>+</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  header: {
    flexDirection: 'row',
    padding: 20,
    alignItems: 'center',
  },
  avatar: {
    width: 80,
    height: 80,
    borderRadius: 40,
    marginRight: 15,
  },
  profileInfo: {
    flex: 1,
  },
  name: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#333',
  },
  username: {
    fontSize: 16,
    color: '#666',
    marginBottom: 5,
  },
  bio: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
  },
  stats: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    paddingVertical: 15,
    borderTopWidth: 1,
    borderBottomWidth: 1,
    borderColor: '#e0e0e0',
  },
  stat: {
    alignItems: 'center',
  },
  statNumber: {
    fontSize: 18,
    fontWeight: 'bold',
    color: '#333',
  },
  statLabel: {
    fontSize: 14,
    color: '#666',
  },
  actions: {
    flexDirection: 'row',
    padding: 15,
  },
  followButton: {
    flex: 1,
    backgroundColor: '#007AFF',
    paddingVertical: 10,
    borderRadius: 5,
    marginRight: 10,
    alignItems: 'center',
  },
  followButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  messageButton: {
    flex: 1,
    backgroundColor: '#f0f0f0',
    paddingVertical: 10,
    borderRadius: 5,
    alignItems: 'center',
  },
  messageButtonText: {
    color: '#333',
    fontSize: 16,
    fontWeight: 'bold',
  },
  tabs: {
    flexDirection: 'row',
    borderBottomWidth: 1,
    borderColor: '#e0e0e0',
  },
  tab: {
    flex: 1,
    paddingVertical: 15,
    alignItems: 'center',
  },
  activeTab: {
    borderBottomWidth: 2,
    borderColor: '#007AFF',
  },
  tabText: {
    fontSize: 16,
    color: '#666',
  },
  activeTabText: {
    color: '#007AFF',
    fontWeight: 'bold',
  },
  content: {
    flex: 1,
  },
  grid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    padding: 5,
  },
  gridItem: {
    width: (width - 10) / numColumns,
    height: (width - 10) / numColumns,
    margin: 2,
    position: 'relative',
  },
  gridImage: {
    width: '100%',
    height: '100%',
  },
  overlay: {
    position: 'absolute',
    bottom: 0,
    left: 0,
    right: 0,
    backgroundColor: 'rgba(0,0,0,0.5)',
    padding: 5,
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
  overlayText: {
    color: 'white',
    fontSize: 12,
    fontWeight: 'bold',
  },
  fab: {
    position: 'absolute',
    bottom: 20,
    right: 20,
    width: 60,
    height: 60,
    borderRadius: 30,
    backgroundColor: '#007AFF',
    justifyContent: 'center',
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 4,
    elevation: 5,
  },
  fabText: {
    color: 'white',
    fontSize: 24,
    fontWeight: 'bold',
  },
});

export default SocialProfileScreen;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **StyleSheet API**: Efficient styling with validation and optimization
- ✅ **Flexbox Layout**: Complete understanding of flex properties and layout
- ✅ **Common Layout Patterns**: Header-content-footer, cards, grids
- ✅ **Responsive Design**: Dimensions API and platform-specific styling
- ✅ **Advanced Techniques**: Dynamic styles, theme support, style composition
- ✅ **Platform-Specific Code**: Platform.select() and conditional styling

### **Best Practices**
1. **Always use StyleSheet.create()** for better performance and validation
2. **Understand flexbox fundamentals** before creating complex layouts
3. **Use Platform.select()** for platform-specific styling
4. **Test layouts on different screen sizes** and orientations
5. **Keep styles organized** and reusable
6. **Use responsive design principles** for better user experience
7. **Avoid inline styles** in production code
8. **Leverage Dimensions API** for responsive layouts

### **Next Steps**
- Learn about React Navigation for multi-screen apps
- Study state management with Context API and Redux
- Explore advanced component patterns and custom hooks
- Practice creating complex layouts and responsive designs

---

## 📚 **Additional Resources**
- [React Native Styling Documentation](https://reactnative.dev/docs/style)
- [Flexbox Guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [Platform-Specific Code](https://reactnative.dev/docs/platform-specific-code)
- [Dimensions API](https://reactnative.dev/docs/dimensions)
- [StyleSheet Reference](https://reactnative.dev/docs/stylesheet)

---

**Next Lesson**: [Lesson 5: State Management with useState & useEffect](Lesson%205_%20State%20Management%20with%20useState%20&%20useEffect.md)