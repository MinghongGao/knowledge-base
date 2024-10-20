# Building Pipelines

## pipe

```typescript
import { pipe } from "effect"
const result = pipe(input, func1, func2, ..., funcN)
```

* First argument is the input.
* For the functions that pass to `pipe`, it will use the previous function's output as the input
* The last function's output is the final output

## map

```typescript
import { pipe, Effect } from "effect"
const mappedEffect = pipe(myEffect, Effect.map(transformation))
// or
const mappedEffect = Effect.map(myEffect, transformation)
// or
const mappedEffect = myEffect.pipe(Effect.map(transformation))
```

* Transform the value inside an Effect, creating a new effect with the transformed value

### Example

```typescript
import { pipe, Effect } from "effect"
 
// Function to add a small service charge to a transaction amount
const addServiceCharge = (amount: number) => amount + 1
 
// Simulated asynchronous task to fetch a transaction amount from a database
const fetchTransactionAmount = Effect.promise(() => Promise.resolve(100))
 
// Apply service charge to the transaction amount
const finalAmount = pipe(fetchTransactionAmount, Effect.map(addServiceCharge))
 
Effect.runPromise(finalAmount).then(console.log) // Output: 101

```

## as

To map an `Effect` to a constant value, replacing the original value, use `Effect.as`:

```typescript
import { pipe, Effect } from "effect"
 
const program = pipe(Effect.succeed(5), Effect.as("new value"))
 
Effect.runPromise(program).then(console.log) // Output: "new value"
```

## flatMap

* the difference between `flatMap` and `map` is on the transformation function
  * For `flatMap`, the transformation function returns an effect,
  * For `map`, the transformation function return a value

### Example

```typescript
import { pipe, Effect } from "effect"
 
// Function to apply a discount safely to a transaction amount
const applyDiscount = (
  total: number,
  discountRate: number
): Effect.Effect<number, Error> =>
  discountRate === 0
    ? Effect.fail(new Error("Discount rate cannot be zero"))
    : Effect.succeed(total - (total * discountRate) / 100)
 
// Simulated asynchronous task to fetch a transaction amount from a database
const fetchTransactionAmount = Effect.promise(() => Promise.resolve(100))
 
const finalAmount = pipe(
  fetchTransactionAmount,
  Effect.flatMap((amount) => applyDiscount(amount, 5))
)
 
Effect.runPromise(finalAmount).then(console.log) // Output: 95
```

## andThen

* all In One function
  1. a value (i.e. same functionality of `Effect.as`)
  2. a function returning a value (i.e. same functionality of `Effect.map`)
  3. a `Promise`
  4. a function returning a `Promise`
  5. an `Effect`
  6. a function returning an `Effect`(i.e. same functionality of `Effect.flatMap`)

### Example

```typescript
import { pipe, Effect } from "effect"
// Function to apply a discount safely to a transaction amount
const applyDiscount = (
  total: number,
  discountRate: number
): Effect.Effect<number, Error> =>
  discountRate === 0
    ? Effect.fail(new Error("Discount rate cannot be zero"))
    : Effect.succeed(total - (total * discountRate) / 100)
// Simulated asynchronous task to fetch a transaction amount from a database
const fetchTransactionAmount = Effect.promise(() => Promise.resolve(100))
// Using Effect.map, Effect.flatMap
const result1 = pipe(
  fetchTransactionAmount,
  Effect.map((amount) => amount * 2),
  Effect.flatMap((amount) => applyDiscount(amount, 5))
)
Effect.runPromise(result1).then(console.log) // Output: 190
// Using Effect.andThen
const result2 = pipe(
  fetchTransactionAmount,
  Effect.andThen((amount) => amount * 2),
  Effect.andThen((amount) => applyDiscount(amount, 5))
)
Effect.runPromise(result2).then(console.log) // Output: 190
```

## tap

The `Effect.tap` API has a similar signature to `Effect.flatMap`, but the result of the transformation function is **ignored**.

```typescript
import { pipe, Effect } from "effect"
 
// Function to apply a discount safely to a transaction amount
const applyDiscount = (
  total: number,
  discountRate: number
): Effect.Effect<number, Error> =>
  discountRate === 0
    ? Effect.fail(new Error("Discount rate cannot be zero"))
    : Effect.succeed(total - (total * discountRate) / 100)
 
// Simulated asynchronous task to fetch a transaction amount from a database
const fetchTransactionAmount = Effect.promise(() => Promise.resolve(100))
 
const finalAmount = pipe(
  fetchTransactionAmount,
  Effect.tap((amount) =>
    Effect.sync(() => console.log(`Apply a discount to: ${amount}`))
  ),
  // `amount` is still available!
  Effect.flatMap((amount) => applyDiscount(amount, 5))
)
 
Effect.runPromise(finalAmount).then(console.log)
/*
Output:
Apply a discount to: 100
95
*/
```

## all

```typescript
import { Effect } from "effect"
const combinedEffect = Effect.all([effect1, effect2, ...])
```

### Example

```typescript
import { Effect } from "effect"
 
// Simulated function to read configuration from a file
const webConfig = Effect.promise(() =>
  Promise.resolve({ dbConnection: "localhost", port: 8080 })
)
 
// Simulated function to test database connectivity
const checkDatabaseConnectivity = Effect.promise(() =>
  Promise.resolve("Connected to Database")
)
 
// Combine both effects to perform startup checks
const startupChecks = Effect.all([webConfig, checkDatabaseConnectivity])
 
Effect.runPromise(startupChecks).then(([config, dbStatus]) => {
  console.log(
    `Configuration: ${JSON.stringify(config)}, DB Status: ${dbStatus}`
  )
})
/*
Output:
Configuration: {"dbConnection":"localhost","port":8080}, DB Status: Connected to Database
*/
```

## Full Example

```typescript
import { Effect, pipe } from "effect"
 
// Function to add a small service charge to a transaction amount
const addServiceCharge = (amount: number) => amount + 1
 
// Function to apply a discount safely to a transaction amount
const applyDiscount = (
  total: number,
  discountRate: number
): Effect.Effect<number, Error> =>
  discountRate === 0
    ? Effect.fail(new Error("Discount rate cannot be zero"))
    : Effect.succeed(total - (total * discountRate) / 100)
 
// Simulated asynchronous task to fetch a transaction amount from a database
const fetchTransactionAmount = Effect.promise(() => Promise.resolve(100))
 
// Simulated asynchronous task to fetch a discount rate from a configuration file
const fetchDiscountRate = Effect.promise(() => Promise.resolve(5))
 
// Assembling the program using a pipeline of effects
const program = pipe(
  Effect.all([fetchTransactionAmount, fetchDiscountRate]),
  Effect.flatMap(([transactionAmount, discountRate]) =>
    applyDiscount(transactionAmount, discountRate)
  ),
  Effect.map(addServiceCharge),
  Effect.map((finalAmount) => `Final amount to charge: ${finalAmount}`)
)
 
// Execute the program and log the result
Effect.runPromise(program).then(console.log) // Output: "Final amount to charge: 96
```

## Cheatsheet

| **Function** | **Input**                                 | **Output**                  |
| ------------ | ----------------------------------------- | --------------------------- |
| `map`        | `Effect<A, E, R>`, `A => B`               | `Effect<B, E, R>`           |
| `flatMap`    | `Effect<A, E, R>`, `A => Effect<B, E, R>` | `Effect<B, E, R>`           |
| `andThen`    | `Effect<A, E, R>`, \*                     | `Effect<B, E, R>`           |
| `tap`        | `Effect<A, E, R>`, `A => Effect<B, E, R>` | `Effect<A, E, R>`           |
| `all`        | `[Effect<A, E, R>, Effect<B, E, R>, ...]` | `Effect<[A, B, ...], E, R>` |

\
