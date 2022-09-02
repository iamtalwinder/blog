---
title: "React form validation with useForm and yup"
date: 2022-09-12T13:00:00Z
lastmod: '2022-09-12'
tags: ['react', 'validation', 'useform', 'yup', 'miragejs', 'mock']
draft: false
summary: "This blog details the creation of a React-based application, showcasing the integration of Material UI for design, react-hook-form and Yup for form validation, and MirageJS for API mocking."
layout: PostSimple
canonicalUrl: https://talwinder.tech/blog/form-validation-in-react-with-use-form-and-yup
---

[See complete code on github](https://github.com/iamtalwinder/react-form-validation-example)

## Overview

<TOCInline toc={props.toc} exclude="Overview" toHeading={2} />

## Introduction
Dive into the intricacies of advanced form validation in React, as this blog explores the seamless integration of react-hook-form and Yup for client-side validation, coupled with MirageJS for simulating API responses, showcasing a comprehensive approach to handling form data and server interactions in modern web applications.

Below is the complete example:

<iframe
  src="https://stackblitz.com/edit/stackblitz-starters-v9argg?embed=1&file=src%2FTodoForm.tsx"
  style={{ width: '100%', height: '500px', border: 'none', borderRadius: '4px', overflow: 'hidden' }}
  title="React Automatic Batching Demo"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; microphone; midi; payment; usb; vr; xr-spatial-tracking">
</iframe>

## Setting up project

Create new `react` project with `typescript`

```bash
npx create-react-app react-form-validation-example --template typescript
```

```bash
cd react-form-validation-example
```

Add material UI
```bash
npm install @mui/material @emotion/react @emotion/styled
```

Add `yup` and `react-hook-form`, `@hookform/resolvers` is required to integrate `yup` schema with `react-hook-form`.
```bash
npm install react-hook-form yup @hookform/resolvers yup
```


## Adding form validation

1. **Validating the Todo Form with Yup**

To begin, we define a validation schema for the todo item using Yup. This schema `const schema: yup.ObjectSchema<Todo>` specifies the rules for each field in our form:

```tsx
const schema: yup.ObjectSchema<Todo> = yup
  .object({
    title: yup.string().min(3).max(20).required('Title is required'),
    description: yup.string().min(3).max(50).required('Description is required'),
    status: yup.mixed<StatusEnum>().oneOf(Object.values(StatusEnum)).required(),
    category: yup.string().required('Category is required'),
  })
  .required();
```

Here, `yup.string()`, `yup.mixed()`, and methods like `min()`, `max()`, `required()`, and `oneOf()` are used to define various constraints for the title, description, status, and category fields.

2. **Integrating useForm with Yup Resolver**

To connect our form with Yup validation, we use `useForm` from `react-hook-form` along with `yupResolver`:

```tsx
const { control, handleSubmit, formState: { errors }, setError, reset } = useForm({
  resolver: yupResolver(schema),
});
```

`useForm` is initialized with `yupResolver`, which adapts the Yup validation schema to be used by `react-hook-form`.

3. **Using Controller for Material UI Components**
Material UI components are wrapped in the `Controller` component from `react-hook-form` to integrate them with the form validation logic:

```tsx
<Controller
  name="title"
  control={control}
  defaultValue=""
  render={({ field }) => (
    <TextField
      {...field}
      label="Title"
      fullWidth
      margin="normal"
      error={!!errors.title}
      helperText={errors.title?.message}
    />
  )}
/>
```

The `Controller` component manages the state and validation of Material UI inputs like `TextField` and `Select`.

4. **Form Submission Logic**

The form submission logic is handled by the `onSubmit` function, which is triggered when the form is submitted:

```tsx
const onSubmit = async (data: Todo) => {
  console.log(data);
  // Further submission logic can be added here
};
```

The `handleSubmit` function from `react-hook-form`, when passed `onSubmit`, takes care of executing validation before running the submit logic.


## Mocking API with MirageJS


```ts:src/mock-server.ts
import { createServer, Response } from 'miragejs';

function createMockServer({ environment = 'development' } = {}) {
  let server = createServer({
    environment,

    routes() {
      this.namespace = 'api'; // Define a namespace for all mock API routes

      this.post('/todos', (schema, request) => {
        const attrs = JSON.parse(request.requestBody);

        if (attrs.title.toLowerCase() === 'Error') {
          return new Response(
            400,
            {},
            { errors: { title: 'Title cannot be "Error"' } }
          );
        }

        return new Response(200, {}, { todo: attrs });
      });
    },
  });

  return server;
}

export default createMockServer;
```

**Explanation:**

- A POST route to `api/todos` is defined, simulating the API endpoint for creating new todo items.
- The server checks if the `title` field in the request body is "Error". If so, it responds with an HTTP 400 status code and a custom error message, mimicking server-side validation.
- For titles not matching the error condition, the server responds with a 200 status code, simulating a successful todo creation.

```tsx:App.tsx
import { Box } from '@mui/material';
import TodoForm from './TodoForm';
import createMockServer from './mock-server';

createMockServer();

function App() {
  return (
    <Box sx={{ maxWidth: 500, margin: 'auto' }}>
      <TodoForm />
    </Box>
  );
}

export default App;
```

## Integrating API response with validation
To integrate API with the todo form change `onSubmit` in `src/TodoFrom.tsx`

```tsx:src/TodoForm.tsx
const onSubmit = async (data: Todo) => {
    try {
      const response = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });

      const responseData = await response.json();

      if (!response.ok) {
        // Iterate over the errors and set them on the respective fields
        if (responseData.errors) {
          Object.keys(responseData.errors).forEach((key) => {
            setError(key as keyof Todo, {
              type: 'manual',
              message: responseData.errors[key],
            });
          });
        }
        throw new Error('Failed to create todo');
      }

      // Handle successful todo creation here
      alert('Todo created successfully');
      reset();
    } catch (error: any) {
      // Show a general error message or handle it as needed
      alert(error.message);
    }
  };
```

**Explanation**

- **Form Submission (`onSubmit`)**: The `onSubmit` function is an asynchronous function that gets triggered when the form is submitted. It sends the form data (`data: Todo`) to the `/api/todos` endpoint using the `fetch` API with a POST request.

- **Error Handling from API Response**: After the API call, it checks if the response was not OK (`!response.ok`). If there are errors in the response (`responseData.errors`), it iterates over these errors and uses the `setError` function from `react-hook-form` to set these errors on the respective form fields. This integrates server-side validation feedback into the form.

- **Successful Submission Handling**: If the API call is successful (no errors), it shows an alert indicating successful todo creation and then resets the form to its initial state using the `reset` function from `react-hook-form`.

- **Catch Block for Network Errors**: The `catch` block handles any potential errors that might occur during the fetch operation (like network issues), displaying an alert with the error message.