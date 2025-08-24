
https://www.geeksforgeeks.org/reactjs/what-is-react-fiber/

In this article, we will learn about React Fiber. React Fiber is a concept of [ReactJS](https://www.geeksforgeeks.org/reactjs/reactjs-introduction/) that is used to render a system faster and smoother. React is one of the popular JavaScript library used to create a responsive user interface. React makes coding simple as compared to other frameworks. After certain changes who is the next element to render the system called [reconciler](https://www.geeksforgeeks.org/reactjs/reactjs-reconciliation/). This algorithm helps to compare two [DOM](https://www.geeksforgeeks.org/javascript/dom-document-object-model/) trees and diff them. React fiber helps to do it better.

**React Native:** It is a framework used for developing native-style applications of IOS and Android using JavaScript. For developing React native applications we have to install Node JS. We first install Node JS. You can follow the steps to install Node JS.

**Installation:** Now we will learn about how to React native installed by using expo cli to run React native app. Follow the below commands to install the React native. 

**Step1:** Install React Native CLI by using the npm command 

npm install -g expo-cli

**Step 2:** Now you can create a project. Suppose the project name is "firstProject"

expo init "firstProject"

**Step 3:** To check if React native is installed or not, use the start command in your project folder

CD "firstProject"
npm start web

**Project structure:** It will look like the following. 

![](https://media.geeksforgeeks.org/wp-content/uploads/20220408095659/Capture32-144x200.png)

Project structure 

**Example:** Here is the first code of React application in App.js. Inside firstproject/web App.js is included.

**App.js**

```js

import React from 'react';
import { Text, View, StyleSheet } from 'react-native';
import Constants from 'expo-constants';

const Home = () => {
    return (
        <Text style={{
            marginTop: 300,
            marginLeft: 10
        }}>
            Geeksforgeeks
        </Text>
    )
}

export default function App() {
    return (
        <View>
            <Home />
        </View>
    );
}

```

**Output:**

![](https://media.geeksforgeeks.org/wp-content/uploads/20220408100756/Capture31-133x200.png)

**React Fiber:** A fiber in a react is just a plain JavaScript object with some properties. The current react reconciler, the fiber reconciler is named after this object and is the default reconciler since react version 16.

React fiber is a complete rewrite of react that fixes a few long-standing issues and offers incredible and offers opportunities heading into the future.

**Goals of React Fiber:** Fiber focuses on animations and responsiveness. It has the ability to split work into chunks and prioritize tasks. We can pause work and come back to it later. We can also reuse previously completed work or maybe abort it if it is not needed. As opposed to the old React reconciler, it is asynchronous.

**Old reconciler: Stack -** Now, in order to truly understand the power of fiber, let's briefly talk about the old. The old reconciler is the **Stack reconciler.** Stack was synchronous and it has this name because it worked like a stack. You could add items, and remove items, but it had to work until the stack was empty. It couldn't be interrupted. Let's think of an example that uses the stack reconciler.

Imagine we have the text field. Ideally, we would like to always be able to type into the text field without any delay. If there is only the text field naturally that's not a problem, but what if there is something else happening? Suppose there is a network request happening in the background, that results in some element being rendered. If we type into the text field while those elements are being rendered, we will experience a delay, because the stack reconciler is in the middle of processing those elements. Now it should be clear where the problem was stack was synchronous and with that come to some major limitations.

**Features of Fiber:** These are some features listed below.

- While fiber comes with different significant performance increases, it's not really about them. It's about the fundamental way React works.
- Fiber makes React **faster** but it makes it **smarter** as well.
- Fiber also improves the development of react and makes it so adding a new feature is significantly easier.

**How to React fiber works?**

Now, let's look at the implementation itself. At the very start, we mentioned that fiber is a just plain javascript object with some properties. The core underlying idea is that the fiber also represents a unit of work.

React first processes those fibers, those units of work and we end with something called **finishwork()**. Afterward, it commits this work resulting in visible changes in the DOM. This all happens in two phases. The first is the render phase and it is during this phase that the processing happens. The second phase is the commit phase. Now, let's talk about them in order.

**Render phase:** This phase is asynchronous. During this phase, React does all sorts of asynchronous things behind the scenes that aren't visible to the user. With it being asynchronous come increased opportunities that I already briefly spoke about. React can prioritize tasks. Pause some work or maybe even discard it and so on. I previously said that during this phase, React processes all of the fibers, which represent the unit of work. During this phase, internal functions like **beginWork()** and **completeWork()** is being called. Those process all of the fiber and we will back to them later.

**Commit phase:** During this phase, it is the function **commitWork()** that is being called. This phase is synchronous and cannot be interrupted.

Whenever React processes a fiber it either handles the work directly or schedules it for the future. Using the feature called time-slicing React can split work into chunks. If some work has a very high priority like animation; React can schedule it in such a way that it gets handled as soon as possible, but if some work has low priority, for example, a network request React can simply delay it for as long as it needs. It uses a function **requestAnimationFrame()** and **requestIdleCallback()** to do that. 

**request AnimationFrame() -** Schedule for high priority functions to be called on the next animation frame. 

**requestIdleCallback() -** Schedules a low priority function to be called during an idle period.

Those functions are supported in the majority of browsers but in case they don't exist a polyfill.

**Structure of React Fiber:** Let's start with a simple example. Whenever we change state that is work, whenever there is a life cycle function that has to be called that is work, whenever there is an update that leads to change in the DOM that is work. We can see that work heavily depends on the fiber.

```js

function App() { // App 
    return (
        <div className="wrapper">// W
            <h2 className="h2">Heading h2</h2>
            <h3 className="h3">Heading h3</h3>
            <h4 className="h4">Heading h4</h4>
        </div>

    );
}
ReactDOM.render(<App />, 
    document.getElementById('root')); // HostRoot
    
```

**Output**:

![](https://media.geeksforgeeks.org/wp-content/uploads/20220408110206/Screenshot20220408105914Chrome-300x145.jpg)

Fiber renders h2, h3, and h4 in a sequence. 

This code shows div us root directory. Imagine a fiber tree starting with node div. This node has a child h2, h3, and h4. Fiber renders div first because it is a root, then it renders h2, h3, and h4 respectively.  h2, h3, and h4 are siblings of each other. 

**Fiber Tree:** There are actually two trees. The first one is called the current tree and the second one is called workInProgress tree. The current tree is that, what's currently on the screen, so it makes sense that React can't make a change to it, because it could result in an inconsistent UI and all sorts of inconveniences. React instead makes changes to the work in progress tree, it simply swaps pointers at the very end. Now workInProgress becomes a current tree, and the current tree is the workInProgress tree.

![](https://media.geeksforgeeks.org/wp-content/uploads/20220406195631/Currenttree-300x191.png)

Implementation of React