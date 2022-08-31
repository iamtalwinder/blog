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




