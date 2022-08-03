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

[See complete code on github](https://github.com/iamtalwinder/angular-ag-grid-state-persistence-blog)

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
