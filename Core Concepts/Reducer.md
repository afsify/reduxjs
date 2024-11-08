# Reducers in Redux

Reducers are essential functions in Redux responsible for updating the state based on dispatched actions. They are pure functions that take the current state and an action, process the action, and return a new state.

### Key Concepts for Reducers

1. **Role of Reducers in Redux**  
   - Reducers define how the application’s state changes in response to actions.
   - Each reducer function takes two arguments: the **current state** and an **action**. It determines what changes to make to the state based on the action type and payload.
   - Instead of modifying the state directly, reducers return a new state object, maintaining Redux’s principle of immutability.
   
   **Example**: Basic reducer for handling a counter  
   ```javascript
   function counterReducer(state = { count: 0 }, action) {
     switch (action.type) {
       case 'INCREMENT':
         return { ...state, count: state.count + 1 };
       case 'DECREMENT':
         return { ...state, count: state.count - 1 };
       default:
         return state;
     }
   }
   ```

2. **Pure Functions and Immutability**  
   - **Pure Functions**: Reducers are pure functions, meaning they have no side effects and produce the same output for the same input. They don’t modify external state or perform asynchronous operations.
   - **Immutability**: Redux encourages immutability; reducers shouldn’t mutate the existing state directly. Instead, they return a new copy of the state object with updates.

   **Example**: Using spread operator to return a new state object  
   ```javascript
   function todoReducer(state = [], action) {
     switch (action.type) {
       case 'ADD_TODO':
         return [...state, action.payload];
       case 'REMOVE_TODO':
         return state.filter(todo => todo.id !== action.payload.id);
       default:
         return state;
     }
   }
   ```

3. **Initial State and Default Parameters**  
   - Reducers use default parameters to set the **initial state** if no state is provided. This is important for defining the shape of the state at the start.
   - By providing an initial state, the application has a predictable starting point for its data, even if no actions have been dispatched yet.

   **Example**: Setting an initial state with default parameters  
   ```javascript
   function userReducer(state = { loggedIn: false, user: null }, action) {
     switch (action.type) {
       case 'LOGIN':
         return { ...state, loggedIn: true, user: action.payload };
       case 'LOGOUT':
         return { ...state, loggedIn: false, user: null };
       default:
         return state;
     }
   }
   ```

4. **Composing Multiple Reducers Using `combineReducers`**  
   - `combineReducers` is a Redux helper function that merges multiple reducers into a single “root reducer” function. Each slice of state is managed by its own reducer, but all are combined into a single state tree.
   - This approach allows each reducer to focus on a specific part of the state, improving code organization and scalability.

   **Example**: Using `combineReducers` to create a root reducer  
   ```javascript
   import { combineReducers } from 'redux';
   import userReducer from './userReducer';
   import cartReducer from './cartReducer';

   const rootReducer = combineReducers({
     user: userReducer,
     cart: cartReducer,
   });
   export default rootReducer;
   ```

   - **How `combineReducers` Works**  
     Each reducer manages a specific slice of state. The `combineReducers` function takes a mapping of reducer functions and associates each one with a key. The returned root reducer uses each key to update the corresponding part of the state tree.

5. **Splitting Logic Across Reducers (Avoiding Nested Reducers)**  
   - Splitting logic across reducers helps avoid overly complex nested reducers. Each reducer should ideally manage a single “slice” of the state, focusing on one specific area of functionality.
   - Rather than creating deeply nested structures, Redux encourages flat and organized state trees with specific reducers for each slice.
   - **Best Practice**: Keep reducers focused and simple. If a reducer becomes complex, break it down by separating logic into additional reducers and combine them using `combineReducers`.

   **Example**: Splitting logic across reducers for `user` and `preferences`  
   ```javascript
   const userReducer = (state = { name: "", age: 0 }, action) => {
     switch (action.type) {
       case 'SET_USER':
         return { ...state, ...action.payload };
       default:
         return state;
     }
   };

   const preferencesReducer = (state = { theme: "light" }, action) => {
     switch (action.type) {
       case 'SET_THEME':
         return { ...state, theme: action.payload };
       default:
         return state;
     }
   };

   // Combining into rootReducer
   const rootReducer = combineReducers({
     user: userReducer,
     preferences: preferencesReducer,
   });
   ```

---

### Summary of Key Points

- **Role of Reducers**: Reducers define how state changes in response to actions; they are central to state updates in Redux.
- **Pure Functions & Immutability**: Reducers should be pure functions and return new state objects without modifying existing state.
- **Initial State & Default Parameters**: Define an initial state to set the default structure and values for each reducer.
- **Combining Reducers**: Use `combineReducers` to organize state into slices, improving code modularity and maintainability.
- **Splitting Logic Across Reducers**: Avoid nested reducers by splitting logic into smaller, focused reducers for each slice of state.

Reducers are the backbone of Redux’s state management, helping maintain a structured, predictable, and immutable state throughout an application.