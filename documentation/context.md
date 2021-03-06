# Context

Solid has Context API for dependency injection which comprises of createContext, Provider control flow, and useContext. createContext lets you create the Context Object. When Solid renders a component that subscribes to this Context object it will read the current context value from the closest matching Provider above it in the tree. If there is not provider above it will use the default value.

Example below using Solid's own state mechanism to create a global store that wraps the whole app. Notice that the `Nested` component can access the counter without it being passed down via Props. While you could arguably use a singleton, this pattern gives clear ownership and allows hierarchical store contexts. You could use stores in this case for common data concerns that could appear multiple times on the same page like pagination, scrolling, etc, and have child components that know how to interact with their associated stores. This is a very powerful pattern as you compose patterns that wrap Solid's primitives and Context to create reusable injectable multi component driving modules.

> **For React Users:** Although the API surface is the same, Solid's Context API can house just about anything. The reactivity comes from the primitives not from the top down hierarchy. This means that it isn't the parent components driving reactivity, so there aren't any performance concerns, but context only updates when the primitives you use update.

```jsx
// counter-context.js
import { createState, createContext } from "solid-js";

export const CounterContext = createContext([{ count: 0 }, {}]);

export function CounterProvider(props) {
  const [state, setState] = createState({ count: props.count || 0 }),
    store = [
      state,
      {
        increment() {
          setState("count", c => c + 1);
        },
        decrement() {
          setState("count", c => c - 1);
        }
      }
    ];

  return (
    <CounterContext.Provider value={store}>
      {props.children}
    </CounterContext.Provider>
  );
}

// index.js
import { render } from "solid-js/dom";
import { CounterProvider } from "./counter-context";
import App from "./app";

render(
  () => (
    // start counter at 2
    <CounterProvider count={2}>
      <App />
    </CounterProvider>
  ),
  document.getElementById("app")
);

// app.js
import Nested from "./nested";

export default const App = () => (
  <>
    <h2>Welcome to Counter App</h2>
    <Nested />
  </>
);

// nested.js
import { useContext } from "solid-js";
import { CounterContext } from "./counter-context";

export default const Nested = () => {
  const [counter, { increment, decrement }] = useContext(CounterContext);
  return (
    <>
      <div>{counter.count}</div>
      <button onclick={increment}>+</button>
      <button onclick={decrement}>-</button>
    </>
  );
};
```
