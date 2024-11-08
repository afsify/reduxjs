# Creating Custom Middleware

Custom middleware in Redux is a powerful mechanism to extend or modify the behavior of the Redux store, allowing you to introduce custom logic in the action dispatching process. Middleware provides a way to add features like logging, routing, asynchronous task handling, and more, without modifying the core Redux store functionality.

### 1. Writing Custom Middleware Functions

A middleware is a higher-order function that wraps the dispatch method. It is inserted into the Redux middleware pipeline to intercept actions that are dispatched to the store. Writing custom middleware gives you full control over how actions are handled before they reach the reducer.

Here’s the basic structure of custom middleware:

```javascript
const customMiddleware = store => next => action => {
  // Custom logic before action is passed to the reducer
  console.log('Dispatching action:', action);

  // Call the next middleware or dispatch the action to the reducer
  return next(action);
};
```

In this example:
- **`store`**: The store object that contains the `dispatch` and `getState` methods.
- **`next`**: The next middleware in the chain, or the dispatch method if there are no other middlewares.
- **`action`**: The dispatched action that is intercepted by the middleware.

### 2. Middleware Signature and Parameters

Middleware functions are written with a specific signature. The structure looks like this:

```javascript
const middleware = store => next => action => {
  // logic
};
```

- **`store`**: An object that contains the Redux store’s `dispatch` and `getState` functions. The store is injected into the middleware, allowing you to access the store’s current state or dispatch other actions.
  
- **`next`**: A function that allows the action to pass to the next middleware in the pipeline. If there are no other middlewares, it passes the action to the reducer. You **must** call `next(action)` in order to continue the action flow.
  
- **`action`**: The action object dispatched by the Redux store. This is the action the middleware is intercepting and can be modified or logged before passing it down the chain.

### 3. Handling Async Tasks and Side Effects in Middleware

One of the most common use cases for custom middleware is handling asynchronous tasks or side effects, such as making API calls or logging.

#### Example: Handling Async Actions (Thunk-like Middleware)

Redux does not handle asynchronous actions out of the box. However, you can create custom middleware to handle actions that return functions or promises, similar to **redux-thunk**. This allows you to dispatch functions that perform asynchronous tasks before dispatching the final action.

Here’s how you could create a middleware that handles async actions:

```javascript
const asyncMiddleware = store => next => action => {
  if (typeof action === 'function') {
    // If action is a function, dispatch it with `store.dispatch`
    return action(store.dispatch, store.getState);
  }

  // If action is not a function, pass it along the chain
  return next(action);
};
```

#### Explanation:
- **Detecting Async Actions**: In this middleware, we check if the dispatched action is a **function**. If it is, we assume it’s an async action, and we invoke it with `dispatch` and `getState` as arguments.
- **Passing non-async Actions**: If the action is not a function, we call `next(action)` to pass it through the middleware pipeline and on to the reducer.

#### Example: Handling Side Effects (Logging Middleware)

You might want to log all dispatched actions and the resulting state after each dispatch. Here’s an example of a logging middleware that handles side effects:

```javascript
const loggerMiddleware = store => next => action => {
  console.log('Dispatching:', action);
  console.log('State before action:', store.getState());

  // Pass the action to the next middleware or reducer
  const result = next(action);

  console.log('State after action:', store.getState());

  return result;
};
```

#### Explanation:
- **Logging before Action**: Logs the dispatched action and the current state of the store.
- **Calling `next(action)`**: Passes the action to the next middleware or to the reducer.
- **Logging after Action**: Logs the updated state after the action has been processed by the reducer.

### 4. Combining Custom Middleware with Redux Store

To use custom middleware, you need to pass it to the store during store creation. When using **`createStore`**, you can pass middleware as an array to `applyMiddleware`:

```javascript
import { createStore, applyMiddleware } from 'redux';

const rootReducer = (state = {}, action) => {
  switch (action.type) {
    case 'EXAMPLE_ACTION':
      return { ...state, updated: true };
    default:
      return state;
  }
};

const store = createStore(
  rootReducer,
  applyMiddleware(customMiddleware, asyncMiddleware, loggerMiddleware) // Add custom middleware here
);
```

If you're using **Redux Toolkit**, the middleware can be added using the `middleware` option in `configureStore()`:

```javascript
import { configureStore } from '@reduxjs/toolkit';

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(customMiddleware, asyncMiddleware)
});
```

### 5. Example of Handling API Requests in Custom Middleware

Here’s an example of middleware that handles API requests, showing how you can manage side effects like fetching data from an API before dispatching actions.

```javascript
const apiMiddleware = store => next => action => {
  if (action.type === 'FETCH_DATA_REQUEST') {
    fetch(action.payload.url)
      .then(response => response.json())
      .then(data => {
        store.dispatch({ type: 'FETCH_DATA_SUCCESS', payload: data });
      })
      .catch(error => {
        store.dispatch({ type: 'FETCH_DATA_FAILURE', payload: error });
      });

    // Avoid dispatching the original action and let the async process take over
    return;
  }

  return next(action); // Pass non-API actions to next middleware
};
```

#### Explanation:
- **Checking Action Type**: The middleware checks if the action is a specific type, in this case, `'FETCH_DATA_REQUEST'`.
- **Making the API Call**: If it is an API call action, it performs an HTTP request using `fetch()`.
- **Dispatching Success/Failure Actions**: Based on the response, it dispatches either a success or failure action.
- **No Need to Pass Action**: The original action (`FETCH_DATA_REQUEST`) is not passed along the middleware chain because the API call is handling the side effect.

### 6. Conclusion

Custom middleware is an essential tool for handling side effects like async requests, logging, and more in a Redux application. By writing middleware, you can intercept actions, modify them, dispatch new actions, or perform side effects before the action reaches the reducer.

- **Middleware Signature**: Middleware follows the `store => next => action => {}` pattern, giving you access to the store, next middleware, and dispatched actions.
- **Async Task Handling**: Middleware is commonly used to handle async operations, such as API calls or timeouts, without needing to modify the core Redux logic.
- **Side Effects**: Custom middleware can be used to log, track, or trigger side effects in response to specific actions, improving application flow and debugging.

By using custom middleware efficiently, you can create powerful and flexible Redux-based applications that handle asynchronous operations and side effects in a structured and clean way.