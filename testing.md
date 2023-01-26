testing guide.

## Table of Contents

*   [testing-guide]
    *   [Basic Rules](#basic-rules)
        1.  [A(A)AA Pattern - Arrange (Assert) Act Assert](#a(a)aa-pattern---arrange-(assert)-act-assert)
        2.  [Create component helper](#create-component-helper)
        3.  [Use: screen](#use-screen)
        4.  [Use: userEvent](#use-user-event-over-fireevent-where-possible)
        5.  [Use correct Query Order](#use-correct-query-order)
    *   [Do's and Don'ts](#dos-and-donts)
        1.  [Use jest-dom assertion](#use-jest-dom-assertion)
        2.  [Use find over waitFor](#use-find-over-waitfor)
        3.  [Always use assertion in waitFor](#always-use-assertion-in-waitfor)
        4.  [Dont perform side effects in waitFor](#dont-perform-side-effects-in-waitfor)


## Basic Rules
  
We strongly encourage testing components' behavior rather than testing their implementation details. What does it mean? Generally, testing the behavior means making assertions about the outcome of a desired action, rather than asserting about the way the component achieved this outcome internally.

This means we can rewrite/refactor a component implementation and have its tests remain the same and not break.

**What we should be testing?** 
1.  Rendering the components with Props (Dev User perspective)
2.  Querying and interacting with the rendered results (end user perspective )   
Note: _**User** is both end user and also the other developer who is consuming our component._ 
3.  Positive as well as **negative** case i.e. what should happen and what shouldn’t.happen
    *   It's quite easy to test the positive path and check if the functionality works and easy to miss to do the negative tests. Negative testing ensures that application behaves as expected when there is a input provided which is not being expected or not allowed.
4.  Should test Edge cases and boundaries.
    *   Unit tests should test both sides of a given boundary. If testing for date and time utilities, try testing one second before midnight and one second after. Check across the date value of 0.0.
5. Try to cover every code path.

**What we should not be testing?**
implementation details including   
*   State,   
*   Component Names,   
*   Css Classes Selectors   
*   Anything that the users don't see

#### A(A)AA Pattern - Arrange (Assert) Act Assert

We should structure our tests by the [A(A)AA pattern](http://wiki.c2.com/?ArrangeActAssert), containing a visual separation between each block.

**A(A)AA improves readability**
The more readable your tests will be, the more it will be easy to fix a bug for not just the one who wrote it, but for other developers working on the project or will work on the project.

*   Arrange: All the setup needed to happen to bring the system to the scenario the test aims to simulate.  
    * mocking functions, 
    * creating objects
    * etc.
*   Assert Initial: Assert on default state / initial value(s)
*   Act: Simulate (user)interaction, for example type some text in an inputbox
*   Assert: Assert on values after simulated interactions

✅ Do - Follow the A(A)AA pattern and have visible separations between each block:

```javascript
test('SearchBox: Should not call onChange when input is disabled', () => {
  // Arange
  const onChange = jest.fn()
  const { user } = createComponent();
  const input = screen.getByRole('textbox')

  // Assert initial
  expect(input.value).toBe('0')

  // Act
  user.type(input, '1')

  // Assert
  expect(onChange).toHaveBeenCalledTimes(1)
  expect(input.value).toBe('1')
})
```

❌ Don't - Write in one block. It's harder to interpret and understand without diving into the code:

```javascript
test('SearchBox: Should not call onChange when input is disabled', () => {
  // Arange
  const onChange = jest.fn()
  const { user } = createComponent();
  const input = screen.getByRole('textbox')
  expect(input.value).toBe('0')
  user.type(input, '1')
  expect(onChange).toHaveBeenCalledTimes(1)
  expect(input.value).toBe('1')
})
```

#### Create component helper

Render component with helper function to prevent repetitive code and isolate component rendering logic.

*   return render utils which can be retrieved by destructuring


✅ Do - Create component with a helper function:

```javascript
const createComponent = (props): ICreateComponent => {
    const someUtil = SomeUtil();

    return {
        someUtil,
        ...render(<SearchBox {...props} />)
    }
}
test('SearchBox: Create component with helper function', () => {
    // Arange
    const { someUtil } = createComponent({
        defaultValue: 1
    });
    ...
})
```

❌ Don't - write out render logic in every test

```javascript
test('SearchBox: Should not write out rendering logic inside test block', () => {
  // Arange
  render(<SearchBox defaultValue={1} />)
  ...
})
```

#### Use: screen

The benefit of using screen is you no longer need to keep the render call destructure up-to-date as you add/remove the queries you need. You only need to type screen and do querying like you are a screenreader (and this stimulates accessibility driven developing).

*   screen can be imported from '@testing-library/react'

✅ Do - Use screen

```javascript
test('SearchBox: Use screen', () => {
    // Arange
    createComponent();
    const error = screen.getByRole('alert');
})
```

❌ Don't - use utils passed by render function

```javascript
test('SearchBox: use destructure util functions', () => {
  // Arange
  const { getByRole } = createComponent();
  const error = getByRole('alert')
})
```


#### Use: userEvent over fireEvent where possible

The benefit of using userEvent over fireEvent: it resembles the user interactions more closely.

*   userEvent can be imported from '@testing-library/user-event'
*   userEvent setup can be initialised in createComponent function, return as user object and restrived from within test by destructuring

✅ Do - Use userEvent

```javascript
const createComponent = (props): ICreateComponent => {
    const user = userEvent.setup();

    return {
        user,
        ...render(<SearchBox {...props} />)
    }
}
test('SearchBox: Use userEvent', () => {
    // Arange
    const { user } = createComponent();
    
    // Act
    user.type(input, 'hello world');
})
```

❌ Don't - use fireEvent

```javascript
test('SearchBox: use destructure util functions', () => {
  // Arange
  createComponent();
  
  // Act
  fireEvent.change(input, {target: {value: 'hello world'}})
})
```

#### Use correct Query Order

1.  **Queries Accessible to Everyone** Queries that reflect the experience of visual/mouse users as well as those that use assistive technology.
    1.  `getByRole`: This can be used to query every element that is exposed in the [accessibility tree](https://developer.mozilla.org/en-US/docs/Glossary/AOM). With the `name` option you can filter the returned elements by their [accessible name](https://www.w3.org/TR/accname-1.1/). This should be your top preference for just about everything. There's not much you can't get with this (if you can't, it's possible your UI is inaccessible). Most often, this will be used with the `name` option like so: `getByRole('button', {name: /submit/i})`. Check the [list of roles](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques#Roles).
    2.  `getByLabelText`: This method is really good for form fields. When navigating through a website form, users find elements using label text. This method emulates that behavior, so it should be your top preference.
    3.  `getByPlaceholderText`: [A placeholder is not a substitute for a label](https://www.nngroup.com/articles/form-design-placeholders/). But if that's all you have, then it's better than alternatives.
    4.  `getByText`: Outside of forms, text content is the main way users find elements. This method can be used to find non-interactive elements (like divs, spans, and paragraphs).
    5.  `getByDisplayValue`: The current value of a form element can be useful when navigating a page with filled-in values.
2.  **Semantic Queries** HTML5 and ARIA compliant selectors. Note that the user experience of interacting with these attributes varies greatly across browsers and assistive technology.
    1.  `getByAltText`: If your element is one which supports `alt` text (`img`, `area`, `input`, and any custom element), then you can use this to find that element.
    2.  `getByTitle`: The title attribute is not consistently read by screenreaders, and is not visible by default for sighted users
3.  **Test IDs**
    1.  `getByTestId`: The user cannot see (or hear) these, so this is only recommended for cases where you can't match by role or text or it doesn't make sense (e.g. the text is dynamic).

## Do's and Don'ts

#### Use jest-dom assertion

It's strongly recommended to use jest-dom because the error messages you get with it are much better.

✅ Do - jest-dom assertion

```javascript
    expect(button).toBeDisabled()
```

❌ Don't

```javascript
    expect(button.disabled).toBe(true)
```

#### Use find over waitFor

use find* any time you want to query for something that may not be available right away.

✅ Do - use find*

```javascript
    const submitButton = await screen.findByRole('button', {name: /submit/i})
```

❌ Don't

```javascript
    const submitButton = await waitFor(() =>
        screen.getByRole('button', {name: /submit/i}),
    )
```

#### Always use assertion in waitFor

Sometimes you need to wait for something to be present

✅ Do - assertion inside waitFor

```javascript
    await waitFor(() => expect(something).toBe('foo'))
```

❌ Don't

```javascript
    // hack to wait for next tick in event loop
    await waitFor(() => {})
    expect(something).toBe('foo')
```


#### Dont perform side effects in waitFor

✅ Do

```javascript
    user.type(input, 'hello world');
    await waitFor(() => {
        expect(list).toHaveLength(3)
    })
```

❌ Don't

```javascript
    await waitFor(() => {
        user.type(input, 'hello world');
        expect(list).toHaveLength(3)
    })
```
