# Lesson 32: Advanced Redux Patterns & Middleware

## 🎯 **Learning Objectives**
- Master advanced Redux patterns and best practices
- Implement custom middleware for complex scenarios
- Use Redux Toolkit for simplified Redux development
- Create reusable Redux logic with ducks pattern
- Implement Redux Saga for complex async flows

## 📚 **Table of Contents**
1. [Redux Toolkit Deep Dive](#redux-toolkit-deep-dive)
2. [Custom Middleware](#custom-middleware)
3. [Redux Saga](#redux-saga)
4. [Ducks Pattern](#ducks-pattern)
5. [Advanced Selectors](#advanced-selectors)
6. [Error Handling Patterns](#error-handling-patterns)
7. [Performance Optimization](#performance-optimization)
8. [Testing Redux Code](#testing-redux-code)
9. [Real-world Patterns](#real-world-patterns)
10. [Practical Examples](#practical-examples)

---

## 🛠️ **Redux Toolkit Deep Dive**

### **RTK Query for API Calls**
```javascript
// store/apiSlice.js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const apiSlice = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({
    baseUrl: 'https://jsonplaceholder.typicode.com',
    prepareHeaders: (headers, { getState }) => {
      const token = getState().auth.token;
      if (token) {
        headers.set('authorization', `Bearer ${token}`);
      }
      return headers;
    },
  }),
  endpoints: (builder) => ({
    getPosts: builder.query({
      query: () => '/posts',
      providesTags: ['Posts'],
    }),
    getPost: builder.query({
      query: (id) => `/posts/${id}`,
      providesTags: (result, error, id) => [{ type: 'Posts', id }],
    }),
    addPost: builder.mutation({
      query: (post) => ({
        url: '/posts',
        method: 'POST',
        body: post,
      }),
      invalidatesTags: ['Posts'],
    }),
    updatePost: builder.mutation({
      query: ({ id, ...patch }) => ({
        url: `/posts/${id}`,
        method: 'PATCH',
        body: patch,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'Posts', id }],
    }),
    deletePost: builder.mutation({
      query: (id) => ({
        url: `/posts/${id}`,
        method: 'DELETE',
      }),
      invalidatesTags: (result, error, id) => [{ type: 'Posts', id }],
    }),
  }),
});

export const {
  useGetPostsQuery,
  useGetPostQuery,
  useAddPostMutation,
  useUpdatePostMutation,
  useDeletePostMutation,
} = apiSlice;
```

### **Advanced Slice Configuration**
```javascript
// store/slices/authSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import AsyncStorage from '@react-native-async-storage/async-storage';

// Async thunks
export const loginUser = createAsyncThunk(
  'auth/login',
  async (credentials, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials),
      });

      if (!response.ok) {
        const error = await response.json();
        return rejectWithValue(error.message);
      }

      const data = await response.json();
      await AsyncStorage.setItem('token', data.token);
      return data;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

export const logoutUser = createAsyncThunk(
  'auth/logout',
  async (_, { getState }) => {
    const { auth } = getState();
    if (auth.token) {
      await fetch('/api/auth/logout', {
        method: 'POST',
        headers: { Authorization: `Bearer ${auth.token}` },
      });
    }
    await AsyncStorage.removeItem('token');
  }
);

// Slice with advanced features
const authSlice = createSlice({
  name: 'auth',
  initialState: {
    user: null,
    token: null,
    isLoading: false,
    error: null,
    lastLogin: null,
  },
  reducers: {
    clearError: (state) => {
      state.error = null;
    },
    updateLastActivity: (state) => {
      state.lastLogin = new Date().toISOString();
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(loginUser.pending, (state) => {
        state.isLoading = true;
        state.error = null;
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.isLoading = false;
        state.user = action.payload.user;
        state.token = action.payload.token;
        state.lastLogin = new Date().toISOString();
        state.error = null;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.isLoading = false;
        state.user = null;
        state.token = null;
        state.error = action.payload;
      })
      .addCase(logoutUser.fulfilled, (state) => {
        state.user = null;
        state.token = null;
        state.error = null;
      });
  },
});

export const { clearError, updateLastActivity } = authSlice.actions;
export default authSlice.reducer;
```

---

## 🔧 **Custom Middleware**

### **Logger Middleware**
```javascript
// middleware/logger.js
const logger = store => next => action => {
  console.group(action.type);
  console.info('Dispatching:', action);

  const result = next(action);

  console.log('Next state:', store.getState());
  console.groupEnd();

  return result;
};

export default logger;
```

### **API Middleware**
```javascript
// middleware/api.js
const api = ({ dispatch }) => next => action => {
  if (action.type !== 'API') {
    return next(action);
  }

  const { url, method = 'GET', data, onSuccess, onError } = action.payload;

  fetch(url, {
    method,
    headers: {
      'Content-Type': 'application/json',
    },
    body: data ? JSON.stringify(data) : undefined,
  })
    .then(response => response.json())
    .then(data => {
      if (onSuccess) {
        dispatch({ type: onSuccess, payload: data });
      }
    })
    .catch(error => {
      if (onError) {
        dispatch({ type: onError, payload: error.message });
      }
    });

  return next(action);
};

export default api;
```

### **Crash Reporter Middleware**
```javascript
// middleware/crashReporter.js
const crashReporter = store => next => action => {
  try {
    return next(action);
  } catch (error) {
    console.error('Caught an exception!', error);

    // Send error to crash reporting service
    // Example: Sentry, Bugsnag, etc.
    // crashReportingService.captureException(error, {
    //   extra: {
    //     action: action.type,
    //     state: store.getState(),
    //   },
    // });

    throw error;
  }
};

export default crashReporter;
```

### **Performance Monitoring Middleware**
```javascript
// middleware/performance.js
const performance = store => next => action => {
  const start = Date.now();
  const result = next(action);
  const end = Date.now();

  const duration = end - start;

  // Log slow actions
  if (duration > 100) {
    console.warn(`Slow action: ${action.type} took ${duration}ms`);
  }

  // Track action performance
  // Example: Send to analytics service
  // analytics.track('action_performance', {
  //   action: action.type,
  //   duration,
  //   timestamp: new Date().toISOString(),
  // });

  return result;
};

export default performance;
```

---

## 🌊 **Redux Saga**

### **Saga Setup**
```javascript
// store/sagas/index.js
import { all } from 'redux-saga/effects';
import authSagas from './authSagas';
import userSagas from './userSagas';
import postSagas from './postSagas';

export default function* rootSaga() {
  yield all([
    authSagas(),
    userSagas(),
    postSagas(),
  ]);
}

// store/index.js
import createSagaMiddleware from 'redux-saga';
import rootSaga from './sagas';

const sagaMiddleware = createSagaMiddleware();

const store = configureStore({
  reducer: rootReducer,
  middleware: [sagaMiddleware],
});

sagaMiddleware.run(rootSaga);
```

### **Authentication Sagas**
```javascript
// sagas/authSagas.js
import { takeLatest, call, put, select } from 'redux-saga/effects';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { loginSuccess, loginFailure, logout } from '../slices/authSlice';

function* loginSaga(action) {
  try {
    const { email, password } = action.payload;

    // Call API
    const response = yield call(loginAPI, { email, password });

    // Store token
    yield call(AsyncStorage.setItem, 'token', response.token);

    // Dispatch success
    yield put(loginSuccess(response));
  } catch (error) {
    yield put(loginFailure(error.message));
  }
}

function* logoutSaga() {
  try {
    const token = yield select(state => state.auth.token);

    if (token) {
      yield call(logoutAPI, token);
    }

    yield call(AsyncStorage.removeItem, 'token');
    yield put(logout());
  } catch (error) {
    console.error('Logout error:', error);
  }
}

function* watchAuth() {
  yield takeLatest('auth/login', loginSaga);
  yield takeLatest('auth/logout', logoutSaga);
}

export default watchAuth;
```

### **Complex Saga Patterns**
```javascript
// sagas/postSagas.js
import { takeEvery, takeLatest, call, put, select, delay, race, take } from 'redux-saga/effects';
import { fetchPostsSuccess, fetchPostsFailure, addPostSuccess, addPostFailure } from '../slices/postsSlice';

function* fetchPostsSaga() {
  try {
    // Show loading
    yield put({ type: 'posts/setLoading', payload: true });

    // Add delay for demo
    yield delay(1000);

    // Call API with timeout
    const { response, timeout } = yield race({
      response: call(fetchPostsAPI),
      timeout: delay(5000), // 5 second timeout
    });

    if (timeout) {
      throw new Error('Request timeout');
    }

    yield put(fetchPostsSuccess(response));
  } catch (error) {
    yield put(fetchPostsFailure(error.message));
  } finally {
    yield put({ type: 'posts/setLoading', payload: false });
  }
}

function* addPostSaga(action) {
  try {
    const { post } = action.payload;
    const token = yield select(state => state.auth.token);

    // Validate post
    if (!post.title || !post.content) {
      throw new Error('Title and content are required');
    }

    // Optimistic update
    const tempId = Date.now();
    yield put({
      type: 'posts/addOptimistic',
      payload: { ...post, id: tempId, isPending: true }
    });

    // API call
    const response = yield call(addPostAPI, post, token);

    // Replace optimistic update with real data
    yield put(addPostSuccess({ ...response, tempId }));

  } catch (error) {
    // Remove optimistic update on error
    yield put({ type: 'posts/removeOptimistic', payload: tempId });
    yield put(addPostFailure(error.message));
  }
}

function* watchPosts() {
  yield takeLatest('posts/fetchPosts', fetchPostsSaga);
  yield takeEvery('posts/addPost', addPostSaga);
}

export default watchPosts;
```

---

## 🦆 **Ducks Pattern**

### **Ducks Structure**
```
src/
├── store/
│   ├── ducks/
│   │   ├── index.js
│   │   ├── auth.js
│   │   ├── posts.js
│   │   └── users.js
│   └── index.js
```

### **Auth Duck**
```javascript
// store/ducks/auth.js
// Actions
const LOGIN_REQUEST = 'myapp/auth/LOGIN_REQUEST';
const LOGIN_SUCCESS = 'myapp/auth/LOGIN_SUCCESS';
const LOGIN_FAILURE = 'myapp/auth/LOGIN_FAILURE';
const LOGOUT = 'myapp/auth/LOGOUT';

// Action Creators
export const loginRequest = (credentials) => ({
  type: LOGIN_REQUEST,
  payload: credentials,
});

export const loginSuccess = (user) => ({
  type: LOGIN_SUCCESS,
  payload: user,
});

export const loginFailure = (error) => ({
  type: LOGIN_FAILURE,
  payload: error,
});

export const logout = () => ({
  type: LOGOUT,
});

// Thunks
export const login = (credentials) => async (dispatch) => {
  try {
    dispatch(loginRequest(credentials));

    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials),
    });

    if (!response.ok) {
      throw new Error('Login failed');
    }

    const user = await response.json();
    dispatch(loginSuccess(user));
  } catch (error) {
    dispatch(loginFailure(error.message));
  }
};

// Initial State
const initialState = {
  user: null,
  isLoading: false,
  error: null,
};

// Reducer
const authReducer = (state = initialState, action) => {
  switch (action.type) {
    case LOGIN_REQUEST:
      return {
        ...state,
        isLoading: true,
        error: null,
      };

    case LOGIN_SUCCESS:
      return {
        ...state,
        isLoading: false,
        user: action.payload,
        error: null,
      };

    case LOGIN_FAILURE:
      return {
        ...state,
        isLoading: false,
        user: null,
        error: action.payload,
      };

    case LOGOUT:
      return {
        ...state,
        user: null,
        error: null,
      };

    default:
      return state;
  }
};

export default authReducer;
```

### **Posts Duck**
```javascript
// store/ducks/posts.js
// Actions
const FETCH_POSTS_REQUEST = 'myapp/posts/FETCH_POSTS_REQUEST';
const FETCH_POSTS_SUCCESS = 'myapp/posts/FETCH_POSTS_SUCCESS';
const FETCH_POSTS_FAILURE = 'myapp/posts/FETCH_POSTS_FAILURE';
const ADD_POST = 'myapp/posts/ADD_POST';
const TOGGLE_POST = 'myapp/posts/TOGGLE_POST';

// Action Creators
export const fetchPostsRequest = () => ({
  type: FETCH_POSTS_REQUEST,
});

export const fetchPostsSuccess = (posts) => ({
  type: FETCH_POSTS_SUCCESS,
  payload: posts,
});

export const fetchPostsFailure = (error) => ({
  type: FETCH_POSTS_FAILURE,
  payload: error,
});

export const addPost = (post) => ({
  type: ADD_POST,
  payload: post,
});

export const togglePost = (id) => ({
  type: TOGGLE_POST,
  payload: id,
});

// Thunks
export const fetchPosts = () => async (dispatch) => {
  try {
    dispatch(fetchPostsRequest());

    const response = await fetch('/api/posts');
    if (!response.ok) {
      throw new Error('Failed to fetch posts');
    }

    const posts = await response.json();
    dispatch(fetchPostsSuccess(posts));
  } catch (error) {
    dispatch(fetchPostsFailure(error.message));
  }
};

// Initial State
const initialState = {
  items: [],
  isLoading: false,
  error: null,
};

// Reducer
const postsReducer = (state = initialState, action) => {
  switch (action.type) {
    case FETCH_POSTS_REQUEST:
      return {
        ...state,
        isLoading: true,
        error: null,
      };

    case FETCH_POSTS_SUCCESS:
      return {
        ...state,
        isLoading: false,
        items: action.payload,
        error: null,
      };

    case FETCH_POSTS_FAILURE:
      return {
        ...state,
        isLoading: false,
        error: action.payload,
      };

    case ADD_POST:
      return {
        ...state,
        items: [...state.items, {
          ...action.payload,
          id: Date.now(),
          completed: false,
        }],
      };

    case TOGGLE_POST:
      return {
        ...state,
        items: state.items.map(post =>
          post.id === action.payload
            ? { ...post, completed: !post.completed }
            : post
        ),
      };

    default:
      return state;
  }
};

export default postsReducer;
```

---

## 🎯 **Advanced Selectors**

### **Reselect Library**
```javascript
// selectors/postSelectors.js
import { createSelector } from 'reselect';

// Input selectors
const getPosts = state => state.posts.items;
const getFilter = state => state.posts.filter;
const getSearchTerm = state => state.posts.searchTerm;

// Memoized selectors
export const getFilteredPosts = createSelector(
  [getPosts, getFilter, getSearchTerm],
  (posts, filter, searchTerm) => {
    let filteredPosts = posts;

    // Apply search filter
    if (searchTerm) {
      filteredPosts = filteredPosts.filter(post =>
        post.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
        post.content.toLowerCase().includes(searchTerm.toLowerCase())
      );
    }

    // Apply status filter
    switch (filter) {
      case 'completed':
        return filteredPosts.filter(post => post.completed);
      case 'active':
        return filteredPosts.filter(post => !post.completed);
      default:
        return filteredPosts;
    }
  }
);

export const getPostsStats = createSelector(
  [getPosts],
  (posts) => {
    const total = posts.length;
    const completed = posts.filter(post => post.completed).length;
    const active = total - completed;
    const completionRate = total > 0 ? (completed / total) * 100 : 0;

    return {
      total,
      completed,
      active,
      completionRate: Math.round(completionRate),
    };
  }
);

export const getPostsByCategory = createSelector(
  [getPosts],
  (posts) => {
    return posts.reduce((categories, post) => {
      const category = post.category || 'uncategorized';
      if (!categories[category]) {
        categories[category] = [];
      }
      categories[category].push(post);
      return categories;
    }, {});
  }
);
```

### **Custom Selector Hooks**
```javascript
// hooks/usePosts.js
import { useMemo } from 'react';
import { useSelector } from 'react-redux';
import { getFilteredPosts, getPostsStats } from '../selectors/postSelectors';

export const usePosts = (filter = 'all', searchTerm = '') => {
  const posts = useSelector(state => getFilteredPosts(state, filter, searchTerm));
  const stats = useSelector(getPostsStats);

  return {
    posts,
    stats,
    isLoading: useSelector(state => state.posts.isLoading),
    error: useSelector(state => state.posts.error),
  };
};

export const usePost = (id) => {
  return useSelector(state => {
    const post = state.posts.items.find(p => p.id === id);
    return {
      post,
      isLoading: state.posts.isLoading,
      error: state.posts.error,
    };
  });
};
```

---

## 🚨 **Error Handling Patterns**

### **Global Error Handler**
```javascript
// middleware/errorHandler.js
const errorHandler = store => next => action => {
  try {
    // Handle async actions
    if (action.type.endsWith('/pending')) {
      // Track pending actions for timeout
      const timeoutId = setTimeout(() => {
        store.dispatch({
          type: action.type.replace('/pending', '/timeout'),
          meta: { requestId: action.meta?.requestId },
        });
      }, 30000); // 30 second timeout

      // Store timeout ID for cleanup
      action.meta = {
        ...action.meta,
        timeoutId,
      };
    }

    if (action.type.endsWith('/fulfilled') || action.type.endsWith('/rejected')) {
      // Clear timeout
      if (action.meta?.timeoutId) {
        clearTimeout(action.meta.timeoutId);
      }
    }

    return next(action);
  } catch (error) {
    console.error('Action error:', error);

    // Dispatch error action
    store.dispatch({
      type: 'app/error',
      payload: {
        message: error.message,
        stack: error.stack,
        action: action.type,
      },
    });

    // Re-throw for Redux to handle
    throw error;
  }
};

export default errorHandler;
```

### **Retry Mechanism**
```javascript
// utils/retry.js
export const retry = async (fn, maxAttempts = 3, delay = 1000) => {
  let lastError;

  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;

      if (attempt === maxAttempts) {
        throw error;
      }

      // Exponential backoff
      const waitTime = delay * Math.pow(2, attempt - 1);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
  }

  throw lastError;
};

// middleware/retry.js
const retryMiddleware = store => next => action => {
  if (action.type.endsWith('/rejected') && action.meta?.retry) {
    const { originalAction, retryCount = 0 } = action.meta;

    if (retryCount < 3) {
      // Retry after delay
      setTimeout(() => {
        store.dispatch({
          ...originalAction,
          meta: {
            ...originalAction.meta,
            retryCount: retryCount + 1,
          },
        });
      }, Math.pow(2, retryCount) * 1000);
    }
  }

  return next(action);
};

export default retryMiddleware;
```

---

## ⚡ **Performance Optimization**

### **Memoized Selectors**
```javascript
// selectors/optimizedSelectors.js
import { createSelector } from 'reselect';

// Deep comparison selector
export const getVisibleTodos = createSelector(
  [
    state => state.todos.items,
    state => state.todos.filter,
    state => state.filters.searchTerm,
  ],
  (todos, filter, searchTerm) => {
    console.log('Computing visible todos...');

    return todos.filter(todo => {
      const matchesFilter = filter === 'all' ||
        (filter === 'completed' && todo.completed) ||
        (filter === 'active' && !todo.completed);

      const matchesSearch = !searchTerm ||
        todo.text.toLowerCase().includes(searchTerm.toLowerCase());

      return matchesFilter && matchesSearch;
    });
  },
  {
    memoizeOptions: {
      equalityCheck: (a, b) => {
        // Custom equality check for arrays
        if (Array.isArray(a) && Array.isArray(b)) {
          return a.length === b.length &&
            a.every((item, index) => item.id === b[index]?.id);
        }
        return a === b;
      },
    },
  }
);
```

### **Action Batching**
```javascript
// middleware/batch.js
const batch = store => next => action => {
  if (action.type === 'BATCH_ACTIONS') {
    // Execute all actions in batch
    action.payload.forEach(singleAction => {
      next(singleAction);
    });
  } else {
    return next(action);
  }
};

// Usage
const batchActions = (actions) => ({
  type: 'BATCH_ACTIONS',
  payload: actions,
});

// In component
const handleBulkUpdate = () => {
  dispatch(batchActions([
    updateTodo({ id: 1, text: 'Updated 1' }),
    updateTodo({ id: 2, text: 'Updated 2' }),
    updateTodo({ id: 3, text: 'Updated 3' }),
  ]));
};
```

---

## 🧪 **Testing Redux Code**

### **Testing Reducers**
```javascript
// __tests__/reducers/authReducer.test.js
import authReducer, { loginSuccess, loginFailure, logout } from '../../store/slices/authSlice';

describe('Auth Reducer', () => {
  const initialState = {
    user: null,
    token: null,
    isLoading: false,
    error: null,
  };

  test('should return initial state', () => {
    expect(authReducer(undefined, { type: undefined })).toEqual(initialState);
  });

  test('should handle login success', () => {
    const user = { id: 1, email: 'test@example.com' };
    const action = loginSuccess(user);

    const expectedState = {
      ...initialState,
      user,
      token: 'mock-token',
    };

    expect(authReducer(initialState, action)).toEqual(expectedState);
  });

  test('should handle login failure', () => {
    const error = 'Invalid credentials';
    const action = loginFailure(error);

    const expectedState = {
      ...initialState,
      error,
    };

    expect(authReducer(initialState, action)).toEqual(expectedState);
  });

  test('should handle logout', () => {
    const loggedInState = {
      ...initialState,
      user: { id: 1, email: 'test@example.com' },
      token: 'mock-token',
    };

    const action = logout();
    const result = authReducer(loggedInState, action);

    expect(result.user).toBeNull();
    expect(result.token).toBeNull();
    expect(result.error).toBeNull();
  });
});
```

### **Testing Async Thunks**
```javascript
// __tests__/async/authThunks.test.js
import configureMockStore from 'redux-mock-store';
import thunk from 'redux-thunk';
import fetchMock from 'fetch-mock';
import { loginUser } from '../../store/slices/authSlice';

const middlewares = [thunk];
const mockStore = configureMockStore(middlewares);

describe('Auth Thunks', () => {
  afterEach(() => {
    fetchMock.restore();
  });

  test('should dispatch success action on successful login', async () => {
    const mockResponse = {
      user: { id: 1, email: 'test@example.com' },
      token: 'mock-jwt-token',
    };

    fetchMock.post('/api/auth/login', {
      body: mockResponse,
      headers: { 'content-type': 'application/json' },
    });

    const expectedActions = [
      { type: 'auth/login/pending' },
      {
        type: 'auth/login/fulfilled',
        payload: mockResponse,
      },
    ];

    const store = mockStore({});
    await store.dispatch(loginUser({
      email: 'test@example.com',
      password: 'password123',
    }));

    expect(store.getActions()).toEqual(expectedActions);
  });

  test('should dispatch failure action on login error', async () => {
    fetchMock.post('/api/auth/login', {
      status: 401,
      body: { message: 'Invalid credentials' },
    });

    const expectedActions = [
      { type: 'auth/login/pending' },
      {
        type: 'auth/login/rejected',
        error: { message: 'Rejected' },
        payload: 'Invalid credentials',
      },
    ];

    const store = mockStore({});
    await store.dispatch(loginUser({
      email: 'test@example.com',
      password: 'wrongpassword',
    }));

    expect(store.getActions()).toEqual(expectedActions);
  });
});
```

---

## 🎯 **Real-world Patterns**

### **Entity Normalization**
```javascript
// store/slices/entitiesSlice.js
import { createSlice, createEntityAdapter } from '@reduxjs/toolkit';

const postsAdapter = createEntityAdapter({
  selectId: (post) => post.id,
  sortComparer: (a, b) => b.createdAt.localeCompare(a.createdAt),
});

const commentsAdapter = createEntityAdapter({
  selectId: (comment) => comment.id,
  sortComparer: (a, b) => a.createdAt.localeCompare(b.createdAt),
});

const entitiesSlice = createSlice({
  name: 'entities',
  initialState: {
    posts: postsAdapter.getInitialState(),
    comments: commentsAdapter.getInitialState(),
    users: {},
  },
  reducers: {
    // Post actions
    postsReceived: postsAdapter.upsertMany,
    postAdded: postsAdapter.addOne,
    postUpdated: postsAdapter.updateOne,
    postRemoved: postsAdapter.removeOne,

    // Comment actions
    commentsReceived: commentsAdapter.upsertMany,
    commentAdded: commentsAdapter.addOne,
    commentUpdated: commentsAdapter.updateOne,
    commentRemoved: commentsAdapter.removeOne,

    // User actions
    userReceived: (state, action) => {
      state.users[action.payload.id] = action.payload;
    },
  },
});

export const {
  postsReceived,
  postAdded,
  postUpdated,
  postRemoved,
  commentsReceived,
  commentAdded,
  commentUpdated,
  commentRemoved,
  userReceived,
} = entitiesSlice.actions;

// Selectors
export const {
  selectAll: selectAllPosts,
  selectById: selectPostById,
  selectIds: selectPostIds,
} = postsAdapter.getSelectors(state => state.entities.posts);

export const {
  selectAll: selectAllComments,
  selectById: selectCommentById,
} = commentsAdapter.getSelectors(state => state.entities.comments);

export default entitiesSlice.reducer;
```

### **Optimistic Updates with Rollback**
```javascript
// middleware/optimisticUpdates.js
const optimisticUpdates = store => next => action => {
  // Check if action has optimistic update
  if (action.meta?.optimistic) {
    const { type, payload, meta } = action;
    const optimisticId = `optimistic_${Date.now()}`;

    // Apply optimistic update
    const optimisticAction = {
      type,
      payload: { ...payload, id: optimisticId, isOptimistic: true },
    };

    next(optimisticAction);

    // Track for potential rollback
    store.dispatch({
      type: 'optimistic/add',
      payload: {
        id: optimisticId,
        action: optimisticAction,
        rollbackType: meta.rollbackType,
      },
    });

    return optimisticId;
  }

  // Handle rollback
  if (action.type === 'ROLLBACK_OPTIMISTIC') {
    const { optimisticId, error } = action.payload;

    store.dispatch({
      type: 'optimistic/remove',
      payload: optimisticId,
    });

    // Dispatch error action
    if (error) {
      store.dispatch({
        type: 'app/error',
        payload: error,
      });
    }
  }

  return next(action);
};

// Usage
const addPostOptimistic = (post) => ({
  type: 'posts/add',
  payload: post,
  meta: {
    optimistic: true,
    rollbackType: 'posts/remove',
  },
});
```

---

## 🎯 **Practical Examples**

### **Advanced Todo App with Redux Toolkit**
```javascript
// store/slices/todosSlice.js
import { createSlice, createAsyncThunk, createEntityAdapter } from '@reduxjs/toolkit';

const todosAdapter = createEntityAdapter({
  selectId: (todo) => todo.id,
  sortComparer: (a, b) => b.createdAt - a.createdAt,
});

// Async thunks
export const fetchTodos = createAsyncThunk(
  'todos/fetchTodos',
  async (_, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/todos');
      if (!response.ok) throw new Error('Failed to fetch todos');
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

export const addTodo = createAsyncThunk(
  'todos/addTodo',
  async (todoData, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(todoData),
      });
      if (!response.ok) throw new Error('Failed to add todo');
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const todosSlice = createSlice({
  name: 'todos',
  initialState: todosAdapter.getInitialState({
    loading: false,
    error: null,
    filter: 'all',
    searchTerm: '',
  }),
  reducers: {
    toggleTodo: (state, action) => {
      const todo = state.entities[action.payload];
      if (todo) {
        todo.completed = !todo.completed;
        todo.updatedAt = Date.now();
      }
    },
    deleteTodo: todosAdapter.removeOne,
    setFilter: (state, action) => {
      state.filter = action.payload;
    },
    setSearchTerm: (state, action) => {
      state.searchTerm = action.payload;
    },
    clearCompleted: (state) => {
      const completedIds = Object.values(state.entities)
        .filter(todo => todo.completed)
        .map(todo => todo.id);
      todosAdapter.removeMany(state, completedIds);
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.loading = false;
        todosAdapter.setAll(state, action.payload);
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      })
      .addCase(addTodo.fulfilled, (state, action) => {
        todosAdapter.addOne(state, action.payload);
      });
  },
});

export const {
  toggleTodo,
  deleteTodo,
  setFilter,
  setSearchTerm,
  clearCompleted,
} = todosSlice.actions;

// Selectors
export const {
  selectAll: selectAllTodos,
  selectById: selectTodoById,
  selectIds: selectTodoIds,
} = todosAdapter.getSelectors(state => state.todos);

export const selectFilteredTodos = createSelector(
  [selectAllTodos, state => state.todos.filter, state => state.todos.searchTerm],
  (todos, filter, searchTerm) => {
    return todos.filter(todo => {
      const matchesFilter = filter === 'all' ||
        (filter === 'completed' && todo.completed) ||
        (filter === 'active' && !todo.completed);

      const matchesSearch = !searchTerm ||
        todo.title.toLowerCase().includes(searchTerm.toLowerCase());

      return matchesFilter && matchesSearch;
    });
  }
);

export const selectTodosStats = createSelector(
  [selectAllTodos],
  (todos) => ({
    total: todos.length,
    completed: todos.filter(todo => todo.completed).length,
    active: todos.filter(todo => !todo.completed).length,
  })
);

export default todosSlice.reducer;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Redux Toolkit**: Simplified Redux with RTK Query
- ✅ **Custom Middleware**: Logger, API, crash reporter, performance monitoring
- ✅ **Redux Saga**: Complex async flows with generators
- ✅ **Ducks Pattern**: Modular Redux organization
- ✅ **Advanced Selectors**: Reselect for memoized computations
- ✅ **Error Handling**: Global error handlers and retry mechanisms
- ✅ **Performance**: Memoization, batching, optimistic updates
- ✅ **Testing**: Unit tests for reducers, integration tests for thunks
- ✅ **Real-world Patterns**: Entity normalization, optimistic updates

### **Best Practices**
1. **Use Redux Toolkit** for new projects - simplifies Redux setup
2. **Implement proper error handling** with retry mechanisms
3. **Use selectors** for computed values and performance
4. **Normalize state shape** to avoid deep nesting
5. **Test reducers thoroughly** as they contain business logic
6. **Handle async operations** with proper loading/error states
7. **Use middleware strategically** for cross-cutting concerns
8. **Monitor performance** and optimize slow operations

### **Next Steps**
- Learn about Redux Persist for state persistence
- Explore Redux Observable for reactive programming
- Implement real-time features with Redux
- Create custom hooks for Redux integration

---

## 🎯 **Assignment**

### **Task 1: Advanced Todo App**
Create a comprehensive todo app with:
- Redux Toolkit for state management
- RTK Query for API calls
- Optimistic updates with rollback
- Advanced filtering and search
- Real-time synchronization
- Offline support with queue management

### **Task 2: E-commerce Cart**
Build a shopping cart with Redux featuring:
- Product catalog management
- Cart operations (add/remove/update)
- Inventory tracking
- Order processing with validation
- Payment integration simulation
- Order history and reordering

### **Task 3: Social Media Feed**
Implement a social media feed with:
- Post creation and editing
- Like/comment system
- Real-time updates with Redux Saga
- Infinite scrolling with pagination
- Content moderation
- User interactions tracking

---

## 📚 **Additional Resources**
- [Redux Toolkit Documentation](https://redux-toolkit.js.org/)
- [Redux Saga Documentation](https://redux-saga.js.org/)
- [Reselect Documentation](https://github.com/reduxjs/reselect)
- [Redux DevTools](https://github.com/reduxjs/redux-devtools)
- [Advanced Redux Patterns](https://redux.js.org/advanced/)

---

**Next Lesson**: [Lesson 33: React Navigation Advanced Patterns](Lesson%2033_%20React%20Navigation%20Advanced%20Patterns.md)