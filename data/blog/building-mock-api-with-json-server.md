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

```json
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

