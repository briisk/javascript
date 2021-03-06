# Briisk JavaScript Test Style Guide

## Table of Contents

  1. [General principles](#general)
  1. [Design principles](#design-principles)
  1. [Structure your tests properly](#structure-your-tests-properly)
  1. [Name your tests properly](#name-your-tests-properly)
  1. [Don't comment tests](#dont-comment-tests)
  1. [Avoid logic in your tests](#avoid-logic-in-your-tests)
  1. [Don't write unnecessary expectations](#dont-write-unnecessary-expectations)
  1. [Setup properly the actions that apply to all the tests involved](#setup-properly-the-actions-that-apply-to-all-the-tests-involved)
  1. [Consider using factory functions in the tests](#consider-using-factory-functions-in-the-tests)
  1. [Jasmine rules](#jasmine-rules)
  1. [Don't test multiple concerns in the same test](#dont-test-multiple-concerns-in-the-same-test)
  1. [Cover the general case and the edge cases](#cover-the-general-case-and-the-edge-cases)
  1. [When applying TDD, always start by writing the simplest failing test](#when-applying-tdd-always-start-by-writing-the-simplest-failing-test)
  1. [When applying TDD, always make small steps in each test-first cycle](#when-applying-tdd-always-make-small-steps-in-each-test-first-cycle)
  1. [Test the behaviour, not the internal implementation](#test-the-behaviour-not-the-internal-implementation)
  1. [Mock everything](#mock-everything)
  1. [Integration tests](#integration-tests)
  1. [Don't write unit tests for complex user interactions](#dont-write-unit-tests-for-complex-user-interactions)
  1. [Test simple user actions](#test-simple-user-actions)
  1. [Review test code first](#review-test-code-first)
  1. [Practice code katas, learn with pair programming](#practice-code-katas-learn-with-pair-programming)


## General principles

### General principles

**Unit = Unit of work**

This could involve **multiple methods and classes** invoked by some public API that can:

+ Return a value or throw an exception
+ Change the state of the system
+ Make 3rd party callsą

A unit test should test the behaviour of a unit of work: for a given input, it expects an end result that can be any of the above.

**Unit tests are isolated and independent of each other**

+ Any given behaviour should be specified in **one and only one test**
+ The execution/order of execution of one test **cannot affect the others**

The code is designed to support this independence (see "Design principles" below).

**Unit tests are lightweight tests**

+ Repeatable
+ Fast
+ Consistent
+ Easy to write and read

**Unit tests are code too**

They must meet the same level of quality as the code being tested. They can be refactored as well to make them more maintainable and/or readable.

• [Back to ToC](#table-of-contents) •

### Design principles

The key to good unit testing is to write **testable code**. Applying simple design principles can help, in particular:

+ Use a **good naming** and **comment** your code (the "why?" not the "how"), keep in mind that comments are not a substitute for bad naming or bad design
+ **DRY**: Don't Repeat Yourself, avoid code duplication
+ **Single responsibility**: each object/function must focus on a single task
+ Keep a **single level of abstraction** in the same component (for example, do not mix business logic with lower-level technical details in the same method)
+ **Minimize dependencies** between components: encapsulate, interchange less information between components
+ **Support configurability** rather than hard-coding, this prevents from having to replicate the exact same environment when testing (e.g.: markup)
+ Apply adequate **design patterns**, especially **dependency injection** that allows to separate objects creation responsibility from business logic
+ Avoid global mutable state

• [Back to ToC](#table-of-contents) •

### Structure your tests properly

Don't hesitate to nest your suites to structure logically your tests in subsets.

**bad**

```js
describe('A set of functionalities', () => {
  it('a set of functionalities should do something nice', () => {
  });

  it('a subset of functionalities should do something great', () => {
  });

  it('a subset of functionalities should do something awesome', () => {
  });

  it('another subset of functionalities should also do something great', () => {
  });
});
```

**good**

```js
describe('A set of functionalities', () => {
  it('should do something nice', () => {
  });

  describe('A subset of functionalities', () => {
    it('should do something great', () => {
    });

    it('should do something awesome', () => {
    });
  });

  describe('Another subset of functionalities', () => {
    it('should also do something great', () => {
    });
  });
});
```

• [Back to ToC](#table-of-contents) •

### Name your tests properly

Tests names should be concise, explicit, descriptive and in correct English. Read the output of the spec runner and verify that it is understandable! Keep in mind that someone else will read it too. Tests can be the live documentation of the code.

**bad**

```js
describe('myGallery', () => {
  it('init set correct property when called (thumb size, thumbs count)', () => {
  });

  // ...
});
```

**good**

```js
describe('The Gallery instance', () => {
  it('should properly calculate the thumb size when initialized', () => {
  });

  it('should properly calculate the thumbs count when initialized', () => {
  });

  // ...
});
```

In order to help you write test names properly, you can use the **"unit of work - scenario/context - expected behaviour"** pattern:

```js
describe('[unit of work]', () => {
  it('should [expected behaviour] when [scenario/context]', () => {
  });
});
```

Or whenever you have many tests that follow the same scenario or are related to the same context:

```js
describe('[unit of work]', () => {
  describe('when [scenario/context]', () => {
    it('should [expected behaviour]', () => {
    });
  });
});
```

For example:

```js
describe('The Gallery instance', () => {
  describe('when initialized', () => {
    it('should properly calculate the thumb size', () => {
    });

    it('should properly calculate the thumbs count', () => {
    });
  });

  // ...
});
```

• [Back to ToC](#table-of-contents) •

### Don't comment tests

Never. Ever. Tests have a reason to be or not.

Don't comment them because they are too slow, too complex or produce false negatives. Instead, make them fast, simple and trustworthy. If not, remove them completely.

• [Back to ToC](#table-of-contents) •

### Avoid logic in your tests

Always use simple statements. Loops and conditionals should not be used. If they are, you add a possible entry point for bugs in the test itself:

+ Conditionals: you don't know which path the test will take
+ Loops: you could be sharing state between tests

**bad**

```js
it('should properly sanitize strings', () => {
  let result;
  const testValues = {
    'Avion'         : 'Avi' + String.fromCharCode(243) + 'n',
    'The-space'     : 'The space',
    'Weird-chars-'  : 'Weird chars!!',
    'file-name.zip' : 'file name.zip',
    'my-name.zip'   : 'my.name.zip'
  };

  for (result in testValues) {
    expect(sanitizeString(testValues[result]) .toEqual(result);
  }
});
```

**good**

```js
it('should properly sanitize strings', () => {
  expect(sanitizeString('Avi'+String.fromCharCode(243)+'n')).toEqual('Avion');
  expect(sanitizeString('The space')).toEqual('The-space');
  expect(sanitizeString('Weird chars!!')).toEqual('Weird-chars-');
  expect(sanitizeString('file name.zip')).toEqual('file-name.zip');
  expect(sanitizeString('my.name.zip')).toEqual('my-name.zip');
});
```

Better: write a test for each type of sanitization. It will give a nice output of all possible cases, improving readability and maintainability.

```js
it('should sanitize a string containing non-ASCII chars', () => {
  expect(sanitizeString('Avi'+String.fromCharCode(243)+'n')).toEqual('Avion');
});

it('should sanitize a string containing spaces', () => {
  expect(sanitizeString('The space')).toEqual('The-space');
});

it('should sanitize a string containing exclamation signs', () => {
  expect(sanitizeString('Weird chars!!')).toEqual('Weird-chars-');
});

it('should sanitize a filename containing spaces', () => {
  expect(sanitizeString('file name.zip')).toEqual('file-name.zip');
});

it('should sanitize a filename containing more than one dot', () => {
  expect(sanitizeString('my.name.zip')).toEqual('my-name.zip');
});
```

• [Back to ToC](#table-of-contents) •

### Don't write unnecessary expectations

Remember, unit tests are a design specification of how a certain *behaviour* should work, not a list of observations of everything the code happens to do.

**bad**

```js
it('should multiply the number passed as parameter and subtract one', () => {
  const multiplySpy = spyOn(Calculator, 'multiple').and.callThrough();
  const subtractSpy = spyOn(Calculator, 'subtract').and.callThrough();

  const result = Calculator.compute(21.5);

  expect(multiplySpy).toHaveBeenCalledWith(21.5);
  expect(subtractSpy).toHaveBeenCalledWith(43, 1);
  expect(result).toBe(42);
});
```

**good**

```js
it('should multiply the number passed as parameter and subtract one', () => {
  const result = Calculator.compute(21.5);
  expect(result).toBe(42);
});
```

This will improve maintainability. Your test is no more tight to implementation details.

• [Back to ToC](#table-of-contents) •

### Setup properly the actions that apply to all the tests involved

**bad**

```js
describe('Saving the user profile', () => {
  let profileModule;
  let notifyUserSpy;
  let onCompleteSpy;

  beforeEach(() => {
    profileModule = new ProfileModule();
    notifyUserSpy = spyOn(profileModule, 'notifyUser');
    onCompleteSpy = jasmine.createSpy();
  });

  it('should send the updated profile data to the server', () => {
    jasmine.Ajax.install();

    profileModule.save();

    const request = jasmine.Ajax.requests.mostRecent();

    expect(request.url).toBe('/profiles/1');
    expect(request.method).toBe('POST');
    expect(request.data()).toEqual({ username: 'mawrkus' });

    jasmine.Ajax.uninstall();
  });

  it('should notify the user', () => {
    jasmine.Ajax.install();

    profileModule.save();

    expect(notifyUserSpy).toHaveBeenCalled();

    jasmine.Ajax.uninstall();
  });

  it('should properly execute the callback passed as parameter', () => {
    jasmine.Ajax.install();

    profileModule.save(onCompleteSpy);

    jasmine.Ajax.uninstall();

    expect(onCompleteSpy).toHaveBeenCalled();
  });
});
```

The setup code should apply to all the tests:

**good**

```js
describe('Saving the user profile', () => {
  let profileModule;

  beforeEach(() => {
    jasmine.Ajax.install();
    profileModule = new ProfileModule();
  });

  afterEach(() => {
    jasmine.Ajax.uninstall();
  });

  it('should send the updated profile data to the server', () => {
    profileModule.save();

    const request = jasmine.Ajax.requests.mostRecent();

    expect(request.url).toBe('/profiles/1');
    expect(request.method).toBe('POST');

  });

  it('should notify the user', () => {
    spyOn(profileModule, 'notifyUser');

    profileModule.save();

    expect(profileModule.notifyUser).toHaveBeenCalled();
  });

  it('should properly execute the callback passed as parameter', () => {
    const onCompleteSpy = jasmine.createSpy();

    profileModule.save(onCompleteSpy);

    expect(onCompleteSpy).toHaveBeenCalled();
  });
});
```

Consider keeping the setup code minimal to preserve readability and maintainability.

• [Back to ToC](#table-of-contents) •

### Consider using factory functions in the tests

Factories can:

- help reducing the setup code, especially if you use dependency injection
- make each test more readable, the creation is a single function call that can be in the test itself instead of the setup
- provide flexibility when creating new instances (setting an initial state, for example)

There's a trade-off to find here between applying the DRY principle and readability.

**bad**

```js
describe('User profile module', () => {
  let profileModule;
  let pubSub;

  beforeEach(() => {
    let element = document.getElementById('my-profile');
    pubSub = new PubSub({ sync: true });

    profileModule = new ProfileModule({
      element,
      pubSub,
      likes: 0
    });
  });

  it('should publish a topic when a new "like" is given', () => {
    spyOn(pubSub, 'notify');
    profileModule.incLikes();
    expect(pubSub.notify).toHaveBeenCalledWith('likes:inc', { count: 1 });
  });

  it('should retrieve the correct number of likes', () => {
    profileModule.incLikes();
    profileModule.incLikes();
    expect(profileModule.getLikes()).toBe(2);
  });
});
```

**good**

```js
describe('User profile module', () => {
  function createProfileModule({
    element = document.getElementById('my-profile'),
    likes = 0,
    pubSub = new PubSub({ sync: true })
  })
  {
    return new ProfileModule({ element, likes, pubSub });
  }

  it('should publish a topic when a new "like" is given', () => {
    const pubSub = jasmine.createSpyObj('pubSub', ['notify']);
    const profileModule = createProfileModule({ pubSub });

    profileModule.incLikes();

    expect(pubSub.notify).toHaveBeenCalledWith('likes:inc');
  });

  it('should retrieve the correct number of likes', () => {
    const profileModule = createProfileModule({ likes: 40 });

    profileModule.incLikes();
    profileModule.incLikes();

    expect(profileModule.getLikes()).toBe(42);
  });
});
```

Factories are particularly useful when dealing with the DOM:

**bad**

```js
describe('The search component', () => {
  describe('when the search button is clicked', () => {
    let container;
    let form;
    let searchInput;
    let submitInput;

    beforeEach(() => {
      fixtures.inject(`<div id="container">
        <form class="js-form" action="/search">
          <input type="search">
          <input type="submit" value="Search">
        </form>
      </div>`);

      container = document.getElementById('container');
      form = container.getElementsByClassName('js-form')[0];
      searchInput = form.querySelector('input[type=search]');
      submitInput = form.querySelector('input[type=submith]');
    });

    it('should validate the text entered', () => {
      const search = new Search({ container });
      spyOn(search, 'validate');

      search.init();

      input(searchInput, 'peace');
      click(submitInput);

      expect(search.validate).toHaveBeenCalledWith('peace');
    });

    // ...
  });
});
```

**good**

```js
function createHTMLFixture() {
  fixtures.inject(`<div id="container">
    <form class="js-form" action="/search">
      <input type="search">
      <input type="submit" value="Search">
    </form>
  </div>`);

  const container = document.getElementById('container');
  const form = container.getElementsByClassName('js-form')[0];
  const searchInput = form.querySelector('input[type=search]');
  const submitInput = form.querySelector('input[type=submith]');

  return {
    container,
    form,
    searchInput,
    submitInput
  };
}

describe('The search component', () => {
  describe('when the search button is clicked', () => {
    it('should validate the text entered', () => {
      const { container, form, searchInput, submitInput } = createHTMLFixture();
      const search = new Search({ container });
      spyOn(search, 'validate');

      search.init();

      input(searchInput, 'peace');
      click(submitInput);

      expect(search.validate).toHaveBeenCalledWith('peace');
    });

    // ...
  });
});
```

• [Back to ToC](#table-of-contents) •

### Jasmine rules

The API documentation of the testing framework/library should be your bedside book!

Having a good knowledge of the API can help you reducing the size/complexity of your test code and, in general, help you during development. A simple example:

#### x and f
**bad**

```js
it('should call a method with the proper arguments', () => {
  const foo = {
    bar: jasmine.createSpy(),
    baz: jasmine.createSpy()
  };

  foo.bar('qux');

  expect(foo.bar).toHaveBeenCalled();
  expect(foo.bar.calls.argsFor(0)).toEqual(['qux']);
});

/*it('should do more but not now', () => {
});

it('should do much more but not now', () => {
});*/
```

**good**

```js
it('should call once a method with the proper arguments', () => {
  const foo = jasmine.createSpyObj('foo', ['bar', 'baz']);

  foo.bar('baz');

  expect(foo.bar).toHaveBeenCalledWith('baz');
});

xit('should do something else but not now', () => {
});

xit('should do something else but not now', () => {
});
```

#### Note

The handy `xit` function used in the example above allows you to ignore specific test without having to comment all the tests below. `xdescribe` does the same for test suites. This could help saving a lot of time when developing.

The handy `fit` function allows you to execute only one test without having to comment all the tests below. `fdescribe` does the same for test suites. This could help saving a lot of time when developing.

More information on the [Jasmine website](http://jasmine.github.io).

• [Back to ToC](#table-of-contents) •

### Don't test multiple concerns in the same test

If a method has several end results, each one should be tested separately. Whenever a bug occurs, it will help you locate the source of the problem.

**bad**

```js
it('should send the profile data to the server and update the profile view properly', () => {
  // expect(...)to(...);
  // expect(...)to(...);
});
```

**good**

```js
it('should send the profile data to the server', () => {
  // expect(...)to(...);
});

it('should update the profile view properly', () => {
  // expect(...)to(...);
});
```

Beware that writing "AND" or "OR" when naming your test smells bad...

• [Back to ToC](#table-of-contents) •

### Cover the general case and the edge cases

"Strange behaviour" usually happens at the edges... Remember that your tests can be the live documentation of your code.

**bad**

```js
it('should properly calculate a RPN expression', () => {
  const result = RPN('5 1 2 + 4 * - 10 /');
  expect(result).toBe(-0.7);
});
```

**good**

```js
describe('The RPN expression evaluator', () => {
  it('should return null when the expression is an empty string', () => {
    const result = RPN('');
    expect(result).toBeNull();
  });

  it('should return the same value when the expression holds a single value', () => {
    const result = RPN('42');
    expect(result).toBe(42);
  });

  it('should properly calculate an expression', () => {
    const result = RPN('5 1 2 + 4 * - 10 /');
    expect(result).toBe(-0.7);
  });

  it('should throw an error whenever an invalid expression is passed', () => {
    const compute = () => RPN('1 + - 1');
    expect(compute).toThrow();
  });
});
```

• [Back to ToC](#table-of-contents) •

### When applying TDD, always start by writing the simplest failing test

**bad**

```js
it('should suppress all chars that appear multiple times', () => {
  expect(keepUniqueChars('Hello Fostonic !!')).toBe('HeFstnic');
});
```

**good**

```js
it('should return an empty string when passed an empty string', () => {
  expect(keepUniqueChars('')).toBe('');
});
```

From there, start building the functionalities incrementally.

• [Back to ToC](#table-of-contents) •

### When applying TDD, always make small steps in each test-first cycle

Build your tests suite from the simple case to the more complex ones. Keep in mind the incremental design. Deliver software fast, incrementally, and in short iterations.

**bad**

```js
it('should return null when the expression is an empty string', () => {
  const result = RPN('');
  expect(result).toBeNull();
});

it('should properly calculate a RPN expression', () => {
  const result = RPN('5 1 2 + 4 * - 10 /');
  expect(result).toBe(-0.7);
});
```

**good**

```js
describe('The RPN expression evaluator', () => {
  it('should return null when the expression is an empty string', () => {
    const result = RPN('');
    expect(result).toBeNull();
  });

  it('should return the same value when the expression holds a single value', () => {
    const result = RPN('42');
    expect(result).toBe(42);
  });

  describe('Additions-only expressions', () => {
    it('should properly calculate a simple addition', () => {
      const result = RPN('41 1 +');
      expect(result).toBe(42);
    });

    it('should properly calculate a complex addition', () => {
      const result = RPN('2 9 + 15 3 + + 7 6 + +');
      expect(result).toBe(42);
    });
  });

  // ...

  describe('Complex expressions', () => {
    it('should properly calculate an expression containing all 4 operators', () => {
      const result = RPN('5 1 2 + 4 * - 10 /');
      expect(result).toBe(-0.7);
    });
  });
});
```

• [Back to ToC](#table-of-contents) •

### Test the behaviour, not the internal implementation

**bad**

```js
it('should add a user in memory', () => {
  userManager.addUser('Dr. Falker', 'Joshua');

  expect(userManager._users[0].name).toBe('Dr. Falker');
  expect(userManager._users[0].password).toBe('Joshua');
});
```

A better approach is to test at the same level of the API:

**good**

```js
it('should add a user in memory', () => {
  userManager.addUser('Dr. Falker', 'Joshua');

  expect(userManager.loginUser('Dr. Falker', 'Joshua')).toBe(true);
});
```

Advantage:

+ Changing the internal implementation of a class/object will not necessarily force you to refactor the tests

Disadvantage:

+ If a test is failing, we might have to debug to know which part of the code needs to be fixed. It means that it would be good to divide your code to smaller components/services

• [Back to ToC](#table-of-contents) •

### Mock everything
### if you are writing unit tests

**bad**

```js
describe('when the user has already visited the page', () => {
  // storage.getItem('page-visited', '1') === '1'
  describe('when the survey is not disabled', () => {
    // storage.getItem('survey-disabled') === null
    it('should display the survey', () => {
      const storage = window.localStorage; // ouch.
      storage.setItem('page-visited', '1');

      const surveyManager = new SurveyManager();
      spyOn(surveyManager, 'display');

      surveyManager.start();

      expect(surveyManager.display).toHaveBeenCalled();
    });
  });

  // ...
});
```

We created a permanent storage of data. What happens if we do not properly clean it?
We might affect the other tests. Let's fix this:

**bad**

```js
describe('when the user has already visited the page', () => {
  // storage.getItem('page-visited', '1') === '1'
  describe('when the survey is not disabled', () => {
    // storage.getItem('survey-disabled') === null
    it('should display the survey', () => {
      const storage = jasmine.createSpyObj('storage', ['setItem', 'getItem']);
      storage.getItem.and.returnValue('1'); // ouch.

      const surveyManager = new SurveyManager(storage);
      spyOn(surveyManager, 'display');

      surveyManager.start();

      expect(surveyManager.display).toHaveBeenCalled();
    });
  });

  // ...
});
```

This test fails, because the survey is considered disabled. Let's fix this:

**good**

```js
describe('when the user has already visited the page', () => {
  // storage.getItem('page-visited', '1') === '1'
  describe('when the survey is not disabled', () => {
    // storage.getItem('survey-disabled') === null
    it('should display the survey', () => {
      const storage = jasmine.createSpyObj('storage', ['setItem', 'getItem']);
      storage.getItem.and.callFake(key => {
        switch (key) {
          case 'page-visited':
            return '1';

          case 'survey-disabled':
            return null;
        }

        return null;
      }); // ouch.

      const surveyManager = new SurveyManager(storage);
      spyOn(surveyManager, 'display');

      surveyManager.start();

      expect(surveyManager.display).toHaveBeenCalled();
    });
  });

  // ...
});
```


#### Takeaway

The idea to keep in mind is that *dependencies can still be "real" objects*. Mock everything in unit tests, but you don't need to mock everything in integration tests.
In particular, consider using the "real" version of the objects if:

- it leads to a simple, nice and easy tests setup
- it does not create a shared state between the tests, causing unexpected side effects
- the code being tested does not make AJAX requests, API calls or browser page reloads
- the speed of execution of the tests stays *within the limits you fixed*

• [Back to ToC](#table-of-contents) •

### Integration tests

* **Unit testing** - Component is a unit (with all logic and template, you shouldn't test logic without template), Service is a unit, Function is a unit. So you are writing unit tests when you are testing only one unit and you are mocking all dependecies, parameters.

* **Integration testing** - you are writing integration tests when you are testing more than one unit. They looks like unit tests but they don't mock everything. They are usefull when some units depends on each other in strong way. For example a big form with many dumb components or two servises which work on the same data.

Integration tests should be complement to unit tests. It means, that you should create unit tests for every unit and then you can create integration tests above them. Why? Because it is simpler to catch where code is wrong with unit tests, also there is possibility that unit tests will pass but integration between components/services will be wrong. Integration tests cover much more code in single spec.

[Unit tests without integration tests](https://michelenasti.com/images/unit-testing-1.gif)

• [Back to ToC](#table-of-contents) •

### Test simple user actions


### Don't write unit tests for complex user interactions

Examples of complex user interactions:

+ Filling a form, drag and dropping some items then submitting the form
+ Clicking a tab, clicking an image thumbnail then navigating through a gallery of images previously loaded from a database
+ (...)

These interactions might involve many units of work and should be handled at a higher level by **functional tests**. They will take more time to execute. They could be flaky (false negatives) and they need debugging whenever a failure is reported.

For functional testing, consider using a test automation framework ([Selenium](http://docs.seleniumhq.org/), ...) or QA manual testing.

• [Back to ToC](#table-of-contents) •

### Test simple user actions

Example of simple user actions:

+ Clicking on a link that toggles the visibility of a DOM element
+ Submitting a form that triggers the form validation
+ (...)

These actions can be easily tested **by simulating DOM events**, for example:

```js
describe('When clicking on the "Preview profile" link', () => {
  it('should show the profile preview if it is hidden', () => {
    const previewLink = document.createElement('a');
    const profileModule = createProfileModule({ previewLink, previewIsVisible: false });

    spyOn(profileModule, 'showPreview');

    click(previewLink);

    expect(profileModule.showPreview).toHaveBeenCalled();
  });

  it('should hide the profile preview if it is displayed', () => {
    const previewLink = document.createElement('a');
    const profileModule = createProfileModule({ previewLink, previewIsVisible: true });

    spyOn(profileModule, 'hidePreview');

    click(previewLink);

    expect(profileModule.hidePreview).toHaveBeenCalled();
  });
});
```

• [Back to ToC](#table-of-contents) •

### Review test code first

When reviewing code, always start by reading the code of the tests. Tests are mini use cases of the code that you can drill into.

It will help you understand the intent of the developer very quickly (could be just by looking at the name of the tests).

• [Back to ToC](#table-of-contents) •

### Practice code katas, learn with pair programming

Because experience is the _only_ teacher. Ultimately, greatness comes from practicing ; applying the theory over and over again, using feedback to get better every time.

• [Back to ToC](#table-of-contents) •

## References

+ Roy Osherove - "JS Unit Testing Good Practices and Horrible Mistakes" : https://www.youtube.com/watch?v=iP0Vl-vU3XM
+ Enrique Amodeo - "Learning Behavior-driven Development with JavaScript" : https://www.packtpub.com/application-development/learning-behavior-driven-development-javascript
+ Steven Sanderson - "Writing Great Unit Tests: Best and Worst Practices" : http://blog.stevensanderson.com/2009/08/24/writing-great-unit-tests-best-and-worst-practises/
+ Rebecca Murphy - "Writing Testable JavaScript" : http://alistapart.com/article/writing-testable-javascript
+ YUI Team - "Writing Effective JavaScript Unit Tests with YUI Test" : http://yuiblog.com/blog/2009/01/05/effective-tests/
+ Colin Snover - "Testable code best practices" : http://www.sitepen.com/blog/2014/07/11/testable-code-best-practices/
+ Miško Hevery - "The Clean Code Talks -- Unit Testing" : https://www.youtube.com/watch?v=wEhu57pih5w
+ Addy Osmani - "Learning JavaScript Design Patterns" : http://addyosmani.com/resources/essentialjsdesignpatterns/book/
+ José Armesto - "Unit Testing sucks (and it’s our fault) " : https://www.youtube.com/watch?v=GZ9iZsMAZFQ
+ Clean code cheat sheet: http://www.planetgeek.ch/2014/11/18/clean-code-cheat-sheet-v-2-4/

• [Back to ToC](#table-of-contents) •

## Contributors

Forked from: https://github.com/mawrkus/js-unit-testing-guide

Marc Mignonsin: https://github.com/mawrkus

Ruben Norte: https://github.com/rubennorte

Daniel Koza: https://github.com/dakolech

• [Back to ToC](#table-of-contents) •
