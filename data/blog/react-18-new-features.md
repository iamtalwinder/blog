---
title: "The Future of UI Development: Exploring React 18's Newest Features"
date: 2022-08-21T16:32:14Z
lastmod: '2022-08-21'
tags: ['react']
draft: false
summary: "Discover how Concurrent Rendering, Automatic Batching, and enhanced Suspense capabilities revolutionize UI design and performance, setting new standards in dynamic web applications."
layout: PostSimple
canonicalUrl: https://talwinder.tech/blog/react-18-newest-features
---

## Overview

<TOCInline toc={props.toc} exclude="Overview" toHeading={2} />


## Introduction

React 18 introduces groundbreaking updates like Concurrent Rendering, transforming how developers build and manage dynamic UIs. This blog delves into these innovations, highlighting their impact on the future of web development.

## How to upgrade to React 18

Let's first see how we can upgrade to React 18

1. Update `react` and `react-dom` to 18
Using npm
```bash
npm install react@18 react-dom@18
```

Using yarn
```bash
yarn add react@18 react-dom@18
```

2. Modify Entry file

In `index.js`, update the rendering method to use the new createRoot.

React 17 (Before):

```js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(<App />, document.getElementById('root'));
```

React 18 (After):

```js
import React from 'react';
import { createRoot } from 'react-dom/client';
import App from './App';

const container = document.getElementById('root');
const root = createRoot(container);
root.render(<App />);
```

## Concept of Concurrent Rendering

Concurrent Rendering in React represents a significant shift in the way React handles rendering of components, allowing for a more efficient and smoother user experience, especially in complex applications. Here's a detailed explanation:

**Non-Blocking Rendering**: Traditional React rendering can block the main thread, leading to potential performance bottlenecks, especially in large or complex applications. Concurrent Rendering introduces a non-blocking approach, where React can prepare multiple versions of the UI at the same time.

**Priority-Based Rendering**: It allows React to interrupt a rendering process if a higher priority update comes in. This means that React can pause, resume, and even abandon renders based on the priority of updates, which is crucial for maintaining responsiveness.


**How to enable concurrent Rendering in react?**

Using `createRoot` to initialize your application puts you in a position to use concurrent features. It's the necessary first step but not sufficient in itself to make all updates concurrent. It enables your application to utilize concurrent features like `Suspense` for data fetching, `useTransition`, and `useDeferredValue`.

## Automatic Batching

[Github](https://github.com/iamtalwinder/react-18-automatic-batching-demo)
[StackBliz](https://stackblitz.com/edit/stackblitz-starters-emnd5j?file=src%2FApp.tsx)
[Deployed Demo](https://stackblitz-starters-emnd5j.stackblitz.io/)

In React 17 and earlier versions, automatic batching of state updates is limited to synchronous event handlers, like clicks or form submissions. This means multiple state updates within these events are batched into a single re-render. However, in asynchronous operations like setTimeout, Promises, and async/await, each state update triggers its own re-render, which can be less efficient.

React 18 improves on this by extending automatic batching to include asynchronous operations. Now, even in contexts like setTimeout or promise resolutions, multiple state updates are batched together, leading to fewer re-renders and enhanced performance. You can opt out of this behavious using `flushSync`.


Consider the following code, In React 18, pressing the 'Update State' button triggers only one re-render of the component, whereas in earlier versions like React 17, it would result in two re-renders.

<iframe
  src="https://stackblitz.com/edit/stackblitz-starters-emnd5j?embed=1&file=src%2Fcomponents%2FTimeoutBatchingDemo.tsx"
  style={{ width: '100%', height: '500px', border: 'none', borderRadius: '4px', overflow: 'hidden' }}
  title="React Automatic Batching Demo"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; microphone; midi; payment; usb; vr; xr-spatial-tracking">
</iframe>

## Transition API

A "transition" in this context is an update that can be delayed without impacting the user experience negatively. The primary purpose of the Transition API is to differentiate between urgent updates, like typing in an input field, and less urgent updates, like filtering a list based on the input.

**Key Benefits**
- **Improved User Experience**: By separating urgent updates from non-urgent ones, the Transition API ensures that the interface remains responsive and smooth.
- **Concurrent Rendering Compatibility**: It works seamlessly with concurrent rendering, allowing less urgent updates to be interrupted if more urgent updates come in.
- **Control Over Rendering Priority**: Developers gain more control over the rendering priority, deciding what updates should be rendered immediately and which can wait.

<iframe
  src="https://stackblitz.com/edit/stackblitz-starters-gdsd1y?embed=1&file=src%2FApp.tsx&hideExplorer=1"
  style={{ width: '100%', height: '500px', border: 'none', borderRadius: '4px', overflow: 'hidden' }}
  title="React Transition API demo"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; microphone; midi; payment; usb; vr; xr-spatial-tracking">
</iframe>

## Suspense Enhancements

- **Concurrent Rendering Integration**: React 18's suspense feature smoothly combines with concurrent rendering to enable more fluid interactions and transitions, especially when loading data. This improves user experience and lowers perceived load times.

- **Streaming Server-Side Rendering (SSR)**: Suspense streams HTML from the server in order to improve SSR capabilities while awaiting data. Time-to-content is accelerated by this feature, which improves initial load performance and SEO.

- **Integration with Data Fetching Libraries**: The new Suspense integrates with data fetching libraries seamlessly, making it easier to handle asynchronous data flows and loading states in applications.

- **Nested Suspense Components**: Better handling of nested suspense components makes it possible to create hierarchical, complex loading states that let various application components wait for resources or data on their own.


**Use Cases**

- **Data-Driven apps**: Suspense makes asynchronous data management easier by helping to gracefully handle loading states in apps that primarily rely on fetching data.
- **code Splitting and Lazy Loading**: It works with React.lazy for code splitting, rendering components only when their code is loaded, thereby optimizing performance and user experience.
- **Complex UIs with Concurrent Features**: In complex UIs utilizing concurrent features like transitions and startTransition, Suspense assists in managing component rendering based on data or resource availability.

<iframe
  src="https://stackblitz.com/edit/stackblitz-starters-lgvgxn?embed=1&file=src%2FApp.tsx&hideExplorer=1"
  style={{ width: '100%', height: '500px', border: 'none', borderRadius: '4px', overflow: 'hidden' }}
  title="React Transition API demo"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; microphone; midi; payment; usb; vr; xr-spatial-tracking">
</iframe>


## useId Hook

`useId` is a new hook in React 18 designed to generate unique, stable identifiers that are consistent across both server and client renders.


### Generating Unique IDs for Accessibility Attributes

Many accessibility features in web development rely on IDs to associate labels and controls (like label for input fields) or to link descriptive elements using ARIA attributes (e.g., `aria-labelledby`, `aria-describedby`). 

In dynamic applications, especially when components are generated on the fly, assigning unique and stable IDs to these elements can be challenging.

The useId hook simplifies this process by generating unique and consistent IDs across renders. 

<iframe
  src="https://stackblitz.com/edit/stackblitz-starters-rcgssd?embed=1&file=src%2FAccesibleFormComponent.tsx&hideExplorer=1"
  style={{ width: '100%', height: '500px', border: 'none', borderRadius: '4px', overflow: 'hidden' }}
  title="React useId demo"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; microphone; midi; payment; usb; vr; xr-spatial-tracking">
</iframe>

**Explanation**

- **Use of useId**: We use useId to generate two unique IDs: one for the input field (inputId) and one for the label (labelId).
- **Linking Label and Input**: The htmlFor attribute in the `<label>` tag is set to the ID of the input field. This links the label to the input, which is important for screen readers and overall accessibility.
- **ARIA Attribute**: Additionally, the aria-labelledby attribute in the `<input>` tag is set to the ID of the label. This provides an accessible name for the input field, further enhancing accessibility.

### Generating IDs for Several Related Elements

Often, elements in a UI are contextually related. For instance, a dropdown menu might have a button to open it and a list of items within it, each requiring unique but related IDs.

<iframe
  src="https://stackblitz.com/edit/stackblitz-starters-rcgssd?embed=1&file=src%2FAccordion.tsx&hideExplorer=1"
  style={{ width: '100%', height: '500px', border: 'none', borderRadius: '4px', overflow: 'hidden' }}
  title="React useId demo"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; microphone; midi; payment; usb; vr; xr-spatial-tracking">
</iframe>

**Explanation:**

- **Accordion Component:** This component renders multiple AccordionItem components, each representing an item in the accordion.

- **AccordionItem Component:** Each item uses useId to generate a unique identifier (accordionId), which is then used to create unique IDs for the button (buttonId) and content section (contentId) of each accordion item.

- **Accessibility:** The `aria-expanded`, `aria-controls`, and `aria-labelledby` attributes are used to enhance the accessibility of the accordion. They provide screen readers with the necessary information about the relationship between the button and the content section.

### Specifying a Shared Prefix for All Generated IDs

If you render multiple independent React applications on a single page, pass `identifierPrefix` as an option to your `createRoot` or `hydrateRoot` calls.

<iframe
  src="https://stackblitz.com/edit/stackblitz-starters-rcgssd?embed=1&file=src%2Findex.tsx&hideExplorer=1"
  style={{ width: '100%', height: '500px', border: 'none', borderRadius: '4px', overflow: 'hidden' }}
  title="React useId demo"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; microphone; midi; payment; usb; vr; xr-spatial-tracking">
</iframe>


