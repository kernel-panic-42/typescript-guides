# TypeScript `Array.reduce()` --- Deep Dive

## Learning Objective

Understand how to use `Array.reduce()` in TypeScript to:\
1. Fold arrays into single values (sums, objects, etc.).\
2. Build function pipelines where values flow through a sequence of
transformations.\
3. Apply both synchronous and asynchronous pipelines in real-world
scenarios.

------------------------------------------------------------------------

## What `reduce()` Does

`reduce()` takes an array and reduces it to a single value by applying a
reducer function to each element.

------------------------------------------------------------------------

## Function Signature

``` ts
array.reduce<T, U>(
  callback: (accumulator: U, currentValue: T, currentIndex: number, array: T[]) => U,
  initialValue: U
): U
```

-   **callback** runs for each element, gets
    `(accumulator, currentValue, currentIndex, array)`\
-   **initialValue** sets the starting accumulator (recommended to
    always provide it).\
-   Returns a single accumulated value.

------------------------------------------------------------------------

## Step-by-Step Trace Example

``` ts
const nums = [1, 2, 3];
const sum = nums.reduce((acc, n, i) => {
  console.log(`i=${i}, acc=${acc}, n=${n} -> next=${acc + n}`);
  return acc + n;
}, 0);
// Logs:
// i=0, acc=0, n=1 -> next=1
// i=1, acc=1, n=2 -> next=3
// i=2, acc=3, n=3 -> next=6
// sum = 6
```

------------------------------------------------------------------------

## Using `reduce()` as a Pipeline

Instead of numbers, imagine an **array of functions**
`(in: string) => string`. We can pipe a string through them in order.

### Transformer Type

``` ts
type StringTransformer = (input: string) => string;
```

### Run Pipeline with Trace

``` ts
function runPipeline(input: string, fns: StringTransformer[]): string {
  return fns.reduce((acc, fn, i) => {
    const next = fn(acc);
    console.log(`step=${i}, before="${acc}", after="${next}"`);
    return next;
  }, input);
}
```

### Example Transformers

``` ts
const trim: StringTransformer           = s => s.trim();
const toUpper: StringTransformer        = s => s.toUpperCase();
const collapseSpaces: StringTransformer = s => s.replace(/\s+/g, " ");
const addBang: StringTransformer        = s => s + "!";
```

### Running the Pipeline

``` ts
const transformers: StringTransformer[] = [
  trim,
  collapseSpaces,
  toUpper,
  addBang,
];

const result = runPipeline("   hello     world   ", transformers);
// Logs:
// step=0, before="   hello     world   ", after="hello     world"
// step=1, before="hello     world",     after="hello world"
// step=2, before="hello world",          after="HELLO WORLD"
// step=3, before="HELLO WORLD",          after="HELLO WORLD!"
// result = "HELLO WORLD!"
```

------------------------------------------------------------------------

## Reusable Helpers

### Left-to-right `pipe`

``` ts
export const pipe = <T>(...fns: Array<(x: T) => T>) =>
  (input: T): T => fns.reduce((acc, fn) => fn(acc), input);

// Usage:
const shout = pipe(trim, collapseSpaces, toUpper, addBang);
const out = shout("   hello     world   "); // "HELLO WORLD!"
```

### Right-to-left `compose`

``` ts
export const compose = <T>(...fns: Array<(x: T) => T>) =>
  (input: T): T => fns.reduceRight((acc, fn) => fn(acc), input);
```

### Async Pipeline

``` ts
type AsyncTransform<T> = (x: T) => Promise<T> | T;

export const pipeAsync = <T>(...fns: AsyncTransform<T>[]) =>
  async (input: T): Promise<T> =>
    fns.reduce<Promise<T>>(async (pAcc, fn, i) => {
      const acc = await pAcc;
      const next = await fn(acc);
      console.log(`step=${i}, before=`, acc, `after=`, next);
      return next;
    }, Promise.resolve(input));
```

------------------------------------------------------------------------

## Key Takeaways

-   `reduce()` folds an array into a single value.\
-   With functions, it becomes a **pipeline builder**.\
-   Prefer `pipe` for left-to-right readability, `compose` for
    right-to-left.\
-   Extendable to async workflows.
