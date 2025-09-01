# Lesson 23: RESTful APIs & GraphQL

## 🎯 **Learning Objectives**
- Master RESTful API design principles and best practices
- Implement GraphQL APIs with Apollo Server
- Understand the differences between REST and GraphQL
- Create efficient API endpoints with proper error handling
- Implement API versioning and documentation

## 📚 **Table of Contents**
1. [RESTful API Design Principles](#restful-api-design-principles)
2. [HTTP Methods and Status Codes](#http-methods-and-status-codes)
3. [API Versioning Strategies](#api-versioning-strategies)
4. [Introduction to GraphQL](#introduction-to-graphql)
5. [GraphQL Schema Design](#graphql-schema-design)
6. [GraphQL Resolvers](#graphql-resolvers)
7. [REST vs GraphQL Comparison](#rest-vs-graphql-comparison)
8. [API Documentation](#api-documentation)
9. [API Security](#api-security)
10. [Practical Examples](#practical-examples)

---

## 📋 **RESTful API Design Principles**

### **REST Principles**
```javascript
// RESTful API structure
const express = require('express');
const router = express.Router();

// Resource: /api/users

// 1. GET /api/users - List all users (READ)
router.get('/users', async (req, res) => {
  try {
    const { page = 1, limit = 10, search, role } = req.query;

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

    // Execute query
    const users = await User.find(query)
      .select('-password') // Exclude sensitive data
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
});

// 2. GET /api/users/:id - Get single user (READ)
router.get('/users/:id', async (req, res) => {
  try {
    const user = await User.findById(req.params.id).select('-password');

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
    if (error.name === 'CastError') {
      return res.status(400).json({
        success: false,
        error: 'Invalid user ID',
      });
    }

    res.status(500).json({
      success: false,
      error: 'Failed to fetch user',
    });
  }
});

// 3. POST /api/users - Create new user (CREATE)
router.post('/users', async (req, res) => {
  try {
    const { firstName, lastName, email, password, role } = req.body;

    // Check if user already exists
    const existingUser = await User.findOne({ email: email.toLowerCase() });
    if (existingUser) {
      return res.status(409).json({
        success: false,
        error: 'User with this email already exists',
      });
    }

    // Create user
    const user = await User.create({
      firstName,
      lastName,
      email: email.toLowerCase(),
      password,
      role: role || 'user',
    });

    // Remove password from response
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

// 4. PUT /api/users/:id - Update user (UPDATE)
router.put('/users/:id', async (req, res) => {
  try {
    const { firstName, lastName, email, role } = req.body;

    const user = await User.findById(req.params.id);

    if (!user) {
      return res.status(404).json({
        success: false,
        error: 'User not found',
      });
    }

    // Update fields
    if (firstName) user.firstName = firstName;
    if (lastName) user.lastName = lastName;
    if (email) user.email = email.toLowerCase();
    if (role) user.role = role;

    await user.save();

    // Remove password from response
    const userResponse = user.toObject();
    delete userResponse.password;

    res.json({
      success: true,
      data: userResponse,
      message: 'User updated successfully',
    });
  } catch (error) {
    if (error.name === 'CastError') {
      return res.status(400).json({
        success: false,
        error: 'Invalid user ID',
      });
    }

    res.status(500).json({
      success: false,
      error: 'Failed to update user',
    });
  }
});

// 5. DELETE /api/users/:id - Delete user (DELETE)
router.delete('/users/:id', async (req, res) => {
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
    if (error.name === 'CastError') {
      return res.status(400).json({
        success: false,
        error: 'Invalid user ID',
      });
    }

    res.status(500).json({
      success: false,
      error: 'Failed to delete user',
    });
  }
});

module.exports = router;
```

### **Resource Naming Conventions**
```javascript
// Good resource naming
GET    /api/users          // List users
GET    /api/users/:id      // Get user by ID
POST   /api/users          // Create user
PUT    /api/users/:id      // Update user
DELETE /api/users/:id      // Delete user

// Nested resources
GET    /api/users/:id/posts     // Get user's posts
POST   /api/users/:id/posts     // Create post for user
GET    /api/posts/:id/comments  // Get post's comments

// Actions on resources
POST   /api/users/:id/follow    // Follow user
DELETE /api/users/:id/follow    // Unfollow user
POST   /api/posts/:id/like      // Like post
DELETE /api/posts/:id/like      // Unlike post

// Filtering and searching
GET    /api/posts?category=tech&author=john
GET    /api/users?search=john&page=1&limit=10

// Sorting
GET    /api/posts?sort=createdAt:desc
GET    /api/users?sort=lastName:asc,firstName:asc
```

---

## 🌐 **HTTP Methods and Status Codes**

### **HTTP Methods**
```javascript
// Method-specific middleware
const methodOverride = (req, res, next) => {
  // Support for clients that don't support all HTTP methods
  if (req.body && req.body._method) {
    req.method = req.body._method.toUpperCase();
  }
  next();
};

// Content-Type based routing
const contentTypeRouter = (req, res, next) => {
  const contentType = req.headers['content-type'];

  if (req.method === 'POST' || req.method === 'PUT') {
    if (!contentType || !contentType.includes('application/json')) {
      return res.status(400).json({
        success: false,
        error: 'Content-Type must be application/json',
      });
    }
  }

  next();
};

// Method validation middleware
const validateMethod = (allowedMethods) => {
  return (req, res, next) => {
    if (!allowedMethods.includes(req.method)) {
      res.set('Allow', allowedMethods.join(', '));
      return res.status(405).json({
        success: false,
        error: `Method ${req.method} not allowed`,
      });
    }
    next();
  };
};

// Usage
router.use('/users', validateMethod(['GET', 'POST']));
router.use('/users/:id', validateMethod(['GET', 'PUT', 'DELETE']));
```

### **HTTP Status Codes**
```javascript
// Status code utility
const statusCodes = {
  // 2xx Success
  OK: 200,
  CREATED: 201,
  ACCEPTED: 202,
  NO_CONTENT: 204,

  // 3xx Redirection
  MOVED_PERMANENTLY: 301,
  FOUND: 302,
  NOT_MODIFIED: 304,

  // 4xx Client Error
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  FORBIDDEN: 403,
  NOT_FOUND: 404,
  METHOD_NOT_ALLOWED: 405,
  CONFLICT: 409,
  UNPROCESSABLE_ENTITY: 422,
  TOO_MANY_REQUESTS: 429,

  // 5xx Server Error
  INTERNAL_SERVER_ERROR: 500,
  NOT_IMPLEMENTED: 501,
  BAD_GATEWAY: 502,
  SERVICE_UNAVAILABLE: 503,
  GATEWAY_TIMEOUT: 504,
};

// Response helper
const sendResponse = (res, statusCode, data = null, message = null) => {
  const response = { success: statusCode < 400 };

  if (data !== null) response.data = data;
  if (message) response.message = message;
  if (!response.success) response.error = message || 'An error occurred';

  return res.status(statusCode).json(response);
};

// Usage in routes
app.get('/api/users', async (req, res) => {
  try {
    const users = await User.find();
    sendResponse(res, statusCodes.OK, users);
  } catch (error) {
    sendResponse(res, statusCodes.INTERNAL_SERVER_ERROR, null, 'Failed to fetch users');
  }
});

app.post('/api/users', async (req, res) => {
  try {
    const user = await User.create(req.body);
    sendResponse(res, statusCodes.CREATED, user, 'User created successfully');
  } catch (error) {
    if (error.name === 'ValidationError') {
      sendResponse(res, statusCodes.BAD_REQUEST, null, 'Validation failed');
    } else {
      sendResponse(res, statusCodes.INTERNAL_SERVER_ERROR, null, 'Failed to create user');
    }
  }
});
```

---

## 📈 **API Versioning Strategies**

### **URL Path Versioning**
```javascript
// Version 1 routes
const v1Router = express.Router();

// V1 API routes
v1Router.get('/users', getUsersV1);
v1Router.post('/users', createUserV1);

// Mount V1 routes
app.use('/api/v1', v1Router);

// Version 2 routes
const v2Router = express.Router();

// V2 API routes with breaking changes
v2Router.get('/users', getUsersV2);
v2Router.post('/users', createUserV2);

// Mount V2 routes
app.use('/api/v2', v2Router);

// Backward compatibility
app.use('/api', v1Router); // Default to V1

// Version detection middleware
const versionMiddleware = (req, res, next) => {
  const version = req.headers['api-version'] ||
                  req.headers['accept-version'] ||
                  req.query.version ||
                  'v1';

  req.apiVersion = version;
  next();
};

app.use(versionMiddleware);
```

### **Header-Based Versioning**
```javascript
// Accept-Version header versioning
const versionRouter = (req, res, next) => {
  const version = req.headers['accept-version'] ||
                  req.headers['api-version'] ||
                  'v1';

  // Route to appropriate version handler
  switch (version) {
    case 'v2':
      return handleV2Request(req, res, next);
    case 'v1':
    default:
      return handleV1Request(req, res, next);
  }
};

// Custom version negotiation
const negotiateVersion = (supportedVersions, requestedVersion) => {
  // Exact match
  if (supportedVersions.includes(requestedVersion)) {
    return requestedVersion;
  }

  // Semantic versioning
  const [major] = requestedVersion.split('.');
  const compatibleVersion = supportedVersions.find(v =>
    v.startsWith(`${major}.`)
  );

  return compatibleVersion || supportedVersions[0];
};
```

### **Content Negotiation**
```javascript
// Content-Type based versioning
const contentNegotiation = (req, res, next) => {
  const accept = req.headers.accept || '';
  const contentType = req.headers['content-type'] || '';

  // API versioning through content type
  if (accept.includes('application/vnd.api.v2+json')) {
    req.apiVersion = 'v2';
  } else if (accept.includes('application/vnd.api.v1+json')) {
    req.apiVersion = 'v1';
  } else {
    req.apiVersion = 'v1'; // Default
  }

  // Set response content type
  res.set('Content-Type', `application/vnd.api.${req.apiVersion}+json`);
  next();
};

// Usage
app.use('/api', contentNegotiation);

// Routes
app.get('/api/users', (req, res) => {
  if (req.apiVersion === 'v2') {
    // V2 response format
    res.json({ data: users, version: 'v2' });
  } else {
    // V1 response format
    res.json(users);
  }
});
```

---

## 🚀 **Introduction to GraphQL**

### **GraphQL Server Setup**
```javascript
const { ApolloServer, gql } = require('apollo-server-express');
const express = require('express');

const app = express();

// GraphQL Schema
const typeDefs = gql`
  type User {
    id: ID!
    firstName: String!
    lastName: String!
    email: String!
    avatar: String
    posts: [Post!]!
    createdAt: String!
  }

  type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
    category: Category!
    tags: [Tag!]!
    likes: [Like!]!
    comments: [Comment!]!
    createdAt: String!
  }

  type Category {
    id: ID!
    name: String!
    slug: String!
    description: String
  }

  type Tag {
    id: ID!
    name: String!
    slug: String!
  }

  type Like {
    id: ID!
    user: User!
    createdAt: String!
  }

  type Comment {
    id: ID!
    content: String!
    author: User!
    post: Post!
    createdAt: String!
  }

  type Query {
    users(limit: Int, offset: Int): [User!]!
    user(id: ID!): User
    posts(limit: Int, offset: Int, category: ID, author: ID): [Post!]!
    post(id: ID!): Post
    categories: [Category!]!
    tags: [Tag!]!
  }

  type Mutation {
    createUser(input: CreateUserInput!): User!
    updateUser(id: ID!, input: UpdateUserInput!): User!
    deleteUser(id: ID!): Boolean!

    createPost(input: CreatePostInput!): Post!
    updatePost(id: ID!, input: UpdatePostInput!): Post!
    deletePost(id: ID!): Boolean!

    likePost(postId: ID!): Like!
    unlikePost(postId: ID!): Boolean!

    createComment(postId: ID!, content: String!): Comment!
  }

  input CreateUserInput {
    firstName: String!
    lastName: String!
    email: String!
    password: String!
  }

  input UpdateUserInput {
    firstName: String
    lastName: String
    email: String
  }

  input CreatePostInput {
    title: String!
    content: String!
    categoryId: ID!
    tagIds: [ID!]
  }

  input UpdatePostInput {
    title: String
    content: String
    categoryId: ID
    tagIds: [ID!]
  }
`;

// Apollo Server setup
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    // Add authentication context
    return {
      user: req.user,
      models: {
        User,
        Post,
        Category,
        Tag,
      },
    };
  },
  introspection: process.env.NODE_ENV !== 'production',
  playground: process.env.NODE_ENV !== 'production',
});

// Apply middleware
server.applyMiddleware({ app });

// Start server
const PORT = process.env.PORT || 4000;
app.listen(PORT, () => {
  console.log(`🚀 Server ready at http://localhost:${PORT}${server.graphqlPath}`);
});
```

---

## 📋 **GraphQL Schema Design**

### **Schema Definition**
```javascript
const { gql } = require('apollo-server-express');

// Comprehensive GraphQL schema
const typeDefs = gql`
  # Interfaces
  interface Node {
    id: ID!
    createdAt: String!
    updatedAt: String!
  }

  interface UserOwned {
    author: User!
  }

  # Enums
  enum UserRole {
    USER
    MODERATOR
    ADMIN
  }

  enum PostStatus {
    DRAFT
    PUBLISHED
    ARCHIVED
  }

  enum SortOrder {
    ASC
    DESC
  }

  # Types
  type User implements Node {
    id: ID!
    firstName: String!
    lastName: String!
    fullName: String!
    email: String!
    avatar: String
    role: UserRole!
    isVerified: Boolean!
    profile: UserProfile
    posts(limit: Int, offset: Int): [Post!]!
    likedPosts(limit: Int, offset: Int): [Post!]!
    comments(limit: Int, offset: Int): [Comment!]!
    followers: [User!]!
    following: [User!]!
    followerCount: Int!
    followingCount: Int!
    createdAt: String!
    updatedAt: String!
  }

  type UserProfile {
    bio: String
    website: String
    location: String
    birthday: String
    socialLinks: SocialLinks
  }

  type SocialLinks {
    twitter: String
    linkedin: String
    github: String
    instagram: String
  }

  type Post implements Node & UserOwned {
    id: ID!
    title: String!
    slug: String!
    content: String!
    excerpt: String!
    featuredImage: String
    author: User!
    category: Category!
    tags: [Tag!]!
    status: PostStatus!
    views: Int!
    readingTime: Int!
    likes: [Like!]!
    likeCount: Int!
    comments: [Comment!]!
    commentCount: Int!
    isLikedByUser: Boolean!
    publishedAt: String
    createdAt: String!
    updatedAt: String!
  }

  type Category implements Node {
    id: ID!
    name: String!
    slug: String!
    description: String
    color: String!
    icon: String
    postCount: Int!
    posts(limit: Int, offset: Int): [Post!]!
    createdAt: String!
    updatedAt: String!
  }

  type Tag implements Node {
    id: ID!
    name: String!
    slug: String!
    description: String
    color: String!
    usageCount: Int!
    posts(limit: Int, offset: Int): [Post!]!
    createdAt: String!
    updatedAt: String!
  }

  type Like implements Node {
    id: ID!
    user: User!
    post: Post!
    createdAt: String!
    updatedAt: String!
  }

  type Comment implements Node & UserOwned {
    id: ID!
    content: String!
    author: User!
    post: Post!
    likes: [Like!]!
    likeCount: Int!
    replies: [Comment!]!
    replyCount: Int!
    isLikedByUser: Boolean!
    createdAt: String!
    updatedAt: String!
  }

  # Connection types for pagination
  type UserConnection {
    edges: [UserEdge!]!
    pageInfo: PageInfo!
    totalCount: Int!
  }

  type UserEdge {
    node: User!
    cursor: String!
  }

  type PostConnection {
    edges: [PostEdge!]!
    pageInfo: PageInfo!
    totalCount: Int!
  }

  type PostEdge {
    node: Post!
    cursor: String!
  }

  type PageInfo {
    hasNextPage: Boolean!
    hasPreviousPage: Boolean!
    startCursor: String
    endCursor: String
  }

  # Queries
  type Query {
    # User queries
    me: User
    user(id: ID!): User
    users(
      first: Int
      after: String
      last: Int
      before: String
      search: String
      role: UserRole
      sortBy: UserSortField
      sortOrder: SortOrder
    ): UserConnection!

    # Post queries
    post(id: ID!): Post
    posts(
      first: Int
      after: String
      last: Int
      before: String
      category: ID
      tag: ID
      author: ID
      status: PostStatus
      search: String
      sortBy: PostSortField
      sortOrder: SortOrder
    ): PostConnection!

    # Category and Tag queries
    categories: [Category!]!
    category(slug: String!): Category
    tags(limit: Int): [Tag!]!
    tag(slug: String!): Tag

    # Search
    search(query: String!, type: SearchType, limit: Int): SearchResult!
  }

  # Mutations
  type Mutation {
    # Authentication
    signUp(input: SignUpInput!): AuthPayload!
    signIn(input: SignInInput!): AuthPayload!
    signOut: Boolean!
    refreshToken: AuthPayload!

    # User mutations
    updateUser(input: UpdateUserInput!): User!
    updateUserProfile(input: UpdateUserProfileInput!): UserProfile!
    changePassword(input: ChangePasswordInput!): Boolean!
    deleteUser: Boolean!

    # Post mutations
    createPost(input: CreatePostInput!): Post!
    updatePost(id: ID!, input: UpdatePostInput!): Post!
    deletePost(id: ID!): Boolean!
    publishPost(id: ID!): Post!
    unpublishPost(id: ID!): Post!

    # Social mutations
    likePost(postId: ID!): Like!
    unlikePost(postId: ID!): Boolean!
    followUser(userId: ID!): Follow!
    unfollowUser(userId: ID!): Boolean!

    # Comment mutations
    createComment(postId: ID!, input: CreateCommentInput!): Comment!
    updateComment(id: ID!, content: String!): Comment!
    deleteComment(id: ID!): Boolean!
    likeComment(commentId: ID!): Like!
    unlikeComment(commentId: ID!): Boolean!

    # Admin mutations
    createCategory(input: CreateCategoryInput!): Category!
    updateCategory(id: ID!, input: UpdateCategoryInput!): Category!
    deleteCategory(id: ID!): Boolean!
    createTag(input: CreateTagInput!): Tag!
    updateTag(id: ID!, input: UpdateTagInput!): Tag!
    deleteTag(id: ID!): Boolean!
  }

  # Subscriptions
  type Subscription {
    postCreated: Post!
    postUpdated: Post!
    postDeleted: Post!
    commentCreated(postId: ID!): Comment!
    userFollowed: Follow!
  }

  # Enums
  enum UserSortField {
    CREATED_AT
    FIRST_NAME
    LAST_NAME
    EMAIL
  }

  enum PostSortField {
    CREATED_AT
    PUBLISHED_AT
    TITLE
    VIEWS
    LIKES
  }

  enum SearchType {
    POSTS
    USERS
    TAGS
    ALL
  }

  # Union types
  union SearchResult = Post | User | Tag

  # Auth types
  type AuthPayload {
    user: User!
    token: String!
    refreshToken: String!
  }

  type Follow {
    id: ID!
    follower: User!
    following: User!
    createdAt: String!
  }

  # Input types
  input SignUpInput {
    firstName: String!
    lastName: String!
    email: String!
    password: String!
  }

  input SignInInput {
    email: String!
    password: String!
  }

  input UpdateUserInput {
    firstName: String
    lastName: String
    email: String
    avatar: String
  }

  input UpdateUserProfileInput {
    bio: String
    website: String
    location: String
    birthday: String
    socialLinks: SocialLinksInput
  }

  input SocialLinksInput {
    twitter: String
    linkedin: String
    github: String
    instagram: String
  }

  input ChangePasswordInput {
    currentPassword: String!
    newPassword: String!
  }

  input CreatePostInput {
    title: String!
    content: String!
    categoryId: ID!
    tagIds: [ID!]
    featuredImage: String
  }

  input UpdatePostInput {
    title: String
    content: String
    categoryId: ID
    tagIds: [ID!]
    featuredImage: String
  }

  input CreateCommentInput {
    content: String!
    parentId: ID # For replies
  }

  input CreateCategoryInput {
    name: String!
    description: String
    color: String
    icon: String
  }

  input UpdateCategoryInput {
    name: String
    description: String
    color: String
    icon: String
  }

  input CreateTagInput {
    name: String!
    description: String
    color: String
  }

  input UpdateTagInput {
    name: String
    description: String
    color: String
  }
`;
```

---

## 🛠️ **GraphQL Resolvers**

### **Query Resolvers**
```javascript
const resolvers = {
  Query: {
    // User queries
    me: async (parent, args, context) => {
      if (!context.user) throw new AuthenticationError('Not authenticated');
      return await User.findById(context.user.id);
    },

    user: async (parent, { id }, context) => {
      return await User.findById(id);
    },

    users: async (parent, args, context) => {
      const {
        first,
        after,
        last,
        before,
        search,
        role,
        sortBy = 'CREATED_AT',
        sortOrder = 'DESC',
      } = args;

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

      // Build sort
      const sortOptions = {};
      switch (sortBy) {
        case 'FIRST_NAME':
          sortOptions.firstName = sortOrder === 'DESC' ? -1 : 1;
          break;
        case 'LAST_NAME':
          sortOptions.lastName = sortOrder === 'DESC' ? -1 : 1;
          break;
        case 'EMAIL':
          sortOptions.email = sortOrder === 'DESC' ? -1 : 1;
          break;
        default:
          sortOptions.createdAt = sortOrder === 'DESC' ? -1 : 1;
      }

      // Execute query
      const users = await User.find(query)
        .sort(sortOptions)
        .limit(first || last || 10);

      return {
        edges: users.map(user => ({
          node: user,
          cursor: user._id.toString(),
        })),
        pageInfo: {
          hasNextPage: users.length === (first || 10),
          hasPreviousPage: false, // Simplified
          startCursor: users[0]?._id.toString(),
          endCursor: users[users.length - 1]?._id.toString(),
        },
        totalCount: await User.countDocuments(query),
      };
    },

    // Post queries
    post: async (parent, { id }, context) => {
      const post = await Post.findById(id)
        .populate('author')
        .populate('category')
        .populate('tags');

      if (post) {
        // Increment view count
        post.views += 1;
        await post.save();

        // Check if user liked the post
        if (context.user) {
          post.isLikedByUser = post.likes.some(
            like => like.user.toString() === context.user.id
          );
        }
      }

      return post;
    },

    posts: async (parent, args, context) => {
      const {
        first,
        after,
        last,
        before,
        category,
        tag,
        author,
        status,
        search,
        sortBy = 'CREATED_AT',
        sortOrder = 'DESC',
      } = args;

      // Build query
      const query = { status: 'published' }; // Default to published posts
      if (category) query.category = category;
      if (tag) query.tags = tag;
      if (author) query.author = author;
      if (status) query.status = status;
      if (search) {
        query.$text = { $search: search };
      }

      // Build sort
      const sortOptions = {};
      switch (sortBy) {
        case 'TITLE':
          sortOptions.title = sortOrder === 'DESC' ? -1 : 1;
          break;
        case 'VIEWS':
          sortOptions.views = sortOrder === 'DESC' ? -1 : 1;
          break;
        case 'LIKES':
          sortOptions.likeCount = sortOrder === 'DESC' ? -1 : 1;
          break;
        case 'PUBLISHED_AT':
          sortOptions.publishedAt = sortOrder === 'DESC' ? -1 : 1;
          break;
        default:
          sortOptions.createdAt = sortOrder === 'DESC' ? -1 : 1;
      }

      // Execute query
      const posts = await Post.find(query)
        .populate('author', 'firstName lastName avatar')
        .populate('category', 'name slug color')
        .populate('tags', 'name slug color')
        .sort(sortOptions)
        .limit(first || last || 10);

      // Add computed fields
      const postsWithComputed = posts.map(post => ({
        ...post.toObject(),
        likeCount: post.likes.length,
        commentCount: post.comments.length,
        readingTime: Math.ceil(post.content.split(' ').length / 200),
        isLikedByUser: context.user ?
          post.likes.some(like => like.user.toString() === context.user.id) :
          false,
      }));

      return {
        edges: postsWithComputed.map(post => ({
          node: post,
          cursor: post._id.toString(),
        })),
        pageInfo: {
          hasNextPage: posts.length === (first || 10),
          hasPreviousPage: false, // Simplified
          startCursor: posts[0]?._id.toString(),
          endCursor: posts[posts.length - 1]?._id.toString(),
        },
        totalCount: await Post.countDocuments(query),
      };
    },

    // Category and Tag queries
    categories: async () => {
      return await Category.find({ isActive: true })
        .sort({ order: 1 });
    },

    category: async (parent, { slug }) => {
      return await Category.findOne({ slug, isActive: true });
    },

    tags: async (parent, { limit = 50 }) => {
      return await Tag.find({ isActive: true })
        .sort({ usageCount: -1 })
        .limit(limit);
    },

    tag: async (parent, { slug }) => {
      return await Tag.findOne({ slug, isActive: true });
    },

    // Search query
    search: async (parent, { query, type, limit = 20 }) => {
      const searchRegex = new RegExp(query, 'i');

      switch (type) {
        case 'POSTS':
          return await Post.find({
            status: 'published',
            $or: [
              { title: searchRegex },
              { content: searchRegex },
              { excerpt: searchRegex },
            ],
          })
            .populate('author', 'firstName lastName')
            .limit(limit);

        case 'USERS':
          return await User.find({
            $or: [
              { firstName: searchRegex },
              { lastName: searchRegex },
              { email: searchRegex },
            ],
          })
            .select('firstName lastName email avatar')
            .limit(limit);

        case 'TAGS':
          return await Tag.find({
            name: searchRegex,
            isActive: true,
          }).limit(limit);

        default: // ALL
          const [posts, users, tags] = await Promise.all([
            Post.find({
              status: 'published',
              $or: [
                { title: searchRegex },
                { content: searchRegex },
              ],
            }).limit(5),
            User.find({
              $or: [
                { firstName: searchRegex },
                { lastName: searchRegex },
              ],
            }).select('firstName lastName email avatar').limit(5),
            Tag.find({
              name: searchRegex,
              isActive: true,
            }).limit(5),
          ]);

          return [...posts, ...users, ...tags];
      }
    },
  },
};
```

### **Mutation Resolvers**
```javascript
const resolvers = {
  Mutation: {
    // Authentication mutations
    signUp: async (parent, { input }) => {
      const { firstName, lastName, email, password } = input;

      // Check if user exists
      const existingUser = await User.findOne({ email: email.toLowerCase() });
      if (existingUser) {
        throw new UserInputError('User with this email already exists');
      }

      // Create user
      const user = await User.create({
        firstName,
        lastName,
        email: email.toLowerCase(),
        password,
      });

      // Generate tokens
      const token = generateToken(user);
      const refreshToken = generateRefreshToken(user);

      return {
        user,
        token,
        refreshToken,
      };
    },

    signIn: async (parent, { input }) => {
      const { email, password } = input;

      const user = await User.findOne({ email: email.toLowerCase() });
      if (!user) {
        throw new AuthenticationError('Invalid credentials');
      }

      const isValidPassword = await user.comparePassword(password);
      if (!isValidPassword) {
        throw new AuthenticationError('Invalid credentials');
      }

      // Update last login
      user.lastLogin = new Date();
      await user.save();

      const token = generateToken(user);
      const refreshToken = generateRefreshToken(user);

      return {
        user,
        token,
        refreshToken,
      };
    },

    // User mutations
    updateUser: async (parent, { input }, context) => {
      if (!context.user) throw new AuthenticationError('Not authenticated');

      const updates = {};
      if (input.firstName) updates.firstName = input.firstName;
      if (input.lastName) updates.lastName = input.lastName;
      if (input.email) updates.email = input.email.toLowerCase();
      if (input.avatar) updates.avatar = input.avatar;

      const user = await User.findByIdAndUpdate(
        context.user.id,
        updates,
        { new: true, runValidators: true }
      );

      if (!user) throw new Error('User not found');

      return user;
    },

    // Post mutations
    createPost: async (parent, { input }, context) => {
      if (!context.user) throw new AuthenticationError('Not authenticated');

      const { title, content, categoryId, tagIds, featuredImage } = input;

      // Validate category exists
      const category = await Category.findById(categoryId);
      if (!category) {
        throw new UserInputError('Invalid category');
      }

      // Validate tags exist
      if (tagIds && tagIds.length > 0) {
        const tags = await Tag.find({ _id: { $in: tagIds } });
        if (tags.length !== tagIds.length) {
          throw new UserInputError('One or more tags are invalid');
        }
      }

      const post = await Post.create({
        title,
        content,
        author: context.user.id,
        category: categoryId,
        tags: tagIds || [],
        featuredImage,
      });

      return await Post.findById(post._id)
        .populate('author', 'firstName lastName avatar')
        .populate('category', 'name slug color')
        .populate('tags', 'name slug color');
    },

    updatePost: async (parent, { id, input }, context) => {
      if (!context.user) throw new AuthenticationError('Not authenticated');

      const post = await Post.findById(id);
      if (!post) throw new Error('Post not found');

      // Check ownership
      if (post.author.toString() !== context.user.id && context.user.role !== 'admin') {
        throw new ForbiddenError('Not authorized to update this post');
      }

      const updates = {};
      if (input.title) updates.title = input.title;
      if (input.content) updates.content = input.content;
      if (input.categoryId) {
        const category = await Category.findById(input.categoryId);
        if (!category) throw new UserInputError('Invalid category');
        updates.category = input.categoryId;
      }
      if (input.tagIds !== undefined) {
        if (input.tagIds.length > 0) {
          const tags = await Tag.find({ _id: { $in: input.tagIds } });
          if (tags.length !== input.tagIds.length) {
            throw new UserInputError('One or more tags are invalid');
          }
        }
        updates.tags = input.tagIds;
      }
      if (input.featuredImage !== undefined) updates.featuredImage = input.featuredImage;

      const updatedPost = await Post.findByIdAndUpdate(
        id,
        updates,
        { new: true, runValidators: true }
      )
        .populate('author', 'firstName lastName avatar')
        .populate('category', 'name slug color')
        .populate('tags', 'name slug color');

      return updatedPost;
    },

    // Social mutations
    likePost: async (parent, { postId }, context) => {
      if (!context.user) throw new AuthenticationError('Not authenticated');

      const post = await Post.findById(postId);
      if (!post) throw new Error('Post not found');

      // Check if already liked
      const existingLike = post.likes.find(
        like => like.user.toString() === context.user.id
      );

      if (existingLike) {
        throw new UserInputError('Post already liked');
      }

      const like = {
        user: context.user.id,
        createdAt: new Date(),
      };

      post.likes.push(like);
      await post.save();

      return {
        id: `${postId}_${context.user.id}`,
        user: context.user,
        post: post,
        createdAt: like.createdAt.toISOString(),
      };
    },

    unlikePost: async (parent, { postId }, context) => {
      if (!context.user) throw new AuthenticationError('Not authenticated');

      const post = await Post.findById(postId);
      if (!post) throw new Error('Post not found');

      const likeIndex = post.likes.findIndex(
        like => like.user.toString() === context.user.id
      );

      if (likeIndex === -1) {
        throw new UserInputError('Post not liked');
      }

      post.likes.splice(likeIndex, 1);
      await post.save();

      return true;
    },

    // Comment mutations
    createComment: async (parent, { postId, input }, context) => {
      if (!context.user) throw new AuthenticationError('Not authenticated');

      const { content, parentId } = input;

      const post = await Post.findById(postId);
      if (!post) throw new Error('Post not found');

      const comment = {
        content,
        author: context.user.id,
        createdAt: new Date(),
      };

      if (parentId) {
        // This is a reply
        const parentComment = post.comments.id(parentId);
        if (!parentComment) throw new Error('Parent comment not found');

        if (!parentComment.replies) parentComment.replies = [];
        parentComment.replies.push(comment);
      } else {
        // This is a top-level comment
        post.comments.push(comment);
      }

      await post.save();

      // Return the created comment with populated author
      const createdComment = post.comments[post.comments.length - 1];
      await post.populate('comments.author', 'firstName lastName avatar');

      return {
        id: createdComment._id,
        content: createdComment.content,
        author: createdComment.author,
        post: post,
        createdAt: createdComment.createdAt.toISOString(),
        likeCount: 0,
        isLikedByUser: false,
        replies: [],
        replyCount: 0,
      };
    },
  },
};
```

---

## ⚖️ **REST vs GraphQL Comparison**

### **When to Use REST**
```javascript
// ✅ Good for REST
// Simple CRUD operations
app.get('/api/users/:id', getUser);
app.post('/api/users', createUser);
app.put('/api/users/:id', updateUser);
app.delete('/api/users/:id', deleteUser);

// File uploads
app.post('/api/upload', upload.single('file'), handleUpload);

// Simple, predictable endpoints
// Easy caching with HTTP headers
// Great for public APIs
// Simple debugging
```

### **When to Use GraphQL**
```javascript
// ✅ Good for GraphQL
// Complex, nested data requirements
query GetUserProfile($userId: ID!) {
  user(id: $userId) {
    firstName
    lastName
    posts(limit: 5) {
      title
      category {
        name
      }
      comments(limit: 3) {
        content
        author {
          firstName
        }
      }
    }
  }
}

// Mobile apps with varying data needs
// Real-time subscriptions
subscription OnPostCreated {
  postCreated {
    id
    title
    author {
      firstName
      lastName
    }
  }
}

// API evolution without breaking changes
// Advanced filtering and sorting
```

### **Hybrid Approach**
```javascript
// Use REST for simple operations, GraphQL for complex queries
const express = require('express');
const { ApolloServer } = require('apollo-server-express');

const app = express();

// REST API for simple operations
app.use('/api/v1', restRoutes);

// GraphQL API for complex operations
const server = new ApolloServer({ typeDefs, resolvers });
server.applyMiddleware({ app, path: '/graphql' });

// Both APIs can coexist
app.listen(3000, () => {
  console.log('REST API: http://localhost:3000/api/v1');
  console.log('GraphQL API: http://localhost:3000/graphql');
});
```

---

## 📚 **API Documentation**

### **Swagger/OpenAPI Documentation**
```javascript
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'Blog API',
      version: '1.0.0',
      description: 'A comprehensive blog API with user management, posts, and comments',
    },
    servers: [
      {
        url: 'http://localhost:3000/api/v1',
        description: 'Development server',
      },
      {
        url: 'https://api.myblog.com/v1',
        description: 'Production server',
      },
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT',
        },
      },
      schemas: {
        User: {
          type: 'object',
          required: ['firstName', 'lastName', 'email', 'password'],
          properties: {
            id: {
              type: 'string',
              description: 'Unique identifier for the user',
            },
            firstName: {
              type: 'string',
              description: 'User\'s first name',
            },
            lastName: {
              type: 'string',
              description: 'User\'s last name',
            },
            email: {
              type: 'string',
              format: 'email',
              description: 'User\'s email address',
            },
            avatar: {
              type: 'string',
              description: 'URL to user\'s avatar image',
            },
            role: {
              type: 'string',
              enum: ['user', 'moderator', 'admin'],
              default: 'user',
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
        Post: {
          type: 'object',
          required: ['title', 'content', 'author', 'category'],
          properties: {
            id: {
              type: 'string',
              description: 'Unique identifier for the post',
            },
            title: {
              type: 'string',
              description: 'Post title',
            },
            content: {
              type: 'string',
              description: 'Post content',
            },
            excerpt: {
              type: 'string',
              description: 'Post excerpt',
            },
            featuredImage: {
              type: 'string',
              description: 'URL to featured image',
            },
            author: {
              type: 'string',
              description: 'Author ID',
            },
            category: {
              type: 'string',
              description: 'Category ID',
            },
            tags: {
              type: 'array',
              items: { type: 'string' },
              description: 'Array of tag IDs',
            },
            status: {
              type: 'string',
              enum: ['draft', 'published', 'archived'],
              default: 'draft',
            },
            views: {
              type: 'integer',
              description: 'Number of views',
              default: 0,
            },
            publishedAt: {
              type: 'string',
              format: 'date-time',
              description: 'Publication timestamp',
            },
            createdAt: {
              type: 'string',
              format: 'date-time',
              description: 'Creation timestamp',
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
      '/api/posts': {
        get: {
          summary: 'Get all posts',
          tags: ['Posts'],
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
              name: 'category',
              in: 'query',
              schema: { type: 'string' },
              description: 'Filter by category ID',
            },
            {
              name: 'author',
              in: 'query',
              schema: { type: 'string' },
              description: 'Filter by author ID',
            },
            {
              name: 'status',
              in: 'query',
              schema: { type: 'string', enum: ['draft', 'published', 'archived'] },
              description: 'Filter by status',
            },
            {
              name: 'search',
              in: 'query',
              schema: { type: 'string' },
              description: 'Search in title and content',
            },
          ],
          responses: {
            200: {
              description: 'Posts retrieved successfully',
              content: {
                'application/json': {
                  schema: {
                    type: 'object',
                    properties: {
                      success: { type: 'boolean', example: true },
                      data: {
                        type: 'array',
                        items: { $ref: '#/components/schemas/Post' },
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
          },
        },
        post: {
          summary: 'Create a new post',
          tags: ['Posts'],
          security: [{ bearerAuth: [] }],
          requestBody: {
            required: true,
            content: {
              'application/json': {
                schema: {
                  type: 'object',
                  required: ['title', 'content', 'categoryId'],
                  properties: {
                    title: { type: 'string', example: 'My Blog Post' },
                    content: { type: 'string', example: 'This is the content of my blog post...' },
                    categoryId: { type: 'string', example: '60d5ecb54bbb4c001f8b4567' },
                    tagIds: {
                      type: 'array',
                      items: { type: 'string' },
                      example: ['60d5ecb54bbb4c001f8b4568', '60d5ecb54bbb4c001f8b4569'],
                    },
                    featuredImage: { type: 'string', example: 'https://example.com/image.jpg' },
                  },
                },
              },
            },
          },
          responses: {
            201: {
              description: 'Post created successfully',
              content: {
                'application/json': {
                  schema: {
                    type: 'object',
                    properties: {
                      success: { type: 'boolean', example: true },
                      data: { $ref: '#/components/schemas/Post' },
                      message: { type: 'string', example: 'Post created successfully' },
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
      },
      '/api/posts/{id}': {
        get: {
          summary: 'Get post by ID',
          tags: ['Posts'],
          parameters: [
            {
              name: 'id',
              in: 'path',
              required: true,
              schema: { type: 'string' },
              description: 'Post ID',
            },
          ],
          responses: {
            200: {
              description: 'Post retrieved successfully',
              content: {
                'application/json': {
                  schema: {
                    type: 'object',
                    properties: {
                      success: { type: 'boolean', example: true },
                      data: { $ref: '#/components/schemas/Post' },
                    },
                  },
                },
              },
            },
            404: {
              description: 'Post not found',
              content: {
                'application/json': {
                  schema: { $ref: '#/components/schemas/Error' },
                },
              },
            },
          },
        },
        put: {
          summary: 'Update post',
          tags: ['Posts'],
          security: [{ bearerAuth: [] }],
          parameters: [
            {
              name: 'id',
              in: 'path',
              required: true,
              schema: { type: 'string' },
              description: 'Post ID',
            },
          ],
          requestBody: {
            required: true,
            content: {
              'application/json': {
                schema: {
                  type: 'object',
                  properties: {
                    title: { type: 'string', example: 'Updated Title' },
                    content: { type: 'string', example: 'Updated content...' },
                    categoryId: { type: 'string', example: '60d5ecb54bbb4c001f8b4567' },
                    tagIds: {
                      type: 'array',
                      items: { type: 'string' },
                      example: ['60d5ecb54bbb4c001f8b4568'],
                    },
                    featuredImage: { type: 'string', example: 'https://example.com/new-image.jpg' },
                  },
                },
              },
            },
          },
          responses: {
            200: {
              description: 'Post updated successfully',
              content: {
                'application/json': {
                  schema: {
                    type: 'object',
                    properties: {
                      success: { type: 'boolean', example: true },
                      data: { $ref: '#/components/schemas/Post' },
                      message: { type: 'string', example: 'Post updated successfully' },
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
              description: 'Post not found',
              content: {
                'application/json': {
                  schema: { $ref: '#/components/schemas/Error' },
                },
              },
            },
          },
        },
        delete: {
          summary: 'Delete post',
          tags: ['Posts'],
          security: [{ bearerAuth: [] }],
          parameters: [
            {
              name: 'id',
              in: 'path',
              required: true,
              schema: { type: 'string' },
              description: 'Post ID',
            },
          ],
          responses: {
            200: {
              description: 'Post deleted successfully',
              content: {
                'application/json': {
                  schema: {
                    type: 'object',
                    properties: {
                      success: { type: 'boolean', example: true },
                      message: { type: 'string', example: 'Post deleted successfully' },
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
              description: 'Post not found',
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
  - ✅ **RESTful API Design**: HTTP methods, status codes, resource naming
  - ✅ **API Versioning**: URL path, header, and content negotiation versioning
  - ✅ **GraphQL Introduction**: Schema design, queries, mutations
  - ✅ **GraphQL Resolvers**: Query and mutation resolvers with authentication
  - ✅ **REST vs GraphQL**: When to use each approach
  - ✅ **API Documentation**: Swagger/OpenAPI and GraphQL documentation
  - ✅ **API Security**: Authentication, authorization, security headers
  - ✅ **Practical Implementation**: Complete blog API with both REST and GraphQL

  ### **Best Practices**
  1. **Use consistent resource naming** - Follow REST conventions
  2. **Implement proper HTTP status codes** - Use appropriate codes for different scenarios
  3. **Version your APIs** - Plan for breaking changes
  4. **Document your APIs** - Keep documentation up to date
  5. **Implement security measures** - Authentication, authorization, rate limiting
  6. **Handle errors gracefully** - Provide meaningful error messages
  7. **Use appropriate content types** - JSON for APIs, proper headers
  8. **Test your APIs thoroughly** - Unit tests, integration tests

  ### **Next Steps**
  - Implement advanced GraphQL features like subscriptions
  - Set up API monitoring and analytics
  - Implement API caching strategies
  - Create API client libraries
  - Learn about API gateways and microservices

  ---

  ## 🎯 **Assignment**

  ### **Task 1: RESTful E-commerce API**
  Create a complete e-commerce REST API with:
  - Product management (CRUD operations)
  - Category and brand management
  - Shopping cart functionality
  - Order processing and management
  - User authentication and profiles
  - Payment integration endpoints
  - Admin dashboard endpoints
  - Comprehensive API documentation

  ### **Task 2: GraphQL Social Media API**
  Build a social media GraphQL API including:
  - User management with followers/following
  - Posts with likes, comments, and shares
  - Real-time subscriptions for new posts
  - Advanced filtering and search
  - Content moderation features
  - Analytics and insights
  - File upload for media content

  ### **Task 3: Hybrid API (REST + GraphQL)**
  Implement a hybrid API system that:
  - Uses REST for simple CRUD operations
  - Uses GraphQL for complex queries
  - Shares authentication and authorization
  - Provides unified API documentation
  - Implements proper error handling
  - Includes comprehensive testing

  ---

  ## 📚 **Additional Resources**
  - [REST API Design Best Practices](https://restfulapi.net/)
  - [GraphQL Documentation](https://graphql.org/learn/)
  - [Apollo Server Documentation](https://www.apollographql.com/docs/apollo-server/)
  - [Swagger/OpenAPI Specification](https://swagger.io/specification/)
  - [JWT Authentication](https://jwt.io/)

  ---

  **Next Lesson**: [Lesson 24: Authentication & Security](Lesson%2024_%20Authentication%20&%20Security.md)
