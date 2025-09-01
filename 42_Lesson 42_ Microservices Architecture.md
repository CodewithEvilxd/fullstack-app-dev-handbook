# Lesson 42: Microservices Architecture

## 🎯 **Learning Objectives**
- Understand microservices architecture principles
- Design scalable microservices systems
- Implement service communication patterns
- Handle distributed system challenges
- Deploy and manage microservices

## 📚 **Table of Contents**
1. [Microservices Fundamentals](#microservices-fundamentals)
2. [Service Design Principles](#service-design-principles)
3. [API Gateway Pattern](#api-gateway-pattern)
4. [Service Communication](#service-communication)
5. [Data Management](#data-management)
6. [Service Discovery](#service-discovery)
7. [Circuit Breaker Pattern](#circuit-breaker-pattern)
8. [Distributed Tracing](#distributed-tracing)
9. [Container Orchestration](#container-orchestration)
10. [Practical Examples](#practical-examples)

---

## 🏗️ **Microservices Fundamentals**

### **What are Microservices?**
Microservices are an architectural approach where a single application is composed of many loosely coupled and independently deployable smaller services.

### **Key Characteristics**
- ✅ **Small, focused services**: Each service has a single responsibility
- ✅ **Independently deployable**: Services can be updated without affecting others
- ✅ **Technology agnostic**: Different services can use different technologies
- ✅ **Resilient**: Failure in one service doesn't bring down the entire system
- ✅ **Scalable**: Individual services can be scaled independently

### **Benefits**
- **Scalability**: Scale individual services based on demand
- **Technology Diversity**: Use the best tool for each job
- **Team Autonomy**: Teams can work independently
- **Fault Isolation**: Problems are contained within services
- **Continuous Deployment**: Deploy services independently

### **Challenges**
- **Complexity**: Managing multiple services is complex
- **Data Consistency**: Ensuring data consistency across services
- **Network Latency**: Communication between services adds latency
- **Testing**: Testing interactions between services is difficult
- **Monitoring**: Monitoring distributed systems is challenging

---

## 📋 **Service Design Principles**

### **Single Responsibility Principle**
```javascript
// Bad: Monolithic user service
class UserService {
  createUser(userData) { /* ... */ }
  sendEmail(user, template) { /* ... */ }
  processPayment(user, amount) { /* ... */ }
  generateReport() { /* ... */ }
  uploadFile(file) { /* ... */ }
}

// Good: Focused microservices
class UserManagementService {
  createUser(userData) { /* ... */ }
  updateUser(userId, userData) { /* ... */ }
  deleteUser(userId) { /* ... */ }
}

class EmailService {
  sendEmail(recipient, template, data) { /* ... */ }
  sendBulkEmail(recipients, template, data) { /* ... */ }
}

class PaymentService {
  processPayment(userId, amount, method) { /* ... */ }
  refundPayment(paymentId) { /* ... */ }
}
```

### **Domain-Driven Design**
```javascript
// User domain service
class UserDomainService {
  constructor(userRepository, eventPublisher) {
    this.userRepository = userRepository;
    this.eventPublisher = eventPublisher;
  }

  async registerUser(userData) {
    // Validate user data
    this.validateUserData(userData);

    // Create user
    const user = await this.userRepository.create(userData);

    // Publish domain event
    await this.eventPublisher.publish('UserRegistered', {
      userId: user.id,
      email: user.email,
      timestamp: new Date(),
    });

    return user;
  }

  validateUserData(userData) {
    if (!userData.email || !userData.password) {
      throw new ValidationError('Email and password are required');
    }

    if (userData.password.length < 8) {
      throw new ValidationError('Password must be at least 8 characters');
    }
  }
}

// Order domain service
class OrderDomainService {
  constructor(orderRepository, inventoryService, paymentService, eventPublisher) {
    this.orderRepository = orderRepository;
    this.inventoryService = inventoryService;
    this.paymentService = paymentService;
    this.eventPublisher = eventPublisher;
  }

  async placeOrder(orderData) {
    // Check inventory
    const inventoryCheck = await this.inventoryService.checkAvailability(
      orderData.items
    );

    if (!inventoryCheck.available) {
      throw new Error('Insufficient inventory');
    }

    // Process payment
    const payment = await this.paymentService.processPayment(
      orderData.paymentMethod,
      orderData.total
    );

    // Create order
    const order = await this.orderRepository.create({
      ...orderData,
      paymentId: payment.id,
      status: 'confirmed',
    });

    // Update inventory
    await this.inventoryService.reserveItems(orderData.items);

    // Publish event
    await this.eventPublisher.publish('OrderPlaced', {
      orderId: order.id,
      userId: orderData.userId,
      total: orderData.total,
      timestamp: new Date(),
    });

    return order;
  }
}
```

### **Bounded Contexts**
```javascript
// User context
class UserContext {
  constructor(userRepository, emailService) {
    this.userRepository = userRepository;
    this.emailService = emailService;
  }

  async changePassword(userId, newPassword) {
    const user = await this.userRepository.findById(userId);

    // Domain logic for password change
    user.passwordHash = await this.hashPassword(newPassword);
    user.passwordChangedAt = new Date();

    await this.userRepository.update(user);

    // Send notification
    await this.emailService.sendPasswordChangeNotification(user.email);
  }
}

// Product context
class ProductContext {
  constructor(productRepository, searchService) {
    this.productRepository = productRepository;
    this.searchService = searchService;
  }

  async createProduct(productData) {
    const product = await this.productRepository.create(productData);

    // Update search index
    await this.searchService.indexProduct(product);

    return product;
  }

  async searchProducts(query, filters) {
    return await this.searchService.search(query, filters);
  }
}
```

---

## 🌐 **API Gateway Pattern**

### **API Gateway Implementation**
```javascript
// api-gateway/index.js
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');
const jwt = require('jsonwebtoken');
const rateLimit = require('express-rate-limit');

const app = express();

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP, please try again later.',
});

app.use(limiter);

// Authentication middleware
const authenticate = (req, res, next) => {
  const token = req.headers.authorization?.replace('Bearer ', '');

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// Request logging middleware
const requestLogger = (req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.originalUrl} ${res.statusCode} ${duration}ms`);
  });

  next();
};

app.use(requestLogger);

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// User service routes
app.use(
  '/api/users',
  authenticate,
  createProxyMiddleware({
    target: 'http://user-service:3001',
    changeOrigin: true,
    pathRewrite: {
      '^/api/users': '/api/v1/users',
    },
    onError: (err, req, res) => {
      console.error('User service error:', err);
      res.status(500).json({ error: 'User service unavailable' });
    },
  })
);

// Product service routes
app.use(
  '/api/products',
  createProxyMiddleware({
    target: 'http://product-service:3002',
    changeOrigin: true,
    pathRewrite: {
      '^/api/products': '/api/v1/products',
    },
    onError: (err, req, res) => {
      console.error('Product service error:', err);
      res.status(500).json({ error: 'Product service unavailable' });
    },
  })
);

// Order service routes (requires authentication)
app.use(
  '/api/orders',
  authenticate,
  createProxyMiddleware({
    target: 'http://order-service:3003',
    changeOrigin: true,
    pathRewrite: {
      '^/api/orders': '/api/v1/orders',
    },
    onError: (err, req, res) => {
      console.error('Order service error:', err);
      res.status(500).json({ error: 'Order service unavailable' });
    },
  })
);

// Aggregated endpoints
app.get('/api/dashboard', authenticate, async (req, res) => {
  try {
    const userId = req.user.id;

    // Fetch data from multiple services
    const [userProfile, recentOrders, recommendedProducts] = await Promise.allSettled([
      fetchUserProfile(userId),
      fetchRecentOrders(userId),
      fetchRecommendedProducts(userId),
    ]);

    res.json({
      user: userProfile.status === 'fulfilled' ? userProfile.value : null,
      recentOrders: recentOrders.status === 'fulfilled' ? recentOrders.value : [],
      recommendations: recommendedProducts.status === 'fulfilled' ? recommendedProducts.value : [],
    });
  } catch (error) {
    console.error('Dashboard aggregation error:', error);
    res.status(500).json({ error: 'Failed to load dashboard' });
  }
});

// Helper functions for service communication
async function fetchUserProfile(userId) {
  const response = await fetch(`http://user-service:3001/api/v1/users/${userId}`);
  if (!response.ok) throw new Error('Failed to fetch user profile');
  return response.json();
}

async function fetchRecentOrders(userId) {
  const response = await fetch(`http://order-service:3003/api/v1/orders?userId=${userId}&limit=5`);
  if (!response.ok) throw new Error('Failed to fetch recent orders');
  return response.json();
}

async function fetchRecommendedProducts(userId) {
  const response = await fetch(`http://product-service:3002/api/v1/products/recommendations?userId=${userId}`);
  if (!response.ok) throw new Error('Failed to fetch recommendations');
  return response.json();
}

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`API Gateway listening on port ${PORT}`);
});
```

### **Request Transformation**
```javascript
// Request transformation middleware
const requestTransformer = (req, res, next) => {
  // Add request ID
  req.requestId = generateRequestId();

  // Transform request body
  if (req.body) {
    req.body = transformRequestBody(req.body);
  }

  // Add common headers
  req.headers['x-request-id'] = req.requestId;
  req.headers['x-user-agent'] = req.get('User-Agent');
  req.headers['x-forwarded-for'] = req.ip;

  next();
};

// Response transformation
const responseTransformer = (proxyRes, req, res) => {
  // Add response headers
  proxyRes.headers['x-api-gateway'] = 'true';
  proxyRes.headers['x-request-id'] = req.requestId;

  // Transform response if needed
  if (proxyRes.headers['content-type']?.includes('application/json')) {
    // Could modify response body here
  }
};

// Circuit breaker for service calls
class CircuitBreaker {
  constructor(serviceName, failureThreshold = 5, recoveryTimeout = 30000) {
    this.serviceName = serviceName;
    this.failureThreshold = failureThreshold;
    this.recoveryTimeout = recoveryTimeout;
    this.failureCount = 0;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.nextAttemptTime = 0;
  }

  async call(serviceCall) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttemptTime) {
        throw new Error(`${this.serviceName} is currently unavailable`);
      }
      this.state = 'HALF_OPEN';
    }

    try {
      const result = await serviceCall();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }

  onFailure() {
    this.failureCount++;
    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
      this.nextAttemptTime = Date.now() + this.recoveryTimeout;
    }
  }
}

// Usage in API Gateway
const userServiceBreaker = new CircuitBreaker('user-service');
const productServiceBreaker = new CircuitBreaker('product-service');

// Protected service calls
app.get('/api/users/:id', authenticate, async (req, res) => {
  try {
    const user = await userServiceBreaker.call(() =>
      fetchUserProfile(req.params.id)
    );
    res.json(user);
  } catch (error) {
    res.status(503).json({ error: 'Service temporarily unavailable' });
  }
});
```

---

## 📡 **Service Communication**

### **Synchronous Communication**
```javascript
// HTTP client for service communication
class ServiceClient {
  constructor(baseURL, serviceName) {
    this.baseURL = baseURL;
    this.serviceName = serviceName;
    this.timeout = 5000; // 5 seconds
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      timeout: this.timeout,
      headers: {
        'Content-Type': 'application/json',
        'X-Service-Name': this.serviceName,
        ...options.headers,
      },
      ...options,
    };

    try {
      const response = await fetch(url, config);
      return await this.handleResponse(response);
    } catch (error) {
      console.error(`Service call failed: ${this.serviceName}${endpoint}`, error);
      throw new ServiceError(`Failed to call ${this.serviceName}`, error);
    }
  }

  async handleResponse(response) {
    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}));
      throw new ServiceError(
        `Service error: ${response.status}`,
        response.status,
        errorData
      );
    }

    const contentType = response.headers.get('content-type');
    if (contentType && contentType.includes('application/json')) {
      return await response.json();
    }

    return await response.text();
  }

  // Convenience methods
  get(endpoint, params = {}) {
    const queryString = new URLSearchParams(params).toString();
    const url = queryString ? `${endpoint}?${queryString}` : endpoint;
    return this.request(url);
  }

  post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  put(endpoint, data) {
    return this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }

  delete(endpoint) {
    return this.request(endpoint, {
      method: 'DELETE',
    });
  }
}

// Custom error class
class ServiceError extends Error {
  constructor(message, status, data) {
    super(message);
    this.name = 'ServiceError';
    this.status = status;
    this.data = data;
  }
}

// Service registry
class ServiceRegistry {
  constructor() {
    this.services = new Map();
  }

  register(name, client) {
    this.services.set(name, client);
  }

  get(name) {
    const service = this.services.get(name);
    if (!service) {
      throw new Error(`Service ${name} not found`);
    }
    return service;
  }
}

// Initialize service clients
const registry = new ServiceRegistry();

registry.register('user', new ServiceClient('http://user-service:3001', 'user-service'));
registry.register('product', new ServiceClient('http://product-service:3002', 'product-service'));
registry.register('order', new ServiceClient('http://order-service:3003', 'order-service'));

export { ServiceClient, ServiceRegistry, ServiceError };
export default registry;
```

### **Asynchronous Communication**
```javascript
// Message queue for async communication
const amqp = require('amqplib');

class MessageQueue {
  constructor() {
    this.connection = null;
    this.channel = null;
    this.exchanges = new Map();
  }

  async connect(url) {
    try {
      this.connection = await amqp.connect(url);
      this.channel = await this.connection.createChannel();

      console.log('Connected to message queue');
    } catch (error) {
      console.error('Failed to connect to message queue:', error);
      throw error;
    }
  }

  async createExchange(name, type = 'topic') {
    if (this.exchanges.has(name)) {
      return this.exchanges.get(name);
    }

    await this.channel.assertExchange(name, type, { durable: true });
    this.exchanges.set(name, name);
    return name;
  }

  async publish(exchange, routingKey, message, options = {}) {
    const exchangeName = await this.createExchange(exchange);

    this.channel.publish(
      exchangeName,
      routingKey,
      Buffer.from(JSON.stringify(message)),
      {
        persistent: true,
        timestamp: Date.now(),
        ...options,
      }
    );

    console.log(`Published message to ${exchange}:${routingKey}`);
  }

  async subscribe(exchange, queueName, routingKey, handler) {
    const exchangeName = await this.createExchange(exchange);

    const queue = await this.channel.assertQueue(queueName, { durable: true });
    await this.channel.bindQueue(queue.queue, exchangeName, routingKey);

    this.channel.consume(queue.queue, async (msg) => {
      if (msg) {
        try {
          const message = JSON.parse(msg.content.toString());
          await handler(message, msg);

          this.channel.ack(msg);
        } catch (error) {
          console.error('Error processing message:', error);
          this.channel.nack(msg, false, false); // Don't requeue
        }
      }
    });

    console.log(`Subscribed to ${exchange}:${routingKey} on queue ${queueName}`);
  }

  async close() {
    if (this.channel) {
      await this.channel.close();
    }
    if (this.connection) {
      await this.connection.close();
    }
  }
}

// Event-driven communication
class EventPublisher {
  constructor(messageQueue) {
    this.messageQueue = messageQueue;
  }

  async publishUserEvent(eventType, userId, data) {
    await this.messageQueue.publish(
      'user_events',
      `user.${eventType}`,
      {
        userId,
        eventType,
        data,
        timestamp: new Date().toISOString(),
      }
    );
  }

  async publishOrderEvent(eventType, orderId, data) {
    await this.messageQueue.publish(
      'order_events',
      `order.${eventType}`,
      {
        orderId,
        eventType,
        data,
        timestamp: new Date().toISOString(),
      }
    );
  }

  async publishProductEvent(eventType, productId, data) {
    await this.messageQueue.publish(
      'product_events',
      `product.${eventType}`,
      {
        productId,
        eventType,
        data,
        timestamp: new Date().toISOString(),
      }
    );
  }
}

// Event handlers
class EventHandlers {
  constructor(userService, orderService, productService) {
    this.userService = userService;
    this.orderService = orderService;
    this.productService = productService;
  }

  async handleUserRegistered(message) {
    console.log('Handling user registered event:', message);

    // Send welcome email
    await this.userService.sendWelcomeEmail(message.userId);

    // Create user profile
    await this.userService.createUserProfile(message.userId);
  }

  async handleOrderPlaced(message) {
    console.log('Handling order placed event:', message);

    // Update inventory
    await this.productService.updateInventory(message.orderId);

    // Send order confirmation
    await this.orderService.sendOrderConfirmation(message.orderId);

    // Update user statistics
    await this.userService.updateUserStats(message.userId, 'orders_placed', 1);
  }

  async handleProductCreated(message) {
    console.log('Handling product created event:', message);

    // Index product for search
    await this.productService.indexProduct(message.productId);

    // Notify interested users
    await this.productService.notifyInterestedUsers(message.productId);
  }
}

// Initialize message queue
const messageQueue = new MessageQueue();
const eventPublisher = new EventPublisher(messageQueue);
const eventHandlers = new EventHandlers(userService, orderService, productService);

// Connect and setup
await messageQueue.connect('amqp://localhost');

// Subscribe to events
await messageQueue.subscribe(
  'user_events',
  'user_service_queue',
  'user.registered',
  eventHandlers.handleUserRegistered.bind(eventHandlers)
);

await messageQueue.subscribe(
  'order_events',
  'order_service_queue',
  'order.placed',
  eventHandlers.handleOrderPlaced.bind(eventHandlers)
);

await messageQueue.subscribe(
  'product_events',
  'product_service_queue',
  'product.created',
  eventHandlers.handleProductCreated.bind(eventHandlers)
);

export { messageQueue, eventPublisher, eventHandlers };
```

---

## 💾 **Data Management**

### **Database per Service**
```javascript
// User service database
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  id: { type: String, required: true, unique: true },
  email: { type: String, required: true, unique: true },
  name: { type: String, required: true },
  passwordHash: { type: String, required: true },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now },
  profile: {
    avatar: String,
    bio: String,
    location: String,
  },
  preferences: {
    notifications: { type: Boolean, default: true },
    theme: { type: String, default: 'light' },
  },
});

const User = mongoose.model('User', userSchema);

// User service
class UserService {
  constructor() {
    this.connectToDatabase();
  }

  async connectToDatabase() {
    try {
      await mongoose.connect('mongodb://localhost:27017/user_service', {
        useNewUrlParser: true,
        useUnifiedTopology: true,
      });
      console.log('User service connected to database');
    } catch (error) {
      console.error('User service database connection error:', error);
    }
  }

  async createUser(userData) {
    const user = new User(userData);
    return await user.save();
  }

  async findUserById(id) {
    return await User.findOne({ id });
  }

  async findUserByEmail(email) {
    return await User.findOne({ email });
  }

  async updateUser(id, updateData) {
    return await User.findOneAndUpdate(
      { id },
      { ...updateData, updatedAt: new Date() },
      { new: true }
    );
  }

  async deleteUser(id) {
    return await User.findOneAndDelete({ id });
  }
}

// Product service database
const productSchema = new mongoose.Schema({
  id: { type: String, required: true, unique: true },
  name: { type: String, required: true },
  description: String,
  price: { type: Number, required: true },
  category: String,
  inventory: { type: Number, default: 0 },
  images: [String],
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now },
  isActive: { type: Boolean, default: true },
});

const Product = mongoose.model('Product', productSchema);

class ProductService {
  constructor() {
    this.connectToDatabase();
  }

  async connectToDatabase() {
    try {
      await mongoose.connect('mongodb://localhost:27017/product_service', {
        useNewUrlParser: true,
        useUnifiedTopology: true,
      });
      console.log('Product service connected to database');
    } catch (error) {
      console.error('Product service database connection error:', error);
    }
  }

  async createProduct(productData) {
    const product = new Product(productData);
    return await product.save();
  }

  async findProductById(id) {
    return await Product.findOne({ id, isActive: true });
  }

  async findProducts(filters = {}, page = 1, limit = 10) {
    const query = { isActive: true, ...filters };
    const skip = (page - 1) * limit;

    const [products, total] = await Promise.all([
      Product.find(query).skip(skip).limit(limit).sort({ createdAt: -1 }),
      Product.countDocuments(query),
    ]);

    return {
      products,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit),
      },
    };
  }

  async updateProduct(id, updateData) {
    return await Product.findOneAndUpdate(
      { id },
      { ...updateData, updatedAt: new Date() },
      { new: true }
    );
  }

  async updateInventory(id, quantity) {
    return await Product.findOneAndUpdate(
      { id },
      { $inc: { inventory: quantity }, updatedAt: new Date() },
      { new: true }
    );
  }
}
```

### **Saga Pattern for Data Consistency**
```javascript
// Saga orchestrator
class SagaOrchestrator {
  constructor() {
    this.sagas = new Map();
    this.activeSagas = new Map();
  }

  registerSaga(name, sagaDefinition) {
    this.sagas.set(name, sagaDefinition);
  }

  async startSaga(sagaName, data) {
    const sagaDefinition = this.sagas.get(sagaName);
    if (!sagaDefinition) {
      throw new Error(`Saga ${sagaName} not found`);
    }

    const sagaId = generateSagaId();
    const sagaInstance = new SagaInstance(sagaId, sagaDefinition, data);

    this.activeSagas.set(sagaId, sagaInstance);

    try {
      await sagaInstance.execute();
      console.log(`Saga ${sagaName} completed successfully`);
    } catch (error) {
      console.error(`Saga ${sagaName} failed:`, error);
      await sagaInstance.compensate();
    } finally {
      this.activeSagas.delete(sagaId);
    }
  }
}

// Saga instance
class SagaInstance {
  constructor(id, definition, initialData) {
    this.id = id;
    this.definition = definition;
    this.data = initialData;
    this.completedSteps = [];
    this.failedStep = null;
  }

  async execute() {
    for (const step of this.definition.steps) {
      try {
        await step.execute(this.data);
        this.completedSteps.push(step);
      } catch (error) {
        this.failedStep = step;
        throw error;
      }
    }
  }

  async compensate() {
    // Execute compensation in reverse order
    for (let i = this.completedSteps.length - 1; i >= 0; i--) {
      const step = this.completedSteps[i];
      try {
        await step.compensate(this.data);
      } catch (compensationError) {
        console.error(`Compensation failed for step ${step.name}:`, compensationError);
        // Continue with other compensations
      }
    }
  }
}

// Order saga definition
const orderSaga = {
  name: 'create_order',
  steps: [
    {
      name: 'validate_order',
      execute: async (data) => {
        // Validate order data
        if (!data.items || data.items.length === 0) {
          throw new Error('Order must contain at least one item');
        }
        console.log('Order validation passed');
      },
      compensate: async (data) => {
        // No compensation needed for validation
        console.log('Order validation compensation - no action needed');
      },
    },
    {
      name: 'reserve_inventory',
      execute: async (data) => {
        // Reserve inventory
        await inventoryService.reserveItems(data.items);
        console.log('Inventory reserved');
      },
      compensate: async (data) => {
        // Release reserved inventory
        await inventoryService.releaseItems(data.items);
        console.log('Inventory reservation released');
      },
    },
    {
      name: 'process_payment',
      execute: async (data) => {
        // Process payment
        const payment = await paymentService.processPayment(data.paymentMethod, data.total);
        data.paymentId = payment.id;
        console.log('Payment processed');
      },
      compensate: async (data) => {
        // Refund payment if it was processed
        if (data.paymentId) {
          await paymentService.refundPayment(data.paymentId);
          console.log('Payment refunded');
        }
      },
    },
    {
      name: 'create_order_record',
      execute: async (data) => {
        // Create order record
        const order = await orderService.createOrder(data);
        data.orderId = order.id;
        console.log('Order record created');
      },
      compensate: async (data) => {
        // Delete order record
        if (data.orderId) {
          await orderService.deleteOrder(data.orderId);
          console.log('Order record deleted');
        }
      },
    },
    {
      name: 'send_confirmation',
      execute: async (data) => {
        // Send order confirmation
        await emailService.sendOrderConfirmation(data.userId, data.orderId);
        console.log('Order confirmation sent');
      },
      compensate: async (data) => {
        // No compensation needed for email
        console.log('Order confirmation - no compensation needed');
      },
    },
  ],
};

// Initialize saga orchestrator
const sagaOrchestrator = new SagaOrchestrator();
sagaOrchestrator.registerSaga('create_order', orderSaga);

// Usage
app.post('/api/orders', async (req, res) => {
  try {
    await sagaOrchestrator.startSaga('create_order', req.body);
    res.status(201).json({ message: 'Order created successfully' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

---

## 🔍 **Service Discovery**

### **Consul Integration**
```javascript
// Service discovery client
const consul = require('consul');

class ServiceDiscovery {
  constructor() {
    this.consul = consul({
      host: process.env.CONSUL_HOST || 'localhost',
      port: process.env.CONSUL_PORT || 8500,
    });
  }

  async registerService(serviceDefinition) {
    const service = {
      id: serviceDefinition.id,
      name: serviceDefinition.name,
      address: serviceDefinition.address,
      port: serviceDefinition.port,
      tags: serviceDefinition.tags || [],
      checks: [
        {
          http: `http://${serviceDefinition.address}:${serviceDefinition.port}/health`,
          interval: '10s',
          timeout: '5s',
        },
      ],
      ...serviceDefinition,
    };

    await this.consul.agent.service.register(service);
    console.log(`Service ${service.name} registered with Consul`);
  }

  async deregisterService(serviceId) {
    await this.consul.agent.service.deregister(serviceId);
    console.log(`Service ${serviceId} deregistered from Consul`);
  }

  async discoverService(serviceName, options = {}) {
    const services = await this.consul.catalog.service.nodes(serviceName);

    if (services.length === 0) {
      throw new Error(`Service ${serviceName} not found`);
    }

    // Filter healthy services
    const healthyServices = services.filter(service => service.Checks.every(check => check.Status === 'passing'));

    if (healthyServices.length === 0) {
      throw new Error(`No healthy instances of ${serviceName} found`);
    }

    // Simple load balancing - return random healthy service
    const randomIndex = Math.floor(Math.random() * healthyServices.length);
    const selectedService = healthyServices[randomIndex];

    return {
      address: selectedService.Service.Address,
      port: selectedService.Service.Port,
    };
  }

  async getServiceHealth(serviceName) {
    const health = await this.consul.health.service({
      service: serviceName,
      passing: true,
    });

    return health.map(entry => ({
      node: entry.Node.Node,
      service: entry.Service,
      checks: entry.Checks,
    }));
  }

  async watchService(serviceName, callback) {
    const watcher = this.consul.watch({
      method: this.consul.catalog.service.nodes,
      options: { service: serviceName },
    });

    watcher.on('change', (data) => {
      callback(data);
    });

    watcher.on('error', (error) => {
      console.error(`Error watching service ${serviceName}:`, error);
    });

    return watcher;
  }
}

// Service registration
const serviceDiscovery = new ServiceDiscovery();

// Register user service
await serviceDiscovery.registerService({
  id: 'user-service-1',
  name: 'user-service',
  address: 'localhost',
  port: 3001,
  tags: ['api', 'users'],
});

// Register product service
await serviceDiscovery.registerService({
  id: 'product-service-1',
  name: 'product-service',
  address: 'localhost',
  port: 3002,
  tags: ['api', 'products'],
});

// Discover and call service
const userServiceAddress = await serviceDiscovery.discoverService('user-service');
const userServiceUrl = `http://${userServiceAddress.address}:${userServiceAddress.port}`;

const response = await fetch(`${userServiceUrl}/api/v1/users/123`);
const user = await response.json();

export default serviceDiscovery;
```

### **Client-Side Load Balancing**
```javascript
// Load balancer
class LoadBalancer {
  constructor(serviceDiscovery) {
    this.serviceDiscovery = serviceDiscovery;
    this.serviceCache = new Map();
    this.cacheTimeout = 30000; // 30 seconds
  }

  async getServiceAddress(serviceName) {
    // Check cache first
    const cached = this.serviceCache.get(serviceName);
    if (cached && Date.now() - cached.timestamp < this.cacheTimeout) {
      return cached.address;
    }

    // Discover service
    const address = await this.serviceDiscovery.discoverService(serviceName);

    // Cache result
    this.serviceCache.set(serviceName, {
      address,
      timestamp: Date.now(),
    });

    return address;
  }

  async callService(serviceName, endpoint, options = {}) {
    const address = await this.getServiceAddress(serviceName);
    const url = `http://${address.address}:${address.port}${endpoint}`;

    return await this.makeRequest(url, options);
  }

  async makeRequest(url, options) {
    const config = {
      timeout: 5000,
      retries: 3,
      ...options,
    };

    let lastError;

    for (let attempt = 1; attempt <= config.retries; attempt++) {
      try {
        const response = await fetch(url, {
          ...config,
          signal: AbortSignal.timeout(config.timeout),
        });

        if (!response.ok) {
          throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }

        return await response.json();
      } catch (error) {
        lastError = error;
        console.warn(`Request attempt ${attempt} failed:`, error.message);

        if (attempt < config.retries) {
          // Wait before retrying (exponential backoff)
          await new Promise(resolve => setTimeout(resolve, Math.pow(2, attempt) * 1000));
        }
      }
    }

    throw lastError;
  }
}

// HTTP client with load balancing
class LoadBalancedHttpClient {
  constructor(serviceDiscovery) {
    this.loadBalancer = new LoadBalancer(serviceDiscovery);
  }

  async get(serviceName, endpoint, params = {}) {
    const queryString = new URLSearchParams(params).toString();
    const fullEndpoint = queryString ? `${endpoint}?${queryString}` : endpoint;

    return this.loadBalancer.callService(serviceName, fullEndpoint, {
      method: 'GET',
    });
  }

  async post(serviceName, endpoint, data) {
    return this.loadBalancer.callService(serviceName, endpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });
  }

  async put(serviceName, endpoint, data) {
    return this.loadBalancer.callService(serviceName, endpoint, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });
  }

  async delete(serviceName, endpoint) {
    return this.loadBalancer.callService(serviceName, endpoint, {
      method: 'DELETE',
    });
  }
}

// Usage
const httpClient = new LoadBalancedHttpClient(serviceDiscovery);

// Call user service
const user = await httpClient.get('user-service', '/api/v1/users/123');

// Call product service
const products = await httpClient.get('product-service', '/api/v1/products', {
  category: 'electronics',
  limit: 10,
});

export default httpClient;
```

---

## 🔌 **Circuit Breaker Pattern**

### **Circuit Breaker Implementation**
```javascript
// Circuit breaker states
const CIRCUIT_STATES = {
  CLOSED: 'CLOSED',
  OPEN: 'OPEN',
  HALF_OPEN: 'HALF_OPEN',
};

class CircuitBreaker {
  constructor(options = {}) {
    this.failureThreshold = options.failureThreshold || 5;
    this.recoveryTimeout = options.recoveryTimeout || 30000;
    this.monitoringPeriod = options.monitoringPeriod || 10000;

    this.state = CIRCUIT_STATES.CLOSED;
    this.failureCount = 0;
    this.lastFailureTime = null;
    this.successCount = 0;
    this.nextAttemptTime = 0;

    this.requestVolumeThreshold = options.requestVolumeThreshold || 10;
    this.errorPercentageThreshold = options.errorPercentageThreshold || 50;

    this.requestCount = 0;
    this.errorCount = 0;
    this.monitoringStartTime = Date.now();
  }

  async execute(action) {
    this.requestCount++;

    if (this.state === CIRCUIT_STATES.OPEN) {
      if (Date.now() < this.nextAttemptTime) {
        throw new CircuitBreakerError('Circuit breaker is OPEN');
      }

      this.state = CIRCUIT_STATES.HALF_OPEN;
    }

    try {
      const result = await action();

      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    this.successCount++;

    if (this.state === CIRCUIT_STATES.HALF_OPEN) {
      this.state = CIRCUIT_STATES.CLOSED;
      console.log('Circuit breaker CLOSED - service recovered');
    }
  }

  onFailure() {
    this.failureCount++;
    this.errorCount++;
    this.lastFailureTime = Date.now();

    if (this.shouldOpenCircuit()) {
      this.openCircuit();
    }
  }

  shouldOpenCircuit() {
    if (this.state === CIRCUIT_STATES.HALF_OPEN) {
      return true;
    }

    if (this.state === CIRCUIT_STATES.CLOSED) {
      // Check failure count
      if (this.failureCount >= this.failureThreshold) {
        return true;
      }

      // Check error percentage
      const errorPercentage = (this.errorCount / this.requestCount) * 100;
      if (this.requestCount >= this.requestVolumeThreshold &&
          errorPercentage >= this.errorPercentageThreshold) {
        return true;
      }
    }

    return false;
  }

  openCircuit() {
    this.state = CIRCUIT_STATES.OPEN;
    this.nextAttemptTime = Date.now() + this.recoveryTimeout;
    console.log(`Circuit breaker OPEN - will retry at ${new Date(this.nextAttemptTime).toISOString()}`);
  }

  getState() {
    return {
      state: this.state,
      failureCount: this.failureCount,
      successCount: this.successCount,
      requestCount: this.requestCount,
      errorCount: this.errorCount,
      nextAttemptTime: this.nextAttemptTime,
    };
  }

  reset() {
    this.state = CIRCUIT_STATES.CLOSED;
    this.failureCount = 0;
    this.successCount = 0;
    this.requestCount = 0;
    this.errorCount = 0;
    this.monitoringStartTime = Date.now();
  }
}

class CircuitBreakerError extends Error {
  constructor(message) {
    super(message);
    this.name = 'CircuitBreakerError';
  }
}

// Circuit breaker registry
class CircuitBreakerRegistry {
  constructor() {
    this.breakers = new Map();
  }

  getBreaker(name, options = {}) {
    if (!this.breakers.has(name)) {
      this.breakers.set(name, new CircuitBreaker(options));
    }
    return this.breakers.get(name);
  }

  getAllBreakers() {
    const breakers = {};
    for (const [name, breaker] of this.breakers) {
      breakers[name] = breaker.getState();
    }
    return breakers;
  }

  resetBreaker(name) {
    const breaker = this.breakers.get(name);
    if (breaker) {
      breaker.reset();
    }
  }

  resetAllBreakers() {
    for (const breaker of this.breakers.values()) {
      breaker.reset();
    }
  }
}

// Global registry
const circuitBreakerRegistry = new CircuitBreakerRegistry();

// HTTP client with circuit breaker
class ResilientHttpClient {
  constructor(serviceDiscovery) {
    this.serviceDiscovery = serviceDiscovery;
  }

  async callService(serviceName, endpoint, options = {}) {
    const breaker = circuitBreakerRegistry.getBreaker(serviceName, {
      failureThreshold: 3,
      recoveryTimeout: 30000,
    });

    return breaker.execute(async () => {
      const address = await this.serviceDiscovery.discoverService(serviceName);
      const url = `http://${address.address}:${address.port}${endpoint}`;

      const response = await fetch(url, {
        timeout: 5000,
        ...options,
      });

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      return response.json();
    });
  }
}

export {
  CircuitBreaker,
  CircuitBreakerRegistry,
  CircuitBreakerError,
  ResilientHttpClient,
  circuitBreakerRegistry,
};
```

---

## 📊 **Distributed Tracing**

### **OpenTelemetry Integration**
```javascript
// tracing.js
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { SimpleSpanProcessor } = require('@opentelemetry/sdk-trace-base');
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger');
const { trace, context, SpanKind, SpanStatusCode } = require('@opentelemetry/api');

// Initialize tracing
function initializeTracing() {
  const provider = new NodeTracerProvider();

  const exporter = new JaegerExporter({
    endpoint: 'http://localhost:14268/api/traces',
  });

  provider.addSpanProcessor(new SimpleSpanProcessor(exporter));
  provider.register();

  console.log('Distributed tracing initialized');
}

// Create tracer
const tracer = trace.getTracer('microservices-tracer');

// Tracing middleware for Express
function tracingMiddleware(req, res, next) {
  const span = tracer.startSpan(`${req.method} ${req.path}`, {
    kind: SpanKind.SERVER,
    attributes: {
      'http.method': req.method,
      'http.url': req.url,
      'http.target': req.path,
      'http.scheme': req.protocol,
      'http.host': req.get('host'),
      'http.user_agent': req.get('User-Agent'),
    },
  });

  // Set span in context
  const ctx = trace.setSpan(context.active(), span);

  // Add span to request for use in handlers
  req.span = span;

  res.on('finish', () => {
    span.setAttributes({
      'http.status_code': res.statusCode,
      'http.status_text': res.statusMessage,
    });

    if (res.statusCode >= 400) {
      span.setStatus({
        code: SpanStatusCode.ERROR,
        message: res.statusMessage,
      });
    }

    span.end();
  });

  res.on('error', (error) => {
    span.recordException(error);
    span.setStatus({
      code: SpanStatusCode.ERROR,
      message: error.message,
    });
    span.end();
  });

  // Continue with next middleware
  context.with(ctx, next);
}

// Database operation tracing
async function traceDatabaseOperation(operation, collection, query, span) {
  const dbSpan = tracer.startSpan(`db.${operation}`, {
    kind: SpanKind.CLIENT,
    attributes: {
      'db.system': 'mongodb',
      'db.collection': collection,
      'db.operation': operation,
    },
  }, span ? trace.setSpan(context.active(), span) : undefined);

  try {
    const result = await query();
    dbSpan.setAttribute('db.rows_affected', result.modifiedCount || result.insertedCount || 0);
    return result;
  } catch (error) {
    dbSpan.recordException(error);
    dbSpan.setStatus({
      code: SpanStatusCode.ERROR,
      message: error.message,
    });
    throw error;
  } finally {
    dbSpan.end();
  }
}

// Service call tracing
async function traceServiceCall(serviceName, operation, span) {
  const serviceSpan = tracer.startSpan(`service.${serviceName}.${operation}`, {
    kind: SpanKind.CLIENT,
    attributes: {
      'service.name': serviceName,
      'service.operation': operation,
    },
  }, span ? trace.setSpan(context.active(), span) : undefined);

  try {
    const result = await operation();
    serviceSpan.setAttribute('service.success', true);
    return result;
  } catch (error) {
    serviceSpan.recordException(error);
    serviceSpan.setStatus({
      code: SpanStatusCode.ERROR,
      message: error.message,
    });
    serviceSpan.setAttribute('service.success', false);
    throw error;
  } finally {
    serviceSpan.end();
  }
}

// Custom span creation
function createSpan(name, attributes = {}, parentSpan = null) {
  return tracer.startSpan(name, {
    kind: SpanKind.INTERNAL,
    attributes,
  }, parentSpan ? trace.setSpan(context.active(), parentSpan) : undefined);
}

// Usage in services
app.use(tracingMiddleware);

// Example service with tracing
app.get('/api/users/:id', async (req, res) => {
  const span = req.span;

  try {
    // Trace database operation
    const user = await traceDatabaseOperation(
      'find',
      'users',
      () => User.findById(req.params.id),
      span
    );

    if (!user) {
      span.setStatus({
        code: SpanStatusCode.ERROR,
        message: 'User not found',
      });
      return res.status(404).json({ error: 'User not found' });
    }

    // Trace service call
    const userProfile = await traceServiceCall(
      'user-profile',
      'getProfile',
      () => profileService.getUserProfile(user.id),
      span
    );

    span.setAttribute('user.id', user.id);
    span.setAttribute('user.found', true);

    res.json({ user, profile: userProfile });
  } catch (error) {
    span.recordException(error);
    span.setStatus({
      code: SpanStatusCode.ERROR,
      message: error.message,
    });
    res.status(500).json({ error: 'Internal server error' });
  }
});

export {
  initializeTracing,
  tracingMiddleware,
  traceDatabaseOperation,
  traceServiceCall,
  createSpan,
  tracer,
};

---

## 🐳 **Container Orchestration**

### **Docker Compose for Microservices**
```yaml
# docker-compose.yml
version: '3.8'

services:
  # API Gateway
  api-gateway:
    build: ./api-gateway
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - JWT_SECRET=your-jwt-secret
    depends_on:
      - user-service
      - product-service
      - order-service
    networks:
      - microservices-network

  # User Service
  user-service:
    build: ./user-service
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongodb:27017/user_service
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongodb
      - redis
    networks:
      - microservices-network

  # Product Service
  product-service:
    build: ./product-service
    ports:
      - "3002:3002"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongodb:27017/product_service
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongodb
      - redis
    networks:
      - microservices-network

  # Order Service
  order-service:
    build: ./order-service
    ports:
      - "3003:3003"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongodb:27017/order_service
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongodb
      - redis
    networks:
      - microservices-network

  # MongoDB
  mongodb:
    image: mongo:5.0
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    networks:
      - microservices-network

  # Redis
  redis:
    image: redis:7.0-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - microservices-network

  # RabbitMQ
  rabbitmq:
    image: rabbitmq:3-management-alpine
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - microservices-network

  # Jaeger (Distributed Tracing)
  jaeger:
    image: jaegertracing/all-in-one:1.35
    ports:
      - "16686:16686"
      - "14268:14268"
    environment:
      COLLECTOR_OTLP_ENABLED: true
    networks:
      - microservices-network

  # Consul (Service Discovery)
  consul:
    image: consul:1.12
    ports:
      - "8500:8500"
      - "8600:8600"
    command: agent -server -bootstrap -ui -client=0.0.0.0
    networks:
      - microservices-network

volumes:
  mongodb_data:
  redis_data:
  rabbitmq_data:

networks:
  microservices-network:
    driver: bridge
```

### **Kubernetes Deployment**
```yaml
# k8s/user-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  labels:
    app: user-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: myapp/user-service:latest
        ports:
        - containerPort: 3001
        env:
        - name: NODE_ENV
          value: "production"
        - name: MONGODB_URI
          value: "mongodb://mongodb-service:27017/user_service"
        - name: REDIS_URL
          value: "redis://redis-service:6379"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3001
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3001
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: user-service
  labels:
    app: user-service
spec:
  selector:
    app: user-service
  ports:
  - port: 3001
    targetPort: 3001
  type: ClusterIP

---
# k8s/api-gateway-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  labels:
    app: api-gateway
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
      - name: api-gateway
        image: myapp/api-gateway:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: jwt-secret
              key: secret
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway
  labels:
    app: api-gateway
spec:
  selector:
    app: api-gateway
  ports:
  - port: 3000
    targetPort: 3000
  type: LoadBalancer

---
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-gateway-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: api.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-gateway
            port:
              number: 3000
```

### **Service Mesh with Istio**
```yaml
# istio/user-service-destinationrule.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: user-service
spec:
  host: user-service
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 10
        maxRequestsPerConnection: 10
    outlierDetection:
      consecutive5xxErrors: 3
      interval: 10s
      baseEjectionTime: 30s

---
# istio/user-service-virtualservice.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: user-service
spec:
  hosts:
  - user-service
  http:
  - match:
    - uri:
        prefix: "/api/v1/users"
    route:
    - destination:
        host: user-service
        subset: v1
    timeout: 10s
    retries:
      attempts: 3
      perTryTimeout: 2s
  - match:
    - uri:
        prefix: "/api/v2/users"
    route:
    - destination:
        host: user-service
        subset: v2
    timeout: 10s

---
# istio/api-gateway-gateway.yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: api-gateway-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "api.myapp.com"

---
# istio/api-gateway-virtualservice.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: api-gateway
spec:
  hosts:
  - "api.myapp.com"
  gateways:
  - api-gateway-gateway
  http:
  - match:
    - uri:
        prefix: "/"
    route:
    - destination:
        host: api-gateway
    timeout: 30s
    corsPolicy:
      allowOrigin:
      - "*"
      allowMethods:
      - GET
      - POST
      - PUT
      - DELETE
      - OPTIONS
      allowHeaders:
      - "*"
```

---

## 🎯 **Practical Examples**

### **Complete E-commerce Microservices System**
```javascript
// services/order-service/index.js
const express = require('express');
const mongoose = require('mongoose');
const amqp = require('amqplib');
const { traceDatabaseOperation, traceServiceCall } = require('../tracing');

const app = express();
app.use(express.json());

// Order Schema
const orderSchema = new mongoose.Schema({
  id: { type: String, required: true, unique: true },
  userId: { type: String, required: true },
  items: [{
    productId: String,
    name: String,
    price: Number,
    quantity: Number,
  }],
  total: { type: Number, required: true },
  status: {
    type: String,
    enum: ['pending', 'confirmed', 'shipped', 'delivered', 'cancelled'],
    default: 'pending'
  },
  shippingAddress: {
    street: String,
    city: String,
    state: String,
    zipCode: String,
    country: String,
  },
  paymentId: String,
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now },
});

const Order = mongoose.model('Order', orderSchema);

// Message Queue Connection
let channel;
async function connectQueue() {
  try {
    const connection = await amqp.connect('amqp://rabbitmq:5672');
    channel = await connection.createChannel();

    // Assert exchanges
    await channel.assertExchange('order_events', 'topic', { durable: true });
    await channel.assertExchange('inventory_events', 'topic', { durable: true });

    console.log('Order service connected to message queue');
  } catch (error) {
    console.error('Failed to connect to message queue:', error);
  }
}

// Publish event
function publishEvent(exchange, routingKey, data) {
  if (channel) {
    channel.publish(exchange, routingKey, Buffer.from(JSON.stringify(data)));
  }
}

// API Routes
app.post('/api/v1/orders', async (req, res) => {
  const span = req.span;

  try {
    const { userId, items, shippingAddress } = req.body;

    // Calculate total
    const total = items.reduce((sum, item) => sum + (item.price * item.quantity), 0);

    // Create order
    const order = await traceDatabaseOperation(
      'create',
      'orders',
      () => Order.create({
        id: generateOrderId(),
        userId,
        items,
        total,
        shippingAddress,
      }),
      span
    );

    // Publish order created event
    publishEvent('order_events', 'order.created', {
      orderId: order.id,
      userId: order.userId,
      total: order.total,
      items: order.items,
    });

    // Reserve inventory
    publishEvent('inventory_events', 'inventory.reserve', {
      orderId: order.id,
      items: order.items,
    });

    res.status(201).json({
      success: true,
      data: order,
    });
  } catch (error) {
    span.recordException(error);
    res.status(500).json({
      success: false,
      error: error.message,
    });
  }
});

app.get('/api/v1/orders/:id', async (req, res) => {
  const span = req.span;

  try {
    const order = await traceDatabaseOperation(
      'find',
      'orders',
      () => Order.findOne({ id: req.params.id }),
      span
    );

    if (!order) {
      return res.status(404).json({
        success: false,
        error: 'Order not found',
      });
    }

    res.json({
      success: true,
      data: order,
    });
  } catch (error) {
    span.recordException(error);
    res.status(500).json({
      success: false,
      error: error.message,
    });
  }
});

app.put('/api/v1/orders/:id/status', async (req, res) => {
  const span = req.span;

  try {
    const { status } = req.body;

    const order = await traceDatabaseOperation(
      'update',
      'orders',
      () => Order.findOneAndUpdate(
        { id: req.params.id },
        { status, updatedAt: new Date() },
        { new: true }
      ),
      span
    );

    if (!order) {
      return res.status(404).json({
        success: false,
        error: 'Order not found',
      });
    }

    // Publish status update event
    publishEvent('order_events', `order.${status}`, {
      orderId: order.id,
      userId: order.userId,
      status,
    });

    res.json({
      success: true,
      data: order,
    });
  } catch (error) {
    span.recordException(error);
    res.status(500).json({
      success: false,
      error: error.message,
    });
  }
});

// Health check
app.get('/health', (req, res) => {
  res.json({
    status: 'ok',
    service: 'order-service',
    timestamp: new Date().toISOString(),
  });
});

// Generate order ID
function generateOrderId() {
  return 'ORD-' + Date.now() + '-' + Math.random().toString(36).substr(2, 5).toUpperCase();
}

// Connect to database and queue
async function initializeService() {
  try {
    // Connect to MongoDB
    await mongoose.connect('mongodb://mongodb:27017/order_service', {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log('Order service connected to database');

    // Connect to message queue
    await connectQueue();

    // Start server
    const PORT = process.env.PORT || 3003;
    app.listen(PORT, () => {
      console.log(`Order service listening on port ${PORT}`);
    });
  } catch (error) {
    console.error('Failed to initialize order service:', error);
    process.exit(1);
  }
}

initializeService();

module.exports = app;
```

### **Event-Driven Inventory Service**
```javascript
// services/inventory-service/index.js
const express = require('express');
const mongoose = require('mongoose');
const amqp = require('amqplib');

const app = express();
app.use(express.json());

// Product Inventory Schema
const inventorySchema = new mongoose.Schema({
  productId: { type: String, required: true, unique: true },
  name: { type: String, required: true },
  sku: { type: String, required: true, unique: true },
  quantity: { type: Number, required: true, default: 0 },
  reserved: { type: Number, default: 0 },
  available: { type: Number, default: 0 },
  minStockLevel: { type: Number, default: 10 },
  maxStockLevel: { type: Number, default: 1000 },
  location: {
    warehouse: String,
    aisle: String,
    shelf: String,
  },
  lastUpdated: { type: Date, default: Date.now },
});

inventorySchema.virtual('available').get(function() {
  return this.quantity - this.reserved;
});

const Inventory = mongoose.model('Inventory', inventorySchema);

// Message Queue Connection
let channel;
async function connectQueue() {
  try {
    const connection = await amqp.connect('amqp://rabbitmq:5672');
    channel = await connection.createChannel();

    // Assert exchanges
    await channel.assertExchange('inventory_events', 'topic', { durable: true });
    await channel.assertExchange('order_events', 'topic', { durable: true });

    // Assert queues
    await channel.assertQueue('inventory_queue', { durable: true });
    await channel.assertQueue('order_inventory_queue', { durable: true });

    // Bind queues to exchanges
    await channel.bindQueue('inventory_queue', 'inventory_events', '#');
    await channel.bindQueue('order_inventory_queue', 'order_events', 'order.created');

    // Setup consumers
    channel.consume('inventory_queue', handleInventoryEvent);
    channel.consume('order_inventory_queue', handleOrderEvent);

    console.log('Inventory service connected to message queue');
  } catch (error) {
    console.error('Failed to connect to message queue:', error);
  }
}

// Handle inventory events
async function handleInventoryEvent(msg) {
  try {
    const event = JSON.parse(msg.content.toString());
    console.log('Processing inventory event:', event.type);

    switch (event.type) {
      case 'stock_update':
        await updateStock(event.productId, event.quantity, event.operation);
        break;
      case 'low_stock_alert':
        await handleLowStockAlert(event.productId);
        break;
    }

    channel.ack(msg);
  } catch (error) {
    console.error('Error processing inventory event:', error);
    channel.nack(msg, false, false); // Don't requeue
  }
}

// Handle order events
async function handleOrderEvent(msg) {
  try {
    const event = JSON.parse(msg.content.toString());
    console.log('Processing order event:', event.type);

    if (event.type === 'created') {
      await reserveInventory(event.orderId, event.items);
    }

    channel.ack(msg);
  } catch (error) {
    console.error('Error processing order event:', error);
    channel.nack(msg, false, false);
  }
}

// Reserve inventory for order
async function reserveInventory(orderId, items) {
  try {
    for (const item of items) {
      const inventory = await Inventory.findOne({ productId: item.productId });

      if (!inventory) {
        throw new Error(`Product ${item.productId} not found in inventory`);
      }

      if (inventory.available < item.quantity) {
        throw new Error(`Insufficient inventory for product ${item.productId}`);
      }

      // Reserve items
      inventory.reserved += item.quantity;
      await inventory.save();

      // Publish reservation event
      publishEvent('inventory_events', 'inventory.reserved', {
        orderId,
        productId: item.productId,
        quantity: item.quantity,
        reserved: inventory.reserved,
        available: inventory.available,
      });
    }
  } catch (error) {
    console.error('Error reserving inventory:', error);

    // Publish reservation failed event
    publishEvent('inventory_events', 'inventory.reservation_failed', {
      orderId,
      error: error.message,
    });

    throw error;
  }
}

// Update stock
async function updateStock(productId, quantity, operation = 'add') {
  const inventory = await Inventory.findOne({ productId });

  if (!inventory) {
    throw new Error(`Product ${productId} not found in inventory`);
  }

  if (operation === 'add') {
    inventory.quantity += quantity;
  } else if (operation === 'subtract') {
    inventory.quantity = Math.max(0, inventory.quantity - quantity);
  }

  inventory.lastUpdated = new Date();
  await inventory.save();

  // Check for low stock
  if (inventory.available <= inventory.minStockLevel) {
    publishEvent('inventory_events', 'inventory.low_stock', {
      productId,
      available: inventory.available,
      minStockLevel: inventory.minStockLevel,
    });
  }

  return inventory;
}

// Handle low stock alert
async function handleLowStockAlert(productId) {
  const inventory = await Inventory.findOne({ productId });

  // Send alert to admin service
  publishEvent('admin_events', 'alert.low_stock', {
    productId,
    productName: inventory.name,
    available: inventory.available,
    minStockLevel: inventory.minStockLevel,
  });
}

// Publish event
function publishEvent(exchange, routingKey, data) {
  if (channel) {
    channel.publish(exchange, routingKey, Buffer.from(JSON.stringify({
      ...data,
      timestamp: new Date().toISOString(),
    })));
  }
}

// API Routes
app.get('/api/v1/inventory/:productId', async (req, res) => {
  try {
    const inventory = await Inventory.findOne({ productId: req.params.productId });

    if (!inventory) {
      return res.status(404).json({
        success: false,
        error: 'Product not found in inventory',
      });
    }

    res.json({
      success: true,
      data: {
        productId: inventory.productId,
        name: inventory.name,
        quantity: inventory.quantity,
        reserved: inventory.reserved,
        available: inventory.available,
        minStockLevel: inventory.minStockLevel,
        location: inventory.location,
        lastUpdated: inventory.lastUpdated,
      },
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message,
    });
  }
});

app.post('/api/v1/inventory/:productId/adjust', async (req, res) => {
  try {
    const { quantity, operation } = req.body;

    const inventory = await updateStock(req.params.productId, quantity, operation);

    res.json({
      success: true,
      data: inventory,
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      error: error.message,
    });
  }
});

// Health check
app.get('/health', (req, res) => {
  res.json({
    status: 'ok',
    service: 'inventory-service',
    timestamp: new Date().toISOString(),
  });
});

// Initialize service
async function initializeService() {
  try {
    // Connect to MongoDB
    await mongoose.connect('mongodb://mongodb:27017/inventory_service', {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    console.log('Inventory service connected to database');

    // Connect to message queue
    await connectQueue();

    // Start server
    const PORT = process.env.PORT || 3004;
    app.listen(PORT, () => {
      console.log(`Inventory service listening on port ${PORT}`);
    });
  } catch (error) {
    console.error('Failed to initialize inventory service:', error);
    process.exit(1);
  }
}

initializeService();

module.exports = app;
```

### **Notification Service with WebSocket**
```javascript
// services/notification-service/index.js
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const amqp = require('amqplib');
const jwt = require('jsonwebtoken');

const app = express();
const server = http.createServer(app);
const io = socketIo(server, {
  cors: {
    origin: "*",
    methods: ["GET", "POST"]
  }
});

// Connected clients
const connectedClients = new Map();

// Message Queue Connection
let channel;
async function connectQueue() {
  try {
    const connection = await amqp.connect('amqp://rabbitmq:5672');
    channel = await connection.createChannel();

    // Assert exchanges
    await channel.assertExchange('user_events', 'topic', { durable: true });
    await channel.assertExchange('order_events', 'topic', { durable: true });
    await channel.assertExchange('product_events', 'topic', { durable: true });
    await channel.assertExchange('admin_events', 'topic', { durable: true });

    // Assert queue
    await channel.assertQueue('notification_queue', { durable: true });

    // Bind to all relevant events
    await channel.bindQueue('notification_queue', 'user_events', '#');
    await channel.bindQueue('notification_queue', 'order_events', '#');
    await channel.bindQueue('notification_queue', 'product_events', '#');
    await channel.bindQueue('notification_queue', 'admin_events', '#');

    // Setup consumer
    channel.consume('notification_queue', handleNotificationEvent);

    console.log('Notification service connected to message queue');
  } catch (error) {
    console.error('Failed to connect to message queue:', error);
  }
}

// Handle notification events
async function handleNotificationEvent(msg) {
  try {
    const event = JSON.parse(msg.content.toString());
    console.log('Processing notification event:', event);

    // Create notification
    const notification = await createNotification(event);

    // Send to relevant users
    await sendNotificationToUsers(notification);

    channel.ack(msg);
  } catch (error) {
    console.error('Error processing notification event:', error);
    channel.nack(msg, false, false);
  }
}

// Create notification from event
async function createNotification(event) {
  const notification = {
    id: generateNotificationId(),
    type: determineNotificationType(event),
    title: generateNotificationTitle(event),
    message: generateNotificationMessage(event),
    data: event,
    recipients: await determineRecipients(event),
    createdAt: new Date(),
  };

  // Store notification (in a real app, you'd save to database)
  console.log('Created notification:', notification);

  return notification;
}

// Determine notification type
function determineNotificationType(event) {
  if (event.type?.includes('order')) return 'order';
  if (event.type?.includes('user')) return 'user';
  if (event.type?.includes('product')) return 'product';
  if (event.type?.includes('admin')) return 'admin';
  return 'general';
}

// Generate notification title
function generateNotificationTitle(event) {
  switch (event.type) {
    case 'order.created':
      return 'Order Placed Successfully';
    case 'order.shipped':
      return 'Order Shipped';
    case 'order.delivered':
      return 'Order Delivered';
    case 'user.registered':
      return 'Welcome to Our Platform';
    case 'product.created':
      return 'New Product Available';
    case 'alert.low_stock':
      return 'Low Stock Alert';
    default:
      return 'Notification';
  }
}

// Generate notification message
function generateNotificationMessage(event) {
  switch (event.type) {
    case 'order.created':
      return `Your order #${event.orderId} has been placed successfully.`;
    case 'order.shipped':
      return `Your order #${event.orderId} has been shipped.`;
    case 'order.delivered':
      return `Your order #${event.orderId} has been delivered.`;
    case 'user.registered':
      return 'Thank you for joining our platform!';
    case 'product.created':
      return `Check out our new product: ${event.productName}`;
    case 'alert.low_stock':
      return `Low stock alert for ${event.productName}`;
    default:
      return 'You have a new notification.';
  }
}

// Determine notification recipients
async function determineRecipients(event) {
  const recipients = [];

  // Add specific user if available
  if (event.userId) {
    recipients.push(event.userId);
  }

  // Add admin users for admin events
  if (event.type?.includes('admin') || event.type?.includes('alert')) {
    recipients.push(...await getAdminUserIds());
  }

  return recipients;
}

// Get admin user IDs (in a real app, this would query the database)
async function getAdminUserIds() {
  return ['admin1', 'admin2']; // Mock admin IDs
}

// Send notification to users
async function sendNotificationToUsers(notification) {
  for (const userId of notification.recipients) {
    const client = connectedClients.get(userId);
    if (client) {
      client.socket.emit('notification', notification);
    }

    // Also send push notification if user is offline
    await sendPushNotification(userId, notification);
  }
}

// Send push notification
async function sendPushNotification(userId, notification) {
  // In a real app, you'd integrate with FCM, APNS, etc.
  console.log(`Sending push notification to ${userId}:`, notification.title);
}

// WebSocket connection handling
io.use(async (socket, next) => {
  try {
    const token = socket.handshake.auth.token;
    if (!token) {
      return next(new Error('Authentication token required'));
    }

    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    socket.userId = decoded.id;
    next();
  } catch (error) {
    next(new Error('Invalid authentication token'));
  }
});

io.on('connection', (socket) => {
  const userId = socket.userId;
  console.log(`User ${userId} connected for notifications`);

  // Store client connection
  connectedClients.set(userId, {
    socket,
    connectedAt: new Date(),
  });

  // Send welcome notification
  socket.emit('notification', {
    id: generateNotificationId(),
    type: 'system',
    title: 'Connected',
    message: 'You are now connected to notifications.',
    createdAt: new Date(),
  });

  socket.on('disconnect', () => {
    console.log(`User ${userId} disconnected from notifications`);
    connectedClients.delete(userId);
  });

  socket.on('mark_as_read', (notificationId) => {
    // Mark notification as read (in a real app, update database)
    console.log(`Notification ${notificationId} marked as read by ${userId}`);
  });
});

// API Routes
app.get('/api/v1/notifications/:userId', (req, res) => {
  // In a real app, fetch notifications from database
  res.json({
    success: true,
    data: [],
  });
});

app.post('/api/v1/notifications/:userId/mark-read', (req, res) => {
  const { notificationIds } = req.body;

  // In a real app, update notifications in database
  console.log(`Marking notifications as read for user ${req.params.userId}:`, notificationIds);

  res.json({
    success: true,
    message: 'Notifications marked as read',
  });
});

// Health check
app.get('/health', (req, res) => {
  res.json({
    status: 'ok',
    service: 'notification-service',
    connectedClients: connectedClients.size,
    timestamp: new Date().toISOString(),
  });
});

// Generate notification ID
function generateNotificationId() {
  return 'NOTIF-' + Date.now() + '-' + Math.random().toString(36).substr(2, 5).toUpperCase();
}

// Initialize service
async function initializeService() {
  try {
    // Connect to message queue
    await connectQueue();

    // Start server
    const PORT = process.env.PORT || 3005;
    server.listen(PORT, () => {
      console.log(`Notification service listening on port ${PORT}`);
    });
  } catch (error) {
    console.error('Failed to initialize notification service:', error);
    process.exit(1);
  }
}

initializeService();

module.exports = app;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Microservices Fundamentals**: Architecture principles and characteristics
- ✅ **Service Design Principles**: Single responsibility, domain-driven design, bounded contexts
- ✅ **API Gateway Pattern**: Request routing, authentication, rate limiting, circuit breakers
- ✅ **Service Communication**: Synchronous HTTP and asynchronous message queues
- ✅ **Data Management**: Database per service and saga pattern for consistency
- ✅ **Service Discovery**: Consul integration and client-side load balancing
- ✅ **Circuit Breaker Pattern**: Failure handling and service resilience
- ✅ **Distributed Tracing**: OpenTelemetry integration and request tracking
- ✅ **Container Orchestration**: Docker Compose and Kubernetes deployments
- ✅ **Practical Implementation**: Complete e-commerce system with order, inventory, and notification services

### **Best Practices**
1. **Design for failure**: Implement circuit breakers, retries, and fallbacks
2. **Use domain-driven design**: Organize services around business domains
3. **Implement proper logging and monitoring**: Track requests across services
4. **Use asynchronous communication**: Decouple services with message queues
5. **Implement service discovery**: Automatically find and connect to services
6. **Use API gateways**: Centralize cross-cutting concerns
7. **Implement distributed tracing**: Track requests across service boundaries
8. **Use container orchestration**: Automate deployment and scaling
9. **Implement health checks**: Monitor service health and availability
10. **Use event-driven architecture**: Loose coupling with events and messages

### **Next Steps**
- Learn about service mesh technologies (Istio, Linkerd)
- Study event sourcing and CQRS patterns
- Explore serverless microservices with AWS Lambda
- Learn about API composition and schema stitching
- Study distributed caching strategies (Redis Cluster)
- Implement blue-green and canary deployments
- Learn about chaos engineering and resilience testing
- Study microservices security patterns
- Explore reactive microservices with Spring WebFlux
- Learn about distributed transactions and eventual consistency

---

## 🎯 **Assignment**

### **Task 1: Complete E-commerce Microservices**
Build a comprehensive e-commerce microservices system with:
- User service with authentication and profiles
- Product catalog service with search and filtering
- Shopping cart service with persistence
- Order processing service with saga orchestration
- Payment service with multiple providers
- Inventory management with real-time updates
- Notification service with email and push notifications
- Review and rating system
- Admin dashboard for analytics and management

### **Task 2: Social Media Platform**
Create a social media microservices architecture including:
- User management with followers/following
- Post service with media uploads
- Feed generation with personalized content
- Like and comment system with real-time updates
- Direct messaging with WebSocket support
- Notification service with push notifications
- Content moderation and spam detection
- Analytics and insights dashboard
- Search and discovery features

### **Task 3: IoT Device Management System**
Implement an IoT device management platform with:
- Device registration and authentication
- Real-time telemetry data collection
- Command and control service for device management
- Firmware update service with versioning
- Alert and monitoring system for device health
- Data analytics service for sensor data
- API gateway with rate limiting and security
- Device grouping and bulk operations
- Historical data storage and querying

### **Task 4: Container Orchestration Setup**
Create a complete container orchestration setup with:
- Docker Compose for local development
- Kubernetes manifests for production deployment
- Service mesh configuration with Istio
- CI/CD pipeline with automated testing and deployment
- Monitoring and logging stack (ELK or EFK)
- Distributed tracing with Jaeger
- Load balancing and ingress configuration
- Secrets management and configuration
- Auto-scaling and resource management

---

## 📚 **Additional Resources**
- [Microservices Architecture Guide](https://microservices.io/)
- [Domain-Driven Design](https://dddcommunity.org/)
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Istio Service Mesh](https://istio.io/)
- [RabbitMQ Documentation](https://www.rabbitmq.com/)
- [Consul Documentation](https://www.consul.io/)
- [OpenTelemetry](https://opentelemetry.io/)

---

**Next Lesson**: [Lesson 43: Containerization & Orchestration](Lesson%2043_%20Containerization%20&%20Orchestration.md)
