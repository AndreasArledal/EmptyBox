# Flux VS Single State Tree

I attended the Reactive Conference in Bratislava a couple of weeks ago. I even gave a talk on [Cerebral](https://youtu.be/BfzjuhX4wJ0?t=6h10m48s) if you want to check it out. One of the questions after the talk was "How does Cerebral relate to Redux?". This is a difficult question to answer because they really try to solve different problems, but I would like the opportunity to answer a redefined question: "How does Flux differ from using a single state tree?". That way I am not in a position to favor my own creation over Redux. You might say now; "But is not Redux a single state tree?". No, it is really not, and I will explain why. I will also be answering a more fundamental question about Flux's verbosity, even in Redux. Yes, I will actually say something bad about Redux. It does not feel good, because Redux is a really great project and Dan Abramov is one of the most humble developers in the community. But I think Dan will agree that even though Redux has become the de facto standard for Flux we should still bring in new ideas and talk about how we can solve our day to day problems with different approaches.

If you have not heard of Flux before you should read up a bit on that first. I will be using [Facebook Flux](https://facebook.github.io/flux/), the [Alt](http://alt.js.org/) project and [Redux](https://github.com/rackt/redux) to explain the evolution of Flux. Then I will compare them to using a single state tree like [Baobab](https://github.com/Yomguithereal/baobab) to point out that there is a different approach.

## Flux basics
Though Flux has evolved with many different implementations there is still a core idea to Flux. I will not talk about the components, but just imagine they are the ones who triggers state changes and retrieves the current state of the application. Lets draw this up:

```javascript

|-----------------|                             |----------------|
| STATE CONTAINER |<-----|                   |--| ACTION CREATOR |---<
|-----------------|      |                   |  |----------------|
                         |                   |
|-----------------|      |   |------------|  |  |----------------|
| STATE CONTAINER |<---------| DISPATCHER |<----| ACTION CREATOR |---<
|-----------------|      |   |------------|  |  |----------------|
                         |         ^         |
|-----------------|      |         |         |  |----------------|
| STATE CONTAINER |<-----|         |         |--| ACTION CREATOR |---<
|-----------------|                |            |----------------|
                                   |
                                   |---------------------------------<

```

A *state container* is either a store or a reducer and with Flux you need to dispatch actions to these state containers. There are mainly two reasons why you do this dispatching. First of all the action object describes a state change in your application without actually doing it. That means this description can be stored. If you reset the state of your application and run these stored state change descriptions you will bring your application back into the exact same state (time travel debugging). The second reason is that it gives you a predictable flow. All requests for state change passes through this dispatcher and reaches all state containers.

### Store / Reducer
You need a place to contain your state. The initial release of Flux calls these state containers **stores**. They are typically created with a plain object, like:

```javascript

const TodosStore = {
  isSaving: false,
  list: []
};
```

With **Alt** this was turned into a class:

```javascript

class TodosStore {
  constructor() {
    this.isSaving = false;
    this.list = [];
  }
}
```

And with **Redux** we use a function, called a reducer. How reducers differ is that you return state changes instead of mutating the existing state. We use the [ImmutableJS](https://facebook.github.io/immutable-js/) project to create our initial state and we will continue to use this library to change our state in the examples below. This gives powerful features as we will see later.

```javascript

import Immutable from 'immutable';

const initialState = Immutable.fromJS({
  isSaving: false,
  list: []  
});

function TodosReducer(state = initialState, action) {
  return state;
}
```

So all these abstractions has the same purpose. They store state. The way they differ is how you act upon actions and change the state. Lets talk about actions first.

### Actions
With traditional Flux you use a **switch** statement and we check the *action type* to act on an action. This means that all actions reaches all stores:

```javascript

import dispatcher from './dispatcher';

const TodosStore = {
  isSaving: false,
  list: []
};

TodosStore.dispatchToken = dispatcher.register((payload) => {
  switch (payload.actionType) {
    case 'SAVING_TODO':
      TodosStore.isSaving = true;
      break;
    case 'ADD_TODO':
      TodosStore.list.push(payload.todo);
      break;
    case 'SAVED_TODO':
      TodosStore.isSaving = false;
      break;
  }
});
```

With **Alt** you actually wire the specific actions to the store. The switch statement is often seen as a verbose construct, and with good reason, it is :-) We will look at the actions implementation in the next section, but this is how you would wire it up inside the store:

```javascript

import TodosActions from './TodosActions';

class TodosStore {
  constructor() {
    this.isSaving = false;
    this.list = [];

    this.bindListeners({
      handleSavingTodo: TodosActions.SAVING_TODO,
      handleAddTodo: TodosActions.ADD_TODO,
      handleSavedTodo: TodosActions.SAVED_TODO
    });
  }
  handleSavingTodo() {
    this.isSaving = true;
  }
  handleAddTodo(todo) {
    this.list.push(todo);
  }
  handleSavedTodo() {
    this.isSaving = false;
  }
}
```

With **Redux** and a reducer you move back to using switch statements. Notice how we use Immutable JS to return completely new state from the reducer.

```javascript

import Immutable from 'immutable';

const initialState = Immutable.fromJS({
  isSaving: false,
  list: []  
});

function TodosReducer(state = initialState, action) {
  switch (action.type) {
    case SAVING_TODO:
      return state.set('isSaving', true);
    case ADD_TODO:
      return state.updateIn(['list'], list => list.push(action.todo));
    case SAVED_TODO:
      return state.set('isSaving', false);
  }
  return state;
}
```

### Action creators
The initial Flux implementation and Redux allows you to dispatch actions directly to stores/reducers, though very often you need an action creator. An action creator is a function that will do multiple dispatches to the stores. This is often related to asynchronous operations, like talking to the server.

With traditional Flux:

```javascript

import dispatcher from './dispatcher';
import ajax from 'ajax';

export default function addTodo(todo) {
  dispatcher.dispatch({actionType: 'SAVING_TODO'});
  ajax.post('/todos', todo)
    .then(() => {
      dispatcher.dispatch({actionType: 'SAVED_TODO'});
      dispatcher.dispatch({actionType: 'ADD_TODO', todo: todo});
    });
}
```

With **Alt** you always have to use an action creator.

```javascript

import ajax from 'ajax';

export default {
  savingTodo() {
    this.dispatch();
  }
  addTodo(todo) {
    this.actions.savingTodo();
    ajax.post('/todos', todo)
      .then(() => {
        this.actions.savedTodo();
        this.dispatch(todo);
      });
  }
  savedTodo() {
    this.dispatch();
  }
};
```

And with **Redux** you also create functions, though you dispatch that function. This function receives the dispatch allowing it to use it multiple times:

*saveTodo.js*
```javascript

import ajax from 'ajax';

export function addTodo() {
  return (dispatch) => {
    dispatch({type: 'SAVING_TODO'});
    ajax.post('/todos', todo)
      .then(() => {
        dispatch({type: 'SAVED_TODO'});
        dispatch({
          type: 'ADD_TODO',
          todo: todo
        });
      });    
  };
};
```

### Comparing Redux with a single state tree
So now we have taken a look at how Flux works. I stated early in this article that **Redux** really is not a single state tree. The reason I state that is because you do not define it as a single state tree, you define the branches separately in reducers and then you attach the branches later. You might say, "what is the difference?". Well, readability. Like our example above:

This is typically how you would define the actual tree with Redux:
```javascript

import todos from './reducers/todos';

{
  todos
}
```

But the tree really looks like this:

```javascript

{
  todos: {
    list: [],
    isSaving: false
  }
}
```

With Redux you do not describe the tree as a whole, but that is one of the greatest benefits of a single state tree. You can just read it and understand the complete state of your application. So lets us look more into a tree that is defined an operated on as a whole, like [Baobab](https://github.com/Yomguithereal/baobab).

```javascript

import Baobab from 'baobab';

const tree = new Baobab({
  todos: {
    isLoading: false,
    list: []
  }
})
```

So this is actually all we need when defining a Baobab tree. We create the tree by passing the object representing all the state. We do not split it into multiple state containers. If we want more state we just add it to this single object. But how do we act upon dispatched actions? Well, when you represent all the state in your application as one state container you do not need a dispatcher and actions. There is only one place to go and that is the Baobab tree. It is still as predictable as traditional Flux.

But how what about the switch statements, we have to change the state of the tree! With a Baobab tree you do not need to define custom state changing logic, you have an API to change the state.

But what about immutability then? You actually do not have to use a reducer to allow immutability, Baobab is also immutable. Think of the tree as always being the same and when you create it, `new Baobab({})`, you pass the first branch, sitting on the top of the tree. That branch can have more branches and so it grows.

So imagine our tree as:

```javascript

isSaving  list
     \     /
      \   /
       \ /
      todos
          \---/
          |   |
          |   | <- Tree
          |___|

```
When we make a change to a branch, like `tree.set(['todos', 'isSaving'], false)`, it will break the whole branch off the tree and also break off any other joined branches, in this case the *list* branch:

```javascript

isSaving         list
     \            /
      \          /
       \        /
      todos
                \---/
                |   |
                |   | <- Tree
                |___|

```

Now it replaces the branch we changed with a completely new one and then it reattaches the list branch.

```javascript

 (false)      (true)
isSaving    isSaving   list
     \           \      /
      \           \    /
       \           \  /
      todos       todos
                      \---/
                      |   |
                      |   | <- Tree
                      |___|

```


What this boils down to is that you do not need a dispatcher and actions with a single state tree, and you change the state of the tree with imperative programming. An example being: `tree.set(['todos', 'isSaving'], true)`. You might have heard that imperative programming is out and the new thing is functional programming. And yeah, it is really great to see all the projects evolving around functional programming, but that does not mean you should never do imperative programming. It is all about the right tool for the job.

#### Tree basics
So let us move back to the beginning of this article and look at how the state changes occur with a single state tree.

```javascript

             |----------|
          |--| FUNCTION |
          |  |----------|
          |
|------|  |  |----------|
| TREE |<----| FUNCTION |
|------|  |  |----------|
          |
          |  |----------|
          |--| FUNCTION |
             |----------|
```

There is no dispatcher and no actions. We just have normal functions that changes the state of the tree.

#### Defining the tree
I already showed you this, but let us recap. To create the state of our application we:

```javascript

import Baobab from 'baobab';

export default new Baobab({
  todos: {
    isSaving: false,
    list: []
  }
});
```

Again, we do not split our state definition into different files and create logic for changing the state. We just describe it "as is".

#### Actions and action creators
Since there are no actions using a single state tree you do not really need action creators either. What you need though is to change the state of the state tree. And the way you do that is:

```javascript

import tree from './tree';
import ajax from 'ajax';

function addTodo(todo) {
  tree.set(['todos', 'isSaving'], true);
  ajax.post('/todos', todo)
    .then(() => {
      tree.set(['todos', 'isSaving'], true);
      tree.push(['todos'], todo);
    });
}
```

With a single state tree like **Baobab** you use imperative programming to do your state changes, just like you do normally in JavaScript. The tree is still immutable though, so any changes to the branches of the tree will replace the whole branch, not just the value on the branch. This makes it possible to do shallow checking of values when rendering React components, making it super fast.

### Summary
There are many things happening in the JavaScript community now and functional programming and even functional reactive programming is really starting to get a foothold. That is great! That said, functional concepts does not necessarily mean better in all scenarios. We have been doing imperative programming for a long time, for better and worse and there are features of the imperative style that is completely lost when replaced by functional approaches. In my opinion, one of those features is readability of changing the state of your application.

And you might say now that the function above is horrible to test or you can not get time travel with this approach. And yeah, you are right. But if you imagine this state tree being your database and you watch this video [Turning the database inside out](https://www.youtube.com/watch?v=fU9hR3kiOK0), you will quickly realize that it is not the database itself that needs to handle these features, you need a layer in front of it.

One such layer is [cerebral](http://christianalfoni.com/cerebral) and it is functional. With Cerebral you use a functional approach to define the flow of state changes in your application, which the functional approach is a lot better at than the imperative approach. An example being:

```javascript

const items = [{title: 'foo', isAwesome: true}, {title: 'bar', isAwesome: false}];

// functional
const isAwesome = (item) => item.isAwesome;
const byTitle = (item) => item.title;

const awesomeItemTitles = items.filter(isAwesome).map(byTitle);

// imperative
const awesomeItemTitles = [];
items.forEach((item) => {
  if (item.isAwesome) {
    awesomeItemTitle.push(item.title);
  }
});
```

When defining flow the functional approach gives you something extremely powerful. It gives you the power to describe what is happening to you application without reading implementation details. The line `items.filter(isAwesome).map(byTitle)` tells you what happens, but the imperative example requires you to read all the implementation details to understand it.

With Cerebral you get the same kind of functional flow, though it allows you to build more complex flows like combining asynchronous flows with synchronous flows, parallel asynchronous flows and even conditional flows. Since your are just referencing functions that operates on its argument you get the testability you want and even time travel debugging.

Thanks for reading through and please comment below if you think I am just being ignorant, do not get the point of Redux at all or maybe you see that I have some points. Nevertheless, I am not stating that Flux or Redux are bad approaches, I am just stating that there are other approaches too!
