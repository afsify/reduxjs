# **Immutable Update Patterns**

### **What is Immutability?**

Immutability means that once an object is created, it cannot be modified. Instead of directly changing the state, we return a new copy of the state with the necessary updates. This is important for Redux because it ensures that state updates are predictable and maintainable, while also allowing Redux to efficiently detect changes to the state for triggering re-renders.

In Redux, state is never mutated directly. Instead, when you update the state, you create a new state object, keeping the old state unchanged.

### **Why is Immutability Important in Redux?**

1. **Predictability**:
   - With immutability, the state is never changed directly, ensuring that each action results in a new state, which can be easily tracked and predicted.
  
2. **Efficiency**:
   - Immutability enables Redux’s change detection mechanism to be more efficient. It compares references of the state to detect changes instead of deeply comparing every object property. This improves performance by reducing unnecessary computations.

3. **Easier Debugging**:
   - When state changes are always made in a controlled, immutable way, it’s easier to debug applications. With tools like Redux DevTools, you can track state changes over time, allowing for better insights into how the state evolves.

4. **Enabling Undo/Redo and Time Travel**:
   - Immutability makes it easier to implement features like undo/redo because you can store past states and revert to them at any point in time.

### **General Immutable Update Patterns in Redux**

To update an immutable state in Redux, we cannot directly modify the existing state. Instead, we must create and return a new copy of the state with the required changes.

Below are common update patterns to handle different data types in Redux:

---

### **1. Updating Arrays in Immutable Way**

When updating arrays, we must avoid directly modifying the array. Instead, we create a new array with the updated values.

- **Adding an item**:
  - Use the spread operator (`...`) to create a new array and add an element.

  ```javascript
  const newState = [...state.items, newItem];
  ```

- **Removing an item**:
  - Use `filter` to create a new array with the items excluding the one to remove.

  ```javascript
  const newState = state.items.filter(item => item.id !== action.payload.id);
  ```

- **Updating an item**:
  - Use `map` to create a new array with the updated item.

  ```javascript
  const newState = state.items.map(item =>
    item.id === action.payload.id ? { ...item, ...action.payload.updatedData } : item
  );
  ```

- **Replacing the array**:
  - Use the spread operator to return a completely new array with all updated items.

  ```javascript
  const newState = [...action.payload.newItems];
  ```

---

### **2. Updating Objects in Immutable Way**

When updating objects, we also create a new object with the updated properties, leaving the old object unchanged.

- **Updating a property**:
  - Use the spread operator to create a new object with the updated property.

  ```javascript
  const newState = { ...state, property: newValue };
  ```

- **Updating a nested object**:
  - Use the spread operator to update the object’s nested properties. This ensures the original nested object is not mutated.

  ```javascript
  const newState = { 
    ...state, 
    nestedObject: { 
      ...state.nestedObject, 
      nestedProperty: newValue 
    } 
  };
  ```

- **Deleting a property**:
  - Use `delete` or `omit` (from libraries like `lodash` or `ramda`) to remove properties, returning a new object without the specified property.

  ```javascript
  const { [action.payload.property]: deleted, ...newState } = state;
  ```

---

### **3. Updating Deeply Nested Structures**

In deeply nested structures, such as an array of objects with deeply nested properties, updating immutably requires multiple levels of object spreading.

- **Deeply Nested Object Update**:
  - When updating deeply nested structures, you must spread each level of the object or array to maintain immutability.

  ```javascript
  const newState = {
    ...state,
    users: state.users.map(user =>
      user.id === action.payload.id
        ? { ...user, profile: { ...user.profile, name: action.payload.newName } }
        : user
    )
  };
  ```

---

### **4. Using `map()` for Array Updates**

For an immutable update of array elements, `map()` is often used to loop over the array and conditionally update items.

- **Example of Array Update with `map()`**:

  ```javascript
  const updatedItems = state.items.map(item => {
    if (item.id === action.payload.id) {
      return { ...item, value: action.payload.newValue };
    }
    return item;
  });
  ```

This will return a new array with the updated item while leaving other items unchanged.

---

### **5. Using `filter()` to Remove Items from an Array**

To remove an item from an array in an immutable way, we can use `filter()`. This creates a new array that contains all the elements except the one we wish to remove.

- **Example of Remove Item**:

  ```javascript
  const newState = state.items.filter(item => item.id !== action.payload.id);
  ```

---

### **6. Using `concat()` to Add Items to an Array**

While you can use the spread operator (`...`) to add an item to an array, you can also use the `concat()` method to achieve an immutable update.

- **Example of Adding an Item**:

  ```javascript
  const newState = state.items.concat(newItem);
  ```

---

### **7. Helper Libraries for Immutability**

To make immutability easier and more readable, you can use libraries that simplify deep cloning and updating nested data structures.

- **`immutability-helper`**: A library that allows for concise immutable updates to arrays and objects.
- **`Immer`**: A library that makes it easier to work with immutable state by using “drafts” of objects.
  
Example using **`Immer`**:

```javascript
import produce from 'immer';

const newState = produce(state, draftState => {
  draftState.items.push(newItem);  // Mutates draftState but keeps it immutable
});
```

---

### **8. Avoiding Direct Mutation of State**

The key to following immutable patterns in Redux is ensuring that you never mutate the original state. Always create a new object or array by using methods like `map()`, `filter()`, `concat()`, or the spread operator to update state. Direct mutation leads to bugs and problems with state tracking in Redux.

---

### **Summary of Immutable Update Patterns**

- **Arrays**: Use methods like `map()`, `filter()`, `concat()`, or the spread operator to create new arrays when adding, removing, or updating items.
- **Objects**: Use the spread operator (`{ ...state, newKey: newValue }`) to create new objects, and for nested objects, apply the spread operator recursively.
- **Deeply Nested Structures**: Apply spread operators at every level of the object to ensure immutability.
- **Helper Libraries**: Use libraries like `Immer` or `immutability-helper` to simplify and optimize immutable updates.

By following immutable update patterns, you ensure that your Redux store is updated correctly, enabling Redux to efficiently track changes and re-render components.