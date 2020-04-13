# Unit Testing

https://jestjs.io

## Jest Setup


```bash
// Project setup
$ npm run test -- --watch

// Global setup (optional)
$ npm install -g jest
$ jest --watch
```


## Directory Tree


```
root
 ├── __mocks__     // mocks
 └── src
      ├── components
      │      └── __test__    // tests components
      └── scene
             └── __test__    // tests scenes
```

**Test folder** - This is where all tests should be, which will automatically detected by Jest

**Mocks folder** - Mock functions or libraries (to return what you want) for your tests to reduce the test scope 


---

## Unit Test Example

```jsx
import React from 'react';
import renderer from 'react-test-renderer';
import { TouchableOpacity } from 'react-native';
import CollapsibleContainer from '../../CollapsibleContainer/index';

describe('<CollapsibleContainer>', () => {
  it('toggles isCollapsed when pressed (starts with collapsed)', () => {
    
    let initialState = true
    
    const inst = renderer.create(<CollapsibleContainer />);
    
    const touchable = inst.root.findByType(TouchableOpacity)
    expect(inst.root.instance.state.isCollapsed).toBe(initialState)
    touchable.props.onPress();
    expect(inst.root.instance.state.isCollapsed).toBe(!initialState)
  });
});
```


# Writing Tests

Create a new file in the following directory  

`/src/scene/__test__/demo/demo.test.js` 

> Note: Directory in test folders should always be organized based on the directory of original file for easier access.

Example:

```
/src/scene/AuthCheck/AuthCheck.test.js
/src/scene/__test__/AuthCheck/AuthCheck.test.js 
```

> You can skip to the end and see the completed demo.test.js by [clicking here](#demo.test.js)


## 1. Creating a Test Suite

Then, import `react` and `react-test-renderer` following with your first test suite

```js
// demo.test.js
import React from 'react';

describe('My Test Suite', () => {
    it('should run test flawlessly', () => {
        // Your tests should be here
    })

    it('should fail the test', () => {
        // Your tests should be here
    })
})
```
 `describe()` is for creating a test suite. A test suite can contain multiple tests depending on the test scope, **it is advised to name it as the component you're testing** for better referral.

https://jestjs.io/docs/en/api#describename-fn

`it()` is for describing the test, you can treat it as a sentence that starts with 'it'. Tests should be contained in this function.

https://jestjs.io/docs/en/api#testname-fn-timeout

> Note: `it()` is an alias of `test()`, hence using either one will work the same.



## 2. Writing your first test

```jsx
let total = 1 + 1

it('should run test flawlessly', () => {
    expect(total).toBe(2)
})

it('should fail the test', () => {
    expect(total).toBe(3)
})
```

`expect()` as what the word means, checking if the retrieved data matches what is expected.

`.toBe` is just one of many ways to compare, the rest can be found in the API docs in the following link https://jestjs.io/docs/en/expect 

in short,

`expect(received).toBe(expected)`


## 3. Running the test

Run the following command in the root folder

```bash
$ npm run test -- --watch
```

You will see the following menu

![jest-menu](https://i.imgur.com/Bu42Mqo.png)

Press `'p'` to filter by filename, and type in `demo`.

![search-test](https://i.imgur.com/AvCzdj2.png)

And it will run the test & will show the following

![test-result](https://i.imgur.com/voCnKDB.png)

Jest clearly shows the test suite, test name and what the expected result should be clearly from the console.


## 4. Importing a Container/Component
First, import these two things:
1. React Test Renderer
2. Container that contains the function to test.

> Note: The following is **only for containers that uses Redux**, you can skip this part and import normally if you're testing components/containers that does not use Redux

```jsx
// demo.test.js
import React from 'react';

import renderer from 'react-test-renderer';
import { MultiCityTravelEditContainer } from '../Travel/MultiCityTravelEditContainer';
```
This will return an error because the container is exported with Redux. To avoid that, we will need to export the class directly (skipping Redux). 

```jsx
// MultiCityTravelEditContainer.js

// imports...

export class MultiCityTravelEditContainer extends...
```
Now we can import the Container with no issue.

## 5. Creating an instance

https://reactjs.org/docs/test-renderer.html

> An instance consists of all the states, props and functions in the component/container. This allows us to run functions and check if the state has been modified.

```jsx
// demo.test.js

...

describe('My Test Suite', () => {
    // renderer is imported from 'react-test-renderer
     const inst = renderer
        .create(<MultiCityTravelEditContainer />)
        .getInstance();

})

```

> Instance are usually created in test suites so it can be used by all of the tests.

For Redux, this will show an error because `travelReducer` props needs be handled.

## 6. Mocking Reducer (Redux only)

Mocking reduce is just simply as passing in the reducer as an object.

```jsx
let travelReducerMock = {
  travelReducer: 'test', // or anything to be tested
};

const inst = renderer
    .create(<MultiCityTravelEditContainer 
        travelReducer={travelReducerMock} />)
    .getInstance();
```


## 7. Using instances
To check that it is working, you can use `console.log` to show the states, functions and props of the instance. You can even use `setState()`

```jsx
const inst = renderer
    .create(<MultiCityTravelEditContainer 
        travelReducer={travelReducerMock} />)
    .getInstance();

console.log(inst) // shows everything

console.log(inst.state) // shows all states
console.log(inst.onChangeValue('params')) // returns what the function should
inst.setState({ data: 'newData' })
```

The console should display all the data given but **somehow the test fails**

```
TypeError: this._picker.setNativeProps is not a function
```

This is because the container is dependent on `setNativeProps` used by `react-native-datepicker`, which is imported from an external library.


## 8. Mocking Functions

https://jestjs.io/docs/en/mock-functions.html

Mocking a library is easy when the library is supported by Jest, you can just add the following line below your imports.

```jsx
// demo.test.js

// import..

jest.mock('react-native-datepicker');
```

> `jest.mock()` automatically set all exports of a module to the Mock Function `jest.fn()`, which basically does the following

```js
export const setNativeProps = jest.fn()
```

## 9. Mock functions manually

https://jestjs.io/docs/en/manual-mocks

```
root
 ├── __mocks__     // mocks
 └── src
      ├── components
      │      └── __test__    // tests components
      └── scene
             └── __test__    // tests scenes
```




Notice the `__mocks__` folder at root directory.
It needs to be adjacent to the libary in `node_modules`

Example:
```
__mocks__/react-native-device-info.js

node_modules/react-native-device-info/
```
Next, we need to export the mocked function from there, which will automatically apply to all the tests (which can be bad sometimes)

```jsx
// react-native-device-info.js

export default {
  getIPAddress: () => {
    return 0;
  },
  getReadableVersionSync: () => {},
  getBundleIdSync: () => {},
  getUniqueId: jest.fn(),
};

```


# `demo.test.js`

```jsx
// demo.test.js
import React from 'react';
import renderer from 'react-test-renderer';
import { MultiCityTravelEditContainer } from '../Travel/MultiCityTravelEditContainer';

jest.mock('react-native-datepicker');

describe('My Test Suite', () => {

  let travelReducerMock = {
    travelReducer: 'test',
  };
  const inst = renderer
  .create(<MultiCityTravelEditContainer travelReducer={travelReducerMock} />)
  .getInstance();  let total = 1 + 1

  console.log(inst) // shows all
  console.log(inst.state) // shows all states
  console.log(inst.onChangeValue('params')) // returns what the function should

inst.setState({ data: 'newData' })

  it('should run test flawlessly', () => {
      expect(total).toBe(2)
  })

  it('should fail the test', () => {
      expect(total).toBe(2)
  })
})
```

 
