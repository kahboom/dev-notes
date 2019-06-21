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

## React Unit Testing

- Fast, small, and reliable. Easy to debug.
- [Pure functions](https://reactjs.org/docs/components-and-props.html#props-are-read-only) are the easiest to test, because there are no side effects.

### Unit Testing Tools

- [Enzyme]
  - Testing framework created by Airbnb.
  - Focuses more on props and checking state.
  - Returns React components in memory or to the DOM while providing a jQuery-like API for traversing the React component tree.
- [Jest]- Testing framework created by Facebook. Comes with create-react-app, though if you want to create snapshot tests, you'd need to run `yarn add --dev react-test-renderer`.
- [Karma](https://karma-runner.github.io/latest/index.html) and [Mocha](https://mochajs.org/) are often used in tandem, where Mocha is a testing framework and Karma is a test runner.
- [react-testing-library]
  - Does not replace Jest, works with it (or Mocha). It is a library, not a framework or test runner.
  - Replaces something like Enzyme.
  - Returns HTML elements.
  - Queries functions by text content (visible on page) or HTML data attributes (for when fetching by text is not possible or practical).
  - Focuses more on the DOM and what gets rendered, interacts with them.
  - Built on top of [DOM Testing Library](https://testing-library.com/docs/dom-testing-library/intro).
- [ReactTestUtils](https://reactjs.org/docs/test-utils.html)
  - Convenience utilities provided OOTB by React, to be used in combination with your testing framework.
- The React team recommends using react-testing-library. Assume in all unit test examples we are using Jest, ReactTestUtils, and react-testing-library for unit tests.

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


### Testing React Hooks

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

### Mocking Dependencies

- If we mock too much, we aren't really testing our app.
- Good use case of mocking with unit tests: REST APIs
- Whether or not to mock third party dependencies, you have to ask yourself if you would want to know whether your app is broken regardless of the reason being due to the third party app or not. In most cases, people will want to know. This is a good argument to _not_ mock these dependencies, because it will more closely resemble the app itself.

### Code Coverage

- Help identify areas that need more unit tests.
- Aiming for 100% code coverage is not only unrealistic, but also can encourage testing things that do not need to be tested and can be quite costly. This is only more possible on very small projects.
- Code coverage tools: [Istanbul] comes built into Jest.
  - To use Instanbul from Jest, you just need to add the `--coverage` flag to the Jest command in the npm script, or you can create a configuration section for Jest in the `package.json` file, and set the `collectCoverage` option to `true` like this: `{ "jest": { "collectCoverage": true } }`.

After configuration, you can run `npm test` and see a different output in the console.

- The first column shows how many statements are covered.
- The second column shows the different branches of conditional statements.
- The third measures the functions that have been tested, and the fourth column shows the lines of code covered by tests.
- The last column would tell you which lines are uncovered.



### Enzyme & Shallow Rendering

- Enzyme introduces a concept of **shallow rendering**, which is [controversial](https://kentcdodds.com/blog/why-i-never-use-shallow-rendering) at the moment.
- The controversy is that shallow rendering provides convenience that sacrifices test reliability. With shallow rendering, it is possible to refactor a component's implementation and and tests would still pass.
- Shallow rendering takes the result of the given component's render method and returns a wrapper object with some utilities for traversing the JS object.
- It renders your component one level deep, making it faster than full DOM rendering.
- This means it doesn't run lifecycle methods (since only the React elements are available), it doesn't allow you to actually interact with DOM elements (because nothing's actually rendered), and it doesn't actually attempt to get the React elements that are returned by your custom components.

---

## React Integration Testing

- Having some unit tests to verify that pieces work in isolation is important, but is useless if you don't also verify that they work together.
- Integration tests strike a great balance on the trade-offs between confidence and speed/expense. This is why it's advisable to spend most of the effort there.
- Tools for integration testing: [Enzyme] and [react-testing-library]

### Requirements:

1. interactions between React components, typically performed via calling prop functions such as `<Component onClick={onClickHandler}>`
2. manipulation of component state
3. direct manipulation of the DOM in React lifecycle methods

- In order to test these kinds of interactions we need a way to not only render a whole component tree, but a means to render the component within a functioning DOM.

To Do App:

```
class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      todos: [
        { text: 'Walk the Dog', priority: 'High' },
        { text: 'Ride bike', priority: 'Low' },
        { text: 'Get a haircut', priority: 'Med' }
      ]
    };
  }

  addTodo = todo => {
    const todos = this.state.todos;
    // simulate an AJAX request
    setTimeout(() => {
      this.setState({ todos: todos.concat(todo) });
    }, 0);
  };

  deleteTodo = value => {
    const todos = this.state.todos;
    const index = todos.indexOf(value);
    if (index > -1) {
      this.setState({ todos: todos.filter((item, i) => i !== index) });
    }
  };

  render() {
    return (
      <div className="App">
        <header className="App-header">
          <h1 className="App-title">React Todos</h1>
        </header>
        <div className="App-body">
          <TodoForm addTodo={this.addTodo} />
          <TodoList todos={this.state.todos} deleteTodo={this.deleteTodo} />
        </div>
      </div>
    );
  }
}
```

To Do Item:

```
class TodoItem extends Component {
  render() {
    const { deleteTodo, todo } = this.props;

    return (
      <div className="TodoItem" data-testid="TodoItem">
        <div data-testid="TodoItem-priority" className={`TodoItem-priority TodoItem--${todo.priority.toLowerCase()}`} />
        <div data-testid="TodoItem-text" className={`TodoItem-text`}>
          {todo.text}
        </div>
        <button onClick={() => deleteTodo(todo)} className="TodoItem-delete">
          -
        </button>
      </div>
    );
  }
}
```

To Do List:

```
// ToDoList.js

class TodoList extends Component {
  render() {
    const { todos, deleteTodo } = this.props;
    return todos.map((todo, i) => <TodoItem deleteTodo={deleteTodo} key={i} todo={todo} />);
  }
}
```


The integration test should describe the following:
> Entering a TODO in the TODO form and clicking “Add” should add the TODO to the end of the list of TODOs.

```
// app.rtl.test.js

test('entering a todo in form adds a todo', async () => {
  const { getByText, getByPlaceholderText, getByTestId, container } = render(<App />);

  // enter todo text in textbox
  getByPlaceholderText('Enter todo text').value = 'My new todo';

  // click Add
  Simulate.click(getByText('Add'));

  // wait for Todo to show up
  await wait(() => getByText('My new todo'));

  // make sure form is cleared
  expect(getByTestId('TodoForm-input').value).toEqual('');

  // make sure todo was added
  expect(getByText('My new todo')).toBeDefined();
});
```

Examples borrowed from [Integration Testing in React](https://medium.com/homeaway-tech-blog/integration-testing-in-react-21f92a55a894).


---

## React E2E Testing

- Simulate real user scenarios.
- Slower and more expensive than unit tests, but provide more value and confidence that your app is working as expected.
- E2E testing tools:
  - [Cypress] - Runs without Selenium
  - [Selenium](https://www.seleniumhq.org/) & [Selenium WebDriver](https://www.seleniumhq.org/projects/webdriver/)
- The key is writing less E2E tests, but well-written ones.


---

## React Snapshot Testing

> A typical snapshot test case for a mobile app renders a UI component, takes a snapshot, then compares it to a reference snapshot file stored alongside the test. The test will fail if the two snapshots do not match: either the change is unexpected, or the reference snapshot needs to be updated to the new version of the UI component.

- Used in lieu of rendering the graphical UI, which would require building the entire app.
- Can be used to implement tests quicker.
- Can be used for pure functions and components.
- The snapshot file gets committed along with code changes.
- Does tend to focus on implementation details, but this [can be useful](https://kentcdodds.com/blog/effective-snapshot-testing) sometimes.
- Because they're more integrated and try to serialize an incomplete system (e.g. one with some kind of side effects: from browser/library/runtime versions to environment to database/API changes), they will tend to have high false-negatives (failing test for which the production code is actually fine and the test just needs to be changed).
- False negatives quickly erode the team's trust in a test to actually find bugs and instead come to be seen as a chore on a checklist they need to satisfy before they can move on to the next thing.
- Good for: Error messages and logs, styling (even E2E tests can miss this)
- Avoid huge snapshots.
- Treat snapshots as code.
- Always see if you can actually change it from a snapshot to a more explicit assertion (because you probably can).
- Snapshot testing tools:
  - [Jest Snapshot Testing](https://facebook.github.io/jest/docs/en/snapshot-testing.html)
  - [React Test Renderer](https://reactjs.org/docs/test-renderer.html) - Converts React components to pure JS objects to be used for a snapshot, without the use of a browser or jsdom.

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

For more information on snapshot testing with Jest, go [here](https://jestjs.io/docs/en/snapshot-testing).

## References

- [Effective React Testing @ JazzCon 2018](https://www.youtube.com/watch?v=Eakp29J38YA) by Jeremy Fairbank
- [Effective Snapshot Testing](https://kentcdodds.com/blog/effective-snapshot-testing) by Kent C. Dodds
- [Integration Testing in React](https://medium.com/homeaway-tech-blog/integration-testing-in-react-21f92a55a894) by Jeffrey Russom
- [Jest Testing React Apps](https://jestjs.io/docs/en/tutorial-react)
- [No To More E2E Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html) by Mike Wacker, Google Testing Blog
- [Painless JavaScript Testing with Jest](https://learning.oreilly.com/videos/advanced-design-patterns/9781788838931/9781788838931-video4_2?autoplay=false) by Michele Bertoli, Advanced Design Patterns with React
- [TestPyramid](https://martinfowler.com/bliki/TestPyramid.html) by Martin Fowler
- [The Testing Trophy](https://twitter.com/kentcdodds/status/960723172591992832/photo/1?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E960723172591992832&ref_url=https%3A%2F%2Fkentcdodds.com%2Fblog%2Fwrite-tests) by Kent C. Dodds
- [Unit Testing with Jest](https://learning.oreilly.com/library/view/learn-react-with/9781789610253/57b8659d-c830-4fe6-bf35-608c5b6ce892.xhtml) by Carl Rippon, Learn React with TypeScript 3
- [What is React Testing Library?](https://www.youtube.com/watch?v=JKOwJUM4_RM&feature=youtu.be) by Scott Tolinski
- [Why I Never Use Shallow Rendering](https://kentcdodds.com/blog/why-i-never-use-shallow-rendering) by Kent C. Dodds
- [Write tests. Not too many. Mostly integration.](https://kentcdodds.com/blog/write-tests) by Kent C. Dodds

[Cypress]: https://www.cypress.io/
[Enzyme]: https://airbnb.io/enzyme/
[Istanbul]: https://istanbul.js.org/
[Jest]: https://jestjs.io/
[react-testing-library]: https://github.com/testing-library/react-testing-library
