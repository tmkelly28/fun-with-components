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

Instead of this,

  ```jsx
  <MyComponent />
  ```

What if I did this,

  ```jsx
  const stuff = MyComponent =>
    <MyComponent />
  ```

Or even this:

  ```jsx
  const stuff = MyComponent =>
    React.createElement(MyComponent)
  ```

--

# Is this useful?

---

# Components as props

I guess I could call it from a component.

  ```jsx
  const ShowStuff = () => <div>{stuff(MyComponent)}</div>
  ```

--

Which is basically this,

  ```jsx
  const ShowStuff = () => <div><MyComponent /></div>
  ```

--

Or I could make Stuff a component itself:

  ```jsx
  const Stuff = ({MyComponent}) =>
    <MyComponent />
  ```

--

Now, we can pass a component into `Stuff` *as a prop*:

  ```jsx
  <Stuff MyComponent={SomeComponent} />
  ```

---

These patterns (known as *components as props* and *render props*)
separate logic from presentation.

They're *super useful*

For example, React Router takes a component prop for its Routes:

  ```jsx
  <Route path='/products/:id' component={OneProduct} />
  ```

--

Or, it can also take a render prop:

  ```jsx
  <Route path='/products/:id' render={
    ({match: {params: {id}}}) => <OneProduct id={id} />
  } />
  ```

---

# Render props

This Maybe component takes a render prop, because it wants to
pass an arbitrary value that may not be an object.

  ```jsx
  const Maybe = ({
    value=null,
    then=_ => null,
    otherwise,
  }) =>
    (value !== null && value !== undefined)
      ? then(value)      // Note: called as a function,
                         // not mounted as a child.
      : otherwise
  ```

--

Usage:

  ```jsx
  <Maybe value={user}
    then={user => `Hi, ${user.name}`}
    otherwise={'Sign in to see a greeting.'}
    />
  ```

---

# Render props & components as props

"Render props" and "components as props" are extremely
similar patterns.

This isn't surprising, since React components are functions.

The main differences:
  - Render props don't have to be the shape of a React component
    - That is, render props can take arguments other than a single
      props object
  - Render props do not take Components which will be mounted in
    the tree. Instead, render functions are called once, from inside
    the container component's render method.

---

# Render functions as children

An increasingly common pattern is using a function as a Component's
children, and treating that function as a render prop:

  ```jsx
  <Animate>{time => <h1>The time is now {time}</h1>}</Animate>
  ```

--

(I'll give this a second to blow your mind.)

---

The trick here is that you can just *call* `props.children` if you
were passed a function. Here's the implementation for that component:

  ```jsx
  class Animate extends React.Component {
    state = {time: 0}

    componentDidMount() {
      this.tick()
    }

    tick = time => {
      this.raf = requestAnimationFrame(this.tick)
      this.setState({time})
    }

    componentWillUnmount() {
      cancelAnimationFrame(this.raf)
    }

    render() {
      // Call props.children with the current time.
      // Our children must be a function!
      this.props.children(this.state.time)
    }
  }
  ```

---

Finally, here's a nifty component:

  ```jsx
  <Await promise={axios.get('/products/22')}
    loading='Fetching product...'
    error={({message}) => 'Error loading product: ${message}`}
  >{
    product => <OneProduct {...product}>
  }</Await>
  ```

--

And here's the implementation:

  ```jsx
  class Await extends React.Component {
    state = {show: this.props.loading || 'Loading...'}

    componentDidMount() {
      this.props.promise
        .then(this.props.children)
        .catch(this.props.error || err => err.message)
        .then(show => this.setState({show}))
    }

    render() {
      return this.state.show
    }
  }
  ```


