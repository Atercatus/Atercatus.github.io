---
title: "Use React Context effectively"
excerpt: "How to use React Context efectively"
last_modified_at: 2019-10-28T12:06:00
---

## Global State and Dispatch

```javascript
const userState = createContext();
const { Provider: UserStateProvider } = userState;
const userDispatch = createContext();
const { Provider: UserDispatchProvider } = userDispatch;
```

- 2개의 state 와 dispatch 를 분리하여 Context를 생성합니다

```javascript
function useUserState() {
  const context = useContext(userState);

  if (!context) {
    throw new Error(`useUserState must be used within a UserProvider`);
  }

  return context;
}

function useUserDispatch() {
  const context = useContext(userDispatch);

  if (!context) {
    throw new Error(`useUserDispatch must be used within a UserProvider`);
  }

  return context;
}
```

```javascript
const reducer = (state, { type, payload }) => {
  switch (type) {
    case "login":
      return payload;
    case "logout":
      return { redirect: true };
    default:
      throw new Error();
  }
};

function UserProvider(props) {
  const [user, dispatch] = useReducer(reducer, {});

  useEffect(() => {
    const refreshSilently = async () => {
      const payload = await reqeustJWTs({ url: routes.refresh });
      dispatch({ type: "login", payload });
    };

    refreshSilently();
  }, []);

  return (
    <UserStateProvider value={user}>
      <UserDispatchProvider value={dispatch}>
        {props.children}
      </UserDispatchProvider>
    </UserStateProvider>
  );
}

export { UserProvider, useUserState, useUserDispatch };
```

- App.js

```javascript
export default function App() {
  return (
    <UserProvider>
      <Router>
        <S.Container>
          <Nav />
          {/* A <Switch> looks through its children <Route>s and
            renders the first one that matches the current URL. */}
          <Switch>
            <Route path={routes.login}>
              <Login />
            </Route>
            <Route path={routes.main}>
              <Main />
            </Route>
          </Switch>
        </S.Container>
      </Router>
    </UserProvider>
  );
}
```

```javascript
export default function Nav() {
  const user = useUserState();
  const dispatch = useUserDispatch();

  return (
    <Container>
      {user.redirect && <Redirect to={routes.login} />}
      <div>
        <span>
          <Link to={routes.main}>main</Link>
        </span>
        {!user.email ? (
          <span>
            <Link to={routes.login}>Login</Link>
          </span>
        ) : (
          <span onClick={() => logout({ user, dispatch })}>Logout</span>
        )}
      </div>
      <span>{user.name}</span>
      <span>{user.email}</span>
    </Container>
  );
}
```
