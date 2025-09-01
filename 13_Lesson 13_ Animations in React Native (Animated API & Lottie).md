# Lesson 13: Animations in React Native (Animated API & Lottie)

## 🎯 **Learning Objectives**
- Master React Native's Animated API
- Implement complex animations with timing and spring
- Use Lottie animations for professional effects
- Create smooth transitions and micro-interactions
- Optimize animation performance

## 📚 **Table of Contents**
1. [Introduction to Animations](#introduction-to-animations)
2. [Animated API Basics](#animated-api-basics)
3. [Timing and Spring Animations](#timing-and-spring-animations)
4. [Interpolation and Transform](#interpolation-and-transform)
5. [Lottie Integration](#lottie-integration)
6. [Advanced Animation Techniques](#advanced-animation-techniques)
7. [Performance Optimization](#performance-optimization)
8. [Practical Examples](#practical-examples)

---

## 🎬 **Introduction to Animations**

Animations make mobile apps feel alive and provide visual feedback to users. React Native offers powerful animation capabilities through the Animated API and third-party libraries like Lottie.

### **Why Animations Matter**
- **User Experience**: Smooth transitions and feedback
- **Visual Hierarchy**: Guide user attention
- **Professional Feel**: Modern apps use animations extensively
- **Performance**: Hardware-accelerated animations

---

## 🎭 **Animated API Basics**

### **Core Components**
```javascript
import { Animated, View, Text } from 'react-native';

// Create animated values
const fadeAnim = new Animated.Value(0);
const slideAnim = new Animated.Value(0);
```

### **Basic Fade Animation**
```javascript
import React, { useEffect, useRef } from 'react';
import { Animated, View, Text, StyleSheet } from 'react-native';

const FadeInView = ({ children }) => {
  const fadeAnim = useRef(new Animated.Value(0)).current;

  useEffect(() => {
    Animated.timing(fadeAnim, {
      toValue: 1,
      duration: 1000,
      useNativeDriver: true,
    }).start();
  }, [fadeAnim]);

  return (
    <Animated.View style={{ opacity: fadeAnim }}>
      {children}
    </Animated.View>
  );
};

const App = () => {
  return (
    <View style={styles.container}>
      <FadeInView>
        <Text style={styles.text}>Fading in!</Text>
      </FadeInView>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  text: {
    fontSize: 24,
    fontWeight: 'bold',
  },
});

export default App;
```

---

## ⏱️ **Timing and Spring Animations**

### **Timing Animation**
```javascript
const timingAnimation = () => {
  Animated.timing(slideAnim, {
    toValue: 100,
    duration: 500,
    easing: Easing.bounce,
    useNativeDriver: true,
  }).start();
};
```

### **Spring Animation**
```javascript
const springAnimation = () => {
  Animated.spring(bounceAnim, {
    toValue: 1,
    friction: 3,
    tension: 40,
    useNativeDriver: true,
  }).start();
};
```

### **Combined Animation Example**
```javascript
import React, { useRef, useEffect } from 'react';
import { Animated, View, Text, TouchableOpacity, StyleSheet, Easing } from 'react-native';

const CombinedAnimation = () => {
  const scaleAnim = useRef(new Animated.Value(1)).current;
  const rotateAnim = useRef(new Animated.Value(0)).current;
  const translateAnim = useRef(new Animated.Value(0)).current;

  const startAnimation = () => {
    Animated.parallel([
      Animated.spring(scaleAnim, {
        toValue: 1.5,
        friction: 3,
        useNativeDriver: true,
      }),
      Animated.timing(rotateAnim, {
        toValue: 1,
        duration: 1000,
        easing: Easing.linear,
        useNativeDriver: true,
      }),
      Animated.timing(translateAnim, {
        toValue: 100,
        duration: 1000,
        easing: Easing.bounce,
        useNativeDriver: true,
      }),
    ]).start(() => {
      // Reset animations
      scaleAnim.setValue(1);
      rotateAnim.setValue(0);
      translateAnim.setValue(0);
    });
  };

  const rotate = rotateAnim.interpolate({
    inputRange: [0, 1],
    outputRange: ['0deg', '360deg'],
  });

  return (
    <View style={styles.container}>
      <Animated.View
        style={{
          transform: [
            { scale: scaleAnim },
            { rotate },
            { translateX: translateAnim },
          ],
        }}
      >
        <TouchableOpacity style={styles.box} onPress={startAnimation}>
          <Text style={styles.text}>Tap to Animate!</Text>
        </TouchableOpacity>
      </Animated.View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  box: {
    width: 100,
    height: 100,
    backgroundColor: '#3498db',
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 10,
  },
  text: {
    color: 'white',
    fontWeight: 'bold',
  },
});

export default CombinedAnimation;
```

---

## 🔄 **Interpolation and Transform**

### **Color Interpolation**
```javascript
const colorAnim = useRef(new Animated.Value(0)).current;

const backgroundColor = colorAnim.interpolate({
  inputRange: [0, 1],
  outputRange: ['#3498db', '#e74c3c'],
});
```

### **Complex Transform Example**
```javascript
import React, { useRef, useEffect } from 'react';
import { Animated, View, StyleSheet } from 'react-native';

const TransformExample = () => {
  const animValue = useRef(new Animated.Value(0)).current;

  useEffect(() => {
    const animate = () => {
      Animated.sequence([
        Animated.timing(animValue, {
          toValue: 1,
          duration: 2000,
          useNativeDriver: true,
        }),
        Animated.timing(animValue, {
          toValue: 0,
          duration: 2000,
          useNativeDriver: true,
        }),
      ]).start(() => animate());
    };
    animate();
  }, []);

  const translateX = animValue.interpolate({
    inputRange: [0, 1],
    outputRange: [0, 200],
  });

  const scale = animValue.interpolate({
    inputRange: [0, 0.5, 1],
    outputRange: [1, 1.5, 1],
  });

  const rotate = animValue.interpolate({
    inputRange: [0, 1],
    outputRange: ['0deg', '180deg'],
  });

  const opacity = animValue.interpolate({
    inputRange: [0, 0.5, 1],
    outputRange: [1, 0.3, 1],
  });

  return (
    <View style={styles.container}>
      <Animated.View
        style={{
          transform: [
            { translateX },
            { scale },
            { rotate },
          ],
          opacity,
        }}
      >
        <View style={styles.box} />
      </Animated.View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  box: {
    width: 100,
    height: 100,
    backgroundColor: '#9b59b6',
    borderRadius: 10,
  },
});

export default TransformExample;
```

---

## 🎨 **Lottie Integration**

### **Installation**
```bash
npm install lottie-react-native
```

### **Basic Lottie Animation**
```javascript
import React from 'react';
import { View, StyleSheet } from 'react-native';
import LottieView from 'lottie-react-native';

const LottieExample = () => {
  return (
    <View style={styles.container}>
      <LottieView
        source={require('./assets/animation.json')}
        autoPlay
        loop
        style={styles.animation}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  animation: {
    width: 200,
    height: 200,
  },
});

export default LottieExample;
```

### **Controlled Lottie Animation**
```javascript
import React, { useRef } from 'react';
import { View, TouchableOpacity, Text, StyleSheet } from 'react-native';
import LottieView from 'lottie-react-native';

const ControlledLottie = () => {
  const animationRef = useRef(null);

  const playAnimation = () => {
    animationRef.current?.play();
  };

  const pauseAnimation = () => {
    animationRef.current?.pause();
  };

  const resetAnimation = () => {
    animationRef.current?.reset();
  };

  return (
    <View style={styles.container}>
      <LottieView
        ref={animationRef}
        source={require('./assets/loading.json')}
        loop={false}
        style={styles.animation}
      />

      <View style={styles.controls}>
        <TouchableOpacity style={styles.button} onPress={playAnimation}>
          <Text style={styles.buttonText}>Play</Text>
        </TouchableOpacity>

        <TouchableOpacity style={styles.button} onPress={pauseAnimation}>
          <Text style={styles.buttonText}>Pause</Text>
        </TouchableOpacity>

        <TouchableOpacity style={styles.button} onPress={resetAnimation}>
          <Text style={styles.buttonText}>Reset</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  animation: {
    width: 150,
    height: 150,
  },
  controls: {
    flexDirection: 'row',
    marginTop: 20,
  },
  button: {
    backgroundColor: '#3498db',
    padding: 10,
    margin: 5,
    borderRadius: 5,
  },
  buttonText: {
    color: 'white',
    fontWeight: 'bold',
  },
});

export default ControlledLottie;
```

---

## 🚀 **Advanced Animation Techniques**

### **Gesture-Based Animations**
```javascript
import React, { useRef } from 'react';
import { Animated, PanResponder, View, StyleSheet } from 'react-native';

const GestureAnimation = () => {
  const pan = useRef(new Animated.ValueXY()).current;

  const panResponder = useRef(
    PanResponder.create({
      onMoveShouldSetPanResponder: () => true,
      onPanResponderGrant: () => {
        pan.setOffset({
          x: pan.x._value,
          y: pan.y._value,
        });
      },
      onPanResponderMove: Animated.event(
        [null, { dx: pan.x, dy: pan.y }],
        { useNativeDriver: false }
      ),
      onPanResponderRelease: () => {
        pan.flattenOffset();
        Animated.spring(pan, {
          toValue: { x: 0, y: 0 },
          useNativeDriver: false,
        }).start();
      },
    })
  ).current;

  return (
    <View style={styles.container}>
      <Animated.View
        style={{
          transform: [{ translateX: pan.x }, { translateY: pan.y }],
        }}
        {...panResponder.panHandlers}
      >
        <View style={styles.box} />
      </Animated.View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  box: {
    width: 100,
    height: 100,
    backgroundColor: '#e67e22',
    borderRadius: 10,
  },
});

export default GestureAnimation;
```

### **Staggered Animations**
```javascript
const staggeredAnimation = () => {
  const animations = items.map((item, index) => {
    return Animated.timing(item.animValue, {
      toValue: 1,
      duration: 500,
      delay: index * 100,
      useNativeDriver: true,
    });
  });

  Animated.stagger(100, animations).start();
};
```

---

## ⚡ **Performance Optimization**

### **Use Native Driver**
```javascript
// ✅ Good - Uses native driver
Animated.timing(opacity, {
  toValue: 1,
  duration: 500,
  useNativeDriver: true, // Hardware accelerated
}).start();

// ❌ Bad - Doesn't use native driver
Animated.timing(marginLeft, {
  toValue: 100,
  duration: 500,
  // useNativeDriver: false, // Layout animation (slow)
}).start();
```

### **Avoid Layout Animations**
```javascript
// ✅ Use transform instead of layout properties
const translateX = animValue.interpolate({
  inputRange: [0, 1],
  outputRange: [0, 100],
});

// Use transform: [{ translateX }]
// Instead of left: translateX (layout property)
```

### **Animation Cleanup**
```javascript
useEffect(() => {
  const animation = Animated.timing(fadeAnim, {
    toValue: 1,
    duration: 1000,
    useNativeDriver: true,
  });

  animation.start();

  return () => {
    animation.stop(); // Cleanup
  };
}, []);
```

---

## 🎯 **Practical Examples**

### **Loading Screen Animation**
```javascript
import React, { useEffect, useRef } from 'react';
import { Animated, View, Text, StyleSheet } from 'react-native';

const LoadingScreen = () => {
  const fadeAnim = useRef(new Animated.Value(0)).current;
  const scaleAnim = useRef(new Animated.Value(0.5)).current;
  const rotateAnim = useRef(new Animated.Value(0)).current;

  useEffect(() => {
    Animated.loop(
      Animated.parallel([
        Animated.timing(fadeAnim, {
          toValue: 1,
          duration: 1000,
          useNativeDriver: true,
        }),
        Animated.spring(scaleAnim, {
          toValue: 1,
          friction: 3,
          useNativeDriver: true,
        }),
        Animated.timing(rotateAnim, {
          toValue: 1,
          duration: 2000,
          useNativeDriver: true,
        }),
      ])
    ).start();
  }, []);

  const rotate = rotateAnim.interpolate({
    inputRange: [0, 1],
    outputRange: ['0deg', '360deg'],
  });

  return (
    <View style={styles.container}>
      <Animated.View
        style={{
          opacity: fadeAnim,
          transform: [
            { scale: scaleAnim },
            { rotate },
          ],
        }}
      >
        <View style={styles.loader}>
          <Text style={styles.text}>Loading...</Text>
        </View>
      </Animated.View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f5f5f5',
  },
  loader: {
    width: 120,
    height: 120,
    backgroundColor: '#3498db',
    borderRadius: 60,
    justifyContent: 'center',
    alignItems: 'center',
  },
  text: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default LoadingScreen;
```

### **Card Flip Animation**
```javascript
import React, { useRef } from 'react';
import { Animated, TouchableOpacity, View, Text, StyleSheet } from 'react-native';

const CardFlip = () => {
  const flipAnim = useRef(new Animated.Value(0)).current;

  const flipCard = () => {
    Animated.spring(flipAnim, {
      toValue: flipAnim._value === 0 ? 1 : 0,
      friction: 8,
      tension: 10,
      useNativeDriver: true,
    }).start();
  };

  const frontInterpolate = flipAnim.interpolate({
    inputRange: [0, 1],
    outputRange: ['0deg', '180deg'],
  });

  const backInterpolate = flipAnim.interpolate({
    inputRange: [0, 1],
    outputRange: ['180deg', '360deg'],
  });

  const frontOpacity = flipAnim.interpolate({
    inputRange: [0, 0.5, 1],
    outputRange: [1, 0, 0],
  });

  const backOpacity = flipAnim.interpolate({
    inputRange: [0, 0.5, 1],
    outputRange: [0, 0, 1],
  });

  return (
    <View style={styles.container}>
      <TouchableOpacity onPress={flipCard}>
        <View>
          {/* Front of card */}
          <Animated.View
            style={{
              transform: [{ rotateY: frontInterpolate }],
              opacity: frontOpacity,
            }}
          >
            <View style={[styles.card, styles.frontCard]}>
              <Text style={styles.cardText}>Front Side</Text>
            </View>
          </Animated.View>

          {/* Back of card */}
          <Animated.View
            style={{
              transform: [{ rotateY: backInterpolate }],
              opacity: backOpacity,
              position: 'absolute',
              top: 0,
            }}
          >
            <View style={[styles.card, styles.backCard]}>
              <Text style={styles.cardText}>Back Side</Text>
            </View>
          </Animated.View>
        </View>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  card: {
    width: 200,
    height: 300,
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 10,
    backfaceVisibility: 'hidden',
  },
  frontCard: {
    backgroundColor: '#3498db',
  },
  backCard: {
    backgroundColor: '#e74c3c',
  },
  cardText: {
    color: 'white',
    fontSize: 24,
    fontWeight: 'bold',
  },
});

export default CardFlip;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Animated API**: Core animation system in React Native
- ✅ **Timing & Spring**: Different animation types and configurations
- ✅ **Interpolation**: Transforming animation values
- ✅ **Lottie Integration**: Professional animation library
- ✅ **Performance**: Native driver and optimization techniques
- ✅ **Gesture Animations**: Interactive animations
- ✅ **Complex Sequences**: Combining multiple animations

### **Best Practices**
1. **Always use `useNativeDriver: true`** for better performance
2. **Prefer transforms over layout properties** for animations
3. **Use interpolation** for complex value transformations
4. **Clean up animations** in useEffect return functions
5. **Test on real devices** for accurate performance

### **Next Steps**
- Practice with different animation combinations
- Experiment with Lottie animations
- Create custom animation hooks
- Implement gesture-based interactions

---

## 🎯 **Assignment**

### **Task 1: Create a Loading Animation**
Build a multi-step loading animation with:
- Spinning indicator
- Pulsing text
- Progress bar animation
- Fade in/out transitions

### **Task 2: Interactive Card Animation**
Create an interactive card that:
- Scales on press
- Changes color smoothly
- Shows/hides content with animation
- Has a bounce effect on release

### **Task 3: Lottie Integration**
- Download Lottie animations from LottieFiles
- Implement them in your app
- Control playback programmatically
- Add user interaction controls

---

## 📚 **Additional Resources**
- [React Native Animated API Documentation](https://reactnative.dev/docs/animated)
- [Lottie Documentation](https://airbnb.io/lottie/)
- [React Native Animation Examples](https://github.com/facebook/react-native/tree/main/packages/rn-tester/js/examples/Animated)
- [LottieFiles](https://lottiefiles.com/) - Free animations library

---

**Next Lesson**: [Lesson 14: Native Modules & Platform-Specific Code](Lesson%2014_%20Native%20Modules%20&%20Platform-Specific%20Code.md)