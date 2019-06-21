# React Testing

- From easiest to most difficult to write: snapshot, unit, integration, E2E

## Snapshot testing

- Can be used to implement tests quicker.
- Can be used for pure functions and components.
- Does tend to focus on implementation details, but this can be useful sometimes.

## Unit testing

- We can test pure functions and components.
- Pure functions are the easiest to test, because there are no side effects.

The test function takes in two parameters:
- The first parameter is a message telling us whether the test passed or not, which will be shown in the test output.
- The second parameter is an arrow function that will contain our test

- The `test.tsx` extension is important because Jest automatically looks for files with this extension when finding tests to execute. Note that if our tests don't contain any JSX, we could use a `test.ts` extension.


### Tips for Writing Unit Tests

>The more your tests resemble the way your software is used, the more confidence they can give you. — Kent C. Dodds 👋

- Things to consider when testing:
  1. Will this test break when there's a mistake that would break the component in production?
  2. Will this test continue to work when there's a fully backward compatible refactor of the component?

- If your test uses `instance()` or `state()`, know that you're testing things that the user couldn't possibly know about or even care about, which will take your tests further from giving you confidence that things will work when your user uses them.
- Do not test implementation details. It can give a false sense of security, and will slow down refactoring. Tests will need to be updated after refactoring and can quickly become obsolete. You should very rarely have to change tests when you refactor code.


### Unit testing tools
- [Enzyme](https://airbnb.io/enzyme/)
- [Jest](https://jestjs.io/)
  - Comes with create-react-app; built by Facebook.
- [react-testing-library](https://github.com/testing-library/react-testing-library)

### Shallow rendering

- Enzyme introduces a concept of **shallow rendering**, which is [controversial](https://kentcdodds.com/blog/why-i-never-use-shallow-rendering) at the moment.
- The controversy is that shallow rendering provides convenience that sacrifices test reliability. With shallow rendering, it is possible to refactor a component's implementation and and tests would still pass.
- Shallow rendering takes the result of the given component's render method and gives a wrapper object with some utilities for traversing the JS object.
- This means it doesn't run lifecycle methods (only the React elements are available), it doesn't allow you to actually interact with DOM elements (because nothing's actually rendered), and it doesn't actually attempt to get the React elements that are returned by your custom components.

### Karma, Mocha

### Mocking dependencies
- If we mock too much, we aren't really testing our app.
- Good use case of mocking with unit tests: REST APIs

### Code coverage

- Help identify areas that need more unit tests.
- Aiming for 100% code coverage is not only unrealistic, but also can encourage testing things that do not need to be tested and can be quite costly. This is only more possible on very small projects.



## Integration testing

- Integration tests strike a great balance on the trade-offs between confidence and speed/expense. This is why it's advisable to spend most (not all, mind you) of your effort there.

## E2E testing

- Slower and more expensive than unit tests, but provides more value and confidence that your app is working as expected.

### E2E testing tools
- [Cypress](https://www.cypress.io/) - Runs without Selenium


## Static testing

## References

- [Effective React Testing @ JazzCon 2018](https://www.youtube.com/watch?v=Eakp29J38YA) by Jeremy Fairbank
- [No To More E2E Tests](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html) by Mike Wacker, Google Testing Blog
- [Painless JavaScript Testing with Jest](https://learning.oreilly.com/videos/advanced-design-patterns/9781788838931/9781788838931-video4_2?autoplay=false) by Michele Bertoli, Advanced Design Patterns with React
- [TestPyramid](https://martinfowler.com/bliki/TestPyramid.html) by Martin Fowler
- [The Testing Trophy](https://twitter.com/kentcdodds/status/960723172591992832/photo/1?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E960723172591992832&ref_url=https%3A%2F%2Fkentcdodds.com%2Fblog%2Fwrite-tests) by Kent C. Dodds
- [Unit Testing with Jest](https://learning.oreilly.com/library/view/learn-react-with/9781789610253/57b8659d-c830-4fe6-bf35-608c5b6ce892.xhtml) by Carl Rippon, Learn React with TypeScript 3
- [Why I Never Use Shallow Rendering](https://kentcdodds.com/blog/why-i-never-use-shallow-rendering) by Kent C. Dodds
- [Write tests. Not too many. Mostly integration.](https://kentcdodds.com/blog/write-tests) by Kent C. Dodds