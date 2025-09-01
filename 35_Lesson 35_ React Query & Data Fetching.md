# Lesson 35: React Query & Data Fetching

## 🎯 **Learning Objectives**
- Master React Query (TanStack Query) for data fetching
- Implement efficient caching and synchronization
- Handle loading states, errors, and optimistic updates
- Create custom hooks for API interactions
- Implement real-time data updates

## 📚 **Table of Contents**
1. [Introduction to React Query](#introduction-to-react-query)
2. [Basic Queries](#basic-queries)
3. [Mutations](#mutations)
4. [Query Invalidation](#query-invalidation)
5. [Caching Strategies](#caching-strategies)
6. [Error Handling](#error-handling)
7. [Optimistic Updates](#optimistic-updates)
8. [Infinite Queries](#infinite-queries)
9. [Background Refetching](#background-refetching)
10. [DevTools Integration](#devtools-integration)
11. [Practical Examples](#practical-examples)

---

## 🔍 **Introduction to React Query**

### **What is React Query?**
React Query (now TanStack Query) is a powerful data synchronization library for React and React Native. It provides:

- **Caching**: Intelligent caching of server state
- **Synchronization**: Background refetching and cache updates
- **Performance**: Optimized re-renders and memory usage
- **Developer Experience**: Great debugging tools and TypeScript support

### **Key Features**
- ✅ **Declarative**: Write queries like components
- ✅ **Automatic**: Background updates and cache management
- ✅ **Powerful**: Pagination, infinite loading, optimistic updates
- ✅ **Flexible**: Works with any async data source
- ✅ **TypeScript**: Full TypeScript support

### **Installation**
```bash
npm install @tanstack/react-query
# For React Native
npm install @tanstack/react-query
```

---

## 📊 **Basic Queries**

### **QueryClient Setup**
```javascript
// App.js
import React from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import AppNavigator from './navigation/AppNavigator';

// Create a client
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      gcTime: 1000 * 60 * 10, // 10 minutes (formerly cacheTime)
      retry: (failureCount, error) => {
        // Don't retry on 4xx errors
        if (error?.status >= 400 && error?.status < 500) {
          return false;
        }
        return failureCount < 3;
      },
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
    },
    mutations: {
      retry: false,
    },
  },
});

const App = () => {
  return (
    <QueryClientProvider client={queryClient}>
      <AppNavigator />
      {__DEV__ && <ReactQueryDevtools initialIsOpen={false} />}
    </QueryClientProvider>
  );
};

export default App;
```

### **Basic Query Hook**
```javascript
// hooks/usePosts.js
import { useQuery } from '@tanstack/react-query';

// API function
const fetchPosts = async (page = 1, limit = 10) => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts?_page=${page}&_limit=${limit}`
  );

  if (!response.ok) {
    throw new Error('Failed to fetch posts');
  }

  return response.json();
};

// Custom hook
export const usePosts = (page = 1, limit = 10) => {
  return useQuery({
    queryKey: ['posts', page, limit],
    queryFn: () => fetchPosts(page, limit),
    keepPreviousData: true,
    // Only run query if page is valid
    enabled: page > 0,
  });
};

// Usage in component
import React from 'react';
import { View, Text, FlatList, ActivityIndicator, StyleSheet } from 'react-native';
import { usePosts } from '../hooks/usePosts';

const PostsList = ({ page = 1 }) => {
  const { data, isLoading, error, isFetching, refetch } = usePosts(page);

  if (isLoading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" color="#007AFF" />
        <Text style={styles.loadingText}>Loading posts...</Text>
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.center}>
        <Text style={styles.errorText}>Error: {error.message}</Text>
        <TouchableOpacity style={styles.retryButton} onPress={refetch}>
          <Text style={styles.retryButtonText}>Retry</Text>
        </TouchableOpacity>
      </View>
    );
  }

  return (
    <View style={styles.container}>
      {isFetching && (
        <View style={styles.refetchingIndicator}>
          <ActivityIndicator size="small" color="#007AFF" />
          <Text style={styles.refetchingText}>Refreshing...</Text>
        </View>
      )}

      <FlatList
        data={data}
        keyExtractor={(item) => item.id.toString()}
        renderItem={({ item }) => (
          <View style={styles.postItem}>
            <Text style={styles.postTitle}>{item.title}</Text>
            <Text style={styles.postBody}>{item.body}</Text>
          </View>
        )}
        onRefresh={refetch}
        refreshing={isFetching}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  center: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  loadingText: {
    marginTop: 10,
    fontSize: 16,
    color: '#666',
  },
  errorText: {
    fontSize: 16,
    color: '#d32f2f',
    textAlign: 'center',
    marginBottom: 20,
  },
  retryButton: {
    backgroundColor: '#007AFF',
    padding: 12,
    borderRadius: 8,
  },
  retryButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  refetchingIndicator: {
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
    padding: 10,
    backgroundColor: '#e3f2fd',
  },
  refetchingText: {
    marginLeft: 10,
    fontSize: 14,
    color: '#1976d2',
  },
  postItem: {
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
    backgroundColor: 'white',
  },
  postTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  postBody: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
  },
});

export default PostsList;
```

### **Query with Parameters**
```javascript
// hooks/useUser.js
import { useQuery } from '@tanstack/react-query';

const fetchUser = async (userId) => {
  const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`);

  if (!response.ok) {
    throw new Error('Failed to fetch user');
  }

  return response.json();
};

export const useUser = (userId) => {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    enabled: !!userId, // Only run if userId exists
    staleTime: 1000 * 60 * 30, // 30 minutes
    cacheTime: 1000 * 60 * 60, // 1 hour
    retry: (failureCount, error) => {
      // Don't retry on 404
      if (error?.status === 404) return false;
      return failureCount < 2;
    },
    select: (data) => ({
      ...data,
      fullName: `${data.name} (${data.username})`,
    }),
  });
};

// Component with dependent queries
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { useUser } from '../hooks/useUser';
import { usePosts } from '../hooks/usePosts';

const UserProfile = ({ userId }) => {
  const { data: user, isLoading: userLoading, error: userError } = useUser(userId);
  const { data: posts, isLoading: postsLoading } = usePosts(1, 5);

  if (userLoading) {
    return <Text>Loading user...</Text>;
  }

  if (userError) {
    return <Text>Error: {userError.message}</Text>;
  }

  return (
    <View style={styles.container}>
      <View style={styles.userInfo}>
        <Text style={styles.name}>{user.fullName}</Text>
        <Text style={styles.email}>{user.email}</Text>
        <Text style={styles.phone}>{user.phone}</Text>
      </View>

      <View style={styles.postsSection}>
        <Text style={styles.sectionTitle}>Recent Posts</Text>
        {postsLoading ? (
          <Text>Loading posts...</Text>
        ) : (
          posts?.slice(0, 3).map(post => (
            <View key={post.id} style={styles.postItem}>
              <Text style={styles.postTitle}>{post.title}</Text>
            </View>
          ))
        )}
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 20,
  },
  userInfo: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 10,
    marginBottom: 20,
  },
  name: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  email: {
    fontSize: 16,
    color: '#666',
    marginBottom: 4,
  },
  phone: {
    fontSize: 16,
    color: '#666',
  },
  postsSection: {
    backgroundColor: 'white',
    padding: 20,
    borderRadius: 10,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 15,
  },
  postItem: {
    padding: 10,
    backgroundColor: '#f5f5f5',
    borderRadius: 5,
    marginBottom: 8,
  },
  postTitle: {
    fontSize: 14,
    fontWeight: '500',
  },
});

export default UserProfile;
```

---

## 🔄 **Mutations**

### **Basic Mutation**
```javascript
// hooks/useCreatePost.js
import { useMutation, useQueryClient } from '@tanstack/react-query';

const createPost = async (postData) => {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(postData),
  });

  if (!response.ok) {
    throw new Error('Failed to create post');
  }

  return response.json();
};

export const useCreatePost = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createPost,
    onSuccess: (newPost) => {
      // Invalidate and refetch posts
      queryClient.invalidateQueries({ queryKey: ['posts'] });

      // Optimistically update the cache
      queryClient.setQueryData(['posts', 1, 10], (oldData) => {
        if (!oldData) return [newPost];
        return [newPost, ...oldData];
      });
    },
    onError: (error) => {
      console.error('Create post error:', error);
      // Could show a toast notification here
    },
  });
};

// Usage in component
import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, StyleSheet, Alert } from 'react-native';
import { useCreatePost } from '../hooks/useCreatePost';

const CreatePostForm = () => {
  const [title, setTitle] = useState('');
  const [body, setBody] = useState('');

  const createPostMutation = useCreatePost();

  const handleSubmit = async () => {
    if (!title.trim() || !body.trim()) {
      Alert.alert('Error', 'Please fill in both title and body');
      return;
    }

    try {
      await createPostMutation.mutateAsync({
        title: title.trim(),
        body: body.trim(),
        userId: 1, // In real app, get from auth context
      });

      // Clear form on success
      setTitle('');
      setBody('');
      Alert.alert('Success', 'Post created successfully!');
    } catch (error) {
      Alert.alert('Error', 'Failed to create post');
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Create New Post</Text>

      <TextInput
        style={styles.input}
        placeholder="Post title..."
        value={title}
        onChangeText={setTitle}
        multiline
      />

      <TextInput
        style={[styles.input, styles.bodyInput]}
        placeholder="Post content..."
        value={body}
        onChangeText={setBody}
        multiline
        numberOfLines={4}
      />

      <TouchableOpacity
        style={[
          styles.submitButton,
          createPostMutation.isLoading && styles.disabledButton,
        ]}
        onPress={handleSubmit}
        disabled={createPostMutation.isLoading}
      >
        <Text style={styles.submitButtonText}>
          {createPostMutation.isLoading ? 'Creating...' : 'Create Post'}
        </Text>
      </TouchableOpacity>

      {createPostMutation.isError && (
        <Text style={styles.errorText}>
          Error: {createPostMutation.error?.message}
        </Text>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 8,
    padding: 12,
    marginBottom: 15,
    fontSize: 16,
  },
  bodyInput: {
    height: 100,
    textAlignVertical: 'top',
  },
  submitButton: {
    backgroundColor: '#28a745',
    padding: 15,
    borderRadius: 8,
    alignItems: 'center',
  },
  disabledButton: {
    backgroundColor: '#6c757d',
  },
  submitButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  errorText: {
    color: '#dc3545',
    textAlign: 'center',
    marginTop: 10,
  },
});

export default CreatePostForm;
```

### **Advanced Mutation with Rollback**
```javascript
// hooks/useUpdatePost.js
import { useMutation, useQueryClient } from '@tanstack/react-query';

const updatePost = async ({ id, ...updates }) => {
  const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`, {
    method: 'PATCH',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(updates),
  });

  if (!response.ok) {
    throw new Error('Failed to update post');
  }

  return response.json();
};

export const useUpdatePost = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updatePost,
    onMutate: async (updatedPost) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['posts'] });

      // Snapshot previous value
      const previousPosts = queryClient.getQueryData(['posts']);

      // Optimistically update cache
      queryClient.setQueryData(['posts'], (old) => {
        if (!old) return old;
        return old.map(post =>
          post.id === updatedPost.id
            ? { ...post, ...updatedPost }
            : post
        );
      });

      // Return context with snapshotted value
      return { previousPosts };
    },
    onError: (err, updatedPost, context) => {
      // Rollback on error
      if (context?.previousPosts) {
        queryClient.setQueryData(['posts'], context.previousPosts);
      }
    },
    onSettled: () => {
      // Always refetch after error or success
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });
};
```

---

## 🔄 **Query Invalidation**

### **Manual Invalidation**
```javascript
// hooks/useInvalidateQueries.js
import { useQueryClient } from '@tanstack/react-query';

export const useInvalidateQueries = () => {
  const queryClient = useQueryClient();

  const invalidatePosts = () => {
    queryClient.invalidateQueries({ queryKey: ['posts'] });
  };

  const invalidateUser = (userId) => {
    queryClient.invalidateQueries({ queryKey: ['user', userId] });
  };

  const invalidateAll = () => {
    queryClient.invalidateQueries();
  };

  const refetchPosts = () => {
    queryClient.refetchQueries({ queryKey: ['posts'] });
  };

  return {
    invalidatePosts,
    invalidateUser,
    invalidateAll,
    refetchPosts,
  };
};
```

### **Selective Invalidation**
```javascript
// Invalidate specific queries
const invalidateSpecificPost = (postId) => {
  queryClient.invalidateQueries({
    queryKey: ['post', postId],
    exact: true, // Only exact matches
  });
};

// Invalidate with predicate
const invalidateUserPosts = (userId) => {
  queryClient.invalidateQueries({
    predicate: (query) => {
      return query.queryKey[0] === 'posts' &&
             query.queryKey[1] === userId;
    },
  });
};

// Invalidate and refetch
const refreshData = async () => {
  await queryClient.invalidateQueries({ queryKey: ['posts'] });
  await queryClient.refetchQueries({ queryKey: ['posts'] });
};
```

---

## 💾 **Caching Strategies**

### **Cache Configuration**
```javascript
// Advanced QueryClient configuration
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      gcTime: 10 * 60 * 1000, // 10 minutes
      refetchOnWindowFocus: false,
      refetchOnReconnect: true,
      refetchInterval: false,
      refetchIntervalInBackground: false,
      retry: (failureCount, error) => {
        // Custom retry logic
        if (error?.status === 404) return false;
        if (error?.status >= 500) return failureCount < 3;
        return false;
      },
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
    },
    mutations: {
      retry: (failureCount, error) => {
        // Don't retry mutations by default
        return false;
      },
      onError: (error) => {
        console.error('Mutation error:', error);
      },
    },
  },
});
```

### **Background Refetching**
```javascript
// Background sync hook
import { useQuery } from '@tanstack/react-query';
import { useEffect } from 'react';
import NetInfo from '@react-native-community/netinfo';

export const useBackgroundSync = (queryKey, queryFn, options = {}) => {
  const query = useQuery({
    queryKey,
    queryFn,
    ...options,
  });

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(state => {
      if (state.isConnected && state.type !== 'none') {
        // Refetch when connection is restored
        query.refetch();
      }
    });

    return unsubscribe;
  }, [query]);

  return query;
};

// Usage
const usePostsWithBackgroundSync = () => {
  return useBackgroundSync(
    ['posts'],
    fetchPosts,
    {
      staleTime: 5 * 60 * 1000,
      enabled: true,
    }
  );
};
```

---

## 🚨 **Error Handling**

### **Global Error Handling**
```javascript
// Error boundary for React Query
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

class QueryErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log error to service
    console.error('Query Error:', error, errorInfo);
  }

  handleRetry = () => {
    this.setState({ hasError: false, error: null });
    // Optionally refetch queries
    this.props.queryClient?.resetQueries();
  };

  render() {
    if (this.state.hasError) {
      return (
        <View style={styles.container}>
          <Text style={styles.title}>Something went wrong</Text>
          <Text style={styles.message}>
            {this.state.error?.message || 'An unexpected error occurred'}
          </Text>
          <TouchableOpacity style={styles.retryButton} onPress={this.handleRetry}>
            <Text style={styles.retryButtonText}>Try Again</Text>
          </TouchableOpacity>
        </View>
      );
    }

    return this.props.children;
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  message: {
    fontSize: 16,
    textAlign: 'center',
    marginBottom: 20,
    color: '#666',
  },
  retryButton: {
    backgroundColor: '#007AFF',
    padding: 12,
    borderRadius: 8,
  },
  retryButtonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default QueryErrorBoundary;
```

### **Query Error Handling**
```javascript
// Custom error handler hook
import { useQuery } from '@tanstack/react-query';
import { useState, useEffect } from 'react';

export const useQueryWithErrorHandling = (queryKey, queryFn, options = {}) => {
  const [errorMessage, setErrorMessage] = useState(null);

  const query = useQuery({
    queryKey,
    queryFn,
    ...options,
    onError: (error) => {
      // Custom error handling
      if (error?.status === 401) {
        // Handle unauthorized
        setErrorMessage('Please log in to continue');
      } else if (error?.status === 403) {
        // Handle forbidden
        setErrorMessage('You do not have permission to access this resource');
      } else if (error?.status >= 500) {
        // Handle server errors
        setErrorMessage('Server error. Please try again later.');
      } else {
        // Handle other errors
        setErrorMessage(error?.message || 'An error occurred');
      }

      // Call original onError if provided
      if (options.onError) {
        options.onError(error);
      }
    },
    onSuccess: () => {
      // Clear error on success
      setErrorMessage(null);

      // Call original onSuccess if provided
      if (options.onSuccess) {
        options.onSuccess();
      }
    },
  });

  return {
    ...query,
    errorMessage,
    clearError: () => setErrorMessage(null),
  };
};
```

---

## ⚡ **Optimistic Updates**

### **Optimistic Update Pattern**
```javascript
// hooks/useOptimisticUpdate.js
import { useMutation, useQueryClient } from '@tanstack/react-query';

export const useOptimisticUpdate = (mutationFn, queryKey) => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn,
    onMutate: async (variables) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey });

      // Snapshot previous value
      const previousData = queryClient.getQueryData(queryKey);

      // Optimistically update cache
      queryClient.setQueryData(queryKey, (old) => {
        if (!old) return old;

        // Apply optimistic update based on mutation type
        if (variables.type === 'add') {
          return [...old, { ...variables.data, id: `temp-${Date.now()}`, isOptimistic: true }];
        }

        if (variables.type === 'update') {
          return old.map(item =>
            item.id === variables.data.id
              ? { ...item, ...variables.data, isOptimistic: true }
              : item
          );
        }

        if (variables.type === 'delete') {
          return old.filter(item => item.id !== variables.id);
        }

        return old;
      });

      return { previousData };
    },
    onError: (err, variables, context) => {
      // Rollback on error
      if (context?.previousData) {
        queryClient.setQueryData(queryKey, context.previousData);
      }
    },
    onSettled: () => {
      // Always refetch after error or success
      queryClient.invalidateQueries({ queryKey });
    },
  });
};

// Usage
const updatePostOptimistically = useOptimisticUpdate(
  async ({ id, ...updates }) => {
    const response = await fetch(`/api/posts/${id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(updates),
    });
    return response.json();
  },
  ['posts']
);

// In component
const handleUpdatePost = (postId, updates) => {
  updatePostOptimistically.mutate({
    type: 'update',
    data: { id: postId, ...updates },
  });
};
```

---

## 🔄 **Infinite Queries**

### **Infinite Scroll Implementation**
```javascript
// hooks/useInfinitePosts.js
import { useInfiniteQuery } from '@tanstack/react-query';

const fetchPostsPage = async ({ pageParam = 1 }) => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts?_page=${pageParam}&_limit=10`
  );

  if (!response.ok) {
    throw new Error('Failed to fetch posts');
  }

  return response.json();
};

export const useInfinitePosts = () => {
  return useInfiniteQuery({
    queryKey: ['posts', 'infinite'],
    queryFn: fetchPostsPage,
    getNextPageParam: (lastPage, allPages) => {
      // If last page has less than 10 items, no more pages
      if (lastPage.length < 10) {
        return undefined;
      }
      // Return next page number
      return allPages.length + 1;
    },
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};

// Infinite scroll component
import React from 'react';
import { FlatList, View, Text, ActivityIndicator, StyleSheet } from 'react-native';
import { useInfinitePosts } from '../hooks/useInfinitePosts';

const InfinitePostsList = () => {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
    error,
  } = useInfinitePosts();

  if (isLoading) {
    return (
      <View style={styles.center}>
        <ActivityIndicator size="large" color="#007AFF" />
      </View>
    );
  }

  if (error) {
    return (
      <View style={styles.center}>
        <Text style={styles.errorText}>Error: {error.message}</Text>
      </View>
    );
  }

  // Flatten all pages into single array
  const posts = data?.pages.flat() || [];

  const renderItem = ({ item }) => (
    <View style={styles.postItem}>
      <Text style={styles.postTitle}>{item.title}</Text>
      <Text style={styles.postBody}>{item.body}</Text>
    </View>
  );

  const renderFooter = () => {
    if (isFetchingNextPage) {
      return (
        <View style={styles.footer}>
          <ActivityIndicator size="small" color="#007AFF" />
          <Text style={styles.footerText}>Loading more...</Text>
        </View>
      );
    }

    if (!hasNextPage) {
      return (
        <View style={styles.footer}>
          <Text style={styles.footerText}>No more posts</Text>
        </View>
      );
    }

    return null;
  };

  return (
    <FlatList
      data={posts}
      renderItem={renderItem}
      keyExtractor={(item) => item.id.toString()}
      onEndReached={() => {
        if (hasNextPage && !isFetchingNextPage) {
          fetchNextPage();
        }
      }}
      onEndReachedThreshold={0.5}
      ListFooterComponent={renderFooter}
      style={styles.container}
    />
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  center: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  errorText: {
    color: '#d32f2f',
    fontSize: 16,
  },
  postItem: {
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
    backgroundColor: 'white',
  },
  postTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  postBody: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20,
  },
  footer: {
    padding: 20,
    alignItems: 'center',
  },
  footerText: {
    marginTop: 10,
    fontSize: 14,
    color: '#666',
  },
});

export default InfinitePostsList;
```

---

## 🔄 **Background Refetching**

### **Background Sync Setup**
```javascript
// Background sync utility
import { useQuery, useQueryClient } from '@tanstack/react-query';
import { AppState } from 'react-native';
import { useEffect } from 'react';

export const useBackgroundRefetch = (queryKey, queryFn, options = {}) => {
  const queryClient = useQueryClient();

  const query = useQuery({
    queryKey,
    queryFn,
    ...options,
  });

  useEffect(() => {
    const subscription = AppState.addEventListener('change', (nextAppState) => {
      if (nextAppState === 'active') {
        // App came to foreground, refetch data
        queryClient.invalidateQueries({ queryKey });
      }
    });

    return () => {
      subscription?.remove();
    };
  }, [queryClient, queryKey]);

  return query;
};

// Network-aware background sync
import NetInfo from '@react-native-community/netinfo';

export const useNetworkAwareRefetch = (queryKey, queryFn, options = {}) => {
  const query = useQuery({
    queryKey,
    queryFn,
    ...options,
    refetchOnReconnect: true,
  });

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(state => {
      if (state.isConnected && state.type !== 'none') {
        // Network restored, refetch if data is stale
        const queryData = queryClient.getQueryState(queryKey);
        if (queryData?.isStale) {
          query.refetch();
        }
      }
    });

    return unsubscribe;
  }, [query, queryKey]);

  return query;
};
```

---

## 🛠️ **DevTools Integration**

### **React Query DevTools**
```javascript
// App.js
import React from 'react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import AppNavigator from './navigation/AppNavigator';

const queryClient = new QueryClient();

const App = () => {
  return (
    <QueryClientProvider client={queryClient}>
      <AppNavigator />
      {__DEV__ && (
        <ReactQueryDevtools
          initialIsOpen={false}
          position="bottom-right"
        />
      )}
    </QueryClientProvider>
  );
};

export default App;
```

### **Custom DevTools Panel**
```javascript
// components/QueryDevTools.js
import React from 'react';
import { View, Text, ScrollView, TouchableOpacity, StyleSheet } from 'react-native';
import { useQueryClient } from '@tanstack/react-query';

const QueryDevTools = () => {
  const queryClient = useQueryClient();

  const queries = queryClient.getQueryCache().getAll();
  const mutations = queryClient.getMutationCache().getAll();

  const invalidateAll = () => {
    queryClient.invalidateQueries();
  };

  const clearAll = () => {
    queryClient.clear();
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>React Query DevTools</Text>

      <View style={styles.controls}>
        <TouchableOpacity style={styles.button} onPress={invalidateAll}>
          <Text style={styles.buttonText}>Invalidate All</Text>
        </TouchableOpacity>

        <TouchableOpacity style={[styles.button, styles.dangerButton]} onPress={clearAll}>
          <Text style={styles.buttonText}>Clear All</Text>
        </TouchableOpacity>
      </View>

      <ScrollView style={styles.queriesList}>
        <Text style={styles.sectionTitle}>Queries ({queries.length})</Text>
        {queries.map((query) => (
          <View key={query.queryKey.join('-')} style={styles.queryItem}>
            <Text style={styles.queryKey}>{JSON.stringify(query.queryKey)}</Text>
            <Text style={styles.queryState}>
              State: {query.state.status}
              {query.state.isFetching && ' (fetching)'}
            </Text>
            <Text style={styles.queryData}>
              Data: {query.state.data ? 'Present' : 'None'}
            </Text>
          </View>
        ))}

        <Text style={styles.sectionTitle}>Mutations ({mutations.length})</Text>
        {mutations.map((mutation, index) => (
          <View key={index} style={styles.mutationItem}>
            <Text style={styles.mutationState}>
              State: {mutation.state.status}
            </Text>
          </View>
        ))}
      </ScrollView>
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
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  controls: {
    flexDirection: 'row',
    marginBottom: 20,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 10,
    borderRadius: 5,
    marginRight: 10,
  },
  dangerButton: {
    backgroundColor: '#dc3545',
  },
  buttonText: {
    color: 'white',
    fontWeight: 'bold',
  },
  queriesList: {
    flex: 1,
  },
  sectionTitle: {
    fontSize: 18,
    fontWeight: 'bold',
    marginTop: 20,
    marginBottom: 10,
  },
  queryItem: {
    backgroundColor: 'white',
    padding: 10,
    borderRadius: 5,
    marginBottom: 10,
  },
  queryKey: {
    fontSize: 14,
    fontWeight: 'bold',
    marginBottom: 5,
  },
  queryState: {
    fontSize: 12,
    color: '#666',
  },
  queryData: {
    fontSize: 12,
    color: '#666',
  },
  mutationItem: {
    backgroundColor: 'white',
    padding: 10,
    borderRadius: 5,
    marginBottom: 10,
  },
  mutationState: {
    fontSize: 14,
  },
});

export default QueryDevTools;
```

---

## 🎯 **Practical Examples**

### **Complete Blog App with React Query**
```javascript
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

// Screens
import HomeScreen from './screens/HomeScreen';
import PostDetailScreen from './screens/PostDetailScreen';
import CreatePostScreen from './screens/CreatePostScreen';

const Stack = createStackNavigator();
const queryClient = new QueryClient();

const App = () => {
  return (
    <QueryClientProvider client={queryClient}>
      <NavigationContainer>
        <Stack.Navigator>
          <Stack.Screen name="Home" component={HomeScreen} />
          <Stack.Screen name="PostDetail" component={PostDetailScreen} />
          <Stack.Screen name="CreatePost" component={CreatePostScreen} />
        </Stack.Navigator>
      </NavigationContainer>
      {__DEV__ && <ReactQueryDevtools />}
    </QueryClientProvider>
  );
};

export default App;
```

### **Blog API Service**
```javascript
// services/blogApi.js
const API_BASE_URL = 'https://jsonplaceholder.typicode.com';

export const blogApi = {
  // Posts
  getPosts: async (page = 1, limit = 10) => {
    const response = await fetch(
      `${API_BASE_URL}/posts?_page=${page}&_limit=${limit}`
    );
    if (!response.ok) throw new Error('Failed to fetch posts');
    return response.json();
  },

  getPost: async (id) => {
    const response = await fetch(`${API_BASE_URL}/posts/${id}`);
    if (!response.ok) throw new Error('Failed to fetch post');
    return response.json();
  },

  createPost: async (post) => {
    const response = await fetch(`${API_BASE_URL}/posts`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(post),
    });
    if (!response.ok) throw new Error('Failed to create post');
    return response.json();
  },

  updatePost: async (id, updates) => {
    const response = await fetch(`${API_BASE_URL}/posts/${id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(updates),
    });
    if (!response.ok) throw new Error('Failed to update post');
    return response.json();
  },

  deletePost: async (id) => {
    const response = await fetch(`${API_BASE_URL}/posts/${id}`, {
      method: 'DELETE',
    });
    if (!response.ok) throw new Error('Failed to delete post');
    return response.json();
  },

  // Comments
  getComments: async (postId) => {
    const response = await fetch(`${API_BASE_URL}/posts/${postId}/comments`);
    if (!response.ok) throw new Error('Failed to fetch comments');
    return response.json();
  },
};
```

### **Custom Hooks for Blog**
```javascript
// hooks/useBlog.js
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { blogApi } from '../services/blogApi';

// Posts hooks
export const usePosts = (page = 1, limit = 10) => {
  return useQuery({
    queryKey: ['posts', page, limit],
    queryFn: () => blogApi.getPosts(page, limit),
    keepPreviousData: true,
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};

export const usePost = (id) => {
  return useQuery({
    queryKey: ['post', id],
    queryFn: () => blogApi.getPost(id),
    enabled: !!id,
    staleTime: 10 * 60 * 1000, // 10 minutes
  });
};

export const useCreatePost = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: blogApi.createPost,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });
};

export const useUpdatePost = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, ...updates }) => blogApi.updatePost(id, updates),
    onSuccess: (updatedPost) => {
      queryClient.invalidateQueries({ queryKey: ['posts'] });
      queryClient.invalidateQueries({ queryKey: ['post', updatedPost.id] });
    },
  });
};

export const useDeletePost = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: blogApi.deletePost,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts'] });
    },
  });
};

// Comments hooks
export const useComments = (postId) => {
  return useQuery({
    queryKey: ['comments', postId],
    queryFn: () => blogApi.getComments(postId),
    enabled: !!postId,
    staleTime: 2 * 60 * 1000, // 2 minutes
  });
};
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **React Query Setup**: QueryClient configuration and provider setup
- ✅ **Basic Queries**: useQuery hook for data fetching
- ✅ **Mutations**: useMutation for data modifications
- ✅ **Query Invalidation**: Cache management and updates
- ✅ **Caching Strategies**: Stale time, cache time, and background refetching
- ✅ **Error Handling**: Global and query-specific error handling
- ✅ **Optimistic Updates**: Immediate UI updates with rollback
- ✅ **Infinite Queries**: Pagination and infinite scrolling
- ✅ **Background Sync**: Network-aware data synchronization
- ✅ **DevTools**: Debugging and development tools

### **Best Practices**
1. **Configure QueryClient properly** for your app's needs
2. **Use descriptive query keys** for cache management
3. **Handle loading and error states** appropriately
4. **Implement optimistic updates** for better UX
5. **Use query invalidation** strategically
6. **Set appropriate stale times** based on data freshness needs
7. **Handle network errors gracefully** with retry logic
8. **Use DevTools in development** for debugging

### **Next Steps**
- Learn about React Query v4 features and migration
- Implement advanced caching strategies
- Add offline support with React Query
- Create custom query hooks for your API
- Set up automated testing for queries and mutations

---

## 🎯 **Assignment**

### **Task 1: Todo App with React Query**
Create a comprehensive todo application with:
- Fetch todos from API with React Query
- Add, update, and delete todos with optimistic updates
- Handle loading states and error handling
- Implement offline support with background sync
- Add infinite scrolling for large todo lists

### **Task 2: E-commerce Product Catalog**
Build a product catalog with:
- Fetch products with pagination
- Search and filter products
- Add to cart with optimistic updates
- Real-time inventory updates
- Product reviews and ratings

### **Task 3: Social Media Feed**
Implement a social media feed with:
- Infinite scrolling for posts
- Real-time updates for new posts
- Like/unlike with optimistic updates
- Comments system with background sync
- User authentication integration

---

## 📚 **Additional Resources**
- [React Query Documentation](https://tanstack.com/query/latest)
- [React Query DevTools](https://tanstack.com/query/devtools)
- [React Query Examples](https://tanstack.com/query/examples)
- [API Integration Patterns](https://tkdodo.eu/blog/practical-react-query)
- [React Query Best Practices](https://tkdodo.eu/blog/react-query-error-handling)

---

**Next Lesson**: [Lesson 36: Advanced Navigation Patterns](Lesson%2036_%20Advanced%20Navigation%20Patterns.md)