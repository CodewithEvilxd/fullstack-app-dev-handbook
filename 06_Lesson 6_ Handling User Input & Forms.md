# 📝 Lesson 6: Handling User Input & Forms

## 🎯 Learning Objectives
Is lesson ke baad aap:
- TextInput component master kar sakte hain
- Form validation implement kar sakte hain
- Different input types handle kar sakte hain
- User input ko efficiently manage kar sakte hain
- Complex forms create kar sakte hain

---

## 📖 TextInput Component Basics

TextInput React Native ka primary component hai user input handle karne ke liye.

### 1. Basic TextInput

```javascript
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  Alert
} from 'react-native';

const BasicInputExample = () => {
  const [text, setText] = useState('');
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = () => {
    Alert.alert('Form Data', `Text: ${text}\nEmail: ${email}\nPassword: ${password}`);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Basic Input Examples</Text>

      {/* Basic Text Input */}
      <View style={styles.inputContainer}>
        <Text style={styles.label}>Name:</Text>
        <TextInput
          style={styles.input}
          placeholder="Enter your name"
          value={text}
          onChangeText={setText}
          autoCapitalize="words"
          autoCorrect={false}
        />
      </View>

      {/* Email Input */}
      <View style={styles.inputContainer}>
        <Text style={styles.label}>Email:</Text>
        <TextInput
          style={styles.input}
          placeholder="Enter your email"
          value={email}
          onChangeText={setEmail}
          keyboardType="email-address"
          autoCapitalize="none"
          autoCorrect={false}
          autoCompleteType="email"
        />
      </View>

      {/* Password Input */}
      <View style={styles.inputContainer}>
        <Text style={styles.label}>Password:</Text>
        <TextInput
          style={styles.input}
          placeholder="Enter your password"
          value={password}
          onChangeText={setPassword}
          secureTextEntry={true}
          autoCapitalize="none"
          autoCorrect={false}
        />
      </View>

      <TouchableOpacity style={styles.button} onPress={handleSubmit}>
        <Text style={styles.buttonText}>Submit</Text>
      </TouchableOpacity>
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
  inputContainer: {
    marginBottom: 20,
  },
  label: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 8,
    color: '#333',
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    paddingHorizontal: 15,
    paddingVertical: 12,
    fontSize: 16,
    backgroundColor: 'white',
  },
  button: {
    backgroundColor: '#007AFF',
    paddingVertical: 15,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 20,
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
});

export default BasicInputExample;
```

### 2. TextInput Properties

```javascript
const TextInputPropertiesExample = () => {
  const [multilineText, setMultilineText] = useState('');
  const [numericValue, setNumericValue] = useState('');
  const [phoneNumber, setPhoneNumber] = useState('');

  return (
    <View style={styles.container}>
      <Text style={styles.title}>TextInput Properties</Text>

      {/* Multiline Input */}
      <View style={styles.inputContainer}>
        <Text style={styles.label}>Description (Multiline):</Text>
        <TextInput
          style={[styles.input, styles.multilineInput]}
          placeholder="Enter description..."
          value={multilineText}
          onChangeText={setMultilineText}
          multiline={true}
          numberOfLines={4}
          textAlignVertical="top"
        />
      </View>

      {/* Numeric Input */}
      <View style={styles.inputContainer}>
        <Text style={styles.label}>Age (Numeric):</Text>
        <TextInput
          style={styles.input}
          placeholder="Enter your age"
          value={numericValue}
          onChangeText={setNumericValue}
          keyboardType="numeric"
          maxLength={3}
        />
      </View>

      {/* Phone Number Input */}
      <View style={styles.inputContainer}>
        <Text style={styles.label}>Phone Number:</Text>
        <TextInput
          style={styles.input}
          placeholder="Enter phone number"
          value={phoneNumber}
          onChangeText={setPhoneNumber}
          keyboardType="phone-pad"
          dataDetectorTypes="phoneNumber"
        />
      </View>

      {/* Input with Character Limit */}
      <View style={styles.inputContainer}>
        <Text style={styles.label}>Bio (Max 100 characters):</Text>
        <TextInput
          style={styles.input}
          placeholder="Tell us about yourself..."
          maxLength={100}
          multiline={true}
        />
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  // ... previous styles
  multilineInput: {
    height: 100,
    textAlignVertical: 'top',
  },
});
```

---

## ✅ Form Validation

### 1. Basic Validation

```javascript
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  Alert
} from 'react-native';

const FormValidationExample = () => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    password: '',
    confirmPassword: ''
  });

  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  // Validation functions
  const validateName = (name) => {
    if (!name.trim()) {
      return 'Name is required';
    }
    if (name.trim().length < 2) {
      return 'Name must be at least 2 characters';
    }
    return '';
  };

  const validateEmail = (email) => {
    if (!email.trim()) {
      return 'Email is required';
    }
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      return 'Please enter a valid email';
    }
    return '';
  };

  const validatePassword = (password) => {
    if (!password) {
      return 'Password is required';
    }
    if (password.length < 6) {
      return 'Password must be at least 6 characters';
    }
    if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(password)) {
      return 'Password must contain uppercase, lowercase, and number';
    }
    return '';
  };

  const validateConfirmPassword = (confirmPassword, password) => {
    if (!confirmPassword) {
      return 'Please confirm your password';
    }
    if (confirmPassword !== password) {
      return 'Passwords do not match';
    }
    return '';
  };

  // Handle input change with validation
  const handleInputChange = (field, value) => {
    setFormData(prev => ({ ...prev, [field]: value }));

    // Clear error when user starts typing
    if (errors[field]) {
      setErrors(prev => ({ ...prev, [field]: '' }));
    }
  };

  // Validate single field
  const validateField = (field, value) => {
    let error = '';

    switch (field) {
      case 'name':
        error = validateName(value);
        break;
      case 'email':
        error = validateEmail(value);
        break;
      case 'password':
        error = validatePassword(value);
        break;
      case 'confirmPassword':
        error = validateConfirmPassword(value, formData.password);
        break;
    }

    setErrors(prev => ({ ...prev, [field]: error }));
    return error === '';
  };

  // Validate entire form
  const validateForm = () => {
    const newErrors = {
      name: validateName(formData.name),
      email: validateEmail(formData.email),
      password: validatePassword(formData.password),
      confirmPassword: validateConfirmPassword(formData.confirmPassword, formData.password)
    };

    setErrors(newErrors);

    // Return true if no errors
    return Object.values(newErrors).every(error => error === '');
  };

  // Handle form submission
  const handleSubmit = async () => {
    if (!validateForm()) {
      Alert.alert('Validation Error', 'Please fix the errors and try again');
      return;
    }

    setIsSubmitting(true);

    try {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 2000));

      Alert.alert('Success', 'Form submitted successfully!');

      // Reset form
      setFormData({
        name: '',
        email: '',
        password: '',
        confirmPassword: ''
      });
      setErrors({});

    } catch (error) {
      Alert.alert('Error', 'Something went wrong. Please try again.');
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Form Validation Example</Text>

      {/* Name Input */}
      <View style={styles.inputContainer}>
        <Text style={styles.label}>Full Name *</Text>
        <TextInput
          style={[
            styles.input,
            errors.name ? styles.inputError : null
          ]}
          placeholder="Enter your full name"
          value={formData.name}
          onChangeText={(value) => handleInputChange('name', value)}
          onBlur={() => validateField('name', formData.name)}
          autoCapitalize="words"
        />
        {errors.name ? (
          <Text style={styles.errorText}>{errors.name}</Text>
        ) : null}
      </View>

      {/* Email Input */}
      <View style={styles.inputContainer}>
        <Text style={styles.label}>Email Address *</Text>
        <TextInput
          style={[
            styles.input,
            errors.email ? styles.inputError : null
          ]}
          placeholder="Enter your email"
          value={formData.email}
          onChangeText={(value) => handleInputChange('email', value)}
          onBlur={() => validateField('email', formData.email)}
          keyboardType="email-address"
          autoCapitalize="none"
          autoCorrect={false}
        />
        {errors.email ? (
          <Text style={styles.errorText}>{errors.email}</Text>
        ) : null}
      </View>

      {/* Password Input */}
      <View style={styles.inputContainer}>
        <Text style={styles.label}>Password *</Text>
        <TextInput
          style={[
            styles.input,
            errors.password ? styles.inputError : null
          ]}
          placeholder="Enter your password"
          value={formData.password}
          onChangeText={(value) => handleInputChange('password', value)}
          onBlur={() => validateField('password', formData.password)}
          secureTextEntry={true}
          autoCapitalize="none"
          autoCorrect={false}
        />
        {errors.password ? (
          <Text style={styles.errorText}>{errors.password}</Text>
        ) : null}
      </View>

      {/* Confirm Password Input */}
      <View style={styles.inputContainer}>
        <Text style={styles.label}>Confirm Password *</Text>
        <TextInput
          style={[
            styles.input,
            errors.confirmPassword ? styles.inputError : null
          ]}
          placeholder="Confirm your password"
          value={formData.confirmPassword}
          onChangeText={(value) => handleInputChange('confirmPassword', value)}
          onBlur={() => validateField('confirmPassword', formData.confirmPassword)}
          secureTextEntry={true}
          autoCapitalize="none"
          autoCorrect={false}
        />
        {errors.confirmPassword ? (
          <Text style={styles.errorText}>{errors.confirmPassword}</Text>
        ) : null}
      </View>

      {/* Submit Button */}
      <TouchableOpacity
        style={[
          styles.button,
          isSubmitting ? styles.buttonDisabled : null
        ]}
        onPress={handleSubmit}
        disabled={isSubmitting}
      >
        <Text style={styles.buttonText}>
          {isSubmitting ? 'Submitting...' : 'Submit'}
        </Text>
      </TouchableOpacity>
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
  inputContainer: {
    marginBottom: 20,
  },
  label: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 8,
    color: '#333',
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    paddingHorizontal: 15,
    paddingVertical: 12,
    fontSize: 16,
    backgroundColor: 'white',
  },
  inputError: {
    borderColor: '#FF3B30',
    borderWidth: 2,
  },
  errorText: {
    color: '#FF3B30',
    fontSize: 14,
    marginTop: 5,
    marginLeft: 5,
  },
  button: {
    backgroundColor: '#007AFF',
    paddingVertical: 15,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 20,
  },
  buttonDisabled: {
    backgroundColor: '#CCCCCC',
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
});

export default FormValidationExample;
```

---

## 🎨 Advanced Input Components

### 1. Custom Input Component

```javascript
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet
} from 'react-native';

const CustomInput = ({
  label,
  value,
  onChangeText,
  placeholder,
  error,
  secureTextEntry = false,
  keyboardType = 'default',
  multiline = false,
  numberOfLines = 1,
  maxLength,
  required = false,
  leftIcon,
  rightIcon,
  onRightIconPress,
  ...props
}) => {
  const [isFocused, setIsFocused] = useState(false);

  return (
    <View style={styles.container}>
      {label && (
        <Text style={styles.label}>
          {label}
          {required && <Text style={styles.required}> *</Text>}
        </Text>
      )}

      <View style={[
        styles.inputContainer,
        isFocused && styles.inputContainerFocused,
        error && styles.inputContainerError
      ]}>
        {leftIcon && (
          <View style={styles.leftIcon}>
            {leftIcon}
          </View>
        )}

        <TextInput
          style={[
            styles.input,
            multiline && styles.multilineInput,
            leftIcon && styles.inputWithLeftIcon,
            rightIcon && styles.inputWithRightIcon
          ]}
          value={value}
          onChangeText={onChangeText}
          placeholder={placeholder}
          secureTextEntry={secureTextEntry}
          keyboardType={keyboardType}
          multiline={multiline}
          numberOfLines={numberOfLines}
          maxLength={maxLength}
          onFocus={() => setIsFocused(true)}
          onBlur={() => setIsFocused(false)}
          placeholderTextColor="#999"
          {...props}
        />

        {rightIcon && (
          <TouchableOpacity
            style={styles.rightIcon}
            onPress={onRightIconPress}
          >
            {rightIcon}
          </TouchableOpacity>
        )}
      </View>

      {error && (
        <Text style={styles.errorText}>{error}</Text>
      )}

      {maxLength && (
        <Text style={styles.characterCount}>
          {value.length}/{maxLength}
        </Text>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    marginBottom: 20,
  },
  label: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 8,
    color: '#333',
  },
  required: {
    color: '#FF3B30',
  },
  inputContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    backgroundColor: 'white',
  },
  inputContainerFocused: {
    borderColor: '#007AFF',
    borderWidth: 2,
  },
  inputContainerError: {
    borderColor: '#FF3B30',
    borderWidth: 2,
  },
  input: {
    flex: 1,
    paddingHorizontal: 15,
    paddingVertical: 12,
    fontSize: 16,
    color: '#333',
  },
  multilineInput: {
    textAlignVertical: 'top',
    minHeight: 100,
  },
  inputWithLeftIcon: {
    paddingLeft: 5,
  },
  inputWithRightIcon: {
    paddingRight: 5,
  },
  leftIcon: {
    paddingLeft: 15,
    paddingRight: 10,
  },
  rightIcon: {
    paddingRight: 15,
    paddingLeft: 10,
  },
  errorText: {
    color: '#FF3B30',
    fontSize: 14,
    marginTop: 5,
    marginLeft: 5,
  },
  characterCount: {
    fontSize: 12,
    color: '#999',
    textAlign: 'right',
    marginTop: 5,
  },
});

export default CustomInput;
```

### 2. Using Custom Input Component

```javascript
import React, { useState } from 'react';
import { View, Text, StyleSheet, TouchableOpacity, Alert } from 'react-native';
import CustomInput from './CustomInput';

const CustomInputExample = () => {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    bio: '',
    phone: ''
  });

  const [showPassword, setShowPassword] = useState(false);
  const [errors, setErrors] = useState({});

  const handleInputChange = (field, value) => {
    setFormData(prev => ({ ...prev, [field]: value }));
    // Clear error when user starts typing
    if (errors[field]) {
      setErrors(prev => ({ ...prev, [field]: '' }));
    }
  };

  const validateForm = () => {
    const newErrors = {};

    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }

    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 6) {
      newErrors.password = 'Password must be at least 6 characters';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = () => {
    if (validateForm()) {
      Alert.alert('Success', 'Form is valid!');
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Custom Input Components</Text>

      <CustomInput
        label="Email Address"
        value={formData.email}
        onChangeText={(value) => handleInputChange('email', value)}
        placeholder="Enter your email"
        keyboardType="email-address"
        error={errors.email}
        required={true}
        leftIcon={<Text>📧</Text>}
      />

      <CustomInput
        label="Password"
        value={formData.password}
        onChangeText={(value) => handleInputChange('password', value)}
        placeholder="Enter your password"
        secureTextEntry={!showPassword}
        error={errors.password}
        required={true}
        leftIcon={<Text>🔒</Text>}
        rightIcon={<Text>{showPassword ? '🙈' : '👁️'}</Text>}
        onRightIconPress={() => setShowPassword(!showPassword)}
      />

      <CustomInput
        label="Bio"
        value={formData.bio}
        onChangeText={(value) => handleInputChange('bio', value)}
        placeholder="Tell us about yourself..."
        multiline={true}
        numberOfLines={4}
        maxLength={200}
      />

      <CustomInput
        label="Phone Number"
        value={formData.phone}
        onChangeText={(value) => handleInputChange('phone', value)}
        placeholder="Enter your phone number"
        keyboardType="phone-pad"
        leftIcon={<Text>📱</Text>}
      />

      <TouchableOpacity style={styles.button} onPress={handleSubmit}>
        <Text style={styles.buttonText}>Submit</Text>
      </TouchableOpacity>
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
  button: {
    backgroundColor: '#007AFF',
    paddingVertical: 15,
    borderRadius: 8,
    alignItems: 'center',
    marginTop: 20,
  },
  buttonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
});

export default CustomInputExample;
```

---

## 📋 Complex Form Example

### 1. Registration Form

```javascript
import React, { useState } from 'react';
import {
  View,
  Text,
  ScrollView,
  TouchableOpacity,
  StyleSheet,
  Alert,
  Switch
} from 'react-native';
import CustomInput from './CustomInput';

const RegistrationForm = () => {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    password: '',
    confirmPassword: '',
    phone: '',
    dateOfBirth: '',
    gender: '',
    address: '',
    city: '',
    zipCode: '',
    agreeToTerms: false,
    subscribeNewsletter: false
  });

  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleInputChange = (field, value) => {
    setFormData(prev => ({ ...prev, [field]: value }));
    if (errors[field]) {
      setErrors(prev => ({ ...prev, [field]: '' }));
    }
  };

  const validateForm = () => {
    const newErrors = {};

    // Required field validations
    if (!formData.firstName.trim()) {
      newErrors.firstName = 'First name is required';
    }

    if (!formData.lastName.trim()) {
      newErrors.lastName = 'Last name is required';
    }

    if (!formData.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }

    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }

    if (formData.password !== formData.confirmPassword) {
      newErrors.confirmPassword = 'Passwords do not match';
    }

    if (!formData.phone.trim()) {
      newErrors.phone = 'Phone number is required';
    }

    if (!formData.agreeToTerms) {
      newErrors.agreeToTerms = 'You must agree to the terms and conditions';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async () => {
    if (!validateForm()) {
      Alert.alert('Validation Error', 'Please fix the errors and try again');
      return;
    }

    setIsSubmitting(true);

    try {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 2000));

      Alert.alert(
        'Success',
        'Registration completed successfully!',
        [
          {
            text: 'OK',
            onPress: () => {
              // Reset form or navigate to next screen
              console.log('Registration successful');
            }
          }
        ]
      );
    } catch (error) {
      Alert.alert('Error', 'Registration failed. Please try again.');
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <ScrollView style={styles.container} showsVerticalScrollIndicator={false}>
      <Text style={styles.title}>Create Account</Text>

      {/* Personal Information */}
      <Text style={styles.sectionTitle}>Personal Information</Text>

      <View style={styles.row}>
        <View style={styles.halfWidth}>
          <CustomInput
            label="First Name"
            value={formData.firstName}
            onChangeText={(value) => handleInputChange('firstName', value)}
            placeholder="First name"
            error={errors.firstName}
            required={true}
            autoCapitalize="words"
          />
        </View>

        <View style={styles.halfWidth}>
          <CustomInput
            label="Last Name"
            value={formData.lastName}
            onChangeText={(value) => handleInputChange('lastName', value)}
            placeholder="Last name"
            error={errors.lastName}
            required={true}
            autoCapitalize="words"
          />
        </View>
      </View>

      <CustomInput
        label="Email Address"
        value={formData.email}
        onChangeText={(value) => handleInputChange('email', value)}
        placeholder="Enter your email"
        keyboardType="email-address"
        error={errors.email}
        required={true}
        autoCapitalize="none"
      />

      <CustomInput
        label="Phone Number"
        value={formData.phone}
        onChangeText={(value) => handleInputChange('phone', value)}
        placeholder="Enter your phone number"
        keyboardType="phone-pad"
        error={errors.phone}
        required={true}
      />

      {/* Security */}
      <Text style={styles.sectionTitle}>Security</Text>

      <CustomInput
        label="Password"
        value={formData.password}
        onChangeText={(value) => handleInputChange('password', value)}
        placeholder="Create a password"
        secureTextEntry={true}
        error={errors.password}
        required={true}
      />

      <CustomInput
        label="Confirm Password"
        value={formData.confirmPassword}
        onChangeText={(value) => handleInputChange('confirmPassword', value)}
        placeholder="Confirm your password"
        secureTextEntry={true}
        error={errors.confirmPassword}
        required={true}
      />

      {/* Address Information */}
      <Text style={styles.sectionTitle}>Address (Optional)</Text>

      <CustomInput
        label="Street Address"
        value={formData.address}
        onChangeText={(value) => handleInputChange('address', value)}
        placeholder="Enter your address"
        multiline={true}
        numberOfLines={2}
      />

      <View style={styles.row}>
        <View style={styles.twoThirds}>
          <CustomInput
            label="City"
            value={formData.city}
            onChangeText={(value) => handleInputChange('city', value)}
            placeholder="City"
            autoCapitalize="words"
          />
        </View>

        <View style={styles.oneThird}>
          <CustomInput
            label="ZIP Code"
            value={formData.zipCode}
            onChangeText={(value) => handleInputChange('zipCode', value)}
            placeholder="ZIP"
            keyboardType="numeric"
            maxLength={6}
          />
        </View>
      </View>

      {/* Preferences */}
      <Text style={styles.sectionTitle}>Preferences</Text>

      <View style={styles.switchContainer}>
        <Text style={styles.switchLabel}>
          I agree to the Terms and Conditions *
        </Text>
        <Switch
          value={formData.agreeToTerms}
          onValueChange={(value) => handleInputChange('agreeToTerms', value)}
          trackColor={{ false: '#767577', true: '#007AFF' }}
          thumbColor={formData.agreeToTerms ? '#007AFF' : '#f4f3f4'}
        />
      </View>
      {errors.agreeToTerms && (
        <Text style={styles.errorText}>{errors.agreeToTerms}</Text>
      )}

      <View style={styles.switchContainer}>
        <Text style={styles.switchLabel}>
          Subscribe to newsletter
        </Text>
        <Switch
          value={formData.subscribeNewsletter}
          onValueChange={(value) => handleInputChange('subscribeNewsletter', value)}
          trackColor={{ false: '#767577', true: '#007AFF' }}
          thumbColor={formData.subscribeNewsletter ? '#007AFF' : '#f4f3f4'}
        />
      </View>

      {/* Submit Button */}
      <TouchableOpacity
        style={[
          styles.submitButton,
          isSubmitting && styles.submitButtonDisabled
        ]}
        onPress={handleSubmit}
        disabled={isSubmitting}
      >
        <Text style={styles.submitButtonText}>
          {isSubmitting ? 'Creating Account...' : 'Create Account'}
        </Text>
      </TouchableOpacity>
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 28,
    fontWeight: 'bold',
    textAlign: 'center',
    marginVertical: 20,
    color: '#333',
  },
  sectionTitle: {
    fontSize: 20,
    fontWeight: 'bold',
    marginTop: 20,
    marginBottom: 15,
    color: '#007AFF',
  },
  row: {
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
  halfWidth: {
    width: '48%',
  },
  twoThirds: {
    width: '65%',
  },
  oneThird: {
    width: '30%',
  },
  switchContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    backgroundColor: 'white',
    padding: 15,
    borderRadius: 8,
    marginBottom: 10,
  },
  switchLabel: {
    fontSize: 16,
    color: '#333',
    flex: 1,
    marginRight: 10,
  },
  errorText: {
    color: '#FF3B30',
    fontSize: 14,
    marginTop: 5,
    marginLeft: 5,
  },
  submitButton: {
    backgroundColor: '#28A745',
    paddingVertical: 18,
    borderRadius: 10,
    alignItems: 'center',
    marginTop: 30,
    marginBottom: 30,
  },
  submitButtonDisabled: {
    backgroundColor: '#CCCCCC',
  },
  submitButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
});

export default RegistrationForm;
```

---

## 📝 **Assignment 6: Form Handling Practice**

### **Task 1: Login Form with Validation (60 minutes)**
Create a login form with:
- Email and password fields
- Remember me checkbox
- Forgot password link
- Form validation
- Loading states
- Error handling

### **Task 2: Contact Form with File Upload (90 minutes)**
Build a contact form that includes:
- Name, email, phone fields
- Message textarea
- File attachment (image/document)
- Form validation
- Progress indicator for file upload
- Success/error feedback

### **Task 3: Multi-Step Registration Form (120 minutes)**
Implement a multi-step registration with:
- Step 1: Personal Information
- Step 2: Account Security
- Step 3: Preferences
- Progress indicator
- Form validation per step
- Data persistence between steps
- Final confirmation

---

## 🧠 **Quiz 6: User Input & Forms**

### **Multiple Choice Questions (1-15)**

**Q1. TextInput component ka primary purpose kya hai?**
a) Display text
b) Handle user input
c) Style text
d) Navigate between screens

**Q2. secureTextEntry prop ka use kab karte hain?**
a) Email input ke liye
b) Password input ke liye
c) Phone input ke liye
d) Name input ke liye

**Q3. keyboardType prop ka purpose kya hai?**
a) TextInput ki styling ke liye
b) Keyboard type specify karne ke liye
c) Text validation ke liye
d) Auto-complete ke liye

**Q4. Form validation mein onBlur event kab trigger hota hai?**
a) Jab user type karta hai
b) Jab input field focus lose karta hai
c) Jab form submit hota hai
d) Jab component mount hota hai

**Q5. Multiline TextInput ke liye kaunsa prop use karte hain?**
a) multiline
b) multiLine
c) textArea
d) textarea

**Q6. TextInput mein character limit set karne ke liye kaunsa prop use karte hain?**
a) maxLength
b) maxLen
c) limit
d) max

**Q7. Custom input component banane ka main benefit kya hai?**
a) Performance improvement
b) Code reusability
c) Bundle size reduction
d) Better debugging

**Q8. Form submission mein loading state dikhane ka purpose kya hai?**
a) UI ko attractive banana
b) User ko feedback dena ki operation in progress hai
c) Network calls ko fast karna
d) Memory usage kam karna

**Q9. Error handling mein inline validation ka matlab kya hai?**
a) Form submit hone ke baad error dikhana
b) User type karte time real-time validation
c) Server se error lena
d) Network error handle karna

**Q10. TouchableOpacity vs Pressable mein kya difference hai?**
a) TouchableOpacity deprecated hai
b) Pressable more flexible hai
c) TouchableOpacity better performance deta hai
d) Pressable sirf iOS pe kaam karta hai

### **True/False Questions (16-20)**

**Q16. TextInput automatically scroll hota hai agar content zyada ho.** (True/False)

**Q17. Form validation sirf client-side kar sakte hain.** (True/False)

**Q18. React Native mein regular HTML forms use kar sakte hain.** (True/False)

**Q19. TextInput component controlled component hai.** (True/False)

**Q20. Auto-capitalization TextInput mein default enabled hoti hai.** (True/False)

### **Code Output Questions (21-25)**

**Q21. Is code ka output kya hoga?**
```javascript
<TextInput
  value="Hello"
  onChangeText={(text) => console.log(text)}
  maxLength={3}
/>
```

**Q22. Is code mein problem kya hai?**
```javascript
const [email, setEmail] = useState('');
<TextInput
  value={email}
  onChangeText={setEmail}
  keyboardType="email-address"
  // Missing validation
/>
```

**Q23. Is validation function ka output kya hoga?**
```javascript
const validateEmail = (email) => {
  if (!email.includes('@')) {
    return 'Invalid email';
  }
  return '';
};

validateEmail('test.com');
```

**Q24. Is code ka behavior kya hoga?**
```javascript
<TextInput
  multiline={true}
  numberOfLines={4}
  textAlignVertical="top"
/>
```

**Q25. Is form submission mein kya galat hai?**
```javascript
const handleSubmit = async () => {
  // No validation
  await submitForm(formData);
  setFormData({}); // Reset without checking success
};
```

---

## 📚 **Answers & Explanations**

### **Multiple Choice Answers**
**Q1.** b) Handle user input  
**Q2.** b) Password input ke liye  
**Q3.** b) Keyboard type specify karne ke liye  
**Q4.** b) Jab input field focus lose karta hai  
**Q5.** a) multiline  
**Q6.** a) maxLength  
**Q7.** b) Code reusability  
**Q8.** b) User ko feedback dena ki operation in progress hai  
**Q9.** b) User type karte time real-time validation  
**Q10.** b) Pressable more flexible hai  

### **True/False Answers**
**Q16.** False - Multiline TextInput ke liye manually scroll enable karna padta hai  
**Q17.** False - Server-side validation bhi kar sakte hain  
**Q18.** False - React Native mein native components use hote hain  
**Q19.** True - Value aur onChangeText props se controlled hota hai  
**Q20.** True - autoCapitalize default "sentences" hota hai  

### **Code Output Answers**
**Q21.** "Hel" (maxLength 3 ke karan sirf pehle 3 characters)  
**Q22.** Missing validation - form submit hone se pehle validation nahi hai  
**Q23.** "Invalid email" (kyonki @ symbol nahi hai)  
**Q24.** Multi-line text input with top alignment  
**Q25.** No error handling aur validation missing hai  

---

## 🎯 **Assignment Solutions**

### **Task 1: Login Form Solution**

```javascript
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  StyleSheet,
  Alert,
  ActivityIndicator
} from 'react-native';

const LoginForm = () => {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    rememberMe: false
  });
  const [errors, setErrors] = useState({});
  const [isLoading, setIsLoading] = useState(false);

  const handleInputChange = (field, value) => {
    setFormData(prev => ({ ...prev, [field]: value }));
    if (errors[field]) {
      setErrors(prev => ({ ...prev, [field]: '' }));
    }
  };

  const validateForm = () => {
    const newErrors = {};

    if (!formData.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Please enter a valid email';
    }

    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 6) {
      newErrors.password = 'Password must be at least 6 characters';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleLogin = async () => {
    if (!validateForm()) return;

    setIsLoading(true);

    try {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 2000));

      Alert.alert('Success', 'Login successful!');
    } catch (error) {
      Alert.alert('Error', 'Login failed. Please try again.');
    } finally {
      setIsLoading(false);
    }
  };

  const handleForgotPassword = () => {
    Alert.alert('Forgot Password', 'Reset link sent to your email');
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Login</Text>

      <View style={styles.inputContainer}>
        <Text style={styles.label}>Email</Text>
        <TextInput
          style={[styles.input, errors.email && styles.inputError]}
          placeholder="Enter your email"
          value={formData.email}
          onChangeText={(value) => handleInputChange('email', value)}
          keyboardType="email-address"
          autoCapitalize="none"
        />
        {errors.email && <Text style={styles.errorText}>{errors.email}</Text>}
      </View>

      <View style={styles.inputContainer}>
        <Text style={styles.label}>Password</Text>
        <TextInput
          style={[styles.input, errors.password && styles.inputError]}
          placeholder="Enter your password"
          value={formData.password}
          onChangeText={(value) => handleInputChange('password', value)}
          secureTextEntry
          autoCapitalize="none"
        />
        {errors.password && <Text style={styles.errorText}>{errors.password}</Text>}
      </View>

      <View style={styles.optionsContainer}>
        <TouchableOpacity
          style={styles.checkboxContainer}
          onPress={() => handleInputChange('rememberMe', !formData.rememberMe)}
        >
          <View style={[styles.checkbox, formData.rememberMe && styles.checkboxChecked]}>
            {formData.rememberMe && <Text style={styles.checkmark}>✓</Text>}
          </View>
          <Text style={styles.checkboxLabel}>Remember me</Text>
        </TouchableOpacity>

        <TouchableOpacity onPress={handleForgotPassword}>
          <Text style={styles.forgotPassword}>Forgot Password?</Text>
        </TouchableOpacity>
      </View>

      <TouchableOpacity
        style={[styles.loginButton, isLoading && styles.loginButtonDisabled]}
        onPress={handleLogin}
        disabled={isLoading}
      >
        {isLoading ? (
          <ActivityIndicator color="white" />
        ) : (
          <Text style={styles.loginButtonText}>Login</Text>
        )}
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    justifyContent: 'center',
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    textAlign: 'center',
    marginBottom: 40,
    color: '#333',
  },
  inputContainer: {
    marginBottom: 20,
  },
  label: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 8,
    color: '#333',
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 15,
    fontSize: 16,
    backgroundColor: 'white',
  },
  inputError: {
    borderColor: '#FF3B30',
  },
  errorText: {
    color: '#FF3B30',
    fontSize: 14,
    marginTop: 5,
  },
  optionsContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: 30,
  },
  checkboxContainer: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  checkbox: {
    width: 20,
    height: 20,
    borderWidth: 2,
    borderColor: '#007AFF',
    borderRadius: 4,
    marginRight: 8,
    justifyContent: 'center',
    alignItems: 'center',
  },
  checkboxChecked: {
    backgroundColor: '#007AFF',
  },
  checkmark: {
    color: 'white',
    fontSize: 14,
    fontWeight: 'bold',
  },
  checkboxLabel: {
    fontSize: 16,
    color: '#333',
  },
  forgotPassword: {
    color: '#007AFF',
    fontSize: 16,
    fontWeight: 'bold',
  },
  loginButton: {
    backgroundColor: '#007AFF',
    paddingVertical: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  loginButtonDisabled: {
    backgroundColor: '#CCCCCC',
  },
  loginButtonText: {
    color: 'white',
    fontSize: 18,
    fontWeight: 'bold',
  },
});

export default LoginForm;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **TextInput Component**: Basic to advanced input handling
- ✅ **Form Validation**: Client-side validation with error handling
- ✅ **Custom Input Components**: Reusable input components with validation
- ✅ **Complex Forms**: Multi-step forms with proper state management
- ✅ **User Experience**: Loading states, error feedback, and validation