---
title: "Optimising React & React Native Apps. Part 1: Controlled vs Uncontrolled Component."
seoDescription: "Performance issue in react and react native apps. Controlled component 's problem and how to fix it."
datePublished: Fri Jul 21 2023 15:01:54 GMT+0000 (Coordinated Universal Time)
cuid: clkcpm810000309mp4k9cajl5
slug: optimising-react-react-native-apps-part-1-controlled-vs-uncontrolled-component
tags: performance, react-native, reactjs, controlled-and-uncontrolled-components

---

As developers, one of our top priorities is to create high-performing and responsive applications. When building applications with React and React Native, understanding how to optimize components is crucial for achieving a smooth user experience. In this two-part series, we will delve into various optimization techniques to make your React and React Native apps faster and more efficient. In Part 1, we'll explore the difference between [controlled components](https://legacy.reactjs.org/docs/forms.html#controlled-components) and [uncontrolled components](https://legacy.reactjs.org/docs/uncontrolled-components.html) and the impact each has on app performance.

This post is focused on React Native for the provided examples but you can apply the same concepts in React apps.

### Understanding Controlled Components

In React and React Native, a controlled component is one whose value is controlled by the component's state. This means that the component's value is updated and maintained through the state, and any changes to the component are managed by event handlers, which update the state accordingly. Controlled components are commonly used with form elements like text inputs, checkboxes, radio buttons, etc.

Let's take a look at the following example:

```typescript
const ParentComponent: React.FC = () => {
  const [value, setValue] = React.useState('');

  const onValueChanged = (text: string) => {
    setValue(text);
  };

  return (
    <View>
      <TextInput
        value={value}
        onChangeText={onValueChanged}
      />
    </View>
  );
};
```

In this example, the `value` state controls the value of the text input. The `onValueChanged` event handler updates the state whenever the user types in the input field, causing a re-render to display the updated value.

### Benefits of Controlled Components

* **Single Source of Truth**: Since the component's value is derived from the state, there is a single source of truth, making it easier to manage and synchronize data.
    
* **Validation and Pre-processing**: Data can be easily validated and pre-processed before it's applied to the component, ensuring consistent and reliable data handling. It allows for immediate data manipulation and real-time feedback to the user.
    

### Understanding Uncontrolled Components

Uncontrolled components, on the other hand, do not rely on the state to control their value. Instead, they directly read and update their value from the DOM or the React Native component itself. Uncontrolled components are commonly used when you don't need to perform any additional operations upon every user interaction. Let's consider the same text input, but this time as an uncontrolled component:

```typescript
const ParentComponent: React.FC = () => {
  const inputRef = React.useRef('');

  const handleButtonClick = () => {
    const text = inputRef.current.value;
    //Do something with text value.
  };

  return (
    <View>
      <TextInput ref={inputRef} />
      <Button onClick={handleButtonClick}> Submit </Button>
    </View>
  );
};
```

In this example, we use a `ref` to directly access the `TextInput` component and retrieve its value when the button is pressed. The value is not managed by React's state but is directly obtained from the native component.

### Benefits of Uncontrolled Components

Uncontrolled components offer certain advantages as well:

* **Reduced Re-renders**: Uncontrolled components can avoid unnecessary re-renders since they do not rely on state changes for value updates.
    
* **Simplified Implementation**: For simple use cases, uncontrolled components require less code and can be quicker to implement.
    
* **Direct Access to Native Components (in case of React native)**: Uncontrolled components allow direct access to native components, which can be useful for integrating with native APIs or third-party libraries.
    

### Choosing the Right Component Approach

When deciding whether to use controlled or uncontrolled components in your React or React Native app, consider the following:

* **Use Controlled Components** when you need precise control over the data flow, validation, and real-time manipulation of the component's value.
    
* **Use Uncontrolled Components** when you want to avoid extra state management and do not require real-time updates for every user interaction.
    

### Performance issues of controlled components and how to mitigate them

To demonstrate the issue, let's consider our previous example with a `ChildComponent` included:

```typescript
import React from 'react';
import {View, Text, StyleSheet, TextInput} from 'react-native';

const ChildComponent: React.FC<{value: string}> = ({value}) => {
  const backgroundColor = `rgb(${Math.floor(Math.random() * 256)}, ${Math.floor(
    Math.random() * 256,
  )}, ${Math.floor(Math.random() * 256)})`;

  return (
    <View style={[styles.card, {backgroundColor}]}>
      <Text>{value}</Text>
    </View>
  );
};

const ParentComponent: React.FC = () => {
  const [value, setValue] = React.useState('');

  const onValueChanged = (text: string) => {
    setValue(text);
  };

  return (
    <View>
      <TextInput
        value={value}
        onChangeText={onValueChanged}
        style={styles.input}
      />
      <ChildComponent value={'This is the child component with fixed value'} />
    </View>
  );
};

const App: React.FC = () => {
  return (
    <View style={styles.container}>
      <ParentComponent />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  input: {
    borderWidth: 1,
    borderColor: 'gray',
    marginTop: 20,
  },
  card: {
    padding: 20,
    marginTop: 40,
    borderColor: 'gray',
    borderWidth: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

This React Native code consists of three components: `ChildComponent`, `ParentComponent`, and the main `App` component. Let's break down each component and its functionality:

**ChildComponent:** displays a colored card with the given `value` prop. The background color of the card is set randomly every time the component renders.

**ParentComponent:** contains a `TextInput` and the `ChildComponent`. The `TextInput` allows users to enter text, and the entered value is stored in the `value` state using the `onValueChanged` function.

**App:** is the entry point of the application. It renders the `ParentComponent` inside a `View`, which serves as the container for the entire app.

That code produces the following user interface:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689949068769/77cb975c-fe83-4163-9bf7-e9403c0d9edd.png align="center")

Notice the background of the `ChildComponent` is set randomly every time the component renders.

Now, let's focus our attention on `ChildComponent` and how re-renders every time we change the text in the `TextInput`:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689949462649/f71e1b24-b6e8-42e0-9ba0-383a4e5f86da.gif align="center")

The ChildComponent receives a value as a prop and we are providing this value with a fixed and hardcoded text, if so, why is this component re-rendering every time? easy! we are changing the state of the `ParentComponent` every time we type something in the input, this provokes re-renders on all its children.

Imagine that we need this TextInput as a controlled component here for some reason, how can we solve this problem in our example? One possible solution: We can use memoization in the `ChildComponent`. Let's see how:

```typescript
const ChildComponent: React.FC<{value: string}> = React.memo(({value}) => {
  const backgroundColor = `rgb(${Math.floor(Math.random() * 256)}, ${Math.floor(
    Math.random() * 256,
  )}, ${Math.floor(Math.random() * 256)})`;

  return (
    <View style={[styles.card, {backgroundColor}]}>
      <Text>{value}</Text>
    </View>
  );
});
```

`React.memo` is a higher-order component (HOC) that can significantly boost performance by preventing unnecessary re-renders of functional components. It memoizes the component and re-renders it only when its props change.

Let's see how our ChildComponent behaves after we introduce `React.memo`:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1689950252226/6827cec2-34c3-4c55-ad65-8d41aaee6d17.gif align="center")

Notice how the `ChildComponent` does not change the background color. This means it's not re-rendering every time the text value is changed in the `TextInput`.

Every time you need to work with forms and want to improve performance you should be aware of the previously explained issue. Sometimes you have bigger components with a lot of children and complex UI including forms, some libraries will make your life easier and among other things will solve the previous performance problem, e.g. React Hook Form (check their **Isolate Re-renders** section on their [home page](https://www.react-hook-form.com/)).

### Conclusion

In Part 1 of our series on optimizing React and React Native apps, we explored the concepts of controlled and uncontrolled components. Controlled components provide a single source of truth and enable data validation and real-time updates. Uncontrolled components can lead to fewer re-renders and offer a simplified approach when direct DOM access is required.

The choice between controlled and uncontrolled components depends on your specific app requirements and performance considerations. Stay tuned for Part 2, where we will continue our journey to optimize React and React Native apps by exploring additional performance enhancement strategies.