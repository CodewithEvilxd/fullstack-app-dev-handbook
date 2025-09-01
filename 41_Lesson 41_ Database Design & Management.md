# Lesson 41: Database Design & Management

## 🎯 **Learning Objectives**
- Master database design principles and normalization
- Implement efficient data modeling for React Native applications
- Understand different database types and their use cases
- Design scalable database schemas with proper relationships
- Implement database migrations and version control
- Optimize database queries and performance
- Handle database security and access control
- Implement data backup and recovery strategies

## 📚 **Table of Contents**
1. [Database Design Principles](#database-design-principles)
2. [Data Modeling & Normalization](#data-modeling--normalization)
3. [Database Types & Selection](#database-types--selection)
4. [Schema Design & Relationships](#schema-design--relationships)
5. [Indexing & Performance](#indexing--performance)
6. [Database Security](#database-security)
7. [Backup & Recovery](#backup--recovery)
8. [Database Migration](#database-migration)
9. [ORM & Query Optimization](#orm--query-optimization)
10. [Practical Examples](#practical-examples)

---

## 📋 **Database Design Principles**

### **Database Design Fundamentals**
```sql
-- Good database design principles
-- 1. Atomicity: Each field contains indivisible data
-- 2. Consistency: Data remains consistent across relationships
-- 3. Isolation: Concurrent operations don't interfere
-- 4. Durability: Committed data survives system failures

-- Example: User table with proper design
CREATE TABLE users (
    id VARCHAR(36) PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20),
    date_of_birth DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    email_verified BOOLEAN DEFAULT FALSE,
    last_login_at TIMESTAMP NULL,

    -- Constraints
    CONSTRAINT chk_email_format CHECK (email LIKE '%@%'),
    CONSTRAINT chk_phone_format CHECK (phone REGEXP '^[+]?[0-9 -()]+$'),
    INDEX idx_email (email),
    INDEX idx_created_at (created_at),
    INDEX idx_last_login (last_login_at)
);
```

### **Entity-Relationship Modeling**
```sql
-- Entity-Relationship Diagram representation
-- Users can have many posts (One-to-Many)
-- Users can follow many users (Many-to-Many)
-- Posts can have many comments (One-to-Many)
-- Posts can have many tags (Many-to-Many)

-- User entity
CREATE TABLE users (
    id VARCHAR(36) PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    -- ... other fields
);

-- Post entity
CREATE TABLE posts (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    published_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_id (user_id),
    INDEX idx_published_at (published_at)
);

-- Comment entity
CREATE TABLE comments (
    id VARCHAR(36) PRIMARY KEY,
    post_id VARCHAR(36) NOT NULL,
    user_id VARCHAR(36) NOT NULL,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_post_id (post_id),
    INDEX idx_user_id (user_id)
);

-- Follow relationship (Many-to-Many)
CREATE TABLE user_follows (
    follower_id VARCHAR(36) NOT NULL,
    following_id VARCHAR(36) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (follower_id, following_id),
    FOREIGN KEY (follower_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (following_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_follower (follower_id),
    INDEX idx_following (following_id)
);

-- Tag entity
CREATE TABLE tags (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    slug VARCHAR(50) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Post-Tag relationship (Many-to-Many)
CREATE TABLE post_tags (
    post_id VARCHAR(36) NOT NULL,
    tag_id VARCHAR(36) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (post_id, tag_id),
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE,
    INDEX idx_post_id (post_id),
    INDEX idx_tag_id (tag_id)
);
```

---

## 🔄 **Data Modeling & Normalization**

### **Normalization Forms**
```sql
-- First Normal Form (1NF): Atomic values, no repeating groups
-- Bad: Repeating phone numbers in one field
CREATE TABLE users_bad (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    phones VARCHAR(500) -- "123-456-7890,098-765-4321,555-123-4567"
);

-- Good: Separate table for phone numbers
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE user_phones (
    id INT PRIMARY KEY,
    user_id INT,
    phone_type ENUM('home', 'work', 'mobile'),
    phone_number VARCHAR(20),
    is_primary BOOLEAN DEFAULT FALSE,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE KEY unique_primary (user_id, is_primary)
);

-- Second Normal Form (2NF): No partial dependencies
-- Third Normal Form (3NF): No transitive dependencies
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    customer_email VARCHAR(255),
    product_name VARCHAR(200),
    product_price DECIMAL(10,2),
    quantity INT,
    order_date DATE,
    total_amount DECIMAL(10,2),

    -- This violates 3NF because product_price depends on product_name
    -- and product_name depends on order_id
);

-- Normalized version
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255) UNIQUE
);

CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(200),
    price DECIMAL(10,2)
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2),

    FOREIGN KEY (customer_id) REFERENCES customers(id)
);

CREATE TABLE order_items (
    id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2),

    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

### **Denormalization for Performance**
```sql
-- Sometimes denormalization is necessary for performance
-- Example: Storing user name in posts for faster queries

CREATE TABLE posts (
    id INT PRIMARY KEY,
    user_id INT,
    author_name VARCHAR(100), -- Denormalized for performance
    title VARCHAR(200),
    content TEXT,
    created_at TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_author_name (author_name)
);

-- Update denormalized data with triggers
DELIMITER //
CREATE TRIGGER update_post_author_name
    AFTER UPDATE ON users
    FOR EACH ROW
BEGIN
    UPDATE posts
    SET author_name = NEW.name
    WHERE user_id = NEW.id;
END;
//
DELIMITER ;

-- Or use application-level updates
-- When user updates their name, also update all their posts
async function updateUserName(userId, newName) {
    // Update user
    await User.update({ name: newName }, { where: { id: userId } });

    // Update denormalized data
    await Post.update(
        { author_name: newName },
        { where: { user_id: userId } }
    );
}
```

---

## 💾 **Database Types & Selection**

### **Relational Databases (SQL)**
```sql
-- MySQL/PostgreSQL/SQLite for structured data
-- Best for: Complex queries, transactions, relationships

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    category_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (category_id) REFERENCES categories(id),
    INDEX idx_category_price (category_id, price),
    FULLTEXT idx_search (name, description)
);

-- Complex queries with JOINs
SELECT
    p.name,
    p.price,
    c.name as category_name,
    COUNT(o.id) as order_count
FROM products p
LEFT JOIN categories c ON p.category_id = c.id
LEFT JOIN order_items oi ON p.id = oi.product_id
LEFT JOIN orders o ON oi.order_id = o.id
WHERE p.price BETWEEN 10 AND 100
GROUP BY p.id, p.name, p.price, c.name
HAVING COUNT(o.id) > 0
ORDER BY p.price ASC;
```

### **NoSQL Databases**
```javascript
// MongoDB for flexible schemas and JSON-like data
// Best for: Rapid development, changing schemas, nested data

const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
    name: {
        type: String,
        required: true,
        index: true,
        text: true // For text search
    },
    description: {
        type: String,
        text: true
    },
    price: {
        type: Number,
        required: true,
        min: 0
    },
    category: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Category',
        required: true
    },
    images: [{
        url: String,
        alt: String,
        isPrimary: Boolean
    }],
    variants: [{
        name: String,
        value: String,
        priceModifier: Number
    }],
    inventory: {
        quantity: Number,
        lowStockThreshold: { type: Number, default: 10 },
        sku: String
    },
    seo: {
        metaTitle: String,
        metaDescription: String,
        slug: { type: String, unique: true }
    },
    tags: [String],
    reviews: [{
        user: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
        rating: { type: Number, min: 1, max: 5 },
        comment: String,
        createdAt: { type: Date, default: Date.now }
    }],
    isActive: { type: Boolean, default: true }
}, {
    timestamps: true
});

// Indexes for performance
productSchema.index({ category: 1, price: 1 });
productSchema.index({ 'inventory.quantity': 1 });
productSchema.index({ tags: 1 });
productSchema.index({ 'seo.slug': 1 });

// Virtual for average rating
productSchema.virtual('averageRating').get(function() {
    if (this.reviews.length === 0) return 0;
    const sum = this.reviews.reduce((acc, review) => acc + review.rating, 0);
    return sum / this.reviews.length;
});

// Instance method
productSchema.methods.isLowStock = function() {
    return this.inventory.quantity <= this.inventory.lowStockThreshold;
};

// Static method
productSchema.statics.findByCategory = function(categoryId) {
    return this.find({ category: categoryId, isActive: true });
};

const Product = mongoose.model('Product', productSchema);

// Usage
const product = new Product({
    name: 'Wireless Headphones',
    description: 'High-quality wireless headphones',
    price: 99.99,
    category: categoryId,
    inventory: { quantity: 50, sku: 'WH-001' }
});

await product.save();
```

### **Choosing the Right Database**
```javascript
// Database selection guide
const databaseGuide = {
    // Use SQL when you need:
    sql: {
        useCases: [
            'Complex relationships between entities',
            'ACID transactions are critical',
            'Data integrity is paramount',
            'Complex queries with JOINs',
            'Structured data with fixed schema'
        ],
        examples: ['PostgreSQL', 'MySQL', 'SQLite'],
        bestFor: 'E-commerce, banking, ERP systems'
    },

    // Use NoSQL when you need:
    nosql: {
        useCases: [
            'Flexible schema that changes frequently',
            'High write/read throughput',
            'Large amounts of unstructured data',
            'Real-time analytics',
            'Geographic data'
        ],
        examples: ['MongoDB', 'Cassandra', 'Redis'],
        bestFor: 'Content management, IoT, real-time apps'
    },

    // Use NewSQL when you need:
    newsql: {
        useCases: [
            'SQL interface with NoSQL scalability',
            'Distributed transactions',
            'High availability',
            'Global data distribution'
        ],
        examples: ['CockroachDB', 'TiDB', 'YugabyteDB'],
        bestFor: 'Global applications, high-scale systems'
    }
};

// Database decision tree
function recommendDatabase(requirements) {
    const { dataStructure, scalability, consistency, queryComplexity } = requirements;

    if (queryComplexity === 'high' && consistency === 'strong') {
        return 'PostgreSQL';
    }

    if (scalability === 'high' && dataStructure === 'flexible') {
        return 'MongoDB';
    }

    if (scalability === 'global' && consistency === 'strong') {
        return 'CockroachDB';
    }

    return 'PostgreSQL'; // Default
}
```

---

## 🔗 **Schema Design & Relationships**

### **Advanced Relationships**
```sql
-- One-to-One relationship
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE user_profiles (
    user_id INT PRIMARY KEY,
    bio TEXT,
    avatar_url VARCHAR(500),
    website VARCHAR(255),

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- One-to-Many relationship
CREATE TABLE categories (
    id INT PRIMARY KEY,
    name VARCHAR(100) UNIQUE,
    parent_id INT NULL,

    FOREIGN KEY (parent_id) REFERENCES categories(id),
    INDEX idx_parent (parent_id)
);

-- Many-to-Many with additional data
CREATE TABLE user_groups (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    description TEXT
);

CREATE TABLE user_group_memberships (
    user_id INT,
    group_id INT,
    role ENUM('member', 'moderator', 'admin') DEFAULT 'member',
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (user_id, group_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (group_id) REFERENCES user_groups(id) ON DELETE CASCADE,
    INDEX idx_user_group (user_id, group_id)
);

-- Polymorphic relationships
CREATE TABLE comments (
    id INT PRIMARY KEY,
    content TEXT,
    author_id INT,
    commentable_type ENUM('post', 'video', 'photo'),
    commentable_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (author_id) REFERENCES users(id),
    INDEX idx_commentable (commentable_type, commentable_id)
);
```

### **Database Constraints**
```sql
-- Check constraints
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    discount_percent DECIMAL(5,2) DEFAULT 0,

    CONSTRAINT chk_price_positive CHECK (price > 0),
    CONSTRAINT chk_discount_range CHECK (discount_percent >= 0 AND discount_percent <= 100),
    CONSTRAINT chk_name_length CHECK (LENGTH(name) >= 2)
);

-- Unique constraints
CREATE TABLE employees (
    id INT PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    employee_id VARCHAR(10) UNIQUE,
    department_id INT,
    manager_id INT,

    FOREIGN KEY (department_id) REFERENCES departments(id),
    FOREIGN KEY (manager_id) REFERENCES employees(id),

    -- Composite unique constraint
    CONSTRAINT unique_dept_manager UNIQUE (department_id, manager_id)
);

-- Triggers for data integrity
DELIMITER //
CREATE TRIGGER prevent_self_follow
    BEFORE INSERT ON user_follows
    FOR EACH ROW
BEGIN
    IF NEW.follower_id = NEW.following_id THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Users cannot follow themselves';
    END IF;
END;
//
DELIMITER ;

-- Soft delete with triggers
CREATE TABLE posts (
    id INT PRIMARY KEY,
    title VARCHAR(255),
    content TEXT,
    deleted_at TIMESTAMP NULL,
    INDEX idx_deleted_at (deleted_at)
);

DELIMITER //
CREATE TRIGGER soft_delete_posts
    BEFORE DELETE ON posts
    FOR EACH ROW
BEGIN
    UPDATE posts SET deleted_at = NOW() WHERE id = OLD.id;
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Use soft delete instead';
END;
//
DELIMITER ;
```

---

## 📊 **Indexing & Performance**

### **Index Strategies**
```sql
-- Single column indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_created_at ON posts(created_at);
CREATE INDEX idx_products_price ON products(price);

-- Composite indexes
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
CREATE INDEX idx_posts_author_status ON posts(author_id, status);

-- Partial indexes
CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;
CREATE INDEX idx_recent_posts ON posts(created_at) WHERE created_at > '2023-01-01';

-- Functional indexes
CREATE INDEX idx_users_name_lower ON users(LOWER(name));
CREATE INDEX idx_posts_title_search ON posts USING gin(to_tsvector('english', title));

-- Covering indexes (include all columns needed for query)
CREATE INDEX idx_user_posts_covering ON posts(user_id, created_at, title)
WHERE status = 'published';

-- Index maintenance
-- Analyze index usage
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Remove unused indexes
DROP INDEX IF EXISTS unused_index_name;

-- Rebuild indexes
REINDEX INDEX idx_users_email;
```

### **Query Optimization**
```sql
-- Bad query: No indexes, full table scan
SELECT * FROM users WHERE created_at > '2023-01-01';
-- Solution: Add index
CREATE INDEX idx_users_created_at ON users(created_at);

-- Bad query: Using function on indexed column
SELECT * FROM users WHERE YEAR(created_at) = 2023;
-- Solution: Use range query
SELECT * FROM users WHERE created_at >= '2023-01-01' AND created_at < '2024-01-01';

-- Bad query: SELECT * with large tables
SELECT * FROM posts WHERE user_id = 123;
-- Solution: Select only needed columns
SELECT id, title, created_at FROM posts WHERE user_id = 123;

-- Bad query: Multiple subqueries
SELECT u.name,
       (SELECT COUNT(*) FROM posts WHERE user_id = u.id) as post_count,
       (SELECT COUNT(*) FROM comments WHERE user_id = u.id) as comment_count
FROM users u;
-- Solution: Use JOINs
SELECT u.name, COUNT(DISTINCT p.id) as post_count, COUNT(DISTINCT c.id) as comment_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
LEFT JOIN comments c ON u.id = c.user_id
GROUP BY u.id, u.name;

-- Pagination optimization
-- Bad: OFFSET with large numbers
SELECT * FROM posts ORDER BY created_at DESC LIMIT 10 OFFSET 10000;
-- Solution: Use cursor-based pagination
SELECT * FROM posts
WHERE created_at < '2023-06-01' -- cursor
ORDER BY created_at DESC
LIMIT 10;
```

---

## 🔒 **Database Security**

### **Access Control**
```sql
-- Create roles with specific permissions
CREATE ROLE app_user;
CREATE ROLE app_admin;
CREATE ROLE app_readonly;

-- Grant permissions
GRANT SELECT, INSERT, UPDATE ON users TO app_user;
GRANT SELECT ON sensitive_data TO app_readonly;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO app_admin;

-- Row Level Security (PostgreSQL)
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Policy for users to only see their own data
CREATE POLICY user_own_data ON users
    FOR ALL USING (auth.uid() = id);

-- Policy for admins to see all data
CREATE POLICY admin_all_data ON users
    FOR ALL USING (
        EXISTS (
            SELECT 1 FROM user_roles
            WHERE user_id = auth.uid()
            AND role = 'admin'
        )
    );

-- Data encryption
-- Encrypt sensitive data
CREATE EXTENSION IF NOT EXISTS pgcrypto;

CREATE TABLE user_credentials (
    user_id INT PRIMARY KEY,
    encrypted_password TEXT,
    encryption_key_id VARCHAR(36),

    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Encrypt data before storing
INSERT INTO user_credentials (user_id, encrypted_password, encryption_key_id)
VALUES (
    1,
    pgp_sym_encrypt('mypassword', 'encryption_key'),
    'key-123'
);

-- Decrypt data when retrieving
SELECT
    user_id,
    pgp_sym_decrypt(encrypted_password, 'encryption_key') as password
FROM user_credentials
WHERE user_id = 1;
```

### **SQL Injection Prevention**
```javascript
// Bad: Direct string concatenation (VULNERABLE)
const userId = req.params.id;
const query = `SELECT * FROM users WHERE id = ${userId}`;

// Good: Parameterized queries
// Using mysql2
const mysql = require('mysql2/promise');
const pool = mysql.createPool(config);

async function getUser(userId) {
    const [rows] = await pool.execute(
        'SELECT * FROM users WHERE id = ?',
        [userId]
    );
    return rows[0];
}

// Using PostgreSQL
const { Client } = require('pg');
const client = new Client(config);

async function getUser(userId) {
    const result = await client.query(
        'SELECT * FROM users WHERE id = $1',
        [userId]
    );
    return result.rows[0];
}

// Using MongoDB (safe by default)
const user = await User.findById(userId);

// ORM protection (Prisma, Sequelize, etc.)
const user = await prisma.user.findUnique({
    where: { id: userId }
});

// Input validation
const Joi = require('joi');

const userSchema = Joi.object({
    email: Joi.string().email().required(),
    name: Joi.string().min(2).max(100).required(),
    age: Joi.number().integer().min(0).max(150)
});

const { error, value } = userSchema.validate(req.body);
if (error) {
    return res.status(400).json({ error: error.details[0].message });
}
```

---

## 💾 **Backup & Recovery**

### **Backup Strategies**
```bash
# PostgreSQL backup
# Full database backup
pg_dump -U username -h localhost database_name > backup.sql

# Compressed backup
pg_dump -U username -h localhost database_name | gzip > backup.sql.gz

# Custom format backup (allows parallel restore)
pg_dump -U username -h localhost -Fc database_name > backup.dump

# Continuous archiving (Point-in-Time Recovery)
# In postgresql.conf:
# wal_level = replica
# archive_mode = on
# archive_command = 'cp %p /var/lib/postgresql/archive/%f'

# MongoDB backup
# mongodump command
mongodump --db mydatabase --out /backup/mongodb

# With authentication
mongodump --db mydatabase --username admin --password secret --out /backup

# MySQL backup
# mysqldump command
mysqldump -u username -p database_name > backup.sql

# With specific tables
mysqldump -u username -p database_name table1 table2 > backup.sql

# Incremental backup
mysqlbackup --backup-dir=/backup --incremental backup
```

### **Recovery Procedures**
```bash
# PostgreSQL recovery
# Restore from SQL dump
psql -U username -h localhost database_name < backup.sql

# Restore from custom format
pg_restore -U username -h localhost -d database_name backup.dump

# Point-in-Time Recovery
# 1. Stop PostgreSQL
# 2. Move WAL files to archive directory
# 3. Create recovery.conf
# 4. Start PostgreSQL

# MongoDB recovery
# mongorestore command
mongorestore --db mydatabase /backup/mongodb/mydatabase

# With authentication
mongorestore --db mydatabase --username admin --password secret /backup

# MySQL recovery
# Restore from dump
mysql -u username -p database_name < backup.sql

# Point-in-Time Recovery
mysqlbinlog --start-datetime="2023-01-01 00:00:00" binlog.000001 | mysql -u root -p
```

### **Automated Backup System**
```javascript
// Automated backup system
const { spawn } = require('child_process');
const fs = require('fs').promises;
const path = require('path');

class DatabaseBackup {
    constructor(config) {
        this.config = config;
        this.backupDir = config.backupDir || './backups';
    }

    async createBackup(databaseType, databaseName) {
        const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
        const filename = `${databaseName}_${timestamp}.backup`;

        switch (databaseType) {
            case 'postgresql':
                return this.createPostgresBackup(databaseName, filename);
            case 'mongodb':
                return this.createMongoBackup(databaseName, filename);
            case 'mysql':
                return this.createMySQLBackup(databaseName, filename);
            default:
                throw new Error(`Unsupported database type: ${databaseType}`);
        }
    }

    async createPostgresBackup(databaseName, filename) {
        const outputPath = path.join(this.backupDir, filename);

        return new Promise((resolve, reject) => {
            const pgDump = spawn('pg_dump', [
                '-U', this.config.username,
                '-h', this.config.host || 'localhost',
                '-Fc', // Custom format
                databaseName
            ], {
                stdio: ['ignore', fs.openSync(outputPath, 'w'), 'pipe']
            });

            pgDump.on('close', (code) => {
                if (code === 0) {
                    resolve(outputPath);
                } else {
                    reject(new Error(`pg_dump failed with code ${code}`));
                }
            });

            pgDump.on('error', reject);
        });
    }

    async createMongoBackup(databaseName, filename) {
        const outputPath = path.join(this.backupDir, filename);

        return new Promise((resolve, reject) => {
            const mongoDump = spawn('mongodump', [
                '--db', databaseName,
                '--out', outputPath,
                '--username', this.config.username,
                '--password', this.config.password,
                '--host', this.config.host || 'localhost'
            ]);

            mongoDump.on('close', (code) => {
                if (code === 0) {
                    resolve(outputPath);
                } else {
                    reject(new Error(`mongodump failed with code ${code}`));
                }
            });

            mongoDump.on('error', reject);
        });
    }

    // Schedule automated backups
    scheduleBackup(databaseType, databaseName, cronExpression) {
        const cron = require('node-cron');

        cron.schedule(cronExpression, async () => {
            try {
                console.log(`Starting scheduled backup for ${databaseName}`);
                const backupPath = await this.createBackup(databaseType, databaseName);
                console.log(`Backup completed: ${backupPath}`);

                // Clean up old backups
                await this.cleanupOldBackups(databaseName, 30); // Keep 30 days

                // Upload to cloud storage
                await this.uploadToCloud(backupPath);

            } catch (error) {
                console.error('Scheduled backup failed:', error);
                // Send alert notification
            }
        });
    }

    async cleanupOldBackups(databaseName, daysToKeep) {
        const files = await fs.readdir(this.backupDir);
        const cutoffDate = new Date();
        cutoffDate.setDate(cutoffDate.getDate() - daysToKeep);

        const oldFiles = files.filter(file => {
            if (!file.startsWith(databaseName)) return false;

            const filePath = path.join(this.backupDir, file);
            const stats = await fs.stat(filePath);
            return stats.mtime < cutoffDate;
        });

        for (const file of oldFiles) {
            await fs.unlink(path.join(this.backupDir, file));
            console.log(`Deleted old backup: ${file}`);
        }
    }

    async uploadToCloud(localPath) {
        // Upload to AWS S3, Google Cloud Storage, etc.
        // Implementation depends on cloud provider
        console.log(`Uploading ${localPath} to cloud storage`);
    }
}

// Usage
const backup = new DatabaseBackup({
    username: 'admin',
    password: 'secret',
    host: 'localhost',
    backupDir: './backups'
});

// Schedule daily backup at 2 AM
backup.scheduleBackup('postgresql', 'myapp', '0 2 * * *');

// Manual backup
backup.createBackup('postgresql', 'myapp')
    .then(path => console.log('Backup created:', path))
    .catch(error => console.error('Backup failed:', error));
```

---

## 🔄 **Database Migration**

### **Migration System**
```javascript
// Database migration system
const fs = require('fs').promises;
const path = require('path');
const crypto = require('crypto');

class DatabaseMigration {
    constructor(db, migrationsDir = './migrations') {
        this.db = db;
        this.migrationsDir = migrationsDir;
        this.migrationsTable = 'schema_migrations';
    }

    async initialize() {
        // Create migrations table if it doesn't exist
        await this.db.query(`
            CREATE TABLE IF NOT EXISTS ${this.migrationsTable} (
                id VARCHAR(255) PRIMARY KEY,
                name VARCHAR(255) NOT NULL,
                executed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        `);
    }

    async createMigration(name) {
        const timestamp = Date.now();
        const filename = `${timestamp}_${name}.js`;

        const migrationTemplate = `
module.exports = {
    up: async (db) => {
        // Migration logic goes here
        await db.query(\`
            -- Add your migration SQL here
        \`);
    },

    down: async (db) => {
        // Rollback logic goes here
        await db.query(\`
            -- Add your rollback SQL here
        \`);
    }
};
        `;

        await fs.writeFile(
            path.join(this.migrationsDir, filename),
            migrationTemplate.trim()
        );

        console.log(`Created migration: ${filename}`);
        return filename;
    }

    async runMigrations() {
        await this.initialize();

        const executedMigrations = await this.getExecutedMigrations();
        const migrationFiles = await this.getMigrationFiles();

        for (const file of migrationFiles) {
            if (!executedMigrations.has(file.id)) {
                console.log(`Running migration: ${file.name}`);

                const migration = require(path.join(this.migrationsDir, file.name));

                try {
                    await migration.up(this.db);
                    await this.recordMigration(file.id, file.name);
                    console.log(`✅ Migration completed: ${file.name}`);
                } catch (error) {
                    console.error(`❌ Migration failed: ${file.name}`, error);
                    throw error;
                }
            }
        }
    }

    async rollbackMigrations(steps = 1) {
        const executedMigrations = await this.getExecutedMigrations();
        const migrationFiles = await this.getMigrationFiles();

        // Get last N executed migrations
        const migrationsToRollback = Array.from(executedMigrations.entries())
            .sort((a, b) => b[1].executed_at - a[1].executed_at)
            .slice(0, steps);

        for (const [id, migration] of migrationsToRollback.reverse()) {
            console.log(`Rolling back migration: ${migration.name}`);

            const migrationModule = require(path.join(this.migrationsDir, migration.name));

            try {
                await migrationModule.down(this.db);
                await this.removeMigrationRecord(id);
                console.log(`✅ Rollback completed: ${migration.name}`);
            } catch (error) {
                console.error(`❌ Rollback failed: ${migration.name}`, error);
                throw error;
            }
        }
    }

    async getExecutedMigrations() {
        const result = await this.db.query(
            `SELECT id, name, executed_at FROM ${this.migrationsTable} ORDER BY executed_at`
        );

        const migrations = new Map();
        result.rows.forEach(row => {
            migrations.set(row.id, {
                name: row.name,
                executed_at: new Date(row.executed_at)
            });
        });

        return migrations;
    }

    async getMigrationFiles() {
        const files = await fs.readdir(this.migrationsDir);

        return files
            .filter(file => file.endsWith('.js'))
            .map(file => {
                const [timestamp, ...nameParts] = file.replace('.js', '').split('_');
                return {
                    id: timestamp,
                    name: file,
                    timestamp: parseInt(timestamp)
                };
            })
            .sort((a, b) => a.timestamp - b.timestamp);
    }

    async recordMigration(id, name) {
        await this.db.query(
            `INSERT INTO ${this.migrationsTable} (id, name) VALUES (?, ?)`,
            [id, name]
        );
    }

    async removeMigrationRecord(id) {
        await this.db.query(
            `DELETE FROM ${this.migrationsTable} WHERE id = ?`,
            [id]
        );
    }

    async getMigrationStatus() {
        const executed = await this.getExecutedMigrations();
        const files = await this.getMigrationFiles();

        return files.map(file => ({
            id: file.id,
            name: file.name,
            executed: executed.has(file.id),
            executed_at: executed.get(file.id)?.executed_at
        }));
    }
}

// Example migration file
// 1672531200000_create_users_table.js
module.exports = {
    up: async (db) => {
        await db.query(`
            CREATE TABLE users (
                id VARCHAR(36) PRIMARY KEY,
                email VARCHAR(255) UNIQUE NOT NULL,
                name VARCHAR(100) NOT NULL,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        `);
    },

    down: async (db) => {
        await db.query(`DROP TABLE IF EXISTS users`);
    }
};

// Usage
const migration = new DatabaseMigration(db);
await migration.runMigrations();

// Create new migration
await migration.createMigration('add_user_roles');

// Rollback last migration
await migration.rollbackMigrations(1);

// Check status
const status = await migration.getMigrationStatus();
console.table(status);
```

---

## 🛠️ **ORM & Query Optimization**

### **Sequelize ORM**
```javascript
// Sequelize setup and models
const { Sequelize, DataTypes, Op } = require('sequelize');

const sequelize = new Sequelize('database', 'username', 'password', {
    host: 'localhost',
    dialect: 'mysql',
    logging: console.log,
    pool: {
        max: 5,
        min: 0,
        acquire: 30000,
        idle: 10000
    }
});

// User model
const User = sequelize.define('User', {
    id: {
        type: DataTypes.UUID,
        defaultValue: DataTypes.UUIDV4,
        primaryKey: true
    },
    email: {
        type: DataTypes.STRING,
        allowNull: false,
        unique: true,
        validate: {
            isEmail: true
        }
    },
    name: {
        type: DataTypes.STRING,
        allowNull: false
    },
    password: {
        type: DataTypes.STRING,
        allowNull: false
    },
    isActive: {
        type: DataTypes.BOOLEAN,
        defaultValue: true
    }
}, {
    indexes: [
        { fields: ['email'] },
        { fields: ['isActive'] },
        { fields: ['createdAt'] }
    ]
});

// Post model
const Post = sequelize.define('Post', {
    title: {
        type: DataTypes.STRING,
        allowNull: false
    },
    content: {
        type: DataTypes.TEXT,
        allowNull: false
    },
    published: {
        type: DataTypes.BOOLEAN,
        defaultValue: false
    }
});

// Define relationships
User.hasMany(Post, { foreignKey: 'userId', as: 'posts' });
Post.belongsTo(User, { foreignKey: 'userId', as: 'author' });

// Optimized queries
class UserService {
    // Efficient user lookup with posts
    async getUserWithPosts(userId) {
        return await User.findByPk(userId, {
            include: [{
                model: Post,
                as: 'posts',
                where: { published: true },
                required: false,
                attributes: ['id', 'title', 'createdAt']
            }],
            attributes: ['id', 'name', 'email', 'createdAt']
        });
    }

    // Paginated user search
    async searchUsers(query, page = 1, limit = 10) {
        const offset = (page - 1) * limit;

        const { count, rows } = await User.findAndCountAll({
            where: {
                [Op.or]: [
                    { name: { [Op.iLike]: `%${query}%` } },
                    { email: { [Op.iLike]: `%${query}%` } }
                ],
                isActive: true
            },
            attributes: ['id', 'name', 'email', 'createdAt'],
            limit,
            offset,
            order: [['name', 'ASC']]
        });

        return {
            users: rows,
            pagination: {
                page,
                limit,
                total: count,
                pages: Math.ceil(count / limit)
            }
        };
    }

    // Bulk operations
    async bulkUpdateUserStatus(userIds, isActive) {
        const [affectedRows] = await User.update(
            { isActive },
            {
                where: { id: { [Op.in]: userIds } },
                returning: true
            }
        );

        return affectedRows;
    }

    // Complex query with aggregations
    async getUserStats(userId) {
        const stats = await Post.findAll({
            where: { userId },
            attributes: [
                [sequelize.fn('COUNT', sequelize.col('id')), 'totalPosts'],
                [sequelize.fn('COUNT', sequelize.literal('CASE WHEN published = true THEN 1 END')), 'publishedPosts'],
                [sequelize.fn('MAX', sequelize.col('createdAt')), 'lastPostDate']
            ],
            raw: true
        });

        return stats[0];
    }
}

module.exports = { User, Post, UserService };
```

### **Mongoose ODM**
```javascript
// Mongoose setup and models
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    email: {
        type: String,
        required: true,
        unique: true,
        lowercase: true,
        trim: true
    },
    name: {
        type: String,
        required: true,
        trim: true
    },
    password: {
        type: String,
        required: true,
        select: false // Don't include in queries by default
    },
    profile: {
        bio: String,
        avatar: String,
        website: String,
        location: String
    },
    isActive: {
        type: Boolean,
        default: true
    },
    lastLogin: Date,
    loginCount: {
        type: Number,
        default: 0
    }
}, {
    timestamps: true
});

// Indexes
userSchema.index({ email: 1 });
userSchema.index({ 'profile.location': 1 });
userSchema.index({ createdAt: -1 });
userSchema.index({ lastLogin: -1 });

// Virtual for full name
userSchema.virtual('fullName').get(function() {
    return `${this.name.first} ${this.name.last}`;
});

// Instance methods
userSchema.methods = {
    // Check password
    async checkPassword(candidatePassword) {
        return await bcrypt.compare(candidatePassword, this.password);
    },

    // Update last login
    async updateLastLogin() {
        this.lastLogin = new Date();
        this.loginCount += 1;
        return this.save();
    },

    // Get public profile
    toPublicProfile() {
        return {
            id: this._id,
            name: this.name,
            profile: this.profile,
            createdAt: this.createdAt
        };
    }
};

// Static methods
userSchema.statics = {
    // Find active users
    findActive() {
        return this.find({ isActive: true });
    },

    // Find users by location
    findByLocation(location) {
        return this.find({ 'profile.location': new RegExp(location, 'i') });
    },

    // Get user statistics
    async getStats() {
        return this.aggregate([
            {
                $group: {
                    _id: null,
                    totalUsers: { $sum: 1 },
                    activeUsers: {
                        $sum: { $cond: ['$isActive', 1, 0] }
                    },
                    averageLoginCount: { $avg: '$loginCount' }
                }
            }
        ]);
    }
};

// Post schema with population
const postSchema = new mongoose.Schema({
    title: {
        type: String,
        required: true,
        trim: true
    },
    content: {
        type: String,
        required: true
    },
    author: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: true
    },
    tags: [{
        type: String,
        trim: true,
        lowercase: true
    }],
    published: {
        type: Boolean,
        default: false
    },
    publishedAt: Date,
    views: {
        type: Number,
        default: 0
    },
    likes: [{
        user: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
        createdAt: { type: Date, default: Date.now }
    }],
    comments: [{
        user: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
        content: String,
        createdAt: { type: Date, default: Date.now }
    }]
}, {
    timestamps: true
});

// Indexes for performance
postSchema.index({ author: 1, createdAt: -1 });
postSchema.index({ tags: 1 });
postSchema.index({ published: 1, publishedAt: -1 });
postSchema.index({ title: 'text', content: 'text' });

// Pre-save middleware
postSchema.pre('save', function(next) {
    if (this.isModified('published') && this.published && !this.publishedAt) {
        this.publishedAt = new Date();
    }
    next();
});

// Optimized queries
class PostService {
    // Get posts with author details
    async getPostsWithAuthors(page = 1, limit = 10) {
        return Post.find({ published: true })
            .populate('author', 'name profile.avatar')
            .sort({ publishedAt: -1 })
            .limit(limit)
            .skip((page - 1) * limit)
            .lean();
    }

    // Search posts
    async searchPosts(query, filters = {}) {
        const searchQuery = {
            published: true,
            $text: { $search: query }
        };

        if (filters.tags) {
            searchQuery.tags = { $in: filters.tags };
        }

        if (filters.author) {
            searchQuery.author = filters.author;
        }

        return Post.find(searchQuery, {
            score: { $meta: 'textScore' }
        })
        .populate('author', 'name')
        .sort({ score: { $meta: 'textScore' } })
        .limit(20);
    }

    // Get user's feed
    async getUserFeed(userId, followingIds, page = 1, limit = 20) {
        return Post.find({
            published: true,
            author: { $in: [...followingIds, userId] }
        })
        .populate('author', 'name profile.avatar')
        .sort({ publishedAt: -1 })
        .limit(limit)
        .skip((page - 1) * limit);
    }

    // Bulk operations
    async bulkPublishPostIds(postIds) {
        return Post.updateMany(
            { _id: { $in: postIds } },
            {
                published: true,
                publishedAt: new Date()
            }
        );
    }

    // Analytics queries
    async getPostAnalytics(postId) {
        return Post.aggregate([
            { $match: { _id: mongoose.Types.ObjectId(postId) } },
            {
                $lookup: {
                    from: 'users',
                    localField: 'author',
                    foreignField: '_id',
                    as: 'author'
                }
            },
            {
                $project: {
                    title: 1,
                    views: 1,
                    likesCount: { $size: '$likes' },
                    commentsCount: { $size: '$comments' },
                    author: { $arrayElemAt: ['$author.name', 0] },
                    publishedAt: 1
                }
            }
        ]);
    }
}

const User = mongoose.model('User', userSchema);
const Post = mongoose.model('Post', postSchema);

module.exports = { User, Post, PostService };
```

---

## 🎯 **Practical Examples**
### **Complete E-commerce Database Schema**
```sql
-- E-commerce database schema
-- Users table
CREATE TABLE users (
    id VARCHAR(36) PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    email_verified BOOLEAN DEFAULT FALSE,

    INDEX idx_email (email),
    INDEX idx_created_at (created_at)
);

-- Products table
CREATE TABLE products (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    compare_at_price DECIMAL(10,2),
    cost_price DECIMAL(10,2),
    sku VARCHAR(100) UNIQUE,
    barcode VARCHAR(100),
    quantity INT DEFAULT 0,
    min_stock_level INT DEFAULT 0,
    max_stock_level INT DEFAULT 1000,
    weight DECIMAL(8,3),
    dimensions JSON, -- {"length": 10, "width": 5, "height": 2}
    category_id VARCHAR(36),
    brand_id VARCHAR(36),
    tags JSON, -- ["electronics", "wireless"]
    images JSON, -- [{"url": "...", "alt": "...", "position": 1}]
    seo JSON, -- {"title": "...", "description": "...", "slug": "..."}
    is_active BOOLEAN DEFAULT TRUE,
    is_featured BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (category_id) REFERENCES categories(id),
    FOREIGN KEY (brand_id) REFERENCES brands(id),
    INDEX idx_category_price (category_id, price),
    INDEX idx_brand (brand_id),
    INDEX idx_sku (sku),
    INDEX idx_is_active (is_active),
    INDEX idx_is_featured (is_featured),
    FULLTEXT idx_search (name, description)
);

-- Categories table
CREATE TABLE categories (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    parent_id VARCHAR(36),
    image_url VARCHAR(500),
    display_order INT DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (parent_id) REFERENCES categories(id),
    INDEX idx_parent (parent_id),
    INDEX idx_slug (slug),
    INDEX idx_display_order (display_order)
);

-- Orders table
CREATE TABLE orders (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    status ENUM('pending', 'confirmed', 'processing', 'shipped', 'delivered', 'cancelled', 'refunded') DEFAULT 'pending',
    payment_status ENUM('pending', 'paid', 'failed', 'refunded') DEFAULT 'pending',
    payment_method VARCHAR(50),
    payment_id VARCHAR(255),
    subtotal DECIMAL(10,2) NOT NULL,
    tax_amount DECIMAL(10,2) DEFAULT 0,
    shipping_amount DECIMAL(10,2) DEFAULT 0,
    discount_amount DECIMAL(10,2) DEFAULT 0,
    total_amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',

    -- Shipping information
    shipping_address JSON,
    billing_address JSON,

    -- Tracking
    tracking_number VARCHAR(100),
    carrier VARCHAR(50),
    estimated_delivery DATE,

    -- Timestamps
    ordered_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    confirmed_at TIMESTAMP NULL,
    shipped_at TIMESTAMP NULL,
    delivered_at TIMESTAMP NULL,
    cancelled_at TIMESTAMP NULL,

    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_user_status (user_id, status),
    INDEX idx_status (status),
    INDEX idx_ordered_at (ordered_at),
    INDEX idx_order_number (order_number)
);

-- Order items table
CREATE TABLE order_items (
    id VARCHAR(36) PRIMARY KEY,
    order_id VARCHAR(36) NOT NULL,
    product_id VARCHAR(36) NOT NULL,
    variant_id VARCHAR(36),
    product_name VARCHAR(255) NOT NULL,
    product_sku VARCHAR(100),
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    total_price DECIMAL(10,2) NOT NULL,
    product_options JSON, -- Selected options like size, color

    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id),
    INDEX idx_order_product (order_id, product_id)
);

-- Shopping cart
CREATE TABLE cart_items (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36),
    session_id VARCHAR(255), -- For guest users
    product_id VARCHAR(36) NOT NULL,
    variant_id VARCHAR(36),
    quantity INT NOT NULL,
    product_options JSON,
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE,
    INDEX idx_user_session (user_id, session_id),
    INDEX idx_product (product_id)
);

-- Reviews and ratings
CREATE TABLE product_reviews (
    id VARCHAR(36) PRIMARY KEY,
    product_id VARCHAR(36) NOT NULL,
    user_id VARCHAR(36) NOT NULL,
    order_id VARCHAR(36), -- Link to order for verification
    rating INT NOT NULL CHECK (rating >= 1 AND rating <= 5),
    title VARCHAR(200),
    comment TEXT,
    is_verified BOOLEAN DEFAULT FALSE,
    helpful_votes INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (order_id) REFERENCES orders(id),
    UNIQUE KEY unique_user_product_review (user_id, product_id),
    INDEX idx_product_rating (product_id, rating),
    INDEX idx_created_at (created_at)
);

-- Inventory tracking
CREATE TABLE inventory_transactions (
    id VARCHAR(36) PRIMARY KEY,
    product_id VARCHAR(36) NOT NULL,
    transaction_type ENUM('purchase', 'sale', 'adjustment', 'return', 'transfer') NOT NULL,
    quantity_change INT NOT NULL,
    previous_quantity INT NOT NULL,
    new_quantity INT NOT NULL,
    reference_id VARCHAR(36), -- Order ID, Purchase ID, etc.
    reference_type VARCHAR(50),
    notes TEXT,
    created_by VARCHAR(36),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (product_id) REFERENCES products(id),
    FOREIGN KEY (created_by) REFERENCES users(id),
    INDEX idx_product_date (product_id, created_at),
    INDEX idx_reference (reference_id, reference_type)
);

-- Wishlist
CREATE TABLE wishlists (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    product_id VARCHAR(36) NOT NULL,
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE,
    UNIQUE KEY unique_user_product_wishlist (user_id, product_id),
    INDEX idx_user_added (user_id, added_at)
);

-- Product views for analytics
CREATE TABLE product_views (
    id VARCHAR(36) PRIMARY KEY,
    product_id VARCHAR(36) NOT NULL,
    user_id VARCHAR(36),
    session_id VARCHAR(255),
    ip_address VARCHAR(45),
    user_agent TEXT,
    viewed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_product_viewed (product_id, viewed_at),
    INDEX idx_user_viewed (user_id, viewed_at),
    INDEX idx_session (session_id)
);

-- Coupons and discounts
CREATE TABLE coupons (
    id VARCHAR(36) PRIMARY KEY,
    code VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    discount_type ENUM('percentage', 'fixed_amount') NOT NULL,
    discount_value DECIMAL(10,2) NOT NULL,
    minimum_order_amount DECIMAL(10,2) DEFAULT 0,
    maximum_discount_amount DECIMAL(10,2),
    usage_limit INT,
    usage_count INT DEFAULT 0,
    user_usage_limit INT DEFAULT 1, -- Per user
    starts_at TIMESTAMP,
    expires_at TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    applicable_products JSON, -- Product IDs or categories
    applicable_users JSON, -- User IDs or segments
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_code (code),
    INDEX idx_active_expires (is_active, expires_at),
    INDEX idx_starts_expires (starts_at, expires_at)
);

-- Coupon usage tracking
CREATE TABLE coupon_usage (
    id VARCHAR(36) PRIMARY KEY,
    coupon_id VARCHAR(36) NOT NULL,
    user_id VARCHAR(36) NOT NULL,
    order_id VARCHAR(36) NOT NULL,
    discount_amount DECIMAL(10,2) NOT NULL,
    used_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (coupon_id) REFERENCES coupons(id),
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (order_id) REFERENCES orders(id),
    UNIQUE KEY unique_coupon_order (coupon_id, order_id),
    INDEX idx_user_used (user_id, used_at)
);
```

### **Social Media Database Schema**
```sql
-- Social media platform database schema
-- Users table (extended)
CREATE TABLE users (
    id VARCHAR(36) PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    display_name VARCHAR(100),
    bio TEXT,
    avatar_url VARCHAR(500),
    cover_url VARCHAR(500),
    website VARCHAR(255),
    location VARCHAR(100),
    birthday DATE,
    is_private BOOLEAN DEFAULT FALSE,
    is_verified BOOLEAN DEFAULT FALSE,
    email_verified BOOLEAN DEFAULT FALSE,
    last_active_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_username (username),
    INDEX idx_email (email),
    INDEX idx_is_private (is_private),
    INDEX idx_last_active (last_active_at),
    FULLTEXT idx_search (username, display_name, bio)
);

-- Posts table
CREATE TABLE posts (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    content TEXT,
    media_urls JSON, -- [{"url": "...", "type": "image/video", "thumbnail": "..."}]
    media_type ENUM('text', 'image', 'video', 'mixed') DEFAULT 'text',
    location VARCHAR(255),
    is_pinned BOOLEAN DEFAULT FALSE,
    is_sensitive BOOLEAN DEFAULT FALSE,
    reply_to_id VARCHAR(36), -- For replies
    repost_of_id VARCHAR(36), -- For reposts/shares
    visibility ENUM('public', 'followers', 'mentioned') DEFAULT 'public',
    published_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    edited_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (reply_to_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (repost_of_id) REFERENCES posts(id) ON DELETE CASCADE,
    INDEX idx_user_published (user_id, published_at),
    INDEX idx_reply_to (reply_to_id),
    INDEX idx_repost_of (repost_of_id),
    INDEX idx_visibility (visibility),
    INDEX idx_published_at (published_at),
    FULLTEXT idx_content_search (content)
);

-- Hashtags table
CREATE TABLE hashtags (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    usage_count INT DEFAULT 0,
    trending_score DECIMAL(8,4) DEFAULT 0,
    last_used_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    INDEX idx_name (name),
    INDEX idx_usage_count (usage_count),
    INDEX idx_trending_score (trending_score),
    INDEX idx_last_used (last_used_at)
);

-- Post hashtags (many-to-many)
CREATE TABLE post_hashtags (
    post_id VARCHAR(36) NOT NULL,
    hashtag_id VARCHAR(36) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (post_id, hashtag_id),
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (hashtag_id) REFERENCES hashtags(id) ON DELETE CASCADE,
    INDEX idx_post (post_id),
    INDEX idx_hashtag (hashtag_id)
);

-- Likes table
CREATE TABLE likes (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    post_id VARCHAR(36) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    UNIQUE KEY unique_user_post_like (user_id, post_id),
    INDEX idx_user_created (user_id, created_at),
    INDEX idx_post_created (post_id, created_at)
);

-- Bookmarks table
CREATE TABLE bookmarks (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    post_id VARCHAR(36) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    UNIQUE KEY unique_user_post_bookmark (user_id, post_id),
    INDEX idx_user_created (user_id, created_at)
);

-- Follows table
CREATE TABLE follows (
    id VARCHAR(36) PRIMARY KEY,
    follower_id VARCHAR(36) NOT NULL,
    following_id VARCHAR(36) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (follower_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (following_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE KEY unique_follow (follower_id, following_id),
    INDEX idx_follower_created (follower_id, created_at),
    INDEX idx_following_created (following_id, created_at)
);

-- Blocks table
CREATE TABLE blocks (
    id VARCHAR(36) PRIMARY KEY,
    blocker_id VARCHAR(36) NOT NULL,
    blocked_id VARCHAR(36) NOT NULL,
    reason TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (blocker_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (blocked_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE KEY unique_block (blocker_id, blocked_id),
    INDEX idx_blocker (blocker_id),
    INDEX idx_blocked (blocked_id)
);

-- Direct messages
CREATE TABLE conversations (
    id VARCHAR(36) PRIMARY KEY,
    type ENUM('direct', 'group') DEFAULT 'direct',
    name VARCHAR(100), -- For group chats
    description TEXT,
    avatar_url VARCHAR(500),
    created_by VARCHAR(36) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (created_by) REFERENCES users(id),
    INDEX idx_created_by (created_by),
    INDEX idx_updated_at (updated_at)
);

-- Conversation participants
CREATE TABLE conversation_participants (
    conversation_id VARCHAR(36) NOT NULL,
    user_id VARCHAR(36) NOT NULL,
    role ENUM('admin', 'member') DEFAULT 'member',
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_read_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (conversation_id, user_id),
    FOREIGN KEY (conversation_id) REFERENCES conversations(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_joined (user_id, joined_at),
    INDEX idx_conversation_last_read (conversation_id, last_read_at)
);

-- Messages table
CREATE TABLE messages (
    id VARCHAR(36) PRIMARY KEY,
    conversation_id VARCHAR(36) NOT NULL,
    sender_id VARCHAR(36) NOT NULL,
    content TEXT,
    message_type ENUM('text', 'image', 'video', 'file', 'system') DEFAULT 'text',
    media_url VARCHAR(500),
    reply_to_id VARCHAR(36), -- Reply to another message
    is_edited BOOLEAN DEFAULT FALSE,
    edited_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (conversation_id) REFERENCES conversations(id) ON DELETE CASCADE,
    FOREIGN KEY (sender_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (reply_to_id) REFERENCES messages(id) ON DELETE SET NULL,
    INDEX idx_conversation_created (conversation_id, created_at),
    INDEX idx_sender_created (sender_id, created_at),
    INDEX idx_reply_to (reply_to_id)
);

-- Notifications table
CREATE TABLE notifications (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    type ENUM('like', 'reply', 'follow', 'mention', 'repost', 'message') NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    related_user_id VARCHAR(36), -- User who triggered the notification
    related_post_id VARCHAR(36),
    related_message_id VARCHAR(36),
    is_read BOOLEAN DEFAULT FALSE,
    read_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (related_user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (related_post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (related_message_id) REFERENCES messages(id) ON DELETE CASCADE,
    INDEX idx_user_created (user_id, created_at),
    INDEX idx_user_unread (user_id, is_read, created_at),
    INDEX idx_type (type)
);

-- Analytics tables
CREATE TABLE post_analytics (
    id VARCHAR(36) PRIMARY KEY,
    post_id VARCHAR(36) NOT NULL,
    date DATE NOT NULL,
    views INT DEFAULT 0,
    likes INT DEFAULT 0,
    replies INT DEFAULT 0,
    reposts INT DEFAULT 0,
    bookmarks INT DEFAULT 0,
    impressions INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    UNIQUE KEY unique_post_date (post_id, date),
    INDEX idx_post_date (post_id, date),
    INDEX idx_date_views (date, views)
);

CREATE TABLE user_analytics (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    date DATE NOT NULL,
    profile_views INT DEFAULT 0,
    new_followers INT DEFAULT 0,
    posts_created INT DEFAULT 0,
    likes_received INT DEFAULT 0,
    replies_received INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE KEY unique_user_date (user_id, date),
    INDEX idx_user_date (user_id, date),
    INDEX idx_date_followers (date, new_followers)
);
```

### **Database Connection and Management**
```javascript
// Database connection manager
const mysql = require('mysql2/promise');
const { MongoClient } = require('mongodb');

class DatabaseManager {
    constructor() {
        this.mysqlPool = null;
        this.mongoClient = null;
        this.isConnected = false;
    }

    async connect(config) {
        try {
            // MySQL connection
            if (config.mysql) {
                this.mysqlPool = mysql.createPool({
                    host: config.mysql.host,
                    user: config.mysql.user,
                    password: config.mysql.password,
                    database: config.mysql.database,
                    waitForConnections: true,
                    connectionLimit: config.mysql.connectionLimit || 10,
                    queueLimit: config.mysql.queueLimit || 0,
                    acquireTimeout: 60000,
                    timeout: 60000,
                });

                // Test MySQL connection
                const mysqlConnection = await this.mysqlPool.getConnection();
                await mysqlConnection.ping();
                mysqlConnection.release();

                console.log('✅ MySQL connected');
            }

            // MongoDB connection
            if (config.mongodb) {
                this.mongoClient = new MongoClient(config.mongodb.uri, {
                    maxPoolSize: config.mongodb.maxPoolSize || 10,
                    serverSelectionTimeoutMS: 5000,
                    socketTimeoutMS: 45000,
                });

                await this.mongoClient.connect();
                console.log('✅ MongoDB connected');
            }

            this.isConnected = true;
            console.log('✅ All databases connected successfully');

        } catch (error) {
            console.error('❌ Database connection failed:', error);
            throw error;
        }
    }

    async disconnect() {
        try {
            if (this.mysqlPool) {
                await this.mysqlPool.end();
                console.log('✅ MySQL connection closed');
            }

            if (this.mongoClient) {
                await this.mongoClient.close();
                console.log('✅ MongoDB connection closed');
            }

            this.isConnected = false;
        } catch (error) {
            console.error('❌ Error closing database connections:', error);
        }
    }

    // MySQL query method
    async query(sql, params = []) {
        if (!this.mysqlPool) {
            throw new Error('MySQL not connected');
        }

        try {
            const [rows, fields] = await this.mysqlPool.execute(sql, params);
            return { rows, fields };
        } catch (error) {
            console.error('MySQL query error:', error);
            throw error;
        }
    }

    // MongoDB collection method
    getCollection(databaseName, collectionName) {
        if (!this.mongoClient) {
            throw new Error('MongoDB not connected');
        }

        return this.mongoClient.db(databaseName).collection(collectionName);
    }

    // Transaction support
    async transaction(callback) {
        if (!this.mysqlPool) {
            throw new Error('MySQL not connected');
        }

        const connection = await this.mysqlPool.getConnection();

        try {
            await connection.beginTransaction();

            const result = await callback(connection);

            await connection.commit();
            return result;

        } catch (error) {
            await connection.rollback();
            throw error;

        } finally {
            connection.release();
        }
    }

    // Health check
    async healthCheck() {
        const health = {
            mysql: false,
            mongodb: false,
            timestamp: new Date().toISOString(),
        };

        try {
            if (this.mysqlPool) {
                const connection = await this.mysqlPool.getConnection();
                await connection.ping();
                connection.release();
                health.mysql = true;
            }
        } catch (error) {
            console.error('MySQL health check failed:', error);
        }

        try {
            if (this.mongoClient) {
                await this.mongoClient.db().admin().ping();
                health.mongodb = true;
            }
        } catch (error) {
            console.error('MongoDB health check failed:', error);
        }

        return health;
    }

    // Get connection stats
    async getStats() {
        const stats = {
            mysql: {},
            mongodb: {},
        };

        try {
            if (this.mysqlPool) {
                // Get MySQL connection stats
                const [rows] = await this.mysqlPool.query('SHOW PROCESSLIST');
                stats.mysql.connections = rows.length;
                stats.mysql.poolSize = this.mysqlPool.pool.config.connectionLimit;
            }
        } catch (error) {
            console.error('Error getting MySQL stats:', error);
        }

        try {
            if (this.mongoClient) {
                const dbStats = await this.mongoClient.db().stats();
                stats.mongodb = {
                    collections: dbStats.collections,
                    objects: dbStats.objects,
                    dataSize: dbStats.dataSize,
                    storageSize: dbStats.storageSize,
                };
            }
        } catch (error) {
            console.error('Error getting MongoDB stats:', error);
        }

        return stats;
    }
}

// Usage
const dbManager = new DatabaseManager();

async function initializeDatabase() {
    await dbManager.connect({
        mysql: {
            host: process.env.DB_HOST,
            user: process.env.DB_USER,
            password: process.env.DB_PASSWORD,
            database: process.env.DB_NAME,
            connectionLimit: 10,
        },
        mongodb: {
            uri: process.env.MONGODB_URI,
            maxPoolSize: 10,
        },
    });
}

module.exports = { DatabaseManager, dbManager };
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Database Design Principles**: Atomicity, consistency, isolation, durability
- ✅ **Data Modeling & Normalization**: 1NF, 2NF, 3NF, and denormalization strategies
- ✅ **Database Types & Selection**: SQL vs NoSQL, when to use each
- ✅ **Schema Design & Relationships**: One-to-one, one-to-many, many-to-many relationships
- ✅ **Indexing & Performance**: Index strategies, query optimization, covering indexes
- ✅ **Database Security**: Access control, SQL injection prevention, data encryption
- ✅ **Backup & Recovery**: Automated backup systems, recovery procedures
- ✅ **Database Migration**: Version control for database schema changes
- ✅ **ORM & Query Optimization**: Sequelize and Mongoose best practices
- ✅ **Practical Implementation**: Complete e-commerce and social media schemas

### **Best Practices**
1. **Design for scalability**: Plan for growth and performance requirements
2. **Use appropriate data types**: Choose the right types for your data
3. **Implement proper indexing**: Index frequently queried columns
4. **Normalize when appropriate**: Balance normalization with query performance
5. **Use transactions**: Ensure data consistency for multi-step operations
6. **Implement proper security**: Use parameterized queries and access controls
7. **Plan for backups**: Regular automated backups with testing
8. **Version your schema**: Use migrations for schema changes
9. **Monitor performance**: Track slow queries and optimize bottlenecks
10. **Document your schema**: Keep documentation up to date

### **Next Steps**
- Learn about database clustering and replication
- Study advanced SQL features (window functions, CTEs)
- Explore time-series databases for analytics
- Learn about database sharding and partitioning
- Study database administration and monitoring tools
- Explore graph databases for complex relationships
- Learn about database caching strategies
- Study ETL processes and data warehousing
- Explore real-time databases and change data capture
- Learn about database testing and quality assurance

---

## 🎯 **Assignment**

### **Task 1: E-commerce Database Design**
Design and implement a complete e-commerce database schema with:
- User management with roles and permissions
- Product catalog with categories, variants, and inventory
- Shopping cart and wishlist functionality
- Order processing with payment integration
- Review and rating system
- Coupon and discount management
- Analytics and reporting tables
- Proper indexing and performance optimization
- Backup and recovery procedures

### **Task 2: Social Media Database Design**
Create a comprehensive social media platform database with:
- User profiles with privacy settings
- Posts with media support and threading
- Likes, comments, and sharing functionality
- Follow/follower relationships
- Direct messaging system
- Hashtag and trending system
- Notification system
- Analytics for engagement tracking
- Content moderation features

### **Task 3: Multi-tenant SaaS Database**
Design a multi-tenant SaaS application database that supports:
- Tenant isolation and data separation
- Shared resources and configurations
- Billing and subscription management
- User management across tenants
- Feature flags and customization
- Analytics and usage tracking
- Backup and recovery for multi-tenant environment
- Performance optimization for large-scale operations

### **Task 4: Database Migration & Optimization**
Implement a complete database migration and optimization system:
- Automated migration system with rollback capability
- Database seeding for initial data
- Performance monitoring and query optimization
- Index management and maintenance
- Automated backup and recovery procedures
- Database health monitoring and alerting
- Query performance analysis and optimization
- Connection pooling and resource management

---

## 📚 **Additional Resources**
- [Database Design for Mere Mortals](https://www.amazon.com/Database-Design-Mere-Mortals-Hands/dp/0321884493)
- [SQL Performance Explained](https://sql-performance-explained.com/)
- [MongoDB Documentation](https://docs.mongodb.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Database Normalization](https://en.wikipedia.org/wiki/Database_normalization)
- [Index Design Guide](https://use-the-index-luke.com/)
- [OWASP Database Security](https://owasp.org/www-project-database-security/)

---

**Next Lesson**: [Lesson 42: Microservices Architecture](Lesson%2042_%20Microservices%20Architecture.md)
   