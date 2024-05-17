# Redux Docs
To make API calls in a Redux application using React, you'll typically use a combination of Redux Toolkit and a middleware like `redux-thunk` or `redux-saga` to handle asynchronous actions. Below is a guide using `redux-toolkit` with `redux-thunk`.

### 1. Setup Redux Toolkit

First, you need to install the necessary packages:

```bash
npm install @reduxjs/toolkit react-redux axios
```

### 2. Create a Redux Slice

Create a slice that handles the state for your API calls. In this example, we will create a slice to fetch user data.

```js
// features/users/usersSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

export const fetchUsers = createAsyncThunk('users/fetchUsers', async () => {
  const response = await axios.get('https://jsonplaceholder.typicode.com/users');
  return response.data;
});

const usersSlice = createSlice({
  name: 'users',
  initialState: {
    users: [],
    status: 'idle',
    error: null,
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.users = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      });
  },
});

export default usersSlice.reducer;
```

### 3. Configure the Store

Configure the Redux store to use the slice you created.

```js
// app/store.js
import { configureStore } from '@reduxjs/toolkit';
import usersReducer from '../features/users/usersSlice';

export const store = configureStore({
  reducer: {
    users: usersReducer,
  },
});
```

### 4. Provide the Store to Your Application

Wrap your application with the `Provider` component and pass the store.

```jsx
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import { store } from './app/store';
import App from './App';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

### 5. Use the Slice in a React Component

Finally, use the slice and dispatch the `fetchUsers` action in your React component.

```jsx
// features/users/UsersList.js
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchUsers } from './usersSlice';

const UsersList = () => {
  const dispatch = useDispatch();
  const users = useSelector((state) => state.users.users);
  const status = useSelector((state) => state.users.status);
  const error = useSelector((state) => state.users.error);

  useEffect(() => {
    if (status === 'idle') {
      dispatch(fetchUsers());
    }
  }, [status, dispatch]);

  let content;

  if (status === 'loading') {
    content = <div>Loading...</div>;
  } else if (status === 'succeeded') {
    content = (
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    );
  } else if (status === 'failed') {
    content = <div>{error}</div>;
  }

  return (
    <section>
      <h2>Users</h2>
      {content}
    </section>
  );
};

export default UsersList;
```

To configure a Redux store with multiple reducers, you can use the `combineReducers` function from Redux. This allows you to manage different parts of your state tree independently with separate reducers.

Here's how you can set up a Redux store with multiple reducers:

### 1. Create Your Slices

First, create your slices. For this example, we'll assume you have two slices: `usersSlice` and `postsSlice`.

**`features/users/usersSlice.js`**:
```js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

export const fetchUsers = createAsyncThunk('users/fetchUsers', async () => {
  const response = await axios.get('https://jsonplaceholder.typicode.com/users');
  return response.data;
});

const usersSlice = createSlice({
  name: 'users',
  initialState: {
    users: [],
    loading: false,
    error: null,
    success: false,
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
        state.success = false;
        state.error = null;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.success = true;
        state.users = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.success = false;
        state.error = action.error.message;
      });
  },
});

export default usersSlice.reducer;
```

**`features/posts/postsSlice.js`**:
```js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

export const fetchPosts = createAsyncThunk('posts/fetchPosts', async () => {
  const response = await axios.get('https://jsonplaceholder.typicode.com/posts');
  return response.data;
});

const postsSlice = createSlice({
  name: 'posts',
  initialState: {
    posts: [],
    loading: false,
    error: null,
    success: false,
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchPosts.pending, (state) => {
        state.loading = true;
        state.success = false;
        state.error = null;
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.loading = false;
        state.success = true;
        state.posts = action.payload;
      })
      .addCase(fetchPosts.rejected, (state, action) => {
        state.loading = false;
        state.success = false;
        state.error = action.error.message;
      });
  },
});

export default postsSlice.reducer;
```

### 2. Combine Reducers

Use `combineReducers` to combine your reducers into a single root reducer.

**`app/rootReducer.js`**:
```js
import { combineReducers } from 'redux';
import usersReducer from '../features/users/usersSlice';
import postsReducer from '../features/posts/postsSlice';

const rootReducer = combineReducers({
  users: usersReducer,
  posts: postsReducer,
});

export default rootReducer;
```

### 3. Configure the Store

Create the Redux store with the combined reducers.

**`app/store.js`**:
```js
import { configureStore } from '@reduxjs/toolkit';
import rootReducer from './rootReducer';

export const store = configureStore({
  reducer: rootReducer,
});
```

### 4. Provide the Store to Your Application

Wrap your application with the `Provider` component and pass the store.

**`index.js`**:
```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import { store } from './app/store';
import App from './App';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

### 5. Use the Slices in Your Components

You can now use the slices in your components as needed.

**`features/users/UsersList.js`**:
```jsx
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchUsers } from './usersSlice';

const UsersList = () => {
  const dispatch = useDispatch();
  const { users, loading, error, success } = useSelector((state) => state.users);

  useEffect(() => {
    if (!success && !loading) {
      dispatch(fetchUsers());
    }
  }, [success, loading, dispatch]);

  let content;

  if (loading) {
    content = <div>Loading...</div>;
  } else if (success) {
    content = (
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    );
  } else if (error) {
    content = <div>{error}</div>;
  }

  return (
    <section>
      <h2>Users</h2>
      {content}
    </section>
  );
};

export default UsersList;
```

**`features/posts/PostsList.js`**:
```jsx
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchPosts } from './postsSlice';

const PostsList = () => {
  const dispatch = useDispatch();
  const { posts, loading, error, success } = useSelector((state) => state.posts);

  useEffect(() => {
    if (!success && !loading) {
      dispatch(fetchPosts());
    }
  }, [success, loading, dispatch]);

  let content;

  if (loading) {
    content = <div>Loading...</div>;
  } else if (success) {
    content = (
      <ul>
        {posts.map((post) => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    );
  } else if (error) {
    content = <div>{error}</div>;
  }

  return (
    <section>
      <h2>Posts</h2>
      {content}
    </section>
  );
};

export default PostsList;
```

### Summary

By using `combineReducers` from Redux, you can manage different parts of your state tree with multiple reducers, making your application more modular and easier to maintain. Each slice handles its own part of the state, and the combined reducer manages the overall state.

### Summary

This example demonstrates a basic setup for making API calls with Redux Toolkit in a React application. It uses `createAsyncThunk` for handling asynchronous actions and `createSlice` to manage the state. The `UsersList` component fetches and displays a list of users from an API.

If you need more advanced usage or want to handle more complex scenarios, consider looking into `redux-saga` for handling side effects in a more scalable way.
   
