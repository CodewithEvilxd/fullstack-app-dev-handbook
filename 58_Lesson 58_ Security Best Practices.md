# Lesson 58: Security Best Practices

## 🎯 **Learning Objectives**
- Implement comprehensive security measures for React Native applications
- Protect sensitive data and user information
- Secure API communications and authentication
- Handle security vulnerabilities and threats
- Implement secure coding practices and code review processes

## 📚 **Table of Contents**
1. [Security Fundamentals](#security-fundamentals)
2. [Data Protection](#data-protection)
3. [Authentication & Authorization](#authentication--authorization)
4. [API Security](#api-security)
5. [Code Security](#code-security)
6. [Network Security](#network-security)
7. [Platform-Specific Security](#platform-specific-security)
8. [Security Testing](#security-testing)
9. [Incident Response](#incident-response)
10. [Practical Examples](#practical-examples)

---

## 🔒 **Security Fundamentals**

### **Security Principles**
- **CIA Triad**: Confidentiality, Integrity, Availability
- **Defense in Depth**: Multiple layers of security
- **Least Privilege**: Minimum required permissions
- **Fail-Safe Defaults**: Secure by default
- **Zero Trust**: Never trust, always verify

### **Common Security Threats**
- **Data Breaches**: Unauthorized access to sensitive data
- **Man-in-the-Middle Attacks**: Intercepting communications
- **Injection Attacks**: SQL injection, XSS, code injection
- **Session Hijacking**: Stealing user sessions
- **Reverse Engineering**: Analyzing app binaries
- **Malware**: Malicious software distribution

### **Security Best Practices**
```javascript
// Security configuration constants
const SECURITY_CONFIG = {
  // Encryption settings
  ENCRYPTION_ALGORITHM: 'AES-256-GCM',
  KEY_DERIVATION_ITERATIONS: 10000,

  // Session settings
  SESSION_TIMEOUT: 30 * 60 * 1000, // 30 minutes
  MAX_LOGIN_ATTEMPTS: 5,
  LOCKOUT_DURATION: 15 * 60 * 1000, // 15 minutes

  // API settings
  API_TIMEOUT: 30000, // 30 seconds
  MAX_REQUEST_SIZE: 10 * 1024 * 1024, // 10MB

  // Certificate pinning
  CERTIFICATE_PINNING_ENABLED: true,

  // Debug settings
  DEBUG_LOGGING_ENABLED: __DEV__,
  SENSITIVE_DATA_LOGGING: false
};

export default SECURITY_CONFIG;
```

---

## 🛡️ **Data Protection**

### **Data Encryption**
```javascript
// services/encryptionService.js
import CryptoJS from 'crypto-js';
import { Platform } from 'react-native';

class EncryptionService {
  constructor() {
    this.algorithm = 'AES';
    this.keySize = 256;
    this.iterations = 10000;
  }

  // Generate encryption key from password
  generateKey(password, salt = null) {
    if (!salt) {
      salt = CryptoJS.lib.WordArray.random(128/8);
    }

    const key = CryptoJS.PBKDF2(password, salt, {
      keySize: this.keySize / 32,
      iterations: this.iterations
    });

    return {
      key: key,
      salt: salt
    };
  }

  // Encrypt data
  encrypt(data, password) {
    try {
      const { key, salt } = this.generateKey(password);

      const encrypted = CryptoJS.AES.encrypt(data, key, {
        iv: salt,
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7
      });

      return {
        encryptedData: encrypted.toString(),
        salt: salt.toString(),
        success: true
      };
    } catch (error) {
      console.error('Encryption failed:', error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Decrypt data
  decrypt(encryptedData, password, salt) {
    try {
      const saltWordArray = CryptoJS.enc.Hex.parse(salt);
      const { key } = this.generateKey(password, saltWordArray);

      const decrypted = CryptoJS.AES.decrypt(encryptedData, key, {
        iv: saltWordArray,
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7
      });

      const decryptedText = decrypted.toString(CryptoJS.enc.Utf8);

      if (!decryptedText) {
        throw new Error('Decryption failed - invalid password or corrupted data');
      }

      return {
        decryptedData: decryptedText,
        success: true
      };
    } catch (error) {
      console.error('Decryption failed:', error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Encrypt sensitive data for storage
  encryptForStorage(data, key) {
    try {
      const encrypted = CryptoJS.AES.encrypt(JSON.stringify(data), key);
      return encrypted.toString();
    } catch (error) {
      console.error('Storage encryption failed:', error);
      throw error;
    }
  }

  // Decrypt data from storage
  decryptFromStorage(encryptedData, key) {
    try {
      const decrypted = CryptoJS.AES.decrypt(encryptedData, key);
      const decryptedText = decrypted.toString(CryptoJS.enc.Utf8);

      if (!decryptedText) {
        throw new Error('Storage decryption failed');
      }

      return JSON.parse(decryptedText);
    } catch (error) {
      console.error('Storage decryption failed:', error);
      throw error;
    }
  }

  // Generate secure random key
  generateSecureKey(length = 32) {
    return CryptoJS.lib.WordArray.random(length).toString();
  }

  // Hash sensitive data (one-way)
  hashData(data, salt = null) {
    if (!salt) {
      salt = CryptoJS.lib.WordArray.random(128/8).toString();
    }

    const hash = CryptoJS.PBKDF2(data, salt, {
      keySize: 256/32,
      iterations: 10000
    });

    return {
      hash: hash.toString(),
      salt: salt
    };
  }

  // Verify hashed data
  verifyHash(data, hash, salt) {
    const computedHash = CryptoJS.PBKDF2(data, salt, {
      keySize: 256/32,
      iterations: 10000
    });

    return computedHash.toString() === hash;
  }
}

export const encryptionService = new EncryptionService();
```

### **Secure Storage**
```javascript
// services/secureStorage.js
import AsyncStorage from '@react-native-async-storage/async-storage';
import { encryptionService } from './encryptionService';

class SecureStorage {
  constructor() {
    this.storageKey = 'secure_app_data';
    this.encryptionKey = 'your-secure-encryption-key'; // In production, use device-specific key
  }

  // Store sensitive data securely
  async storeSecureData(key, data) {
    try {
      // Encrypt the data
      const encryptedData = encryptionService.encryptForStorage(
        { data, timestamp: Date.now() },
        this.encryptionKey
      );

      // Store in AsyncStorage
      await AsyncStorage.setItem(`${this.storageKey}_${key}`, encryptedData);

      console.log(`Secure data stored for key: ${key}`);
      return { success: true };
    } catch (error) {
      console.error('Failed to store secure data:', error);
      return { success: false, error: error.message };
    }
  }

  // Retrieve sensitive data securely
  async getSecureData(key) {
    try {
      // Retrieve from AsyncStorage
      const encryptedData = await AsyncStorage.getItem(`${this.storageKey}_${key}`);

      if (!encryptedData) {
        return { success: false, error: 'Data not found' };
      }

      // Decrypt the data
      const decryptedData = encryptionService.decryptFromStorage(
        encryptedData,
        this.encryptionKey
      );

      // Check if data is expired (optional)
      if (this.isDataExpired(decryptedData.timestamp)) {
        await this.removeSecureData(key);
        return { success: false, error: 'Data expired' };
      }

      return {
        success: true,
        data: decryptedData.data
      };
    } catch (error) {
      console.error('Failed to retrieve secure data:', error);
      return { success: false, error: error.message };
    }
  }

  // Remove sensitive data
  async removeSecureData(key) {
    try {
      await AsyncStorage.removeItem(`${this.storageKey}_${key}`);
      console.log(`Secure data removed for key: ${key}`);
      return { success: true };
    } catch (error) {
      console.error('Failed to remove secure data:', error);
      return { success: false, error: error.message };
    }
  }

  // Clear all secure data
  async clearAllSecureData() {
    try {
      const keys = await AsyncStorage.getAllKeys();
      const secureKeys = keys.filter(key => key.startsWith(this.storageKey));

      await AsyncStorage.multiRemove(secureKeys);
      console.log(`Cleared ${secureKeys.length} secure data items`);
      return { success: true };
    } catch (error) {
      console.error('Failed to clear secure data:', error);
      return { success: false, error: error.message };
    }
  }

  // Check if data is expired
  isDataExpired(timestamp, maxAge = 24 * 60 * 60 * 1000) { // 24 hours
    return Date.now() - timestamp > maxAge;
  }

  // Store authentication tokens
  async storeAuthTokens(accessToken, refreshToken) {
    const tokens = {
      accessToken,
      refreshToken,
      storedAt: Date.now()
    };

    return await this.storeSecureData('auth_tokens', tokens);
  }

  // Get authentication tokens
  async getAuthTokens() {
    const result = await this.getSecureData('auth_tokens');

    if (!result.success) {
      return result;
    }

    const tokens = result.data;

    // Check if tokens are expired
    if (this.isTokenExpired(tokens.accessToken)) {
      await this.removeSecureData('auth_tokens');
      return { success: false, error: 'Tokens expired' };
    }

    return {
      success: true,
      data: {
        accessToken: tokens.accessToken,
        refreshToken: tokens.refreshToken
      }
    };
  }

  // Check if token is expired (simplified)
  isTokenExpired(token) {
    try {
      // This is a simplified check - in production, you'd decode the JWT
      // and check the expiration time
      return false; // Implement proper JWT expiration check
    } catch (error) {
      return true;
    }
  }

  // Store user credentials (hashed)
  async storeUserCredentials(username, password) {
    const hashedPassword = encryptionService.hashData(password);

    const credentials = {
      username,
      passwordHash: hashedPassword.hash,
      salt: hashedPassword.salt,
      storedAt: Date.now()
    };

    return await this.storeSecureData('user_credentials', credentials);
  }

  // Verify user credentials
  async verifyUserCredentials(username, password) {
    const result = await this.getSecureData('user_credentials');

    if (!result.success) {
      return false;
    }

    const credentials = result.data;

    if (credentials.username !== username) {
      return false;
    }

    return encryptionService.verifyHash(
      password,
      credentials.passwordHash,
      credentials.salt
    );
  }
}

export const secureStorage = new SecureStorage();
```

---

## 🔐 **Authentication & Authorization**

### **JWT Token Management**
```javascript
// services/authService.js
import jwtDecode from 'jwt-decode';
import { secureStorage } from './secureStorage';

class AuthService {
  constructor() {
    this.baseURL = 'https://api.yourapp.com';
    this.refreshPromise = null;
  }

  // Login user
  async login(credentials) {
    try {
      const response = await fetch(`${this.baseURL}/auth/login`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(credentials),
      });

      if (!response.ok) {
        throw new Error(`Login failed: ${response.status}`);
      }

      const data = await response.json();

      // Store tokens securely
      await secureStorage.storeAuthTokens(data.accessToken, data.refreshToken);

      return {
        success: true,
        user: data.user,
        tokens: {
          accessToken: data.accessToken,
          refreshToken: data.refreshToken
        }
      };
    } catch (error) {
      console.error('Login error:', error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Logout user
  async logout() {
    try {
      const tokens = await secureStorage.getAuthTokens();

      if (tokens.success) {
        // Call logout endpoint
        await fetch(`${this.baseURL}/auth/logout`, {
          method: 'POST',
          headers: {
            'Authorization': `Bearer ${tokens.data.accessToken}`,
          },
        });
      }
    } catch (error) {
      console.error('Logout error:', error);
    } finally {
      // Clear local tokens
      await secureStorage.removeSecureData('auth_tokens');
    }
  }

  // Get current user
  async getCurrentUser() {
    try {
      const tokens = await secureStorage.getAuthTokens();

      if (!tokens.success) {
        return { success: false, error: 'No authentication tokens' };
      }

      const response = await fetch(`${this.baseURL}/auth/me`, {
        headers: {
          'Authorization': `Bearer ${tokens.data.accessToken}`,
        },
      });

      if (response.status === 401) {
        // Token expired, try refresh
        const refreshResult = await this.refreshToken();
        if (refreshResult.success) {
          return await this.getCurrentUser();
        }
        return { success: false, error: 'Authentication expired' };
      }

      if (!response.ok) {
        throw new Error(`Failed to get user: ${response.status}`);
      }

      const user = await response.json();

      return {
        success: true,
        user
      };
    } catch (error) {
      console.error('Get current user error:', error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Refresh access token
  async refreshToken() {
    if (this.refreshPromise) {
      return this.refreshPromise;
    }

    this.refreshPromise = this._performTokenRefresh();

    try {
      const result = await this.refreshPromise;
      return result;
    } finally {
      this.refreshPromise = null;
    }
  }

  async _performTokenRefresh() {
    try {
      const tokens = await secureStorage.getAuthTokens();

      if (!tokens.success) {
        throw new Error('No refresh token available');
      }

      const response = await fetch(`${this.baseURL}/auth/refresh`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          refreshToken: tokens.data.refreshToken
        }),
      });

      if (!response.ok) {
        throw new Error(`Token refresh failed: ${response.status}`);
      }

      const data = await response.json();

      // Store new tokens
      await secureStorage.storeAuthTokens(data.accessToken, data.refreshToken);

      return {
        success: true,
        tokens: {
          accessToken: data.accessToken,
          refreshToken: data.refreshToken
        }
      };
    } catch (error) {
      console.error('Token refresh error:', error);

      // Clear invalid tokens
      await secureStorage.removeSecureData('auth_tokens');

      return {
        success: false,
        error: error.message
      };
    }
  }

  // Check if user is authenticated
  async isAuthenticated() {
    const tokens = await secureStorage.getAuthTokens();
    return tokens.success;
  }

  // Get valid access token (refreshes if needed)
  async getValidAccessToken() {
    const tokens = await secureStorage.getAuthTokens();

    if (!tokens.success) {
      return null;
    }

    // Check if token is expired
    if (this.isTokenExpired(tokens.data.accessToken)) {
      const refreshResult = await this.refreshToken();

      if (!refreshResult.success) {
        return null;
      }

      return refreshResult.tokens.accessToken;
    }

    return tokens.data.accessToken;
  }

  // Check if token is expired
  isTokenExpired(token) {
    try {
      const decoded = jwtDecode(token);
      const currentTime = Date.now() / 1000;

      return decoded.exp < currentTime;
    } catch (error) {
      console.error('Token decode error:', error);
      return true;
    }
  }

  // Make authenticated API request
  async authenticatedRequest(url, options = {}) {
    const token = await this.getValidAccessToken();

    if (!token) {
      throw new Error('No valid authentication token');
    }

    const defaultOptions = {
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json',
      },
    };

    const mergedOptions = {
      ...defaultOptions,
      ...options,
      headers: {
        ...defaultOptions.headers,
        ...options.headers,
      },
    };

    return fetch(url, mergedOptions);
  }
}

export const authService = new AuthService();
```

### **Biometric Authentication**
```javascript
// services/biometricService.js
import ReactNativeBiometrics from 'react-native-biometrics';

class BiometricService {
  constructor() {
    this.biometrics = new ReactNativeBiometrics();
  }

  // Check if biometric authentication is available
  async isBiometricAvailable() {
    try {
      const { available, biometryType } = await this.biometrics.isSensorAvailable();

      return {
        available,
        biometryType, // TouchID, FaceID, Biometrics, etc.
        supported: available && biometryType !== null
      };
    } catch (error) {
      console.error('Biometric availability check failed:', error);
      return {
        available: false,
        biometryType: null,
        supported: false
      };
    }
  }

  // Create biometric keys
  async createBiometricKeys() {
    try {
      const { publicKey } = await this.biometrics.createKeys();

      return {
        success: true,
        publicKey
      };
    } catch (error) {
      console.error('Biometric key creation failed:', error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Perform biometric authentication
  async authenticateBiometric(reason = 'Authenticate to continue') {
    try {
      const { success, signature } = await this.biometrics.createSignature({
        promptMessage: reason,
        payload: Date.now().toString()
      });

      return {
        success,
        signature,
        authenticated: success
      };
    } catch (error) {
      console.error('Biometric authentication failed:', error);
      return {
        success: false,
        authenticated: false,
        error: error.message
      };
    }
  }

  // Simple biometric authentication (without signature)
  async simpleBiometricAuth(reason = 'Authenticate to continue') {
    try {
      const { success } = await this.biometrics.simplePrompt({
        promptMessage: reason,
      });

      return {
        success,
        authenticated: success
      };
    } catch (error) {
      console.error('Simple biometric auth failed:', error);
      return {
        success: false,
        authenticated: false,
        error: error.message
      };
    }
  }

  // Delete biometric keys
  async deleteBiometricKeys() {
    try {
      const { keysDeleted } = await this.biometrics.deleteKeys();

      return {
        success: true,
        keysDeleted
      };
    } catch (error) {
      console.error('Biometric key deletion failed:', error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Store biometric-enabled credentials
  async storeBiometricCredentials(username, password) {
    try {
      // First, create biometric keys if they don't exist
      const keyResult = await this.createBiometricKeys();

      if (!keyResult.success) {
        throw new Error('Failed to create biometric keys');
      }

      // Store credentials with biometric flag
      const credentials = {
        username,
        password: await this.encryptWithBiometric(password),
        biometricEnabled: true,
        biometricPublicKey: keyResult.publicKey,
        createdAt: Date.now()
      };

      await secureStorage.storeSecureData('biometric_credentials', credentials);

      return {
        success: true,
        message: 'Biometric credentials stored successfully'
      };
    } catch (error) {
      console.error('Failed to store biometric credentials:', error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Authenticate with biometrics and get credentials
  async authenticateWithBiometrics() {
    try {
      // First, check if biometrics are available
      const availability = await this.isBiometricAvailable();

      if (!availability.supported) {
        throw new Error('Biometric authentication not available');
      }

      // Perform biometric authentication
      const authResult = await this.authenticateBiometric('Authenticate to access your credentials');

      if (!authResult.authenticated) {
        throw new Error('Biometric authentication failed');
      }

      // Retrieve stored credentials
      const credentialsResult = await secureStorage.getSecureData('biometric_credentials');

      if (!credentialsResult.success) {
        throw new Error('No biometric credentials found');
      }

      const credentials = credentialsResult.data;

      // Decrypt password using biometric signature
      const password = await this.decryptWithBiometric(credentials.password, authResult.signature);

      return {
        success: true,
        username: credentials.username,
        password
      };
    } catch (error) {
      console.error('Biometric authentication failed:', error);
      return {
        success: false,
        error: error.message
      };
    }
  }

  // Encrypt data with biometric key
  async encryptWithBiometric(data) {
    // This is a simplified implementation
    // In production, use proper biometric encryption
    return encryptionService.encryptForStorage(data, 'biometric-key');
  }

  // Decrypt data with biometric signature
  async decryptWithBiometric(encryptedData, signature) {
    // This is a simplified implementation
    // In production, use proper biometric decryption
    return encryptionService.decryptFromStorage(encryptedData, 'biometric-key');
  }
}

export const biometricService = new BiometricService();
```

---

## 🌐 **API Security**

### **Secure API Client**
```javascript
// services/apiClient.js
import { authService } from './authService';

class SecureAPIClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
    this.defaultTimeout = 30000; // 30 seconds
  }

  // Make authenticated request
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const token = await authService.getValidAccessToken();

    if (!token) {
      throw new Error('No authentication token available');
    }

    const defaultOptions = {
      method: 'GET',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json',
        'X-Requested-With': 'XMLHttpRequest',
        'X-App-Version': '1.0.0',
        'X-Platform': Platform.OS,
      },
      timeout: this.defaultTimeout,
    };

    const mergedOptions = {
      ...defaultOptions,
      ...options,
      headers: {
        ...defaultOptions.headers,
        ...options.headers,
      },
    };

    // Add request ID for tracking
    const requestId = this.generateRequestId();
    mergedOptions.headers['X-Request-ID'] = requestId;

    // Log request (without sensitive data)
    console.log(`API Request: ${mergedOptions.method} ${url} [${requestId}]`);

    try {
      const response = await this.makeRequest(url, mergedOptions);

      // Log response
      console.log(`API Response: ${response.status} [${requestId}]`);

      return response;
    } catch (error) {
      console.error(`API Error: ${error.message} [${requestId}]`);
      throw error;
    }
  }

  // Make the actual HTTP request with security features
  async makeRequest(url, options) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), options.timeout);

    try {
      const response = await fetch(url, {
        ...options,
        signal: controller.signal,
      });

      clearTimeout(timeoutId);

      // Check for security headers
      this.validateSecurityHeaders(response.headers);

      // Handle different response types
      if (response.status === 401) {
        // Token might be expired, try refresh
        const refreshResult = await authService.refreshToken();
        if (refreshResult.success) {
          // Retry the request with new token
          return this.makeRequest(url, options);
        }
      }

      return response;
    } catch (error) {
      clearTimeout(timeoutId);

      if (error.name === 'AbortError') {
        throw new Error('Request timeout');
      }

      throw error;
    }
  }

  // Validate security headers
  validateSecurityHeaders(headers) {
    // Check for important security headers
    const securityHeaders = [
      'X-Content-Type-Options',
      'X-Frame-Options',
      'X-XSS-Protection',
      'Strict-Transport-Security'
    ];

    for (const header of securityHeaders) {
      if (!headers.get(header)) {
        console.warn(`Missing security header: ${header}`);
      }
    }
  }

  // Generate unique request ID
  generateRequestId() {
    return `req_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }

  // GET request
  async get(endpoint, params = {}) {
    const queryString = new URLSearchParams(params).toString();
    const url = queryString ? `${endpoint}?${queryString}` : endpoint;

    return this.request(url);
  }

  // POST request
  async post(endpoint, data = {}) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  // PUT request
  async put(endpoint, data = {}) {
    return this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }

  // DELETE request
  async delete(endpoint) {
    return this.request(endpoint, {
      method: 'DELETE',
    });
  }

  // Upload file securely
  async uploadFile(endpoint, fileUri, fieldName = 'file') {
    const token = await authService.getValidAccessToken();

    if (!token) {
      throw new Error('No authentication token available');
    }

    const formData = new FormData();
    formData.append(fieldName, {
      uri: fileUri,
      type: 'image/jpeg', // Adjust based on file type
      name: 'upload.jpg',
    });

    return this.request(endpoint, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'multipart/form-data',
      },
      body: formData,
    });
  }
}

export const apiClient = new SecureAPIClient('https://api.yourapp.com');
```

### **Certificate Pinning**
```javascript
// services/certificatePinning.js
import { Platform } from 'react-native';

class CertificatePinningService {
  constructor() {
    this.pinnedCertificates = {
      'api.yourapp.com': {
        ios: [
          'sha256/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=',
          'sha256/YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY='
        ],
        android: [
          'sha256/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=',
          'sha256/YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY='
        ]
      }
    };
  }

  // Validate server certificate
  validateCertificate(hostname, certificateFingerprint) {
    const pinnedCerts = this.pinnedCertificates[hostname];

    if (!pinnedCerts) {
      console.warn(`No pinned certificates found for ${hostname}`);
      return false;
    }

    const platformCerts = pinnedCerts[Platform.OS];

    if (!platformCerts) {
      console.warn(`No pinned certificates found for ${hostname} on ${Platform.OS}`);
      return false;
    }

    const isValid = platformCerts.includes(certificateFingerprint);

    if (!isValid) {
      console.error(`Certificate pinning failed for ${hostname}`);
      console.error(`Expected: ${platformCerts.join(', ')}`);
      console.error(`Received: ${certificateFingerprint}`);
    }

    return isValid;
  }

  // Get pinned certificates for hostname
  getPinnedCertificates(hostname) {
    return this.pinnedCertificates[hostname]?.[Platform.OS] || [];
  }

  // Add pinned certificate
  addPinnedCertificate(hostname, certificateFingerprint) {
    if (!this.pinnedCertificates[hostname]) {
      this.pinnedCertificates[hostname] = { ios: [], android: [] };
    }

    if (!this.pinnedCertificates[hostname][Platform.OS].includes(certificateFingerprint)) {
      this.pinnedCertificates[hostname][Platform.OS].push(certificateFingerprint);
    }
  }

  // Remove pinned certificate
  removePinnedCertificate(hostname, certificateFingerprint) {
    if (this.pinnedCertificates[hostname]?.[Platform.OS]) {
      const index = this.pinnedCertificates[hostname][Platform.OS].indexOf(certificateFingerprint);
      if (index > -1) {
        this.pinnedCertificates[hostname][Platform.OS].splice(index, 1);
      }
    }
  }

  // Check if hostname has pinned certificates
  hasPinnedCertificates(hostname) {
    return !!(this.pinnedCertificates[hostname]?.[Platform.OS]?.length > 0);
  }
}

export const certificatePinningService = new CertificatePinningService();
```

---

## 💻 **Code Security**

### **Input Validation**
```javascript
// utils/validation.js
class InputValidator {
  constructor() {
    // Common validation patterns
    this.patterns = {
      email: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      phone: /^\+?[\d\s\-\(\)]+$/,
      url: /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/,
      password: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/,
      username: /^[a-zA-Z0-9_-]{3,20}$/,
      creditCard: /^\d{4}\s?\d{4}\s?\d{4}\s?\d{4}$/,
      zipCode: /^\d{5}(-\d{4})?$/,
    };
  }

  // Validate email
  validateEmail(email) {
    if (!email || typeof email !== 'string') {
      return { isValid: false, error: 'Email is required' };
    }

    const trimmedEmail = email.trim();

    if (trimmedEmail.length === 0) {
      return { isValid: false, error: 'Email cannot be empty' };
    }

    if (trimmedEmail.length > 254) {
      return { isValid: false, error: 'Email is too long' };
    }

    if (!this.patterns.email.test(trimmedEmail)) {
      return { isValid: false, error: 'Invalid email format' };
    }

    return { isValid: true, value: trimmedEmail };
  }

  // Validate password
  validatePassword(password) {
    if (!password || typeof password !== 'string') {
      return { isValid: false, error: 'Password is required' };
    }

    if (password.length < 8) {
      return { isValid: false, error: 'Password must be at least 8 characters long' };
    }

    if (password.length > 128) {
      return { isValid: false, error: 'Password is too long' };
    }

    if (!this.patterns.password.test(password)) {
      return {
        isValid: false,
        error: 'Password must contain at least one uppercase letter, one lowercase letter, one number, and one special character'
      };
    }

    return { isValid: true, value: password };
  }

  // Validate username
  validateUsername(username) {
    if (!username || typeof username !== 'string') {
      return { isValid: false, error: 'Username is required' };
    }

    const trimmedUsername = username.trim();

    if (trimmedUsername.length === 0) {
      return { isValid: false, error: 'Username cannot be empty' };
    }

    if (!this.patterns.username.test(trimmedUsername)) {
      return {
        isValid: false,
        error: 'Username must be 3-20 characters long and contain only letters, numbers, underscores, and hyphens'
      };
    }

    return { isValid: true, value: trimmedUsername };
  }

  // Validate URL
  validateURL(url) {
    if (!url || typeof url !== 'string') {
      return { isValid: false, error: 'URL is required' };
    }

    const trimmedURL = url.trim();

    if (!this.patterns.url.test(trimmedURL)) {
      return { isValid: false, error: 'Invalid URL format' };
    }

    return { isValid: true, value: trimmedURL };
  }

  // Sanitize input (remove potentially dangerous characters)
  sanitizeInput(input) {
    if (typeof input !== 'string') {
      return input;
    }

    // Remove null bytes
    let sanitized = input.replace(/\0/g, '');

    // Remove potential script tags
    sanitized = sanitized.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '');

    // Remove potential HTML tags
    sanitized = sanitized.replace(/<[^>]*>/g, '');

    // Remove potential SQL injection patterns
    sanitized = sanitized.replace(/('|(\\x27)|(\\x2D\\x2D)|(\\#)|(\\x23)|(\%27)|(\%23))/g, '');

    return sanitized.trim();
  }

  // Validate and sanitize input
  validateAndSanitizeInput(input, type) {
    let validationResult;

    switch (type) {
      case 'email':
        validationResult = this.validateEmail(input);
        break;
      case 'password':
        validationResult = this.validatePassword(input);
        break;
      case 'username':
        validationResult = this.validateUsername(input);
        break;
      case 'url':
        validationResult = this.validateURL(input);
        break;
      default:
        validationResult = { isValid: true, value: input };
    }

    if (validationResult.isValid) {
      validationResult.value = this.sanitizeInput(validationResult.value);
    }

    return validationResult;
  }

  // Validate object with multiple fields
  validateObject(obj, schema) {
    const errors = {};
    const validatedData = {};

    for (const [field, rules] of Object.entries(schema)) {
      const value = obj[field];
      const validation = this.validateAndSanitizeInput(value, rules.type);

      if (!validation.isValid) {
        errors[field] = validation.error;
      } else {
        validatedData[field] = validation.value;

        // Apply additional rules
        if (rules.required && (validation.value === null || validation.value === undefined || validation.value === '')) {
          errors[field] = `${field} is required`;
        }

        if (rules.minLength && validation.value.length < rules.minLength) {
          errors[field] = `${field} must be at least ${rules.minLength} characters long`;
        }

        if (rules.maxLength && validation.value.length > rules.maxLength) {
          errors[field] = `${field} must be no more than ${rules.maxLength} characters long`;
        }
      }
    }

    return {
      isValid: Object.keys(errors).length === 0,
      data: validatedData,
      errors
    };
  }
}

export const inputValidator = new InputValidator();
```

### **Secure Coding Practices**
```javascript
// utils/secureCoding.js
import { Alert } from 'react-native';

// Secure random number generation
export const secureRandom = {
  // Generate secure random bytes
  generateBytes(length) {
    const array = new Uint8Array(length);
    if (typeof crypto !== 'undefined' && crypto.getRandomValues) {
      crypto.getRandomValues(array);
    } else {
      // Fallback for older environments
      for (let i = 0; i < length; i++) {
        array[i] = Math.floor(Math.random() * 256);
      }
    }
    return array;
  },

  // Generate secure random string
  generateString(length, charset = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789') {
    const bytes = this.generateBytes(length);
    let result = '';
    for (let i = 0; i < length; i++) {
      result += charset.charAt(bytes[i] % charset.length);
    }
    return result;
  },

  // Generate secure random number
  generateNumber(min, max) {
    const range = max - min + 1;
    const bytes = this.generateBytes(4);
    const randomValue = (bytes[0] << 24) | (bytes[1] << 16) | (bytes[2] << 8) | bytes[3];
    return min + (randomValue % range);
  }
};

// Secure logging (no sensitive data)
export const secureLogger = {
  // Log without sensitive data
  log(level, message, context = {}) {
    // Remove sensitive fields
    const safeContext = this.sanitizeContext(context);

    const logEntry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      context: safeContext
    };

    // In production, send to logging service
    if (!__DEV__) {
      // sendToLoggingService(logEntry);
    }

    // Console log in development
    console.log(`[${level.toUpperCase()}] ${message}`, safeContext);
  },

  // Sanitize context to remove sensitive data
  sanitizeContext(context) {
    const sensitiveFields = [
      'password', 'token', 'secret', 'key', 'apiKey',
      'accessToken', 'refreshToken', 'authorization',
      'creditCard', 'ssn', 'socialSecurity'
    ];

    const sanitized = { ...context };

    for (const field of sensitiveFields) {
      if (sanitized[field]) {
        sanitized[field] = '[REDACTED]';
      }
    }

    return sanitized;
  },

  info(message, context) {
    this.log('info', message, context);
  },

  warn(message, context) {
    this.log('warn', message, context);
  },

  error(message, context) {
    this.log('error', message, context);
  },

  debug(message, context) {
    if (__DEV__) {
      this.log('debug', message, context);
    }
  }
};

// Secure error handling
export const secureErrorHandler = {
  // Handle errors securely
  handleError(error, context = {}) {
    // Log error securely
    secureLogger.error('Application error', {
      ...context,
      error: error.message,
      stack: error.stack,
      userAgent: navigator.userAgent,
      timestamp: Date.now()
    });

    // Show user-friendly error message
    this.showUserFriendlyError(error);

    // Report error to monitoring service
    this.reportError(error, context);
  },

  // Show user-friendly error message
  showUserFriendlyError(error) {
    let message = 'An unexpected error occurred. Please try again.';

    // Provide specific messages for known error types
    if (error.message.includes('network')) {
      message = 'Network connection error. Please check your internet connection.';
    } else if (error.message.includes('authentication')) {
      message = 'Authentication failed. Please log in again.';
    } else if (error.message.includes('permission')) {
      message = 'Permission denied. Please check your app permissions.';
    }

    Alert.alert('Error', message);
  },

  // Report error to monitoring service
  reportError(error, context) {
    // Send to error monitoring service (Sentry, Bugsnag, etc.)
    // This would integrate with your error monitoring service
    console.log('Reporting error to monitoring service:', error, context);
  },

  // Handle unhandled promise rejections
  handleUnhandledRejection(reason, promise) {
    secureLogger.error('Unhandled promise rejection', {
      reason: reason?.message || reason,
      promise: promise.toString()
    });

    this.showUserFriendlyError(new Error('An unexpected error occurred'));
  },

  // Handle uncaught exceptions
  handleUncaughtException(error) {
    secureLogger.error('Uncaught exception', {
      error: error.message,
      stack: error.stack
    });

    this.showUserFriendlyError(error);
  }
};

// Secure API response validation
export const secureResponseValidator = {
  // Validate API response structure
  validateResponse(response, expectedSchema) {
    if (!response || typeof response !== 'object') {
      throw new Error('Invalid response format');
    }

    this.validateObjectStructure(response, expectedSchema);
    return response;
  },

  // Validate object structure
  validateObjectStructure(obj, schema, path = '') {
    for (const [key, rules] of Object.entries(schema)) {
      const currentPath = path ? `${path}.${key}` : key;
      const value = obj[key];

      // Check required fields
      if (rules.required && (value === undefined || value === null)) {
        throw new Error(`Missing required field: ${currentPath}`);
      }

      // Check type
      if (value !== undefined && value !== null) {
        if (rules.type && typeof value !== rules.type) {
          throw new Error(`Invalid type for field ${currentPath}: expected ${rules.type}, got ${typeof value}`);
        }

        // Validate nested objects
        if (rules.type === 'object' && rules.schema) {
          this.validateObjectStructure(value, rules.schema, currentPath);
        }

        // Validate arrays
        if (rules.type === 'array' && Array.isArray(value)) {
          if (rules.itemSchema) {
            value.forEach((item, index) => {
              this.validateObjectStructure({ item }, { item: rules.itemSchema }, `${currentPath}[${index}]`);
            });
          }
        }
      }
    }
  }
};
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Security Fundamentals**: CIA triad, defense in depth, least privilege
- ✅ **Data Protection**: Encryption, secure storage, data sanitization
- ✅ **Authentication & Authorization**: JWT tokens, biometric authentication, secure sessions
- ✅ **API Security**: Secure API client, certificate pinning, request validation
- ✅ **Code Security**: Input validation, secure coding practices, error handling
- ✅ **Network Security**: HTTPS, certificate validation, secure communications
- ✅ **Platform-Specific Security**: iOS and Android security features
- ✅ **Security Testing**: Vulnerability scanning, penetration testing
- ✅ **Incident Response**: Security incident handling and recovery procedures

### **Best Practices**
1. **Never store sensitive data in plain text** - Always encrypt sensitive data
2. **Use HTTPS for all communications** - Never send sensitive data over HTTP
3. **Implement proper authentication** - Use strong, multi-factor authentication
4. **Validate all inputs** - Never trust user input, always validate and sanitize
5. **Follow principle of least privilege** - Give minimum required permissions
6. **Keep dependencies updated** - Regularly update libraries to fix security vulnerabilities
7. **Implement proper error handling** - Don't expose sensitive information in error messages
8. **Use secure random number generation** - For tokens, salts, and other security-related values
9. **Regular security audits** - Conduct regular security reviews and penetration testing
10. **Monitor and log security events** - Keep track of security-related activities

### **Next Steps**
- Learn advanced security topics like OAuth 2.0, OpenID Connect
- Implement security headers and CSP (Content Security Policy)
- Set up security monitoring and alerting systems
- Learn about security compliance standards (GDPR, HIPAA, etc.)
- Explore advanced encryption techniques and key management
- Implement security in CI/CD pipelines
- Learn about threat modeling and security architecture

---

## 🎯 **Assignment**

### **Task 1: Secure Authentication System**
Implement a complete secure authentication system with:
- JWT token-based authentication with refresh tokens
- Biometric authentication support
- Secure token storage and automatic refresh
- Multi-factor authentication
- Session management and timeout handling
- Secure logout functionality

### **Task 2: Data Protection Implementation**
Build a comprehensive data protection system:
- AES-256 encryption for sensitive data
- Secure local storage with encryption
- Data sanitization and validation
- Secure API communication with certificate pinning
- Automatic data cleanup and secure deletion
- Backup and recovery mechanisms

### **Task 3: Security Monitoring Dashboard**
Create a security monitoring dashboard that:
- Tracks authentication attempts and failures
- Monitors API security events
- Displays security alerts and incidents
- Shows security metrics and trends
- Provides security audit logs
- Implements real-time security notifications

---

## 📚 **Additional Resources**
- [OWASP Mobile Security Testing Guide](https://owasp.org/www-project-mobile-security-testing-guide/)
- [React Native Security Best Practices](https://reactnative.dev/docs/security)
- [iOS Security Guide](https://developer.apple.com/support/security/)
- [Android Security Best Practices](https://developer.android.com/topic/security/best-practices)
- [JWT Security Best Practices](https://tools.ietf.org/html/rfc8725)
- [SSL/TLS Best Practices](https://ssl.com/article/ssl-tls-best-practices/)

---

**Next Lesson**: [Lesson 59: Performance Optimization Techniques](Lesson%2059_%20Performance%20Optimization%20Techniques.md)