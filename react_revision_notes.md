# React Revision Notes

> Interview-oriented revision notes covering the React concepts
> discussed so far.

------------------------------------------------------------------------

## 1. React Mental Model

React is a JavaScript library for building user interfaces using
components.

A useful mental model:

``` text
Props + State
     ↓
Component renders
     ↓
React element tree
     ↓
Reconciliation
     ↓
Commit necessary DOM changes
```

### Render vs DOM update

A **re-render** means React calls the component again to calculate the
new UI. It does **not** mean React destroys and recreates the entire
DOM.

React compares the previous and new element trees during
**reconciliation**, then applies only the necessary DOM changes during
the commit phase.

The "Virtual DOM" is useful as a mental model, but it is not literally a
complete browser DOM copy.

------------------------------------------------------------------------

# 2. State and State Snapshots

State is a snapshot associated with a particular render.

``` jsx
const [count, setCount] = useState(0);

function handleClick() {
  setCount(count + 1);
  console.log(count);
}
```

The `console.log` prints the value from the current render, not the
newly scheduled value.

### Multiple updates

This:

``` jsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

when `count` is `0`, results in:

``` text
1
```

because all three calls use the same render snapshot.

### Functional updater

``` jsx
setCount(prev => prev + 1);
setCount(prev => prev + 1);
setCount(prev => prev + 1);
```

This results in:

``` text
3
```

The functional updater receives the latest pending state value.

### Important wording

`prev` is not simply "the previous render."

It is the latest state value React has available when processing that
update.

------------------------------------------------------------------------

# 3. State Immutability

React state should be treated as immutable.

## Objects

Avoid:

``` jsx
user.name = "Sonam";
setUser(user);
```

Instead:

``` jsx
setUser({
  ...user,
  name: "Sonam"
});
```

`...user` copies the existing object's properties into a new object.

## Arrays

Avoid:

``` jsx
items.push("D");
setItems(items);
```

Instead:

``` jsx
setItems([...items, "D"]);
```

## Nested objects

``` jsx
setUser({
  ...user,
  address: {
    ...user.address,
    city: "Paro"
  }
});
```

You create new references along the path to the property being changed.

------------------------------------------------------------------------

# 4. Hooks

Hooks are JavaScript functions provided by React or written by
developers that let function components use React features and reusable
stateful logic.

Common built-in Hooks:

``` text
useState
useEffect
useRef
useContext
useReducer
useMemo
useCallback
```

### Rules of Hooks

Hooks should be called:

-   At the top level of a function component
-   At the top level of a custom Hook

Do not call Hooks inside:

-   Conditions
-   Loops
-   Nested functions
-   Regular non-Hook functions

------------------------------------------------------------------------

# 5. useState

Basic usage:

``` jsx
const [count, setCount] = useState(0);
```

-   `count` = current render's state snapshot
-   `setCount` = schedules a state update

State updates cause React to render again when appropriate.

------------------------------------------------------------------------

# 6. useEffect

`useEffect` is used to synchronize a component with an external system.

Examples:

-   API requests
-   Timers
-   Event listeners
-   Subscriptions
-   WebSockets
-   `localStorage`
-   `document.title`

General pattern:

``` jsx
useEffect(() => {
  // side effect

  return () => {
    // cleanup
  };
}, [dependencies]);
```

## Dependency array

### No dependency array

``` jsx
useEffect(() => {
  console.log("effect");
});
```

Runs after every render.

### Empty dependency array

``` jsx
useEffect(() => {
  console.log("effect");
}, []);
```

Runs after the initial mount in the normal production mental model.

Development Strict Mode can intentionally perform extra setup/cleanup
cycles.

### Dependency

``` jsx
useEffect(() => {
  console.log(count);
}, [count]);
```

Runs after mount and when `count` changes.

------------------------------------------------------------------------

# 7. Mount, Re-render, Unmount

### Mount

The component is committed to the DOM for the first time.

### Re-render

React calls the component again because state, props, context, or
another relevant update caused rendering.

### Unmount

The component is removed from the UI tree.

------------------------------------------------------------------------

# 8. Effect Cleanup

Cleanup runs:

1.  Before an effect reruns because its dependencies changed.
2.  When the component unmounts.

Example:

``` jsx
useEffect(() => {
  const timer = setTimeout(() => {
    console.log("done");
  }, 1000);

  return () => clearTimeout(timer);
}, [query]);
```

If `query` changes before the timer finishes, React runs cleanup and
cancels the old timer.

------------------------------------------------------------------------

# 9. Debouncing with useEffect

Debouncing waits until activity stops for a specified amount of time
before executing a function.

``` jsx
useEffect(() => {
  const timer = setTimeout(() => {
    console.log("Searching:", query);
  }, 500);

  return () => clearTimeout(timer);
}, [query]);
```

Typing:

``` text
r
re
rea
reac
react
```

causes cleanup to cancel the previous timer each time.

Only after the user stops typing for 500ms does the final timer execute.

### clearTimeout vs AbortController

`clearTimeout()`:

> Prevents a scheduled operation from starting.

`AbortController.abort()`:

> Aborts an already-started fetch request from the browser side.

------------------------------------------------------------------------

# 10. useEffect and API Requests

Basic API request:

``` jsx
useEffect(() => {
  fetch("/api/users")
    .then(res => res.json())
    .then(data => setUsers(data));
}, []);
```

## HTTP errors

`fetch()` does not reject merely because the server returns 404 or 500.

You should check:

``` jsx
const response = await fetch("/api/users");

if (!response.ok) {
  throw new Error("Request failed");
}

const data = await response.json();
```

`response.ok` is true for HTTP status codes from 200--299.

------------------------------------------------------------------------

# 11. Race Conditions in API Requests

Consider:

``` jsx
useEffect(() => {
  fetch(`/api/search?q=${query}`)
    .then(res => res.json())
    .then(data => setResults(data));
}, [query]);
```

If the user types:

``` text
r → re → rea
```

three requests may be sent.

The requests can finish in a different order:

``` text
r    ───────────────→ response
re   ───────→ response
rea  ──→ response
```

The old `r` request could finish last and overwrite the newer `rea`
results.

This is a **race condition**.

## AbortController solution

``` jsx
useEffect(() => {
  const controller = new AbortController();

  fetch(`/api/search?q=${query}`, {
    signal: controller.signal
  })
    .then(res => res.json())
    .then(data => setResults(data))
    .catch(error => {
      if (error.name !== "AbortError") {
        console.error(error);
      }
    });

  return () => {
    controller.abort();
  };
}, [query]);
```

Flow:

``` text
query = "r"
    ↓
request 1 starts

query = "re"
    ↓
cleanup → abort request 1
    ↓
request 2 starts

query = "rea"
    ↓
cleanup → abort request 2
    ↓
request 3 starts
```

### Important nuance

Aborting a browser fetch does not guarantee that server-side work
already performed by the server is undone.

------------------------------------------------------------------------

# 12. Stale Closures

A closure can capture values from the render in which it was created.

Example:

``` jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setCount(count + 1);
    }, 1000);

    return () => clearInterval(timer);
  }, []);

  return <h1>{count}</h1>;
}
```

The effect runs once and captures:

``` text
count = 0
```

The interval repeatedly executes:

``` jsx
setCount(0 + 1);
```

So the counter behaves like:

``` text
0 → 1 → 1 → 1 → 1...
```

This is a **stale closure** problem.

## Fix: Functional updater

``` jsx
setCount(prev => prev + 1);
```

Now React provides the latest state value:

``` text
0 → 1 → 2 → 3 → 4...
```

## Alternative

You can include `count` in the dependencies:

``` jsx
useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1);
  }, 1000);

  return () => clearInterval(timer);
}, [count]);
```

Each count change:

``` text
cleanup old interval
       ↓
create new interval
       ↓
new interval captures latest count
```

For a simple counter interval, the functional updater is generally
cleaner because the interval itself does not need to be recreated on
every count change.

------------------------------------------------------------------------

# 13. useRef

`useRef` returns a persistent mutable object:

``` jsx
const ref = useRef(initialValue);
```

Conceptually:

``` js
{
  current: initialValue
}
```

Changing `.current` does **not** cause a re-render.

## Example

``` jsx
const countRef = useRef(0);

countRef.current++;
console.log(countRef.current);
```

You can change it repeatedly:

``` text
1
2
3
```

without triggering a render.

## Main uses

### 1. DOM access

``` jsx
const inputRef = useRef(null);

return <input ref={inputRef} />;
```

After the input is mounted:

``` jsx
inputRef.current.focus();
```

React assigns the DOM node to `.current`.

### 2. Persistent mutable values

Useful for:

-   Timer IDs
-   Previous values
-   DOM nodes
-   Mutable values that do not need to affect rendering

------------------------------------------------------------------------

# 14. State vs Ref

  -----------------------------------------------------------------------
  State                               Ref
  ----------------------------------- -----------------------------------
  Used for data that affects UI       Used for persistent mutable values

  Updating it schedules a render      Changing `.current` does not
                                      schedule a render

  React manages state updates         You directly mutate `.current`

  UI should normally derive from it   UI does not automatically react to
                                      changes
  -----------------------------------------------------------------------

Rule of thumb:

> If changing the value should update the UI, use state. If it must
> persist but does not need to trigger UI updates, a ref may be
> appropriate.

------------------------------------------------------------------------

# 15. Controlled vs Uncontrolled Inputs

## Controlled input

React state is the source of truth.

``` jsx
const [email, setEmail] = useState("");

<input
  value={email}
  onChange={e => setEmail(e.target.value)}
/>
```

Every keystroke updates state and causes a render.

Useful when UI needs to react immediately:

-   Validation
-   Character count
-   Enable/disable buttons
-   Conditional UI
-   Formatting

## Uncontrolled input

The DOM is the source of truth.

``` jsx
const emailRef = useRef(null);

<input ref={emailRef} />
```

Read the value when needed:

``` jsx
emailRef.current.value
```

The DOM value still changes when the user types, but React state does
not track it and changing the ref does not trigger rendering.

Good when you mainly need the value at submission time.

------------------------------------------------------------------------

# 16. useReducer

Basic syntax:

``` jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

A reducer is:

``` jsx
function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return state + 1;

    case "decrement":
      return state - 1;

    case "reset":
      return 0;

    default:
      return state;
  }
}
```

Dispatch sends an action:

``` jsx
dispatch({ type: "increment" });
```

Actions describe what happened.

``` jsx
dispatch({
  type: "increment",
  amount: 5
});
```

## Object state

``` jsx
function reducer(state, action) {
  switch (action.type) {
    case "setName":
      return {
        ...state,
        name: action.name
      };

    default:
      return state;
  }
}
```

The spread preserves the other fields.

## When to use useReducer

Do not choose it merely because there are many state variables.

Use it when:

-   State transitions are complex.
-   Multiple pieces of state are related.
-   There are many possible transitions.
-   Several state values change together.
-   Centralizing transition logic improves clarity.

For simple independent state:

``` jsx
const [name, setName] = useState("");
const [email, setEmail] = useState("");
const [loading, setLoading] = useState(false);
```

may be clearer.

### useReducer vs Redux

`useReducer` is built into React.

Redux is an external state-management library that also uses concepts
such as:

``` text
state
actions
reducers
dispatch
```

------------------------------------------------------------------------

# 17. useMemo

`useMemo` memoizes a calculated value/reference.

``` jsx
const result = useMemo(() => {
  return expensiveCalculation(items);
}, [items]);
```

If `items` has not changed according to dependency comparison, React can
reuse the memoized value.

If `items` changes, the calculation runs again.

### Important

`useMemo` does **not** prevent the component from re-rendering.

It memoizes the **value**, not the component render.

------------------------------------------------------------------------

# 18. useCallback

`useCallback` memoizes a function reference.

``` jsx
const handleClick = useCallback(() => {
  console.log("clicked");
}, []);
```

If dependencies don't change, React can preserve the same function
reference between renders.

### useMemo vs useCallback

``` text
useMemo     → memoizes a value/reference
useCallback → memoizes a function reference
```

Think:

``` jsx
const value = useMemo(() => calculate(), [deps]);

const functionRef = useCallback(() => {
  // logic
}, [deps]);
```

`useCallback` does not make the function itself inherently faster. Its
main benefit is preserving the function reference, especially when
passing callbacks to memoized children.

------------------------------------------------------------------------

# 19. React.memo

`React.memo` can skip rendering a component when its props are
unchanged.

``` jsx
const Child = React.memo(function Child({ name }) {
  console.log("Child rendered");

  return <h1>{name}</h1>;
});
```

If the parent renders again but `name` is unchanged, React may skip
rendering `Child`.

### Important

`React.memo` does not mean:

> "This component never renders again."

It means React can bail out when its props are considered unchanged.

By default it performs shallow comparison of props.

------------------------------------------------------------------------

# 20. Object and Function References with React.memo

Consider:

``` jsx
const user = {
  name: "Ugyen"
};

<Child user={user} />
```

If `user` is created inside the parent on every render:

``` jsx
function Parent() {
  const user = {
    name: "Ugyen"
  };

  return <Child user={user} />;
}
```

each render creates a new object:

``` text
previous user !== new user
```

Even though the contents are the same.

`React.memo` sees a changed reference and may render the child again.

## useMemo can stabilize the reference

``` jsx
const user = useMemo(() => ({
  name: "Ugyen"
}), []);
```

Now the same object reference can be reused while dependencies remain
unchanged.

------------------------------------------------------------------------

# 21. React.memo + useCallback

A common pattern:

``` jsx
const Child = React.memo(({ onClick }) => {
  console.log("Child rendered");

  return <button onClick={onClick}>Click</button>;
});

function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>

      <Child onClick={handleClick} />
    </>
  );
}
```

Without `useCallback`:

``` jsx
const handleClick = () => {};
```

a new function reference is created on every parent render.

Therefore:

``` text
Parent renders
    ↓
new handleClick reference
    ↓
Child prop changed
    ↓
React.memo cannot bail out
```

With `useCallback`, the function reference can remain stable.

### Mental model

``` text
React.memo   → memoizes/bails out component based on props
useMemo      → memoizes a value/reference
useCallback  → memoizes a function reference
```

Don't use memoization everywhere automatically. It adds complexity and
has its own overhead.

------------------------------------------------------------------------

# 22. Context

Context allows data to be made available to descendants without manually
passing props through every intermediate component.

## Prop drilling

Without context:

``` text
App
 ↓ user
Parent
 ↓ user
Child
 ↓ user
DeepChild
```

If `Parent` and `Child` don't need `user`, passing it through them is
prop drilling.

## createContext

``` jsx
const ThemeContext = createContext();
```

The default value can be anything:

``` jsx
const ThemeContext = createContext(null);
```

or:

``` jsx
const ThemeContext = createContext("light");
```

The default is used when a component has no matching Provider above it.

## Provider

``` jsx
<ThemeContext.Provider value="dark">
  <App />
</ThemeContext.Provider>
```

## Consumer

``` jsx
const theme = useContext(ThemeContext);
```

`useContext` consumes an existing context.

`createContext` creates the context.

------------------------------------------------------------------------

# 23. Multiple Contexts

Each context has its own Provider.

``` jsx
<AuthContext.Provider value={authValue}>
  <ThemeContext.Provider value={themeValue}>
    <App />
  </ThemeContext.Provider>
</AuthContext.Provider>
```

Components inside the corresponding provider subtree can consume that
context.

A Provider does not have to wrap the entire application. It can wrap any
subtree that needs the context.

------------------------------------------------------------------------

# 24. Context Value and Object References

This can create a new object every render:

``` jsx
<ThemeContext.Provider value={{ theme, setTheme }}>
  {children}
</ThemeContext.Provider>
```

Even if `theme` hasn't changed, the object itself is new.

You can stabilize it:

``` jsx
const value = useMemo(
  () => ({ theme, setTheme }),
  [theme]
);

<ThemeContext.Provider value={value}>
  {children}
</ThemeContext.Provider>
```

Context consumers update when the consumed context value changes.

------------------------------------------------------------------------

# 25. Custom Hooks

A custom Hook is a JavaScript function that:

-   Starts with `use`
-   Can use other Hooks
-   Reuses stateful logic
-   Does not primarily exist to reuse UI

Example:

``` jsx
function useCounter() {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount(prev => prev + 1);
  };

  return {
    count,
    increment
  };
}
```

Use it:

``` jsx
function Counter() {
  const { count, increment } = useCounter();

  return (
    <>
      <p>{count}</p>
      <button onClick={increment}>+</button>
    </>
  );
}
```

### Important

If two components call:

``` jsx
useCounter();
```

they get **separate state**.

Custom Hooks share logic, not automatically the same state.

------------------------------------------------------------------------

# 26. useAuth Custom Hook

A common context helper:

``` jsx
function useAuth() {
  return useContext(AuthContext);
}
```

Then components can simply write:

``` jsx
const { user, logout } = useAuth();
```

instead of repeatedly writing:

``` jsx
useContext(AuthContext);
```

------------------------------------------------------------------------

# 27. Component Composition and children

`children` is a special prop containing whatever is placed between a
component's opening and closing tags.

Example:

``` jsx
<Card>
  <h1>Hello</h1>
</Card>
```

Inside `Card`:

``` jsx
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}
```

Here:

``` jsx
children === <h1>Hello</h1>
```

Multiple children are possible:

``` jsx
<Card>
  <h1>Hello</h1>
  <p>Welcome</p>
</Card>
```

Composition means building larger UI by combining components.

The outer component provides structure/behavior, while the parent
supplies inner content through `children`.

------------------------------------------------------------------------

# 28. AuthProvider

A common pattern is to create an authentication context and provider.

``` jsx
import {
  createContext,
  useEffect,
  useState
} from "react";

export const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function checkAuth() {
      try {
        const response = await fetch("/api/me");

        if (response.ok) {
          const data = await response.json();
          setUser(data);
        }
      } catch (error) {
        console.error(error);
      } finally {
        setLoading(false);
      }
    }

    checkAuth();
  }, []);

  if (loading) {
    return <p>Checking authentication...</p>;
  }

  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
}
```

Use it:

``` jsx
<AuthProvider>
  <App />
</AuthProvider>
```

Any component inside this subtree can consume the context.

------------------------------------------------------------------------

# 29. Authentication vs AuthProvider

These are different concepts.

### Authentication

The backend/security process of determining whether a user is
authenticated.

For example:

``` text
POST /api/login
        ↓
backend verifies credentials
        ↓
session/cookie established
```

### AuthProvider

A React-side state-management mechanism that makes authentication state
available to the UI.

For example:

``` text
backend says user is authenticated
        ↓
AuthProvider
        ↓
user = {...}
        ↓
components can access user
```

AuthProvider itself does not prove identity.

------------------------------------------------------------------------

# 30. Why `useState(false)` Alone Doesn't Maintain Login

This is not enough:

``` jsx
const [isLoggedIn, setIsLoggedIn] = useState(false);
```

React state lives in memory.

After a full page refresh:

``` text
React application restarts
        ↓
state initialized again
        ↓
isLoggedIn = false
```

Authentication therefore needs a persistent mechanism.

------------------------------------------------------------------------

# 31. Auth Persistence with a Backend

A common architecture uses a server-side session or authentication
cookie.

Login:

``` text
Login form
    ↓
POST /api/login
    ↓
Backend verifies credentials
    ↓
Session/cookie established
    ↓
AuthProvider setUser(user)
```

After a refresh:

``` text
React starts
    ↓
user = null
loading = true
    ↓
GET /api/me
    ↓
Browser sends applicable authentication cookie
    ↓
Backend verifies authentication
    ↓
200 → user data
401 → unauthenticated
    ↓
loading = false
```

The endpoint does not have to literally be `/api/me`. It could be
`/api/session` or another endpoint serving the same purpose.

------------------------------------------------------------------------

# 32. Why Auth Loading State Matters

Initial state:

``` jsx
const [user, setUser] = useState(null);
const [loading, setLoading] = useState(true);
```

Without `loading`, the app may see:

``` text
user = null
```

and immediately redirect to `/login` before the `/api/me` request
finishes.

This creates an authentication flicker.

Correct mental model:

``` text
user = null
loading = true
        ↓
auth check
        ↓
authenticated?
   ↙          ↘
yes            no
 ↓              ↓
user=data       user=null
        ↓
loading=false
```

`loading` here represents **authentication initialization/checking**,
not necessarily a login button request.

For a login request itself, a separate state such as:

``` jsx
const [isLoggingIn, setIsLoggingIn] = useState(false);
```

can be used.

------------------------------------------------------------------------

# 33. HttpOnly Cookies

An `HttpOnly` cookie cannot be read by JavaScript:

``` js
document.cookie
```

will not expose the HttpOnly cookie value.

However, the browser can still send the cookie to the appropriate
backend request when cookie rules allow it.

This gives the frontend a flow like:

``` text
Browser
  │
  │ request /api/me
  │ + authentication cookie
  ↓
Backend
  │
  │ verifies session
  ↓
User data / 401
```

The frontend does not need to know the session token itself.

Common cookie security attributes include:

``` text
HttpOnly
Secure
SameSite
```

For cross-origin requests, cookie behavior also depends on credentials
and CORS configuration.

------------------------------------------------------------------------

# 34. localStorage and Authentication

You can store UI-related information in localStorage:

``` js
localStorage.setItem("theme", "dark");
```

But:

``` js
localStorage.setItem("isLoggedIn", "true");
```

is not proof that the user is authenticated.

The frontend can modify it.

Sensitive authentication tokens stored in localStorage are also
accessible to JavaScript and therefore can be exposed by XSS.

A server-validated HttpOnly cookie can reduce exposure of authentication
credentials to JavaScript.

------------------------------------------------------------------------

# 35. Logout

A proper logout generally involves both:

1.  Invalidate/end the backend session.
2.  Clear the React-side user state.

Conceptually:

``` text
logout
  ↓
POST /api/logout
  ↓
backend invalidates session
  ↓
setUser(null)
```

Simply doing:

``` jsx
setUser(null);
```

changes the UI state but does not necessarily invalidate the backend
session.

------------------------------------------------------------------------

# 36. React Router

With modern React Router:

``` jsx
const router = createBrowserRouter([
  {
    path: "/",
    element: <Home />
  },
  {
    path: "/about",
    element: <About />
  }
]);
```

Then:

``` jsx
<RouterProvider router={router} />
```

------------------------------------------------------------------------

# 37. Nested Routes

Example:

``` jsx
const router = createBrowserRouter([
  {
    path: "/products",
    element: <Products />,
    children: [
      {
        path: ":id",
        element: <ProductDetails />
      }
    ]
  }
]);
```

`Products` must render an `<Outlet />`:

``` jsx
import { Outlet } from "react-router-dom";

function Products() {
  return (
    <div>
      <h1>Products</h1>
      <Outlet />
    </div>
  );
}
```

For:

``` text
/products/42
```

the structure is:

``` text
Products
   ↓
Outlet
   ↓
ProductDetails
```

`ProductDetails` appears wherever `<Outlet />` is placed.

------------------------------------------------------------------------

# 38. Dynamic Route Parameters

This:

``` jsx
path: ":id"
```

is a dynamic route parameter.

It can match:

``` text
/products/42
/products/15
/products/abc
```

Get the parameter:

``` jsx
import { useParams } from "react-router-dom";

function ProductDetails() {
  const { id } = useParams();

  console.log(id);
}
```

For:

``` text
/products/42
```

the result is:

``` js
{
  id: "42"
}
```

The URL parameter is normally a string.

------------------------------------------------------------------------

# 39. Link vs useNavigate

## Link

Declarative user-driven navigation:

``` jsx
<Link to="/about">About</Link>
```

Typical navigation UI:

``` jsx
<nav>
  <Link to="/">Home</Link>
  <Link to="/products">Products</Link>
  <Link to="/about">About</Link>
</nav>
```

## useNavigate

Programmatic navigation:

``` jsx
const navigate = useNavigate();

function handleLogin() {
  // login logic
  navigate("/dashboard");
}
```

Can also navigate backward:

``` jsx
navigate(-1);
```

### Interview answer

> `Link` is used for declarative user-driven navigation, while
> `useNavigate` is used for programmatic navigation when application
> logic needs to trigger navigation.

------------------------------------------------------------------------

# 40. Route Loaders

With `createBrowserRouter`, a route can have a loader:

``` jsx
{
  path: "/products/:id",
  element: <ProductDetails />,
  loader: async ({ params }) => {
    const response = await fetch(
      `/api/products/${params.id}`
    );

    return response.json();
  }
}
```

Component:

``` jsx
function ProductDetails() {
  const product = useLoaderData();

  return <h1>{product.name}</h1>;
}
```

Flow:

``` text
Navigation
    ↓
Route matching
    ↓
Loader runs
    ↓
Data loads
    ↓
Route renders
    ↓
useLoaderData()
```

A loader is useful for **route-level data loading**.

It does not replace `useEffect` for every kind of side effect.

------------------------------------------------------------------------

# 41. Protected Routes with createBrowserRouter

A route can use a loader to redirect unauthenticated users:

``` jsx
{
  path: "/dashboard",
  element: <Dashboard />,
  loader: () => {
    const isLoggedIn = checkAuth();

    if (!isLoggedIn) {
      throw redirect("/login");
    }

    return null;
  }
}
```

Conceptually:

``` text
/dashboard
    ↓
check authentication
    ↓
authenticated?
   ↙       ↘
 yes        no
 ↓          ↓
Dashboard   /login
```

### Important security point

A frontend route guard is **not backend security**.

The backend must independently authenticate and authorize protected API
requests.

------------------------------------------------------------------------

# 42. AuthProvider + Protected Routing

With asynchronous auth initialization, don't redirect simply because:

``` jsx
user === null
```

Initially:

``` text
user = null
loading = true
```

The application should wait until the authentication check completes.

Only then decide:

``` text
loading?
  ↓
wait

loading = false
  ↓
user exists?
 ↙       ↘
yes       no
 ↓         ↓
dashboard  login
```

------------------------------------------------------------------------

# 43. CSR vs SSR

## Client-Side Rendering (CSR)

A simplified CSR flow:

``` text
Browser requests page
        ↓
Server sends minimal HTML
        ↓
Browser downloads JavaScript
        ↓
React executes
        ↓
UI is built on the client
```

## Server-Side Rendering (SSR)

Simplified:

``` text
Browser requests page
        ↓
Server renders HTML
        ↓
Browser receives ready HTML
        ↓
Browser displays content
        ↓
JavaScript loads
        ↓
React hydrates the HTML
```

### Important distinction

CSR vs SSR describes **where/how the initial UI is rendered**.

Client-side routing describes **how navigation happens after the
application is running**.

Modern SSR applications can still use client-side routing and hydration.

------------------------------------------------------------------------

# 44. Lazy Loading

`React.lazy()` allows component code to be loaded only when needed.

``` jsx
const Admin = React.lazy(() => import("./Admin"));
```

Wrap it in `Suspense`:

``` jsx
<Suspense fallback={<p>Loading...</p>}>
  <Admin />
</Suspense>
```

Flow:

``` text
Need Admin component
       ↓
Download Admin chunk
       ↓
Show Suspense fallback
       ↓
Component loaded
       ↓
Render Admin
```

This enables code splitting and can reduce the amount of JavaScript
loaded initially.

------------------------------------------------------------------------

# 45. Error Boundaries

An Error Boundary catches certain errors in the rendering tree and shows
fallback UI instead of allowing the entire affected UI subtree to crash.

Conceptually:

``` text
Error Boundary
      │
      ├── Header
      ├── Sidebar
      └── Main
           ↓
       Component
           ↓
         error
           ↓
    fallback UI
```

Error boundaries are useful for rendering-related failures.

They are **not a replacement for handling API errors**.

For example, a failed fetch should generally be handled with:

``` jsx
try {
  const response = await fetch("/api/data");

  if (!response.ok) {
    throw new Error("Request failed");
  }

  const data = await response.json();
} catch (error) {
  setError(error.message);
}
```

------------------------------------------------------------------------

# 46. Forms

## Controlled form input

``` jsx
const [email, setEmail] = useState("");

<input
  value={email}
  onChange={e => setEmail(e.target.value)}
/>
```

`onChange` updates the state.

Without it:

``` jsx
<input value={email} />
```

the input's displayed value remains controlled by the state and the user
cannot update it normally.

------------------------------------------------------------------------

# 47. Form Submission

Use an event handler for submission:

``` jsx
function handleSubmit(e) {
  e.preventDefault();

  submitToAPI(email);
}
```

``` jsx
<form onSubmit={handleSubmit}>
  <input
    value={email}
    onChange={e => setEmail(e.target.value)}
  />

  <button type="submit">
    Submit
  </button>
</form>
```

`e.preventDefault()` prevents the browser's normal form submission
behavior, such as navigating/reloading the page.

------------------------------------------------------------------------

# 48. Form Validation

Guard clauses can stop invalid submission:

``` jsx
function handleSubmit(e) {
  e.preventDefault();

  if (!email.includes("@")) {
    console.log("Invalid email");
    return;
  }

  submitToAPI(email);
}
```

The `return` stops the remaining submission logic.

------------------------------------------------------------------------

# 49. Don't Use useEffect for Form Submission

Avoid:

``` jsx
useEffect(() => {
  submitToAPI(email);
}, [email]);
```

This would submit whenever the email state changes.

Instead:

``` jsx
function handleSubmit(e) {
  e.preventDefault();
  submitToAPI(email);
}
```

An explicit user action belongs in an event handler.

`useEffect` is for synchronization with external systems, not for
replacing event handlers.

------------------------------------------------------------------------

# 50. Lifting State Up

When sibling components need to share state, move the state to their
nearest common ancestor.

Example:

``` jsx
function Parent() {
  const [name, setName] = useState("");

  return (
    <>
      <NameInput
        name={name}
        setName={setName}
      />

      <NameDisplay name={name} />
    </>
  );
}
```

Data flows down:

``` text
Parent state
   ↓
props
   ↓
children
```

Events/actions can flow upward through callbacks:

``` text
Child event
   ↓
callback prop
   ↓
Parent updates state
   ↓
new props
   ↓
children update
```

------------------------------------------------------------------------

# 51. Keys in Lists

Keys provide stable identity for list items during reconciliation.

Prefer:

``` jsx
users.map(user => (
  <User key={user.id} user={user} />
))
```

over:

``` jsx
users.map((user, index) => (
  <User key={index} user={user} />
))
```

when the list can change order, insert, or delete items.

## Why IDs are better

Suppose:

``` text
1
2
3
```

Delete `2`:

``` text
1
3
```

Stable keys allow React to understand:

``` text
1 → same item
3 → same item
2 → removed
```

This helps preserve the correct component/DOM identity and state.

### Key is special

`key` is used by React and is not available as:

``` jsx
props.key
```

If a component needs the ID, pass it separately:

``` jsx
<User
  key={user.id}
  id={user.id}
/>
```

------------------------------------------------------------------------

# 52. Key vs React.memo

They solve different problems.

### Key

Controls **identity during reconciliation**.

``` jsx
<User key={user.id} />
```

### React.memo

Can prevent a component from rendering when its props are unchanged.

``` jsx
const User = React.memo(...)
```

Mental model:

``` text
key        → "Which item is this?"
React.memo → "Do I need to render this component again?"
```

------------------------------------------------------------------------

# 53. Architecture / Separation of Concerns

A practical React project can be organized like:

``` text
src/
├── components/
├── hooks/
├── context/
├── services/
├── pages/
└── App.jsx
```

Typical responsibilities:

### components/

Reusable UI components.

``` text
Button
Card
Navbar
RestaurantCard
```

### hooks/

Reusable stateful logic.

``` text
useAuth
useFetch
useDebounce
useCounter
```

### context/

Shared application-level state/context.

``` text
AuthContext
ThemeContext
```

### services/

API/backend communication.

``` text
authService
productService
restaurantService
```

### pages/

Route-level UI.

``` text
Home
Login
Dashboard
ProductDetails
```

### App.jsx

Application composition and setup.

------------------------------------------------------------------------

# 54. Separation of Concerns

A useful mental model:

``` text
Component
    ↓
UI / presentation

Custom Hook
    ↓
Reusable stateful logic

Context
    ↓
Shared values/state

Service
    ↓
API/backend communication

Page
    ↓
Route-level composition
```

The goal is not to create files just for the sake of creating files.
Separate code when the separation improves clarity, reuse, testing, or
maintainability.

------------------------------------------------------------------------

# 55. Important Interview Mental Models

## State snapshot

``` text
Each render has its own state snapshot.
```

## Functional updater

``` text
setState(prev => newValue)
```

Use when the new state depends on previous/latest state.

## Stale closure

``` text
Callback created during an old render
        ↓
captures old value
        ↓
later executes using old value
```

## Effect cleanup

``` text
dependency changes
        ↓
cleanup old effect
        ↓
run new effect
```

## AbortController

``` text
obsolete request
        ↓
abort it
        ↓
prevent stale request from completing normally
```

## Ref

``` text
persistent mutable value
        ↓
doesn't trigger render
```

## useMemo

``` text
memoized value/reference
```

## useCallback

``` text
memoized function reference
```

## React.memo

``` text
possible component render bailout
```

## Context

``` text
share values with descendants
without prop drilling
```

## useReducer

``` text
centralize complex state-transition logic
```

## Key

``` text
stable identity for list items
```

## Link

``` text
declarative navigation
```

## useNavigate

``` text
programmatic navigation
```

## Loader

``` text
route-level data loading
```

## children

``` text
content passed between component opening/closing tags
```

------------------------------------------------------------------------

# 56. Quick Comparison Table

  -----------------------------------------------------------------------
  Concept                             Main purpose
  ----------------------------------- -----------------------------------
  `useState`                          Store component state

  `useEffect`                         Synchronize with external systems

  `useRef`                            Persist mutable value / access DOM
                                      without rerender

  `useReducer`                        Centralize complex state
                                      transitions

  `useMemo`                           Memoize a calculated
                                      value/reference

  `useCallback`                       Memoize a function reference

  `useContext`                        Consume context

  `createContext`                     Create context

  Custom Hook                         Reuse stateful logic

  `React.memo`                        Skip unnecessary child renders when
                                      props are unchanged

  `key`                               Give list items stable identity

  `Link`                              Declarative navigation

  `useNavigate`                       Programmatic navigation

  `Outlet`                            Render nested route element

  `useParams`                         Read dynamic route parameters

  `loader`                            Load data for a route

  `Suspense`                          Show fallback while supported
                                      async/lazy content loads

  Error Boundary                      Show fallback for rendering errors
                                      in a child tree
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 57. High-Value Interview Traps

### `useEffect([])` does not mean "never runs"

It runs after the initial mount, with development Strict Mode
potentially performing extra setup/cleanup behavior.

### `setState` does not immediately change the current variable

``` jsx
setCount(1);
console.log(count);
```

The current render's `count` remains unchanged.

### `useRef` changes don't cause rendering

``` jsx
ref.current = 10;
```

does not cause React to render again.

### `fetch()` doesn't reject on 404/500

Check:

``` jsx
if (!response.ok) {
  throw new Error("Request failed");
}
```

### `React.memo` doesn't prevent all renders

It can bail out when props are unchanged.

### `useMemo` doesn't memoize a component

It memoizes a value/reference.

### `useCallback` doesn't make a function automatically faster

It preserves the function reference when dependencies don't change.

### Context isn't global state by itself

Context makes a value available to descendants. The actual state can
still be managed with `useState`, `useReducer`, or another mechanism.

### Frontend route protection isn't backend security

The backend must authenticate and authorize protected API requests.

### localStorage authentication flags aren't trustworthy authentication

The client can modify them. Authentication should be validated by the
backend.

### `key` isn't available as `props.key`

React uses it internally for identity.

### `children` isn't automatically a component

It is a special prop containing whatever content is passed between the
component's tags.

------------------------------------------------------------------------

# 58. Interview Answer Templates

## What is a custom Hook?

> A custom Hook is a JavaScript function whose name starts with `use`
> and that can use other Hooks to encapsulate and reuse stateful logic
> between components. It shares logic, not component state.

## What is `useEffect`?

> `useEffect` lets a component synchronize with external systems such as
> APIs, timers, subscriptions, event listeners, or browser APIs. React
> runs the effect after commit, and the cleanup runs before the effect
> reruns when dependencies change and when the component unmounts.

## What is a stale closure?

> A stale closure occurs when an asynchronous callback captures a value
> from an older render and later uses that outdated value. Functional
> state updates can avoid this when the new state depends on the
> previous state.

## Why use `useReducer`?

> `useReducer` is useful when state transitions are complex, related, or
> numerous. It centralizes transition logic in a reducer and makes state
> changes explicit through actions.

## Difference between useMemo and useCallback?

> `useMemo` memoizes a calculated value or reference, while
> `useCallback` memoizes a function reference.

## Why use React.memo?

> `React.memo` can prevent a component from rendering when its props
> have not changed according to its comparison. It is a performance
> optimization, not a guarantee that the component never renders.

## Why use AbortController with search?

> Search requests can finish out of order, allowing an older response to
> overwrite a newer result. Aborting the previous request during effect
> cleanup prevents obsolete requests from completing normally.

## What is Context?

> Context allows data to be consumed by components in a provider's
> subtree without manually passing that data through intermediate
> components.

## What is lifting state up?

> Lifting state up means moving shared state to the nearest common
> parent of components that need it, then passing data and callbacks
> down through props.

## What is a dynamic route?

> A dynamic route contains a parameter such as `:id`. React Router
> extracts the value with `useParams()`.

## Link vs useNavigate?

> `Link` is declarative navigation intended for navigation UI, while
> `useNavigate` provides programmatic navigation when application logic
> needs to change the route.

------------------------------------------------------------------------

# 59. Final React Mental Model

When reasoning about a React problem, ask:

``` text
1. What causes the component to render?
       ↓
2. What state/props/context does this render see?
       ↓
3. Is some callback closing over an old render?
       ↓
4. Is an effect synchronizing with an external system?
       ↓
5. Does the effect need cleanup?
       ↓
6. Is state being updated immutably?
       ↓
7. Should this value be state or a ref?
       ↓
8. Is state transition logic complex enough for useReducer?
       ↓
9. Are unnecessary renders caused by changing references?
       ↓
10. Is memoization actually needed?
       ↓
11. For routing: what route matched, what params exist,
    and where is the Outlet?
       ↓
12. For auth: has the backend actually verified authentication?
```

This mental model is more useful in interviews than memorizing isolated
Hook definitions.
