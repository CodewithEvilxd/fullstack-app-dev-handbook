# Lesson 48: Advanced State Management Patterns

## 🎯 **Learning Objectives**
- Master advanced state management patterns in React Native
- Implement complex state architectures with multiple libraries
- Create scalable state management solutions
- Handle asynchronous state updates and side effects
- Optimize state performance and memory usage

## 📚 **Table of Contents**
1. [State Management Fundamentals](#state-management-fundamentals)
2. [Redux Advanced Patterns](#redux-advanced-patterns)
3. [Zustand for Simple State](#zustand-for-simple-state)
4. [Recoil for React Native](#recoil-for-react-native)
5. [Jotai Atomic State](#jotai-atomic-state)
6. [XState State Machines](#xstate-state-machines)
7. [MobX Reactive State](#mobx-reactive-state)
8. [State Persistence](#state-persistence)
9. [Performance Optimization](#performance-optimization)
10. [Practical Examples](#practical-examples)

---

## 🏗️ **State Management Fundamentals**

### **State Management Evolution**
- **Local State**: useState, useReducer for component-level state
- **Global State**: Context API for simple global state
- **Advanced State**: Redux, Zustand, Recoil for complex applications
- **Reactive State**: MobX for reactive programming
- **State Machines**: XState for complex state transitions

### **When to Use Different Solutions**
- **useState/useReducer**: Simple component state
- **Context API**: Small to medium apps with simple global state
- **Redux**: Large apps with complex state logic and debugging needs
- **Zustand**: Medium apps needing simple but powerful state management
- **Recoil**: Apps with complex derived state and async operations
- **MobX**: Apps needing reactive programming and automatic updates
- **XState**: Apps with complex state machines and workflows

---

## 🔄 **Redux Advanced Patterns**

### **Redux Toolkit Setup**
```javascript
// store/index.js
import { configureStore } from '@reduxjs/toolkit';
import userSlice from './slices/userSlice';
import postsSlice from './slices/postsSlice';
import uiSlice from './slices/uiSlice';

export const store = configureStore({
  reducer: {
    user: userSlice,
    posts: postsSlice,
    ui: uiSlice,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE'],
      },
    }),
  devTools: process.env.NODE_ENV !== 'production',
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### **Advanced Slice with Async Thunks**
```javascript
// store/slices/userSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import api from '../../services/api';

// Async thunks
export const loginUser = createAsyncThunk(
  'user/login',
  async ({ email, password }, { rejectWithValue }) => {
    try {
      const response = await api.post('/auth/login', { email, password });
      return response.data;
    } catch (error) {
      return rejectWithValue(error.response?.data || 'Login failed');
    }
  }
);

export const fetchUserProfile = createAsyncThunk(
  'user/fetchProfile',
  async (userId, { rejectWithValue }) => {
    try {
      const response = await api.get(`/users/${userId}`);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.response?.data || 'Failed to fetch profile');
    }
  }
);

export const updateUserProfile = createAsyncThunk(
  'user/updateProfile',
  async ({ userId, profileData }, { rejectWithValue }) => {
    try {
      const response = await api.put(`/users/${userId}`, profileData);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.response?.data || 'Failed to update profile');
    }
  }
);

// Slice
const userSlice = createSlice({
  name: 'user',
  initialState: {
    currentUser: null,
    profile: null,
    loading: false,
    error: null,
    isAuthenticated: false,
    lastLogin: null,
  },
  reducers: {
    logout: (state) => {
      state.currentUser = null;
      state.profile = null;
      state.isAuthenticated = false;
      state.lastLogin = null;
    },
    clearError: (state) => {
      state.error = null;
    },
    updateLastActivity: (state) => {
      state.lastActivity = new Date().toISOString();
    },
  },
  extraReducers: (builder) => {
    builder
      // Login
      .addCase(loginUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.loading = false;
        state.currentUser = action.payload.user;
        state.isAuthenticated = true;
        state.lastLogin = new Date().toISOString();
        state.error = null;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
        state.isAuthenticated = false;
      })

      // Fetch Profile
      .addCase(fetchUserProfile.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchUserProfile.fulfilled, (state, action) => {
        state.loading = false;
        state.profile = action.payload;
      })
      .addCase(fetchUserProfile.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      })

      // Update Profile
      .addCase(updateUserProfile.pending, (state) => {
        state.loading = true;
      })
      .addCase(updateUserProfile.fulfilled, (state, action) => {
        state.loading = false;
        state.profile = action.payload;
        state.error = null;
      })
      .addCase(updateUserProfile.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  },
});

export const { logout, clearError, updateLastActivity } = userSlice.actions;
export default userSlice.reducer;
```

### **Redux Selectors and Reselect**
```javascript
// store/selectors/userSelectors.js
import { createSelector } from 'reselect';

// Basic selectors
export const selectUser = (state) => state.user;
export const selectCurrentUser = (state) => state.user.currentUser;
export const selectUserProfile = (state) => state.user.profile;
export const selectUserLoading = (state) => state.user.loading;
export const selectUserError = (state) => state.user.error;
export const selectIsAuthenticated = (state) => state.user.isAuthenticated;

// Memoized selectors using reselect
export const selectUserFullName = createSelector(
  [selectCurrentUser],
  (user) => {
    if (!user) return '';
    return `${user.firstName} ${user.lastName}`.trim();
  }
);

export const selectUserDisplayName = createSelector(
  [selectCurrentUser],
  (user) => {
    if (!user) return 'Guest';
    return user.displayName || user.username || selectUserFullName({ user });
  }
);

export const selectUserInitials = createSelector(
  [selectCurrentUser],
  (user) => {
    if (!user) return 'GU';
    const firstName = user.firstName || '';
    const lastName = user.lastName || '';
    return `${firstName.charAt(0)}${lastName.charAt(0)}`.toUpperCase() || 'GU';
  }
);

export const selectUserPermissions = createSelector(
  [selectCurrentUser],
  (user) => {
    return user?.permissions || [];
  }
);

export const selectHasPermission = (permission) => createSelector(
  [selectUserPermissions],
  (permissions) => {
    return permissions.includes(permission);
  }
);

export const selectUserStats = createSelector(
  [selectUserProfile],
  (profile) => {
    if (!profile) return null;

    return {
      postsCount: profile.postsCount || 0,
      followersCount: profile.followersCount || 0,
      followingCount: profile.followingCount || 0,
      joinDate: profile.createdAt,
      lastActive: profile.lastActive,
    };
  }
);

export const selectUserStatus = createSelector(
  [selectUser],
  (user) => {
    if (user.loading) return 'loading';
    if (user.error) return 'error';
    if (user.isAuthenticated) return 'authenticated';
    return 'unauthenticated';
  }
);
```

### **Redux Saga for Complex Async Logic**
```javascript
// store/sagas/userSaga.js
import { call, put, takeEvery, takeLatest, delay, select } from 'redux-saga/effects';
import {
  loginUser,
  fetchUserProfile,
  updateUserProfile,
  logout,
} from '../slices/userSlice';
import api from '../../services/api';

// Worker sagas
function* loginUserSaga(action) {
  try {
    const { email, password } = action.payload;

    // Call API
    const response = yield call(api.post, '/auth/login', { email, password });

    // Store token
    yield call(storeAuthToken, response.data.token);

    // Fetch user profile
    yield put(fetchUserProfile(response.data.user.id));

    // Navigate to main screen
    yield call(navigateToMain);

  } catch (error) {
    yield put(loginUser.rejected(error.message));
  }
}

function* fetchUserProfileSaga(action) {
  try {
    const userId = action.payload;
    const response = yield call(api.get, `/users/${userId}`);
    yield put(fetchUserProfile.fulfilled(response.data));
  } catch (error) {
    yield put(fetchUserProfile.rejected(error.message));
  }
}

function* updateUserProfileSaga(action) {
  try {
    const { userId, profileData } = action.payload;

    // Optimistic update
    yield put(updateUserProfile.fulfilled({
      ...profileData,
      id: userId,
      updatedAt: new Date().toISOString(),
    }));

    // API call
    const response = yield call(api.put, `/users/${userId}`, profileData);

    // Confirm update
    yield put(updateUserProfile.fulfilled(response.data));

  } catch (error) {
    // Revert optimistic update on error
    yield put(updateUserProfile.rejected(error.message));
    yield call(revertProfileUpdate);
  }
}

function* logoutSaga() {
  try {
    // Clear stored token
    yield call(clearAuthToken);

    // Clear user data
    yield put(logout());

    // Navigate to login
    yield call(navigateToLogin);

  } catch (error) {
    console.error('Logout error:', error);
  }
}

function* watchUserActivity() {
  while (true) {
    // Wait for any user action
    yield takeEvery('*', function* (action) {
      if (action.type.includes('user/')) {
        // Update last activity
        yield put(updateLastActivity());

        // Debounce API call to update server
        yield delay(5000);
        yield call(updateLastActivityOnServer);
      }
    });
  }
}

function* autoRefreshTokenSaga() {
  while (true) {
    try {
      // Check if token needs refresh
      const tokenExpiry = yield select(selectTokenExpiry);
      const now = Date.now();

      if (tokenExpiry - now < 5 * 60 * 1000) { // 5 minutes
        yield call(refreshAuthToken);
      }

      yield delay(60000); // Check every minute
    } catch (error) {
      console.error('Auto refresh error:', error);
      yield delay(30000); // Retry in 30 seconds
    }
  }
}

// Watcher sagas
export function* watchUserActions() {
  yield takeLatest(loginUser.pending.type, loginUserSaga);
  yield takeEvery(fetchUserProfile.pending.type, fetchUserProfileSaga);
  yield takeLatest(updateUserProfile.pending.type, updateUserProfileSaga);
  yield takeEvery(logout.type, logoutSaga);
}

export function* watchUserActivitySaga() {
  yield watchUserActivity();
}

export function* watchAutoRefreshSaga() {
  yield autoRefreshTokenSaga();
}

// Helper functions
function* storeAuthToken(token) {
  // Store in secure storage
}

function* clearAuthToken() {
  // Clear from secure storage
}

function* navigateToMain() {
  // Navigation logic
}

function* navigateToLogin() {
  // Navigation logic
}

function* revertProfileUpdate() {
  // Revert optimistic update
}

function* updateLastActivityOnServer() {
  // Update server
}

function* refreshAuthToken() {
  // Refresh token logic
}
```

---

## 🐻 **Zustand for Simple State**

### **Basic Zustand Store**
```javascript
// store/useAuthStore.js
import create from 'zustand';
import { persist, devtools } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

const useAuthStore = create(
  devtools(
    persist(
      (set, get) => ({
        // State
        user: null,
        token: null,
        isAuthenticated: false,
        loading: false,
        error: null,

        // Actions
        login: async (credentials) => {
          set({ loading: true, error: null });

          try {
            const response = await api.post('/auth/login', credentials);

            set({
              user: response.data.user,
              token: response.data.token,
              isAuthenticated: true,
              loading: false,
            });

            return { success: true };
          } catch (error) {
            set({
              error: error.message,
              loading: false,
            });

            return { success: false, error: error.message };
          }
        },

        logout: () => {
          set({
            user: null,
            token: null,
            isAuthenticated: false,
            error: null,
          });
        },

        updateProfile: (profileData) => {
          set((state) => ({
            user: { ...state.user, ...profileData },
          }));
        },

        clearError: () => {
          set({ error: null });
        },

        // Computed properties
        get displayName() {
          const { user } = get();
          if (!user) return 'Guest';
          return user.displayName || `${user.firstName} ${user.lastName}`.trim();
        },

        get userInitials() {
          const { user } = get();
          if (!user) return 'GU';
          return `${user.firstName?.charAt(0) || ''}${user.lastName?.charAt(0) || ''}`.toUpperCase() || 'GU';
        },
      }),
      {
        name: 'auth-storage',
        getStorage: () => AsyncStorage,
        partialize: (state) => ({
          user: state.user,
          token: state.token,
          isAuthenticated: state.isAuthenticated,
        }),
      }
    ),
    { name: 'auth-store' }
  )
);

export default useAuthStore;
```

### **Advanced Zustand with Slices**
```javascript
// store/slices/usePostsSlice.js
import create from 'zustand';
import { subscribeWithSelector } from 'zustand/middleware';

const usePostsSlice = create(
  subscribeWithSelector((set, get) => ({
    // State
    posts: [],
    loading: false,
    error: null,
    pagination: {
      page: 1,
      limit: 10,
      hasMore: true,
      total: 0,
    },

    // Actions
    fetchPosts: async (page = 1) => {
      set({ loading: true, error: null });

      try {
        const response = await api.get('/posts', {
          params: { page, limit: get().pagination.limit },
        });

        set((state) => ({
          posts: page === 1 ? response.data.posts : [...state.posts, ...response.data.posts],
          pagination: {
            ...state.pagination,
            page,
            hasMore: response.data.hasMore,
            total: response.data.total,
          },
          loading: false,
        }));

        return { success: true };
      } catch (error) {
        set({ error: error.message, loading: false });
        return { success: false, error: error.message };
      }
    },

    createPost: async (postData) => {
      set({ loading: true, error: null });

      try {
        const response = await api.post('/posts', postData);

        set((state) => ({
          posts: [response.data, ...state.posts],
          loading: false,
        }));

        return { success: true, post: response.data };
      } catch (error) {
        set({ error: error.message, loading: false });
        return { success: false, error: error.message };
      }
    },

    updatePost: async (postId, updateData) => {
      try {
        const response = await api.put(`/posts/${postId}`, updateData);

        set((state) => ({
          posts: state.posts.map(post =>
            post.id === postId ? { ...post, ...response.data } : post
          ),
        }));

        return { success: true };
      } catch (error) {
        set({ error: error.message });
        return { success: false, error: error.message };
      }
    },

    deletePost: async (postId) => {
      try {
        await api.delete(`/posts/${postId}`);

        set((state) => ({
          posts: state.posts.filter(post => post.id !== postId),
        }));

        return { success: true };
      } catch (error) {
        set({ error: error.message });
        return { success: false, error: error.message };
      }
    },

    likePost: async (postId) => {
      const { posts } = get();
      const post = posts.find(p => p.id === postId);

      if (!post) return;

      // Optimistic update
      const wasLiked = post.isLiked;
      set((state) => ({
        posts: state.posts.map(p =>
          p.id === postId
            ? {
                ...p,
                isLiked: !wasLiked,
                likesCount: wasLiked ? p.likesCount - 1 : p.likesCount + 1,
              }
            : p
        ),
      }));

      try {
        await api.post(`/posts/${postId}/like`);
      } catch (error) {
        // Revert optimistic update
        set((state) => ({
          posts: state.posts.map(p =>
            p.id === postId
              ? {
                  ...p,
                  isLiked: wasLiked,
                  likesCount: wasLiked ? p.likesCount + 1 : p.likesCount - 1,
                }
              : p
          ),
          error: error.message,
        }));
      }
    },

    // Selectors
    getPostById: (postId) => {
      return get().posts.find(post => post.id === postId);
    },

    getPostsByUser: (userId) => {
      return get().posts.filter(post => post.userId === userId);
    },

    getLikedPosts: () => {
      return get().posts.filter(post => post.isLiked);
    },

    // Computed
    get totalPosts() {
      return get().posts.length;
    },

    get hasMorePosts() {
      return get().pagination.hasMore;
    },
  }))
);

// Subscribe to specific changes
usePostsSlice.subscribe(
  (state) => state.posts.length,
  (postsCount) => {
    console.log('Posts count changed:', postsCount);
  }
);

export default usePostsSlice;
```

---

## ⚛️ **Recoil for React Native**

### **Recoil Atoms and Selectors**
```javascript
// store/atoms/userAtoms.js
import { atom, selector } from 'recoil';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Atoms
export const userState = atom({
  key: 'userState',
  default: null,
  effects_UNSTABLE: [
    ({ setSelf, onSet }) => {
      // Load from AsyncStorage
      const loadUser = async () => {
        try {
          const userData = await AsyncStorage.getItem('user');
          if (userData) {
            setSelf(JSON.parse(userData));
          }
        } catch (error) {
          console.error('Error loading user:', error);
        }
      };

      loadUser();

      // Save to AsyncStorage when state changes
      onSet((newValue) => {
        if (newValue) {
          AsyncStorage.setItem('user', JSON.stringify(newValue));
        } else {
          AsyncStorage.removeItem('user');
        }
      });
    },
  ],
});

export const authTokenState = atom({
  key: 'authTokenState',
  default: null,
  effects_UNSTABLE: [
    ({ setSelf, onSet }) => {
      const loadToken = async () => {
        try {
          const token = await AsyncStorage.getItem('authToken');
          if (token) {
            setSelf(token);
          }
        } catch (error) {
          console.error('Error loading token:', error);
        }
      };

      loadToken();

      onSet((newValue) => {
        if (newValue) {
          AsyncStorage.setItem('authToken', newValue);
        } else {
          AsyncStorage.removeItem('authToken');
        }
      });
    },
  ],
});

export const loadingState = atom({
  key: 'loadingState',
  default: false,
});

export const errorState = atom({
  key: 'errorState',
  default: null,
});

// Selectors
export const isAuthenticatedState = selector({
  key: 'isAuthenticatedState',
  get: ({ get }) => {
    const user = get(userState);
    const token = get(authTokenState);
    return !!(user && token);
  },
});

export const userDisplayNameState = selector({
  key: 'userDisplayNameState',
  get: ({ get }) => {
    const user = get(userState);
    if (!user) return 'Guest';

    return user.displayName ||
           user.username ||
           `${user.firstName || ''} ${user.lastName || ''}`.trim() ||
           'User';
  },
});

export const userInitialsState = selector({
  key: 'userInitialsState',
  get: ({ get }) => {
    const user = get(userState);
    if (!user) return 'GU';

    const firstName = user.firstName || '';
    const lastName = user.lastName || '';

    return `${firstName.charAt(0)}${lastName.charAt(0)}`.toUpperCase() || 'GU';
  },
});

export const userPermissionsState = selector({
  key: 'userPermissionsState',
  get: ({ get }) => {
    const user = get(userState);
    return user?.permissions || [];
  },
});

export const hasPermissionState = selectorFamily({
  key: 'hasPermissionState',
  get: (permission) => ({ get }) => {
    const permissions = get(userPermissionsState);
    return permissions.includes(permission);
  },
});
```

### **Recoil Async Data Queries**
```javascript
// store/selectors/asyncSelectors.js
import { selector, selectorFamily } from 'recoil';
import api from '../../services/api';

// Async selectors
export const userProfileQuery = selectorFamily({
  key: 'userProfileQuery',
  get: (userId) => async () => {
    try {
      const response = await api.get(`/users/${userId}`);
      return response.data;
    } catch (error) {
      throw error;
    }
  },
});

export const userPostsQuery = selectorFamily({
  key: 'userPostsQuery',
  get: (userId) => async () => {
    try {
      const response = await api.get(`/users/${userId}/posts`);
      return response.data;
    } catch (error) {
      return [];
    }
  },
});

export const postsQuery = selector({
  key: 'postsQuery',
  get: async () => {
    try {
      const response = await api.get('/posts');
      return response.data;
    } catch (error) {
      throw error;
    }
  },
});

// Paginated posts
export const paginatedPostsQuery = selectorFamily({
  key: 'paginatedPostsQuery',
  get: ({ page, limit }) => async () => {
    try {
      const response = await api.get('/posts', {
        params: { page, limit },
      });
      return response.data;
    } catch (error) {
      throw error;
    }
  },
});

// Search posts
export const searchPostsQuery = selectorFamily({
  key: 'searchPostsQuery',
  get: (query) => async () => {
    if (!query.trim()) return [];

    try {
      const response = await api.get('/posts/search', {
        params: { q: query },
      });
      return response.data;
    } catch (error) {
      throw error;
    }
  },
});
```

### **Recoil Hooks and Components**
```javascript
// components/UserProfile.js
import React, { Suspense } from 'react';
import { View, Text, ActivityIndicator, StyleSheet } from 'react-native';
import { useRecoilValue, useRecoilState, useSetRecoilState } from 'recoil';
import {
  userState,
  userProfileQuery,
  loadingState,
  errorState,
} from '../store/atoms/userAtoms';

const UserProfileContent = ({ userId }) => {
  const user = useRecoilValue(userState);
  const profile = useRecoilValue(userProfileQuery(userId));
  const [loading, setLoading] = useRecoilState(loadingState);
  const setError = useSetRecoilState(errorState);

  const handleUpdateProfile = async (updateData) => {
    setLoading(true);
    try {
      await api.put(`/users/${userId}`, updateData);
      // Profile will automatically update due to query invalidation
      setError(null);
    } catch (error) {
      setError(error.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <View style={styles.container}>
      <Text style={styles.name}>{profile.name}</Text>
      <Text style={styles.email}>{profile.email}</Text>
      <Text style={styles.bio}>{profile.bio}</Text>

      {loading && <ActivityIndicator size="small" color="#007AFF" />}
    </View>
  );
};

const UserProfile = ({ userId }) => {
  return (
    <Suspense fallback={<ActivityIndicator size="large" color="#007AFF" />}>
      <ErrorBoundary>
        <UserProfileContent userId={userId} />
      </ErrorBoundary>
    </Suspense>
  );
};

// Error boundary for Recoil errors
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Recoil Error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <View style={styles.errorContainer}>
          <Text style={styles.errorText}>
            Something went wrong loading this content.
          </Text>
          <TouchableOpacity
            style={styles.retryButton}
            onPress={() => this.setState({ hasError: false, error: null })}
          >
            <Text style={styles.retryText}>Retry</Text>
          </TouchableOpacity>
        </View>
      );
    }

    return this.props.children;
  }
}

const styles = StyleSheet.create({
  container: {
    padding: 20,
  },
  name: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 8,
  },
  email: {
    fontSize: 16,
    color: '#666',
    marginBottom: 8,
  },
  bio: {
    fontSize: 14,
    color: '#333',
  },
  errorContainer: {
    padding: 20,
    alignItems: 'center',
  },
  errorText: {
    fontSize: 16,
    color: '#666',
    textAlign: 'center',
    marginBottom: 16,
  },
  retryButton: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 20,
    paddingVertical: 10,
    borderRadius: 6,
  },
  retryText: {
    color: '#fff',
    fontSize: 16,
  },
});

export default UserProfile;
```

---

## ⚛️ **Jotai Atomic State**

### **Jotai Atoms and Primitives**
```javascript
// store/atoms.js
import { atom, atomFamily, selector, selectorFamily } from 'jotai';
import { atomWithStorage, createJSONStorage } from 'jotai/utils';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Basic atoms
export const countAtom = atom(0);

export const userAtom = atomWithStorage('user', null, {
  ...createJSONStorage(() => AsyncStorage),
  delayInit: true,
});

export const postsAtom = atom([]);

// Derived atoms
export const doubledCountAtom = atom(
  (get) => get(countAtom) * 2
);

export const isAuthenticatedAtom = atom(
  (get) => !!get(userAtom)
);

export const userDisplayNameAtom = atom(
  (get) => {
    const user = get(userAtom);
    if (!user) return 'Guest';
    return user.displayName || user.name || 'User';
  }
);

// Writable derived atoms
export const incrementCountAtom = atom(
  null,
  (get, set) => {
    set(countAtom, get(countAtom) + 1);
  }
);

export const resetCountAtom = atom(
  null,
  (get, set) => {
    set(countAtom, 0);
  }
);

// Async atoms
export const asyncUserAtom = atom(async () => {
  try {
    const response = await api.get('/user/profile');
    return response.data;
  } catch (error) {
    return null;
  }
});

// Atom family for dynamic atoms
export const postAtomFamily = atomFamily(
  (postId) => atom(async () => {
    try {
      const response = await api.get(`/posts/${postId}`);
      return response.data;
    } catch (error) {
      return null;
    }
  }),
  (a, b) => a === b
);

// Selector family
export const userPostsSelector = selectorFamily(
  (userId) => async () => {
    try {
      const response = await api.get(`/users/${userId}/posts`);
      return response.data;
    } catch (error) {
      return [];
    }
  },
  (a, b) => a === b
);
```

### **Jotai Custom Hooks**
```javascript
// hooks/useAuth.js
import { useAtom, useSetAtom, useAtomValue } from 'jotai';
import { userAtom, loadingAtom, errorAtom } from '../store/atoms';
import api from '../services/api';

export const useAuth = () => {
  const [user, setUser] = useAtom(userAtom);
  const [loading, setLoading] = useAtom(loadingAtom);
  const [error, setError] = useAtom(errorAtom);

  const login = async (credentials) => {
    setLoading(true);
    setError(null);

    try {
      const response = await api.post('/auth/login', credentials);
      setUser(response.data.user);
      return { success: true };
    } catch (error) {
      setError(error.message);
      return { success: false, error: error.message };
    } finally {
      setLoading(false);
    }
  };

  const logout = () => {
    setUser(null);
    setError(null);
  };

  const updateProfile = async (profileData) => {
    if (!user) return { success: false, error: 'Not authenticated' };

    setLoading(true);
    setError(null);

    try {
      const response = await api.put(`/users/${user.id}`, profileData);
      setUser({ ...user, ...response.data });
      return { success: true };
    } catch (error) {
      setError(error.message);
      return { success: false, error: error.message };
    } finally {
      setLoading(false);
    }
  };

  return {
    user,
    loading,
    error,
    login,
    logout,
    updateProfile,
    isAuthenticated: !!user,
  };
};

// hooks/usePosts.js
import { useAtom, useAtomValue } from 'jotai';
import { postsAtom, loadingAtom, errorAtom } from '../store/atoms';
import { postAtomFamily } from '../store/atoms';
import api from '../services/api';

export const usePosts = () => {
  const [posts, setPosts] = useAtom(postsAtom);
  const loading = useAtomValue(loadingAtom);
  const [error, setError] = useAtom(errorAtom);

  const fetchPosts = async () => {
    setError(null);

    try {
      const response = await api.get('/posts');
      setPosts(response.data);
      return { success: true };
    } catch (error) {
      setError(error.message);
      return { success: false, error: error.message };
    }
  };

  const createPost = async (postData) => {
    setError(null);

    try {
      const response = await api.post('/posts', postData);
      setPosts([response.data, ...posts]);
      return { success: true, post: response.data };
    } catch (error) {
      setError(error.message);
      return { success: false, error: error.message };
    }
  };

  const updatePost = async (postId, updateData) => {
    setError(null);

    try {
      const response = await api.put(`/posts/${postId}`, updateData);
      setPosts(posts.map(post =>
        post.id === postId ? { ...post, ...response.data } : post
      ));
      return { success: true };
    } catch (error) {
      setError(error.message);
      return { success: false, error: error.message };
    }
  };

  const deletePost = async (postId) => {
    setError(null);

    try {
      await api.delete(`/posts/${postId}`);
      setPosts(posts.filter(post => post.id !== postId));
      return { success: true };
    } catch (error) {
      setError(error.message);
      return { success: false, error: error.message };
    }
  };

  return {
    posts,
    loading,
    error,
    fetchPosts,
    createPost,
    updatePost,
    deletePost,
  };
};

// hooks/usePost.js
import { useAtomValue } from 'jotai';
import { postAtomFamily } from '../store/atoms';

export const usePost = (postId) => {
  const post = useAtomValue(postAtomFamily(postId));

  return {
    post,
    loading: !post,
    error: null, // Jotai handles errors internally
  };
};
```

---

## 🔄 **XState State Machines**

### **Basic State Machine**
```javascript
// machines/authMachine.js
import { createMachine, assign } from 'xstate';

export const authMachine = createMachine({
  id: 'auth',
  initial: 'checking',

  context: {
    user: null,
    error: null,
  },

  states: {
    checking: {
      invoke: {
        src: 'checkAuthStatus',
        onDone: {
          target: 'authenticated',
          actions: assign({
            user: (_, event) => event.data.user,
          }),
        },
        onError: {
          target: 'unauthenticated',
        },
      },
    },

    unauthenticated: {
      on: {
        LOGIN: 'loggingIn',
      },
    },

    loggingIn: {
      invoke: {
        src: 'login',
        onDone: {
          target: 'authenticated',
          actions: assign({
            user: (_, event) => event.data.user,
            error: null,
          }),
        },
        onError: {
          target: 'loginError',
          actions: assign({
            error: (_, event) => event.data,
          }),
        },
      },
    },

    loginError: {
      on: {
        RETRY: 'loggingIn',
        CLEAR_ERROR: {
          actions: assign({
            error: null,
          }),
        },
      },
    },

    authenticated: {
      on: {
        LOGOUT: {
          target: 'unauthenticated',
          actions: assign({
            user: null,
          }),
        },
        REFRESH: 'refreshing',
      },
    },

    refreshing: {
      invoke: {
        src: 'refreshToken',
        onDone: {
          target: 'authenticated',
          actions: assign({
            user: (_, event) => event.data.user,
          }),
        },
        onError: 'unauthenticated',
      },
    },
  },
}, {
  services: {
    checkAuthStatus: async () => {
      // Check if user is logged in
      const token = await getStoredToken();
      if (!token) {
        throw new Error('No token found');
      }

      const user = await validateToken(token);
      return { user };
    },

    login: async (_, event) => {
      const { email, password } = event.credentials;
      const response = await api.post('/auth/login', { email, password });
      await storeToken(response.data.token);
      return { user: response.data.user };
    },

    refreshToken: async () => {
      const refreshToken = await getStoredRefreshToken();
      const response = await api.post('/auth/refresh', { refreshToken });
      await storeToken(response.data.token);
      return { user: response.data.user };
    },
  },
});
```

### **Complex State Machine with Guards**
```javascript
// machines/orderMachine.js
import { createMachine, assign } from 'xstate';

export const orderMachine = createMachine({
  id: 'order',
  initial: 'idle',

  context: {
    order: null,
    payment: null,
    error: null,
    retryCount: 0,
  },

  states: {
    idle: {
      on: {
        START_ORDER: 'creating',
      },
    },

    creating: {
      invoke: {
        src: 'createOrder',
        onDone: {
          target: 'pendingPayment',
          actions: assign({
            order: (_, event) => event.data,
            error: null,
          }),
        },
        onError: {
          target: 'creationFailed',
          actions: assign({
            error: (_, event) => event.data,
          }),
        },
      },
    },

    creationFailed: {
      on: {
        RETRY: {
          target: 'creating',
          cond: 'canRetry',
          actions: assign({
            retryCount: (context) => context.retryCount + 1,
          }),
        },
        CANCEL: 'cancelled',
      },
    },

    pendingPayment: {
      on: {
        PAYMENT_STARTED: 'processingPayment',
        CANCEL: 'cancelled',
      },
    },

    processingPayment: {
      invoke: {
        src: 'processPayment',
        onDone: {
          target: 'paymentCompleted',
          actions: assign({
            payment: (_, event) => event.data,
          }),
        },
        onError: {
          target: 'paymentFailed',
          actions: assign({
            error: (_, event) => event.data,
          }),
        },
      },
    },

    paymentFailed: {
      on: {
        RETRY_PAYMENT: {
          target: 'processingPayment',
          cond: 'canRetryPayment',
        },
        CANCEL: 'cancelled',
      },
    },

    paymentCompleted: {
      invoke: {
        src: 'confirmOrder',
        onDone: 'confirmed',
        onError: 'confirmationFailed',
      },
    },

    confirmationFailed: {
      on: {
        RETRY_CONFIRMATION: 'paymentCompleted',
        CANCEL: 'cancelled',
      },
    },

    confirmed: {
      type: 'final',
    },

    cancelled: {
      type: 'final',
      entry: 'cleanupOrder',
    },
  },
}, {
  guards: {
    canRetry: (context) => context.retryCount < 3,
    canRetryPayment: (context) => context.retryCount < 2,
  },

  actions: {
    cleanupOrder: assign({
      order: null,
      payment: null,
      error: null,
      retryCount: 0,
    }),
  },

  services: {
    createOrder: async (context, event) => {
      const { items, shippingAddress } = event.orderData;
      const response = await api.post('/orders', {
        items,
        shippingAddress,
      });
      return response.data;
    },

    processPayment: async (context, event) => {
      const { paymentMethod, amount } = event.paymentData;
      const response = await api.post('/payments', {
        orderId: context.order.id,
        paymentMethod,
        amount,
      });
      return response.data;
    },

    confirmOrder: async (context) => {
      await api.post(`/orders/${context.order.id}/confirm`);
      return true;
    },
  },
});
```

### **React Integration with XState**
```javascript
// hooks/useAuthMachine.js
import { useMachine } from '@xstate/react';
import { authMachine } from '../machines/authMachine';

export const useAuthMachine = () => {
  const [state, send] = useMachine(authMachine);

  const login = (credentials) => {
    send('LOGIN', { credentials });
  };

  const logout = () => {
    send('LOGOUT');
  };

  const retryLogin = () => {
    send('RETRY');
  };

  const clearError = () => {
    send('CLEAR_ERROR');
  };

  const refresh = () => {
    send('REFRESH');
  };

  return {
    // State
    user: state.context.user,
    error: state.context.error,
    isAuthenticated: state.matches('authenticated'),
    isLoading: state.matches('loggingIn') || state.matches('checking') || state.matches('refreshing'),
    isLoginError: state.matches('loginError'),

    // Actions
    login,
    logout,
    retryLogin,
    clearError,
    refresh,

    // Current state for debugging
    currentState: state.value,
  };
};

// hooks/useOrderMachine.js
import { useMachine } from '@xstate/react';
import { orderMachine } from '../machines/orderMachine';

export const useOrderMachine = () => {
  const [state, send] = useMachine(orderMachine);

  const startOrder = (orderData) => {
    send('START_ORDER', { orderData });
  };

  const startPayment = (paymentData) => {
    send('PAYMENT_STARTED', { paymentData });
  };

  const retry = () => {
    if (state.matches('creationFailed')) {
      send('RETRY');
    } else if (state.matches('paymentFailed')) {
      send('RETRY_PAYMENT');
    } else if (state.matches('confirmationFailed')) {
      send('RETRY_CONFIRMATION');
    }
  };

  const cancel = () => {
    send('CANCEL');
  };

  return {
    // State
    order: state.context.order,
    payment: state.context.payment,
    error: state.context.error,
    retryCount: state.context.retryCount,

    // Status
    isIdle: state.matches('idle'),
    isCreating: state.matches('creating'),
    isPendingPayment: state.matches('pendingPayment'),
    isProcessingPayment: state.matches('processingPayment'),
    isConfirmed: state.matches('confirmed'),
    isCancelled: state.matches('cancelled'),
    hasError: state.matches('creationFailed') ||
              state.matches('paymentFailed') ||
              state.matches('confirmationFailed'),

    // Actions
    startOrder,
    startPayment,
    retry,
    cancel,

    // Current state
    currentState: state.value,
  };
};
```

---

## 🏃 **MobX Reactive State**

### **MobX Store Setup**
```javascript
// store/MobXStore.js
import { makeObservable, observable, action, computed, reaction, autorun } from 'mobx';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { makePersistable } from 'mobx-persist-store';

class AuthStore {
  // Observable state
  user = null;
  token = null;
  isAuthenticated = false;
  loading = false;
  error = null;

  constructor() {
    makeObservable(this, {
      // State
      user: observable,
      token: observable,
      isAuthenticated: observable,
      loading: observable,
      error: observable,

      // Computed
      displayName: computed,
      userInitials: computed,
      hasValidToken: computed,

      // Actions
      login: action,
      logout: action,
      updateProfile: action,
      clearError: action,
      setLoading: action,
    });

    // Make store persistable
    makePersistable(this, {
      name: 'AuthStore',
      properties: ['user', 'token', 'isAuthenticated'],
      storage: AsyncStorage,
    });

    // Auto-run reactions
    this.setupReactions();
  }

  // Computed properties
  get displayName() {
    if (!this.user) return 'Guest';
    return this.user.displayName ||
           this.user.username ||
           `${this.user.firstName || ''} ${this.user.lastName || ''}`.trim() ||
           'User';
  }

  get userInitials() {
    if (!this.user) return 'GU';
    const firstName = this.user.firstName || '';
    const lastName = this.user.lastName || '';
    return `${firstName.charAt(0)}${lastName.charAt(0)}`.toUpperCase() || 'GU';
  }

  get hasValidToken() {
    return this.token && this.isTokenValid(this.token);
  }

  // Actions
  async login(credentials) {
    this.loading = true;
    this.error = null;

    try {
      const response = await api.post('/auth/login', credentials);

      this.user = response.data.user;
      this.token = response.data.token;
      this.isAuthenticated = true;

      return { success: true };
    } catch (error) {
      this.error = error.message;
      this.isAuthenticated = false;
      return { success: false, error: error.message };
    } finally {
      this.loading = false;
    }
}

export default new BiometricAuth();
```

---

## 📚 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Redux Advanced Patterns**: Toolkit, Sagas, Selectors, Reselect
- ✅ **Zustand**: Simple state management with persistence
- ✅ **Recoil**: Atomic state with React integration
- ✅ **Jotai**: Primitive and flexible state management
- ✅ **XState**: State machines for complex workflows
- ✅ **MobX**: Reactive state management
- ✅ **State Persistence**: AsyncStorage integration
- ✅ **Performance Optimization**: Memoization and lazy loading
- ✅ **Testing State**: Unit and integration testing

### **Best Practices**
1. **Choose the right tool**: Match state management to app complexity
2. **Keep state normalized**: Avoid data duplication
3. **Use selectors**: Computed values for derived state
4. **Handle async operations**: Proper loading and error states
5. **Persist selectively**: Only persist necessary state
6. **Test state logic**: Comprehensive test coverage
7. **Monitor performance**: Track state update performance
8. **Document state structure**: Clear state schema documentation
9. **Version state**: Handle state migrations
10. **Security first**: Protect sensitive state data

### **Next Steps**
- Learn advanced Redux patterns like RTK Query
- Explore state management in microservices
- Implement real-time state synchronization
- Create custom state management libraries
- Learn state management in different frameworks

---

## 🎯 **Assignment**

### **Task 1: Redux Store Implementation**
Create a complete Redux store with:
- User authentication slice with async thunks
- Posts management with optimistic updates
- UI state management
- Custom selectors with reselect
- Redux Saga for complex async logic
- State persistence with redux-persist

### **Task 2: Zustand State Management**
Build a Zustand store with:
- Authentication state with persistence
- Posts management with optimistic updates
- Real-time subscriptions
- State slices for modularity
- Error handling and loading states
- Integration with AsyncStorage

### **Task 3: State Machine Implementation**
Implement XState machines for:
- User authentication flow
- Order processing workflow
- Form state management
- Modal/dialog state
- Navigation state
- Error recovery flows

---

## 📚 **Additional Resources**
- [Redux Toolkit Documentation](https://redux-toolkit.js.org/)
- [Zustand Documentation](https://zustand-demo.pmnd.rs/)
- [Recoil Documentation](https://recoiljs.org/)
- [Jotai Documentation](https://jotai.org/)
- [XState Documentation](https://xstate.js.org/)
- [MobX Documentation](https://mobx.js.org/)
- [Redux Saga Documentation](https://redux-saga.js.org/)

---

*This lesson provides comprehensive coverage of advanced state management patterns in React Native, with practical examples and best practices for building scalable applications.*
 