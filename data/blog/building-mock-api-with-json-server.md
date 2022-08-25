---
title: "Building Mock API with JSON Server: Integrating Authentication, and Dynamic Data Generation"
date: 2022-09-05T13:00:00Z
lastmod: '2022-09-05'
tags: ['javascript', 'nodejs', 'mock-api', 'json-server']
draft: false
summary: "Master the art of API mocking with authentication and dynamic data generation to accelerate your development process."
layout: PostSimple
canonicalUrl: https://talwinder.tech/blog/building-mock-api-with-json-server
---

[See complete code on github](https://github.com/iamtalwinder/json-server-mock-api)

## Overview

<TOCInline toc={props.toc} exclude="Overview" toHeading={2} />

## Introduction

In an era where front-end and back-end development often occur simultaneously, having a mock API is crucial. It allows front-end developers to continue their work without waiting for the back-end services to be fully implemented, leading to a more efficient and streamlined development process.

**Key takeaways**

- **Authentication:** We'll integrate basic authentication to secure our mock API, simulating real-world scenarios where certain API routes require user authentication.
- **Dynamic Data Generation:** To mimic real data, we’ll use Faker.js. This powerful library generates realistic datasets, enhancing the testing and development experience.
- **Docker Integration:** Finally, we’ll containerize our mock API with Docker. This step ensures our setup is easily replicable and deployable, mimicking a production-like environment.

## Getting Started with JSON Server

JSON Server provides a quick and seamless way to create a full-fake REST API with minimal setup. It's not just easy to set up but also offers flexibility in defining data structures that closely resemble your actual backend.

1. **Install JSON Server:**

```bash
npm install -g json-server
```

2. **Create a `db.json` File:** JSON Server uses a simple JSON file as the database. Let's create one. In your project folder, make a file named db.json. For the start, let's add some sample data. For example:

```json:db.json
{
  "posts": [
    { "id": 1, "title": "Hello World", "author": "Jane Doe" }
  ],
  "comments": [
    { "id": 1, "body": "Some comment", "postId": 1 }
  ],
  "profile": {
    "name": "Jane Doe"
  }
}
```

3. **Start JSON Server:** To get your API up and running, execute the following command in your terminal:

```bash
json-server --watch db.json
```

4. **Accessing Your API:** Once the server starts, it will display the resources available. By default, JSON Server will run on port 3000. You can access your new API through `http://localhost:3000/posts`, `http://localhost:3000/comments`, and so on.

## Adding Authentication

In real-world applications, APIs often require authentication to protect sensitive data and functionalities. When developing front-end applications, it’s crucial to simulate this behavior to ensure that the UI interacts correctly with protected endpoints. For this 
purpose we need to use middlewares, lets create a new project:

**Create a new project:**
```bash
mkdir json-server-mock-api
```

```bash
cd json-server-mock-api
```

```bash
npm init --y
```

**Install required dependencies:**

```bash
npm i json-server jsonwebtoken dotenv @faker-js/faker
```

**Following will be our project structure:**

```bash
    src
      - db
         -- db.json
         -- index.js
         -- seed-data.js
      - middleware
         -- authenticate.js
      - routes
         -- auth.js
      - config.js
      - server.js
```

**Explanation:**

- **`/src` Directory**: The main source code of the project.

  - **`/db`**: Contains files related to the mock database.
    - `db.json`: The mock database file, which JSON Server uses to create the API.
    - `index.js`: Central file for database configurations or exports.
    - `seed-data.js`: A script to generate and populate `db.json` with dynamic data using Faker.

  - **`/middleware`**: Holds middleware functions.
    - `authenticate.js`: The authentication middleware to protect certain API routes.

  - **`/routes`**: Contains files defining various API routes.
    - `auth.js`: Manages authentication-related routes, like login and token validation.

  - `config.js`: A configuration file for the project, possibly including settings like database paths, API keys, or environment-specific configurations.

  - `server.js`: The main server file where JSON Server is configured and started. This file integrates all the routes, middleware, and database configurations.


```js:src/config.json
const path = require('path');

require('dotenv').config();

const env = process.env;

module.exports = {
  JWT_SECRET_KEY: env.JWT_SECRET_KEY,
  DB_PATH: path.join(__dirname, './db/db.json'),
  PORT: env.PORT || 3000
};
```

```js:src/middlewares/authenticate.js
const jwt = require('jsonwebtoken');
const { JWT_SECRET_KEY } = require('../config');

const authenticate = (req, res, next) => {
  const authHeader = req.headers.authorization;

  if (authHeader) {
    const token = authHeader.split(' ')[1];

    jwt.verify(token, JWT_SECRET_KEY, (err, user) => {
      if (err) {
        return res.sendStatus(403);
      }

      req.user = user;
      next();
    });
  } else {
    res.sendStatus(401);
  }
};

module.exports = authenticate;
```

**Explanation:**

The `authenticate` middleware is designed to secure routes in an Express.js application by validating JSON Web Tokens (JWT). Here's how it functions:

1. **Bearer Token Extraction**:
   - The middleware expects an authentication token in the `Authorization` header of the incoming HTTP request.
   - The expected format is `Bearer [token]`. The middleware splits the header value by a space and retrieves the token.

2. **Token Verification with JWT**:
   - The extracted token is then verified using the `jwt.verify` method from the `jsonwebtoken` package.
   - The `JWT_SECRET_KEY`, obtained from the application's configuration, is used to verify the token's integrity and authenticity.

3. **Response Based on Verification**:
   - **Valid Token**: If the token is verified successfully, the request is allowed to proceed to the next middleware or route handler.
   - **Invalid or No Token**: If the token is missing, invalid, or expired, the middleware responds with an appropriate HTTP status (401 Unauthorized or 403 Forbidden), indicating that access is denied.

This approach ensures that only requests with a valid JWT are granted access to protected routes.


```js:src/routes/auth.js
const express = require('express');
const jwt = require('jsonwebtoken');

const db = require('../db');
const { JWT_SECRET_KEY } = require('../config');

const router = express.Router();

router.post('/signup', (req, res) => {
  const { username, password } = req.body;

  if (!username || !password) {
    return res.status(400).send('Missing username or password');
  }

  const existingUser = db.get('users').find({ username }).value();
  if (existingUser) {
    return res.status(409).send('Username already exists');
  }

  db.get('users').push({ id: Date.now(), username, password }).write();

  res.status(201).send('User created');
});

router.post('/login', (req, res) => {
  const { username, password } = req.body;
  const user = db.get('users').find({ username, password }).value();

  if (!user) {
    return res.status(401).send('Incorrect username or password');
  }

  const accessToken = jwt.sign({ username, userId: user.id }, JWT_SECRET_KEY, { expiresIn: '15m' });
  const refreshToken = jwt.sign({ username, userId: user.id }, JWT_SECRET_KEY, { expiresIn: '7d' });

  res.status(200).json({ accessToken, refreshToken });
});

router.post('/refresh', (req, res) => {
  const { refreshToken } = req.body;
  if (!refreshToken) {
    return res.status(401).send('Refresh Token Required');
  }

  try {
    const userData = jwt.verify(refreshToken, JWT_SECRET_KEY);
    const newAccessToken = jwt.sign({ username: userData.username, userId: userData.userId }, JWT_SECRET_KEY, { expiresIn: '15m' });

    res.status(200).json({ accessToken: newAccessToken });
  } catch (error) {
    return res.status(403).send('Invalid Refresh Token');
  }
});

module.exports = router;
```

lets combine all these in server file

```js:src/server.js
const jsonServer = require('json-server');
const { DB_PATH, PORT } = require('./config');
const authRoutes = require('./routes/auth');
const authenticate = require('./middlewares/authenticate');

const router = jsonServer.router(DB_PATH);
const server = jsonServer.create();
const middlewares = jsonServer.defaults();

server.use(middlewares);
server.use(jsonServer.bodyParser);

server.use(authRoutes);

// Add authentication to routes in db.json
server.use('/users', authenticate);
server.use('/posts', authenticate);
server.use('/comments', authenticate);

// Custom routes
server.get('/protected', authenticate, (req, res) => {
  res.status(200).send('Protected data');
});

server.use(router);

server.listen(PORT, () => {
  console.log('Server is running');
});
```

Add following scripts in `package.json`

```json:package.json
{
    "scripts": {
        "seed:data": "node src/db/seed-data.js",
        "start": "node src/server.js",
    }
}
```

Run the project:

```bash
npm start
```



