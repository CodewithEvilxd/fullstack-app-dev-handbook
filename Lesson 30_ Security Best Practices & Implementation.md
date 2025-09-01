# Lesson 30: Security Best Practices & Implementation

## 🎯 **Learning Objectives**
- Implement comprehensive security measures for React Native and Node.js applications
- Understand common security vulnerabilities and how to prevent them
- Set up secure authentication and authorization systems
- Implement data encryption and secure communication
- Follow security best practices for production deployment

## 📚 **Table of Contents**
1. [Security Fundamentals](#security-fundamentals)
2. [Authentication Security](#authentication-security)
3. [API Security](#api-security)
4. [Data Protection](#data-protection)
5. [Secure Communication](#secure-communication)
6. [Input Validation & Sanitization](#input-validation--sanitization)
7. [Session Management](#session-management)
8. [Error Handling Security](#error-handling-security)
9. [Security Monitoring](#security-monitoring)
10. [Compliance & Standards](#compliance--standards)
11. [Practical Examples](#practical-examples)

---

## 🔐 **Security Fundamentals**

### **Security Principles**
```javascript
// Defense in Depth - Multiple layers of security
const securityLayers = {
  perimeter: {
    firewall: true,
    ddos_protection: true,
    rate_limiting: true,
  },
  network: {
    https_only: true,
    certificate_pinning: true,
    vpn_required: false,
  },
  application: {
    input_validation: true,
    authentication: true,
    authorization: true,
    encryption: true,
  },
  data: {
    encryption_at_rest: true,
    encryption_in_transit: true,
    backup_encryption: true,
  },
};

// Principle of Least Privilege
const userPermissions = {
  admin: ['read', 'write', 'delete', 'manage_users'],
  moderator: ['read', 'write', 'delete'],
  user: ['read', 'write'],
  guest: ['read'],
};

// Zero Trust Architecture
const zeroTrustVerification = {
  verifyIdentity: (user) => {
    // Always verify, never trust
    return validateToken(user.token) && checkPermissions(user);
  },

  verifyRequest: (request) => {
    // Validate every request
    return {
      authenticated: validateToken(request.token),
      authorized: checkPermissions(request.user, request.action),
      sanitized: sanitizeInput(request.data),
      rateLimited: !isRateLimited(request.ip),
    };
  },
};
```

---

## 🔑 **Authentication Security**

### **Secure Password Implementation**
```javascript
const bcrypt = require('bcrypt');
const crypto = require('crypto');
const jwt = require('jsonwebtoken');

class SecureAuth {
  constructor() {
    this.saltRounds = 12;
    this.jwtSecret = process.env.JWT_SECRET;
    this.refreshTokenSecret = process.env.JWT_REFRESH_SECRET;
    this.maxLoginAttempts = 5;
    this.lockoutDuration = 15 * 60 * 1000; // 15 minutes
  }

  // Secure password hashing
  async hashPassword(password) {
    // Validate password strength first
    this.validatePasswordStrength(password);

    const salt = await bcrypt.genSalt(this.saltRounds);
    return bcrypt.hash(password, salt);
  }

  // Verify password with timing attack protection
  async verifyPassword(password, hashedPassword) {
    // Use constant-time comparison
    return await bcrypt.compare(password, hashedPassword);
  }

  // Password strength validation
  validatePasswordStrength(password) {
    const errors = [];

    if (password.length < 12) {
      errors.push('Password must be at least 12 characters long');
    }

    if (!/(?=.*[a-z])/.test(password)) {
      errors.push('Password must contain at least one lowercase letter');
    }

    if (!/(?=.*[A-Z])/.test(password)) {
      errors.push('Password must contain at least one uppercase letter');
    }

    if (!/(?=.*\d)/.test(password)) {
      errors.push('Password must contain at least one number');
    }

    if (!/(?=.*[@$!%*?&])/.test(password)) {
      errors.push('Password must contain at least one special character');
    }

    // Check for common passwords
    const commonPasswords = ['password', '123456', 'qwerty', 'admin'];
    if (commonPasswords.includes(password.toLowerCase())) {
      errors.push('Password is too common');
    }

    if (errors.length > 0) {
      throw new Error(`Password validation failed: ${errors.join(', ')}`);
    }
  }

  // Account lockout protection
  async checkLoginAttempts(userId) {
    const key = `login_attempts:${userId}`;
    const attempts = await this.getCacheValue(key) || 0;

    if (attempts >= this.maxLoginAttempts) {
      const lockoutKey = `lockout:${userId}`;
      const lockoutTime = await this.getCacheValue(lockoutKey);

      if (lockoutTime && Date.now() < lockoutTime) {
        const remainingTime = Math.ceil((lockoutTime - Date.now()) / 1000 / 60);
        throw new Error(`Account locked. Try again in ${remainingTime} minutes`);
      }

      // Reset attempts after lockout period
      await this.deleteCacheValue(key);
      await this.deleteCacheValue(lockoutKey);
    }

    return attempts;
  }

  // Record failed login attempt
  async recordFailedAttempt(userId) {
    const key = `login_attempts:${userId}`;
    const attempts = await this.getCacheValue(key) || 0;
    const newAttempts = attempts + 1;

    await this.setCacheValue(key, newAttempts, 15 * 60); // 15 minutes

    if (newAttempts >= this.maxLoginAttempts) {
      const lockoutKey = `lockout:${userId}`;
      await this.setCacheValue(lockoutKey, Date.now() + this.lockoutDuration, 15 * 60);
    }
  }

  // Clear login attempts on successful login
  async clearLoginAttempts(userId) {
    const key = `login_attempts:${userId}`;
    await this.deleteCacheValue(key);
  }

  // Secure JWT token generation
  generateAccessToken(payload) {
    return jwt.sign(
      {
        ...payload,
        iat: Math.floor(Date.now() / 1000),
        iss: 'secure-app',
        aud: 'secure-app-users',
      },
      this.jwtSecret,
      {
        expiresIn: '15m',
        algorithm: 'HS256',
      }
    );
  }

  generateRefreshToken(payload) {
    return jwt.sign(
      {
        ...payload,
        type: 'refresh',
        iat: Math.floor(Date.now() / 1000),
      },
      this.refreshTokenSecret,
      {
        expiresIn: '7d',
        algorithm: 'HS256',
      }
    );
  }

  // Secure token verification
  verifyAccessToken(token) {
    try {
      return jwt.verify(token, this.jwtSecret, {
        issuer: 'secure-app',
        audience: 'secure-app-users',
        algorithms: ['HS256'],
      });
    } catch (error) {
      if (error.name === 'TokenExpiredError') {
        throw new Error('Token expired');
      }
      if (error.name === 'JsonWebTokenError') {
        throw new Error('Invalid token');
      }
      throw error;
    }
  }

  // Token rotation for refresh tokens
  async rotateRefreshToken(oldRefreshToken) {
    const decoded = jwt.verify(oldRefreshToken, this.refreshTokenSecret);

    // Invalidate old refresh token
    await this.blacklistToken(oldRefreshToken);

    // Generate new tokens
    const newAccessToken = this.generateAccessToken({
      id: decoded.id,
      email: decoded.email,
      role: decoded.role,
    });

    const newRefreshToken = this.generateRefreshToken({
      id: decoded.id,
    });

    return { newAccessToken, newRefreshToken };
  }

  // Token blacklisting
  async blacklistToken(token) {
    const decoded = jwt.decode(token);
    if (decoded && decoded.exp) {
      const ttl = decoded.exp - Math.floor(Date.now() / 1000);
      if (ttl > 0) {
        await this.setCacheValue(`blacklist:${token}`, true, ttl);
      }
    }
  }

  // Check if token is blacklisted
  async isTokenBlacklisted(token) {
    return await this.getCacheValue(`blacklist:${token}`);
  }

  // Cache helpers (implement with Redis in production)
  async getCacheValue(key) {
    // Implement Redis get
    return null;
  }

  async setCacheValue(key, value, ttl) {
    // Implement Redis set with TTL
  }

  async deleteCacheValue(key) {
    // Implement Redis delete
  }
}

module.exports = SecureAuth;
```

---

## 🛡️ **API Security**

### **Secure API Implementation**
```javascript
const express = require('express');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const cors = require('cors');
const crypto = require('crypto');

const app = express();

// Security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'", 'https://fonts.googleapis.com'],
      fontSrc: ["'self'", 'https://fonts.gstatic.com'],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'"],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: [],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
}));

// CORS configuration
app.use(cors({
  origin: (origin, callback) => {
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-API-Key'],
}));

// Rate limiting
const createRateLimit = (windowMs, max, message) => {
  return rateLimit({
    windowMs,
    max,
    message: {
      error: message,
      retryAfter: Math.ceil(windowMs / 1000),
    },
    standardHeaders: true,
    legacyHeaders: false,
    handler: (req, res) => {
      // Log rate limit violations
      console.warn(`Rate limit exceeded for IP: ${req.ip}, Path: ${req.path}`);
      res.status(429).json({
        error: message,
        retryAfter: Math.ceil(windowMs / 1000),
      });
    },
  });
};

// Different rate limits for different endpoints
app.use('/api/auth/', createRateLimit(15 * 60 * 1000, 5, 'Too many auth attempts'));
app.use('/api/public/', createRateLimit(15 * 60 * 1000, 100, 'Too many requests'));
app.use('/api/private/', createRateLimit(15 * 60 * 1000, 1000, 'Too many requests'));

// Request size limits
app.use(express.json({
  limit: '10mb',
  verify: (req, res, buf) => {
    // Verify request body integrity
    const expectedLength = parseInt(req.headers['content-length']);
    if (buf.length !== expectedLength) {
      throw new Error('Request body length mismatch');
    }
  },
}));

app.use(express.urlencoded({
  extended: true,
  limit: '10mb',
}));

// API key authentication
const authenticateApiKey = async (req, res, next) => {
  const apiKey = req.headers['x-api-key'];

  if (!apiKey) {
    return res.status(401).json({ error: 'API key required' });
  }

  // Validate API key format
  if (!/^[a-f0-9]{64}$/i.test(apiKey)) {
    return res.status(401).json({ error: 'Invalid API key format' });
  }

  // Check API key in database/cache
  const apiKeyData = await validateApiKey(apiKey);
  if (!apiKeyData) {
    return res.status(401).json({ error: 'Invalid API key' });
  }

  req.apiUser = apiKeyData.user;
  req.apiPermissions = apiKeyData.permissions;
  next();
};

// JWT authentication
const authenticateJWT = async (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }

  try {
    // Check if token is blacklisted
    const isBlacklisted = await checkTokenBlacklist(token);
    if (isBlacklisted) {
      return res.status(401).json({ error: 'Token has been revoked' });
    }

    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({
        error: 'Token expired',
        code: 'TOKEN_EXPIRED',
      });
    }
    return res.status(403).json({ error: 'Invalid token' });
  }
};

// Request logging with sensitive data masking
const logRequest = (req, res, next) => {
  const start = Date.now();

  // Mask sensitive headers
  const safeHeaders = { ...req.headers };
  if (safeHeaders.authorization) {
    safeHeaders.authorization = '[REDACTED]';
  }
  if (safeHeaders['x-api-key']) {
    safeHeaders['x-api-key'] = '[REDACTED]';
  }

  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${new Date().toISOString()} - ${req.method} ${req.originalUrl} ${res.statusCode} ${duration}ms`, {
      ip: req.ip,
      userAgent: req.get('User-Agent'),
      headers: safeHeaders,
    });
  });

  next();
};

app.use(logRequest);

// Input validation middleware
const validateInput = (schema) => {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, { abortEarly: false });

    if (error) {
      const errors = error.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message,
      }));

      return res.status(400).json({
        error: 'Validation failed',
        details: errors,
      });
    }

    req.body = value; // Use sanitized values
    next();
  };
};

// SQL injection prevention (when using SQL databases)
const sanitizeSqlInput = (input) => {
  if (typeof input === 'string') {
    // Remove potentially dangerous characters
    return input.replace(/['";\\]/g, '');
  }
  return input;
};

// XSS prevention
const sanitizeHtml = (input) => {
  if (typeof input === 'string') {
    return input
      .replace(/</g, '<')
      .replace(/>/g, '>')
      .replace(/"/g, '"')
      .replace(/'/g, '&#x27;')
      .replace(/\//g, '&#x2F;');
  }
  return input;
};

// Secure file upload
const multer = require('multer');
const path = require('path');

const fileFilter = (req, file, cb) => {
  // Allowed file types
  const allowedTypes = [
    'image/jpeg',
    'image/png',
    'image/gif',
    'application/pdf',
    'text/plain',
  ];

  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error(`File type ${file.mimetype} is not allowed`), false);
  }
};

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');
  },
  filename: (req, file, cb) => {
    // Generate secure filename
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    const extension = path.extname(file.originalname);
    const basename = path.basename(file.originalname, extension).replace(/[^a-zA-Z0-9]/g, '_');
    cb(null, `${basename}-${uniqueSuffix}${extension}`);
  },
});

const upload = multer({
  storage,
  fileFilter,
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB
  },
});

// Error handling middleware
app.use((error, req, res, next) => {
  console.error('Error:', error);

  // Don't leak error details in production
  const isDevelopment = process.env.NODE_ENV === 'development';

  res.status(error.status || 500).json({
    error: isDevelopment ? error.message : 'Internal server error',
    ...(isDevelopment && { stack: error.stack }),
  });
});

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
  });
});

module.exports = app;
```

---

## 🔒 **Data Protection**

### **Data Encryption**
```javascript
const crypto = require('crypto');
const fs = require('fs').promises;

class DataEncryption {
  constructor() {
    this.algorithm = 'aes-256-gcm';
    this.keyLength = 32; // 256 bits
    this.ivLength = 16; // 128 bits
    this.tagLength = 16; // 128 bits
  }

  // Generate encryption key
  generateKey() {
    return crypto.randomBytes(this.keyLength);
  }

  // Derive key from password
  deriveKey(password, salt) {
    return crypto.scryptSync(password, salt, this.keyLength);
  }

  // Encrypt data
  encrypt(text, key) {
    const iv = crypto.randomBytes(this.ivLength);
    const cipher = crypto.createCipher(this.algorithm, key);

    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');

    const authTag = cipher.getAuthTag();

    return {
      encrypted,
      iv: iv.toString('hex'),
      authTag: authTag.toString('hex'),
    };
  }

  // Decrypt data
  decrypt(encryptedData, key) {
    const { encrypted, iv, authTag } = encryptedData;

    const decipher = crypto.createDecipher(this.algorithm, key);
    decipher.setAuthTag(Buffer.from(authTag, 'hex'));

    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');

    return decrypted;
  }

  // Encrypt file
  async encryptFile(inputPath, outputPath, key) {
    const inputBuffer = await fs.readFile(inputPath);
    const encrypted = this.encrypt(inputBuffer.toString('base64'), key);

    const encryptedData = JSON.stringify(encrypted);
    await fs.writeFile(outputPath, encryptedData, 'utf8');
  }

  // Decrypt file
  async decryptFile(inputPath, outputPath, key) {
    const encryptedData = await fs.readFile(inputPath, 'utf8');
    const encrypted = JSON.parse(encryptedData);
    const decrypted = this.decrypt(encrypted, key);

    const outputBuffer = Buffer.from(decrypted, 'base64');
    await fs.writeFile(outputPath, outputBuffer);
  }

  // Hash sensitive data (one-way)
  hashData(data, saltRounds = 12) {
    const salt = crypto.randomBytes(16);
    const hash = crypto.scryptSync(data, salt, 64);

    return {
      hash: hash.toString('hex'),
      salt: salt.toString('hex'),
    };
  }

  // Verify hashed data
  verifyHash(data, hashData) {
    const { hash, salt } = hashData;
    const hashBuffer = Buffer.from(hash, 'hex');
    const saltBuffer = Buffer.from(salt, 'hex');

    const computedHash = crypto.scryptSync(data, saltBuffer, 64);
    return crypto.timingSafeEqual(hashBuffer, computedHash);
  }
}

// Database field encryption
class DatabaseEncryption {
  constructor(encryptionService) {
    this.encryption = encryptionService;
    this.key = this.encryption.generateKey();
  }

  // Encrypt field before saving to database
  encryptField(value) {
    if (!value) return value;
    return this.encryption.encrypt(JSON.stringify(value), this.key);
  }

  // Decrypt field when retrieving from database
  decryptField(encryptedValue) {
    if (!encryptedValue) return encryptedValue;

    try {
      const decrypted = this.encryption.decrypt(encryptedValue, this.key);
      return JSON.parse(decrypted);
    } catch (error) {
      console.error('Failed to decrypt field:', error);
      return null;
    }
  }
}

// Secure environment variables
class SecureConfig {
  constructor() {
    this.encryption = new DataEncryption();
    this.configKey = process.env.CONFIG_ENCRYPTION_KEY;
  }

  // Encrypt sensitive config values
  encryptConfigValue(value) {
    if (!this.configKey) {
      throw new Error('CONFIG_ENCRYPTION_KEY not set');
    }

    const key = Buffer.from(this.configKey, 'hex');
    return this.encryption.encrypt(value, key);
  }

  // Decrypt sensitive config values
  decryptConfigValue(encryptedValue) {
    if (!this.configKey) {
      throw new Error('CONFIG_ENCRYPTION_KEY not set');
    }

    const key = Buffer.from(this.configKey, 'hex');
    return this.encryption.decrypt(encryptedValue, key);
  }

  // Secure database URL
  getSecureDatabaseUrl() {
    const encryptedUrl = process.env.DATABASE_URL_ENCRYPTED;
    if (encryptedUrl) {
      return this.decryptConfigValue(JSON.parse(encryptedUrl));
    }
    return process.env.DATABASE_URL;
  }
}

module.exports = {
  DataEncryption,
  DatabaseEncryption,
  SecureConfig,
};
```

---

## 🔐 **Secure Communication**

### **HTTPS & SSL/TLS**
```javascript
const https = require('https');
const express = require('express');
const fs = require('fs');

const app = express();

// SSL/TLS configuration
const sslOptions = {
  key: fs.readFileSync('path/to/private-key.pem'),
  cert: fs.readFileSync('path/to/certificate.pem'),
  ca: fs.readFileSync('path/to/ca-bundle.pem'),
  // Security options
  secureOptions: crypto.constants.SSL_OP_NO_TLSv1 | crypto.constants.SSL_OP_NO_TLSv1_1,
  ciphers: [
    'ECDHE-RSA-AES128-GCM-SHA256',
    'ECDHE-RSA-AES256-GCM-SHA384',
    'ECDHE-RSA-AES128-SHA256',
    'ECDHE-RSA-AES256-SHA384',
  ].join(':'),
  honorCipherOrder: true,
  requestCert: false,
  rejectUnauthorized: false,
};

// HSTS middleware
app.use((req, res, next) => {
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
  next();
});

// Certificate pinning (client-side)
const certificatePinning = {
  // For React Native apps
  publicKeyHashes: [
    'sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=',
    'sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=',
  ],

  validateCertificate: (certificate) => {
    const certHash = crypto.createHash('sha256')
      .update(certificate.raw)
      .digest('base64');

    return certificatePinning.publicKeyHashes.includes(`sha256/${certHash}`);
  },
};

// Secure WebSocket
const WebSocket = require('ws');
const wss = new WebSocket.Server({
  server: https.createServer(sslOptions, app),
  perMessageDeflate: false, // Disable compression for security
});

// WebSocket security middleware
wss.on('connection', (ws, req) => {
  // Validate origin
  const origin = req.headers.origin;
  const allowedOrigins = ['https://yourapp.com', 'https://app.yourapp.com'];

  if (!allowedOrigins.includes(origin)) {
    ws.close(1008, 'Policy violation');
    return;
  }

  // Rate limiting for WebSocket connections
  const clientIp = req.connection.remoteAddress;
  if (isRateLimited(clientIp)) {
    ws.close(1008, 'Rate limit exceeded');
    return;
  }

  // Authenticate WebSocket connection
  const token = req.url.split('token=')[1];
  if (!token || !validateToken(token)) {
    ws.close(1008, 'Authentication failed');
    return;
  }

  ws.on('message', (message) => {
    try {
      const data = JSON.parse(message);

      // Validate message structure
      if (!data.type || typeof data.type !== 'string') {
        ws.send(JSON.stringify({ error: 'Invalid message format' }));
        return;
      }

      // Process message based on type
      handleWebSocketMessage(ws, data);
    } catch (error) {
      console.error('WebSocket message error:', error);
      ws.send(JSON.stringify({ error: 'Invalid message' }));
    }
  });

  ws.on('close', () => {
    console.log('WebSocket connection closed');
  });

  ws.on('error', (error) => {
    console.error('WebSocket error:', error);
  });
});
```

---

## ✅ **Input Validation & Sanitization**

### **Comprehensive Input Validation**
```javascript
const Joi = require('joi');
const validator = require('validator');
const DOMPurify = require('dompurify');
const { JSDOM } = require('jsdom');

class InputValidator {
  constructor() {
    // Initialize DOMPurify
    const window = new JSDOM('').window;
    this.DOMPurify = DOMPurify(window);
  }

  // User registration validation schema
  getUserRegistrationSchema() {
    return Joi.object({
      firstName: Joi.string()
        .min(2)
        .max(50)
        .pattern(/^[a-zA-Z\s]+$/)
        .required()
        .messages({
          'string.pattern.base': 'First name can only contain letters and spaces',
        }),

      lastName: Joi.string()
        .min(2)
        .max(50)
        .pattern(/^[a-zA-Z\s]+$/)
        .required(),

      email: Joi.string()
        .email({ minDomainSegments: 2 })
        .lowercase()
        .required(),

      password: Joi.string()
        .min(12)
        .max(128)
        .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
        .required()
        .messages({
          'string.pattern.base': 'Password must contain at least one uppercase letter, one lowercase letter, one number, and one special character',
        }),

      confirmPassword: Joi.string()
        .valid(Joi.ref('password'))
        .required()
        .messages({
          'any.only': 'Passwords do not match',
        }),

      phoneNumber: Joi.string()
        .pattern(/^\+?[1-9]\d{1,14}$/)
        .optional(),

      dateOfBirth: Joi.date()
        .max('now')
        .min(new Date(Date.now() - 100 * 365 * 24 * 60 * 60 * 1000)) // 100 years ago
        .optional(),

      termsAccepted: Joi.boolean()
        .valid(true)
        .required()
        .messages({
          'any.only': 'You must accept the terms and conditions',
        }),
    });
  }

  // API request validation schema
  getApiRequestSchema() {
    return Joi.object({
      userId: Joi.string()
        .pattern(/^[a-f\d]{24}$/i)
        .required()
        .messages({
          'string.pattern.base': 'Invalid user ID format',
        }),

      action: Joi.string()
        .valid('create', 'read', 'update', 'delete')
        .required(),

      data: Joi.object()
        .when('action', {
          is: Joi.valid('create', 'update'),
          then: Joi.required(),
          otherwise: Joi.forbidden(),
        }),

      limit: Joi.number()
        .integer()
        .min(1)
        .max(100)
        .default(10),

      offset: Joi.number()
        .integer()
        .min(0)
        .default(0),
    });
  }

  // File upload validation
  validateFileUpload(file) {
    const errors = [];

    // Check file size (5MB limit)
    const maxSize = 5 * 1024 * 1024;
    if (file.size > maxSize) {
      errors.push('File size exceeds 5MB limit');
    }

    // Check file type
    const allowedTypes = [
      'image/jpeg',
      'image/png',
      'image/gif',
      'application/pdf',
      'text/plain',
    ];

    if (!allowedTypes.includes(file.mimetype)) {
      errors.push(`File type ${file.mimetype} is not allowed`);
    }

    // Check filename for malicious patterns
    const maliciousPatterns = [
      /\.\./,  // Directory traversal
      /[<>:"|?*]/,  // Invalid filename characters
      /^\./,   // Hidden files
      /script/i,  // Script files
      /exe$/i,  // Executable files
    ];

    for (const pattern of maliciousPatterns) {
      if (pattern.test(file.originalname)) {
        errors.push('Invalid filename');
        break;
      }
    }

    return {
      isValid: errors.length === 0,
      errors,
    };
  }

  // Sanitize HTML input
  sanitizeHtml(input) {
    if (typeof input !== 'string') return input;

    return this.DOMPurify.sanitize(input, {
      ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6'],
      ALLOWED_ATTR: [],
    });
  }

  // Sanitize SQL input (additional layer)
  sanitizeSql(input) {
    if (typeof input !== 'string') return input;

    // Remove potentially dangerous characters
    return input.replace(/['";\\]/g, '');
  }

  // Validate and sanitize email
  validateEmail(email) {
    if (!validator.isEmail(email)) {
      throw new Error('Invalid email format');
    }

    // Additional validation
    if (email.length > 254) {
      throw new Error('Email is too long');
    }

    // Check for disposable email domains
    const disposableDomains = ['10minutemail.com', 'temp-mail.org'];
    const domain = email.split('@')[1];

    if (disposableDomains.includes(domain)) {
      throw new Error('Disposable email addresses are not allowed');
    }

    return validator.normalizeEmail(email);
  }

  // Validate URL
  validateUrl(url) {
    if (!validator.isURL(url, {
      protocols: ['http', 'https'],
      require_protocol: true,
    })) {
      throw new Error('Invalid URL format');
    }

    // Additional security checks
    if (url.includes('localhost') || url.includes('127.0.0.1')) {
      throw new Error('Local URLs are not allowed');
    }

    return url;
  }

  // Comprehensive input sanitization
  sanitizeInput(input, options = {}) {
    const {
      allowHtml = false,
      maxLength = 10000,
      trim = true,
    } = options;

    if (typeof input !== 'string') return input;

    let sanitized = input;

    // Trim whitespace
    if (trim) {
      sanitized = sanitized.trim();
    }

    // Limit length
    if (sanitized.length > maxLength) {
      sanitized = sanitized.substring(0, maxLength);
    }

    // Sanitize HTML if not allowed
    if (!allowHtml) {
      sanitized = this.sanitizeHtml(sanitized);
    }

    // Remove null bytes and other control characters
    sanitized = sanitized.replace(/[\x00-\x1F\x7F]/g, '');

    return sanitized;
  }

  // Validate complete request
  async validateRequest(req, schema) {
    try {
      // Validate request body
      const { error: bodyError, value: bodyValue } = schema.validate(req.body, {
        abortEarly: false,
        stripUnknown: true,
      });

      if (bodyError) {
        const errors = bodyError.details.map(detail => ({
          field: detail.path.join('.'),
          message: detail.message,
          value: detail.context.value,
        }));

        return {
          isValid: false,
          errors,
          sanitizedData: null,
        };
      }

      // Validate query parameters
      const querySchema = Joi.object({
        limit: Joi.number().integer().min(1).max(100).default(10),
        offset: Joi.number().integer().min(0).default(0),
        sort: Joi.string().pattern(/^[a-zA-Z_]+:(asc|desc)$/),
      });

      const { error: queryError, value: queryValue } = querySchema.validate(req.query, {
        abortEarly: false,
        stripUnknown: true,
      });

      if (queryError) {
        const errors = queryError.details.map(detail => ({
          field: `query.${detail.path.join('.')}`,
          message: detail.message,
        }));

        return {
          isValid: false,
          errors,
          sanitizedData: null,
        };
      }

      // Sanitize all string inputs
      const sanitizedBody = this.sanitizeObjectStrings(bodyValue);
      const sanitizedQuery = this.sanitizeObjectStrings(queryValue);

      return {
        isValid: true,
        errors: [],
        sanitizedData: {
          body: sanitizedBody,
          query: sanitizedQuery,
        },
      };
    } catch (error) {
      return {
        isValid: false,
        errors: [{ field: 'validation', message: error.message }],
        sanitizedData: null,
      };
    }
  }

  // Helper to sanitize all strings in an object
  sanitizeObjectStrings(obj) {
    if (typeof obj !== 'object' || obj === null) return obj;

    const sanitized = { ...obj };

    for (const [key, value] of Object.entries(sanitized)) {
      if (typeof value === 'string') {
        sanitized[key] = this.sanitizeInput(value);
      } else if (typeof value === 'object') {
        sanitized[key] = this.sanitizeObjectStrings(value);
      }
    }

    return sanitized;
  }
}

module.exports = InputValidator;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Security Fundamentals**: Defense in depth, least privilege, zero trust
- ✅ **Authentication Security**: Secure password hashing, account lockout, JWT security
- ✅ **API Security**: Rate limiting, input validation, secure headers, CORS
- ✅ **Data Protection**: Encryption at rest and in transit, secure key management
- ✅ **Secure Communication**: HTTPS, SSL/TLS, certificate pinning
- ✅ **Input Validation**: Comprehensive validation schemas, sanitization
- ✅ **Session Management**: Secure session handling, token rotation
- ✅ **Error Handling**: Secure error responses, logging without data leakage
- ✅ **Security Monitoring**: Alerting, logging, intrusion detection
- ✅ **Compliance**: GDPR, HIPAA, security standards

### **Best Practices**
1. **Never trust user input** - Always validate and sanitize
2. **Use HTTPS everywhere** - Encrypt all communications
3. **Implement proper authentication** - Strong passwords, multi-factor authentication
4. **Follow principle of least privilege** - Minimal required permissions
5. **Keep dependencies updated** - Regular security patches
6. **Log security events** - Monitor for suspicious activities
7. **Use secure headers** - CSP, HSTS, X-Frame-Options
8. **Encrypt sensitive data** - Both at rest and in transit
9. **Implement rate limiting** - Prevent abuse and DoS attacks
10. **Regular security audits** - Penetration testing and code reviews

### **Next Steps**
- Implement security monitoring and alerting systems
- Set up automated security testing and vulnerability scanning
- Create incident response plans and procedures
- Implement security awareness training for development teams
- Regular security audits and compliance checks
- Stay updated with latest security threats and patches

---

## 🎯 **Assignment**

### **Task 1: Secure Authentication System**
Create a complete secure authentication system with:
- Strong password policies and validation
- Account lockout protection
- JWT token security with rotation
- Multi-factor authentication (optional)
- Secure password reset functionality
- Session management and timeout

### **Task 2: Secure API Implementation**
Build a secure REST API with:
- Comprehensive input validation and sanitization
- Rate limiting and DDoS protection
- Secure headers and CORS configuration
- API key authentication
- Request/response encryption
- Security logging and monitoring

### **Task 3: Data Protection System**
Implement a data protection system with:
- Database field encryption
- File encryption for uploads
- Secure configuration management
- Data backup encryption
- GDPR compliance features
- Audit logging for data access

---

## 📚 **Additional Resources**
- [OWASP Top 10](https://owasp.org/www-project-top-ten/) - Web application security risks
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework) - Security standards
- [JWT Security Best Practices](https://tools.ietf.org/html/rfc8725)
- [SSL/TLS Best Practices](https://ssl-config.mozilla.org/)
- [Security Headers](https://securityheaders.com/)
- [Node.js Security Best Practices](https://nodejs.org/en/docs/guides/security/)
- [React Native Security](https://reactnative.dev/docs/security)

---

**Next Lesson**: [Lesson 31: Advanced State Management (Redux Toolkit, Zustand)](Lesson%2031_%20Advanced%20State%20Management%20(Redux%20Toolkit,%20Zustand).md)