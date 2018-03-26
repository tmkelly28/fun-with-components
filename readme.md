class: center, middle

# Fun with Components
---
class: center, middle

# But first, let's review JSX...
---

## Q: What does this get compiled to?

  ```jsx
  <MyComponent />
  ```

--

## A:

  ```js
  React.createElement(MyComponent)
  ```

---

# Q: What if MyComponent isn't a component?

  ```jsx
  const MyComponent = 'I am not a component.'

  <MyComponent />
  ```

--

# A: Same thing.

  ```js
  React.createElement(MyComponent)
  ```

--

You actually probably know this already.

When you fail to properly export a Component, it's *React* that complains
(in the browser's console at runtime), rather than the compiler (in your terminal
at build time).

---

# How JSX understands components

## (spoiler: it doesn't)

--

- JSX doesn't know or care if the component you name exists.
- It doesn't look up components by filename.
- The JSX compiler just generates JS *assuming* that there is
  (or will be) a component of that name in scope by the time
  the JS runs.

---
# Q: What does this compile as?

  ```jsx
  <div />
  ```
--
# A:

  ```js
  React.createElement('div')
  ```
---
# Why the quotes?
- For HTML tags, JSX creates elements whose types are strings
---

# Q: How does it know the difference?

  ```jsx
  <Div />  // Becomes React.createElement(Div)
  <div />  // Becomes React.createElement('div')
  ```

--

# A: Yep, it literally looks at the capitalization.

If your JSX tag begins with a capital letter, it generates
a React Element whose type is a Component

When the Element is rendered, that Component will get mounted:
  - If it's a pure function, it'll get called to generate
    its render output
  - If it's a class, it'll get instantiated with `new`, its
    lifecycle methods will get called, and `render` will
    be called to generate its render output.

Otherwise, it generates a React Element whose type is a string.
These Elements become HTML tags of that name when they're rendered.

---

# Q: What about props?

  ```jsx
  <MyComponent title='hello' foo={bar} />
  ```
--
# A: Props are the second argument to `createElement`

  ```js
  React.createElement(MyComponent, {title: 'hello', foo: bar})
  ```  

---

# Q: What about nested elements?

  ```jsx
  <MyComponent title='hello'>
    <h1>Hi there</h1>
    <AnotherComponent />
  </MyComponent>
  ```
--
# A: Children are the third argument to `createElement` 😉

  ```js
  React.createElement(MyComponent, {title: 'hello'}, [
    React.createElement('h1'),
    React.createElement(AnotherComponent),
  ])
  ```

---

# A: children also gets put on props, so this also works:

  ```js
  React.createElement(MyComponent, {
    title: 'hello',
    children: [
      React.createElement('h1'),
      React.createElement(AnotherComponent),
    ]
  })
  ```

---
class: center, middle

# Now let's play.

---
