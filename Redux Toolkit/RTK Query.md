# RTK Query (Redux Toolkit Query)

RTK Query is a powerful data fetching and caching tool built into Redux Toolkit. It provides a set of utilities to manage server-side data fetching, caching, synchronization, and other related operations in a Redux-powered application.

### 1. What is RTK Query?

RTK Query is an abstraction on top of Redux that simplifies data fetching, caching, and synchronization of data between the client and the server. It is part of Redux Toolkit and is used to handle API requests and manage data in your Redux store automatically.

### 2. Why Use RTK Query?

- **Simplified Data Fetching**: RTK Query simplifies the process of handling API calls by automating common tasks such as caching, request lifecycle management, and error handling.
  
- **Optimized Caching**: RTK Query offers built-in caching strategies, reducing unnecessary API calls by reusing previously fetched data.

- **Automatic Redux Slice Generation**: RTK Query generates Redux slices and reducers automatically to store API responses and manage loading and error states.

- **Reduced Boilerplate**: It drastically reduces the boilerplate code associated with traditional Redux patterns, such as action creators, reducers, and selectors.

- **Optimized Performance**: It optimizes network requests by using strategies like polling, pagination, and caching out of the box.

### 3. Key Features of RTK Query

- **Auto-generated reducers and actions**: It automatically creates reducers and actions for different states such as loading, success, and error.
  
- **Caching**: RTK Query automatically caches responses and offers cache invalidation features.

- **Polling**: You can set polling intervals to refresh data automatically.

- **Optimistic Updates**: RTK Query supports optimistic updates to show real-time data while waiting for the response from the server.

- **Pagination**: It supports pagination for API responses, making it easier to deal with large datasets.

- **Error Handling**: It provides a simplified way to handle loading, success, and error states.

### 4. Setting Up RTK Query

1. **Install Redux Toolkit** (if not installed already):
   ```bash
   npm install @reduxjs/toolkit react-redux
   ```

2. **Set up the API service**: The `createApi` function from RTK Query is used to define an API slice. It accepts an object that defines the base URL for your API, endpoints, and how to handle those endpoints.

```javascript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

const api = createApi({
  reducerPath: 'api', // Store slice name
  baseQuery: fetchBaseQuery({ baseUrl: 'https://api.example.com/' }), // Base URL
  endpoints: (builder) => ({
    // Define the endpoints
    getPosts: builder.query({
      query: () => 'posts', // GET /posts
    }),
    getUser: builder.query({
      query: (id) => `users/${id}`, // GET /users/:id
    }),
  }),
});

export const { useGetPostsQuery, useGetUserQuery } = api; // Generated hooks for components

export default api;
```

3. **Add the API slice to the store**:

```javascript
import { configureStore } from '@reduxjs/toolkit';
import api from './services/api';

const store = configureStore({
  reducer: {
    [api.reducerPath]: api.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(api.middleware), // Add RTK Query middleware
});

export default store;
```

### 5. Using RTK Query in Components

After defining the API service, you can use the auto-generated hooks in your components to fetch data.

```javascript
import React from 'react';
import { useGetPostsQuery } from './services/api';

const PostsList = () => {
  const { data, error, isLoading } = useGetPostsQuery();

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error occurred: {error.message}</div>;

  return (
    <ul>
      {data.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
};

export default PostsList;
```

- **`useGetPostsQuery`**: The generated hook is automatically connected to Redux and fetches data from the `getPosts` endpoint.
- **`isLoading`**: Tracks the loading state.
- **`error`**: Handles errors that occur during the API request.
- **`data`**: Contains the data returned from the server.

### 6. Handling Mutations with RTK Query

For creating, updating, or deleting data, you can use mutations in RTK Query.

```javascript
const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://api.example.com/' }),
  endpoints: (builder) => ({
    createPost: builder.mutation({
      query: (newPost) => ({
        url: 'posts',
        method: 'POST',
        body: newPost,
      }),
    }),
  }),
});

export const { useCreatePostMutation } = api;
```

To use this mutation in a component:

```javascript
import React, { useState } from 'react';
import { useCreatePostMutation } from './services/api';

const CreatePost = () => {
  const [newPost, setNewPost] = useState({ title: '', body: '' });
  const [createPost, { isLoading, error }] = useCreatePostMutation();

  const handleSubmit = async (e) => {
    e.preventDefault();
    await createPost(newPost); // Trigger the mutation
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={newPost.title}
        onChange={(e) => setNewPost({ ...newPost, title: e.target.value })}
      />
      <textarea
        value={newPost.body}
        onChange={(e) => setNewPost({ ...newPost, body: e.target.value })}
      />
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Creating...' : 'Create Post'}
      </button>
      {error && <div>Error: {error.message}</div>}
    </form>
  );
};

export default CreatePost;
```

- **`useCreatePostMutation`**: The generated hook handles the POST request to create a new post.
- **`isLoading`**: Tracks the loading state for the mutation.
- **`error`**: Handles errors for the mutation.

### 7. RTK Query Caching and Cache Management

RTK Query automatically caches API responses and updates the cache as needed. Some key features related to caching include:

- **Caching by Default**: Data fetched through RTK Query is cached by default and will be reused until it expires.
  
- **Cache Lifetimes**: You can set cache lifetimes for different endpoints to determine how long the cache stays valid.

- **Cache Invalidation**: You can invalidate the cache when data changes, ensuring that stale data is removed and refreshed.

Example of cache invalidation when creating a post:

```javascript
const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://api.example.com/' }),
  endpoints: (builder) => ({
    createPost: builder.mutation({
      query: (newPost) => ({
        url: 'posts',
        method: 'POST',
        body: newPost,
      }),
      // Invalidate the cache for 'getPosts' endpoint after mutation
      invalidatesTags: [{ type: 'Posts', id: 'LIST' }],
    }),
    getPosts: builder.query({
      query: () => 'posts',
      providesTags: ['Posts'],
    }),
  }),
});

export const { useCreatePostMutation, useGetPostsQuery } = api;
```

### 8. Advanced RTK Query Features

- **Polling**: RTK Query supports polling by setting a polling interval for a query endpoint.

```javascript
const { data } = useGetPostsQuery(undefined, {
  pollingInterval: 3000, // Poll every 3 seconds
});
```

- **Optimistic Updates**: You can implement optimistic updates to show UI changes before the server responds.

```javascript
const api = createApi({
  endpoints: (builder) => ({
    createPost: builder.mutation({
      query: (newPost) => ({
        url: 'posts',
        method: 'POST',
        body: newPost,
      }),
      // Optimistic update: Directly add the new post to the state
      onQueryStarted: async (newPost, { dispatch, queryFulfilled }) => {
        const patchResult = dispatch(
          api.util.updateQueryData('getPosts', undefined, (draft) => {
            draft.push(newPost);
          })
        );
        try {
          await queryFulfilled;
        } catch {
          patchResult.undo(); // Undo if the request fails
        }
      },
    }),
  }),
});
```

### 9. Conclusion

RTK Query is a powerful tool that simplifies managing API requests, state management, and caching in Redux. It reduces the need for boilerplate code while providing robust features like caching, pagination, optimistic updates, and polling. RTK Query helps build efficient and scalable applications with minimal effort, leveraging Redux for automatic data synchronization.

