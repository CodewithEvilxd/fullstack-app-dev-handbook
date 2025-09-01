# Lesson 21: Backend Fundamentals (Node.js & Express)

## 🎯 **Learning Objectives**
- Understand server-side development with Node.js
- Build RESTful APIs with Express.js
- Implement authentication and authorization
- Handle database operations with MongoDB
- Create scalable backend architecture

## 📚 **Table of Contents**
1. [Introduction to Backend Development](#introduction-to-backend-development)
2. [Node.js Fundamentals](#nodejs-fundamentals)
3. [Express.js Framework](#expressjs-framework)
4. [RESTful API Design](#restful-api-design)
5. [Authentication & Authorization](#authentication--authorization)
6. [Database Integration](#database-integration)
7. [Error Handling & Validation](#error-handling--validation)
8. [Security Best Practices](#security-best-practices)
9. [API Testing](#api-testing)
10. [Practical Examples](#practical-examples)

---

## 🚀 **Introduction to Backend Development**

Backend development involves creating the server-side logic that powers your applications. Node.js with Express.js provides a robust foundation for building scalable APIs.

### **Backend Architecture**
```
Client (React Native) ↔️ API Gateway ↔️ Microservices
                              ↕️
                        Database Layer
                              ↕️
                       Caching Layer
```

### **Key Concepts**
- **Server**: Handles requests and responses
- **API**: Interface for client-server communication
- **Database**: Persistent data storage
- **Authentication**: User identity verification
- **Authorization**: Access control and permissions

---

## 🟢 **Node.js Fundamentals**

### **Basic Node.js Server**
```javascript
// server.js
const http = require('http');

const server = http.createServer((req, res) => {
  console.log(`${req.method} ${req.url}`);

  if (req.url === '/' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello, World!');
  } else if (req.url === '/api/users' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ users: [] }));
  } else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('Not Found');
  }
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### **File System Operations**
```javascript
const fs = require('fs').promises;
const path = require('path');

// Read file asynchronously
async function readFile(filePath) {
  try {
    const data = await fs.readFile(filePath, 'utf8');
    return data;
  } catch (error) {
    console.error('Error reading file:', error);
    throw error;
  }
}

// Write file asynchronously
async function writeFile(filePath, content) {
  try {
    await fs.writeFile(filePath, content, 'utf8');
    console.log('File written successfully');
  } catch (error) {
    console.error('Error writing file:', error);
    throw error;
  }
}

// Create directory
async function createDirectory(dirPath) {
  try {
    await fs.mkdir(dirPath, { recursive: true });
    console.log('Directory created successfully');
  } catch (error) {
    console.error('Error creating directory:', error);
    throw error;
  }
}

// List directory contents
async function listDirectory(dirPath) {
  try {
    const files = await fs.readdir(dirPath);
    return files;
  } catch (error) {
    console.error('Error listing directory:', error);
    throw error;
  }
}

// File operations example
async function fileOperationsDemo() {
  const filePath = './data/users.json';
  const dirPath = './data';

  // Create directory if it doesn't exist
  await createDirectory(dirPath);

  // Write sample data
  const sampleData = JSON.stringify([
    { id: 1, name: 'John Doe', email: 'john@example.com' },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
  ], null, 2);

  await writeFile(filePath, sampleData);

  // Read and parse data
  const data = await readFile(filePath);
  const users = JSON.parse(data);
  console.log('Users:', users);

  // List directory
  const files = await listDirectory(dirPath);
  console.log('Files in directory:', files);
}

module.exports = {
  readFile,
  writeFile,
  createDirectory,
  listDirectory,
  fileOperationsDemo,
};
```

### **Environment Variables**
```javascript
// .env
PORT=3000
NODE_ENV=development
DATABASE_URL=mongodb://localhost:27017/myapp
JWT_SECRET=your-secret-key-here
API_KEY=your-api-key-here

// config.js
require('dotenv').config();

const config = {
  port: process.env.PORT || 3000,
  nodeEnv: process.env.NODE_ENV || 'development',
  databaseUrl: process.env.DATABASE_URL,
  jwtSecret: process.env.JWT_SECRET,
  apiKey: process.env.API_KEY,
};

module.exports = config;
```

---

## ⚡ **Express.js Framework**

### **Basic Express Server**
```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');

const app = express();

// Middleware
app.use(helmet()); // Security headers
app.use(cors()); // Cross-origin resource sharing
app.use(morgan('combined')); // Logging
app.use(express.json()); // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse URL-encoded bodies

// Routes
app.get('/', (req, res) => {
  res.json({
    message: 'Welcome to Express API',
    timestamp: new Date().toISOString(),
  });
});

app.get('/api/health', (req, res) => {
  res.json({
    status: 'OK',
    uptime: process.uptime(),
    timestamp: new Date().toISOString(),
  });
});

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({
    error: 'Something went wrong!',
    message: process.env.NODE_ENV === 'development' ? err.message : undefined,
  });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({
    error: 'Not Found',
    message: `Route ${req.method} ${req.path} not found`,
  });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Express server running on port ${PORT}`);
});

module.exports = app;
```

### **Route Organization**
```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

// In-memory user storage (replace with database in production)
let users = [
  { id: 1, name: 'John Doe', email: 'john@example.com' },
  { id: 2, name: 'Jane Smith', email: 'jane@example.com' },
];

// GET /users - Get all users
router.get('/', (req, res) => {
  const { page = 1, limit = 10, search } = req.query;

  let filteredUsers = users;

  // Search functionality
  if (search) {
    filteredUsers = users.filter(user =>
      user.name.toLowerCase().includes(search.toLowerCase()) ||
      user.email.toLowerCase().includes(search.toLowerCase())
    );
  }

  // Pagination
  const startIndex = (page - 1) * limit;
  const endIndex = startIndex + parseInt(limit);
  const paginatedUsers = filteredUsers.slice(startIndex, endIndex);

  res.json({
    users: paginatedUsers,
    pagination: {
      page: parseInt(page),
      limit: parseInt(limit),
      total: filteredUsers.length,
      pages: Math.ceil(filteredUsers.length / limit),
    },
  });
});

// GET /users/:id - Get user by ID
router.get('/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));

  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  res.json({ user });
});

// POST /users - Create new user
router.post('/', (req, res) => {
  const { name, email } = req.body;

  // Validation
  if (!name || !email) {
    return res.status(400).json({
      error: 'Validation error',
      message: 'Name and email are required',
    });
  }

  // Check if email already exists
  const existingUser = users.find(u => u.email === email);
  if (existingUser) {
    return res.status(409).json({
      error: 'Conflict',
      message: 'Email already exists',
    });
  }

  const newUser = {
    id: users.length + 1,
    name,
    email,
    createdAt: new Date().toISOString(),
  };

  users.push(newUser);
  res.status(201).json({ user: newUser });
});

// PUT /users/:id - Update user
router.put('/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));

  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  const { name, email } = req.body;

  // Validation
  if (!name || !email) {
    return res.status(400).json({
      error: 'Validation error',
      message: 'Name and email are required',
    });
  }

  // Check if email already exists (excluding current user)
  const existingUser = users.find(u => u.email === email && u.id !== user.id);
  if (existingUser) {
    return res.status(409).json({
      error: 'Conflict',
      message: 'Email already exists',
    });
  }

  user.name = name;
  user.email = email;
  user.updatedAt = new Date().toISOString();

  res.json({ user });
});

// DELETE /users/:id - Delete user
router.delete('/:id', (req, res) => {
  const userIndex = users.findIndex(u => u.id === parseInt(req.params.id));

  if (userIndex === -1) {
    return res.status(404).json({ error: 'User not found' });
  }

  const deletedUser = users.splice(userIndex, 1)[0];
  res.json({ user: deletedUser });
});

module.exports = router;
```

### **Middleware Implementation**
```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');
const config = require('../config');

const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN

  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }

  jwt.verify(token, config.jwtSecret, (err, user) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid or expired token' });
    }

    req.user = user;
    next();
  });
};

const requireRole = (roles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }

    next();
  };
};

module.exports = {
  authenticateToken,
  requireRole,
};

// middleware/validation.js
const validateUser = (req, res, next) => {
  const { name, email } = req.body;

  const errors = [];

  if (!name || typeof name !== 'string' || name.trim().length < 2) {
    errors.push('Name must be at least 2 characters long');
  }

  if (!email || typeof email !== 'string') {
    errors.push('Email is required');
  } else {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      errors.push('Invalid email format');
    }
  }

  if (errors.length > 0) {
    return res.status(400).json({
      error: 'Validation failed',
      details: errors,
    });
  }

  // Sanitize input
  req.body.name = name.trim();
  req.body.email = email.toLowerCase().trim();

  next();
};

module.exports = {
  validateUser,
};

// middleware/rateLimit.js
const rateLimit = require('express-rate-limit');

const createRateLimit = (windowMs, maxRequests, message) => {
  return rateLimit({
    windowMs,
    max: maxRequests,
    message: {
      error: 'Too many requests',
      message,
    },
    standardHeaders: true,
    legacyHeaders: false,
  });
};

const authRateLimit = createRateLimit(
  15 * 60 * 1000, // 15 minutes
  5, // 5 attempts
  'Too many authentication attempts, please try again later'
);

const apiRateLimit = createRateLimit(
  15 * 60 * 1000, // 15 minutes
  100, // 100 requests
  'Too many API requests, please try again later'
);

module.exports = {
  authRateLimit,
  apiRateLimit,
};
```

---

## 🔄 **RESTful API Design**

### **REST Principles**
```javascript
// Complete RESTful API example
const express = require('express');
const router = express.Router();

// Resource: /api/posts

// GET /api/posts - List all posts
router.get('/posts', async (req, res) => {
  try {
    const { page = 1, limit = 10, author, category } = req.query;

    const query = {};
    if (author) query.author = author;
    if (category) query.category = category;

    const posts = await Post.find(query)
      .populate('author', 'name email')
      .sort({ createdAt: -1 })
      .limit(limit * 1)
      .skip((page - 1) * limit);

    const total = await Post.countDocuments(query);

    res.json({
      posts,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total,
        pages: Math.ceil(total / limit),
      },
    });
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch posts' });
  }
});

// GET /api/posts/:id - Get single post
router.get('/posts/:id', async (req, res) => {
  try {
    const post = await Post.findById(req.params.id)
      .populate('author', 'name email')
      .populate('comments.author', 'name');

    if (!post) {
      return res.status(404).json({ error: 'Post not found' });
    }

    res.json({ post });
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch post' });
  }
});

// POST /api/posts - Create new post
router.post('/posts', authenticateToken, async (req, res) => {
  try {
    const { title, content, category, tags } = req.body;

    const post = new Post({
      title,
      content,
      category,
      tags: tags || [],
      author: req.user.id,
    });

    await post.save();

    const populatedPost = await Post.findById(post._id)
      .populate('author', 'name email');

    res.status(201).json({ post: populatedPost });
  } catch (error) {
    res.status(500).json({ error: 'Failed to create post' });
  }
});

// PUT /api/posts/:id - Update post
router.put('/posts/:id', authenticateToken, async (req, res) => {
  try {
    const post = await Post.findById(req.params.id);

    if (!post) {
      return res.status(404).json({ error: 'Post not found' });
    }

    // Check ownership
    if (post.author.toString() !== req.user.id) {
      return res.status(403).json({ error: 'Not authorized to update this post' });
    }

    const { title, content, category, tags } = req.body;

    post.title = title || post.title;
    post.content = content || post.content;
    post.category = category || post.category;
    post.tags = tags || post.tags;
    post.updatedAt = new Date();

    await post.save();

    const updatedPost = await Post.findById(post._id)
      .populate('author', 'name email');

    res.json({ post: updatedPost });
  } catch (error) {
    res.status(500).json({ error: 'Failed to update post' });
  }
});

// DELETE /api/posts/:id - Delete post
router.delete('/posts/:id', authenticateToken, async (req, res) => {
  try {
    const post = await Post.findById(req.params.id);

    if (!post) {
      return res.status(404).json({ error: 'Post not found' });
    }

    // Check ownership
    if (post.author.toString() !== req.user.id) {
      return res.status(403).json({ error: 'Not authorized to delete this post' });
    }

    await Post.findByIdAndDelete(req.params.id);
    res.json({ message: 'Post deleted successfully' });
  } catch (error) {
    res.status(500).json({ error: 'Failed to delete post' });
  }
});

// Sub-resources
// GET /api/posts/:id/comments - Get post comments
router.get('/posts/:id/comments', async (req, res) => {
  try {
    const post = await Post.findById(req.params.id)
      .populate('comments.author', 'name');

    if (!post) {
      return res.status(404).json({ error: 'Post not found' });
    }

    res.json({ comments: post.comments });
  } catch (error) {
    res.status(500).json({ error: 'Failed to fetch comments' });
  }
});

// POST /api/posts/:id/comments - Add comment to post
router.post('/posts/:id/comments', authenticateToken, async (req, res) => {
  try {
    const post = await Post.findById(req.params.id);

    if (!post) {
      return res.status(404).json({ error: 'Post not found' });
    }

    const { content } = req.body;

    const newComment = {
      content,
      author: req.user.id,
      createdAt: new Date(),
    };

    post.comments.push(newComment);
    await post.save();

    const updatedPost = await Post.findById(post._id)
      .populate('comments.author', 'name');

    res.status(201).json({
      comment: updatedPost.comments[updatedPost.comments.length - 1]
    });
  } catch (error) {
    res.status(500).json({ error: 'Failed to add comment' });
  }
});

module.exports = router;
```

### **API Versioning**
```javascript
// routes/v1/index.js
const express = require('express');
const router = express.Router();

// Mount versioned routes
router.use('/users', require('./users'));
router.use('/posts', require('./posts'));
router.use('/auth', require('./auth'));

// API info
router.get('/', (req, res) => {
  res.json({
    version: '1.0.0',
    description: 'Blog API v1',
    endpoints: {
      users: '/api/v1/users',
      posts: '/api/v1/posts',
      auth: '/api/v1/auth',
    },
  });
});

module.exports = router;

// server.js - API versioning setup
const express = require('express');
const app = express();

// API versioning
app.use('/api/v1', require('./routes/v1'));

// Redirect old API calls
app.use('/api', (req, res) => {
  res.redirect(301, `/api/v1${req.path}`);
});

// Handle unsupported versions
app.use('/api/v*', (req, res) => {
  res.status(400).json({
    error: 'API version not supported',
    supportedVersions: ['v1'],
  });
});
```

---

## 🔐 **Authentication & Authorization**

### **JWT Authentication**
```javascript
// utils/jwt.js
const jwt = require('jsonwebtoken');
const config = require('../config');

const generateToken = (payload, expiresIn = '24h') => {
  return jwt.sign(payload, config.jwtSecret, { expiresIn });
};

const verifyToken = (token) => {
  try {
    return jwt.verify(token, config.jwtSecret);
  } catch (error) {
    throw new Error('Invalid token');
  }
};

const generateRefreshToken = (payload) => {
  return jwt.sign(payload, config.jwtRefreshSecret, { expiresIn: '7d' });
};

const verifyRefreshToken = (token) => {
  try {
    return jwt.verify(token, config.jwtRefreshSecret);
  } catch (error) {
    throw new Error('Invalid refresh token');
  }
};

module.exports = {
  generateToken,
  verifyToken,
  generateRefreshToken,
  verifyRefreshToken,
};

// routes/auth.js
const express = require('express');
const bcrypt = require('bcrypt');
const router = express.Router();
const { generateToken, generateRefreshToken, verifyRefreshToken } = require('../utils/jwt');

// In-memory user storage (replace with database)
const users = [];
const refreshTokens = [];

// POST /auth/register
router.post('/register', async (req, res) => {
  try {
    const { name, email, password } = req.body;

    // Check if user already exists
    const existingUser = users.find(u => u.email === email);
    if (existingUser) {
      return res.status(409).json({ error: 'User already exists' });
    }

    // Hash password
    const saltRounds = 10;
    const hashedPassword = await bcrypt.hash(password, saltRounds);

    // Create user
    const user = {
      id: users.length + 1,
      name,
      email,
      password: hashedPassword,
      role: 'user',
      createdAt: new Date(),
    };

    users.push(user);

    // Generate tokens
    const accessToken = generateToken({
      id: user.id,
      email: user.email,
      role: user.role,
    });

    const refreshToken = generateRefreshToken({
      id: user.id,
      email: user.email,
    });

    // Store refresh token
    refreshTokens.push(refreshToken);

    res.status(201).json({
      user: {
        id: user.id,
        name: user.name,
        email: user.email,
        role: user.role,
      },
      accessToken,
      refreshToken,
    });
  } catch (error) {
    res.status(500).json({ error: 'Registration failed' });
  }
});

// POST /auth/login
router.post('/login', async (req, res) => {
  try {
    const { email, password } = req.body;

    // Find user
    const user = users.find(u => u.email === email);
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    // Verify password
    const isValidPassword = await bcrypt.compare(password, user.password);
    if (!isValidPassword) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    // Generate tokens
    const accessToken = generateToken({
      id: user.id,
      email: user.email,
      role: user.role,
    });

    const refreshToken = generateRefreshToken({
      id: user.id,
      email: user.email,
    });

    // Store refresh token
    refreshTokens.push(refreshToken);

    res.json({
      user: {
        id: user.id,
        name: user.name,
        email: user.email,
        role: user.role,
      },
      accessToken,
      refreshToken,
    });
  } catch (error) {
    res.status(500).json({ error: 'Login failed' });
  }
});

// POST /auth/refresh
router.post('/refresh', (req, res) => {
  try {
    const { refreshToken } = req.body;

    if (!refreshToken) {
      return res.status(401).json({ error: 'Refresh token required' });
    }

    if (!refreshTokens.includes(refreshToken)) {
      return res.status(403).json({ error: 'Invalid refresh token' });
    }

    const decoded = verifyRefreshToken(refreshToken);

    // Generate new access token
    const accessToken = generateToken({
      id: decoded.id,
      email: decoded.email,
      role: decoded.role,
    });

    res.json({ accessToken });
  } catch (error) {
    res.status(403).json({ error: 'Invalid refresh token' });
  }
});

// POST /auth/logout
router.post('/logout', (req, res) => {
  try {
    const { refreshToken } = req.body;

    // Remove refresh token
    const index = refreshTokens.indexOf(refreshToken);
    if (index > -1) {
      refreshTokens.splice(index, 1);
    }

    res.json({ message: 'Logged out successfully' });
  } catch (error) {
    res.status(500).json({ error: 'Logout failed' });
  }
});

module.exports = router;
```

### **Role-Based Authorization**
```javascript
// middleware/authorization.js
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        error: 'Insufficient permissions',
        required: roles,
        current: req.user.role,
      });
    }

    next();
  };
};

const isOwner = (resourceField = 'userId') => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    const resourceId = req.params.id || req.body[resourceField];

    if (req.user.id !== resourceId && req.user.role !== 'admin') {
      return res.status(403).json({ error: 'Access denied' });
    }

    next();
  };
};

module.exports = {
  authorize,
  isOwner,
};

// Usage in routes
const { authorize, isOwner } = require('../middleware/authorization');

// Admin only routes
router.get('/admin/users', authorize('admin'), getAllUsers);
router.delete('/admin/users/:id', authorize('admin'), deleteUser);

// Owner or admin routes
router.put('/users/:id', isOwner(), updateUser);
router.delete('/users/:id', isOwner(), deleteUser);

// Multiple roles
router.get('/moderator/content', authorize('moderator', 'admin'), getContent);
```

---

## 🗄️ **Database Integration**

### **MongoDB with Mongoose**
```javascript
// models/User.js
const mongoose = require('mongoose');
const bcrypt = require('bcrypt');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    maxlength: [50, 'Name cannot exceed 50 characters'],
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    validate: {
      validator: function(email) {
        return /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(email);
      },
      message: 'Please enter a valid email',
    },
  },
  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: [6, 'Password must be at least 6 characters'],
    select: false, // Don't include in queries by default
  },
  role: {
    type: String,
    enum: ['user', 'moderator', 'admin'],
    default: 'user',
  },
  avatar: {
    type: String,
    default: null,
  },
  isActive: {
    type: Boolean,
    default: true,
  },
  lastLogin: {
    type: Date,
    default: null,
  },
  emailVerified: {
    type: Boolean,
    default: false,
  },
  emailVerificationToken: String,
  passwordResetToken: String,
  passwordResetExpires: Date,
}, {
  timestamps: true,
});

// Index for better performance
userSchema.index({ email: 1 });
userSchema.index({ createdAt: -1 });

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();

  try {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error) {
    next(error);
  }
});

// Compare password method
userSchema.methods.comparePassword = async function(candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};

// Transform output
userSchema.methods.toJSON = function() {
  const userObject = this.toObject();
  delete userObject.password;
  delete userObject.emailVerificationToken;
  delete userObject.passwordResetToken;
  delete userObject.passwordResetExpires;
  return userObject;
};

// Static methods
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email: email.toLowerCase() });
};

userSchema.statics.findActiveUsers = function() {
  return this.find({ isActive: true });
};

module.exports = mongoose.model('User', userSchema);

// models/Post.js
const mongoose = require('mongoose');

const postSchema = new mongoose.Schema({
  title: {
    type: String,
    required: [true, 'Title is required'],
    trim: true,
    maxlength: [100, 'Title cannot exceed 100 characters'],
  },
  content: {
    type: String,
    required: [true, 'Content is required'],
  },
  excerpt: {
    type: String,
    maxlength: [200, 'Excerpt cannot exceed 200 characters'],
  },
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
  },
  category: {
    type: String,
    required: [true, 'Category is required'],
    enum: ['technology', 'lifestyle', 'business', 'health', 'education'],
  },
  tags: [{
    type: String,
    trim: true,
    lowercase: true,
  }],
  status: {
    type: String,
    enum: ['draft', 'published', 'archived'],
    default: 'draft',
  },
  featuredImage: {
    type: String,
    default: null,
  },
  views: {
    type: Number,
    default: 0,
  },
  likes: [{
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
    },
    createdAt: {
      type: Date,
      default: Date.now,
    },
  }],
  comments: [{
    content: {
      type: String,
      required: true,
    },
    author: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
    createdAt: {
      type: Date,
      default: Date.now,
    },
  }],
  publishedAt: {
    type: Date,
    default: null,
  },
}, {
  timestamps: true,
});

// Indexes
postSchema.index({ author: 1, createdAt: -1 });
postSchema.index({ category: 1, publishedAt: -1 });
postSchema.index({ status: 1, publishedAt: -1 });
postSchema.index({ tags: 1 });

// Virtual for like count
postSchema.virtual('likeCount').get(function() {
  return this.likes.length;
});

// Virtual for comment count
postSchema.virtual('commentCount').get(function() {
  return this.comments.length;
});

// Pre-save middleware
postSchema.pre('save', function(next) {
  if (this.isModified('content') && !this.excerpt) {
    this.excerpt = this.content.substring(0, 200) + '...';
  }

  if (this.status === 'published' && !this.publishedAt) {
    this.publishedAt = new Date();
  }

  next();
});

// Static methods
postSchema.statics.findPublished = function() {
  return this.find({ status: 'published' }).sort({ publishedAt: -1 });
};

postSchema.statics.findByCategory = function(category) {
  return this.find({
    status: 'published',
    category,
  }).sort({ publishedAt: -1 });
};

postSchema.statics.findByTag = function(tag) {
  return this.find({
    status: 'published',
    tags: tag,
  }).sort({ publishedAt: -1 });
};

module.exports = mongoose.model('Post', postSchema);
```

### **Database Connection**
```javascript
// config/database.js
const mongoose = require('mongoose');
const config = require('./index');

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(config.databaseUrl, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
      useCreateIndex: true,
      useFindAndModify: false,
    });

    console.log(`MongoDB Connected: ${conn.connection.host}`);

    // Handle connection events
    mongoose.connection.on('error', (err) => {
      console.error('MongoDB connection error:', err);
    });

    mongoose.connection.on('disconnected', () => {
      console.log('MongoDB disconnected');
    });

    // Graceful shutdown
    process.on('SIGINT', async () => {
      await mongoose.connection.close();
      console.log('MongoDB connection closed through app termination');
      process.exit(0);
    });

  } catch (error) {
    console.error('Database connection error:', error);
    process.exit(1);
  }
};

module.exports = connectDB;
```

---

## 🛡️ **Error Handling & Validation**

### **Global Error Handler**
```javascript
// middleware/errorHandler.js
const errorHandler = (err, req, res, next) => {
  let error = { ...err };
  error.message = err.message;

  // Log error
  console.error(err);

  // Mongoose bad ObjectId
  if (err.name === 'CastError') {
    const message = 'Resource not found';
    error = { message, statusCode: 404 };
  }

  // Mongoose duplicate key
  if (err.code === 11000) {
    const message = 'Duplicate field value entered';
    error = { message, statusCode: 400 };
  }

  // Mongoose validation error
  if (err.name === 'ValidationError') {
    const message = Object.values(err.errors).map(val => val.message).join(', ');
    error = { message, statusCode: 400 };
  }

  // JWT errors
  if (err.name === 'JsonWebTokenError') {
    const message = 'Invalid token';
    error = { message, statusCode: 401 };
  }

  if (err.name === 'TokenExpiredError') {
    const message = 'Token expired';
    error = { message, statusCode: 401 };
  }

  res.status(error.statusCode || 500).json({
    success: false,
    error: error.message || 'Server Error',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
  });
};

module.exports = errorHandler;

// middleware/asyncHandler.js
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

module.exports = asyncHandler;

// Usage in routes
const asyncHandler = require('../middleware/asyncHandler');

// Instead of:
// router.get('/users', async (req, res) => {
//   try {
//     const users = await User.find();
//     res.json({ users });
//   } catch (error) {
//     res.status(500).json({ error: 'Failed to fetch users' });
//   }
// });

// Use:
router.get('/users', asyncHandler(async (req, res) => {
  const users = await User.find();
  res.json({ users });
}));
```

### **Input Validation**
```javascript
// validation/userValidation.js
const { body, param, query } = require('express-validator');

const validateCreateUser = [
  body('name')
    .trim()
    .isLength({ min: 2, max: 50 })
    .withMessage('Name must be between 2 and 50 characters')
    .matches(/^[a-zA-Z\s]+$/)
    .withMessage('Name can only contain letters and spaces'),

  body('email')
    .isEmail()
    .normalizeEmail()
    .withMessage('Please provide a valid email'),

  body('password')
    .isLength({ min: 6 })
    .withMessage('Password must be at least 6 characters long')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
    .withMessage('Password must contain at least one lowercase letter, one uppercase letter, and one number'),

  body('role')
    .optional()
    .isIn(['user', 'moderator', 'admin'])
    .withMessage('Invalid role'),
];

const validateUpdateUser = [
  param('id')
    .isMongoId()
    .withMessage('Invalid user ID'),

  body('name')
    .optional()
    .trim()
    .isLength({ min: 2, max: 50 })
    .withMessage('Name must be between 2 and 50 characters'),

  body('email')
    .optional()
    .isEmail()
    .normalizeEmail()
    .withMessage('Please provide a valid email'),
];

const validateUserQuery = [
  query('page')
    .optional()
    .isInt({ min: 1 })
    .withMessage('Page must be a positive integer'),

  query('limit')
    .optional()
    .isInt({ min: 1, max: 100 })
    .withMessage('Limit must be between 1 and 100'),

  query('search')
    .optional()
    .trim()
    .isLength({ min: 1 })
    .withMessage('Search query cannot be empty'),
];

module.exports = {
  validateCreateUser,
  validateUpdateUser,
  validateUserQuery,
};

// middleware/validationResult.js
const { validationResult } = require('express-validator');

const handleValidationErrors = (req, res, next) => {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      error: 'Validation failed',
      details: errors.array(),
    });
  }

  next();
};

module.exports = handleValidationErrors;

// Usage in routes
const {
  validateCreateUser,
  validateUpdateUser,
  validateUserQuery,
} = require('../validation/userValidation');
const handleValidationErrors = require('../middleware/validationResult');

router.post('/users', validateCreateUser, handleValidationErrors, createUser);
router.put('/users/:id', validateUpdateUser, handleValidationErrors, updateUser);
router.get('/users', validateUserQuery, handleValidationErrors, getUsers);
```

---

## 🔒 **Security Best Practices**

### **Security Middleware**
```javascript
// middleware/security.js
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');

// Security headers
const securityHeaders = helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", 'data:', 'https:'],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
});

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: {
    error: 'Too many requests from this IP, please try again later.',
  },
  standardHeaders: true,
  legacyHeaders: false,
});

// API rate limiting
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 500, // limit each IP to 500 API requests per windowMs
  message: {
    error: 'Too many API requests, please try again later.',
  },
});

// Auth rate limiting
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // limit each IP to 5 auth attempts per windowMs
  message: {
    error: 'Too many authentication attempts, please try again later.',
  },
  skipSuccessfulRequests: true,
});

// Data sanitization
const dataSanitization = [
  mongoSanitize(), // Prevent NoSQL injection
  xss(), // Prevent XSS attacks
  hpp(), // Prevent HTTP Parameter Pollution
];

module.exports = {
  securityHeaders,
  limiter,
  apiLimiter,
  authLimiter,
  dataSanitization,
};
```

### **Password Security**
```javascript
// utils/password.js
const bcrypt = require('bcrypt');
const crypto = require('crypto');

const hashPassword = async (password) => {
  const saltRounds = 12;
  return bcrypt.hash(password, saltRounds);
};

const comparePassword = async (password, hashedPassword) => {
  return bcrypt.compare(password, hashedPassword);
};

const generatePasswordResetToken = () => {
  return crypto.randomBytes(32).toString('hex');
};

const generateSecureToken = (length = 32) => {
  return crypto.randomBytes(length).toString('hex');
};

module.exports = {
  hashPassword,
  comparePassword,
  generatePasswordResetToken,
  generateSecureToken,
};
```

### **CORS Configuration**
```javascript
// config/cors.js
const corsOptions = {
  origin: function (origin, callback) {
    // Allow requests with no origin (mobile apps, etc.)
    if (!origin) return callback(null, true);

    const allowedOrigins = [
      'http://localhost:3000',
      'http://localhost:3001',
      'https://yourapp.com',
      'https://www.yourapp.com',
    ];

    if (allowedOrigins.indexOf(origin) !== -1) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
  optionsSuccessStatus: 200,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
  allowedHeaders: [
    'Origin',
    'X-Requested-With',
    'Content-Type',
    'Accept',
    'Authorization',
    'Cache-Control',
  ],
};

module.exports = corsOptions;
```

---

## 🧪 **API Testing**

### **Unit Tests for API**
```javascript
// __tests__/routes/users.test.js
const request = require('supertest');
const express = require('express');
const userRoutes = require('../../routes/users');

// Mock database
jest.mock('../../models/User');

const User = require('../../models/User');

const app = express();
app.use(express.json());
app.use('/api/users', userRoutes);

describe('User Routes', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('GET /api/users', () => {
    test('should return all users', async () => {
      const mockUsers = [
        { id: 1, name: 'John Doe', email: 'john@example.com' },
        { id: 2, name: 'Jane Smith', email: 'jane@example.com' },
      ];

      User.find.mockResolvedValue(mockUsers);

      const response = await request(app).get('/api/users');

      expect(response.status).toBe(200);
      expect(response.body).toEqual({ users: mockUsers });
      expect(User.find).toHaveBeenCalledTimes(1);
    });

    test('should handle database errors', async () => {
      User.find.mockRejectedValue(new Error('Database error'));

      const response = await request(app).get('/api/users');

      expect(response.status).toBe(500);
      expect(response.body).toHaveProperty('error');
    });
  });

  describe('POST /api/users', () => {
    test('should create a new user', async () => {
      const newUser = {
        name: 'John Doe',
        email: 'john@example.com',
        password: 'password123',
      };

      const savedUser = {
        id: 1,
        ...newUser,
        createdAt: new Date(),
      };

      User.findOne.mockResolvedValue(null);
      User.prototype.save.mockResolvedValue(savedUser);

      const response = await request(app)
        .post('/api/users')
        .send(newUser);

      expect(response.status).toBe(201);
      expect(response.body).toHaveProperty('user');
      expect(response.body.user.name).toBe(newUser.name);
    });

    test('should validate required fields', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({});

      expect(response.status).toBe(400);
      expect(response.body).toHaveProperty('error');
    });

    test('should prevent duplicate emails', async () => {
      const existingUser = {
        id: 1,
        name: 'Existing User',
        email: 'john@example.com',
      };

      User.findOne.mockResolvedValue(existingUser);

      const response = await request(app)
        .post('/api/users')
        .send({
          name: 'John Doe',
          email: 'john@example.com',
          password: 'password123',
        });

      expect(response.status).toBe(409);
      expect(response.body.error).toBe('Email already exists');
    });
  });
});
```

