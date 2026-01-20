# React App (From Scratch) – Auth, Role-based Routing, Lazy Loading

This guide helps you set up a **single React app** (no Vite, no CRA) with:

* Login / Signup
* JWT stored in **cookies**
* Centralized API service module
* Role-based authorization
* Dynamic menu from API
* Routing + Lazy loading
* Material UI

---

## 1. Project Setup (No Vite / No CRA)

### Folder Structure

```
react-auth-app/
├── public/
│   └── index.html
├── src/
│   ├── api/
│   │   └── apiClient.js
│   ├── auth/
│   │   ├── AuthContext.js
│   │   ├── RequireAuth.jsx
│   │   └── QueryResolver.jsx
│   ├── layout/
│   │   └── MainLayout.jsx
│   ├── pages/
│   │   ├── Login.jsx
│   │   ├── Signup.jsx
│   │   ├── Home.jsx
│   │   └── Dashboard.jsx
│   ├── routes/
│   │   └── AppRoutes.jsx
│   ├── App.jsx
│   └── index.js
├── package.json
└── webpack.config.js
```

---

## 2. Install Dependencies

```bash
npm init -y

npm install react react-dom react-router-dom
npm install @mui/material @emotion/react @emotion/styled
npm install webpack webpack-cli webpack-dev-server
npm install babel-loader @babel/core @babel/preset-env @babel/preset-react
```

---

## 3. Webpack Config

### `webpack.config.js`

```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
    publicPath: '/'
  },
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env', '@babel/preset-react']
          }
        }
      }
    ]
  },
  devServer: {
    historyApiFallback: true,
    port: 3000
  },
  resolve: {
    extensions: ['.js', '.jsx']
  }
};
```

---

## 4. Entry Files

### `public/index.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <title>React Auth App</title>
  </head>
  <body>
    <div id="root"></div>
    <script src="/bundle.js"></script>
  </body>
</html>
```

### `src/index.js`

```js
import React from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';

createRoot(document.getElementById('root')).render(<App />);
```

---

## 5. API Service Module

### `src/api/apiClient.js`

```js
const API_BASE = 'http://localhost:8080';

export async function apiRequest(url, options = {}) {
  const response = await fetch(API_BASE + url, {
    credentials: 'include', // sends cookies
    headers: { 'Content-Type': 'application/json' },
    ...options
  });

  if (!response.ok) throw new Error('API Error');
  return response.json();
}
```

---

## 6. Auth Context (JWT in Cookies)

### `src/auth/AuthContext.js`

```js
import React, { createContext, useContext, useEffect, useState } from 'react';
import { apiRequest } from '../api/apiClient';

const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [menus, setMenus] = useState([]);

  useEffect(() => {
    apiRequest('/auth/me')
      .then(data => {
        setUser(data.user);
        setMenus(data.menus);
      })
      .catch(() => setUser(null));
  }, []);

  return (
    <AuthContext.Provider value={{ user, menus, setUser }}>
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => useContext(AuthContext);
```

---

## 7. Route Guard (Role Based)

### `src/auth/RequireAuth.jsx`

```js
import { Navigate } from 'react-router-dom';
import { useAuth } from './AuthContext';

export default function RequireAuth({ role, children }) {
  const { user } = useAuth();

  if (!user) return <Navigate to="/login" />;
  if (role && user.role !== role) return <Navigate to="/" />;

  return children;
}
```

---

## 8. Query Resolver (Menu-based Access)

### `src/auth/QueryResolver.jsx`

```js
import { useAuth } from './AuthContext';

export default function QueryResolver({ menu, children }) {
  const { menus } = useAuth();

  return menus.includes(menu) ? children : null;
}
```

---

## 9. Pages

### Login

```js
import { Button, TextField } from '@mui/material';
import { apiRequest } from '../api/apiClient';

export default function Login() {
  const login = async () => {
    await apiRequest('/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email: 'a@b.com', password: '1234' })
    });
    window.location.href = '/dashboard';
  };

  return <Button onClick={login}>Login</Button>;
}
```

### Home (Public)

```js
export default function Home() {
  return <h2>Public Home</h2>;
}
```

### Dashboard (Protected)

```js
export default function Dashboard() {
  return <h2>Admin Dashboard</h2>;
}
```

---

## 10. Lazy Routing

### `src/routes/AppRoutes.jsx`

```js
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';
import RequireAuth from '../auth/RequireAuth';

const Home = lazy(() => import('../pages/Home'));
const Login = lazy(() => import('../pages/Login'));
const Signup = lazy(() => import('../pages/Signup'));
const Dashboard = lazy(() => import('../pages/Dashboard'));

export default function AppRoutes() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/login" element={<Login />} />
        <Route path="/signup" element={<Signup />} />
        <Route
          path="/dashboard"
          element={
            <RequireAuth role="ADMIN">
              <Dashboard />
            </RequireAuth>
          }
        />
      </Routes>
    </Suspense>
  );
}
```

---

## 11. App Root

### `src/App.jsx`

```js
import { BrowserRouter } from 'react-router-dom';
import { AuthProvider } from './auth/AuthContext';
import AppRoutes from './routes/AppRoutes';

export default function App() {
  return (
    <BrowserRouter>
      <AuthProvider>
        <AppRoutes />
      </AuthProvider>
    </BrowserRouter>
  );
}
```

---

## 12. Run App

```bash
npx webpack serve --mode development
```

---

## Next Steps

* Dynamic sidebar from `menus`
* Refresh token handling
* Logout (clear cookie on backend)
* Material UI layout + theme

If you want, I can:

* Add **dynamic menu rendering**
* Add **JWT refresh flow**
* Connect this with your **Spring Boot gateway**
