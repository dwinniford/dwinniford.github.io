---
layout: post
title:      "Getting started with Jest"
date:       2020-08-13 14:17:36 +0000
permalink:  getting_started_with_jest
---


Although testing doesn't always sound like the most enjoyable part of building applications I was happy to discover a wealth of great documentation on using Jest to test React apps. (Nothing gets me pumped up the same as great documentation...)  I tried to avoid most of the blogs and focus on learning from the official docs.  Here's what I've digested from these resources:

* [Create React App](http://create-react-app.dev/docs/running-tests)
* [Jest docs](http://jestjs.io/docs/en/tutorial-react)
* [React docs](http://reactjs.org/docs/testing.html)
* [Free Code Camp post](http://www.freecodecamp.org/news/testing-react-hooks/)
* [React Test Library](http://testing-library.com/docs/react-testing-library/intro)

## Tips and Thoughts from Create React App

I decided to use Jest because it comes configured with create-react-app.  I was a little skeptical because I have only ever interacted with tests using chai and mocha.  However Jest claims its been updated so chai and mocha shouldn't be necessary anymore.

Organizationally its best to put your test file in the same folder as the component it is testing so that importing will be simple.  For example: App.js and App.test.js.

Basic syntax:

Use `it()` or `test()`
```
test('string describing the test', callback)
```
Use `expect()` and a matcher to make an assertion
```
expect(valueBeingTested).matcher(correctValue)
```
If you're wondering, `it()` and `test()` [are exactly the same](http://jestjs.io/docs/en/api.html#testname-fn-timeout).
Check out [matchers in the docs.](http://jestjs.io/docs/en/expect.html#content)

A great starting point is to write basic 'smoke' tests.  These are really easy to write but provide a lot of value.  (Test that a component renders without crashing). Here's the example from create react app:

```
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

it('renders without crashing', () => {
  const div = document.createElement('div');
  ReactDOM.render(<App />, div);
});
```

*npm test only runs tests on code changed since the last commit but you can easily force into to rerun all the tests.*

## Tips from a Free Code Camp post

At this point I wanted to read a little more about different types of tests and this blog was a great resource.  Here's 4 kinds of tests:
1. Unit - Tests components in isolation, usually with shallow rendering.
2. Integration - Tests how multiple components interact.
3. End-to-end - Test involving multiple components, processes and steps.  Tests the entire flow of a process in the app, usually in a simulated browser.
4. Snapshot - Tests if the code is different than in a snapshot file.

The author of this post is of the opinion that its best to focus on integration test.

> Many integration tests. No snapshot tests. Few unit tests. Few e to e tests.
> 

Using `getByText()` is a great way to focus on integration tests that avoid testing implementation details.


## Tips from the React docs

Interestingly React roughly divides tests into only two categories:

> * Rendering component trees in a simplified test environment and asserting on their output.
> 
> * Running a complete app in a realistic browser environment (also known as “end-to-end” tests).
> 

Here's some other thoughts from React:
1. The difference between unit testing and integration testing is blurry.  (To me, their first category sounds like a combination of these.)
2. Don't test implementation details. Test how the user will actually interact with the interface. 
3. [Setup and tear down](http://reactjs.org/docs/testing-recipes.html#setup--teardown) is important.  Move creating/removing the dom element into a before/after hook.
4. Use [act()](http://reactjs.org/docs/testing-recipes.html#act) to make sure all updates to the dom are completed before assertions are made.  However [React Testing Library](http://testing-library.com/docs/react-testing-library/intro) helpers are already wrapped in act.


Here's an integration test I wrote combining most of these tips:

```
import React from 'react';
import ReactDOM, {unmountComponentAtNode} from 'react-dom'
import { render, fireEvent } from '@testing-library/react';
import App from './App';
import { Provider } from 'react-redux'
import store from './redux/store.js'

// handle setup/teardown with before and after hooks
let container = null 

beforeEach(() => {
    container = document.createElement('div')
    document.body.appendChild(container)
})

afterEach(() => {
    unmountComponentAtNode(container)
    container.remove()
    container = null
})


// Test that the dashboard link redirects to the 'Build a Recipe' page.
test('it redirects to build a recipe on dashboard link click', () => {
  // 1. Mount app. (Wrap the app in Provider so that it doesn't through an error.)
  const {getByText} = render(<Provider store={store}><App /></Provider>, container)
  // 2. Check for the presence of the initial title.
  expect(getByText('Daily Dozen Home')).toBeInTheDocument()
  // 3. Find and click the 'Dashboard' button.
  fireEvent.click(getByText('Dashboard'))
  // 4. Check for the presence of the new title.
  expect(getByText('Build a Recipe')).toBeInTheDocument()

})
```


