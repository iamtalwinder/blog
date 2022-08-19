---
title: "Understanding JS call, apply and bind functions"
date: 2022-08-28T17:32:14Z
lastmod: '2022-08-28'
tags: ['javascript']
draft: false
summary: "In this blog, we'll walk through JavaScript's call, bind, and apply methods, explaining their uses and differences to enhance your coding efficiency and flexibility."
layout: PostSimple
canonicalUrl: https://talwinder.tech/blog/understanding-js-call-apply-and-bing-functions
---

## Overview

<TOCInline toc={props.toc} exclude="Overview" toHeading={2} />

## Understanding Function Context in JavaScript

In JavaScript, `this` refers to the object that the function is a part of. The value of this is usually determined by how a function is called (its execution context). It's important to note that this may not refer to the object that defines the function, but rather to the object that invokes the function.

- **Global Context**: In the global execution context (outside of any function), this refers to the global object. In a browser, it’s window; in Node.js, it’s global.

```js
console.log(this); // In a browser, this will log the Window object
```

- **Object Methods**: When a function is called as a method of an object, this is set to the object the method is called on.

```js
const myObject = {
  name: "Example",
  showName: function() {
    console.log(this.name); // 'this' refers to myObject
  }
};

myObject.showName(); // Logs "Example"
```

- **Constructor Functions**: When a function is used as a constructor (with the `new` keyword), this refers to the newly created instance.

```js
function Person(name) {
  this.name = name;
}

const person1 = new Person("John");
console.log(person1.name); // Logs "John"
```

- **Event Handlers**

In the context of DOM event handlers, this refers to the element that received the event.

```html
<!-- HTML -->
<button id="myButton">Click me</button>

<script>
  // JavaScript
  document.getElementById("myButton").addEventListener('click', function() {
    console.log(this); // 'this' will refer to the element #myButton
  });
</script>
```

## Call method

The `call` method in JavaScript is a predefined function method that allows you to call a function with a specific `this` value and individual arguments. This is particularly useful for method borrowing, where you can call a function that belongs to another object, and pass in your own object as the context (`this` value).

**Basic Usage**

```js
function greet(greeting, punctuation) {
  return `${greeting}, my name is ${this.name}${punctuation}`;
}

const person = {
  name: 'Bob'
};

console.log(greet.call(person, 'Hi', '!')); // Outputs: 'Hi, my name is Bob!'

```

**Method Borrowing:**

```js

const person1 = {
  fullName: function() {
    return this.firstName + " " + this.lastName;
  }
};

const person2 = {
  firstName:"John",
  lastName: "Doe"
};

console.log(person1.fullName.call(person2)); // Outputs: John Doe

```

## Apply method

## Bind method

