# RxJS Operators and Concepts with Examples

RxJS is a powerful library for reactive programming in JavaScript. Let me explain the operators and concepts you've listed with practical examples.

## Transformation Operators

### 1. map
Transforms each value emitted by the source Observable.

```typescript
import { of } from 'rxjs';
import { map } from 'rxjs/operators';

const source$ = of(1, 2, 3);
const mapped$ = source$.pipe(
  map(x => x * 10)
);

mapped$.subscribe(x => console.log(x));
// Output: 10, 20, 30
```

### 2. switchMap
Projects each source value to an Observable, then flattens it, but cancels previous inner Observables when a new source value arrives.

```typescript
import { fromEvent, interval } from 'rxjs';
import { switchMap } from 'rxjs/operators';

const button = document.getElementById('myButton');
const clicks$ = fromEvent(button, 'click');

// On each click, start a new interval, canceling the previous one
const result$ = clicks$.pipe(
  switchMap(() => interval(1000))
);

result$.subscribe(x => console.log(x));
// Output: 0, 1, 2... (resets on each click)
```

### 3. mergeMap (flatMap)
Projects each source value to an Observable and merges them, allowing multiple inner Observables at once.

```typescript
import { of } from 'rxjs';
import { mergeMap, delay } from 'rxjs/operators';

const source$ = of('Hello', 'World');
const result$ = source$.pipe(
  mergeMap(value => of(value + '!').pipe(delay(1000)))
);

result$.subscribe(x => console.log(x));
// Output: (after 1s) Hello!, (after 1s) World!
```

### 4. concatMap
Projects each source value to an Observable and concatenates them, waiting for each inner Observable to complete before subscribing to the next.

```typescript
import { of } from 'rxjs';
import { concatMap, delay } from 'rxjs/operators';

const source$ = of(1000, 500, 2000);

// Each inner observable takes the value as delay time
const result$ = source$.pipe(
  concatMap(value => of(`Delayed by: ${value}ms`).pipe(delay(value)))
);

result$.subscribe(x => console.log(x));
// Output: (after 1s) Delayed by: 1000ms
//         (after 0.5s more) Delayed by: 500ms
//         (after 2s more) Delayed by: 2000ms
```

## Types and Subjects

### 1. Observable
A stream of values over time that can be subscribed to.

```typescript
import { Observable } from 'rxjs';

const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});

console.log('Before subscribe');
observable.subscribe({
  next(x) { console.log('Got value ' + x); },
  error(err) { console.error('Error: ' + err); },
  complete() { console.log('Done'); }
});
console.log('After subscribe');

/* Output:
Before subscribe
Got value 1
Got value 2
Got value 3
After subscribe
Got value 4
Done
*/
```

### 2. Subject
A special type of Observable that allows values to be multicasted to many Observers.

```typescript
import { Subject } from 'rxjs';

const subject = new Subject<number>();

subject.subscribe({
  next: (v) => console.log(`Observer A: ${v}`)
});

subject.subscribe({
  next: (v) => console.log(`Observer B: ${v}`)
});

subject.next(1);
subject.next(2);

// Output:
// Observer A: 1
// Observer B: 1
// Observer A: 2
// Observer B: 2
```

### 3. BehaviorSubject
A variant of Subject that requires an initial value and emits its current value to new subscribers.

```typescript
import { BehaviorSubject } from 'rxjs';

const subject = new BehaviorSubject(0); // 0 is the initial value

subject.subscribe({
  next: (v) => console.log(`Observer A: ${v}`)
});

subject.next(1);
subject.next(2);

subject.subscribe({
  next: (v) => console.log(`Observer B: ${v}`)
});

subject.next(3);

/* Output:
Observer A: 0 (initial value)
Observer A: 1
Observer A: 2
Observer B: 2 (current value when subscribed)
Observer A: 3
Observer B: 3
*/
```

### 4. Subscription
Represents a disposable resource, usually the execution of an Observable.

```typescript
import { interval } from 'rxjs';

const observable = interval(1000);
const subscription = observable.subscribe(x => console.log(x));

// Later:
// This cancels the ongoing Observable execution which
// was started by calling subscribe with an Observer.
setTimeout(() => {
  subscription.unsubscribe();
}, 5000);
// Output: 0, 1, 2, 3, 4 (then stops)
```

## Conversion Functions

### 1. firstValueFrom
Converts an observable to a promise that resolves with the first emitted value.

```typescript
import { interval, firstValueFrom } from 'rxjs';
import { take } from 'rxjs/operators';

async function execute() {
  const source$ = interval(1000).pipe(take(5));
  const firstValue = await firstValueFrom(source$);
  console.log(`First value: ${firstValue}`);
}

execute();
// Output after 1s: First value: 0
```

### 2. lastValueFrom
Converts an observable to a promise that resolves with the last emitted value when the observable completes.

```typescript
import { interval, lastValueFrom } from 'rxjs';
import { take } from 'rxjs/operators';

async function execute() {
  const source$ = interval(1000).pipe(take(5));
  const lastValue = await lastValueFrom(source$);
  console.log(`Last value: ${lastValue}`);
}

execute();
// Output after 5s: Last value: 4
```

# More RxJS Operators with Examples

Let's cover the additional operators you've requested, organized by category.

## Creation Operators

### 1. `of`
Creates an Observable that emits the arguments you provide, then completes.

```typescript
import { of } from 'rxjs';

const numbers$ = of(1, 2, 3);
numbers$.subscribe({
  next: val => console.log(val),
  complete: () => console.log('Complete!')
});

// Output:
// 1
// 2
// 3
// Complete!
```

### 2. `from`
Creates an Observable from an array, promise, iterable, or other observable-like object.

```typescript
import { from } from 'rxjs';

// From array
const arraySource$ = from([1, 2, 3]);
arraySource$.subscribe(val => console.log(val));

// From promise
const promiseSource$ = from(new Promise(resolve => resolve('Hello!')));
promiseSource$.subscribe(val => console.log(val));

// From string
const stringSource$ = from('Hello');
stringSource$.subscribe(val => console.log(val));
```

### 3. `throwError`
Creates an Observable that emits no items and immediately throws an error.

```typescript
import { throwError } from 'rxjs';

const error$ = throwError(() => new Error('Something went wrong!'));

error$.subscribe({
  next: val => console.log(val),
  error: err => console.error('Error:', err.message)
});

// Output:
// Error: Something went wrong!
```

### 4. `EMPTY`
An Observable that emits no items and immediately completes.

```typescript
import { EMPTY } from 'rxjs';

EMPTY.subscribe({
  next: val => console.log(val),
  complete: () => console.log('Complete!')
});

// Output:
// Complete!
```

## Filtering Operators

### 1. `filter`
Filters items emitted by the source Observable.

```typescript
import { from } from 'rxjs';
import { filter } from 'rxjs/operators';

const source$ = from([1, 2, 3, 4, 5]);
const evenNumbers$ = source$.pipe(
  filter(num => num % 2 === 0)
);

evenNumbers$.subscribe(val => console.log(val));
// Output: 2, 4
```

### 2. `debounceTime`
Emits a value from the source Observable only after a particular time span has passed without another source emission.

```typescript
import { fromEvent } from 'rxjs';
import { debounceTime } from 'rxjs/operators';

const searchBox = document.getElementById('search');
const input$ = fromEvent(searchBox, 'input');

input$.pipe(
  debounceTime(300)
).subscribe(event => {
  console.log('Search for:', event.target.value);
});
// Only emits when 300ms have passed since last input
```

### 3. `distinctUntilChanged`
Only emits when the current value is different from the last.

```typescript
import { of } from 'rxjs';
import { distinctUntilChanged } from 'rxjs/operators';

const source$ = of(1, 1, 2, 2, 3, 3, 3, 4, 4, 5);
const distinct$ = source$.pipe(
  distinctUntilChanged()
);

distinct$.subscribe(val => console.log(val));
// Output: 1, 2, 3, 4, 5
```

### 4. `take`
Emits only the first N values from the source.

```typescript
import { interval } from 'rxjs';
import { take } from 'rxjs/operators';

const numbers$ = interval(1000);
const firstFive$ = numbers$.pipe(take(5));

firstFive$.subscribe(val => console.log(val));
// Output: 0, 1, 2, 3, 4 (one per second)
```

### 5. `defaultIfEmpty`
Emits a default value if the source Observable completes without emitting any values.

```typescript
import { EMPTY } from 'rxjs';
import { defaultIfEmpty } from 'rxjs/operators';

EMPTY.pipe(
  defaultIfEmpty('No values emitted')
).subscribe(val => console.log(val));
// Output: No values emitted
```

## Error Handling Operators

### 1. `catchError`
Catches errors on the observable to be handled by returning a new observable or throwing an error.

```typescript
import { throwError, of } from 'rxjs';
import { catchError } from 'rxjs/operators';

const failingHttp$ = throwError(() => new Error('Network error'));

const result$ = failingHttp$.pipe(
  catchError(err => {
    console.error('Caught error:', err.message);
    return of('Fallback value');
  })
);

result$.subscribe(val => console.log(val));
// Output:
// Caught error: Network error
// Fallback value
```

### 2. `finalize`
Calls a function when observable completes or errors.

```typescript
import { of } from 'rxjs';
import { finalize } from 'rxjs/operators';

const source$ = of(1, 2, 3);

source$.pipe(
  finalize(() => console.log('Sequence complete'))
).subscribe(val => console.log(val));

// Output:
// 1
// 2
// 3
// Sequence complete
```

## Utility Operators

### 1. `tap`
Perform side effects for each emission on the source Observable, but return an Observable identical to the source.

```typescript
import { of } from 'rxjs';
import { tap } from 'rxjs/operators';

const source$ = of(1, 2, 3);

source$.pipe(
  tap(val => console.log(`Before map: ${val}`)),
  map(val => val * 10),
  tap(val => console.log(`After map: ${val}`))
).subscribe();

// Output:
// Before map: 1
// After map: 10
// Before map: 2
// After map: 20
// Before map: 3
// After map: 30
```

### 2. `forkJoin`
Accepts an Array or Object of Observables and waits for all of them to complete, then emits an array or object with the last values from each.

```typescript
import { forkJoin, of, timer } from 'rxjs';

const observable$ = forkJoin({
  foo: of(1, 2, 3, 4),
  bar: Promise.resolve(8),
  baz: timer(1000)
});

observable$.subscribe({
  next: value => console.log(value),
  complete: () => console.log('Complete!')
});

// After 1 second:
// Output: { foo: 4, bar: 8, baz: 0 }
// Complete!
```



