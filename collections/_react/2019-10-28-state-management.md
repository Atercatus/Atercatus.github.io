---
title: "Application State Management"
excerpt: "React에서의 상태관리를 알아보자"
last_modified_at: 2019-10-28T08:06:00
---

## useContext 를 사용해보자

- contexts/user.js

```javascript
const userContext = createContext();
const { Provider } = userContext;

function useUserContext() {
  const context = useContext(userContext);

  if (!context) {
    throw new Error(`useUserContext must be used within a UserProvider`);
  }

  return context;
}

function UserProvider(props) {
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

  const [user, dispatch] = useReducer(reducer, {});

  useEffect(() => {
    const refreshSilently = async () => {
      const payload = await reqeustJWTs({ url: routes.refresh });
      dispatch({ type: "login", payload });
    };

    refreshSilently();
  }, []);

  const value = { user, dispatch };

  return <Provider value={value}>{props.children}</Provider>;
}
```

- Nav.js

```javascript
export default function Nav() {
  const { user, dispatch } = useUserContext(); // consumer 대체

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
