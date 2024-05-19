Sure! Let's go through each of the advanced functionalities in React-Redux in detail.

### 1. **Normalization of State**

Normalization of state ensures that your Redux store does not have redundant or nested data, making it easier to manage and update. This is particularly useful when dealing with relational data (like a list of articles, each with an author).

- **Normalization**: Breaking down nested entities into flat structures.
- **Denormalization**: Transforming the flat structure back into nested structures as needed.

**Example**:
Using `normalizr`, you can define schemas for your entities and normalize the data.

```js
import { schema, normalize } from 'normalizr';

// Define a users schema
const user = new schema.Entity('users');

// Define your articles schema
const article = new schema.Entity('articles', {
  author: user,
});

// Define your API response
const originalData = {
  id: '123',
  author: {
    id: '1',
    name: 'Paul',
  },
};

// Normalize the data
const normalizedData = normalize(originalData, article);

console.log(normalizedData);
```

### 2. **Selectors for Derived State**

Selectors are functions that extract and compute derived state from your Redux store. They help in avoiding redundant computations and can improve performance when used with memoization.

- **Basic Selector**: Extracts a piece of state.
- **Memoized Selector**: Only recomputes when its input changes.

**Example**:
Using `reselect` to create memoized selectors.

```js
import { createSelector } from 'reselect';

const selectUsers = (state) => state.users;
const selectUserById = (state, userId) => state.users[userId];

const makeSelectUserById = () => createSelector(
  [selectUsers, selectUserById],
  (users, user) => user
);

// Usage in a component
const userId = 1;
const user = useSelector((state) => makeSelectUserById()(state, userId));
```

### 3. **Code Splitting and Lazy Loading**

Code splitting allows you to load parts of your application on demand, reducing the initial load time. This improves performance and user experience.

- **React.lazy**: Used to dynamically import components.
- **Suspense**: Provides a fallback UI while the component is being loaded.

**Example**:
Using `React.lazy` and `Suspense` for lazy loading.

```jsx
import React, { Suspense, lazy } from 'react';

const SomeComponent = lazy(() => import('./SomeComponent'));

const App = () => (
  <Suspense fallback={<div>Loading...</div>}>
    <SomeComponent />
  </Suspense>
);
```

### 4. **Optimistic Updates**

Optimistic updates immediately reflect changes in the UI while the actual operation is performed in the background. This provides a responsive user experience.

- **Optimistic UI Update**: Update the UI before the server confirms the operation.
- **Rollback**: Revert the UI update if the operation fails.

**Example**:
Using optimistic updates in a Redux action.

```js
const updateUser = createAsyncThunk('users/updateUser', async (user, { dispatch, getState }) => {
  const originalUser = getState().users[user.id];
  dispatch(userUpdated(user)); // Optimistically update the UI
  try {
    const response = await api.updateUser(user);
    return response.data;
  } catch (error) {
    dispatch(userUpdated(originalUser)); // Rollback on error
    throw error;
  }
});
```

### 5. **Middleware for Advanced Logic**

Middleware allows you to inject logic between dispatching an action and the moment it reaches the reducer. This can be used for logging, crash reporting, handling asynchronous tasks, etc.

- **Custom Middleware**: A function that wraps the dispatch method.
- **Thunk Middleware**: Handles async operations by dispatching functions.

**Example**:
Creating a logger middleware.

```js
const loggerMiddleware = (store) => (next) => (action) => {
  console.log('Dispatching:', action);
  let result = next(action);
  console.log('Next state:', store.getState());
  return result;
};

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(loggerMiddleware),
});
```

### 6. **Using Redux Toolkit for Advanced Features**

Redux Toolkit provides a set of tools to simplify Redux development, such as `createSlice`, `createAsyncThunk`, and `createEntityAdapter`.

- **createSlice**: Creates a slice of the Redux store with reducers and actions.
- **createAsyncThunk**: Handles async operations and generates action creators.
- **createEntityAdapter**: Manages normalized state for entities.

**Example**:
Using `createEntityAdapter` to manage normalized state.

```js
import { createSlice, createEntityAdapter } from '@reduxjs/toolkit';

const usersAdapter = createEntityAdapter();
const initialState = usersAdapter.getInitialState({
  loading: false,
  error: null,
});

const usersSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        usersAdapter.setAll(state, action.payload);
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  },
});

export default usersSlice.reducer;
```

### 7. **Redux Persist for State Persistence**

Persist and rehydrate a Redux store's state across page reloads using `redux-persist`. This ensures that the state is saved in local storage and restored when the user revisits the app.

- **redux-persist**: Middleware to persist and rehydrate the Redux store.

**Example**:
Setting up Redux Persist.

```js
import { persistStore, persistReducer } from 'redux-persist';
import storage from 'redux-persist/lib/storage';
import { configureStore } from '@reduxjs/toolkit';
import rootReducer from './rootReducer';

const persistConfig = {
  key: 'root',
  storage,
};

const persistedReducer = persistReducer(persistConfig, rootReducer);

const store = configureStore({
  reducer: persistedReducer,
});

const persistor = persistStore(store);

// Use PersistGate in your app to delay rendering until the state is rehydrated
```

### 8. **Handling Complex Async Flows with Redux-Saga**

Redux-Saga is a middleware library for handling complex asynchronous flows and side effects in Redux using generator functions.

- **Effects**: Functions to perform async tasks (e.g., call, put, takeEvery).
- **Generator Functions**: Functions that can be paused and resumed, ideal for async operations.

**Example**:
Using Redux-Saga for handling side effects.

```js
import { call, put, takeEvery } from 'redux-saga/effects';
import { fetchUserSuccess, fetchUserFailure } from './userSlice';
import api from './api';

// Worker saga
function* fetchUserSaga(action) {
  try {
    const user = yield call(api.fetchUser, action.payload);
    yield put(fetchUserSuccess(user));
  } catch (error) {
    yield put(fetchUserFailure(error));
  }
}

// Watcher saga
function* watchFetchUser() {
  yield takeEvery('user/fetchUserRequest', fetchUserSaga);
}

export default watchFetchUser;
```

### Summary

These advanced functionalities in React-Redux help in managing complex state, optimizing performance, handling asynchronous operations effectively, and providing a better user experience. By leveraging these techniques, you can build scalable, maintainable, and high-performance applications.
