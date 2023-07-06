
# JSX Intro

Take a look at the following line of code:

```
const h1 = <h1>Hello world</h1>;
```

What kind of weird hybrid code is that? Is it JavaScript? HTML? Or something else?

It seems like it must be JavaScript since it starts with `const` and ends with `;`. If you tried to run that in an HTML file, it wouldn't work.

However, the code also contains `<h1>Hello world</h1>`, which looks exactly like HTML. *That* part wouldn't work if you tried to run it in a JavaScript file.
The answer is... a JavaScript file! Despite what it looks like, your code doesn't actually contain any HTML at all.

The part that looks like HTML, `<h1>Hello world</h1>`, is something called *JSX*.

*JSX* is a syntax extension for JavaScript. It was written to be used with React. JSX code looks a lot like HTML.

What does "syntax extension" mean?

In this case, it means that JSX is not valid JavaScript. Web browsers can't read it!

If a JavaScript file contains JSX code, then that file will have to be *compiled*. This means that before the file reaches a web browser, a *JSX compiler* will translate any JSX into regular JavaScript.


JSX Elements
------------

A basic unit of JSX is called a JSX *element*.

Here's an example of a JSX element:

```
<h1>Hello world</h1>
```

This JSX element looks exactly like HTML! The only noticeable difference is that you would find it in a JavaScript file, instead of in an HTML file.


Attributes In JSX
-----------------

JSX elements can have *attributes*, just like how HTML elements can.

A JSX attribute is written using HTML-like syntax: a *name*, followed by an equals sign, followed by a *value*. The *value* should be wrapped in quotes, like this:

```
my-attribute-name="my-attribute-value"
```

Here are some JSX elements with *attributes*:

```
<a href='http://www.example.com'>Welcome to the Web</a>; const title = <h1 id='title'>Introduction to React.js: Part I</h1>;
```

A single JSX element can have many attributes, just like in HTML:

```
const panda = <img src='images/panda.jpg' alt='panda' width='500px' height='500px'>;
```

There's a rule that we haven't mentioned: a JSX expression must have exactly *one* outermost element.

In other words, this code will work:

```
const paragraphs = (  <div id="i-am-the-outermost-element">    <p>I am a paragraph.</p>    <p>I, too, am a paragraph.</p>  </div>);
```

But this code will not work:

```
const paragraphs = (  <p>I am a paragraph.</p>   <p>I, too, am a paragraph.</p>);
```

The *first opening tag* and the *final closing tag* of a JSX expression must belong to the same JSX element!

It's easy to forget about this rule and end up with errors that are tough to diagnose.

If you notice that a JSX expression has multiple outer elements, the solution is usually simple: wrap the JSX expression in a `<div>` element.


class vs className
------------------

This lesson will cover more advanced JSX. You'll learn some powerful tricks and some common errors to avoid.

Grammar in JSX is mostly the same as in HTML, but there are subtle differences to watch out for. The most frequent of these involves the word `class`.

In HTML, it's common to use `class` as an attribute name:

```
<h1 class="big">Title</h1>
```

In JSX, you can't use the word `class`! You have to use `className` instead:

```
<h1 className="big">Title</h1>
```

This is because JSX gets translated into JavaScript, and `class` is a reserved word in JavaScript.

When JSX is *rendered*, JSX `className` attributes are automatically rendered as `class` attributes.


Self-Closing Tags
-----------------

Another common JSX error involves *self-closing tags*.

What's a self-closing tag?

Most HTML elements use two tags: an *opening tag* (`<div>`), and a *closing tag* (`</div>`). However, some HTML elements such as `<img>` and `<input>` use only one tag. The tag that belongs to a single-tag element isn't an opening tag or a closing tag; it's a *self-closing tag.*

When you write a self-closing tag in HTML, it is *optional* to include a forward slash immediately before the final angle bracket:

```
// Fine in HTML with a slash:<br /> // Also fine, without the slash:<br>
```

But, in JSX, you *have to* include the slash. If you write a self-closing tag in JSX and forget the slash, you will raise an error:

```
// Fine in JSX:<br /> // NOT FINE AT ALL in JSX:<br>
```


Variables in JSX
----------------

When you inject JavaScript into JSX, that JavaScript is part of the same environment as the rest of the JavaScript in your file.

That means that you can access variables while inside of a JSX expression, even if those variables were declared outside of the JSX code block.
// Declare a variable:\
const name ='Gerdo';

// Access your variable inside of a JSX expression:\
const greeting =<p>Hello, {name}!</p>;


Variable Attributes in JSX
--------------------------

When writing JSX, it's common to use variables to set *attributes*.

Here's an example of how that might work:

```
// Use a variable to set the `height` and `width` attributes: const sideLength = "200px"; const panda = (  <img     src="images/panda.jpg"     alt="panda"     height={sideLength}     width={sideLength} />);
```

Notice how in this example, the `<img />`'s attributes each get their own line. This can make your code more readable if you have a lot of attributes for one element.

*Object properties* are also often used to set attributes:

```
const pics = {  panda: "http://bit.ly/1Tqltv5",  owl: "http://bit.ly/1XGtkM3",  owlCat: "http://bit.ly/1Upbczi"};  const panda = (  <img     src={pics.panda}     alt="Lazy Panda" />); const owl = (  <img     src={pics.owl}     alt="Unimpressed Owl" />); const owlCat = (  <img     src={pics.owlCat}     alt="Ghastly Abomination" />);
```

Event Listeners in JSX
----------------------

JSX elements can have *event listeners*, just like HTML elements can. Programming in React means constantly working with event listeners.

You create an event listener by giving a JSX element a special *attribute*. Here's an example:

```
<img onClick={clickAlert} />
```

An event listener attribute's *name* should be something like `onClick` or `onMouseOver`: the word `on` plus the type of event that you're listening for. Look through the [common components list in React docs](https://react.dev/reference/react-dom/components/common#) to browse supported event names.

An event listener attribute's *value* should be a function. The above example would only work if `clickAlert` were a valid function that had been defined elsewhere:

```
function clickAlert() {  alert('You clicked this image!');} <img onClick={clickAlert} />
```

Note that in HTML, event listener *names* are written in all lowercase, such as `onclick` or `onmouseover`. In JSX, event listener names are written in camelCase, such as `onClick` or `onMouseOver`.


.map in JSX
-----------

The `.map()` array method comes up often in React. It's good to get in the habit of using it alongside JSX.

If you want to create a list of JSX elements, then using `.map()` is often the most efficient way. It can look odd at first:

```
const strings = ['Home', 'Shop', 'About Me']; const listItems = strings.map(string => <li>{string}</li>); <ul>{listItems}</ul>
```

In the above example, we start out with an array of strings. We call `.map()` on this array of strings, and the `.map()` call returns a new array of `<li>`s.

On the last line of the example, note that `{listItems}` will evaluate to an array, because it's the returned value of `.map()`! JSX `<li>`s don't have to be in an array like this, but they *can* be.


React.createElement
-------------------

You can write React code without using JSX at all!

The majority of React programmers do use JSX, but you should understand that it is possible to write React code without it.

The following JSX expression:

```
const h1 = <h1>Hello world</h1>;
```

can be rewritten without JSX, like this:

```
const h1 = React.createElement(  "h1",  null,  "Hello world");
```

When a JSX element is compiled, the compiler *transforms* the JSX element into the method that you see above: `React.createElement()`. Every JSX element is secretly a call to `React.createElement()`.

We won't go in-depth into how `React.createElement()` works, but check out [the React documentation on `createElement()`](https://react.dev/reference/react/createElement) to learn more.



# React 

React Components
----------------

React applications are made of components.

What's a component?

A component is a small, reusable chunk of code that is responsible for one job. That job is often to render some HTML and re-render it when some data changes.

Take a look at the code below. This code will create and render a new React component:

```
import React from 'react';import ReactDOM from 'react-dom/client'; function MyComponent() {  return <h1>Hello world</h1>;} ReactDOM.createRoot(document.getElementById('app')).render(<MyComponent />);
```

A lot of these look unfamiliar but do not worry. We are going to unpack that code, one small piece at a time. By the end of this lesson, you'll understand how to build a React component!


Import React
------------

The first React component we created in the last exercise started with importing `react`. The line that did this is:

```
import React from 'react';
```

This creates an object named `React`, which contains methods necessary to use the React library. `React` is imported from the `'react'` package, which should be installed in your project as a dependency. With the object, we can start utilizing features of the `react` library!


Another import we need in addition to React is ReactDOM:

```
import ReactDOM from 'react-dom/client';
```

The methods imported from `'react-dom'` interact with the DOM.

The methods imported from `'react'` do not deal with the DOM at all. They don't engage directly with anything that isn't part of React.


Create a Function Component
---------------------------

You've learned that a *React component* is a small, reusable chunk of code that is responsible for one job, which often involves rendering HTML and re-rendering it whenever some data changes.

It's useful to think of components as smaller pieces of our interface. Combined, they are the building blocks that make up a React application. In a website, we can create a component for the search bar, another component for the navigation bar, and another component for the dashboard content itself.

Here's another fact about components: we can use JavaScript functions to define a new React component. This is called a function component.

In the past, React components were defined using Javascript classes. But since the introduction of Hooks (something we'll discuss later), function components have become the standard in modern React applications.

After we define our functional component, we can use it to create as many instances of that component as we want.

Let's take a look at the example from the first exercise:

```
import React from 'react'; function MyComponent() {  return <h1>Hello, I'm a functional React Component!</h1>;} export default MyComponent;
```

On the third line, a function is defined with the name `MyComponent`. Inside, the function returns a React element in JSX syntax:

```
return <h1>Hello, I'm a functional React Component!</h1>;
```

Combined, this makes a basic React functional component.

On the last line of the above code block, `MyComponent` is exported so it can be used later.


Name a Functional Component
---------------------------

Good! Creating a JavaScript function is the way to declare a new *functional component*.

When you declare a new functional component, you need to give that component a *name.* On our finished component, our component's name was `MyComponent`:

```
function MyComponent() {  return <h1>Hello world</h1>;}
```

Function component names must start with capitalization and are conventionally created with PascalCase! Due to how JSX tags are compiled, capitalization indicates that it is a React component rather than an HTML tag.

This is a React-specific nuance! If you are creating a component, be sure to name it starting with a capital letter so it interprets it as a React component. If it begins with a lowercase letter, React will begin looking for a built-in component such as `div` and `input` instead and fail.

## Props
props
-----

When thinking in the frame of a React application, components are small pieces of a whole. Together, they make up the interface that users will see.

With each component playing a role in the interface, there are times when components must be able to communicate with other components.

In this lesson, you will learn another way that components can interact: a component *passing information* to another component.

Information that gets passed from one component to another is known as props.

Props can be used to customize the output of each component depending on the information that is passed in.

By allowing components to communicate with each other, we can add a level of flexibility that was not possible before.

By the end of this lesson, you should be able to:

-   Pass, access, and display props.
-   Use props to create conditional statements.
-   Define event handlers in a component and pass them to other components.
-   Work with a component's children.
-   Assign default values to props.

Access a Component's props
--------------------------

Every component has something called `props`.

A component's `props` is an object. It holds information about that component.

You've seen this before, but you might not have realized it! Let's take a look at the HTML button tag. There are several pieces of information we can pass to the button tag, such as the `type` of the button.

```
<button type="submit" value="Submit"> Submit </button>
```

In this example, we've passed two pieces of information to the button tag, a `type` and a `value`. Depending on what `type` attribute we give to the `<button>` element, it will treat the form differently. In the same way, we can pass information to our own components to specify how they behave!

Props serve the same purpose for components as arguments do for functions.

To access a component's `props` object, you can reference the `props` object and the dot notation for its properties. Here's an example:

```
props.name
```

This would retrieve the `name` property from the `props` object.


Pass `props` to a Component
---------------------------

To take advantage of `props`, we need to *pass information* to a React component. In the previous exercise, we rendered an empty `props` object because we did not pass any `props` to our `PropsDisplayer` component.

How do we pass `props`? By giving the component an *attribute*:

```
<Greeting name="Jamel" />
```

Let's say that you want to pass a component the message, `"We're great!"`. Here's how you can do it:

```
<SloganDisplay message="We're great!" />
```

As you can see, to pass information to a component, you need a *name* for the information that you want to pass.

In the above example, we used the name `message`. You can use any name you want.

If you want to pass information that isn't a string, then wrap that information in curly braces. Here's how you would pass an array:

```
<Greeting myInfo={["Astronaut", "Narek", "43"]} />
```

In this next example, we pass several pieces of information to `<Greeting />`. The values that aren't strings are wrapped in curly braces:


Render a Component's props
--------------------------

Props allow us to customize the component by passing it information.

We've learned how to *pass* information to a component's `props` object. You will often want a component to *display* the information that you pass.

To make sure that a function component can use the `props` object, define your function component with `props` as the parameter:

```
function Button(props) {  return <button>{props.displayText}</button>;}
```

In the example, `props` is accepted as a parameter, and the object values are accessed with the dot notation accessors pattern (`object.propertyName`).

Alternatively, since `props` is an object, you can also use destructuring syntax like so:

```
function Button({displayText}) {  return <button>{displayText}</button>;}
```


Pass props From Component To Component
--------------------------------------

You have learned how to pass a `prop` to a component:

```
<Greeting firstName="Esmerelda" />
```

You have also learned how to access and display a passed-in `prop`:

```
return <h1>{props.firstName}</h1>;
```

The most common use of `props` is to pass information to a component from *a different component*.

Props in React travel in a one-way direction, from the top to bottom, parent to child.

Let's explore the parent-child relationship of passing `props` a bit further.

```
function App() {    return <Product name="Apple Watch" price = {399} rating = "4.5/5.0" />;}
```

In this example, `App` is the parent and `Product` is the child. `App` passes three props to `Product` (`name`, `price`, and `rating`), which can then be read inside the child component.

Props passed down are immutable, meaning they cannot be changed. If a component wants new values for its props, it needs to rely on the parent component to pass it new ones.


Giving Default Values to props
------------------------------

Take a look at the `Button` component. Notice that on line 8, `Button` expects to receive a prop named `text`. The received `text` will be displayed inside of a `<button>` element.

What if nobody passes any `text` to `Button`?

If nobody passes any `text` to `Button`, then `Button`'s display will be blank. It would be better if `Button` could display a default message instead.

You can make this happen by specifying a default value for the prop. There are three ways to do this!

The first method is adding a `defaultProps` static property to the component:

```
function Example(props) {  return <h1>{props.text}</h1>} Example.defaultProps = {  text: 'This is default text',};
```

You can also specify the default value directly in the function definition:

```
function Example({text='This is default text'}) {   return <h1>{text}</h1>}
```

Lastly, you can also set the default value in the function body:

```
function Example(props) {  const {text = 'This is default text'} = props;  return <h1>{text}</h1>}
```

If an `<Example />` doesn't get passed any text, then it will display "This is default text".

If an `<Example />` *does* get passed some text, then it will display that passed-in text.
