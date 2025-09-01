
# 📝 Assignments & Quizzes: Complete Assessment Guide
## React Native Fundamentals (Lessons 1-10)

---

## 📚 Assessment Overview

Har lesson ke liye comprehensive assignments aur quizzes included hain jo aapke learning ko reinforce karenge aur practical skills develop karenge.

### Assessment Structure:
- **📝 Quizzes**: Multiple choice, true/false, aur short answer questions
- **💻 Coding Assignments**: Practical implementation tasks
- **🏗️ Mini Projects**: Real-world application building
- **🎯 Challenge Tasks**: Advanced problem-solving exercises

---

# 📱 LESSON 1: Introduction to React Native & Environment Setup

## 🧠 Quiz 1: React Native Basics (20 Questions)

### Multiple Choice Questions (1-10)

**Q1. React Native kis company ne develop kiya hai?**
a) Google  
b) Facebook (Meta)  
c) Microsoft  
d) Apple

**Q2. React Native mein kaunsi language primarily use hoti hai?**
a) Java  
b) Swift  
c) JavaScript  
d) Kotlin

**Q3. Cross-platform development ka main advantage kya hai?**
a) Sirf iOS ke liye development  
b) Sirf Android ke liye development  
c) Ek codebase se multiple platforms  
d) Web development only

**Q4. Metro bundler ka primary function kya hai?**
a) App ko bundle karna aur serve karna  
b) Database management  
c) UI design  
d) Testing automation

**Q5. Hot Reloading ka benefit kya hai?**
a) App ki speed badhti hai  
b) Code changes instantly reflect hote hain  
c) Memory usage kam hoti hai  
d) Battery life improve hoti hai

**Q6. React Native CLI install karne ke liye kaunsa command use karte hain?**
a) npm install react-native  
b) npm install -g @react-native-community/cli  
c) npm install -g react-native-cli  
d) Both b and c

**Q7. Android development ke liye kya zaroori hai?**
a) Xcode  
b) Android Studio  
c) Visual Studio  
d) IntelliJ IDEA

**Q8. iOS development sirf kahan possible hai?**
a) Windows  
b) Linux  
c) macOS  
d) Any platform

**Q9. ANDROID_HOME environment variable kya point karta hai?**
a) Android app directory  
b) Android SDK location  
c) Android emulator  
d) Android project folder

**Q10. npx react-native init command kya karta hai?**
a) Existing project run karta hai  
b) New React Native project create karta hai  
c) Dependencies install karta hai  
d) App build karta hai

### True/False Questions (11-15)

**Q11. React Native apps native performance provide karte hain.** (True/False)

**Q12. Expo CLI beginners ke liye recommended hai.** (True/False)

**Q13. React Native sirf mobile apps ke liye use hota hai.** (True/False)

**Q14. iOS development ke liye Windows machine sufficient hai.** (True/False)

**Q15. Hot Reloading production builds mein available hota hai.** (True/False)

### Short Answer Questions (16-20)

**Q16. React Native aur Flutter mein 3 main differences batao.**

**Q17. Development environment setup mein kaunse main steps hain?**

**Q18. Metro bundler ko reset kaise karte hain?**

**Q19. Android emulator start karne ke 2 tarike batao.**

**Q20. React Native project structure mein main folders kaunse hain?**

---

## 💻 Assignment 1: Environment Setup & First App

### Task 1: Complete Environment Setup (60 minutes)
**Objective:** Complete React Native development environment setup

**Requirements:**
1. Install Node.js (latest LTS version)
2. Install React Native CLI
3. Setup Android Studio with required SDKs
4. Configure environment variables (ANDROID_HOME, PATH)
5. Create and run first React Native app
6. Take screenshots of successful setup

**Deliverables:**
- Screenshots of successful installation
- Screenshot of running app on emulator
- Brief report of any issues faced and solutions

### Task 2: Modify Default App (45 minutes)
**Objective:** Customize the default React Native app

**Requirements:**
1. Change app title to "My First React Native App"
2. Add your name and current date
3. Change background color to your favorite color
4. Add a button that shows an alert with your name
5. Add an image (can be from internet)

**Code Template:**
```javascript
import React from 'react';
import {
  SafeAreaView,
  Text,
  View,
  TouchableOpacity,
  Alert,
  Image,
  StyleSheet
} from 'react-native';

const App = () => {
  const showAlert = () => {
    // Your code here
  };

  return (
    <SafeAreaView style={styles.container}>
      {/* Your UI components here */}
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  // Your styles here
});

export default App;
```

### Task 3: Troubleshooting Exercise (30 minutes)
**Objective:** Practice common troubleshooting scenarios

**Scenarios to solve:**
1. Metro bundler not starting
2. Android emulator not detected
3. Build failed due to missing dependencies
4. Port already in use error

**Deliverables:**
- Document each problem and solution
- Create a personal troubleshooting guide

---

# 🔥 LESSON 2: JavaScript ES6+ Essentials

## 🧠 Quiz 2: Modern JavaScript (25 Questions)

### Multiple Choice Questions (1-15)

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

**Q6. Destructuring assignment ka main purpose kya hai?**
a) Objects create karna  
b) Arrays aur objects se values extract karna  
c) Functions define karna  
d) Variables declare karna

**Q7. const variable ko reassign kar sakte hain?**
a) Haan, always  
b) Nahi, never  
c) Sirf objects ke case mein  
d) Sirf arrays ke case mein

**Q8. Promise.all() kya karta hai?**
a) Sirf first promise resolve karta hai  
b) All promises parallel mein run karta hai  
c) Promises ko sequence mein run karta hai  
d) Promises cancel karta hai

**Q9. Array.filter() method kya return karta hai?**
a) Boolean value  
b) Single element  
c) New array with filtered elements  
d) Original array

**Q10. Default parameters kab use hote hain?**
a) Jab function call nahi hota  
b) Jab parameter undefined ya missing ho  
c) Jab parameter null ho  
d) Jab parameter false ho

**Q11. Object.keys() method kya return karta hai?**
a) Object values  
b) Object properties ka array  
c) Object entries  
d) Object length

**Q12. Array.reduce() ka initial value optional hai?**
a) Haan, always optional  
b) Nahi, always required  
c) Depends on array length  
d) Depends on callback function

**Q13. Template literals multi-line strings support karte hain?**
a) Haan  
b) Nahi  
c) Sirf ES2020 mein  
d) Sirf Node.js mein

**Q14. Arrow functions ko constructor ke roop mein use kar sakte hain?**
a) Haan  
b) Nahi  
c) Sirf ES6 mein  
d) Sirf strict mode mein

**Q15. Destructuring sirf objects ke saath kaam karta hai?**
a) Haan, sirf objects  
b) Nahi, arrays aur objects dono  
c) Sirf arrays  
d) Sirf strings

### Code Output Questions (16-20)

**Q16. Is code ka output kya hoga?**
```javascript
const arr = [1, 2, 3];
const newArr = [...arr, 4, 5];
console.log(newArr);
```
a) [1, 2, 3]  
b) [4, 5]  
c) [1, 2, 3, 4, 5]  
d) Error

**Q17. Is code ka output kya hoga?**
```javascript
const user = { name: 'John', age: 25 };
const { name, age, city = 'Mumbai' } = user;
console.log(city);
```
a) undefined  
b) 'Mumbai'  
c) null  
d) Error

**Q18. Is code ka output kya hoga?**
```javascript
const numbers = [1, 2, 3, 4, 5];
const result = numbers.filter(n => n > 3).map(n => n * 2);
console.log(result);
```
a) [2, 4, 6, 8, 10]  
b) [8, 10]  
c) [4, 5]  
d) [1, 2, 3, 8, 10]

**Q19. Is code ka output kya hoga?**
```javascript
const getName = () => 'John';
const getAge = () => 25;
const user = { getName(), getAge() };
console.log(user);
```
a) { getName: 'John', getAge: 25 }  
b) { getName: function, getAge: function }  
c) Error  
d) { getName: undefined, getAge: undefined }

**Q20. Is code ka output kya hoga?**
```javascript
const promise1 = Promise.resolve(1);
const promise2 = Promise.resolve(2);
Promise.all([promise1, promise2]).then(console.log);
```
a) 1  
b) 2  
c) [1, 2]  
d) Error

### Short Answer Questions (21-25)

**Q21. Arrow functions aur regular functions mein 3 main differences batao.**

**Q22. Destructuring assignment ke 3 practical use cases batao.**

**Q23. Promise chain aur async/await mein kya difference hai?**

**Q24. Spread operator ke 4 different use cases batao.**

**Q25. Array methods (map, filter, reduce) ka practical example do.**

---

## 💻 Assignment 2: Modern JavaScript Practice

### Task 1: Array Methods Mastery (60 minutes)
**Objective:** Master essential array methods

**Given Data:**
```javascript
const products = [
  { id: 1, name: 'iPhone', price: 80000, category: 'electronics', inStock: true },
  { id: 2, name: 'Shirt', price: 1500, category: 'clothing', inStock: false },
  { id: 3, name: 'Laptop', price: 60000, category: 'electronics', inStock: true },
  { id: 4, name: 'Jeans', price: 2500, category: 'clothing', inStock: true },
  { id: 5, name: 'Watch', price: 15000, category: 'accessories', inStock: false }
];
```

**Requirements:**
1. Get all product names using map()
2. Filter products under ₹20,000
3. Find the most expensive product
4. Calculate total value of in-stock products using reduce()
5. Group products by category using reduce()
6. Check if any electronics are out of stock using some()
7. Check if all products have prices above ₹1000 using every()
8. Create a new array with 10% discount applied to all prices

**Expected Output Format:**
```javascript
const results = {
  productNames: [...],
  affordableProducts: [...],
  mostExpensive: {...},
  totalInStockValue: number,
  groupedByCategory: {...},
  electronicsOutOfStock: boolean,
  allAbove1000: boolean,
  discountedProducts: [...]
};
```

### Task 2: Async/Await Practice (90 minutes)
**Objective:** Master asynchronous JavaScript

**Requirements:**
Create a weather app component with following features:

1. **fetchWeatherData function:**
   - Use async/await
   - Simulate API call with setTimeout
   - Return weather data object
   - Handle errors properly

2. **WeatherApp component:**
   - Show loading state while fetching
   - Display weather data when loaded
   - Show error message if fetch fails
   - Include refresh functionality

**Code Template:**
```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

// Simulate weather API
const fetchWeatherData = async (city) => {
  // Your implementation here
  // Should return: { city, temperature, condition, humidity }
};

const WeatherApp = () => {
  // Your state management here
  
  // Your functions here
  
  return (
    // Your UI here
  );
};
```

### Task 3: Destructuring & Spread Practice (45 minutes)
**Objective:** Master destructuring and spread operator

**Given Data:**
```javascript
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
```

**Tasks:**
1. Extract name, age, and email using destructuring
2. Extract city and state from nested address
3. Create a new user object with updated age using spread
4. Merge preferences with new settings using spread
5. Create a function that accepts destructured parameters
6. Create an array of user info using destructuring

---

# 🧩 LESSON 3: React Native Components & JSX

## 🧠 Quiz 3: Components & JSX (20 Questions)

### Multiple Choice Questions (1-12)

**Q1. JSX mein multiple root elements return karne ke liye kya use karte hain?**
a) div tag  
b) React.Fragment ya <>  
c) span tag  
d) View component

**Q2. JSX mein JavaScript expressions embed karne ke liye kya use karte hain?**
a) {{ }}  
b) { }  
c) ( )  
d) [ ]

**Q3. React Native mein HTML div tag ke equivalent kya hai?**
a) Text  
b) View  
c) Container  
d) Section

**Q4. Conditional rendering ke liye kaunsa operator commonly use hota hai?**
a) if-else  
b) switch  
c) Ternary operator (?:)  
d) for loop

**Q5. List rendering mein key prop kyu zaroori hai?**
a) Styling ke liye  
b) Performance optimization ke liye  
c) Event handling ke liye  
d) Navigation ke liye

**Q6. Functional component mein props kaise access karte hain?**
a) this.props  
b) props parameter  
c) getProps()  
d) useProps()

**Q7. Component composition ka matlab kya hai?**
a) Components ko combine karna  
b) Components ko delete karna  
c) Components ko style karna  
d) Components ko test karna

**Q8. Higher-Order Component (HOC) kya hai?**
a) Component jo dusre components return karta hai  
b) Component jo props accept karta hai  
c) Component jo state manage karta hai  
d) Component jo events handle karta hai

**Q9. React.memo() ka use kab karte hain?**
a) State management ke liye  
b) Performance optimization ke liye  
c) Event handling ke liye  
d) Styling ke liye

**Q10. Default props set karne ka correct tarika kya hai?**
a) Component.defaultProps  
b) Component.props.default  
c) Default parameter values  
d) Both a and c

**Q11. JSX mein comments kaise likhte hain?**
a) // comment  
b) /* comment */  
c) {/* comment */}  
d) <!-- comment -->

**Q12. Fragment ka short syntax kya hai?**
a) <Fragment>  
b) <>  
c) <React.Fragment>  
d) <div>

### True/False Questions (13-16)

**Q13. JSX mein class attribute use kar sakte hain.** (True/False)

**Q14. React components ka naam capital letter se start hona chahiye.** (True/False)

**Q15. JSX compile hone ke baad JavaScript functions ban jate hain.** (True/False)

**Q16. Functional components mein lifecycle methods use kar sakte hain.** (True/False)

### Short Answer Questions (17-20)

**Q17. JSX aur HTML mein 3 main differences batao.**

**Q18. Component composition ke 3 benefits batao.**

**Q19. Props aur state mein kya difference hai?**

**Q20. Higher-Order Component ka practical example do.**

---

## 💻 Assignment 3: Component Building

### Task 1: Custom Component Library (120 minutes)
**Objective:** Create reusable custom components

**Requirements:**
Create following custom components:

1. **CustomButton Component:**
   - Props: title, onPress, backgroundColor, textColor, disabled
   - Default styling
   - Loading state support
   - Different sizes (small, medium, large)

2. **CustomCard Component:**
   - Props: title, content, image, onPress
   - Shadow styling
   - Responsive design
   - Optional image

3. **CustomInput Component:**
   - Props: label, placeholder, value, onChangeText, error
   - Validation styling
   - Icon support
   - Different input types

4. **CustomModal Component:**
   - Props: visible, onClose, title, children
   - Backdrop press to close
   - Animation support
   - Custom styling

**Code Structure:**
```
components/
├── CustomButton.js
├── CustomCard.js
├── CustomInput.js
├── CustomModal.js
└── index.js (export all components)
```

### Task 2: Component Composition (90 minutes)
**Objective:** Build complex UI using component composition

**Requirements:**
Create a User Profile Screen using your custom components:

1. **UserProfile Component:**
   - Use CustomCard for profile info
   - Use CustomButton for actions
   - Use CustomInput for editing
   - Use CustomModal for confirmations

2. **Features to implement:**
   - Display user information
   - Edit profile functionality
   - Delete account confirmation
   - Save/Cancel actions

**Expected UI:**
- Profile picture placeholder
- User details (name, email, phone)
- Edit button
- Delete account button
- Modal for confirmations

### Task 3: Component Optimization (60 minutes)
**Objective:** Optimize components for performance

**Requirements:**
1. Add React.memo to appropriate components
2. Implement proper prop validation
3. Add default props where needed
4. Create HOC for loading states
5. Implement error boundaries

**Performance Checklist:**
- [ ] Components wrapped with React.memo where appropriate
- [ ] Props properly validated
- [ ] Default props defined
- [ ] HOC implemented for common functionality
- [ ] Error boundary implemented

---

# 🎨 LESSON 4: Styling in React Native

## 🧠 Quiz 4: Styling & Flexbox (25 Questions)

### Multiple Choice Questions (1-15)

**Q1. React Native mein styling kaise karte hain?**
a) CSS files  
b) JavaScript objects  
c) HTML attributes  
d) External stylesheets

**Q2. StyleSheet.create() use karne ka main benefit kya hai?**
a) Better performance  
b) Style validation  
c) Code organization  
d) All of the above

**Q3. React Native mein default flex direction kya hai?**
a) row  
b) column  
c) row-reverse  
d) column-reverse

**Q4. justifyContent property main axis ko control karta hai ya cross axis ko?**
a) Main axis  
b) Cross axis  
c) Both  
d) Neither

**Q5. alignItems property kaunsa axis control karta hai?**
a) Main axis  
b) Cross axis  
c) Both  
d) Neither

**Q6. flex: 1 ka matlab kya hai?**
a) 1px width/height  
b) Available space ka 1 part  
c) 1% of parent  
d) Fixed size of 1

**Q7. Responsive design ke liye kaunsa API use karte hain?**
a) Screen API  
b) Dimensions API  
c) Window API  
d) Device API

**Q8. Platform-specific styling ke liye kya use karte hain?**
a) Platform.OS  
b) Platform.select()  
c) Both a and b  
d) Neither

**Q9. Shadow effect sirf kahan kaam karta hai?**
a) Android only  
b) iOS only  
c) Both platforms  
d) Web only

**Q10. Elevation property kahan use hota hai?**
a) iOS only  
b) Android only  
c) Both platforms  
d) Web only

**Q11. Absolute positioning ke liye kya use karte hain?**
a) position: 'absolute'  
b) position: 'fixed'  
c) position: 'relative'  
d) position: 'static'

**Q12. Border radius sirf corners ko affect karta hai?**
a) Haan  
b) Nahi  
c) Depends on shape  
d) Only on iOS

**Q13. Text styling properties View component mein kaam karti hain?**
a) Haan  
b) Nahi  
c) Sometimes  
d) Only fontSize

**Q14. Flexbox mein space-between kya karta hai?**
a) Equal space around items  
b) Equal space between items  
c) No space between items  
d) Maximum space between items

**Q15. alignSelf property kya karta hai?**
a) All children ko align karta hai  
b) Single child ko align karta hai  
c) Parent ko align karta hai  
d) Text ko align karta hai

### Flexbox Layout Questions (16-20)

**Q16. Is layout ka output kya hoga?**
```javascript
<View style={{
  flexDirection: 'row',
  justifyContent: 'space-between',
  alignItems: 'center'
}}>
  <View style={{width: 50, height: 50, backgroundColor: 'red'}} />
  <View style={{width: 50, height: 50, backgroundColor: 'blue'}} />
</View>
```
a) Items left mein aligned  
b) Items right mein aligned  
c) Items equally spaced with space between  
d) Items overlapping

**Q17. flex: 1 aur flex: 2 wale do components ka size ratio kya hoga?**
a) 1:1  
b) 1:2  
c) 2:1  
d) Equal size

**Q18. alignItems: 'stretch' kya karta hai?**
a) Items ko main axis mein stretch karta hai  
b) Items ko cross axis mein stretch karta hai  
c) Items ko both axes mein stretch karta hai  
d) Kuch nahi karta

**Q19. flexWrap: 'wrap' kab use karte hain?**
a) Items ko next line mein wrap karne ke liye  
b) Items ko hide karne ke liye  
c) Items ko overlap karne ke liye  
d) Items ko center karne ke liye

**Q20. position: 'absolute' use karne se kya hota hai?**
a) Element normal flow mein rehta hai  
b) Element normal flow se bahar ho jata hai  
c) Element invisible ho jata hai  
d) Element parent ke center mein aa jata hai

### Short Answer Questions (21-25)

**Q21. Flexbox ke main concepts (main axis, cross axis) explain karo.**

**Q22. Responsive design ke liye 3 techniques batao.**

**Q23. Platform-specific styling kaise implement karte hain?**

**Q24. Shadow aur elevation mein kya difference hai?**

**Q25. StyleSheet.create() vs inline styling mein kya difference hai?**

---

## 💻 Assignment 4: Advanced Styling

### Task 1: Flexbox Mastery (90 minutes)
**Objective:** Master flexbox layouts

**Requirements:**
Create following layouts using flexbox:

1. **Header-Content-Footer Layout:**
   - Fixed header (60px height)
   - Flexible content area
   - Fixed footer (50px height)

2. **Card Grid Layout:**
   - 2 columns on phone
   - 3 columns on tablet
   - Equal height cards
   - Responsive spacing

3. **Navigation Bar:**
   - Horizontal layout
   - Icons with labels
   - Equal spacing
   - Active state styling

4. **Profile Layout:**
   - Avatar on left
   - User info on right
   - Action buttons at bottom
   - Responsive design

**Code Template:**
```javascript
const FlexboxLayouts = () => {
  return (
    <ScrollView style={styles.container}>
      {/* Header-Content-Footer */}
      <View style={styles.layout1}>
        {/* Your implementation */}
      </View>
      
      {/* Card Grid */}
      <View style={styles.layout2}>
        {/* Your implementation */}
      </View>
      
      {/* Navigation Bar */}
      <View style={styles.layout3}>
        {/* Your implementation */}
      </View>
      
      {/* Profile Layout */}
      <View style={styles.layout4}>
        {/* Your implementation */}
      </View>
    </ScrollView>
  );
};
```

### Task 2: Responsive Design (75 minutes)
**Objective:** Create responsive layouts

**Requirements:**
1. **Responsive Text Component:**
   - Font size adapts to screen size
   - Line height adjusts accordingly
   - Minimum and maximum font sizes

2. **Responsive Grid:**
   - 1 column on small screens (<400px)
   - 2 columns on medium screens (400-600px)
   - 3 columns on large screens (>600px)

3. **Responsive Navigation:**
   - Bottom tabs on phones
   - Side drawer on tablets
   - Automatic switching based on screen size

**Implementation Guide:**
```javascript
import { Dimensions } from 'react-native';

const { width, height } = Dimensions.get('window');
const isTablet = width > 768;
const isSmallScreen = width < 400;

// Use these variables for responsive styling
```

### Task 3: Advanced Styling Techniques (60 minutes)
**Objective:** Implement advanced styling patterns

**Requirements:**
1. **Theme System:**
   - Light and dark themes
   - Color constants
   - Theme switching functionality

2. **Animation Styles:**
   - Hover effects (for touchable components)
   - Loading animations
   - Transition effects

3. **Platform-Specific Styling:**
   - iOS vs Android differences
   - Platform.select() usage
   - Conditional styling

**Theme Structure:**
```javascript
const themes = {
  light: {
    backgroundColor: '#ffffff',
    textColor: '#000000',
    primaryColor: '#007AFF',
    // ... more colors
  },
  dark: {
    backgroundColor: '#000000',
    textColor: '#ffffff',
    primaryColor: '#0A84FF',
    // ... more colors
  }
};
```

---

# ⚡ LESSON 5: State Management with useState & useEffect

## 🧠 Quiz 5: Hooks & State Management (30 Questions)

### Multiple Choice Questions (1-20)

**Q1. useState hook kya return karta hai?**
a) Current state value  
b) State setter function  
c) Array with state value and setter  
d) Object with state properties

**Q2. useEffect hook kab run hota hai?**
a) Component mount hone pe  
b) Component update hone pe  
c) Component unmount hone pe  
d) All of the above (depends on dependencies)

**Q3. Empty dependency array [] ka matlab kya hai useEffect mein?**
a) Effect har render pe chalega  
b) Effect sirf mount pe chalega  
c) Effect kabhi nahi chalega  
d) Effect sirf unmount pe chalega

**Q4. State update function asynchronous hai ya synchronous?**
a) Synchronous  
b) Asynchronous  
c) Depends on state type  
d) Depends on component type

**Q5. Functional update pattern kab use karte hain?**
a) Jab previous state ki zaroorat ho  
b) Jab state object ho  
c) Jab state array ho  
d) Jab state string ho

**Q6. useEffect mein cleanup function kab run hota hai?**
a) Component mount hone pe  
b) Component update hone pe  
c) Component unmount hone pe  
d) Effect re-run hone se pehle

**Q7. Multiple useState hooks ek component mein use kar sakte hain?**
a) Nahi  
b) Haan, unlimited  
c) Maximum 5  
d) Maximum 10

**Q8. State update immediately reflect hota hai?**
a) Haan, immediately  
b) Nahi, next render mein  
c) Depends on state type  
d) Only in development mode

**Q9. useEffect mein dependency array mein kya include karna chahiye?**
a) All variables used in effect  
b) Only state variables  
c) Only props  
d) Nothing

**Q10. Infinite loop se bachne ke liye kya karna chahiye?**
a) useEffect use na karo  
b) Proper dependency array use karo  
c) State update na karo  
d) Component unmount karo

**Q11. Object state update karne ka correct tarika kya hai?**
a) setState(newObject)  
b) setState({...prevState, newProperty})  
c) setState(prevState.property = newValue)  
d) setState(Object.assign(prevState, newObject))

**Q12. Array state mein new item add karne ka correct tarika kya hai?**
a) setState(prevArray.push(newItem))  
b) setState([...prevArray, newItem])  
c) setState(prevArray.concat(newItem))  
d) Both b and c

**Q13. useEffect mein async function directly use kar sakte hain?**
a) Haan  
b) Nahi  
c) Sirf ES2020 mein  
d) Sirf development mein

**Q14. State update ke baad immediately console.log karne se kya milega?**
a) Updated state  
b) Previous state  
c) Undefined  
d) Error

**Q15. Custom hook banane ka main benefit kya hai?**
a) Performance improvement  
b) Code reusability  
c) Better styling  
d) Faster rendering

**Q16. useEffect cleanup function return karna zaroori hai?**
a) Haan, always  
b) Nahi, optional