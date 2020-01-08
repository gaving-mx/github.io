# UNIT TEST

## What

In computer programming, unit testing is a method by which **individual units** of source code, sets of one or more computer program modules together with associated control data, usage procedures, and operating procedures are tested to determine if they are fit for use.

[https://en.wikipedia.org/wiki/Unit_testing](https://en.wikipedia.org/wiki/Unit_testing)

```
_.chunk(['a', 'b', 'c', 'd'], 2); //[['a', 'b'], ['c', 'd']]
_.chunk(['a', 'b', 'c', 'd'], 3); //[['a', 'b', 'c'], ['d']]

```

## Pass or fail ?

How to make sure _.chunk runs correctly? 
Write a function called matcher to log result in console. If all cases passed, then we say this function is fit to use.


```
matcher('split an array into groups the length of size.', _.chunk(['a', 'b', 'c', 'd'], 2), [['a', 'b'], ['c', 'd']]);
matcher('if array can't be split evenly, the final chunk will be the remaining elements.', _.chunk(['a', 'b', 'c', 'd'], 3), [['a', 'b', 'c'], ['d']]);

```

```
function matcher(desc, result, expect) {
    if (isEqual(result, expect)) {
        console.info(desc + '- pass');
    } else {
        console.error(desc + '- fail.');
        console.error('expect: ' + expect + ' received:' + result);
    }
}

```

## First test

What a basic unit test case looks like?

```
describe('add().', () => {
    it('Add should get the sum of two numbers.', () => {
        expect(add(1,2)).toBe(3);
    });
    it('Negative numbers should work too.', () => {
        expect(add(1, -2)).toBe(-1);
    });
});

```

## Basic concepts

* Suites
* Specs
* Expectations
* Matchers

### describe()

The describe function is for **grouping related specs**, typically each test file has one at the top level. The string parameter is for naming the collection of specs, and will be concatenated with specs to make a spec's full name. Calls to describe can be **nested**.

#### Nested Describe

This allows a suite to be composed as a tree of functions

```
describe('Util.', () => {
    describe('add().', () => {
        ...
    });
    describe('minus().', () => {
        ...
    });
    ...
});

```

### it()

Specs are defined by calling the function it. The string is the title of the spec and the function is the spec, or test. A spec contains **one or more expectations** that test the state of the code. An expectation is an assertion that is either true or false. A spec with **all true** expectations is a passing spec. A spec with **one or more false** expectations is a failing spec.

### A spec with several expectations

```
describe('getFormatNumber().', () => {
    it('get currency code should work.', () => {
        expect(getFormatNumber()).toBe('CA$ ');
        expect(getFormatNumber('CNY')).toBe('CN&#xA5; ');
    });
});

```

### expect()

Expectations are built with the function expect which takes a value, called the actual. It is **chained with a Matcher function**, which takes the expected value.

### Matchers

Each matcher implements a boolean comparison between the actual value and the expected value. It is responsible for reporting if the expectation is true or false.

[https://facebook.github.io/jest/docs/en/expect.html](https://facebook.github.io/jest/docs/en/expect.html)

what else do we have ?

## Setup and Teardown

To help a test suite DRY up any duplicated setup and teardown code, we should use the global **beforeEach, afterEach, beforeAll, and afterAll** functions.

## Spy

A spy can stub any function and tracks calls to it and all arguments.

[https://facebook.github.io/jest/docs/en/mock-function-api.html](https://facebook.github.io/jest/docs/en/mock-function-api.html)

```
describe('Aop and spy.', () => {
    let mockFn;
    beforeEach(() => {
        mockFn = jest.fn();
        mockFn(123);
    });
    it('spy tracks calls.', () => {
        expect(mockFn).toHaveBeenCalled();
        expect(mockFn).toHaveBeenCalledWith(123);
    });
})

```

# DEMO TIME

## Frameworks

* [Jasmine](https://jasmine.github.io/)
* [Mocha](https://mochajs.org/)
* [Jest](https://facebook.github.io/jest/)

## React Project

In react projects, we need to cover components, models, services. The test coverage needs to reach 80%.

### Model

#### Test Name Space

```
import Model from '';
import { MR_COMMON } from '';

it('namespace should right.', () => {
    expect(Model.namespace).toEqual(MR_COMMON);
});

```

#### Test Reducer

Treat reducer as normal function.

```
[Actions.SET_STATE](state, { payload }) {
  return { ...state, ...payload };
}

```

```
it('set state.', () => {
  let state = {};
  const payload = {
    id: 1,
    name: 'abc'
  };
  const newState = Model.reducers[fn](state, { payload });
  expect(newState.id).toEqual(payload.id);
  expect(newState.name).toEqual(payload.name);
  expect(state).not.toBe(newState);
});

```

#### Effect

Treat effect as generator https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator)

```
*[Actions.GET_CUSTOMERS](payload, { call, put }) {
  const { data: { data } } = yield call(CommonService.getCustomers);
  yield put({ type: Actions.SET_STATE, payload: { customerList: data } });
}

```

```
it('get customers.', () => {
  call.mockReturnValue({data: {data: []})
  const g = Model.effects[Actions.GET_CUSTOMERS](null, { call, put });
  invokeGenerator(g)
  expect(call).toHaveBeenCallWith(CommonService.getCustomers);
  const { type, payload } = getTypeAndPayload(put);
  expect(type).toEqual(Actions.SET_STATE);
  expect(payload).toEqual({ customerList: [] });
});

```

## Enzyme

Enzyme is a JavaScript Testing utility for React that makes it easier to assert, manipulate, and traverse your React Components' output.

Enzyme is not a unit testing framework. It does not have a test runner or an assertion library.

[http://airbnb.io/enzyme/](http://airbnb.io/enzyme/)

[https://www.codementor.io/vijayst/unit-testing-react-components-jest-or-enzyme-du1087lh8](https://www.codementor.io/vijayst/unit-testing-react-components-jest-or-enzyme-du1087lh8)

The [shallow](http://airbnb.io/enzyme/docs/api/shallow.html) function in Enzyme does a shallow rendering to the DOM. Shallow rendering does not render any components nested within the Welcome component.

The [find](http://airbnb.io/enzyme/docs/api/ShallowWrapper/find.html) method in Enzyme accepts jQuery-like selectors to retrieve a node from the DOM tree.

### Testing the structure

We will create a List component which displays a list of items in tabular format. The React component under test is shown below.

### Testing the behavior

We will create an Add component which has a form element with an input text and a button. We want to test the behavior of the component when the button is clicked. The code for Add component is shown below.

The Add component calls the handleAdd method when the button is clicked. The handleAdd method calls the onAdd function set in the props.

We will use jest.fn() to create a mock function. We will supply the mock to the onAdd prop of our component.

When the button is clicked, our mock should be called with the appropriate arguments.

## Questions

* [where to write test code?](https://facebook.github.io/jest/docs/en/configuration.html#testmatch-array-string)
* [how to run test?](https://facebook.github.io/jest/docs/en/cli.html)
* [where to see coverage?](https://facebook.github.io/jest/docs/en/cli.html#coverage)

# Thanks
