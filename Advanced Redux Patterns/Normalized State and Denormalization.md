# **Normalized State and Denormalization**

### **What is Normalized State?**

**Normalized State** refers to the practice of storing data in a flat, non-nested structure within the Redux store. Instead of having deeply nested objects, normalized data typically involves storing data in separate collections (or tables) with **unique IDs** for each entry. This makes it easier to manage and update the state, as well as to avoid issues with data duplication and maintain consistency.

### **Why Normalize State?**

1. **Efficiency**: 
   - **Avoid duplication**: When you store data in a nested structure, there can be a lot of data duplication, especially if an entity is used in multiple places (e.g., users with multiple posts). Normalization reduces this duplication by referencing shared entities by their ID.
   - **Faster updates**: It’s easier to update data when it’s stored in a normalized way, as you can modify the specific entity without needing to traverse deeply nested objects.
   
2. **Maintain Consistency**: 
   - Normalizing data ensures that you can reference the same piece of data in multiple places without worrying about inconsistent updates.

3. **Easier Refactoring**: 
   - Changes to entities or data models can be handled more easily without needing to adjust deeply nested data structures.

---

### **Normalized Data Structure**

In a **normalized state**, data is typically stored as **collections of entities** with **unique IDs** for each entity. There are usually two main collections:
- **Entities**: A collection of objects, each with a unique ID.
- **IDs**: A collection that references the IDs of related entities, typically for relational data.

#### **Example of Normalized State**

Imagine an application that manages **posts** and **users**, where each post has an associated user.

Before normalization, a typical state might look like this:

```javascript
{
  posts: [
    { id: 1, title: 'First Post', user: { id: 1, name: 'Alice' } },
    { id: 2, title: 'Second Post', user: { id: 2, name: 'Bob' } }
  ]
}
```

This structure has nested objects (`user` inside `post`), which can cause redundancy (e.g., `Alice` and `Bob` are repeated multiple times for each post they create).

After normalizing, the state might look like this:

```javascript
{
  posts: {
    1: { id: 1, title: 'First Post', userId: 1 },
    2: { id: 2, title: 'Second Post', userId: 2 }
  },
  users: {
    1: { id: 1, name: 'Alice' },
    2: { id: 2, name: 'Bob' }
  },
  postIds: [1, 2],   // The IDs of the posts
  userIds: [1, 2]    // The IDs of the users
}
```

In this normalized state:
- The `posts` are stored with a unique `userId` linking them to the `users`.
- The `users` are stored independently with their own unique IDs.
- Both `postIds` and `userIds` arrays are used to reference which posts and users exist, keeping the relationships clear and efficient.

---

### **Normalization Benefits**

- **Reduced Duplication**: If a user has multiple posts, the user object is stored only once, reducing duplication.
- **Easy Updates**: Updating a user's information (e.g., name) doesn’t require updating every post associated with that user.
- **Efficient Lookups**: Using IDs to reference entities allows quick lookups and reduces the need to traverse complex object trees.

---

### **How to Normalize Data in Redux**

To normalize data in Redux, developers often use a library called **normalizr**, which provides a set of utilities to automatically normalize complex, nested data.

#### **Using `normalizr`**:

```javascript
import { normalize, schema } from 'normalizr';

// Define your schema
const user = new schema.Entity('users');
const post = new schema.Entity('posts', {
  author: user
});

// Normalize the data
const originalData = [
  {
    id: 1,
    title: 'First Post',
    author: { id: 1, name: 'Alice' }
  },
  {
    id: 2,
    title: 'Second Post',
    author: { id: 2, name: 'Bob' }
  }
];

const normalizedData = normalize(originalData, [post]);
console.log(normalizedData);
```

This will output:

```javascript
{
  entities: {
    posts: {
      1: { id: 1, title: 'First Post', author: 1 },
      2: { id: 2, title: 'Second Post', author: 2 }
    },
    users: {
      1: { id: 1, name: 'Alice' },
      2: { id: 2, name: 'Bob' }
    }
  },
  result: [1, 2]
}
```

Now, your state is normalized and ready for easy state management.

---

### **Denormalization**

**Denormalization** is the process of converting the normalized state back into a nested structure. This might be necessary when you need to display the data in a way that mimics the original nested structure, such as in a component that expects the data to be in a certain format.

Denormalization typically occurs at the UI layer, where the normalized state is transformed back into the shape required for rendering. It's often done using selectors.

#### **Denormalization Example**

Given the normalized state we created earlier:

```javascript
const state = {
  posts: {
    1: { id: 1, title: 'First Post', userId: 1 },
    2: { id: 2, title: 'Second Post', userId: 2 }
  },
  users: {
    1: { id: 1, name: 'Alice' },
    2: { id: 2, name: 'Bob' }
  },
  postIds: [1, 2],
  userIds: [1, 2]
};
```

To denormalize the data for a UI component, we might do something like this:

```javascript
const denormalizedPosts = state.postIds.map(postId => {
  const post = state.posts[postId];
  const user = state.users[post.userId];
  return {
    ...post,
    author: user
  };
});

console.log(denormalizedPosts);
```

This will output:

```javascript
[
  { id: 1, title: 'First Post', author: { id: 1, name: 'Alice' } },
  { id: 2, title: 'Second Post', author: { id: 2, name: 'Bob' } }
]
```

Now, the data is in the nested form expected by your UI component.

---

### **When to Use Normalization and Denormalization**

- **Normalize**: 
  - Use normalization when you have relational data (e.g., posts, users) and need to efficiently manage, update, and query the data.
  - Normalize your state if you want to reduce duplication and improve performance in large applications.
  
- **Denormalize**:
  - Denormalization should be done when you need to render data in a specific nested structure.
  - Use denormalization when your UI layer needs the data in a different format than the store’s normalized state.

---

### **Challenges of Normalization and Denormalization**

1. **Overhead in Code**:
   - Normalizing and denormalizing data can add some complexity to your code, especially if you're manually performing denormalization.

2. **Increased Memory Use**:
   - Storing multiple copies of related data (e.g., IDs and entities) can increase memory usage, especially in large applications.

3. **Complexity in Relational Data**:
   - When working with deeply nested relational data, you may need to carefully manage how entities are related, which can add complexity.

---

### **Conclusion**

- **Normalized State**: A flat structure with unique IDs for each entity that eliminates redundancy, improves performance, and simplifies state updates.
- **Denormalization**: The process of reconstructing the nested relationships for UI components that require it.
- **Tools like `normalizr`** help automate the normalization process, making it easier to manage complex state in Redux.

By adopting normalization for your Redux state, you can build more scalable and maintainable applications, especially when dealing with relational data.