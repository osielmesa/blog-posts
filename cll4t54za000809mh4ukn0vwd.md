---
title: "12 React and React Native questions and answers - 4th post"
seoDescription: "React and React Native questions and answers"
datePublished: Thu Aug 10 2023 06:58:08 GMT+0000 (Coordinated Universal Time)
cuid: cll4t54za000809mh4ukn0vwd
slug: 12-react-and-react-native-questions-and-answers-4th-post
tags: react-native, questions, reactjs, answers

---

This is the 4th post ([**previous post here**](https://osiel.hashnode.dev/12-react-and-react-native-questions-and-answers-3rd-post)) from a series where I am covering 12 react and react native common questions and answers. I will continue growing this list from time to time. Useful links can be found in the answers. Corrections and suggestions are more than welcome.

1. **What is the role of PureComponent in React?**
    
    [`PureComponent`](https://react.dev/reference/react/PureComponent) is a class component in React that extends `Component`. It automatically implements a shallow prop and state comparison in the `shouldComponentUpdate` method, preventing unnecessary re-renders when props and state haven't changed. It's suitable for performance optimization when dealing with components that have no complex logic or internal state changes.
    
2. **What is Server-Side Rendering (SSR) in React?**
    
    Server-Side Rendering is a technique where React components are rendered on the server and sent to the client as HTML. This improves initial loading performance and SEO, as search engines can crawl and index the fully rendered content. Libraries like [Next.js](https://nextjs.org/) provide built-in support for SSR in React applications.
    
3. **What is the significance of React Native's "Bridge"?**
    
    React Native's "Bridge" refers to the communication layer that allows JavaScript code to interact with native code on iOS and Android platforms. The Bridge enables seamless communication between the JavaScript thread and the native UI thread, allowing React Native to render native components and handle device-specific functionality. React Antive's bridge is very important but contains [intrinsic limitations](https://reactnative.dev/docs/the-new-architecture/why#old-architectures-issues) to consider, the new [React Native architecture](https://reactnative.dev/docs/the-new-architecture/why#new-architectures-improvements) is designed to mitigate them.
    
4. **How does React Native handle platform-specific code?**
    
    React Native provides a mechanism to write platform-specific code using conditional rendering and special files. You can use the [`Platform`](https://reactnative.dev/docs/platform) module to conditionally render components, styles or logic based on the current platform. Additionally, you can create files named like `Component.ios.js` or [`Component.android`](http://Component.android)`.js` to provide platform-specific implementations.
    
5. **What is the purpose of React Native's "Touchable" components?**
    
    React Native provides "[Pressable](https://reactnative.dev/docs/pressable)" components (e.g., [`TouchableOpacity`](https://reactnative.dev/docs/touchableopacity), [`TouchableHighlight`](https://reactnative.dev/docs/touchablehighlight), [`TouchableWithoutFeedback`](https://reactnative.dev/docs/touchablewithoutfeedback)) that allow you to create touchable elements like buttons, links, or interactive elements. These components respond to user touch events and provide visual feedback, enhancing the user experience in your app.
    
6. **What are React Native's "Native Modules"?**
    
    React Native's [Native Modules](https://reactnative.dev/docs/native-modules-intro) are JavaScript modules that allow you to call native code functions from your JavaScript code. They provide a bridge between the JavaScript layer and the native platform code, enabling you to access native device capabilities that are not available through standard JavaScript APIs. Native Modules will be deprecated in the future when the New Architecture will be stable. The New Architecture uses [Turbo Native Module](https://reactnative.dev/docs/the-new-architecture/pillars-turbomodules) and [Fabric Native Components](https://reactnative.dev/docs/the-new-architecture/pillars-fabric-components) to achieve similar results.
    
7. **What is React Native's "flexbox" and how is it used for layout?**
    
    [Flexbox](https://reactnative.dev/docs/flexbox) is a layout model that allows you to design flexible and responsive layouts in React Native. It provides a set of CSS-like properties that enable you to distribute available space among components, align items, and control their size and order. Flexbox simplifies the process of creating complex and dynamic layouts in mobile apps.
    
8. To handle deep linking in React Native, you can use libraries like [`react-navigation`](https://reactnavigation.org/docs/deep-linking/). These kinds of libraries allow you to set up URL schemes or custom URLs that open specific screens or trigger actions within your React Native app, making it possible to navigate to specific parts of the app from external sources like push notifications or web links.
    
9. **What are React Hooks Rules of Use?**
    
    [React Hooks](https://react.dev/reference/react) Rules of Use include:
    
    * Hooks should only be called at the top level of a function component or a custom hook.
        
    * Hooks should not be called conditionally.
        
    * Hooks should not be called inside loops, nested functions, or event handlers.
        
    * Custom hooks should always start with the prefix "use" to differentiate them from regular functions.
        
10. What is the purpose of React's `Fragment` shorthand syntax (`<>...</>`)?
    
    The [`Fragment`](https://react.dev/reference/react/Fragment) shorthand syntax (`<>...</>`) serves the same purpose as React's `Fragment` component but provides a more concise way to group multiple elements without adding an extra DOM element. It's particularly useful when you need to return multiple elements from a component's render method without introducing a wrapping div or span to the DOM.
    
11. What is React Native's Metro bundler?
    
    [Metro](https://facebook.github.io/metro/) is the default [JavaScript bundler used in React Native](https://reactnative.dev/docs/metro) applications. It is responsible for transforming and bundling JavaScript code, assets, and other resources into a format that can be run on mobile devices. Metro optimizes the bundle size and performs transformations like minification, dead code elimination, and tree-shaking to improve the app's performance.
    
12. How does React Native's Hot Reloading work?
    
    React Native's [Hot Reloading](https://reactnative.dev/blog/2016/03/24/introducing-hot-reloading) allows developers to see instant updates to the code changes they make without requiring a full app restart. When Hot Reloading is enabled, the changes in the code are sent to the running app, which reloads only the modified components, preserving the app's current state. This significantly speeds up the development process and helps developers see real-time changes.
    

Previous post can be found here: [**12 React and React Native questions and answers - 3rd post.**](https://osiel.hashnode.dev/12-react-and-react-native-questions-and-answers-3rd-post)