---
title: "12 React and React Native questions and answers - 3rd post"
seoDescription: "React and React Native questions and answers"
datePublished: Mon Jul 10 2023 17:25:58 GMT+0000 (Coordinated Universal Time)
cuid: cljx4x52u000s0algg2h48avm
slug: 12-react-and-react-native-questions-and-answers-3rd-post
tags: react-native, questions, reactjs

---

This is the 3rd post ([**previous post here**](https://osiel.hashnode.dev/12-react-and-react-native-questions-and-answers-2nd-post)) from a series where I will cover 12 react and react native common questions and answers every week until we reach 60 of them. After reaching that goal I will continue growing this list but from time to time. Useful links can be found in the answers. Corrections and suggestions are more than welcome.

1. What is the purpose of React Router in React?
    
    [React Router](https://reactrouter.com/en/main/start/overview) is a popular library used for handling routing in React applications. It allows you to create a [single-page application](https://developer.mozilla.org/en-US/docs/Glossary/SPA) with multiple views by defining different routes and mapping them to corresponding components. React Router enables navigation within the application without the need to reload the page.
    
2. What is the significance of React Fragments?
    
    [React Fragments](https://react.dev/reference/react/Fragment) provide a way to group multiple elements together without introducing an additional wrapping element. It's useful when you want to return multiple elements from a component's render method without adding unnecessary divs or spans to the [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction). Fragments help keep the DOM structure cleaner and avoid any unwanted side effects caused by additional elements.
    
3. How does React handle forms and form inputs?
    
    In React, form inputs are typically [controlled components](https://legacy.reactjs.org/docs/forms.html#controlled-components). This means that their value is controlled by the component's state. You can handle form input changes by providing an `onChange` event handler that updates the state value. By doing so, you have full control over the form input values and can perform validations or trigger other actions based on user input. You can also use uncontrolled components in React where form data is handled by the DOM itself. To write an [uncontrolled component](https://legacy.reactjs.org/docs/uncontrolled-components.html), instead of writing an event handler for every state update, you can [use a ref](https://legacy.reactjs.org/docs/refs-and-the-dom.html) to get form values from the DOM.
    
4. What is the concept of "lifting state up" in React?
    
    "[Lifting state up](https://react.dev/learn/sharing-state-between-components)" is a pattern in React where the state is moved from a child component to its parent component. This is useful when multiple child components need to share the same state or when the state needs to be accessed or modified by components higher in the component tree. By lifting the state up, the parent component becomes the source of truth for the shared state, and it can pass down the state and necessary callbacks as props to the child components.
    
5. What is the purpose of React Portals?
    
    [React Portals](https://legacy.reactjs.org/docs/portals.html) provide a way to render children components into a different DOM subtree that is outside the current component hierarchy. This is useful when you need to render a component at a different DOM location, such as a modal or a tooltip that needs to be positioned relative to an element outside the current component tree. Portals allow you to render components into a different part of the DOM while maintaining their logical parent component.
    
6. What is the purpose of React Context?
    
    [React Context](https://legacy.reactjs.org/docs/context.html) is a feature that allows you to share data between components without having to pass it explicitly through props at each level of the component tree. It provides a way to define data at the top level and make it accessible to all descendant components that opt into the context. This is particularly useful for sharing global state, such as theme settings or user authentication, across multiple components without the need for [prop drilling](https://react.dev/learn/passing-data-deeply-with-context#the-problem-with-passing-props).
    
7. What is the significance of React.memo()?
    
    [React.memo](https://legacy.reactjs.org/docs/react-api.html#reactmemo)() is a higher-order component (HOC) provided by React that memoizes a component's rendering output. It optimizes performance by preventing unnecessary re-renders of a component when its props haven't changed. React.memo() compares the new props with the previous props using shallow equality and re-renders the component only if the props have changed. It is especially useful for functional components that receive large or frequently changing props.
    
8. What is the concept of code splitting in React?
    
    [Code splitting](https://legacy.reactjs.org/docs/code-splitting.html#gatsby-focus-wrapper) is a technique used to split a React application's bundle into smaller chunks that can be loaded on-demand. It helps improve the initial load time of the application by reducing the amount of JavaScript that needs to be downloaded and parsed. It allows you to load only the code that is needed for a particular route or component, improving the application's performance, it's very related to the use of [lazy](https://react.dev/reference/react/lazy) and [Suspense](https://react.dev/reference/react/Suspense) in React.
    
9. How can you handle asynchronous operations in React?
    
    You can use [JavaScript promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) to handle asynchronous operations in React. Promises represent a value that may not be available immediately but will be resolved at a later time. You can use the `then()` method to handle the resolved value and the `catch()` method to handle any errors that occur during the asynchronous operation.
    
    Also, the [async/await syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) provides a more concise and readable way to handle asynchronous operations. It allows you to write asynchronous code that looks like synchronous code, making it easier to reason about and maintain. By marking a function as `async`, you can use the `await` keyword to pause the function execution until a promise is resolved or rejected.
    
10. What is the purpose of AsyncStorage in React Native?
    
    [AsyncStorage](https://react-native-async-storage.github.io/async-storage/docs/usage) is a simple, asynchronous, and persistent key-value storage system provided by React Native. It allows you to store and retrieve non-sensitive data locally on the user's device. AsyncStorage is commonly used for caching data or storing user preferences in React Native applications. If you want to know more about [Local Storage in React Native](https://osiel.hashnode.dev/local-storage-in-react-native) you can read my post about it [here](https://osiel.hashnode.dev/local-storage-in-react-native) when a guide is provided.
    
11. How can you handle push notifications in React Native?
    
    To handle push notifications in React Native, you can utilize libraries like [Firebase Cloud Messaging (FCM)](https://firebase.google.com/docs/cloud-messaging) or [OneSignal](https://onesignal.com/), among others. These libraries provide APIs and tools to register your application for push notifications, receive and handle incoming notifications, and customize the presentation of notifications. You can set up event handlers in your React Native code to perform specific actions when a push notification is received, such as displaying an alert or navigating to a specific screen in your application.
    
12. What is the concept of virtual DOM in React?
    
    The concept of the [virtual DOM](https://legacy.reactjs.org/docs/faq-internals.html) is a core part of React's rendering process. It is an abstraction of the real [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction) (Document Object Model) and serves as a lightweight representation of the actual HTML structure of a web page.
    
    When you render a React component, it generates a virtual DOM tree, which is a JavaScript object that describes the structure and properties of the elements in the component's render method. This virtual DOM tree reflects the desired state of the UI based on the component's current props and state.
    
    The virtual DOM tree allows React to efficiently calculate and identify the minimal set of changes needed to update the real DOM. Instead of directly manipulating the entire real DOM whenever a component's state or props change, React compares the previous virtual DOM with the new one and determines the optimal way to update the real DOM.
    

Previous post can be found here: [**12 React and React Native questions and answers - 2nd post.**](https://osiel.hashnode.dev/12-react-and-react-native-questions-and-answers-2nd-post)