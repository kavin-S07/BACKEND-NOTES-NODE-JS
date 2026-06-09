
## BACKEND SETUP

---

## BACKEND LAYERS

---

## SERVER CONNECTION LAYER

---

1. Import/Require the required packages
   - Express
   - CORS

   Example:
   ```js
   const express = require('express');
   const cors = require('cors');
   ```

---

2. Create an Express Application

   Example:
   ```js
   const app = express();
   ```

---

3. Connect Frontend and Backend using CORS

   Example:
   ```js
   app.use(cors({
       origin: 'http://localhost:3000',
       credentials: true
   }));
   ```

   Purpose:
   - Allows frontend to access backend APIs.
   - Prevents CORS errors.

---

4. Enable JSON Data Parsing

   Example:
   ```js
   app.use(express.json());
   ```

   Purpose:
   - Converts incoming JSON request data into JavaScript objects.
   - Access data using `req.body`.

---

5. Import/Require Routers

   Example:
   ```js
   const authRouter = require('./Router/authRouter');
   ```

---

6. Register Routers with Express

   Example:
   ```js
   app.use('/auth', authRouter);
   ```

   Purpose:
   - Connects routes to the server.
   - Base path: `/auth`

   Examples:
   ```
   POST /auth/login
   POST /auth/signup
   ```

---

7. Create and Start the Server

   Example:
   ```js
   const PORT = 5000;

   app.listen(PORT, () => {
       console.log(`Server running on port ${PORT}`);
   });
   ```

---

## DATABASE LAYER

---

1. Import/Require the PostgreSQL (`pg`) library.

   Example:
   ```js
   const { Pool } = require('pg');
   ```

---

2. Create a Pool object and provide the database URL.

   Example:
   ```js
   const pool = new Pool({
       connectionString: process.env.DATABASE_URL
   });
   ```

---

3. Check whether the database is connected.

   Example:
   ```js
   pool.connect()
       .then(() => console.log('Database Connected'))
       .catch(err => console.log('Database Connection Failed', err));
   ```

---

4. Export the pool object.

   Example:
   ```js
   module.exports = pool;
   ```

   Purpose:
   - Connect Node.js application to PostgreSQL database.
   - Execute SQL queries using `pool.query()`.
   - Reuse database connections efficiently.

---

## CONTROLLER LAYER

---

1. Import/Require the database object (Pool).

   Example:
   ```js
   const pool = require('../config/db');
   ```

---

2. Import/Require required libraries.

   Example:
   ```js
   const bcrypt = require('bcryptjs');
   const jwt = require('jsonwebtoken');
   ```

---

3. JWT (JSON Web Token)

   Purpose:
   - Create authentication tokens.
   - Verify user identity.

   Syntax:
   ```js
   const token = jwt.sign(
       { id: user.id },
       process.env.JWT_SECRET,
       { expiresIn: '1d' }
   );
   ```

---

4. bcryptjs

   Purpose:
   - Hash passwords.
   - Compare passwords during login.

   Hash Password:
   ```js
   const hashedPassword = await bcrypt.hash(password, 10);
   ```

   Compare Password:
   ```js
   const isMatch = await bcrypt.compare(password, user.password);
   ```

---

5. Create Controller Method

   Syntax:
   ```js
   const methodName = async (req, res) => {
       const { name, email, password } = req.body;

       try {
           // Business Logic

           res.status(200).json({
               message: 'Success'
           });

       } catch (error) {
           res.status(500).json({
               message: 'Internal Server Error'
           });
       }
   };
   ```

---

6. Export Controller Methods

   Syntax:
   ```js
   module.exports = {
       methodName
   };
   ```

   Purpose:
   - Receive request from router.
   - Validate data.
   - Execute database queries.
   - Generate JWT tokens.
   - Hash and compare passwords.
   - Send response to client.

---

## MIDDLEWARE LAYER

---

1. Middleware

   Syntax:
   ```js
   const middlewareName = (req, res, next) => {
       // Logic

       next();
   };
   ```

---

2. Parameters

   - `req`  → Request Object
   - `res`  → Response Object
   - `next` → Pass control to next middleware/controller

---

3. JWT Authentication Middleware

   Example:
   ```js
   const jwt = require('jsonwebtoken');

   const authMiddleware = (req, res, next) => {
       const token = req.headers.authorization;

       if (!token) {
           return res.status(401).json({
               message: 'Access Denied'
           });
       }

       try {
           const decoded = jwt.verify(
               token,
               process.env.JWT_SECRET
           );

           req.user = decoded;

           next();

       } catch (error) {
           return res.status(401).json({
               message: 'Invalid Token'
           });
       }
   };
   ```

---

4. Export Middleware

   ```js
   module.exports = authMiddleware;
   ```

---

5. Use Middleware in Router

   ```js
   router.get(
       '/profile',
       authMiddleware,
       getProfile
   );
   ```

   Purpose:
   - Authenticate users.
   - Authorize access.
   - Validate requests.
   - Execute logic before controller.
   - Protect private routes.

   Flow:
   ```
   Request
      |
   Middleware
      |
   next()
      |
   Controller
      |
   Response
   ```

---

## ROUTER LAYER

---

1. Import/Require Express.

   Example:
   ```js
   const express = require('express');
   ```

---

2. Create Router Object.

   Example:
   ```js
   const router = express.Router();
   ```

---

3. Import/Require Controller Methods.

   Example:
   ```js
   const {
       signup,
       login
   } = require('../Controller/authController');
   ```

---

4. Import/Require Middleware (Optional).

   Example:
   ```js
   const authMiddleware = require('../Middleware/authMiddleware');
   ```

---

5. Create Routes.

   Syntax:
   ```js
   router.RESTAPIMethod(
       '/path',
       controllerMethod
   );
   ```

   Examples:
   ```js
   router.post('/signup', signup);

   router.post('/login', login);

   router.get(
       '/profile',
       authMiddleware,
       getProfile
   );
   ```

---

6. Export Router.

   Example:
   ```js
   module.exports = router;
   ```

   Purpose:
   - Define API endpoints.
   - Connect requests to controllers.
   - Apply middleware before controllers.
   - Organize routes in a separate file.

   Flow:
   ```
   Request
      |
   Router
      |
   Middleware (Optional)
      |
   Controller
      |
   Response
   ```

---
# FRONTEND LAYER

---

## 1. Create React Application

Command:
```
npx create-react-app appname
```

---

## 2. Important Files

```
public/index.html
src/App.js
src/index.js
```

---

## 3. Delete Unnecessary Files

```
logo.svg
App.test.js
reportWebVitals.js
setupTests.js
```

---

## 4. Run React Application

Command:
```
npm start
```

---

## 5. Install Packages

Syntax:
```
npm install packagename
```

Examples:
```
npm install axios
npm install react-router-dom
```

---

## 6. App Layer — `src/App.js`

Purpose:
- Main component.
- Contains all Routes.

Example:
```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/"         element={<Home />} />
        <Route path="/login"    element={<Login />} />
        <Route path="/register" element={<Register />} />
      </Routes>
    </BrowserRouter>
  );
}
```

---

## 7. Index Layer — `src/index.js`

Purpose:
- Entry point of React Application.
- Renders App Component into HTML.

Example:
```jsx
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

---

## 8. Components Layer — `src/components/`

Purpose:
- Store reusable UI pieces.

Examples:
```
Navbar
Footer
Sidebar
Button
Header
```

---

## 9. Pages Layer — `src/pages/`

Purpose:
- Store full application pages.

Examples:
```
Home
Login
Register
Dashboard
Profile
```

---

## 10. Context Layer — `src/context/`

Purpose:
- Global State Management.
- Share data between components without prop drilling.

Examples:
```
AuthContext
UserContext
ThemeContext
```

Create Context:
```jsx
import { createContext, useContext, useState } from 'react';

const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);

  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => useContext(AuthContext);
```

Use in Component:
```jsx
const { user, setUser } = useAuth();
```

Wrap in `index.js`:
```jsx
root.render(
  <AuthProvider>
    <App />
  </AuthProvider>
);
```

---

## 11. Services Layer — `src/services/`

Purpose:
- Connect Frontend to Backend.
- Manage all API requests in one place.

---

## 12. Axios — Install

```
npm install axios
```

---

## 13. Create Axios Instance — `src/services/api.js`

Purpose:
- Set base URL once.
- Reuse across all API calls.

Example:
```js
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:5000'
});
```

---

## 14. Attach JWT Token — Interceptor

Purpose:
- Automatically attach token to every request.

Example:
```js
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});
```

---

## 15. API Methods

Example:
```js
export const authAPI = {
  login: (email, password) =>
    api.post('/auth/login', { email, password }),

  register: (name, email, password) =>
    api.post('/auth/register', { name, email, password })
};
```

Use in Component:
```js
const response = await authAPI.login(email, password);
localStorage.setItem('token', response.data.token);
```

---

## 16. Types Layer — `src/types/` (TypeScript only)

Purpose:
- Define data structures using interfaces.

Example:
```ts
export interface User {
  id: number;
  name: string;
  email: string;
}
```

Use:
```ts
const [user, setUser] = useState<User | null>(null);
```

---

## 17. Folder Structure

```
src/
├── components/
├── context/
├── pages/
├── services/
├── types/
├── App.tsx
└── index.tsx
```

---

## FLOW

```
User
  |
Page
  |
Component
  |
Service (Axios)
  |
Backend API
  |
Database
  |
Response
  |
React UI Update
```

---
