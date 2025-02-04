# Чеклист рефакторинга React-проектов

## 1. Организация и структура компонента

- [ ] Разбивайте большие компоненты на маленькие и переиспользуемые
 ```jsx
    // До
    function LargeComponent() {
      return (
        <div>
          <header>...</header>
          <main>...</main>
          <footer>...</footer>
        </div>
      );
    }
  
    // После
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
  - [ ] Каждый компонент должен иметь только одну зону ответственности
  ```jsx
      // До
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
    
    // После
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
- [ ] Перемещайте переиспользуемую логику в хуки
```jsx
  // До
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
  
    // ... остальная часть компонента
  }
  å
  // После
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
    // ... остальная часть компонента
  }
```

## 2. Управление состояниями
- [ ] Используйте `useReducer` для сложной логики состояний вместо нескольких `useState`
```jsx
// До
function Counter() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);

  const increment = () => setCount(count + step);
  const decrement = () => setCount(count - step);
  const updateStep = (newStep) => setStep(newStep);

  // ... остальная часть компонента
}

// После
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + state.step };
    case 'DECREMENT':
      return { ...state, count: state.count - state.step };
    case 'SET_STEP':
      return { ...state, step: action.payload };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0, step: 1 });

  const increment = () => dispatch({ type: 'INCREMENT' });
  const decrement = () => dispatch({ type: 'DECREMENT' });
  const updateStep = (newStep) => dispatch({ type: 'SET_STEP', payload: newStep });

  // ... остальная часть компонента
}
```
- [ ] Кешируйте дорогостоящие вычисления с помощью `useMemo`
```jsx
// До
function ExpensiveComponent({ data }) {
  const expensiveResult = expensiveCalculation(data);
  return <div>{expensiveResult}</div>;
}

// После
function ExpensiveComponent({ data }) {
  const expensiveResult = useMemo(() => expensiveCalculation(data), [data]);
  return <div>{expensiveResult}</div>;
}
```
- [ ] Предотвращайте ре-рендеры компонента с помощью `React.memo` (Используйте это с осторожностью. Больше об этом ниже.)
```jsx
// До
function MyComponent({ value }) {
  return <div>{value}</div>;
}

// После
const MyComponent = React.memo(function MyComponent({ value }) {
  return <div>{value}</div>;
});
```

## 3. JSX & Render Optimization

- [ ] Extract long JSX blocks into smaller components
```jsx
// Before
function LongComponent() {
  return (
    <div>
      <header>
        <h1>Title</h1>
        <nav>
          <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/about">About</a></li>
            <li><a href="/contact">Contact</a></li>
          </ul>
        </nav>
      </header>
      <main>
        {/* ... */}
      </main>
      <footer>
        {/* ... */}
      </footer>
    </div>
  );
}

// After
function Header() {
  return (
    <header>
      <h1>Title</h1>
      <Nav />
    </header>
  );
}

function Nav() {
  return (
    <nav>
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
    </nav>
  );
}

function Main() {
  return <main>{/* ... */}</main>;
}

function Footer() {
  return <footer>{/* ... */}</footer>;
}

function LongComponent() {
  return (
    <div>
      <Header />
      <Main />
      <Footer />
    </div>
  );
}
```
- [ ] Don't use indexes as keys in lists to avoid rendering issues
```jsx
// Before (using index as key)
function List({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
}

// After (using unique id as key)
function List({ items }) {
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```
- [ ] Avoid inline functions in JSX when possible (move them outside render)
```jsx
// Before
function Component({ onClick }) {
  return (
    <button onClick={() => {
      console.log('Clicked');
      onClick();
    }}>
      Click me
    </button>
  );
}

// After
function Component({ onClick }) {
  const handleClick = () => {
    console.log('Clicked');
    onClick();
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

## 4. API Calls & Side Effects

- [ ] Encapsulate API calls inside custom hooks (e.g., `useFetch`)
```jsx
// Before
function Component() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('https://api.example.com/data')
      .then(response => response.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading...</div>;
  return <div>{data}</div>;
}

// After
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(url)
      .then(response => response.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, [url]);

  return { data, loading };
}

function Component() {
  const { data, loading } = useFetch('https://api.example.com/data');

  if (loading) return <div>Loading...</div>;
  return <div>{data}</div>;
}
```
- [ ] Always clean up effects inside `useEffect` to prevent memory leaks
```jsx
// Before
function Component() {
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('Tick');
    }, 1000);
  }, []);

  return <div>Component with timer</div>;
}

// After
function Component() {
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('Tick');
    }, 1000);

    return () => {
      clearInterval(timer);
    };
  }, []);

  return <div>Component with timer</div>;
}
```

## 5. Props & Context

- [ ] Avoid prop drilling; use `context` or state management libraries (Redux or other) instead
```jsx
// Before (prop drilling)
function GrandParent({ theme }) {
  return <Parent theme={theme} />;
}
function Parent({ theme }) {
  return <Child theme={theme} />;
}
function Child({ theme }) {
  return <div className={theme}>Themed content</div>;
}

// After (using context)
const ThemeContext = React.createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function GrandParent() {
  return (
    <ThemeProvider>
      <Parent />
    </ThemeProvider>
  );
}
function Parent() {
  return <Child />;
}
function Child() {
  const { theme } = useContext(ThemeContext);
  return <div className={theme}>Themed content</div>;
}
```
- [ ] Define `defaultProps` or handle missing props gracefully
```jsx
// Before
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// After
function Greeting({ name = 'Guest' }) {
  return <h1>Hello, {name}!</h1>;
}

// Or using defaultProps
Greeting.defaultProps = {
  name: 'Guest'
};
```
- [ ] Prefer TypeScript or PropTypes for prop validation
```jsx
// Using PropTypes
import PropTypes from 'prop-types';

function User({ name, age }) {
  return <div>{name} is {age} years old</div>;
}

User.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number.isRequired
};

// Using TypeScript
interface UserProps {
  name: string;
  age: number;
}

function User({ name, age }: UserProps) {
  return <div>{name} is {age} years old</div>;
}
```

## 6. Performance Improvements

- [ ] Use lazy loading (`React.lazy` + `Suspense`) for large components
```jsx
// Before
import LargeComponent from './LargeComponent';

function App() {
  return (
    <div>
      <LargeComponent />
    </div>
  );
}

// After
import React, { Suspense } from 'react';
const LargeComponent = React.lazy(() => import('./LargeComponent'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <LargeComponent />
      </Suspense>
    </div>
  );
}
```
- [ ] Optimize re-renders by passing only necessary props
```jsx
// Before
function Parent({ user }) {
  return <Child user={user} />;
}

function Child({ user }) {
  return <div>{user.name}</div>;
}

// After
function Parent({ user }) {
  return <Child name={user.name} />;
}

function Child({ name }) {
  return <div>{name}</div>;
}
```
- [ ] Debounce input handlers to prevent excessive state updates
```jsx
// Before
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');

  const handleChange = (e) => {
    setSearchTerm(e.target.value);
    // Perform search operation
  };

  return <input type="text" onChange={handleChange} />;
}

// After
import { debounce } from 'lodash';

function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');

  const debouncedSearch = useCallback(
    debounce((term) => {
      // Perform search operation
    }, 300),
    []
  );

  const handleChange = (e) => {
    setSearchTerm(e.target.value);
    debouncedSearch(e.target.value);
  };

  return <input type="text" onChange={handleChange} />;
}
```

## 7. Code Cleanliness & Readability

- [ ] Follow a consistent naming convention for variables, functions, and components
```jsx
// Bad naming conventions
const age = 30;
function totalPrice() { ... }
function userProfile() { ... }

// Good naming conventions
const userAge = 30;
function calculateTotalPrice() { ... }
function fetchUserProfile() { ... }
```
- [ ] Keep files organized in a scalable folder structure (`components/`, `hooks/`, `utils/`)
```jsx
//Before
src/
├── App.js
├── Header.js
├── api.js
├── utils.js
├── constants.js

//After
src/
├── components/
│   ├── Header.js
│   └── Footer.js
├── hooks/
│   └── useFetch.js
├── utils/
│   └── formatDate.js
└── App.js
```
- [ ] Always wrap external libraries to keep your codebase flexible and maintainable
```jsx
// Before (direct use of external library)
import axios from 'axios';

function fetchData() {
  return axios.get('https://api.exampleu.com/data');
}

// After (wrapped external library)
import axios from 'axios';

const api = {
  get: (url) => axios.get(url),
  post: (url, data) => axios.post(url, data),
};

function fetchData() {
  return api.get('https://api.example.com/data');
}
```
- [ ] Use Higher-Order Components (HOC) for reusability
```jsx
// HOC that checks if the user is logged in
function withAuth(WrappedComponent) {
  return function WithAuth(props) {
    const isLoggedIn = !!localStorage.getItem('authToken');
    
    if (!isLoggedIn) {
      return <p>Please login to view this content</p>;
    }

    return <WrappedComponent {...props} />;
  };
}

//Usage
const Dashboard = () => {
  return <h1>Welcome to your dashboard Hakim!</h1>;
};

export default withAuth(Dashboard);

```
