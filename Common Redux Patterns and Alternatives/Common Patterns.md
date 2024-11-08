# Common Patterns in Redux

When working with Redux, there are several common patterns that help structure your application efficiently, improve code readability, and maintain scalability. These patterns focus on how state is managed, how actions are dispatched, and how reducers update the state. Here are the most common patterns you should be aware of when using Redux:

### 1. **Action and Reducer Pattern**

This is the most basic and fundamental pattern in Redux, where actions describe state transitions and reducers update the state based on those actions.

- **Action Creators**: Functions that create actions. These functions are responsible for constructing the `action` objects that will be dispatched to the Redux store.

    ```javascript
    const addItem = (item) => ({
      type: 'ADD_ITEM',
      payload: item
    });
    ```

- **Reducers**: Functions that take the current state and an action, and return a new state based on the action type. Reducers are pure functions, meaning they do not mutate the state.

    ```javascript
    const itemsReducer = (state = [], action) => {
      switch (action.type) {
        case 'ADD_ITEM':
          return [...state, action.payload];
        case 'REMOVE_ITEM':
          return state.filter(item => item.id !== action.payload.id);
        default:
          return state;
      }
    };
    ```

### 2. **Normalizing State Pattern**

To keep the state flat and prevent nesting, it’s a common practice to normalize state. This makes it easier to update and manage data by keeping it in a flat structure rather than deeply nested objects.

- **Normalization**: Store entities (e.g., users, posts) by their unique ID rather than nested arrays. This approach prevents data duplication and makes it easier to manage complex structures.

    ```javascript
    // Initial state with a normalized structure
    const initialState = {
      users: {
        1: { id: 1, name: 'Alice' },
        2: { id: 2, name: 'Bob' }
      },
      posts: {
        101: { id: 101, title: 'Hello World', userId: 1 },
        102: { id: 102, title: 'Redux Patterns', userId: 2 }
      }
    };
    ```

- **Libraries**: You can use libraries like `normalizr` to automate the normalization of data.

    ```javascript
    import { normalize, schema } from 'normalizr';

    const user = new schema.Entity('users');
    const post = new schema.Entity('posts', {
      author: user
    });

    const data = {
      id: 1,
      author: { id: 1, name: 'Alice' },
      title: 'My Post'
    };

    const normalizedData = normalize(data, post);
    ```

### 3. **Thunk Pattern for Async Actions**

One of the most common patterns in Redux is handling **asynchronous actions** using **redux-thunk**, a middleware that allows action creators to return functions instead of objects. These functions can then dispatch actions asynchronously, such as when fetching data from an API.

- **Thunk Middleware**: Thunks allow you to dispatch multiple actions and manage asynchronous side effects like network requests.

    ```javascript
    // Action creator using thunk to fetch data
    const fetchData = () => async (dispatch) => {
      dispatch({ type: 'FETCH_DATA_REQUEST' });
      try {
        const response = await fetch('/api/data');
        const data = await response.json();
        dispatch({ type: 'FETCH_DATA_SUCCESS', payload: data });
      } catch (error) {
        dispatch({ type: 'FETCH_DATA_FAILURE', error });
      }
    };
    ```

- **Advantages**:
  - Async actions are isolated in thunks, separating concerns from reducers.
  - You can dispatch multiple actions (e.g., loading state changes and error handling) within a single thunk.

### 4. **Selector Pattern**

Selectors are functions that compute derived data from the Redux state. The **selector pattern** helps to keep components decoupled from the structure of the state, making it easier to update the state structure without breaking the components.

- **Reselect**: A library that provides efficient selectors by memoizing the results of computations. This avoids recalculating derived data unless the state actually changes.

    ```javascript
    import { createSelector } from 'reselect';

    const getItems = (state) => state.items;

    const getSortedItems = createSelector(
      [getItems],
      (items) => items.sort((a, b) => a.name.localeCompare(b.name))
    );
    ```

- **Benefits**:
  - Provides a clean abstraction of how data is fetched from the state.
  - Can optimize performance by memoizing the result and preventing unnecessary re-renders.

### 5. **Container and Presentational Components Pattern**

This pattern divides components into **container components** and **presentational components**, which helps in organizing the components and making them more reusable and testable.

- **Container Components**: These components are aware of the Redux state and dispatch actions. They are responsible for fetching data and passing it down to the presentational components.

    ```javascript
    const mapStateToProps = (state) => ({
      items: state.items
    });

    const mapDispatchToProps = (dispatch) => ({
      addItem: (item) => dispatch(addItem(item))
    });

    const ItemContainer = connect(mapStateToProps, mapDispatchToProps)(ItemList);
    ```

- **Presentational Components**: These components are purely focused on rendering UI based on props. They do not have direct access to the Redux state or dispatch actions.

    ```javascript
    const ItemList = ({ items, addItem }) => (
      <div>
        {items.map(item => (
          <div key={item.id}>{item.name}</div>
        ))}
        <button onClick={() => addItem({ id: 3, name: 'New Item' })}>
          Add Item
        </button>
      </div>
    );
    ```

### 6. **Redux-Form Pattern**

Redux-Form is a common pattern for managing form state in Redux. It allows the form’s input state to be stored in the Redux store, ensuring that the form is consistently managed and easy to persist or validate.

- **Form State Management**: Redux-Form helps to manage form data, validation, and submission through Redux. It connects form fields to the Redux store and handles form submission logic.

    ```javascript
    import { Field, reduxForm } from 'redux-form';

    const MyForm = ({ handleSubmit }) => (
      <form onSubmit={handleSubmit}>
        <Field name="name" component="input" type="text" />
        <button type="submit">Submit</button>
      </form>
    );

    export default reduxForm({ form: 'myForm' })(MyForm);
    ```

### 7. **Redux-Persist Pattern**

For applications that need to persist state across page reloads or sessions, `redux-persist` is a popular solution. It stores the Redux state in `localStorage` or `sessionStorage` and reloads it when the app starts.

- **Persistent State**: Use `redux-persist` to automatically save part of your Redux state and load it back when the app is reinitialized.

    ```javascript
    import storage from 'redux-persist/lib/storage';
    import { persistReducer } from 'redux-persist';

    const persistConfig = {
      key: 'root',
      storage
    };

    const persistedReducer = persistReducer(persistConfig, rootReducer);
    ```

- **Advantages**:
  - Persist user authentication data, settings, or other important state across sessions.
  - Reduces the need for re-fetching data that does not change frequently.

### 8. **Feature-Based Folder Structure**

A **feature-based folder structure** organizes Redux logic (actions, reducers, selectors) by features instead of by file type. This makes it easier to manage and scale as the project grows.

- **Folder Organization**:
  - Instead of having separate folders for actions, reducers, and components, group everything by feature (e.g., `auth`, `items`, `cart`).
  - This reduces complexity when scaling the app and keeps related files together.

    ```
    src/
    ├── features/
    │   ├── auth/
    │   │   ├── authActions.js
    │   │   ├── authReducer.js
    │   │   ├── authSelectors.js
    │   │   └── AuthComponent.js
    │   ├── items/
    │   │   ├── itemsActions.js
    │   │   ├── itemsReducer.js
    │   │   ├── itemsSelectors.js
    │   │   └── ItemsList.js
    └── index.js
    ```

### Conclusion

These common patterns—such as the Action and Reducer Pattern, Normalizing State, Thunk for Async Actions, Selectors, Container and Presentational Components, Redux-Form, Redux-Persist, and Feature-Based Folder Structure—help organize Redux applications in a clean, scalable, and maintainable way. Using these patterns appropriately ensures that your Redux codebase remains easy to manage as your application grows.