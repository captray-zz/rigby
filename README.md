[![Build Status](https://travis-ci.org/captray-zz/rigby.svg?branch=master)](https://travis-ci.org/captray/rigby)

# rigby

React is great, but... y'know.  Name inspired from Silicon Valley on HBO
[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/pF8o3UrwksU/0.jpg)](https://www.youtube.com/watch?v=pF8o3UrwksU)

## Why

Less boilerplate when creating Stores and a more fluent API for doing so. When using rigby with React it allows you to access shared store state and have changes to that state reflected in the UI.

## Installation

[npm i rigby](https://www.npmjs.com/package/rigby)

## Creating a Store

```javascript

Rigby.createStore('YourStoreName', {
  state: {
    todos: [
      { 
        text: 'Your Data Goes Here',
        complete: false
      }
    ]
  },
  actions: {
    addTodo: function(text, complete) {
      this.state.todos.push({ text: text, complete: complete });
      this.emitChange();
    }
  }
});

```

Or with ES2015+:

```javascript
import Store from "rigby";

class YourStore extends Store {
  constructor() {
    super("YourStoreName");

    this.state.todos = [ 
      { text: "Your Data Goes Here", complete: false }
    ];
  }

  addTodo(text, complete) {
      this.state.todos.push({ text, complete });
      this.emitChange();
  }
}

```

Or with TypeScript:

```javascript
import Store from "rigby";

interface ToDo {
  text: string;
  complete: boolean;
}

interface YourStoreState {
  todos: ToDo[];
}

class YourStore extends Store<YourStoreState> {
  constructor() {
    super("YourStoreName");

    this.state.todos = [ 
      { text: "Your Data Goes Here", complete: false }
    ];
  }

  addTodo(text: string, complete: boolean) {
      this.state.todos.push({ text, complete });
      this.emitChange();
  }
}

```

## Dispatching Actions

```javascript
import Rigby from "rigby";

Rigby.dispatch("addTodo", "Your New Todo", false);
```

or call the methods on the store directly
```javascript
import TodoStore from "./store";

TodoStore.addTodo("Your New Todo", false);
```

## With React Classes
```javascript
import * as React from 'react'
import CounterStore from "./store";

export class Counter extends React.Component {
    onStoreChangeCallbackId = 0;
    state = CounterStore.getState();
    componentDidMount() {
        this.onStoreChangeCallbackId = CounterStore.listen((counterStoreState) => {
            // can use forceUpdate here too if you'd prefer and grab the store state in render
            this.setState(counterStoreState);
        });
    }
    componentWillUnmount() {
        // stop listening for changes
        CounterStore.mute(this.onStoreChangeCallbackId);
    }
    render() {
        const { count } = this.state;
        return (
            <button onClick={() => CounterStore.increment()}>{count}</button>
        )
    }
}
```

## With React Hooks

This is the hook itself, currently not in the source, so copy this into your project.
```javascript
import * as React from "react";

export function useRigby(store) {
    const [, forceUpdate] = React.useState({});
    React.useEffect(() => {
        const callbackId = store.listen(() => {
            forceUpdate({});
        });

        return () => store.mute(callbackId);
    }, []);

    return {
        state: store.getState(),
        store
    };
}
```
Typescript version
```typescript
import * as React from "react";

export function useRigby<T extends Store<any>>(store: T) {
    const [, forceUpdate] = React.useState({});
    React.useEffect(() => {
        const callbackId = store.listen(() => {
            forceUpdate({});
        });

        return () => store.mute(callbackId);
    }, []);

    return {
        state: store.getState(),
        store
    };
}
```

Using the hook:
```typescript
import { useRigby } from "../hooks";
import CounterStore from "./store";

export function Counter() {
    const { state: counterStoreState, store } = useRigby(CounterStore);
    return (
        <button onClick={() => store.increment()}>Increment {counterStoreState.count}</button>
    );
}
```

## Plans

This is a thought experiment, and there are plans for versions that allow for using RxJS or move in the direction of things like Cycle and Yolk.
