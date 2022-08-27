---
title: Post Two
date: '2021-12-15'
---

### Variables

You can define variables in your `<script>` tag and then use it with brackets:

```svelte
<script>
	let foo = 'bar';
</script>

<p class={bar}>Some text</p>
```

> **Tip**: It also works inside double-quotes: `<p class="primary-{bar}">Some text</p>`

Moreover, when var = attribute, (`src={src}`) you can use it this way: `{src}`.

### Reactivity

**Reactivity is triggered by assignments**
What does it mean? Every change will be triggered after an assignment has been done.

More on that later.

##### Variables

Sometimes a variable is result from a computation, you'll have to use a _reactive declaration_:

```js
let count = 0;
$: doubled = count * 2;
```

##### Statements

We're not limited to variable declaration, we can also use reactive statements:

```js
$: console.log(`the count is ${count}`);
```

Here `console.log` is called whenever `count` value changes.
Notice the usage of _`_ (backquotes).

> **Tip**: You can group more statements by using `$: { ... }`.

##### Updating objects

Because Svelte's reactivity is **triggered** by **assignments** following code won't do anything when called:

```js
function addNumber() {
	numbers.push(numbers.length + 1);
}
```

Unless you add it this assignment:

```js
function addNumber() {
	numbers.push(numbers.length + 1);
	numbers = numbers;
}
```

### Props

There's an easy way to pass props to a child component. Just `export` it !

```svelte
// Nested.svelte
<script>
  export let answer = 42
</script>
<p>The answer is {answer}</p>

// App.svelte
<script>
  import Nested from './Nested.svelte'
</script>

<Nested/> // Will display: The answer is 42
```

> **Tip**: You can also use `<Nested answer={42}/>` to set a default value to a prop that hasn't been initialized into it's own component.

When passing multiple **props** typically as an object of properties, you can **spread** them to a component instead of specifying each one with `...`:

```svelte
<script>
	import Info from './Info.svelte';

	const pkg = {
		name: 'svelte',
		version: 3,
		speed: 'blazing',
		website: 'https://svelte.dev'
	};
</script>

<Info name={pkg.name} version={pkg.version} speed={pkg.speed} website={pkg.website} />
<Info {...pkg} />
```

Assuming we have `export`ed `name`, `version`, and so on in the `Info` component.

### Logic

#### Conditions

To conditionally render some markup, we wrap it in an if block:

```svelte
{#if user.loggedIn}
	<button on:click={toggle}> Log out </button>
{/if}

{#if !user.loggedIn}
	<button on:click={toggle}> Log in </button>
{/if}
```

You can also make use of :

- `{:else}`
- `{:else if .. }`

#### Loops

Expressed this way:

```svelte
{#each cats as cat, i}
	<li>
		<a target="_blank" href="https://www.youtube.com/watch?v={cat.id}">
			{i}: {cat.name}
		</a>
	</li>
{/each}
```

> **Tip**: You can also de-structure as: `{#each cats as { id, name }}` and then use `{name}` instead of `{cat.name}`

When a value isn't **exported** but is computed by a **prop**, you can go into problems when updating values since it's a _copy_ and not a _reference_.
Thus, be sure to **identify** the right object when updating its value(s):

```svelte
{#each cats as cat, i (cat)}
	// Notice the (cat)
	<li>
		<a target="_blank" href="https://www.youtube.com/watch?v={cat.id}">
			{i}: {cat.name}
		</a>
	</li>
{/each}
```

#### Promises

You can deal with **promises** and display at each of the promise lifecycle:

```svelte
{#await promise}
	<p>Loading...</p>
{:then result}
	<p>It returns {result}</p>
{:catch error}
	<p>An error occurred: {error.message}</p>
{/await}
```

> **Tip**: You can also skip loading step by using: `{#await promise then result}`

### Events

You can bind functions / events to your tags:

```svelte
<script>
	let count = 0;

	function handleClick() {
		count += 1;
	}
</script>

<button on:click={handleClick}>
	Clicked {count}
	{count === 1 ? 'time' : 'times'}
</button>
```

You can use all javascript event (click, mousemove, ...).
You can also _modify_ event by _piping_ **modifiers** to it:
`<button on:click|once={handleClick}>`
You can _chain_ them by adding pipes.

#### Event dispatcher

You can create an event dispatcher inside a component.
It must be called when component is first instantiated.

```svelte
<script>
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function sayHello() {
		dispatch('message', {
			text: 'Hello!'
		});
	}
</script>
```

On the other side:

```svelte
<script>
	import Inner from './Inner.svelte';

	function handleMessage(event) {
		alert(event.detail.text);
	}
</script>

<Inner on:message={handleMessage} /> // on:eventname
```

#### Event forwarding

You can handle events from any other component by calling them with their own event:

```svelte
<script>
	import Inner from './Inner.svelte';
</script>

// Outer.svelte
<Inner on:message />
```

Here we can intercept event `message` from `Inner`.
Then, **when called**, we can define how the component **reacts**:

```svelte
<Outer on:message={handleMessage} />
```

### Bindings

Data flow is _top down_, that means a **parent** component can **set props on a child** component but **not the other way** around.

**Bindings** can break that rule:

```svelte
<script>
	let name = 'world';
</script>

<input bind:value={name} /> // name and value are tied together, they are bind

<h1>Hello {name}!</h1>
```

You can also use a **group** of bindings:

```svelte
<script>
	let flavourList = ['Cookies and cream', 'Mint choc chip', 'Raspberry ripple'];
</script>

<h2>Flavours</h2>

{#each flavourList as flavour}
	<label>
		<input type="checkbox" bind:group={flavourList} value={flavour} />
		{flavour}
	</label>
{/each}
```

### Lifecycle

Is ruled by a bunch of functions:

- `onMount`

```svelte
<script>
	import { onMount } from 'svelte';

	let photos = [];

	onMount(async () => {
		const res = await fetch(`https://jsonplaceholder.typicode.com/photos?_limit=20`);
		photos = await res.json();
	});
</script>
```

- `onDestroy`
- `beforeUpdate`
- `afterUpdate`

### Store

A store is simply an object with a **subscribe** method that allows interested parties to be **notified** whenever the **store value** changes:

```svelte
<script>
	let countValue;
	const unsubscribe = count.subscribe((value) => {
		count_value = value;
	});
	onDestroy(unsubscribe);
</script>

<h1>The count is {count_value}</h1>
```

Then, you can **update** the stored value:

```svelte
<script>
	function increment() {
		count.update((n) => n + 1);
	}
</script>
```

Or set a value:

```svelte
<script>
	function reset() {
		count.set(0);
	}
</script>
```

> **Best practice**: Auto-subscription

#### Usage

You can use auto-subscription to get rid of `subscribe` and `onDestroy` methods:

```svelte
<script>
	import { storeValue } from './AnotherComponent';
	// ...other imports come after
</script>

<h1>The count is {$storeValue}</h1>
```

> **Warning**: The store value must be imported or declared at the top-level scope of a component !

#### Readable store

It's a read-only store.

```svelte
<script>
	export const time = readable(new Date(), function start(set) {
		// Implementation
		set();

		return function stop() {
			// Implementation
		};
	});
</script>
```

The **first argument** can be null, it represents an **initial value**.
You then have to define a `set` callback which is called when the store gets its **first subscriber** and a `stop` callback when the last subscriber unsubscribes.

#### Writable store

```svelte
<script>
	import { writable } from 'svelte/store';

	const count = writable(0, () => {
		console.log('got a subscriber');
		return () => console.log('no more subscribers');
	});
</script>
```

It can be altered with `set` and `update`. It can also be `bind` to a tag.

#### Derived store

...from another store.

```svelte
<script>
	import { derived } from 'svelte/store';

	const delayed = derived(
		time,
		($a, set) => {
			setTimeout(() => set($a), 1000);
		},
		'one moment...'
	);

	// or (read-only derived):

	const elapsed = derived(time, ($time) => Math.round(($time - start) / 1000));
</script>
```

First argument is the original store. **Second** one is **facultative** and added if you want to **write** on the store.

#### Custom store

```js
// store.js

import { writable } from 'svelte/store';
function createCount() {
	const { subscribe, set, update } = writable(0);

	return {
		subscribe,
		increment: () => update((n) => n + 1),
		decrement: () => update((n) => n - 1),
		reset: () => set(0)
	};
}
export const count = createCount();
```

In the component:

```svelte
<script>
	import { count } from './store.js';
</script>

<h1>The count is {$count}</h1>

<button on:click={count.increment}>+</button>
<button on:click={count.decrement}>-</button>
<button on:click={count.reset}>reset</button>
```

### Motion

Stores are cool but you can make them even smoother by adding animation when transposed to UI.

#### Tweened

It's usable like an ordinary store:

```svelte
<script>
	import { tweened } from 'svelte/motion';
	import { cubicOut } from 'svelte/easing';

	const progress = tweened(0, {
		duration: 400,
		easing: cubicOut
	});
</script>

<progress value={$progress} />

<button on:click={() => progress.set(0)}>0%</button>
```

#### Spring

...works better with frequently changing value.

```svelte
<script>
	import { spring } from 'svelte/motion';

	let coords = spring(
		{ x: 50, y: 50 },
		{
			stiffness: 0.1,
			damping: 0.25
		}
	);
	let size = spring(10);
</script>
```

### Transitions

There're a lot:

- Fade in/out: `transition:fade` (ie: `<p transition:fade>Some text</p>`)
- Fly (disappear in a given direction): `transition:fly`
- ...

### Dynamic classes

To affect a `class` to a tag dynamically when condition are met:

```js
<button
  class:active="{current === 'foo'}"
>
  Text
</button>

<style>
.active {
  background-color: #ff3e00;
  color: white;
}
</style>
```

Here we affect the class `.active` when condition is true.

### Slots

Slots give you the opportunity to shape a component inside another one.

```svelte
<script>
	import Box from './Box.svelte';
</script>

<Box>
	<h2>Hello!</h2>
	<p>This is a box. It can contain anything.</p>
</Box>

// Box.svelte
<div class="box">
	<slot>A default value here</slot>
</div>
```

Slots can be named:

```svelte
<script>
	import ContactCard from './ContactCard.svelte';
</script>

<ContactCard>
	<span slot="name">T. Pellegatta</span>
</ContactCard>

// ContactCard.svelte
<article class="contact-card">
	<h2>
		<slot name="name">
			<span class="missing">Unknown name</span> // Will display T. Pellegatta because it has been defined
		</slot>
	</h2>

	<div class="address">
		<slot name="address">
			<span class="missing">Unknown address</span>
		</slot>
	</div>
</article>
```

You can pass props to slots:

```svelte
<!-- App.svelte -->
<FancyList {items}>
	<div slot="item" let:item>{item.text}</div>
	<p slot="footer">Copyright (c) 2019 Svelte Industries</p>
</FancyList>

<!-- FancyList.svelte -->
<ul>
	{#each items as item}
		<li class="fancy">
			<slot name="item" {item} />
		</li>
	{/each}
</ul>

<slot name="footer" />
```

The `let:` directive goes on the element with the slot attribute.

### Contexts

Almost the same as **stores** but **less reactive**.
[Documentation](https://svelte.dev/docs#setContext)

### Debugging

Instead of using `debugger;` use `{@debug variable}`

## Integrations

### GraphQL integration

...with Apollo

You'll have to import some plugins:

- `apollo-boost`
- `graphql`
- `svelte-apollo`

Then, in the main component instantiate a client:

```svelte
<script>
	import ApolloClient from 'apollo-boost';
	import { setClient } from 'svelte-apollo';
	const client = new ApolloClient({
		uri: 'http://site/graphqlApi/endpoint'
	});
	setClient(client);
</script>
```

In another one (or the same) use it to fetch/push data:

```svelte
<script>
  import { getClient, query } from 'svelte-apollo'
  import { gql } from 'apollo-boost'

  const GET_TODOS = gql`
    {
      getTodos {
        id
        text
      }
    }
  `

  const client = getClient()
  const todos = query(client, { query: GET_TODOS })
</script>


{#await $todos}
  Loading...
{:then result}
  <p>Total todos: {result.data.getTodos.length}</p>
  <ul>
    {each result.data.getTodos as todo}
      <li>{todo.text}</li>
    {/each}
  </ul>
{:catch error}
  <p>{error.message}</p>
{/await}
```
