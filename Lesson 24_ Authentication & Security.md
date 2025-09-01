# Lesson 24: Authentication & Security

## 🎯 **Learning Objectives**
- Implement secure authentication systems
- Understand JWT tokens and session management
- Create role-based access control (RBAC)
- Implement security best practices
- Handle common security vulnerabilities

## 📚 **Table of Contents**
1. [Authentication Fundamentals](#authentication-fundamentals)
2. [JWT Implementation](#jwt-implementation)
3. [Password Security](#password-security)
4. [Session Management](#session-management)
5. [Role-Based Access Control](#role-based-access-control)
6. [OAuth Integration](#oauth-integration)
7. [Security Best Practices](#security-best-practices)
8. [Common Vulnerabilities](#common-vulnerabilities)
9. [API Security](#api-security)
10. [Practical Examples](#practical-examples)

---

## 🔐 **Authentication Fundamentals**

### **Authentication Types**
```javascript
// 1. Session-based Authentication
// Server stores session data, client receives session ID
const sessionAuth = {
  login: async (email, password) => {
    // Validate credentials
    const user = await User.findOne({ email }).select('+password');
    const isValid = await user.comparePassword(password);

    if (!isValid) throw new Error('Invalid credentials');

    // Create session
    const sessionId = crypto.randomBytes(32).toString('hex');
    await redis.set(`session:${sessionId}`, user.id, 'EX', 24 * 60 * 60); // 24 hours

    return { sessionId, user };
  },

  authenticate: async (sessionId) => {
    const userId = await redis.get(`session:${sessionId}`);
    if (!userId) throw new Error('Invalid session');

    return await User.findById(userId);
  },

  logout: async (sessionId) => {
    await redis.del(`session:${sessionId}`);
  },
};

// 2. Token-based Authentication (JWT)
const tokenAuth = {
  generateTokens: (user) => {
    const accessToken = jwt.sign(
      { id: user.id, email: user.email, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );

    const refreshToken = jwt.sign(
      { id: user.id },
      process.env.JWT_REFRESH_SECRET,
      { expiresIn: '7d' }
    );

    return { accessToken, refreshToken };
  },

  verifyToken: (token, secret = process.env.JWT_SECRET) => {
    return jwt.verify(token, secret);
  },
};

// 3. API Key Authentication
const apiKeyAuth = {
  generateApiKey: () => {
    return crypto.randomBytes(32).toString('hex');
  },

  authenticateApiKey: async (apiKey) => {
    const user = await User.findOne({ apiKey });
    if (!user) throw new Error('Invalid API key');

    return user;
  },
};
```

---

## 🎫 **JWT Implementation**

### **JWT Token Management**
```javascript
const jwt = require('jsonwebtoken');
const crypto = require('crypto');

class JWTManager {
  constructor() {
    this.accessTokenSecret = process.env.JWT_SECRET;
    this.refreshTokenSecret = process.env.JWT_REFRESH_SECRET;
    this.accessTokenExpiry = '15m';
    this.refreshTokenExpiry = '7d';
  }

  // Generate access token
  generateAccessToken(payload) {
    return jwt.sign(payload, this.accessTokenSecret, {
      expiresIn: this.accessTokenExpiry,
      issuer: 'myapp.com',
      audience: 'myapp-users',
    });
  }

  // Generate refresh token
  generateRefreshToken(payload) {
    return jwt.sign(payload, this.refreshTokenSecret, {
      expiresIn: this.refreshTokenExpiry,
      issuer: 'myapp.com',
      audience: 'myapp-refresh',
    });
  }

  // Generate both tokens
  generateTokens(user) {
    const payload = {
      id: user.id,
      email: user.email,
      role: user.role,
    };

    return {
      accessToken: this.generateAccessToken(payload),
      refreshToken: this.generateRefreshToken({ id: user.id }),
    };
  }

  // Verify access token
  verifyAccessToken(token) {
    try {
      return jwt.verify(token, this.accessTokenSecret, {
        issuer: 'myapp.com',
        audience: 'myapp-users',
      });
    } catch (error) {
      throw new Error('Invalid access token');
    }
  }

  // Verify refresh token
  verifyRefreshToken(token) {
    try {
      return jwt.verify(token, this.refreshTokenSecret, {
        issuer: 'myapp.com',
        audience: 'myapp-refresh',
      });
    } catch (error) {
      throw new Error('Invalid refresh token');
    }
  }

  // Decode token without verification
  decodeToken(token) {
    return jwt.decode(token);
  }

  // Check if token is expired
  isTokenExpired(token) {
    try {
      const decoded = jwt.decode(token);
      const currentTime = Math.floor(Date.now() / 1000);
      return decoded.exp < currentTime;
    } catch (error) {
      return true;
    }
  }

  // Refresh access token
  async refreshAccessToken(refreshToken) {
    const decoded = this.verifyRefreshToken(refreshToken);
    const user = await User.findById(decoded.id);

    if (!user) {
      throw new Error('User not found');
    }

    return this.generateAccessToken({
      id: user.id,
      email: user.email,
      role: user.role,
    });
  }
}

const jwtManager = new JWTManager();
module.exports = jwtManager;
```

### **JWT Middleware**
```javascript
const jwtManager = require('./jwtManager');

// Authentication middleware
const authenticateToken = async (req, res, next) => {
  try {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN

    if (!token) {
      return res.status(401).json({
        success: false,
        error: 'Access token required',
      });
    }

    // Verify token
    const decoded = jwtManager.verifyAccessToken(token);

    // Get user from database
    const user = await User.findById(decoded.id).select('-password');
    if (!user) {
      return res.status(401).json({
        success: false,
        error: 'User not found',
      });
    }

    // Attach user to request
    req.user = user;
    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({
        success: false,
        error: 'Token expired',
        code: 'TOKEN_EXPIRED',
      });
    }

    return res.status(403).json({
      success: false,
      error: 'Invalid token',
    });
  }
};

// Optional authentication (doesn't fail if no token)
const optionalAuth = async (req, res, next) => {
  try {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];

    if (token) {
      const decoded = jwtManager.verifyAccessToken(token);
      const user = await User.findById(decoded.id).select('-password');
      req.user = user;
    }
  } catch (error) {
    // Ignore auth errors for optional auth
  }

  next();
};

// Refresh token endpoint
const refreshToken = async (req, res) => {
  try {
    const { refreshToken } = req.body;

    if (!refreshToken) {
      return res.status(400).json({
        success: false,
        error: 'Refresh token required',
      });
    }

    const newAccessToken = await jwtManager.refreshAccessToken(refreshToken);

    res.json({
      success: true,
      accessToken: newAccessToken,
    });
  } catch (error) {
    res.status(403).json({
      success: false,
      error: 'Invalid refresh token',
    });
  }
};

module.exports = {
  authenticateToken,
  optionalAuth,
  refreshToken,
};
```

---

## 🔒 **Password Security**

### **Password Hashing & Validation**
```javascript
const bcrypt = require('bcrypt');
const crypto = require('crypto');

class PasswordManager {
  constructor() {
    this.saltRounds = 12;
  }

  // Hash password
  async hashPassword(password) {
    const salt = await bcrypt.genSalt(this.saltRounds);
    return bcrypt.hash(password, salt);
  }

  // Verify password
  async verifyPassword(password, hashedPassword) {
    return bcrypt.compare(password, hashedPassword);
  }

  // Generate password reset token
  generateResetToken() {
    return crypto.randomBytes(32).toString('hex');
  }

  // Validate password strength
  validatePasswordStrength(password) {
    const errors = [];

    if (password.length < 8) {
      errors.push('Password must be at least 8 characters long');
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

    return {
      isValid: errors.length === 0,
      errors,
    };
  }

  // Check password history (prevent reuse)
  async checkPasswordHistory(userId, newPassword) {
    const user = await User.findById(userId).select('passwordHistory');

    if (!user.passwordHistory) return true;

    for (const oldPassword of user.passwordHistory) {
      const isSame = await bcrypt.compare(newPassword, oldPassword);
      if (isSame) {
        return false;
      }
    }

    return true;
  }

  // Update password with history
  async updatePassword(userId, newPassword) {
    const user = await User.findById(userId);

    // Check if new password is different from current
    const isSameAsCurrent = await bcrypt.compare(newPassword, user.password);
    if (isSameAsCurrent) {
      throw new Error('New password must be different from current password');
    }

    // Check password history
    const isReused = !(await this.checkPasswordHistory(userId, newPassword));
    if (isReused) {
      throw new Error('Password has been used recently. Please choose a different password');
    }

    // Hash new password
    const hashedPassword = await this.hashPassword(newPassword);

    // Update password history
    if (!user.passwordHistory) user.passwordHistory = [];
    user.passwordHistory.unshift(user.password); // Add current password to history
    user.passwordHistory = user.passwordHistory.slice(0, 5); // Keep last 5 passwords

    // Update password
    user.password = hashedPassword;
    user.passwordChangedAt = new Date();

    await user.save();

    return user;
  }
}

const passwordManager = new PasswordManager();
module.exports = passwordManager;
```

### **Password Reset System**
```javascript
const crypto = require('crypto');
const nodemailer = require('nodemailer');

class PasswordResetManager {
  constructor() {
    this.resetTokenExpiry = 10 * 60 * 1000; // 10 minutes
  }

  // Generate password reset token
  generateResetToken() {
    const resetToken = crypto.randomBytes(32).toString('hex');
    const hashedToken = crypto.createHash('sha256').update(resetToken).digest('hex');

    return { resetToken, hashedToken };
  }

  // Send password reset email
  async sendResetEmail(email, resetToken) {
    const transporter = nodemailer.createTransporter({
      host: process.env.EMAIL_HOST,
      port: process.env.EMAIL_PORT,
      secure: false,
      auth: {
        user: process.env.EMAIL_USER,
        pass: process.env.EMAIL_PASS,
      },
    });

    const resetUrl = `${process.env.FRONTEND_URL}/reset-password/${resetToken}`;

    const mailOptions = {
      from: process.env.EMAIL_FROM,
      to: email,
      subject: 'Password Reset Request',
      html: `
        <h2>Password Reset Request</h2>
        <p>You requested a password reset for your account.</p>
        <p>Please click the link below to reset your password:</p>
        <a href="${resetUrl}" style="display: inline-block; padding: 10px 20px; background-color: #007bff; color: white; text-decoration: none; border-radius: 5px;">Reset Password</a>
        <p>This link will expire in 10 minutes.</p>
        <p>If you didn't request this, please ignore this email.</p>
      `,
    };

    await transporter.sendMail(mailOptions);
  }

  // Request password reset
  async requestPasswordReset(email) {
    const user = await User.findOne({ email });

    if (!user) {
      // Don't reveal if email exists or not
      return { success: true, message: 'If an account with that email exists, a reset link has been sent.' };
    }

    // Generate reset token
    const { resetToken, hashedToken } = this.generateResetToken();

    // Save hashed token to user
    user.passwordResetToken = hashedToken;
    user.passwordResetExpires = new Date(Date.now() + this.resetTokenExpiry);
    await user.save();

    // Send reset email
    try {
      await this.sendResetEmail(email, resetToken);
    } catch (error) {
      // Clean up token if email fails
      user.passwordResetToken = undefined;
      user.passwordResetExpires = undefined;
      await user.save();

      throw new Error('Failed to send reset email');
    }

    return { success: true, message: 'Password reset link sent to your email.' };
  }

  // Reset password
  async resetPassword(token, newPassword) {
    // Hash the token
    const hashedToken = crypto.createHash('sha256').update(token).digest('hex');

    // Find user with valid reset token
    const user = await User.findOne({
      passwordResetToken: hashedToken,
      passwordResetExpires: { $gt: new Date() },
    });

    if (!user) {
      throw new Error('Invalid or expired reset token');
    }

    // Update password
    await passwordManager.updatePassword(user.id, newPassword);

    // Clear reset token
    user.passwordResetToken = undefined;
    user.passwordResetExpires = undefined;
    await user.save();

    return { success: true, message: 'Password reset successfully' };
  }

  // Clean up expired tokens (run periodically)
  async cleanupExpiredTokens() {
    const result = await User.updateMany(
      { passwordResetExpires: { $lt: new Date() } },
      {
        $unset: {
          passwordResetToken: 1,
          passwordResetExpires: 1,
        },
      }
    );

    console.log(`Cleaned up ${result.modifiedCount} expired reset tokens`);
  }
}

const passwordResetManager = new PasswordResetManager();
module.exports = passwordResetManager;
```

---

## 👥 **Role-Based Access Control**

### **RBAC Implementation**
```javascript
// Permission definitions
const PERMISSIONS = {
  // User permissions
  USER_READ: 'user:read',
  USER_CREATE: 'user:create',
  USER_UPDATE: 'user:update',
  USER_DELETE: 'user:delete',

  // Post permissions
  POST_READ: 'post:read',
  POST_CREATE: 'post:create',
  POST_UPDATE: 'post:update',
  POST_DELETE: 'post:delete',
  POST_PUBLISH: 'post:publish',

  // Comment permissions
  COMMENT_READ: 'comment:read',
  COMMENT_CREATE: 'comment:create',
  COMMENT_UPDATE: 'comment:update',
  COMMENT_DELETE: 'comment:delete',

  // Admin permissions
  ADMIN_USER_MANAGE: 'admin:user:manage',
  ADMIN_POST_MANAGE: 'admin:post:manage',
  ADMIN_SYSTEM_MANAGE: 'admin:system:manage',
};

// Role definitions
const ROLES = {
  USER: [
    PERMISSIONS.USER_READ,
    PERMISSIONS.USER_UPDATE,
    PERMISSIONS.POST_READ,
    PERMISSIONS.POST_CREATE,
    PERMISSIONS.POST_UPDATE,
    PERMISSIONS.COMMENT_READ,
    PERMISSIONS.COMMENT_CREATE,
    PERMISSIONS.COMMENT_UPDATE,
  ],

  MODERATOR: [
    ...ROLES.USER,
    PERMISSIONS.POST_PUBLISH,
    PERMISSIONS.COMMENT_DELETE,
    PERMISSIONS.USER_UPDATE,
  ],

  ADMIN: [
    ...ROLES.MODERATOR,
    PERMISSIONS.USER_CREATE,
    PERMISSIONS.USER_DELETE,
    PERMISSIONS.POST_DELETE,
    PERMISSIONS.ADMIN_USER_MANAGE,
    PERMISSIONS.ADMIN_POST_MANAGE,
    PERMISSIONS.ADMIN_SYSTEM_MANAGE,
  ],
};

class RBACManager {
  constructor() {
    this.roles = ROLES;
    this.permissions = PERMISSIONS;
  }

  // Check if user has permission
  hasPermission(user, permission) {
    if (!user || !user.role) return false;

    const userPermissions = this.roles[user.role] || [];
    return userPermissions.includes(permission);
  }

  // Check if user has any of the permissions
  hasAnyPermission(user, permissions) {
    return permissions.some(permission => this.hasPermission(user, permission));
  }

  // Check if user has all permissions
  hasAllPermissions(user, permissions) {
    return permissions.every(permission => this.hasPermission(user, permission));
  }

  // Get user permissions
  getUserPermissions(user) {
    if (!user || !user.role) return [];
    return this.roles[user.role] || [];
  }

  // Add custom role
  addRole(roleName, permissions) {
    this.roles[roleName] = permissions;
  }

  // Add permission to role
  addPermissionToRole(roleName, permission) {
    if (!this.roles[roleName]) {
      this.roles[roleName] = [];
    }

    if (!this.roles[roleName].includes(permission)) {
      this.roles[roleName].push(permission);
    }
  }

  // Remove permission from role
  removePermissionFromRole(roleName, permission) {
    if (this.roles[roleName]) {
      this.roles[roleName] = this.roles[roleName].filter(p => p !== permission);
    }
  }
}

const rbacManager = new RBACManager();
module.exports = { rbacManager, PERMISSIONS };
```

### **RBAC Middleware**
```javascript
const { rbacManager, PERMISSIONS } = require('./rbacManager');

// Permission middleware
const requirePermission = (permission) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({
        success: false,
        error: 'Authentication required',
      });
    }

    if (!rbacManager.hasPermission(req.user, permission)) {
      return res.status(403).json({
        success: false,
        error: 'Insufficient permissions',
        required: permission,
        userRole: req.user.role,
      });
    }

    next();
  };
};

// Multiple permissions middleware
const requireAnyPermission = (permissions) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({
        success: false,
        error: 'Authentication required',
      });
    }

    if (!rbacManager.hasAnyPermission(req.user, permissions)) {
      return res.status(403).json({
        success: false,
        error: 'Insufficient permissions',
        required: permissions,
        userRole: req.user.role,
      });
    }

    next();
  };
};

// Resource ownership middleware
const requireOwnership = (resourceType, resourceIdParam = 'id') => {
  return async (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({
        success: false,
        error: 'Authentication required',
      });
    }

    const resourceId = req.params[resourceIdParam];
    let resource;

    try {
      switch (resourceType) {
        case 'user':
          resource = await User.findById(resourceId);
          break;
        case 'post':
          resource = await Post.findById(resourceId);
          break;
        case 'comment':
          resource = await Comment.findById(resourceId);
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
          error: `${resourceType} not found`,
        });
      }

      // Check ownership or admin role
      const isOwner = resource.author?.toString() === req.user.id ||
                     resource.user?.toString() === req.user.id;
      const isAdmin = req.user.role === 'admin';

      if (!isOwner && !isAdmin) {
        return res.status(403).json({
          success: false,
          error: 'Access denied',
        });
      }

      // Attach resource to request
      req[resourceType] = resource;
      next();
    } catch (error) {
      res.status(500).json({
        success: false,
        error: 'Server error',
      });
    }
  };
};

// Usage in routes
const { authenticateToken } = require('./authMiddleware');
const { requirePermission, requireAnyPermission, requireOwnership } = require('./rbacMiddleware');

// User routes
router.get('/users', authenticateToken, requirePermission(PERMISSIONS.USER_READ));
router.post('/users', requirePermission(PERMISSIONS.USER_CREATE));
router.put('/users/:id', authenticateToken, requireOwnership('user'));
router.delete('/users/:id', authenticateToken, requireAnyPermission([
  PERMISSIONS.USER_DELETE,
  PERMISSIONS.ADMIN_USER_MANAGE,
]));

// Post routes
router.get('/posts', authenticateToken, requirePermission(PERMISSIONS.POST_READ));
router.post('/posts', authenticateToken, requirePermission(PERMISSIONS.POST_CREATE));
router.put('/posts/:id', authenticateToken, requireOwnership('post'));
router.delete('/posts/:id', authenticateToken, requireAnyPermission([
  PERMISSIONS.POST_DELETE,
  PERMISSIONS.ADMIN_POST_MANAGE,
]));
```

---

## 🔑 **OAuth Integration**

### **OAuth 2.0 Implementation**
```javascript
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;
const FacebookStrategy = require('passport-facebook').Strategy;
const GitHubStrategy = require('passport-github2').Strategy;

class OAuthManager {
  constructor() {
    this.configurePassport();
  }

  configurePassport() {
    // Serialize user for session
    passport.serializeUser((user, done) => {
      done(null, user.id);
    });

    // Deserialize user from session
    passport.deserializeUser(async (id, done) => {
      try {
        const user = await User.findById(id);
        done(null, user);
      } catch (error) {
        done(error, null);
      }
    });

    // Google OAuth Strategy
    passport.use(new GoogleStrategy({
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: '/auth/google/callback',
    }, this.handleGoogleAuth.bind(this)));

    // Facebook OAuth Strategy
    passport.use(new FacebookStrategy({
      clientID: process.env.FACEBOOK_APP_ID,
      clientSecret: process.env.FACEBOOK_APP_SECRET,
      callbackURL: '/auth/facebook/callback',
      profileFields: ['id', 'emails', 'name'],
    }, this.handleFacebookAuth.bind(this)));

    // GitHub OAuth Strategy
    passport.use(new GitHubStrategy({
      clientID: process.env.GITHUB_CLIENT_ID,
      clientSecret: process.env.GITHUB_CLIENT_SECRET,
      callbackURL: '/auth/github/callback',
    }, this.handleGitHubAuth.bind(this)));
  }

  async handleGoogleAuth(accessToken, refreshToken, profile, done) {
    try {
      await this.handleOAuthUser(profile, 'google', done);
    } catch (error) {
      done(error, null);
    }
  }

  async handleFacebookAuth(accessToken, refreshToken, profile, done) {
    try {
      await this.handleOAuthUser(profile, 'facebook', done);
    } catch (error) {
      done(error, null);
    }
  }

  async handleGitHubAuth(accessToken, refreshToken, profile, done) {
    try {
      await this.handleOAuthUser(profile, 'github', done);
    } catch (error) {
      done(error, null);
    }
  }

  async handleOAuthUser(profile, provider, done) {
    const email = profile.emails?.[0]?.value;
    const firstName = profile.name?.givenName || profile.displayName?.split(' ')[0];
    const lastName = profile.name?.familyName || profile.displayName?.split(' ').slice(1).join(' ');

    if (!email) {
      return done(new Error('Email not provided by OAuth provider'), null);
    }

    // Check if user exists
    let user = await User.findOne({ email });

    if (user) {
      // Update OAuth info if not already linked
      if (!user.oauth[provider]) {
        user.oauth[provider] = {
          id: profile.id,
          connectedAt: new Date(),
        };
        await user.save();
      }
      return done(null, user);
    }

    // Create new user
    user = await User.create({
      firstName,
      lastName,
      email,
      password: await passwordManager.hashPassword(crypto.randomBytes(32).toString('hex')),
      isVerified: true,
      oauth: {
        [provider]: {
          id: profile.id,
          connectedAt: new Date(),
        },
      },
    });

    done(null, user);
  }

  // Generate OAuth routes
  getRoutes() {
    const router = express.Router();

    // Google OAuth
    router.get('/google',
      passport.authenticate('google', { scope: ['profile', 'email'] })
    );

    router.get('/google/callback',
      passport.authenticate('google', { failureRedirect: '/login' }),
      (req, res) => {
        // Generate JWT token and redirect
        const { accessToken, refreshToken } = jwtManager.generateTokens(req.user);
        res.redirect(`${process.env.FRONTEND_URL}/auth/callback?token=${accessToken}&refreshToken=${refreshToken}`);
      }
    );

    // Facebook OAuth
    router.get('/facebook',
      passport.authenticate('facebook', { scope: ['email'] })
    );

    router.get('/facebook/callback',
      passport.authenticate('facebook', { failureRedirect: '/login' }),
      (req, res) => {
        const { accessToken, refreshToken } = jwtManager.generateTokens(req.user);
        res.redirect(`${process.env.FRONTEND_URL}/auth/callback?token=${accessToken}&refreshToken=${refreshToken}`);
      }
    );

    // GitHub OAuth
    router.get('/github',
      passport.authenticate('github', { scope: ['user:email'] })
    );

    router.get('/github/callback',
      passport.authenticate('github', { failureRedirect: '/login' }),
      (req, res) => {
        const { accessToken, refreshToken } = jwtManager.generateTokens(req.user);
        res.redirect(`${process.env.FRONTEND_URL}/auth/callback?token=${accessToken}&refreshToken=${refreshToken}`);
      }
    );

    return router;
  }
}

const oauthManager = new OAuthManager();
module.exports = oauthManager;
```

---

## 🛡️ **Security Best Practices**

### **Input Validation & Sanitization**
```javascript
const validator = require('validator');
const xss = require('xss');
const rateLimit = require('express-rate-limit');

class SecurityManager {
  // Sanitize user input
  sanitizeInput(input) {
    if (typeof input === 'string') {
      // Remove HTML tags and encode special characters
      return xss(validator.escape(input.trim()));
    }

    if (typeof input === 'object' && input !== null) {
      const sanitized = {};
      for (const [key, value] of Object.entries(input)) {
        sanitized[key] = this.sanitizeInput(value);
      }
      return sanitized;
    }

    return input;
  }

  // Validate email
  validateEmail(email) {
    return validator.isEmail(email, {
      allow_utf8_local_part: false,
      require_tld: true,
    });
  }

  // Validate URL
  validateUrl(url) {
    return validator.isURL(url, {
      protocols: ['http', 'https'],
      require_protocol: true,
    });
  }

  // Validate MongoDB ObjectId
  validateObjectId(id) {
    return validator.isMongoId(id);
  }

  // Rate limiting configurations
  getRateLimiters() {
    const authLimiter = rateLimit({
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: 5, // 5 attempts per window
      message: {
        success: false,
        error: 'Too many authentication attempts, please try again later',
      },
      standardHeaders: true,
      legacyHeaders: false,
      skipSuccessfulRequests: true,
    });

    const apiLimiter = rateLimit({
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: 100, // 100 requests per window
      message: {
        success: false,
        error: 'Too many requests, please try again later',
      },
    });

    const createLimiter = rateLimit({
      windowMs: 60 * 60 * 1000, // 1 hour
      max: 10, // 10 creations per hour
      message: {
        success: false,
        error: 'Creation limit exceeded, please try again later',
      },
    });

    return { authLimiter, apiLimiter, createLimiter };
  }

  // CORS configuration
  getCorsOptions() {
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];

    return {
      origin: function (origin, callback) {
        // Allow requests with no origin (mobile apps, etc.)
        if (!origin) return callback(null, true);

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
  }

  // Helmet security headers
  getSecurityHeaders() {
    return {
      contentSecurityPolicy: {
        directives: {
          defaultSrc: ["'self'"],
          styleSrc: ["'self'", "'unsafe-inline'"],
          scriptSrc: ["'self'"],
          imgSrc: ["'self'", 'data:', 'https:'],
          fontSrc: ["'self'", 'https:', 'data:'],
          connectSrc: ["'self'", 'https:'],
          objectSrc: ["'none'"],
          upgradeInsecureRequests: [],
        },
      },
      hsts: {
        maxAge: 31536000,
        includeSubDomains: true,
        preload: true,
      },
      noSniff: true,
      xssFilter: true,
      referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
    };
  }
}

const securityManager = new SecurityManager();
module.exports = securityManager;
```

---

## 🚨 **Common Vulnerabilities**

### **SQL Injection Prevention**
```javascript
// ❌ Vulnerable to SQL injection
app.get('/users/:id', (req, res) => {
  const query = `SELECT * FROM users WHERE id = ${req.params.id}`;
  db.query(query, (err, result) => {
    res.json(result);
  });
});

// ✅ Safe from SQL injection
app.get('/users/:id', (req, res) => {
  const query = 'SELECT * FROM users WHERE id = ?';
  db.query(query, [req.params.id], (err, result) => {
    res.json(result);
  });
});

// For MongoDB
// ❌ Vulnerable to NoSQL injection
const user = await User.findOne({ email: req.body.email });

// ✅ Safe from NoSQL injection
const user = await User.findOne({
  email: validator.escape(req.body.email)
});
```

### **XSS Prevention**
```javascript
// ❌ Vulnerable to XSS
app.get('/search', (req, res) => {
  const query = req.query.q;
  res.send(`<h1>Search results for: ${query}</h1>`);
});

// ✅ Safe from XSS
const xss = require('xss');
app.get('/search', (req, res) => {
  const query = xss(req.query.q);
  res.send(`<h1>Search results for: ${query}</h1>`);
});
```

### **CSRF Protection**
```javascript
const csrf = require('csurf');

// CSRF protection middleware
const csrfProtection = csrf({
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
  },
});

// Add CSRF token to responses
app.use((req, res, next) => {
  res.locals.csrfToken = req.csrfToken ? req.csrfToken() : null;
  next();
});

// Protect sensitive routes
app.post('/users', csrfProtection, createUser);
app.put('/users/:id', csrfProtection, updateUser);
app.delete('/users/:id', csrfProtection, deleteUser);
```

---

## 🔐 **API Security**

### **API Key Management**
```javascript
class APIKeyManager {
  constructor() {
    this.keys = new Map(); // In production, use Redis or database
  }

  // Generate API key
  generateKey(userId, permissions = ['read']) {
    const apiKey = crypto.randomBytes(32).toString('hex');
    const hashedKey = crypto.createHash('sha256').update(apiKey).digest('hex');

    this.keys.set(hashedKey, {
      userId,
      permissions,
      createdAt: new Date(),
      lastUsed: null,
      isActive: true,
    });

    return apiKey; // Return unhashed key to user
  }

  // Verify API key
  async verifyKey(apiKey) {
    const hashedKey = crypto.createHash('sha256').update(apiKey).digest('hex');
    const keyData = this.keys.get(hashedKey);

    if (!keyData || !keyData.isActive) {
      return null;
    }

    // Update last used
    keyData.lastUsed = new Date();
    this.keys.set(hashedKey, keyData);

    return keyData;
  }

  // Revoke API key
  revokeKey(apiKey) {
    const hashedKey = crypto.createHash('sha256').update(apiKey).digest('hex');
    const keyData = this.keys.get(hashedKey);

    if (keyData) {
      keyData.isActive = false;
      this.keys.set(hashedKey, keyData);
      return true;
    }

    return false;
  }

  // Get user's API keys
  getUserKeys(userId) {
    const userKeys = [];

    for (const [hashedKey, keyData] of this.keys.entries()) {
      if (keyData.userId === userId) {
        userKeys.push({
          id: hashedKey,
          permissions: keyData.permissions,
          createdAt: keyData.createdAt,
          lastUsed: keyData.lastUsed,
          isActive: keyData.isActive,
        });
      }
    }

    return userKeys;
  }
}

const apiKeyManager = new APIKeyManager();

// API key authentication middleware
const authenticateApiKey = async (req, res, next) => {
  const apiKey = req.headers['x-api-key'] || req.query.apiKey;

  if (!apiKey) {
    return res.status(401).json({
      success: false,
      error: 'API key required',
    });
  }

  const keyData = await apiKeyManager.verifyKey(apiKey);

  if (!keyData) {
    return res.status(401).json({
      success: false,
      error: 'Invalid or inactive API key',
    });
  }

  // Attach user and permissions to request
  req.apiUser = { id: keyData.userId };
  req.apiPermissions = keyData.permissions;

  next();
};

// Permission check middleware
const requireApiPermission = (permission) => {
  return (req, res, next) => {
    if (!req.apiPermissions || !req.apiPermissions.includes(permission)) {
      return res.status(403).json({
        success: false,
        error: 'Insufficient API permissions',
        required: permission,
        available: req.apiPermissions,
      });
    }

    next();
  };
};
```

---

## 🎯 **Practical Examples**

### **Complete Authentication System**
```javascript
const express = require('express');
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const crypto = require('crypto');

const app = express();
app.use(express.json());

// In-memory storage (replace with database in production)
const users = [];
const refreshTokens = [];

// Register endpoint
app.post('/auth/register', async (req, res) => {
  try {
    const { email, password, firstName, lastName } = req.body;

    // Check if user exists
    const existingUser = users.find(u => u.email === email);
    if (existingUser) {
      return res.status(409).json({ error: 'User already exists' });
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(password, 12);

    // Create user
    const user = {
      id: users.length + 1,
      email,
      password: hashedPassword,
      firstName,
      lastName,
      role: 'user',
      isVerified: false,
      createdAt: new Date(),
    };

    users.push(user);

    // Generate tokens
    const accessToken = jwt.sign(
      { id: user.id, email: user.email, role: user.role },
      'your-secret-key',
      { expiresIn: '15m' }
    );

    const refreshToken = jwt.sign(
      { id: user.id },
      'your-refresh-secret-key',
      { expiresIn: '7d' }
    );

    refreshTokens.push(refreshToken);

    res.status(201).json({
      user: {
        id: user.id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
        role: user.role,
      },
      accessToken,
      refreshToken,
    });
  } catch (error) {
    res.status(500).json({ error: 'Registration failed' });
  }
});

// Login endpoint
app.post('/auth/login', async (req, res) => {
  try {
    const { email, password } = req.body;

    const user = users.find(u => u.email === email);
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    const isValidPassword = await bcrypt.compare(password, user.password);
    if (!isValidPassword) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    const accessToken = jwt.sign(
      { id: user.id, email: user.email, role: user.role },
      'your-secret-key',
      { expiresIn: '15m' }
    );

    const refreshToken = jwt.sign(
      { id: user.id },
      'your-refresh-secret-key',
      { expiresIn: '7d' }
    );

    refreshTokens.push(refreshToken);

    res.json({
      user: {
        id: user.id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
        role: user.role,
      },
      accessToken,
      refreshToken,
    });
  } catch (error) {
    res.status(500).json({ error: 'Login failed' });
  }
});

// Refresh token endpoint
app.post('/auth/refresh', (req, res) => {
  const { refreshToken } = req.body;

  if (!refreshToken || !refreshTokens.includes(refreshToken)) {
    return res.status(403).json({ error: 'Invalid refresh token' });
  }

  jwt.verify(refreshToken, 'your-refresh-secret-key', (err, decoded) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid refresh token' });
    }

    const user = users.find(u => u.id === decoded.id);
    if (!user) {
      return res.status(403).json({ error: 'User not found' });
    }

    const accessToken = jwt.sign(
      { id: user.id, email: user.email, role: user.role },
      'your-secret-key',
      { expiresIn: '15m' }
    );

    res.json({ accessToken });
  });
});

// Protected route middleware
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }

  jwt.verify(token, 'your-secret-key', (err, user) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid token' });
    }

    req.user = user;
    next();
  });
};

// Protected route
app.get('/api/profile', authenticateToken, (req, res) => {
  const user = users.find(u => u.id === req.user.id);
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }

  res.json({
    id: user.id,
    email: user.email,
    firstName: user.firstName,
    lastName: user.lastName,
    role: user.role,
  });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Authentication Types**: Session-based, token-based, and API key authentication
- ✅ **JWT Implementation**: Token generation, verification, and refresh mechanisms
- ✅ **Password Security**: Hashing, validation, and reset functionality
- ✅ **Role-Based Access Control**: Permission systems and middleware
- ✅ **OAuth Integration**: Social login with Google, Facebook, and GitHub
- ✅ **Security Best Practices**: Input validation, rate limiting, and CORS
- ✅ **Common Vulnerabilities**: Prevention of SQL injection, XSS, and CSRF
- ✅ **API Security**: API key management and request authentication

### **Best Practices**
1. **Always hash passwords** using strong algorithms like bcrypt
2. **Use JWT tokens** with appropriate expiration times
3. **Implement refresh tokens** for better security
4. **Validate all inputs** and sanitize user data
5. **Use HTTPS** in production environments
6. **Implement rate limiting** to prevent abuse
7. **Log security events** for monitoring and debugging
8. **Regularly update dependencies** to patch security vulnerabilities

### **Next Steps**
- Implement multi-factor authentication (MFA)
- Add biometric authentication support
- Create comprehensive audit logging
- Implement API rate limiting with Redis
- Add security headers and CSP policies
- Set up automated security testing

---

## 🎯 **Assignment**

### **Task 1: Complete Authentication System**
Create a full-featured authentication system with:
- User registration and login
- JWT token-based authentication
- Password reset functionality
- Email verification
- Role-based access control
- Social login integration

### **Task 2: Secure API Implementation**
Build a secure REST API with:
- Input validation and sanitization
- Rate limiting and CORS configuration
- Comprehensive error handling
- API documentation with Swagger
- Security headers and best practices

### **Task 3: OAuth Integration**
Implement OAuth authentication with:
- Google OAuth 2.0 integration
- Facebook login
- GitHub authentication
- Token management and refresh
- User profile synchronization

---

## 📚 **Additional Resources**
- [OWASP Security Guidelines](https://owasp.org/www-project-top-ten/)
- [JWT.io](https://jwt.io/) - JWT debugger and library
- [bcrypt](https://www.npmjs.com/package/bcrypt) - Password hashing
- [Passport.js](http://www.passportjs.org/) - Authentication middleware
- [Helmet](https://helmetjs.github.io/) - Security headers
- [Express Rate Limit](https://www.npmjs.com/package/express-rate-limit)

---

**Next Lesson**: [Lesson 25: Real-time Communication (WebSockets & Socket.io)](Lesson%2025_%20Real-time%20Communication%20(WebSockets%20&%20Socket.io).md)