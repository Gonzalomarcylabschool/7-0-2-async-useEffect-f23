# Fetching and useEffect

We've already learned about one hook, `useState`. Time for another one! In this lesson, we'll learn how to use the `useEffect` hook to send API fetch requests.

**Table of Contents**
- [Terms](#terms)
- [Fetching with event handlers](#fetching-with-event-handlers)
- [Challenge 1: Make a Dog API app](#challenge-1-make-a-dog-api-app)
- [useEffect](#useeffect)
  - [useEffect Syntax](#useeffect-syntax)
  - [The Effect callback](#the-effect-callback)
  - [The Dependency Array](#the-dependency-array)
- [Challenge 2: Fetch On Render](#challenge-2-fetch-on-render)
- [Fetching With a Form On Change](#fetching-with-a-form-on-change)
- [Quiz](#quiz)

## Terms

- **Side effect** — Anything that happens outside of React such sending a `fetch` request, starting an animation, or setting up a server connection. 
  - Side effects can be triggered by user events like submitting a form or clicking a button.
- **`useEffect`** – A react hook for executing "side effects" caused by a component rendering, not a particular event.
  - **Hooks** — Functions that provide a wide variety of features for React components. They all begin with `use()`.
- **Dependency Array** — The array of values provided to `useEffect` that React will watch for changes. If changes occur in the dependency array, the effect will run again.
- **Conditional Rendering** — Rendering different JSX depending on the current state. This can be useful when fetching to show either the fetched data or an error message if the fetch failed.


## Fetching with event handlers

> For a refresher on how to fetch, look at the `src/utils/fetchData.js` helper function. It returns an array with two values `[data, error]` (a "tuple").

In React, sending a fetch request is referred to as an "effect" or "side effect" since it happens outside of the normal scope of what React handles. 

Side effects are often executed in event handlers.

Check out the `1-joke-fetch-on-click` React project. In our application, we can render utilize a random joke API to send a fetch request in response to a button click:

```jsx
const JOKE_API_URL = "https://v2.jokeai.dev/joke/Pun?blacklistFlags=nsfw,religious,political,racist,sexist,explicit&type=twopart";

const defaultJoke = {
  setup: "What do you call a pile of cats?",
  delivery: "A meowntain",
};

function App() {
  // Create state for the fetched data
  const [joke, setJoke] = useState(defaultJoke);
  // Always create state to store any errors
  const [error, setError] = useState('');

  // Make the event handler async
  const handleClick = async () => {
    const [data, error] = await fetchData(JOKE_API_URL);
    if (data) setJoke(data);
    if (error) setError(error);
  }

  // Conditional Rendering
  if (error) return <p>{error.message}</p>

  return (
    <>
      <button onClick={handleClick}>Get Random Joke</button>

      <div className="joke">
        <h1>{joke.setup}</h1>
        <p>{joke.delivery}</p>
      </div>
    </>
  );
}
```

This example demonstrates a few important concepts:
* When fetching, the fetched data should be stored in state (`joke`)
* We should also make a piece of state to store an error if one is returned (`error`)
* We can use conditional rendering to render an error message if there was one.

## Challenge 1: Make a Dog API app

1. Create the app
```
npm create vite@latest
# Name it dog-fetcher
# Select React
# Select JavaScript
cd dog-fetcher
npm i
npm run dev
```

2. Then, copy the the `src/utils` folder from the `1-joke-fetch-on-click` folder into your own `src/` folder. Don't forget to `import` it into your `App.jsx` file!
3. Replace the `App` contents with your own app that has a `<button>` and an `<img>`. 
4. The `App` should have a `dogPicture` and an `error` state
5. When the user clicks on the button, it should send a fetch to https://dog.ceo/api/breeds/image/random and update either the `dogPicture` or `error` state depending on the returned value
6. The `img` should render the `dog` state or the `error` message.

**<details><summary style="color: purple">Potential Solution</summary>**

```jsx
import { useState } from 'react'
import fetchData from './utils/fetchData'
import './App.css'

const DOG_API = "https://dog.ceo/api/breeds/image/random";

function App() {
  // Create state for the fetched data
  const [dog, setDog] = useState();
  // Always create state to store any errors
  const [error, setError] = useState('');

  // Make the event handler async
  const handleClick = async () => {
    const [data, error] = await fetchData(DOG_API);
    if (data) setDog(data.message);
    if (error) setError(error);
  }

  // Conditional Rendering
  if (error) return <p>{error.message}</p>

  return (
    <>
      <button onClick={handleClick}>Get Random Dog Picture</button>
      <img src={dog} alt="" />
    </>
  );
}

export default App
```

</details><br>

## useEffect

There are two ways to perform a side effect like fetching:
1. In response to user events
2. In response to the component rendering ("reacting to the component rendering")

In our current joke API app, we only send a fetch in response to the user clicking on the button. But what if we want to show a joke when the page first renders?

We can accomplish this with the hook `useEffect` — a react hook for executing "side effects" caused by a component rendering, not a particular event.

> **Q: How do we know that this is a hook?**

### useEffect Syntax

`useEffect` takes in two arguments:
1. A callback function
2. [optional] A "dependency array" 

It should be invoked at the top of the component, next to the other hooks used by the component (often below `useState`)

```jsx
function App() {
  const [joke, setJoke] = useState(defualtJoke);
  const [error, setError] = useState();

  // invoke useEffect at the top of the component, next to
  // the other hooks
  useEffect(() => {
    const doFetch = async () => {
      const [data, error] = await fetchData(JOKE_API_URL);
      if (data) setJoke(data);
      if (error) setError(error);
    };
    doFetch();
  }, []);

// handleClick
// return JSX to render the joke
}
```

Notice that this callback creates a `async doFetch` function that fetches, and sets the `joke` or the `error` state depending on what is returned.



### The Effect callback

Why do we need to define `doFetch` and then invoke it? Why not just make the callback itself async.

Unfortunately, we can't make the callback async — we get an error

```jsx
// Throws an error
useEffect(async () => {
  const [data, error] = await fetchData(JOKE_API_URL);
  if (data) setJoke(data);
  if (error) setError(error);
}, []);
```

So, inside of the callback, we make an `async` function that does the fetch and then invoke it immediately.

```jsx
function App() {
  const [joke, setJoke] = useState(defualtJoke);
  const [error, setError] = useState();

  useEffect(() => {
    const doFetch = async () => {
      const [data, error] = await fetchData(JOKE_API_URL);
      if (data) setJoke(data);
      if (error) setError(error);
    };
    doFetch();
  }, []);

// handleClick
// return JSX to render the joke
}
```

### The Dependency Array

`useEffect(effect, dependencyArray)` needs to accept an `effect` callback but the second argument `dependencyArray` is optional. There are three ways that we can provide this value:

```jsx
useEffect(effect); // execute after EVERY re-render
useEffect(effect, []); // only execute the effect once
useEffect(effect, [valueA, valueB]); // re-run the effect whenever the array changes between renders
```

- If the array is omitted, the effect is executed on EVERY render of the component.
- If the array is empty, the effect is only executed on the first render of the component.
- If the dependency array is provided, the effect will be only re-run on future renders if the values in the array change between renders.

## Challenge 2: Fetch On Render

Add to your dog API app by having it render a dog image on the first render (and only on that first render!)

**<details><summary style="color: purple">Potential Solution</summary>**

```jsx
import { useState, useEffect } from 'react'
import fetchData from './utils/fetchData'
import './App.css'

const DOG_API = "https://dog.ceo/api/breeds/image/random";

function App() {
  // Create state for the fetched data
  const [dog, setDog] = useState();
  // Always create state to store any errors
  const [error, setError] = useState('');

  useEffect(() => {
    const doFetch = async () => {
      const [data, error] = await fetchData(DOG_API);
      if (data) setDog(data.message);
      if (error) setError(error);
    }
    doFetch();
  }, []);

  // Make the event handler async
  const handleClick = async () => {
    const [data, error] = await fetchData(DOG_API);
    if (data) setDog(data.message);
    if (error) setError(error);
  }

  // Conditional Rendering
  if (error) return <p>{error.message}</p>

  return (
    <>
      <button onClick={handleClick}>Get Random Dog Picture</button>
      <img src={dog} alt="" />
    </>
  );
}

export default App
```

</details><br>

## Fetching With a Form On Change

A cool way to fetch is using a form whenever the text input changes:

```jsx
function App() {
  const [joke, setJoke] = useState(defaultJoke);
  const [error, setError] = useState();
  const [query, setQuery] = useState("");

  useEffect(() => {
    const doFetch = async () => {
      const [data, error] = await fetchData(JOKE_API_URL);
      if (data) setJoke(data);
      if (error) setError(error);
    };
    doFetch();
  }, [query]);

  const handleClick = async () => {
    const [data, error] = await fetchData(JOKE_API_URL);
    if (data) setJoke(data);
    if (error) setError(error);
  }

  if (error) return <p>{error.message}</p>

  return (
    <>
      <form>
        <input
          type="text"
          placeholder="query"
          value={query}
          onChange={(e) => setQuery(e.target.value)}
        />
      </form>

      <button onClick={handleClick}>Get Random Joke</button>

      <div className="joke">
        <h1>{joke.setup}</h1>
        <p>{joke.delivery}</p>
      </div>
    </>
  );
}
```


**<details><summary style="color: purple">Q: When / how many times will this effect run?</summary>**
> Each time the `onChange` event fires (every input change)
</details><br>


## Quiz

* When should you `fetch` using `useEffect` vs. an event handler?
