[cerebral-immutable-store](https://github.com/christianalfoni/cerebral-immutable-store) - @christianalfoni **/** [immutable-store](https://github.com/christianalfoni/immutable-store) - @christianalfoni

`npm install cerebral-immutable-store`


- An immutable state tree
- Allows you to map state to new state
- Supports recording signals in Cerebral

```javascript

// controller.js

import Controller from 'cerebral';
import Model from 'cerebral-immutable-store';
import request from 'superagent';

// The initial state of the application
const state = {
  isLoading: false,
  user: null,
  error: null
};

// Instantiate the model
const model = Model(state);

// Any default input you want each action to receive. In this
// example we pass in superagent ajax library
const defaultInput = {
  utils: {
    request: request
  }
};

// Instantiate the controller
export default Controller(model, defaultInput);
```