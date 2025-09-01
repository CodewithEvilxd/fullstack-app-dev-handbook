# Lesson 22: Database Design & MongoDB

## 🎯 **Learning Objectives**
- Understand database design principles and best practices
- Master MongoDB fundamentals and operations
- Design efficient database schemas for React Native apps
- Implement data relationships and indexing strategies
- Optimize database queries and performance

## 📚 **Table of Contents**
1. [Introduction to Database Design](#introduction-to-database-design)
2. [MongoDB Fundamentals](#mongodb-fundamentals)
3. [Schema Design Principles](#schema-design-principles)
4. [Data Relationships](#data-relationships)
5. [Indexing Strategies](#indexing-strategies)
6. [Query Optimization](#query-optimization)
7. [Data Validation](#data-validation)
8. [Backup and Recovery](#backup-and-recovery)
9. [Performance Monitoring](#performance-monitoring)
10. [Practical Examples](#practical-examples)

---

## 🗄️ **Introduction to Database Design**

Database design is crucial for building scalable and maintainable applications. Good design ensures data integrity, performance, and flexibility.

### **Database Design Principles**
- **Normalization**: Reduce data redundancy and improve data integrity
- **Denormalization**: Optimize read performance by duplicating data
- **Scalability**: Design for growth and high traffic
- **Maintainability**: Easy to modify and extend
- **Performance**: Optimize for common query patterns

### **MongoDB vs Traditional RDBMS**
```
Traditional RDBMS          MongoDB
=================          =======
Tables                     Collections
Rows                       Documents
Columns                    Fields
Joins                      Embedded Documents / References
Schemas                    Flexible Schemas
ACID Transactions          Multi-Document Transactions
```

---

## 🍃 **MongoDB Fundamentals**

### **MongoDB Installation & Setup**
```bash
# Install MongoDB (macOS with Homebrew)
brew install mongodb-community
brew services start mongodb-community

# Install MongoDB (Ubuntu)
sudo apt-get install mongodb

# Install MongoDB (Windows)
# Download from mongodb.com and run installer

# Start MongoDB service
sudo systemctl start mongodb
sudo systemctl enable mongodb

# Connect to MongoDB shell
mongo
```

### **Basic MongoDB Operations**
```javascript
// Connect to MongoDB
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/myapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  useCreateIndex: true,
  useFindAndModify: false,
});

// Handle connection events
mongoose.connection.on('connected', () => {
  console.log('MongoDB connected');
});

mongoose.connection.on('error', (err) => {
  console.error('MongoDB connection error:', err);
});

mongoose.connection.on('disconnected', () => {
  console.log('MongoDB disconnected');
});

// Close connection on app termination
process.on('SIGINT', async () => {
  await mongoose.connection.close();
  process.exit(0);
});
```

### **CRUD Operations**
```javascript
// Create a document
const user = new User({
  name: 'John Doe',
  email: 'john@example.com',
  age: 30,
});

await user.save();

// Read documents
// Find all users
const users = await User.find();

// Find user by ID
const user = await User.findById('60d5ecb54bbb4c001f8b4567');

// Find users with conditions
const adults = await User.find({ age: { $gte: 18 } });

// Update documents
// Update one document
await User.updateOne(
  { _id: '60d5ecb54bbb4c001f8b4567' },
  { $set: { name: 'Jane Doe' } }
);

// Update multiple documents
await User.updateMany(
  { age: { $lt: 18 } },
  { $set: { isMinor: true } }
);

// Delete documents
// Delete one document
await User.deleteOne({ _id: '60d5ecb54bbb4c001f8b4567' });

// Delete multiple documents
await User.deleteMany({ isActive: false });
```

---

## 📋 **Schema Design Principles**

### **Schema Design Best Practices**
```javascript
// User Schema Example
const userSchema = new mongoose.Schema({
  // Basic Information
  firstName: {
    type: String,
    required: [true, 'First name is required'],
    trim: true,
    maxlength: [50, 'First name cannot exceed 50 characters'],
  },

  lastName: {
    type: String,
    required: [true, 'Last name is required'],
    trim: true,
    maxlength: [50, 'Last name cannot exceed 50 characters'],
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

  // Profile Information
  avatar: {
    type: String,
    default: null,
  },

  bio: {
    type: String,
    maxlength: [500, 'Bio cannot exceed 500 characters'],
  },

  // Account Status
  isActive: {
    type: Boolean,
    default: true,
  },

  isVerified: {
    type: 'boolean',
    description: 'Whether the user email is verified',
    default: false,
  },
  createdAt: {
    type: 'string',
    format: 'date-time',
    description: 'Account creation timestamp',
  },
  updatedAt: {
    type: 'string',
    format: 'date-time',
    description: 'Last update timestamp',
  },
},
},
Error: {
type: 'object',
properties: {
  success: {
    type: 'boolean',
    example: false,
  },
  error: {
    type: 'string',
    description: 'Error message',
  },
  details: {
    type: 'array',
    items: {
      type: 'object',
      properties: {
        field: { type: 'string' },
        message: { type: 'string' },
      },
    },
  },
},
},
},
},
paths: {
'/api/users': {
get: {
summary: 'Get all users',
tags: ['Users'],
security: [{ bearerAuth: [] }],
parameters: [
  {
    name: 'page',
    in: 'query',
    schema: { type: 'integer', minimum: 1 },
    description: 'Page number',
  },
  {
    name: 'limit',
    in: 'query',
    schema: { type: 'integer', minimum: 1, maximum: 100 },
    description: 'Items per page',
  },
  {
    name: 'search',
    in: 'query',
    schema: { type: 'string' },
    description: 'Search query',
  },
  {
    name: 'role',
    in: 'query',
    schema: { type: 'string', enum: ['user', 'moderator', 'admin'] },
    description: 'Filter by role',
  },
],
responses: {
  200: {
    description: 'Users retrieved successfully',
    content: {
      'application/json': {
        schema: {
          type: 'object',
          properties: {
            success: { type: 'boolean', example: true },
            data: {
              type: 'array',
              items: { $ref: '#/components/schemas/User' },
            },
            pagination: {
              type: 'object',
              properties: {
                page: { type: 'integer' },
                limit: { type: 'integer' },
                total: { type: 'integer' },
                pages: { type: 'integer' },
              },
            },
          },
        },
      },
    },
  },
  401: {
    description: 'Unauthorized',
    content: {
      'application/json': {
        schema: { $ref: '#/components/schemas/Error' },
      },
    },
  },
},
},
post: {
summary: 'Create a new user',
tags: ['Users'],
requestBody: {
  required: true,
  content: {
    'application/json': {
      schema: {
        type: 'object',
        required: ['firstName', 'lastName', 'email', 'password'],
        properties: {
          firstName: { type: 'string', example: 'John' },
          lastName: { type: 'string', example: 'Doe' },
          email: { type: 'string', format: 'email', example: 'john@example.com' },
          password: { type: 'string', example: 'password123' },
          role: { type: 'string', enum: ['user', 'moderator', 'admin'], example: 'user' },
        },
      },
    },
  },
},
responses: {
  201: {
    description: 'User created successfully',
    content: {
      'application/json': {
        schema: {
          type: 'object',
          properties: {
            success: { type: 'boolean', example: true },
            data: { $ref: '#/components/schemas/User' },
            message: { type: 'string', example: 'User created successfully' },
          },
        },
      },
    },
  },
  400: {
    description: 'Validation error',
    content: {
      'application/json': {
        schema: { $ref: '#/components/schemas/Error' },
      },
    },
  },
  409: {
    description: 'User already exists',
    content: {
      'application/json': {
        schema: { $ref: '#/components/schemas/Error' },
      },
    },
  },
},
},
},
'/api/users/{id}': {
get: {
summary: 'Get user by ID',
tags: ['Users'],
parameters: [
  {
    name: 'id',
    in: 'path',
    required: true,
    schema: { type: 'string' },
    description: 'User ID',
  },
],
responses: {
  200: {
    description: 'User retrieved successfully',
    content: {
      'application/json': {
        schema: {
          type: 'object',
          properties: {
            success: { type: 'boolean', example: true },
            data: { $ref: '#/components/schemas/User' },
          },
        },
      },
    },
  },
  404: {
    description: 'User not found',
    content: {
      'application/json': {
        schema: { $ref: '#/components/schemas/Error' },
      },
    },
  },
},
},
put: {
summary: 'Update user',
tags: ['Users'],
security: [{ bearerAuth: [] }],
parameters: [
  {
    name: 'id',
    in: 'path',
    required: true,
    schema: { type: 'string' },
    description: 'User ID',
  },
],
requestBody: {
  required: true,
  content: {
    'application/json': {
      schema: {
        type: 'object',
        properties: {
          firstName: { type: 'string', example: 'John' },
          lastName: { type: 'string', example: 'Doe' },
          email: { type: 'string', format: 'email', example: 'john@example.com' },
          role: { type: 'string', enum: ['user', 'moderator', 'admin'], example: 'user' },
        },
      },
    },
  },
},
responses: {
  200: {
    description: 'User updated successfully',
    content: {
      'application/json': {
        schema: {
          type: 'object',
          properties: {
            success: { type: 'boolean', example: true },
            data: { $ref: '#/components/schemas/User' },
            message: { type: 'string', example: 'User updated successfully' },
          },
        },
      },
    },
  },
  403: {
    description: 'Forbidden - Not authorized',
    content: {
      'application/json': {
        schema: { $ref: '#/components/schemas/Error' },
      },
    },
  },
  404: {
    description: 'User not found',
    content: {
      'application/json': {
        schema: { $ref: '#/components/schemas/Error' },
      },
    },
  },
},
},
delete: {
summary: 'Delete user',
tags: ['Users'],
security: [{ bearerAuth: [] }],
parameters: [
  {
    name: 'id',
    in: 'path',
    required: true,
    schema: { type: 'string' },
    description: 'User ID',
  },
],
responses: {
  200: {
    description: 'User deleted successfully',
    content: {
      'application/json': {
        schema: {
          type: 'object',
          properties: {
            success: { type: 'boolean', example: true },
            message: { type: 'string', example: 'User deleted successfully' },
          },
        },
      },
    },
  },
  403: {
    description: 'Forbidden - Not authorized',
    content: {
      'application/json': {
        schema: { $ref: '#/components/schemas/Error' },
      },
    },
  },
  404: {
    description: 'User not found',
    content: {
      'application/json': {
        schema: { $ref: '#/components/schemas/Error' },
      },
    },
  },
},
},
},
},
};

const specs = swaggerJsdoc(options);
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs));

// Export specs for testing
module.exports = specs;
```

### **GraphQL Documentation**
```javascript
// GraphQL schema documentation
const { printSchema } = require('graphql');

// Generate schema documentation
const schemaDocumentation = printSchema(schema);

// Save to file
const fs = require('fs');
fs.writeFileSync('./docs/schema.graphql', schemaDocumentation);

// Generate HTML documentation
const { renderPlaygroundPage } = require('graphql-playground-html');

app.get('/graphql-docs', (req, res) => {
const playground = renderPlaygroundPage({
endpoint: '/graphql',
title: 'Blog API GraphQL',
description: 'GraphQL API for blog management',
});
res.set('Content-Type', 'text/html');
res.send(playground);
});
```

---

## 🔒 **API Security**

### **Authentication Middleware**
```javascript
const jwt = require('jsonwebtoken');
const config = require('../config');

// JWT authentication middleware
const authenticateToken = (req, res, next) => {
const authHeader = req.headers['authorization'];
const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN

if (!token) {
return res.status(401).json({
success: false,
error: 'Access token required',
});
}

jwt.verify(token, config.jwtSecret, (err, user) => {
if (err) {
return res.status(403).json({
success: false,
error: 'Invalid or expired token',
});
}

req.user = user;
next();
});
};

// Role-based authorization
const requireRole = (roles) => {
return (req, res, next) => {
if (!req.user) {
return res.status(401).json({
success: false,
error: 'Authentication required',
});
}

if (!roles.includes(req.user.role)) {
return res.status(403).json({
success: false,
error: 'Insufficient permissions',
});
}

next();
};
};

// Resource ownership check
const requireOwnership = (resourceType) => {
return async (req, res, next) => {
try {
const resourceId = req.params.id;
let resource;

switch (resourceType) {
case 'post':
  resource = await Post.findById(resourceId);
  break;
case 'comment':
  // Find post containing the comment
  resource = await Post.findOne({ 'comments._id': resourceId });
  break;
default:
  return res.status(400).json({
    success: false,
    error: 'Invalid resource type',
  });
}

if (!resource) {
return res.status(404).json({
  success: false,
  error: 'Resource not found',
});
}

// Check ownership
if (resource.author.toString() !== req.user.id && req.user.role !== 'admin') {
return res.status(403).json({
  success: false,
  error: 'Not authorized to access this resource',
});
}

req.resource = resource;
next();
} catch (error) {
res.status(500).json({
success: false,
error: 'Authorization check failed',
});
}
};
};

module.exports = {
authenticateToken,
requireRole,
requireOwnership,
};
```

### **Security Headers**
```javascript
const helmet = require('helmet');
const cors = require('cors');

// Security middleware
const securityMiddleware = [
// Basic security headers
helmet(),

// CORS configuration
cors({
origin: process.env.NODE_ENV === 'production'
? ['https://yourapp.com', 'https://www.yourapp.com']
: ['http://localhost:3000', 'http://localhost:3001'],
credentials: true,
methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
allowedHeaders: [
'Origin',
'X-Requested-With',
'Content-Type',
'Accept',
'Authorization',
'Cache-Control',
],
}),

// Rate limiting
require('./middleware/rateLimit'),

// Data sanitization
require('./middleware/dataSanitization'),
];

module.exports = securityMiddleware;
```

---

## 🧪 **Practical Examples**

### **Complete Blog API**
```javascript
// server.js
const express = require('express');
const mongoose = require('mongoose');
const config = require('./config');
const securityMiddleware = require('./middleware/security');
const errorHandler = require('./middleware/errorHandler');

const app = express();

// Connect to MongoDB
mongoose.connect(config.databaseUrl, {
useNewUrlParser: true,
useUnifiedTopology: true,
useCreateIndex: true,
useFindAndModify: false,
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('MongoDB connection error:', err));

// Apply security middleware
app.use(securityMiddleware);

// Body parsing
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Routes
app.use('/api/v1/auth', require('./routes/auth'));
app.use('/api/v1/users', require('./routes/users'));
app.use('/api/v1/posts', require('./routes/posts'));
app.use('/api/v1/categories', require('./routes/categories'));
app.use('/api/v1/tags', require('./routes/tags'));

// Health check
app.get('/api/health', (req, res) => {
res.json({
status: 'OK',
timestamp: new Date().toISOString(),
uptime: process.uptime(),
database: mongoose.connection.readyState === 1 ? 'connected' : 'disconnected',
});
});

// API documentation
if (process.env.NODE_ENV !== 'production') {
const swaggerSetup = require('./docs/swagger');
swaggerSetup(app);
console.log('📚 API Documentation: http://localhost:3000/api-docs');
}

// Error handling
app.use(errorHandler);

// 404 handler
app.use((req, res) => {
res.status(404).json({
success: false,
error: 'Route not found',
message: `${req.method} ${req.path} not found`,
});
});

const PORT = config.port || 3000;
app.listen(PORT, () => {
console.log(`🚀 Server running on port ${PORT}`);
console.log(`📊 Environment: ${config.nodeEnv}`);
});

module.exports = app;
```

### **User Management System**
```javascript
// routes/users.js
const express = require('express');
const router = express.Router();
const User = require('../models/User');
const {
authenticateToken,
requireRole,
requireOwnership,
} = require('../middleware/auth');
const { validateCreateUser, validateUpdateUser } = require('../validation/userValidation');

// GET /api/users - Get all users (Admin only)
router.get('/',
authenticateToken,
requireRole(['admin', 'moderator']),
async (req, res) => {
try {
const { page = 1, limit = 10, search, role, isActive } = req.query;

// Build query
const query = {};
if (search) {
query.$or = [
  { firstName: new RegExp(search, 'i') },
  { lastName: new RegExp(search, 'i') },
  { email: new RegExp(search, 'i') },
];
}
if (role) query.role = role;
if (isActive !== undefined) query.isActive = isActive === 'true';

const users = await User.find(query)
.select('-password')
.sort({ createdAt: -1 })
.limit(limit * 1)
.skip((page - 1) * limit);

const total = await User.countDocuments(query);

res.json({
success: true,
data: users,
pagination: {
  page: parseInt(page),
  limit: parseInt(limit),
  total,
  pages: Math.ceil(total / limit),
},
});
} catch (error) {
res.status(500).json({
success: false,
error: 'Failed to fetch users',
});
}
}
);

// GET /api/users/profile - Get current user profile
router.get('/profile', authenticateToken, async (req, res) => {
try {
const user = await User.findById(req.user.id).select('-password');
if (!user) {
return res.status(404).json({
success: false,
error: 'User not found',
});
}

res.json({
success: true,
data: user,
});
} catch (error) {
res.status(500).json({
success: false,
error: 'Failed to fetch user profile',
});
}
});

// POST /api/users - Create new user
router.post('/', validateCreateUser, async (req, res) => {
try {
const { firstName, lastName, email, password, role } = req.body;

// Check if user exists
const existingUser = await User.findOne({ email: email.toLowerCase() });
if (existingUser) {
return res.status(409).json({
success: false,
error: 'User with this email already exists',
});
}

const user = await User.create({
firstName,
lastName,
email: email.toLowerCase(),
password,
role: role || 'user',
});

const userResponse = user.toObject();
delete userResponse.password;

res.status(201).json({
success: true,
data: userResponse,
message: 'User created successfully',
});
} catch (error) {
if (error.name === 'ValidationError') {
const errors = Object.values(error.errors).map(err => err.message);
return res.status(400).json({
success: false,
error: 'Validation failed',
details: errors,
});
}

res.status(500).json({
success: false,
error: 'Failed to create user',
});
}
});

// PUT /api/users/:id - Update user
router.put('/:id',
authenticateToken,
validateUpdateUser,
async (req, res) => {
try {
const user = await User.findById(req.params.id);
if (!user) {
return res.status(404).json({
  success: false,
  error: 'User not found',
});
}

// Check permissions
if (user._id.toString() !== req.user.id && req.user.role !== 'admin') {
return res.status(403).json({
  success: false,
  error: 'Not authorized to update this user',
});
}

const { firstName, lastName, email, role, isActive } = req.body;

// Update fields
if (firstName) user.firstName = firstName;
if (lastName) user.lastName = lastName;
if (email) user.email = email.toLowerCase();
if (role && req.user.role === 'admin') user.role = role;
if (isActive !== undefined && req.user.role === 'admin') user.isActive = isActive;

await user.save();

const userResponse = user.toObject();
delete userResponse.password;

res.json({
success: true,
data: userResponse,
message: 'User updated successfully',
});
} catch (error) {
if (error.name === 'ValidationError') {
const errors = Object.values(error.errors).map(err => err.message);
return res.status(400).json({
  success: false,
  error: 'Validation failed',
  details: errors,
});
}

res.status(500).json({
success: false,
error: 'Failed to update user',
});
}
}
);

// DELETE /api/users/:id - Delete user (Admin only)
router.delete('/:id',
authenticateToken,
requireRole(['admin']),
async (req, res) => {
try {
const user = await User.findById(req.params.id);
if (!user) {
return res.status(404).json({
  success: false,
  error: 'User not found',
});
}

await User.findByIdAndDelete(req.params.id);

res.json({
success: true,
message: 'User deleted successfully',
});
} catch (error) {
res.status(500).json({
success: false,
error: 'Failed to delete user',
});
}
}
);

module.exports = router;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Database Design Principles**: Normalization, denormalization, scalability
- ✅ **MongoDB Fundamentals**: CRUD operations, connection management
- ✅ **Schema Design**: Mongoose schemas, validation, indexes
- ✅ **Data Relationships**: One-to-one, one-to-many, many-to-many
- ✅ **Indexing Strategies**: Single, compound, text, geospatial indexes
- ✅ **Query Optimization**: Efficient queries, aggregation pipelines
- ✅ **Data Validation**: Schema validation, custom validators
- ✅ **Backup & Recovery**: Database backup strategies
- ✅ **Performance Monitoring**: Index usage, slow query analysis

### **Best Practices**
1. **Design schemas based on query patterns** - Optimize for read operations
2. **Use appropriate indexes** - Balance between query performance and write performance
3. **Implement proper validation** - Both at schema and application level
4. **Handle relationships efficiently** - Use embedding vs referencing appropriately
5. **Monitor performance** - Use MongoDB profiling and explain plans
6. **Plan for scalability** - Design for horizontal scaling when needed
7. **Backup regularly** - Implement automated backup strategies
8. **Use connection pooling** - Efficient database connection management

### **Next Steps**
- Implement advanced aggregation pipelines
- Set up database replication and sharding
- Implement database migrations
- Create database performance dashboards
- Learn about NoSQL database alternatives

---

## 🎯 **Assignment**

### **Task 1: E-commerce Database Design**
Design and implement a complete e-commerce database with:
- User management with roles and permissions
- Product catalog with categories and variants
- Shopping cart and order management
- Payment and shipping integration
- Review and rating system
- Inventory management
- Analytics and reporting

### **Task 2: Social Media Platform**
Create a social media database schema including:
- User profiles with followers/following
- Posts with likes, comments, and shares
- Media attachments and hashtags
- Direct messaging system
- Notification system
- Content moderation features
- Analytics and insights

### **Task 3: Learning Management System**
Design a database for an online learning platform with:
- Course and lesson management
- Student enrollment and progress tracking
- Quiz and assessment system
- Certificate generation
- Discussion forums
- Video streaming integration
- Progress analytics

---

## 📚 **Additional Resources**
- [MongoDB Documentation](https://docs.mongodb.com/)
- [Mongoose Documentation](https://mongoosejs.com/docs/)
- [Database Design Best Practices](https://www.mongodb.com/blog/post/database-design-best-practices)
- [MongoDB University](https://university.mongodb.com/)
- [Database Indexing Guide](https://docs.mongodb.com/manual/indexes/)

---

**Next Lesson**: [Lesson 23: RESTful APIs & GraphQL](Lesson%2017_%20Push%20Notifications%20&%20Local%20Notifications.md)
    type: Boolean,
    default: false,
  },

  role: {
    type: String,
    enum: ['user', 'moderator', 'admin'],
    default: 'user',
  },

  // Timestamps
  lastLogin: {
    type: Date,
    default: null,
  },

  emailVerifiedAt: {
    type: Date,
    default: null,
  },
}, {
  timestamps: true, // Adds createdAt and updatedAt
  toJSON: { virtuals: true },
  toObject: { virtuals: true },
});

// Virtual for full name
userSchema.virtual('fullName').get(function() {
  return `${this.firstName} ${this.lastName}`;
});

// Index for better performance
userSchema.index({ email: 1 });
userSchema.index({ createdAt: -1 });
userSchema.index({ isActive: 1, role: 1 });

// Pre-save middleware
userSchema.pre('save', async function(next) {
  // Hash password if modified
  if (this.isModified('password')) {
    const bcrypt = require('bcrypt');
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
  }

  // Set email verification timestamp
  if (this.isModified('isVerified') && this.isVerified && !this.emailVerifiedAt) {
    this.emailVerifiedAt = new Date();
  }

  next();
});

// Instance methods
userSchema.methods.comparePassword = async function(candidatePassword) {
  const bcrypt = require('bcrypt');
  return bcrypt.compare(candidatePassword, this.password);
};

userSchema.methods.getPublicProfile = function() {
  return {
    id: this._id,
    firstName: this.firstName,
    lastName: this.lastName,
    fullName: this.fullName,
    email: this.email,
    avatar: this.avatar,
    bio: this.bio,
    isVerified: this.isVerified,
    role: this.role,
    createdAt: this.createdAt,
  };
};

// Static methods
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email: email.toLowerCase() });
};

userSchema.statics.findActiveUsers = function() {
  return this.find({ isActive: true });
};

userSchema.statics.getUserStats = async function() {
  const stats = await this.aggregate([
    {
      $group: {
        _id: null,
        totalUsers: { $sum: 1 },
        activeUsers: {
          $sum: { $cond: ['$isActive', 1, 0] }
        },
        verifiedUsers: {
          $sum: { $cond: ['$isVerified', 1, 0] }
        },
      },
    },
  ]);

  return stats[0] || {
    totalUsers: 0,
    activeUsers: 0,
    verifiedUsers: 0,
  };
};

module.exports = mongoose.model('User', userSchema);
```

### **Post Schema with Relationships**
```javascript
const postSchema = new mongoose.Schema({
  title: {
    type: String,
    required: [true, 'Title is required'],
    trim: true,
    maxlength: [200, 'Title cannot exceed 200 characters'],
  },

  content: {
    type: String,
    required: [true, 'Content is required'],
  },

  excerpt: {
    type: String,
    maxlength: [300, 'Excerpt cannot exceed 300 characters'],
  },

  featuredImage: {
    type: String,
    default: null,
  },

  // Author reference
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
  },

  // Category reference
  category: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Category',
    required: true,
  },

  // Tags (many-to-many relationship)
  tags: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Tag',
  }],

  // Publication status
  status: {
    type: String,
    enum: ['draft', 'published', 'archived'],
    default: 'draft',
  },

  // SEO fields
  slug: {
    type: String,
    unique: true,
    lowercase: true,
  },

  metaTitle: {
    type: String,
    maxlength: [60, 'Meta title cannot exceed 60 characters'],
  },

  metaDescription: {
    type: String,
    maxlength: [160, 'Meta description cannot exceed 160 characters'],
  },

  // Engagement metrics
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

  // Comments (embedded documents)
  comments: [{
    content: {
      type: String,
      required: true,
      maxlength: [1000, 'Comment cannot exceed 1000 characters'],
    },
    author: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: true,
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
    replies: [{
      content: {
        type: String,
        required: true,
        maxlength: [500, 'Reply cannot exceed 500 characters'],
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
    createdAt: {
      type: Date,
      default: Date.now,
    },
    updatedAt: {
      type: Date,
      default: Date.now,
    },
  }],

  // Publication dates
  publishedAt: {
    type: Date,
    default: null,
  },

  // Scheduled publication
  scheduledAt: {
    type: Date,
    default: null,
  },
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true },
});

// Indexes for performance
postSchema.index({ author: 1, createdAt: -1 });
postSchema.index({ category: 1, publishedAt: -1 });
postSchema.index({ status: 1, publishedAt: -1 });
postSchema.index({ tags: 1 });
postSchema.index({ slug: 1 });
postSchema.index({ views: -1 });
postSchema.index({ publishedAt: -1 });

// Virtuals
postSchema.virtual('likeCount').get(function() {
  return this.likes.length;
});

postSchema.virtual('commentCount').get(function() {
  return this.comments.length;
});

postSchema.virtual('readingTime').get(function() {
  const wordsPerMinute = 200;
  const words = this.content.split(' ').length;
  return Math.ceil(words / wordsPerMinute);
});

// Pre-save middleware
postSchema.pre('save', function(next) {
  // Generate slug from title
  if (this.isModified('title') && !this.slug) {
    this.slug = this.title
      .toLowerCase()
      .replace(/[^a-zA-Z0-9 ]/g, '')
      .replace(/\s+/g, '-')
      .replace(/-+/g, '-')
      .trim('-');
  }

  // Generate excerpt
  if (this.isModified('content') && !this.excerpt) {
    this.excerpt = this.content.substring(0, 300) + '...';
  }

  // Set published date
  if (this.status === 'published' && !this.publishedAt) {
    this.publishedAt = new Date();
  }

  next();
});

// Static methods
postSchema.statics.findPublished = function() {
  return this.find({ status: 'published' })
    .populate('author', 'firstName lastName avatar')
    .populate('category', 'name slug')
    .sort({ publishedAt: -1 });
};

postSchema.statics.findByCategory = function(categorySlug) {
  return this.find({ status: 'published' })
    .populate({
      path: 'category',
      match: { slug: categorySlug },
    })
    .populate('author', 'firstName lastName avatar')
    .sort({ publishedAt: -1 });
};

postSchema.statics.getPopularPosts = function(limit = 10) {
  return this.find({ status: 'published' })
    .populate('author', 'firstName lastName avatar')
    .sort({ views: -1 })
    .limit(limit);
};

postSchema.statics.searchPosts = function(query, options = {}) {
  const { page = 1, limit = 10 } = options;

  return this.find(
    {
      status: 'published',
      $text: { $search: query },
    },
    {
      score: { $meta: 'textScore' },
    }
  )
    .populate('author', 'firstName lastName avatar')
    .sort({ score: { $meta: 'textScore' } })
    .skip((page - 1) * limit)
    .limit(limit);
};

// Instance methods
postSchema.methods.incrementViews = function() {
  this.views += 1;
  return this.save();
};

postSchema.methods.addLike = function(userId) {
  if (!this.likes.some(like => like.user.toString() === userId.toString())) {
    this.likes.push({ user: userId });
    return this.save();
  }
  return this;
};

postSchema.methods.removeLike = function(userId) {
  this.likes = this.likes.filter(
    like => like.user.toString() !== userId.toString()
  );
  return this.save();
};

postSchema.methods.addComment = function(userId, content) {
  this.comments.push({
    content,
    author: userId,
  });
  return this.save();
};

module.exports = mongoose.model('Post', postSchema);
```

---

## 🔗 **Data Relationships**

### **One-to-One Relationships**
```javascript
// User Profile (One-to-One with User)
const userProfileSchema = new mongoose.Schema({
  user: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
    unique: true,
  },

  // Personal Information
  dateOfBirth: Date,
  gender: {
    type: String,
    enum: ['male', 'female', 'other'],
  },

  // Contact Information
  phone: {
    type: String,
    validate: {
      validator: function(phone) {
        return /^\+?[\d\s\-\(\)]+$/.test(phone);
      },
      message: 'Please enter a valid phone number',
    },
  },

  // Address
  address: {
    street: String,
    city: String,
    state: String,
    zipCode: String,
    country: String,
  },

  // Social Links
  socialLinks: {
    website: String,
    linkedin: String,
    twitter: String,
    github: String,
  },

  // Preferences
  preferences: {
    theme: {
      type: String,
      enum: ['light', 'dark', 'auto'],
      default: 'auto',
    },
    language: {
      type: String,
      default: 'en',
    },
    notifications: {
      email: { type: Boolean, default: true },
      push: { type: Boolean, default: true },
      sms: { type: Boolean, default: false },
    },
  },
}, {
  timestamps: true,
});

userProfileSchema.index({ user: 1 });

// Static methods
userProfileSchema.statics.findByUserId = function(userId) {
  return this.findOne({ user: userId });
};

userProfileSchema.statics.createForUser = function(userId, profileData) {
  return this.create({
    user: userId,
    ...profileData,
  });
};

module.exports = mongoose.model('UserProfile', userProfileSchema);
```

### **One-to-Many Relationships**
```javascript
// Category Schema (One-to-Many with Posts)
const categorySchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Category name is required'],
    unique: true,
    trim: true,
    maxlength: [50, 'Category name cannot exceed 50 characters'],
  },

  slug: {
    type: String,
    unique: true,
    lowercase: true,
  },

  description: {
    type: String,
    maxlength: [200, 'Description cannot exceed 200 characters'],
  },

  color: {
    type: String,
    default: '#007bff',
    validate: {
      validator: function(color) {
        return /^#[0-9A-F]{6}$/i.test(color);
      },
      message: 'Color must be a valid hex color',
    },
  },

  icon: {
    type: String,
    default: null,
  },

  // Metadata
  postCount: {
    type: Number,
    default: 0,
  },

  isActive: {
    type: Boolean,
    default: true,
  },

  order: {
    type: Number,
    default: 0,
  },
}, {
  timestamps: true,
});

// Indexes
categorySchema.index({ slug: 1 });
categorySchema.index({ isActive: 1 });
categorySchema.index({ order: 1 });

// Pre-save middleware
categorySchema.pre('save', function(next) {
  if (this.isModified('name') && !this.slug) {
    this.slug = this.name
      .toLowerCase()
      .replace(/[^a-zA-Z0-9 ]/g, '')
      .replace(/\s+/g, '-')
      .replace(/-+/g, '-')
      .trim('-');
  }
  next();
});

// Virtual for posts
categorySchema.virtual('posts', {
  ref: 'Post',
  localField: '_id',
  foreignField: 'category',
  justOne: false,
  options: { sort: { createdAt: -1 } },
});

// Static methods
categorySchema.statics.findActive = function() {
  return this.find({ isActive: true }).sort({ order: 1 });
};

categorySchema.statics.findBySlug = function(slug) {
  return this.findOne({ slug, isActive: true });
};

categorySchema.statics.updatePostCount = async function(categoryId) {
  const Post = mongoose.model('Post');
  const count = await Post.countDocuments({
    category: categoryId,
    status: 'published',
  });

  return this.updateOne(
    { _id: categoryId },
    { $set: { postCount: count } }
  );
};

module.exports = mongoose.model('Category', categorySchema);
```

### **Many-to-Many Relationships**
```javascript
// Tag Schema (Many-to-Many with Posts)
const tagSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Tag name is required'],
    unique: true,
    trim: true,
    lowercase: true,
    maxlength: [30, 'Tag name cannot exceed 30 characters'],
  },

  slug: {
    type: String,
    unique: true,
    lowercase: true,
  },

  description: {
    type: String,
    maxlength: [100, 'Description cannot exceed 100 characters'],
  },

  color: {
    type: String,
    default: '#6c757d',
  },

  // Usage statistics
  usageCount: {
    type: Number,
    default: 0,
  },

  isActive: {
    type: Boolean,
    default: true,
  },
}, {
  timestamps: true,
});

// Indexes
tagSchema.index({ name: 1 });
tagSchema.index({ slug: 1 });
tagSchema.index({ usageCount: -1 });

// Pre-save middleware
tagSchema.pre('save', function(next) {
  if (this.isModified('name') && !this.slug) {
    this.slug = this.name.replace(/\s+/g, '-').toLowerCase();
  }
  next();
});

// Virtual for posts
tagSchema.virtual('posts', {
  ref: 'Post',
  localField: '_id',
  foreignField: 'tags',
  justOne: false,
  options: { sort: { createdAt: -1 } },
});

// Static methods
tagSchema.statics.findPopular = function(limit = 20) {
  return this.find({ isActive: true })
    .sort({ usageCount: -1 })
    .limit(limit);
};

tagSchema.statics.findByName = function(name) {
  return this.findOne({
    name: name.toLowerCase(),
    isActive: true,
  });
};

tagSchema.statics.createIfNotExists = async function(tagName, options = {}) {
  let tag = await this.findByName(tagName);

  if (!tag) {
    tag = await this.create({
      name: tagName,
      ...options,
    });
  }

  return tag;
};

tagSchema.statics.updateUsageCount = async function(tagId) {
  const Post = mongoose.model('Post');
  const count = await Post.countDocuments({
    tags: tagId,
    status: 'published',
  });

  return this.updateOne(
    { _id: tagId },
    { $set: { usageCount: count } }
  );
};

module.exports = mongoose.model('Tag', tagSchema);
```

---

## 📊 **Indexing Strategies**

### **Index Types and Usage**
```javascript
// Single Field Index
userSchema.index({ email: 1 }); // Ascending order
userSchema.index({ createdAt: -1 }); // Descending order

// Compound Index
userSchema.index({ isActive: 1, role: 1 });
postSchema.index({ author: 1, createdAt: -1 });

// Text Index for Full-Text Search
postSchema.index({
  title: 'text',
  content: 'text',
  excerpt: 'text',
}, {
  weights: {
    title: 10,
    content: 4,
    excerpt: 2,
  },
  name: 'PostTextIndex',
});

// Geospatial Index
const locationSchema = new mongoose.Schema({
  name: String,
  location: {
    type: { type: String, default: 'Point' },
    coordinates: [Number], // [longitude, latitude]
  },
});

locationSchema.index({ location: '2dsphere' });

// TTL Index (Time To Live)
const sessionSchema = new mongoose.Schema({
  userId: mongoose.Schema.Types.ObjectId,
  token: String,
  createdAt: {
    type: Date,
    default: Date.now,
    expires: 86400, // 24 hours in seconds
  },
});

// Hashed Index for High Cardinality
userSchema.index({ email: 'hashed' });

// Sparse Index (only index documents that have the field)
userSchema.index({ lastLogin: 1 }, { sparse: true });

// Unique Index
userSchema.index({ email: 1 }, { unique: true });

// Partial Index (index only documents matching condition)
postSchema.index(
  { publishedAt: 1 },
  {
    partialFilterExpression: {
      status: 'published',
      publishedAt: { $exists: true },
    },
  }
);
```

### **Index Performance Analysis**
```javascript
// Analyze index usage
const analyzeIndexes = async () => {
  const db = mongoose.connection.db;

  // Get collection stats
  const stats = await db.collection('users').stats();
  console.log('Collection Stats:', {
    totalDocuments: stats.count,
    totalSize: stats.size,
    avgDocumentSize: stats.avgObjSize,
    indexes: stats.nindexes,
  });

  // Analyze specific index
  const indexStats = await db.collection('users').aggregate([
    { $indexStats: {} },
  ]).toArray();

  console.log('Index Usage Stats:', indexStats);

  // Explain query performance
  const explanation = await db.collection('users').find({
    isActive: true,
    role: 'user',
  }).explain('executionStats');

  console.log('Query Explanation:', {
    executionTimeMillis: explanation.executionStats.executionTimeMillis,
    totalDocsExamined: explanation.executionStats.totalDocsExamined,
    totalDocsReturned: explanation.executionStats.totalDocsReturned,
    indexesUsed: explanation.executionStats.winningPlan?.inputStage?.indexName,
  });
};

// Monitor slow queries
const monitorSlowQueries = () => {
  const db = mongoose.connection.db;

  // Enable profiling for slow queries (>100ms)
  db.setProfilingLevel(1, { slowms: 100 });

  // Listen for slow queries
  const changeStream = db.collection('system.profile').watch();

  changeStream.on('change', (change) => {
    const { operationType, fullDocument } = change;

    if (operationType === 'insert' && fullDocument) {
      console.log('Slow Query Detected:', {
        operation: fullDocument.op,
        collection: fullDocument.ns,
        duration: fullDocument.millis,
        query: fullDocument.query,
        plan: fullDocument.planSummary,
      });
    }
  });
};
```

---

## ⚡ **Query Optimization**

### **Efficient Query Patterns**
```javascript
// Optimized User Queries
const getUsersOptimized = async (filters = {}, options = {}) => {
  const {
    page = 1,
    limit = 10,
    sortBy = 'createdAt',
    sortOrder = -1,
    search,
  } = options;

  // Build query object
  const query = { isActive: true };

  // Add filters
  if (filters.role) query.role = filters.role;
  if (filters.isVerified !== undefined) query.isVerified = filters.isVerified;

  // Add search
  if (search) {
    query.$or = [
      { firstName: new RegExp(search, 'i') },
      { lastName: new RegExp(search, 'i') },
      { email: new RegExp(search, 'i') },
    ];
  }

  // Execute query with optimization
  const users = await User.find(query)
    .select('firstName lastName email avatar isVerified role createdAt') // Only select needed fields
    .sort({ [sortBy]: sortOrder })
    .skip((page - 1) * limit)
    .limit(limit)
    .lean(); // Return plain objects instead of Mongoose documents

  // Get total count for pagination
  const total = await User.countDocuments(query);

  return {
    users,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit),
    },
  };
};

// Optimized Post Queries
const getPostsOptimized = async (filters = {}, options = {}) => {
  const {
    page = 1,
    limit = 10,
    includeAuthor = true,
    includeCategory = true,
    includeComments = false,
  } = options;

  // Build aggregation pipeline for complex queries
  const pipeline = [];

  // Match stage
  const matchStage = { status: 'published' };
  if (filters.category) matchStage.category = mongoose.Types.ObjectId(filters.category);
  if (filters.author) matchStage.author = mongoose.Types.ObjectId(filters.author);
  if (filters.tags && filters.tags.length > 0) {
    matchStage.tags = { $in: filters.tags.map(id => mongoose.Types.ObjectId(id)) };
  }
  pipeline.push({ $match: matchStage });

  // Lookup stages for population
  if (includeAuthor) {
    pipeline.push({
      $lookup: {
        from: 'users',
        localField: 'author',
        foreignField: '_id',
        as: 'author',
        pipeline: [
          {
            $project: {
              firstName: 1,
              lastName: 1,
              avatar: 1,
              isVerified: 1,
            },
          },
        ],
      },
    });
    pipeline.push({ $unwind: '$author' });
  }

  if (includeCategory) {
    pipeline.push({
      $lookup: {
        from: 'categories',
        localField: 'category',
        foreignField: '_id',
        as: 'category',
        pipeline: [
          {
            $project: {
              name: 1,
              slug: 1,
              color: 1,
            },
          },
        ],
      },
    });
    pipeline.push({ $unwind: '$category' });
  }

  if (includeComments) {
    pipeline.push({
      $lookup: {
        from: 'users',
        let: { commentAuthorIds: '$comments.author' },
        pipeline: [
          {
            $match: {
              $expr: { $in: ['$_id', '$$commentAuthorIds'] },
            },
          },
          {
            $project: {
              firstName: 1,
              lastName: 1,
              avatar: 1,
            },
          },
        ],
        as: 'commentAuthors',
      },
    });
  }

  // Add fields for computed values
  pipeline.push({
    $addFields: {
      likeCount: { $size: '$likes' },
      commentCount: { $size: '$comments' },
      readingTime: {
        $ceil: {
          $divide: [
            { $size: { $split: ['$content', ' '] } },
            200, // words per minute
          ],
        },
      },
    },
  });

  // Sort stage
  pipeline.push({ $sort: { publishedAt: -1 } });

  // Pagination
  pipeline.push({ $skip: (page - 1) * limit });
  pipeline.push({ $limit: limit });

  // Execute aggregation
  const posts = await Post.aggregate(pipeline);

  // Get total count
  const totalPipeline = [
    { $match: matchStage },
    { $count: 'total' },
  ];
  const totalResult = await Post.aggregate(totalPipeline);
  const total = totalResult[0]?.total || 0;

  return {
    posts,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit),
    },
  };
};
```

### **Caching Strategies**
```javascript
// Redis Caching Implementation
const redis = require('redis');
const client = redis.createClient();

client.on('error', (err) => console.error('Redis Client Error', err));

// Cache middleware
const cacheMiddleware = (keyGenerator, ttl = 300) => {
  return async (req, res, next) => {
    const key = keyGenerator(req);

    try {
      const cachedData = await client.get(key);
      if (cachedData) {
        console.log('Cache hit for key:', key);
        return res.json(JSON.parse(cachedData));
      }
    } catch (error) {
      console.error('Cache read error:', error);
    }

    // Store original send method
    const originalSend = res.json;

    // Override json method to cache response
    res.json = function(data) {
      try {
        client.setex(key, ttl, JSON.stringify(data));
        console.log('Cache set for key:', key);
      } catch (error) {
        console.error('Cache write error:', error);
      }

      // Call original method
      originalSend.call(this, data);
    };

    next();
  };
};

// Usage
app.get('/api/users',
  cacheMiddleware((req) => `users:${req.query.page || 1}:${req.query.limit || 10}`),
  getUsers
);

// Invalidate cache
const invalidateUserCache = async (userId) => {
  const keys = await client.keys(`users:*`);
  if (keys.length > 0) {
    await client.del(keys);
  }
};

// Cache with tags for better invalidation
class CacheManager {
  constructor() {
    this.client = redis.createClient();
    this.tags = new Map();
  }

  async set(key, data, options = {}) {
    const { ttl = 300, tags = [] } = options;

    // Store data
    await this.client.setex(key, ttl, JSON.stringify(data));

    // Store tag relationships
    for (const tag of tags) {
      if (!this.tags.has(tag)) {
        this.tags.set(tag, new Set());
      }
      this.tags.get(tag).add(key);
    }
  }

  async get(key) {
    const data = await this.client.get(key);
    return data ? JSON.parse(data) : null;
  }

  async invalidateTag(tag) {
    const keys = this.tags.get(tag);
    if (keys) {
      const keysArray = Array.from(keys);
      if (keysArray.length > 0) {
        await this.client.del(keysArray);
        this.tags.delete(tag);
      }
    }
  }

  async invalidateKey(key) {
    await this.client.del(key);

    // Remove from tag sets
    for (const [tag, keys] of this.tags) {
      keys.delete(key);
      if (keys.size === 0) {
        this.tags.delete(tag);
      }
    }
  }
}

const cacheManager = new CacheManager();

// Usage
app.get('/api/posts/:id', async (req, res) => {
  const { id } = req.params;
  const cacheKey = `post:${id}`;

  // Try cache first
  let post = await cacheManager.get(cacheKey);

  if (!post) {
    // Fetch from database
    post = await Post.findById(id).populate('author category');

    // Cache with tags
    await cacheManager.set(cacheKey, post, {
      ttl: 600, // 10 minutes
      tags: ['posts', `post:${id}`, `author:${post.author._id}`],
    });
  }

  res.json(post);
});

// Invalidate cache when post is updated
app.put('/api/posts/:id', async (req, res) => {
  const { id } = req.params;
  const updatedPost = await Post.findByIdAndUpdate(id, req.body, { new: true });

  // Invalidate related cache
  await cacheManager.invalidateTag(`post:${id}`);

  res.json(updatedPost);
});
```

---

## ✅ **Data Validation**

### **Schema Validation**
```javascript
// Enhanced validation with custom validators
const userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    validate: [
      {
        validator: function(email) {
          return /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(email);
        },
        message: 'Please enter a valid email address',
      },
      {
        validator: async function(email) {
          const user = await this.constructor.findOne({ email });
          return !user || user._id.equals(this._id);
        },
        message: 'Email address is already registered',
      },
    ],
  },

  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: [8, 'Password must be at least 8 characters long'],
    validate: {
      validator: function(password) {
        // At least one uppercase, one lowercase, one number, one special character
        return /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/.test(password);
      },
      message: 'Password must contain at least one uppercase letter, one lowercase letter, one number, and one special character',
    },
  },

  age: {
    type: Number,
    min: [13, 'Age must be at least 13'],
    max: [120, 'Age cannot exceed 120'],
    validate: {
      validator: Number.isInteger,
      message: 'Age must be a whole number',
    },
  },

  phone: {
    type: String,
    validate: {
      validator: function(phone) {
        return /^(\+91[-\s]?)?[0]?(91)?[789]\d{9}$/.test(phone);
      },
      message: 'Please enter a valid Indian phone number',
    },
  },

  website: {
    type: String,
    validate: {
      validator: function(url) {
        return /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/.test(url);
      },
      message: 'Please enter a valid URL',
    },
  },
});

// Custom validation for business logic
userSchema.pre('validate', function(next) {
  // Custom validation logic
  if (this.role === 'admin' && this.age < 21) {
    this.invalidate('age', 'Admin must be at least 21 years old');
  }

  if (this.isVerified && !this.emailVerifiedAt) {
    this.invalidate('isVerified', 'Verified users must have email verification timestamp');
  }

  next();
});
```

### **Runtime Validation**
```javascript
// Input sanitization middleware
const sanitizeInput = (req, res, next) => {
  // Sanitize strings
  const sanitizeString = (str) => {
    if (typeof str !== 'string') return str;
    return str
      .trim()
      .replace(/[<>]/g, '') // Remove potential HTML tags
      .slice(0, 1000); // Limit length
  };

  // Recursively sanitize object
  const sanitizeObject = (obj) => {
    for (const key in obj) {
      if (typeof obj[key] === 'string') {
        obj[key] = sanitizeString(obj[key]);
      } else if (typeof obj[key] === 'object' && obj[key] !== null) {
        sanitizeObject(obj[key]);
      }
    }
  };

  if (req.body) sanitizeObject(req.body);
  if (req.query) sanitizeObject(req.query);
  if (req.params) sanitizeObject(req.params);

  next();
};

// Rate limiting for validation endpoints
const validationLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 10, // 10 validation requests per window
  message: 'Too many validation requests, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
});

// Validation result formatter
const formatValidationErrors = (errors) => {
  return errors.array().map(error => ({
    field: error.param,
    message: error.msg,
    value: error.value,
  }));
};

// Comprehensive validation middleware
const validateRequest = (validations) => {
  return async (req, res, next) => {
    // Run express-validator validations
    await Promise.all(validations.map(validation => validation.run(req)));

    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        success: false,
        error: 'Validation failed',
        details: formatValidationErrors(errors),
      });
    }

    next();
  };
};
```

---

## 💾 **Backup and Recovery**

### **Database Backup Strategies**
```javascript
// MongoDB Backup Script
const { spawn } = require('child_process');
const path = require('path');
const fs = require('fs').promises;

class DatabaseBackup {
  constructor(options = {}) {
    this.host = options.host || 'localhost';
    this.port = options.port || 27017;
    this.database = options.database || 'myapp';
    this.username = options.username;
    this.password = options.password;
    this.backupDir = options.backupDir || './backups';
  }

  async createBackup() {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    const backupPath = path.join(this.backupDir, `backup-${timestamp}`);

    // Ensure backup directory exists
    await fs.mkdir(backupPath, { recursive: true });

    return new Promise((resolve, reject) => {
      const mongodump = spawn('mongodump', [
        `--host=${this.host}`,
        `--port=${this.port}`,
        `--db=${this.database}`,
        `--out=${backupPath}`,
        ...(this.username ? [`--username=${this.username}`] : []),
        ...(this.password ? [`--password=${this.password}`] : []),
      ]);

      mongodump.stdout.on('data', (data) => {
        console.log('Backup stdout:', data.toString());
      });

      mongodump.stderr.on('data', (data) => {
        console.error('Backup stderr:', data.toString());
      });

      mongodump.on('close', (code) => {
        if (code === 0) {
          console.log(`Backup created successfully: ${backupPath}`);
          resolve(backupPath);
        } else {
          reject(new Error(`Backup failed with code ${code}`));
        }
      });

      mongodump.on('error', (error) => {
        reject(error);
      });
    });
  }

  async restoreBackup(backupPath) {
    return new Promise((resolve, reject) => {
      const mongorestore = spawn('mongorestore', [
        `--host=${this.host}`,
        `--port=${this.port}`,
        `--db=${this.database}`,
        backupPath,
        ...(this.username ? [`--username=${this.username}`] : []),
        ...(this.password ? [`--password=${this.password}`] : []),
      ]);

      mongorestore.stdout.on('data', (data) => {
        console.log('Restore stdout:', data.toString());
      });

      mongorestore.stderr.on('data', (data) => {
        console.error('Restore stderr:', data.toString());
      });

      mongorestore.on('close', (code) => {
        if (code === 0) {
          console.log('Database restored successfully');
          resolve();
        } else {
          reject(new Error(`Restore failed with code ${code}`));
        }
      });

      mongorestore.on('error', (error) => {
        reject(error);
      });
    });
  }

  async listBackups() {
    try {
      const files = await fs.readdir(this.backupDir);
      const backups = files
        .filter(file => file.startsWith('backup-'))
        .map(file => ({
          name: file,
          path: path.join(this.backupDir, file),
          createdAt: new Date(file.replace('backup-', '').replace(/-/g, ':')),
        }))
        .sort((a, b) => b.createdAt - a.createdAt);

      return backups;
    } catch (error) {
      console.error('Error listing backups:', error);
      return [];
    }
  }

  async cleanupOldBackups(retentionDays = 30) {
    const backups = await this.listBackups();
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - retentionDays);

    const oldBackups = backups.filter(backup => backup.createdAt < cutoffDate);

    for (const backup of oldBackups) {
      try {
        await fs.rmdir(backup.path, { recursive: true });
        console.log(`Cleaned up old backup: ${backup.name}`);
      } catch (error) {
        console.error(`Error cleaning up backup ${backup.name}:`, error);
      }
    }
  }

  async scheduleBackup(cronExpression = '0 2 * * *') {
    // This would typically use node-cron or similar
    const cron = require('node-cron');

    cron.schedule(cronExpression, async () => {
      try {
        console.log('Starting scheduled backup...');
        await this.createBackup();
        await this.cleanupOldBackups();
        console.log('Scheduled backup completed');
      } catch (error) {
        console.error('Scheduled backup failed:', error);
      }
    });

    console.log(`Backup scheduled with cron expression: ${cronExpression}`);
  }
}

// Usage
const backupManager = new DatabaseBackup({
  host: 'localhost',
  port: 27017,
  database: 'myapp',
  backupDir: './backups',
});

// Create backup
app.post('/api/admin/backup', async (req, res) => {
  try {
    const backupPath = await backupManager.createBackup();
    res.json({ success: true, backupPath });
  } catch (error) {
    res.status(500).json({ error: 'Backup failed', details: error.message });
  }
});

// List backups
app.get('/api/admin/backups', async (req, res) => {
  try {
    const backups = await backupManager.listBackups();
    res.json({ backups });
  } catch (error) {
    res.status(500).json({ error: 'Failed to list backups' });
  }
});

// Restore backup
app.post('/api/admin/restore/:backupName', async (req, res) => {
  try {
    const { backupName } = req.params;
    const backupPath = path.join(backupManager.backupDir, backupName);

    // Verify backup exists
    await fs.access(backupPath);

    await backupManager.restoreBackup(backupPath);
    res.json({ success: true, message: 'Database restored successfully' });
  } catch (error) {
    res.status(500).json({ error: 'Restore failed', details: error.message });
  }
});

// Schedule automatic backups
backupManager.scheduleBackup('0 2 * * *'); // Daily at 2 AM
```

### **Point-in-Time Recovery**
```javascript
// MongoDB Oplog Backup for Point-in-Time Recovery
class PointInTimeRecovery {
  constructor(options = {}) {
    this.host = options.host || 'localhost';
    this.port = options.port || 27017;
    this.database = options.database || 'myapp';
    this.oplogPath = options.oplogPath || './oplog';
  }

  async backupOplog() {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    const oplogBackupPath = path.join(this.oplogPath, `oplog-${timestamp}.bson`);

    return new Promise((resolve, reject) => {
      const mongodump = spawn('mongodump', [
        `--host=${this.host}`,
        `--port=${this.port}`,
        `--db=local`,
        `--collection=oplog.rs`,
        `--out=${this.oplogPath}`,
        `--query={"ts":{"$gt":{"$timestamp":{"t":${Math.floor(Date.now() / 1000)},"i":0}}}}`,
      ]);

      mongodump.on('close', (code) => {
        if (code === 0) {
          resolve(oplogBackupPath);
        } else {
          reject(new Error(`Oplog backup failed with code ${code}`));
        }
      });

      mongodump.on('error', reject);
    });
  }

  async restoreToPointInTime(targetTimestamp) {
    // This is a simplified example
    // In production, you would use mongorestore with oplog replay
    console.log(`Restoring to timestamp: ${targetTimestamp}`);

    // Implementation would involve:
    // 1. Restore base backup
    // 2. Apply oplog operations up to target timestamp
    // 3. Handle conflicts and rollbacks
  }
}
```

---

## 📊 **Performance Monitoring**

### **Database Performance Metrics**
```javascript
// Performance monitoring middleware
const performanceMonitor = (req, res, next) => {
  const start = Date.now();
  const startMemory = process.memoryUsage();

  res.on('finish', () => {
    const duration = Date.now() - start;
    const endMemory = process.memoryUsage();
    const memoryDelta = endMemory.heapUsed - startMemory.heapUsed;

    // Log slow queries
    if (duration > 1000) { // 1 second
      console.log(`Slow Query: ${req.method} ${req.path} - ${duration}ms`);
    }

    // Log memory usage
    if (Math.abs(memoryDelta) > 10 * 1024 * 1024) { // 10MB
      console.log(`Memory Change: ${req.method} ${req.path} - ${(memoryDelta / 1024 / 1024).toFixed(2)}MB`);
    }

    // Store metrics for analysis
    storePerformanceMetrics({
      method: req.method,
      path: req.path,
      duration,
      memoryDelta,
      timestamp: new Date(),
      statusCode: res.statusCode,
    });
  });

  next();
};

// Database connection monitoring
const monitorDatabaseConnection = () => {
  const db = mongoose.connection;

  setInterval(() => {
    db.db.admin().ping((err, result) => {
      if (err) {
        console.error('Database ping failed:', err);
        // Send alert or take corrective action
      } else {
        console.log('Database ping successful');
      }
    });
  }, 30000); // Check every 30 seconds
};

// Query performance analysis
const analyzeQueryPerformance = async () => {
  const db = mongoose.connection.db;

  // Enable profiling
  await db.setProfilingLevel(2, { slowms: 50 });

  // Monitor system.profile collection
  const changeStream = db.collection('system.profile').watch();

  changeStream.on('change', (change) => {
    const { operationType, fullDocument } = change;

    if (operationType === 'insert' && fullDocument) {
      const { op, ns, millis, planSummary, query } = fullDocument;

      if (millis > 100) {
        console.log('Slow Query Detected:', {
          operation: op,
          collection: ns,
          duration: millis,
          plan: planSummary,
          query: JSON.stringify(query).substring(0, 200),
        });

        // Store for further analysis
        storeSlowQuery({
          operation: op,
          collection: ns,
          duration: millis,
          plan: planSummary,
          query,
          timestamp: new Date(),
        });
      }
    }
  });
};

// Index usage analysis
const analyzeIndexUsage = async () => {
  const db = mongoose.connection.db;

  const indexStats = await db.collection('users').aggregate([
    { $indexStats: {} },
  ]).toArray();

  console.log('Index Usage Statistics:');
  indexStats.forEach(stat => {
    console.log(`Index: ${stat.name}`);
    console.log(`  Usage Count: ${stat.accesses?.count || 0}`);
    console.log(`  Since: ${stat.accesses?.since || 'N/A'}`);
  });

  // Identify unused indexes
  const unusedIndexes = indexStats.filter(stat =>
    !stat.name.startsWith('_id_') && (!stat.accesses || stat.accesses.count === 0)
  );

  if (unusedIndexes.length > 0) {
    console.log('Unused Indexes Found:');
    unusedIndexes.forEach(index => {
      console.log(`  - ${index.name}`);
    });
  }
};

// Connection pool monitoring
const monitorConnectionPool = () => {
  const db = mongoose.connection;

  setInterval(() => {
    const poolSize = db.db.serverConfig.poolSize;
    const availableConnections = db.db.serverConfig.availableConnections;

    console.log('Connection Pool Status:', {
      poolSize,
      availableConnections,
      usedConnections: poolSize - availableConnections,
    });

    // Alert if connection pool is nearly exhausted
    if (availableConnections < 2) {
      console.warn('WARNING: Connection pool nearly exhausted!');
      // Send alert or scale up connections
    }
  }, 60000); // Check every minute
};
```

### **Performance Optimization Dashboard**
```javascript
// Performance metrics storage
const performanceMetrics = [];

const storePerformanceMetrics = (metrics) => {
  performanceMetrics.push(metrics);

  // Keep only last 1000 metrics
  if (performanceMetrics.length > 1000) {
    performanceMetrics.shift();
  }
};

const storeSlowQuery = (query) => {
  // Store slow queries for analysis
  // In production, you might store this in a separate collection
  console.log('Slow query stored for analysis:', query);
};

// Performance dashboard endpoint
app.get('/api/admin/performance', (req, res) => {
  const now = Date.now();
  const lastHour = now - (60 * 60 * 1000);

  // Filter metrics from last hour
  const recentMetrics = performanceMetrics.filter(
    metric => metric.timestamp.getTime() > lastHour
  );

  // Calculate statistics
  const stats = {
    totalRequests: recentMetrics.length,
    averageResponseTime: recentMetrics.reduce((sum, m) => sum + m.duration, 0) / recentMetrics.length,
    slowestRequest: Math.max(...recentMetrics.map(m => m.duration)),
    fastestRequest: Math.min(...recentMetrics.map(m => m.duration)),
    errorRate: recentMetrics.filter(m => m.statusCode >= 400).length / recentMetrics.length * 100,
    memoryUsage: process.memoryUsage(),
    uptime: process.uptime(),
  };

  // Group by endpoint
  const endpointStats = {};
  recentMetrics.forEach(metric => {
    const key = `${metric.method} ${metric.path}`;
    if (!endpointStats[key]) {
      endpointStats[key] = {
        count: 0,
        totalDuration: 0,
        errors: 0,
      };
    }

    endpointStats[key].count++;
    endpointStats[key].totalDuration += metric.duration;
    if (metric.statusCode >= 400) {
      endpointStats[key].errors++;
    }
  });

  // Calculate averages
  Object.keys(endpointStats).forEach(key => {
    const stat = endpointStats[key];
    stat.averageDuration = stat.totalDuration / stat.count;
    stat.errorRate = stat.errors / stat.count * 100;
  });

  res.json({
    summary: stats,
    endpoints: endpointStats,
    timestamp: new Date(),
  });
});
```

---

## 🎯 **Practical Examples**

### **Complete Blog Database Schema**
```javascript
// models/index.js - Model exports
const User = require('./User');
const Post = require('./Post');
const Category = require('./Category');
const Tag = require('./Tag');
const Comment = require('./Comment');

module.exports = {
  User,
  Post,
  Category,
  Tag,
  Comment,
};

// models/Comment.js
const mongoose = require('mongoose');

const commentSchema = new mongoose.Schema({
  content: {
    type: String,
    required: [true, 'Comment content is required'],
    maxlength: [1000, 'Comment cannot exceed 1000 characters'],
  },

  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true,
  },

  post: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Post',
    required: true,
  },

  parentComment: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Comment',
    default: null,
  },

  replies: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Comment',
  }],

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

  isApproved: {
    type: Boolean,
    default: true,
  },

  isDeleted: {
    type: Boolean,
    default: false,
  },
}, {
  timestamps: true,
});

// Indexes
commentSchema.index({ post: 1, createdAt: -1 });
commentSchema.index({ author: 1, createdAt: -1 });
commentSchema.index({ parentComment: 1 });

// Virtuals
commentSchema.virtual('likeCount').get(function() {
  return this.likes.length;
});

commentSchema.virtual('replyCount').get(function() {
  return this.replies.length;
});

// Methods
commentSchema.methods.addLike = function(userId) {
  if (!this.likes.some(like => like.user.toString() === userId.toString())) {
    this.likes.push({ user: userId });
    return this.save();
  }
  return this;
};

commentSchema.methods.removeLike = function(userId) {
  this.likes = this.likes.filter(
    like => like.user.toString() !== userId.toString()
  );
  return this.save();
};

commentSchema.methods.addReply = function(replyId) {
  if (!this.replies.includes(replyId)) {
    this.replies.push(replyId);
    return this.save();
  }
  return this;
};

module.exports = mongoose.model('Comment', commentSchema);
```

### **Database Service Layer**
```javascript
// services/database.js
const mongoose = require('mongoose');
const { User, Post, Category, Tag, Comment } = require('../models');

class DatabaseService {
  constructor() {
    this.isConnected = false;
  }

  async connect(uri) {
    try {
      await mongoose.connect(uri, {
        useNewUrlParser: true,
        useUnifiedTopology: true,
        useCreateIndex: true,
        useFindAndModify: false,
      });

      this.isConnected = true;
      console.log('Database connected successfully');

      // Handle connection events
      mongoose.connection.on('error', (err) => {
        console.error('Database connection error:', err);
        this.isConnected = false;
      });

      mongoose.connection.on('disconnected', () => {
        console.log('Database disconnected');
        this.isConnected = false;
      });

    } catch (error) {
      console.error('Database connection failed:', error);
      throw error;
    }
  }

  async disconnect() {
    try {
      await mongoose.connection.close();
      this.isConnected = false;
      console.log('Database disconnected successfully');
    } catch (error) {
      console.error('Database disconnection error:', error);
      throw error;
    }
  }

  // User operations
  async createUser(userData) {
    const user = new User(userData);
    return await user.save();
  }

  async findUserById(id) {
    return await User.findById(id);
  }

  async findUserByEmail(email) {
    return await User.findOne({ email: email.toLowerCase() });
  }

  async updateUser(id, updateData) {
    return await User.findByIdAndUpdate(id, updateData, { new: true });
  }

  async deleteUser(id) {
    return await User.findByIdAndDelete(id);
  }

  // Post operations
  async createPost(postData) {
    const post = new Post(postData);
    return await post.save();
  }

  async findPosts(query = {}, options = {}) {
    const { page = 1, limit = 10, sort = '-createdAt', populate = [] } = options;

    let queryBuilder = Post.find(query)
      .sort(sort)
      .skip((page - 1) * limit)
      .limit(limit);

    // Apply population
    populate.forEach(field => {
      queryBuilder = queryBuilder.populate(field);
    });

    const posts = await queryBuilder;
    const total = await Post.countDocuments(query);

    return {
      posts,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit),
      },
    };
  }

  async findPostById(id, populate = []) {
    let query = Post.findById(id);

    populate.forEach(field => {
      query = query.populate(field);
    });

    return await query;
  }

  async updatePost(id, updateData) {
    return await Post.findByIdAndUpdate(id, updateData, { new: true });
  }

  async deletePost(id) {
    return await Post.findByIdAndDelete(id);
  }

  // Category operations
  async createCategory(categoryData) {
    const category = new Category(categoryData);
    return await category.save();
  }

  async findCategories(query = {}) {
    return await Category.find(query).sort({ order: 1 });
  }

  async updateCategory(id, updateData) {
    return await Category.findByIdAndUpdate(id, updateData, { new: true });
  }

  // Tag operations
  async createTag(tagData) {
    const tag = new Tag(tagData);
    return await tag.save();
  }

  async findTags(query = {}, options = {}) {
    const { limit = 20, sort = '-usageCount' } = options;
    return await Tag.find(query).sort(sort).limit(limit);
  }

  // Comment operations
  async createComment(commentData) {
    const comment = new Comment(commentData);
    return await comment.save();
  }

  async findCommentsByPost(postId, options = {}) {
    const { page = 1, limit = 10, populate = ['author'] } = options;

    let query = Comment.find({
      post: postId,
      isDeleted: false,
      parentComment: null, // Only top-level comments
    })
    .sort('-createdAt')
    .skip((page - 1) * limit)
    .limit(limit);

    populate.forEach(field => {
      query = query.populate(field);
    });

    const comments = await query;
    const total = await Comment.countDocuments({
      post: postId,
      isDeleted: false,
      parentComment: null,
    });

    return {
      comments,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit),
      },
    };
  }

  // Utility methods
  async getDatabaseStats() {
    const db = mongoose.connection.db;
    const stats = await db.stats();

    return {
      database: stats.db,
      collections: stats.collections,
      objects: stats.objects,
      dataSize: stats.dataSize,
      storageSize: stats.storageSize,
      indexes: stats.indexes,
      indexSize: stats.indexSize,
    };
  }

  async healthCheck() {
    try {
      await mongoose.connection.db.admin().ping();
      return {
        status: 'healthy',
        database: mongoose.connection.name,
        connected: this.isConnected,
      };
    } catch (error) {
      return {
        status: 'unhealthy',
        error: error.message,
        connected: false,
      };
    }
  }
}

// Export singleton instance
const databaseService = new DatabaseService();
module.exports = databaseService;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Database Design Principles**: Normalization, denormalization, scalability
- ✅ **MongoDB Fundamentals**: CRUD operations, connection management
- ✅ **Schema Design**: Mongoose schemas, validation, indexes
- ✅ **Data Relationships**: One-to-one, one-to-many, many-to-many
- ✅ **Indexing Strategies**: Single, compound, text, geospatial indexes
- ✅ **Query Optimization**: Efficient queries, aggregation pipelines
- ✅ **Data Validation**: Schema validation, custom validators
- ✅ **Backup & Recovery**: Database backup strategies
- ✅ **Performance Monitoring**: Index usage, slow query analysis

### **Best Practices**
1. **Design schemas based on query patterns** - Optimize for read operations
2. **Use appropriate indexes** - Balance between query performance and write performance
3. **Implement proper validation** - Both at schema and application level
4. **Handle relationships efficiently** - Use embedding vs referencing appropriately
5. **Monitor performance** - Use MongoDB profiling and explain plans
6. **Plan for scalability** - Design for horizontal scaling when needed
7. **Backup regularly** - Implement automated backup strategies
8. **Use connection pooling** - Efficient database connection management

### **Next Steps**
- Implement advanced aggregation pipelines
- Set up database replication and sharding
- Implement database migrations
- Create database performance dashboards
- Learn about NoSQL database alternatives

---

## 🎯 **Assignment**

### **Task 1: E-commerce Database Design**
Design and implement a complete e-commerce database with:
- User management with roles and permissions
- Product catalog with categories and variants
- Shopping cart and order management
- Payment and shipping integration
- Review and rating system
- Inventory management
- Analytics and reporting

### **Task 2: Social Media Platform**
Create a social media database schema including:
- User profiles with followers/following
- Posts with likes, comments, and shares
- Media attachments and hashtags
- Direct messaging system
- Notification system
- Content moderation features
- Analytics and insights

### **Task 3: Learning Management System**
Design a database for an online learning platform with:
- Course and lesson management
- Student enrollment and progress tracking
- Quiz and assessment system
- Certificate generation
- Discussion forums
- Video streaming integration
- Progress analytics

---

## 📚 **Additional Resources**
- [MongoDB Documentation](https://docs.mongodb.com/)
- [Mongoose Documentation](https://mongoosejs.com/docs/)
- [Database Design Best Practices](https://www.mongodb.com/blog/post/database-design-best-practices)
- [MongoDB University](https://university.mongodb.com/)
- [Database Indexing Guide](https://docs.mongodb.com/manual/indexes/)

---

**Next Lesson**: [Lesson 23: RESTful APIs & GraphQL](Lesson%2017_%20Push%20Notifications%20&%20Local%20Notifications.md)