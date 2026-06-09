# BACKEND AND FRONTEND

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