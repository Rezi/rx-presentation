---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: /rxjs.png
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS
css: unocss
---

# ReactiveX

<div class="pt-12">
 <!--  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span> -->
</div>

<div class="abs-br m-6 flex gap-2">
  <!-- <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a> -->

by Tomas Rezac

</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# How can we access data in JS?


### Synchronously

```ts
let variable = 123;
console.log(variable);
```

### Asynchronously

```ts {all|7|3,5|11|4|8-9}
let variable;
function runAfterDelay(callback){
  setTimeout(()=>{
    callback()
  },300)
}
runAfterDelay(()=>{
  variable = 123;
  console.log(variable)
})
console.log(variable);
```

<style>
 h3 {
  padding-block:1rem;
} 
</style>
----
# Accessing data with promises?

### Waiting for value to be resolved before using it (Promises)

```ts {all|3,4,7|9|11|5-6}
let variable;

const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    variable = 123;
    resolve(variable);
  }, 300);
});
console.log(variable); // undefined

myPromise.then(console.log); //123
```

If don't you call `myPromise.then(console.log);` straight away in the code but rather after 300ms you get the value logged in console just after calling the **then**.

- Promises are superior to callbacks, they can work with **async await**, have helper functions like `all()`, `race()`, `any()` and can be be chained.
<style>
 h3 {
  padding-block:1rem;
} 
</style>


----
# Accessing data with observables?

### Can be synchronous or asynchronous, depends on data producer.

```ts {all|2-4|6-9|11-16|18}
let variable;
const observable = new Observable((observer) => {
  observer.next(Math.random());
});

observable.subscribe((data1) => {
  console.log(data1); // 0.24234234
  variable = data1;
});

observable.subscribe({
  next: (data2) => {
    console.log(data2); // 0.89787687
    variable = data2;
  }
});

console.log(variable); // 0.89787687
```


<style>
 h3 {
  padding-block:1rem;
} 
</style>


---

# ReactiveX Vocabulary

- Pull vs Push
- Observable (hot/cold)
- Subscription
- Observer
- operator
- stream
- pipe
- multicasting


<!--
Here is another comment.
-->

---



# Pull vs Push
  - **Pull** - data are used by consumer whenever consumer decide to use them.
    - consumer decides when the data is received
    - provider provides data upon request
    - **Advantage:** data are requested only when needed
  - **Push** - data can be used by consumer just after producer send them.
    - data are not explicitly requested by consumer.
    - producer decides when data can be used.
    - **Advantages:**
      - Client has data in real time
      - Producer only produce data when they change

ReactiveX uses **Push** data flow




<!--
Here is another comment.
-->

---

# Observable 

- is an Object with built in functions implementing the **Observer Design Pattern:**
  - Subject: It is considered as the keeper of information, of data or of business logic.
  - Register/Attach: Observers register themselves to the subject because they want to be notified when there is a change.
- Observable has Monadic structure:
  - "Monads are a kind of abstract data type constructor that encapsulate program logic instead of data in the domain model."[^1]
- In ReactiveX Subject (Observable) needs to be subscribed (Registered/Attached) in order to start producing data.
- new Observable(callback) 


[^1]: [Introduction to Rx](http://introtorx.com/Content/v1.0.10621.0/10_LeavingTheMonad.html)

<!--
Here is another comment.
-->

---

# Subscription 

- Act of a consumer requesting access to observe a producer
- Subscription is finalized on:
  - `error`: When some of the operator functions failed.
  - `complete`: Observable knows there won't be more values comming
  - `unsubscription`: We are not interesed in future values in the stream and enforce its closing by .unsubscribe()


<!--
Here is another comment.
-->

---

# Observer 

- is an Object with functions, consuming values delivered by Observable:

```
const observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};
```

- it is used as argument of `subscribe` method:
  - `.subscribe({next, error, complete})`
- if only callback functions are provided, without Observer object, those functions are internally turned to the Observer object
  - `.subscribe(next, error, complete)` are turned to `.subscribe({next, error, complete})`

<!--
Here is another comment.
-->
---

# Operator (pipeable)

- A factory function that creates an **operator function**.

=>

# Operator function
  - Is a pure function
  - an Observable transformer in chain between Initial Observable and Observer **.subscribe(Observer)**
  - Each operator function returns a new *Observable* to continue an observation chain - also known as a “stream” in reactiveX.
  - Basic operator function actually `.subscribe()` to the input Observable and create new Observable on which it calls `.next()`, then returns it. Calling `next()` invoke the next operator in the chain to get value in Observer `.subscribe()`

---

# .pipe()

- Pipe is a simple helper function which call operator functions passed to it one by one. 
- Each operator function in pipe takes output of the previous (Observable) or initial Observable, as argument and return new Observable, which is passed to next operator function or subscriber.

```ts

class Observable {
  pipe(...operators): Observable<any> {
    return operators.reduce((source, operatorFn) => operatorFn(source), this);
  }
}
```


- Usage

  ```ts
  myObservable.pipe(
    someOperator(),
    otherOperator((dataInStream)=> transformationOf(dataInStream)),
  ).subscribe(console.log) 
  ```


---
layout: 7-5
---

# Operator example


```ts{1,18|2,17|3,16|5-15|all}
function myMap<T, R>(callback: (input: T, index: number) => R) {
	return function (source: Observable<T>): Observable<R> {
		return new Observable((subscriber) => {
			let index = 0;
			return source.subscribe({
				next(value) {
					subscriber.next(callback(value, index++));
				},
				error(error) {
					subscriber.error(error);
				},
				complete() {
					subscriber.complete();
				}
			});
		});
	};
}
```

::right::

# Usage

```ts
import { interval } from 'rxjs';
import { map } from 'rxjs/operators';

interval(1000).pipe(
  myMap((num, index) => num + index)
).subscribe(console.log) // 0, 2, 4, 6
```



---

# Recycling operators, when creating new ones


```ts
import { map } from "rxjs";

function powerOfTwoMap() {
	return (source: Observable<number>): Observable<number> => {
		return source.pipe(map((number:number)=> number ** 2));
	};
}
```
&nbsp; 

# Simplified

```ts
import { map } from "rxjs";

function powerOfTwoMap() {
	return map((number:number) => number ** 2);
}
```

---


# Observable typology:


- Hot Observable 
  - When its producer's value was created outside of the context of the subscribe action
  - Same value from producer is **shared** among many subscribers
  - Contains list of of subscribers, when the the list length is 0, subscription is finalized.
- Cold Observable 
  - creates a new producer during subscribe for every new subscription. Is unicasting by design.
- Multicasting
  - The act of one producer being observed by many consumers.
- Unicasting
  - The act of one producer being observed only one consumer
  - Not necessarily must be cold.


<!--
Here is another comment.
-->

---

# Special Observables

- ReactiveX includes special types of Observable: **Subject, BehaviorSubject, AsyncSubject, ReplaySubject**
- All **Subjects** are **Multicasting** by default
- Every Subject is an **Observable** and **Observer** at the same time
  - you can **.subscribe()** to it as to **Observable**
  - you can call **.next()**, **.error()** and **.complete()** on it as on Observer

```
const subject = new Subject<number>();
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});
 
subject.next(1);
subject.next(2);
// observerA: 1, observerB: 1
// observerA: 2, observerB: 2
```
<!--
Here is another comment.
-->

---

# What is it good for?

- ReactiveX lets you build event driven apps without imperative code.
- Simplifying asynchronous programming by providing unified API across different data sources and types.
- Enhancing code reusability and maintainability
- Provides parallel processing by default (great scalability)
- Facilitating interoperability between different programming languages and platforms through its wide language support.

---

# Example

```ts {1-15|19-26|27-28|29-47|48-49|52-56|57-58|all} {maxHeight:'400px'}
import { Observable, filter, map, scan, throttleTime } from 'rxjs';

import { webSocket } from 'rxjs/webSocket';

// Define the WebSocket URL for the stock exchange
const wsUrl = 'wss://foo-stock-exchange.com';

interface StockData {
	symbol: string;
	price: string;
	timestamp: number;
}

// Create an observable that listens for incoming data from the web socket
const stockData$: Observable<StockData> = webSocket(wsUrl);

// Apply a series of operators to the stock data observable to transform, filter, and aggregate the data as needed
const processedData$ = stockData$.pipe(
	// Map the raw data to a more usable format
	map((rawData: StockData) => {
		return {
			symbol: rawData.symbol,
			price: parseFloat(rawData.price),
			timestamp: new Date(rawData.timestamp)
		};
	}),
	// Filter out any unwanted data
	filter((data) => data.symbol !== 'AAPL'),
	// Aggregate the data over time
	scan(
		(acc, data) => {
			return {
				symbol: data.symbol,
				maxPrice: Math.max(acc.maxPrice, data.price),
				minPrice: Math.min(acc.minPrice, data.price),
				latestPrice: data.price,
				timestamp: data.timestamp
			};
		},
		{
			symbol: '',
			maxPrice: 0,
			minPrice: 0,
			latestPrice: 0,
			timestamp: new Date()
		}
	),
	// Limit the frequency of updates to the UI
	throttleTime(1000)
);

// Subscribe to the processed data observable to receive the data and update the UI
const mySubscription = processedData$.subscribe((data) => {
	console.log(data);
	// Update the UI with the processed data
});
// Unsubscribe after 50s
setTimeout(mySubscription.unsubscribe, 50000);
```

---

# Marble diagrams

![Local Image](/marble-diagram-anatomy.svg)

<style>
p {
  opacity: 1 !important;
}
</style>

---
layout: center
---

# Show time

---
layout: iframe

# the web page source
url: https://rx-cars.netlify.app/

---
---
layout: center
---

# Questions?


<!-- ---


# Code


<arrow v-click="1" x1="400" y1="420" x2="230" y2="330" color="#564" width="3" arrowSize="1" />

[^1]: [Learn More](https://sli.dev/guide/syntax.html#line-highlighting)

<style>
.footnotes-sep {
  @apply mt-20 opacity-10;
}
.footnotes {
  @apply text-sm opacity-75;
}
.footnote-backref {
  display: none;
}
</style>
 -->
