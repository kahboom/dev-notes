# React Testing

- From easiest to most difficult to write: snapshot, unit, integration, E2E
- From least to most important*: snapshot, unit, integration, E2E 

_* where the level of importance is directly proportional to the confidence you have that your tests are reflective of your app working as intended_


## Testing Pyramid

- A visual aid used to find the right balance of test types.
- The bulk of your tests are unit tests at the bottom of the pyramid. As you move up the pyramid, your tests gets larger, but at the same time the number of tests (the width of your pyramid) gets smaller.
- Google suggests a 70/20/10 split: 70% unit tests, 20% integration tests, and 10% end-to-end tests. The exact mix will be different for each team, but in general, it should retain that pyramid shape. Try to avoid these anti-patterns:
  - **Inverted pyramid/ice cream cone**. Mostly end-to-end tests, few integration tests, and even less unit tests. 
  - **Hourglass**. A lot of unit tests, then many end-to-end tests where integration tests should be used. The hourglass has many unit tests at the bottom and many end-to-end tests at the top, but few integration tests in the middle. 

![Google Testing Pyramid](https://2.bp.blogspot.com/-YTzv_O4TnkA/VTgexlumP1I/AAAAAAAAAJ8/57-rnwyvP6g/s1600/image02.png)


---

## Unit testing

- Fast, small, and reliable. Easy to debug.
- [Pure functions](https://reactjs.org/docs/components-and-props.html#props-are-read-only) are the easiest to test, because there are no side effects.

### Unit testing tools

- [Enzyme](https://airbnb.io/enzyme/)
- [Jest](https://jestjs.io/)
  - Comes with create-react-app; built by Facebook.
- [Karma](https://karma-runner.github.io/latest/index.html) and [Mocha](https://mochajs.org/) are often used in tandem, where Mocha is a testing framework and Karma is a test runner.
- [react-testing-library](https://github.com/testing-library/react-testing-library)
  - Does not replace Jest, works with it (or Mocha).
  - Replaces something like Enzyme.

- The React team recommends using react-testing-library. Facebook uses Jest to test.

Assume in all unit test examples we are using Jest and react-testing-library.

### Jest & react-testing-library

You can run Jest directly from the command line like this: `jest my-test --notify --config=config.json`

Here is an example taken from [examples/react-testing-library](https://github.com/facebook/jest/tree/master/examples/react-testing-library) of a component for a simple checkbox that swaps between two labels:

```
// CheckboxWithLabel.js

import React from 'react';

export default class CheckboxWithLabel extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isChecked: false};

    // bind manually because React class components don't auto-bind
    // http://facebook.github.io/react/blog/2015/01/27/react-v0.13.0-beta-1.html#autobinding
    this.onChange = this.onChange.bind(this);
  }

  onChange() {
    this.setState({isChecked: !this.state.isChecked});
  }

  render() {
    return (
      <label>
        <input
          type="checkbox"
          checked={this.state.isChecked}
          onChange={this.onChange}
        />
        {this.state.isChecked ? this.props.labelOn : this.props.labelOff}
      </label>
    );
  }
}
```

And the test:

```
// __tests__/CheckboxWithLabel-test.js
import React from 'react';
import {render, fireEvent, cleanup} from 'react-testing-library';
import CheckboxWithLabel from '../CheckboxWithLabel';

// automatically unmount and cleanup DOM after the test is finished.
afterEach(cleanup);

it('CheckboxWithLabel changes the text after click', () => {
  const {queryByLabelText, getByLabelText} = render(
    <CheckboxWithLabel labelOn="On" labelOff="Off" />,
  );

  expect(queryByLabelText(/off/i)).toBeTruthy();

  fireEvent.click(getByLabelText(/off/i));

  expect(queryByLabelText(/on/i)).toBeTruthy();
});
```


The test function takes in two parameters:
- The first parameter is a message telling us whether the test passed or not, which will be shown in the test output.
- The second parameter is an arrow function that will contain our test

- The `test.tsx` or `test.js` extension is important because Jest automatically looks for files with this extension when finding tests to execute. Note that if our tests don't contain any JSX, we could use a `test.ts` extension.


### Tips for Writing Unit Tests

>The more your tests resemble the way your software is used, the more confidence they can give you. — Kent C. Dodds 👋

- Things to consider when testing:
  1. Will this test break when there's a mistake that would break the component in production?
  2. Will this test continue to work when there's a fully backward compatible refactor of the component?

- If your test uses `instance()` or `state()`, know that you're testing things that the user couldn't possibly know about or even care about, which will take your tests further from giving you confidence that things will work when your user uses them.
- Do not test implementation details. It can give a false sense of security, and will slow down refactoring. Tests will need to be updated after refactoring and can quickly become obsolete. You should very rarely have to change tests when you refactor code.


### Testing React hooks

- Tools for testing hooks: [react-hooks-testing-library](https://github.com/testing-library/react-hooks-testing-library)
- Creates a test harness for hooks that handles running them within the body of a function component.
- You don't need to explicitly test all hooks. Use cases:

| When to use this library | When not to use this library|
| ------------- |-------------|
| - You're writing a library with one or more custom hooks that are not directly tied a component | - Your hook is defined along side a component and is only used there |
| - You have a complex hook that is difficult to test through component interactions | - Your hook is easy to test by just testing the components using it |

Example with `react-hooks-testing-library`:

`useCounter.js`
```typescript
import { useState, useCallback } from 'react'

function useCounter() {
  const [count, setCount] = useState(0)

  const increment = useCallback(() => setCount((x) => x + 1), [])

  return { count, increment }
}
export default useCounter
```

`useCounter.test.js`

```typescript
import { renderHook, act } from '@testing-library/react-hooks'
import useCounter from './useCounter'

test('should increment counter', () => {
  const { result } = renderHook(() => useCounter())

  act(() => {
    result.current.increment()
  })

  expect(result.current.count).toBe(1)
})
```

### Mocking dependencies

- If we mock too much, we aren't really testing our app.
- Good use case of mocking with unit tests: REST APIs
- Whether or not to mock third party dependencies, you have to ask yourself if you would want to know whether your app is broken regardless of the reason being due to the third party app or not. In most cases, people will want to know. This is a good argument to _not_ mock these dependencies, because it will more closely resemble the app itself.

### Code coverage

- Help identify areas that need more unit tests.
- Code coverage tools: [Istanbul](https://istanbul.js.org/) comes built into Jest
- Aiming for 100% code coverage is not only unrealistic, but also can encourage testing things that do not need to be tested and can be quite costly. This is only more possible on very small projects.


### Enzyme & shallow rendering

- Enzyme introduces a concept of **shallow rendering**, which is [controversial](https://kentcdodds.com/blog/why-i-never-use-shallow-rendering) at the moment.
- The controversy is that shallow rendering provides convenience that sacrifices test reliability. With shallow rendering, it is possible to refactor a component's implementation and and tests would still pass.
- Shallow rendering takes the result of the given component's render method and returns a wrapper object with some utilities for traversing the JS object.
- It renders your component one level deep, making it faster than full DOM rendering.
- This means it doesn't run lifecycle methods (since only the React elements are available), it doesn't allow you to actually interact with DOM elements (because nothing's actually rendered), and it doesn't actually attempt to get the React elements that are returned by your custom components.


## Integration testing

- Having some unit tests to verify that pieces work in isolation is important, but is useless if you don't also verify that they work together.
- Integration tests strike a great balance on the trade-offs between confidence and speed/expense. This is why it's advisable to spend most of the effort there.
- 

## E2E testing

- Simulate real user scenarios.
- Slower and more expensive than unit tests, but provide more value and confidence that your app is working as expected.
- E2E testing tools:
  - [Cypress](https://www.cypress.io/) - Runs without Selenium
  - [Selenium](https://www.seleniumhq.org/) & [Selenium WebDriver](https://www.seleniumhq.org/projects/webdriver/)
- The key is writing less E2E tests, but well-written ones.



## Snapshot testing

> A typical snapshot test case for a mobile app renders a UI component, takes a snapshot, then compares it to a reference snapshot file stored alongside the test. The test will fail if the two snapshots do not match: either the change is unexpected, or the reference snapshot needs to be updated to the new version of the UI component.

- Can be used to implement tests quicker.
- Can be used for pure functions and components.
- The snapshot file gets committed along with code changes.
- Does tend to focus on implementation details, but this [can be useful](https://kentcdodds.com/blog/effective-snapshot-testing) sometimes.
- Because they're more integrated and try to serialize an incomplete system (e.g. one with some kind of side effects: from browser/library/runtime versions to environment to database/API changes), they will tend to have high false-negatives (failing test for which the production code is actually fine and the test just needs to be changed).
- False negatives quickly erode the team's trust in a test to actually find bugs and instead come to be seen as a chore on a checklist they need to satisfy before they can move on to the next thing.
- Good for: Error messages and logs, styling (even E2E tests can miss this)
- Avoid huge snapshots.
- Always see if you can actually change it from a snapshot to a more explicit assertion (because you probably can).
- Tools: [Jest Snapshot Testing](https://facebook.github.io/jest/docs/en/snapshot-testing.html)

Example of a component that renders hyperlinks:

```javascript
// Link.react.js
import React from 'react';

const STATUS = {
  HOVERED: 'hovered',
  NORMAL: 'normal',
};

export default class Link extends React.Component {
  constructor(props) {
    super(props);

    this._onMouseEnter = this._onMouseEnter.bind(this);
    this._onMouseLeave = this._onMouseLeave.bind(this);

    this.state = {
      class: STATUS.NORMAL,
    };
  }

  _onMouseEnter() {
    this.setState({class: STATUS.HOVERED});
  }

  _onMouseLeave() {
    this.setState({class: STATUS.NORMAL});
  }

  render() {
    return (
      <a
        className={this.state.class}
        href={this.props.page || '#'}
        onMouseEnter={this._onMouseEnter}
        onMouseLeave={this._onMouseLeave}
      >
        {this.props.children}
      </a>
    );
  }
}
```

Snapshot test:

```javascript
// Link.react.test.js
import React from 'react';
import Link from '../Link.react';
import renderer from 'react-test-renderer';

test('Link changes the class when hovered', () => {
  const component = renderer.create(
    <Link page="http://www.facebook.com">Facebook</Link>,
  );
  let tree = component.toJSON();
  expect(tree).toMatchSnapshot();

  // manually trigger the callback
  tree.props.onMouseEnter();
  // re-rendering
  tree = component.toJSON();
  expect(tree).toMatchSnapshot();

  // manually trigger the callback
  tree.props.onMouseLeave();
  // re-rendering
  tree = component.toJSON();
  expect(tree).toMatchSnapshot();
});
```

When it is run with `yarn test` or `jest`, a file gets created. In this case, it'd be named something like `__tests__/__snapshots__/Link.react.test.js.snap`.

## References

- [Effective React Testing @ JazzCon 2018](https://www.youtube.com/watch?v=Eakp29J38YA) by Jeremy Fairbank
- [Effective Snapshot Testing](https://kentcdodds.com/blog/effective-snapshot-testing) by Kent C. Dodds
- [Jest Testing React Apps](https://jestjs.io/docs/en/tutorial-react)
- [No To More E2E Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html) by Mike Wacker, Google Testing Blog
- [Painless JavaScript Testing with Jest](https://learning.oreilly.com/videos/advanced-design-patterns/9781788838931/9781788838931-video4_2?autoplay=false) by Michele Bertoli, Advanced Design Patterns with React
- [TestPyramid](https://martinfowler.com/bliki/TestPyramid.html) by Martin Fowler
- [The Testing Trophy](https://twitter.com/kentcdodds/status/960723172591992832/photo/1?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E960723172591992832&ref_url=https%3A%2F%2Fkentcdodds.com%2Fblog%2Fwrite-tests) by Kent C. Dodds
- [Unit Testing with Jest](https://learning.oreilly.com/library/view/learn-react-with/9781789610253/57b8659d-c830-4fe6-bf35-608c5b6ce892.xhtml) by Carl Rippon, Learn React with TypeScript 3
- [What is React Testing Library?](https://www.youtube.com/watch?v=JKOwJUM4_RM&feature=youtu.be) by Scott Tolinski
- [Why I Never Use Shallow Rendering](https://kentcdodds.com/blog/why-i-never-use-shallow-rendering) by Kent C. Dodds
- [Write tests. Not too many. Mostly integration.](https://kentcdodds.com/blog/write-tests) by Kent C. Dodds