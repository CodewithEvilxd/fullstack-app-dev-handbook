# 🚇 Metro Bundler Setup Guide for React Native
## Complete Configuration & Optimization Manual

---

## 🎯 **Overview**
Metro is the JavaScript bundler for React Native. It takes in an entry file and its dependencies and creates a single JavaScript file that can be executed in a React Native app. This guide covers complete Metro setup, configuration, and optimization.

---

## 📋 **Table of Contents**
1. [What is Metro?](#what-is-metro)
2. [Installation & Setup](#installation--setup)
3. [Basic Configuration](#basic-configuration)
4. [Advanced Configuration](#advanced-configuration)
5. [Performance Optimization](#performance-optimization)
6. [Troubleshooting](#troubleshooting)
7. [Custom Transformers](#custom-transformers)
8. [Integration with Tools](#integration-with-tools)
9. [Best Practices](#best-practices)

---

## 🤔 **What is Metro?**

### **Metro's Role in React Native:**
- **JavaScript Bundler**: Combines all JavaScript files into a single bundle
- **Asset Processor**: Handles images, fonts, and other assets
- **Development Server**: Provides hot reloading and development tools
- **Module Resolver**: Resolves import statements and dependencies
- **Transformer**: Converts modern JavaScript/TypeScript to compatible code

### **Key Features:**
```bash
✅ Fast bundling with incremental builds
✅ Hot Module Replacement (HMR)
✅ Asset processing and optimization
✅ Source map generation
✅ Customizable transformers
✅ Plugin system
✅ Development server with live reload
```

### **Architecture:**
```
Source Code ──► Metro Bundler ──► Bundle
     │               │
     └─ Transformers ┘
         │
         └─ Babel, TypeScript, etc.
```

---

## 📥 **Installation & Setup**

### **Automatic Installation (React Native CLI)**
```bash
# When creating a new React Native project
npx react-native init MyProject

# Metro is automatically included
# Located in: node_modules/metro
```

### **Manual Installation**
```bash
# Install Metro packages
npm install metro metro-config metro-core metro-react-native-babel-preset metro-resolver

# For React Native specific features
npm install @react-native/metro-config
```

### **Expo Projects**
```bash
# Metro is pre-configured in Expo projects
npx create-expo-app MyExpoApp

# Metro config is automatically set up
# Custom configuration can be added via metro.config.js
```

---

## ⚙️ **Basic Configuration**

### **Creating Metro Configuration**
```javascript
// metro.config.js
const { getDefaultConfig, mergeConfig } = require('@react-native/metro-config');

/**
 * Metro configuration
 * https://facebook.github.io/metro/docs/configuration
 */
const config = {
  // Your Metro configuration options
};

module.exports = mergeConfig(getDefaultConfig(__dirname), config);
```

### **Basic Configuration Options**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  // Entry point
  index: 'index.js',

  // Asset extensions
  assetExts: ['png', 'jpg', 'jpeg', 'gif', 'svg', 'ttf'],

  // Source extensions
  sourceExts: ['js', 'jsx', 'ts', 'tsx', 'json'],

  // Platforms
  platforms: ['ios', 'android'],

  // Watch folders
  watchFolders: [
    __dirname,
    // Add additional folders to watch
  ],

  // Resolver options
  resolver: {
    // Module resolution options
  },

  // Transformer options
  transformer: {
    // Code transformation options
  },
};
```

### **Project Structure with Metro**
```bash
MyReactNativeApp/
├── metro.config.js          # Metro configuration
├── index.js                 # Entry point
├── package.json
├── node_modules/
├── android/
├── ios/
└── src/                     # Source code
    ├── components/
    ├── screens/
    ├── utils/
    └── assets/
```

---

## 🔧 **Advanced Configuration**

### **Module Resolution**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  resolver: {
    // Alias configuration
    alias: {
      '@': './src',
      '@components': './src/components',
      '@screens': './src/screens',
      '@utils': './src/utils',
      '@assets': './src/assets',
    },

    // Extra node modules
    extraNodeModules: {
      // Custom module mappings
    },

    // Platform-specific extensions
    platforms: ['ios', 'android', 'web'],

    // Asset resolutions
    assetResolutions: ['@2x', '@3x'],

    // Block list for excluding files
    blockList: [
      /node_modules\/react\/.*/,
      /node_modules\/react-native\/.*/,
    ],
  },
};
```

### **Transformer Configuration**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  transformer: {
    // Babel configuration
    babelTransformerPath: require.resolve('react-native-svg-transformer'),

    // Asset processing
    assetPlugins: ['expo-asset/tools/hashAssetFiles'],

    // Minification
    minifierConfig: {
      compress: {
        // Compression options
        drop_console: true,
        drop_debugger: true,
      },
      mangle: {
        // Mangle options
      },
    },

    // Source maps
    sourceMap: true,

    // Experimental features
    experimentalImportSupport: false,
    inlineRequires: false,
  },
};
```

### **Custom Asset Processing**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  transformer: {
    // Custom asset transformer
    assetTransformerPath: require.resolve('./customAssetTransformer'),

    // Asset options
    assetOptions: {
      scales: [0.5, 1, 2, 3],
      format: 'png',
    },
  },

  // Server configuration
  server: {
    port: 8081,
    host: 'localhost',
    enhanceMiddleware: (middleware) => {
      // Custom middleware
      return middleware;
    },
  },
};
```

### **Monorepo Configuration**
```javascript
// metro.config.js (for monorepo)
const { getDefaultConfig } = require('@react-native/metro-config');
const path = require('path');

module.exports = {
  ...getDefaultConfig(__dirname),

  // Watch all packages in monorepo
  watchFolders: [
    path.resolve(__dirname, '../../packages'),
  ],

  resolver: {
    // Resolve modules from monorepo packages
    extraNodeModules: {
      '@mycompany/ui': path.resolve(__dirname, '../../packages/ui'),
      '@mycompany/utils': path.resolve(__dirname, '../../packages/utils'),
    },

    // Block node_modules from monorepo root
    blockList: [
      /\/packages\/.*\/node_modules\//,
    ],
  },
};
```

---

## ⚡ **Performance Optimization**

### **Bundle Splitting**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  // Enable bundle splitting
  transformer: {
    enableBabelRCLookup: true,
    lazyImports: [
      'react-navigation',
      'react-native-vector-icons',
      // Add other libraries to lazy load
    ],
  },
};
```

### **Caching Configuration**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  // Cache configuration
  cacheStores: [
    {
      type: 'file',
      // Cache directory
      cacheDirectory: path.join(__dirname, 'node_modules', '.cache', 'metro'),
      // Cache version
      cacheVersion: '1.0',
    },
  ],

  // Reset cache on configuration change
  resetCache: process.env.RESET_METRO_CACHE === 'true',
};
```

### **Parallel Processing**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  // Enable parallel processing
  maxWorkers: 4,

  // Transformer options for performance
  transformer: {
    enableBabelRCLookup: true,
    experimentalImportSupport: false,
    inlineRequires: true,
  },
};
```

### **Asset Optimization**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  transformer: {
    // Asset optimization
    assetOptions: {
      scales: [1, 2, 3],
      format: 'png',
      quality: 0.8,
    },

    // Image processing
    assetPlugins: [
      'expo-asset/tools/hashAssetFiles',
      // Custom asset plugins
    ],
  },
};
```

---

## 🔧 **Troubleshooting**

### **Common Issues & Solutions**

#### **Issue 1: Module Resolution Errors**
```bash
# Error: Unable to resolve module
# Solution: Check metro.config.js resolver configuration

resolver: {
  alias: {
    '@': './src',
  },
  extraNodeModules: {
    // Add missing modules
  },
}
```

#### **Issue 2: Bundle Size Issues**
```bash
# Check bundle size
npx react-native bundle --platform android --dev false --entry-file index.js --bundle-output bundle.android.js --sourcemap-output bundle.android.js.map

# Analyze bundle
npx metro-bundle-analyzer bundle.android.js

# Solutions:
# 1. Enable tree shaking
# 2. Use dynamic imports
# 3. Remove unused dependencies
```

#### **Issue 3: Hot Reload Not Working**
```bash
# Clear Metro cache
npx react-native start --reset-cache

# Check file watching
# Ensure no antivirus blocking file changes

# Metro configuration
module.exports = {
  ...getDefaultConfig(__dirname),
  watchFolders: [
    __dirname,
  ],
};
```

#### **Issue 4: Asset Loading Issues**
```bash
# Check asset extensions
assetExts: ['png', 'jpg', 'jpeg', 'gif', 'svg', 'ttf', 'otf'],

// Check asset resolution
resolver: {
  assetResolutions: ['@1x', '@2x', '@3x'],
},
```

#### **Issue 5: Build Performance Issues**
```bash
# Enable cache
transformer: {
  enableBabelRCLookup: true,
},

# Increase workers
maxWorkers: 4,

# Use persistent cache
cacheStores: [
  {
    type: 'file',
    cacheDirectory: './.metro-cache',
  },
],
```

### **Debugging Metro**
```bash
# Enable verbose logging
npx react-native start --verbose

# Debug specific modules
resolver: {
  resolveRequest: (context, moduleName, platform) => {
    console.log('Resolving:', moduleName);
    return context.resolveRequest(context, moduleName, platform);
  },
},
```

---

## 🔄 **Custom Transformers**

### **Creating Custom Transformer**
```javascript
// customTransformer.js
const { transformSync } = require('@babel/core');
const path = require('path');

module.exports = {
  process(src, filePath) {
    // Custom transformation logic
    if (filePath.endsWith('.custom')) {
      // Transform .custom files
      return {
        code: transformCustomCode(src),
        map: null,
      };
    }

    // Default Babel transformation
    const result = transformSync(src, {
      filename: filePath,
      presets: ['module:metro-react-native-babel-preset'],
    });

    return {
      code: result.code,
      map: result.map,
    };
  },

  getCacheKey() {
    // Return cache key for this transformer
    return 'custom-transformer-v1';
  },
};

function transformCustomCode(src) {
  // Custom transformation logic
  return src.replace(/custom-syntax/g, 'transformed-syntax');
}
```

### **SVG Transformer**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  transformer: {
    babelTransformerPath: require.resolve('react-native-svg-transformer'),
  },

  resolver: {
    assetExts: ['png', 'jpg', 'jpeg', 'gif', 'svg'].filter(ext => ext !== 'svg'),
    sourceExts: ['js', 'jsx', 'ts', 'tsx', 'svg'],
  },
};
```

### **TypeScript Transformer**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  transformer: {
    babelTransformerPath: require.resolve('react-native-typescript-transformer'),
  },

  resolver: {
    sourceExts: ['js', 'jsx', 'ts', 'tsx'],
  },
};
```

---

## 🔗 **Integration with Tools**

### **Integration with Jest**
```javascript
// jest.config.js
module.exports = {
  preset: 'react-native',
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?)/)',
  ],
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
};
```

### **Integration with ESLint**
```javascript
// .eslintrc.js
module.exports = {
  root: true,
  extends: ['@react-native-community'],
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint'],
  rules: {
    // Custom rules
  },
  settings: {
    'import/resolver': {
      'babel-module': {
        root: ['./src'],
        alias: {
          '@': './src',
        },
      },
    },
  },
};
```

### **Integration with Prettier**
```javascript
// .prettierrc.js
module.exports = {
  semi: true,
  trailingComma: 'es5',
  singleQuote: true,
  printWidth: 80,
  tabWidth: 2,
  useTabs: false,
};
```

### **Integration with VS Code**
```json
// .vscode/settings.json
{
  "javascript.preferences.importModuleSpecifier": "relative",
  "typescript.preferences.importModuleSpecifier": "relative",
  "emmet.includeLanguages": {
    "javascript": "javascriptreact",
    "typescript": "typescriptreact"
  },
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  }
}
```

---

## 📚 **Best Practices**

### **Configuration Organization**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');
const path = require('path');

// Separate configuration sections
const resolverConfig = {
  alias: {
    '@': './src',
    '@components': './src/components',
    '@screens': './src/screens',
  },
  extraNodeModules: {},
};

const transformerConfig = {
  enableBabelRCLookup: true,
  minifierConfig: {
    compress: {
      drop_console: __DEV__ ? false : true,
    },
  },
};

const serverConfig = {
  port: process.env.METRO_PORT || 8081,
  host: process.env.METRO_HOST || 'localhost',
};

module.exports = {
  ...getDefaultConfig(__dirname),
  resolver: resolverConfig,
  transformer: transformerConfig,
  server: serverConfig,
};
```

### **Performance Monitoring**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  // Performance monitoring
  transformer: {
    enableBabelRCLookup: true,
    minifierConfig: {
      compress: {
        drop_console: true,
      },
    },
  },

  // Bundle analysis
  serializer: {
    createModuleIdFactory: () => {
      return (path) => {
        // Custom module ID generation
        return path;
      };
    },
  },
};
```

### **Development vs Production**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');
const isProduction = process.env.NODE_ENV === 'production';

module.exports = {
  ...getDefaultConfig(__dirname),

  transformer: {
    minifierConfig: {
      compress: {
        drop_console: isProduction,
        drop_debugger: isProduction,
      },
    },
    sourceMap: !isProduction,
  },

  resolver: {
    alias: {
      'react-native': isProduction
        ? 'react-native'
        : 'react-native/Libraries/react-native/react-native-implementation',
    },
  },
};
```

### **Security Considerations**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  // Security: Block sensitive files
  resolver: {
    blockList: [
      /.*\.key$/,
      /.*\.pem$/,
      /.*\.env$/,
      /node_modules\/.*\/secrets\/.*/,
    ],
  },

  // Security: Limit asset access
  transformer: {
    assetOptions: {
      // Only allow specific asset types
      allowedExtensions: ['png', 'jpg', 'jpeg', 'gif', 'svg'],
    },
  },
};
```

---

## 🚀 **Advanced Features**

### **Custom Metro Server**
```javascript
// customMetroServer.js
const Metro = require('metro');
const { getDefaultConfig } = require('@react-native/metro-config');

async function startCustomMetro() {
  const config = await getDefaultConfig(__dirname);

  const customConfig = {
    ...config,
    server: {
      ...config.server,
      enhanceMiddleware: (middleware) => {
        // Add custom middleware
        middleware.use('/api', (req, res) => {
          res.json({ message: 'Custom Metro API' });
        });
        return middleware;
      },
    },
  };

  const metroServer = await Metro.runMetro(customConfig);
  console.log('Custom Metro server started');
}

startCustomMetro();
```

### **Plugin System**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  // Custom plugins
  plugins: [
    {
      name: 'custom-plugin',
      setup: (metroConfig) => {
        // Plugin setup logic
        console.log('Custom plugin loaded');
      },
    },
  ],
};
```

### **Custom Module Resolution**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  resolver: {
    resolveRequest: (context, moduleName, platform) => {
      // Custom resolution logic
      if (moduleName.startsWith('@company/')) {
        // Resolve company-specific modules
        return {
          filePath: path.resolve(__dirname, 'packages', moduleName.replace('@company/', '')),
          type: 'sourceFile',
        };
      }

      // Default resolution
      return context.resolveRequest(context, moduleName, platform);
    },
  },
};
```

---

## 📊 **Monitoring & Analytics**

### **Bundle Analysis**
```bash
# Analyze bundle size
npx metro-bundle-analyzer

# Generate bundle report
npx react-native bundle \
  --platform android \
  --dev false \
  --entry-file index.js \
  --bundle-output bundle.android.js \
  --sourcemap-output bundle.android.js.map

# View bundle composition
npx webpack-bundle-analyzer bundle.android.js
```

### **Performance Metrics**
```javascript
// metro.config.js
const { getDefaultConfig } = require('@react-native/metro-config');

module.exports = {
  ...getDefaultConfig(__dirname),

  // Performance monitoring
  transformer: {
    enableBabelRCLookup: true,
    reportTransformedFiles: true,
  },

  // Bundle metrics
  serializer: {
    createModuleIdFactory: () => {
      const moduleIds = new Map();
      let nextId = 0;

      return (path) => {
        if (!moduleIds.has(path)) {
          moduleIds.set(path, nextId++);
        }
        return moduleIds.get(path);
      };
    },
  },
};
```

---

## 🎯 **Conclusion**

Metro is a powerful and flexible bundler specifically designed for React Native applications. Proper configuration and optimization are crucial for development efficiency and production performance.

### **Key Takeaways:**
- ✅ Configure Metro for your project structure
- ✅ Use aliases for clean imports
- ✅ Optimize for production builds
- ✅ Implement proper caching strategies
- ✅ Monitor bundle size and performance
- ✅ Use custom transformers when needed

### **Next Steps:**
1. **Configure**: Set up Metro for your React Native project
2. **Optimize**: Implement performance optimizations
3. **Monitor**: Track bundle size and build performance
4. **Customize**: Add custom transformers and resolvers
5. **Scale**: Handle monorepos and large codebases

### **Additional Resources:**
- [Metro Documentation](https://facebook.github.io/metro/)
- [React Native Metro Config](https://github.com/facebook/metro)
- [Metro Performance Guide](https://facebook.github.io/metro/docs/performance)
- [Custom Transformers](https://facebook.github.io/metro/docs/custom-transformers)

---

*Happy Metro Bundling! 🚇📦*