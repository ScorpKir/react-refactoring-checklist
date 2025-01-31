# React Refactoring Checklist

## 1. Component Structure & Organization

- [ ] Break down large components into smaller, reusable ones
 ```jsx
    // Before
    function LargeComponent() {
      return (
        <div>
          <header>...</header>
          <main>...</main>
          <footer>...</footer>
        </div>
      );
    }
  
    // After
    function Header() { ... }
    function Main() { ... }
    function Footer() { ... }
    function LargeComponent() {
      return (
        <div>
          <Header />
          <Main />
          <Footer />
        </div>
      );
    }
  ```
  - [ ] Keep components focused on a single responsibility
  ```jsx
      // Before
    function UserProfile({ user, posts }) {
      return (
        <div>
          <h1>{user.name}</h1>
          <p>{user.bio}</p>
          <ul>
            {posts.map(post => <li key={post.id}>{post.title}</li>)}
          </ul>
        </div>
      );
    }
    
    // After
    function UserInfo({ user }) {
      return (
        <div>
          <h1>{user.name}</h1>
          <p>{user.bio}</p>
        </div>
      );
    }
    
    function UserPosts({ posts }) {
      return (
        <ul>
          {posts.map(post => <li key={post.id}>{post.title}</li>)}
        </ul>
      );
    }
    
    function UserProfile({ user, posts }) {
      return (
        <div>
          <UserInfo user={user} />
          <UserPosts posts={posts} />
        </div>
      );
    }
  ```
- [ ] Move reusable logic to custom hooks
```jsx
  // Before
  function Component() {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);
  
    useEffect(() => {
      fetchData()
        .then(data => setData(data))
        .catch(error => setError(error))
        .finally(() => setLoading(false));
    }, []);
  
    // ... rest of the component
  }
  å
  // After
  function useFetch(url) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);
  
    useEffect(() => {
      fetch(url)
        .then(response => response.json())
        .then(data => setData(data))
        .catch(error => setError(error))
        .finally(() => setLoading(false));
    }, [url]);
  
    return { data, loading, error };
  }
  
  function Component() {
    const { data, loading, error } = useFetch('https://api.example.com/data');
    // ... rest of the component
  }
```

## 2. State Management

- [ ] Lift state only when necessary (avoid unnecessary prop drilling)
- [ ] Use `useReducer` for complex state logic instead of multiple `useState`
- [ ] Memoize expensive computations with `useMemo`
- [ ] Prevent re-renders with `React.memo`

## 3. JSX & Render Optimization

- [ ] Extract long JSX blocks into smaller components
- [ ] Don't use indexes as keys in lists to avoid rendering issues
- [ ] Avoid inline functions in JSX when possible (move them outside render)

## 4. API Calls & Side Effects

- [ ] Encapsulate API calls inside custom hooks (e.g., `useFetch`)
- [ ] Always clean up effects inside `useEffect` to prevent memory leaks

## 5. Props & Context

- [ ] Avoid prop drilling; use `context` or state management libraries (Redux or other) instead
- [ ] Define `defaultProps` or handle missing props gracefully
- [ ] Prefer TypeScript or PropTypes for prop validation

## 6. Performance Improvements

- [ ] Use lazy loading (`React.lazy` + `Suspense`) for large components
- [ ] Optimize re-renders by passing only necessary props
- [ ] Debounce input handlers to prevent excessive state updates

## 7. Code Cleanliness & Readability

- [ ] Follow a consistent naming convention for variables, functions, and components
- [ ] Keep files organized in a scalable folder structure (`components/`, `hooks/`, `utils/`)
- [ ] Always wrap external libraries to keep your codebase flexible and maintainable
- [ ] Use Higher-Order Components (HOC) for reusability
