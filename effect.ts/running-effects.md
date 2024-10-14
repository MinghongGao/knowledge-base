# Running Effects



This page records how to run an effect.

### Common

#### Effect.runSync

```typescript
import { Effect } from "effect"
 
const program = Effect.sync(() => {
  console.log("Hello, World!")
  return 1
})
 
const result = Effect.runSync(program)
// Output: Hello, World!
 
console.log(result)
// Output: 1

```

```typescript
import { Effect } from "effect"
 
Effect.runSync(Effect.fail("my error")) // throws
 
Effect.runSync(Effect.promise(() => Promise.resolve(1))) // throws
```

* Immedicately run the effect synchronously
* Result is the success payload
* Throws error when failure or the task is async.

#### Effect.runSyncExit

```typescript
import { Effect } from "effect"
 
const result1 = Effect.runSyncExit(Effect.succeed(1))
console.log(result1)
/*
Output:
{
  _id: "Exit",
  _tag: "Success",
  value: 1
}
*/
 
const result2 = Effect.runSyncExit(Effect.fail("my error"))
console.log(result2)
/*
Output:
{
  _id: "Exit",
  _tag: "Failure",
  cause: {
    _id: "Cause",
    _tag: "Fail",
    failure: "my error"
  }
}
*/
```

* Immedicately run the effect synchronously
* Result is the Exit payload

#### Effect.runPromise

```typescript
import { Effect } from "effect"
 
Effect.runPromise(Effect.succeed(1)).then(console.log) // Output: 1
```

```typescript
import { Effect } from "effect"
 
Effect.runPromise(Effect.fail("my error")) // rejects
```

* Execute an effect and obtain the result as a `Promise`

#### Effect.runPromiseExit

```typescript
import { Effect } from "effect"
 
Effect.runPromiseExit(Effect.succeed(1)).then(console.log)
/*
Output:
{
  _id: "Exit",
  _tag: "Success",
  value: 1
}
*/
 
Effect.runPromiseExit(Effect.fail("my error")).then(console.log)
/*
Output:
{
  _id: "Exit",
  _tag: "Failure",
  cause: {
    _id: "Cause",
    _tag: "Fail",
    failure: "my error"
  }
}
*/
```

#### Effect.runFork

```typescript
import { Effect, Console, Schedule, Fiber } from "effect"
 
const program = Effect.repeat(
  Console.log("running..."),
  Schedule.spaced("200 millis")
)
 
const fiber = Effect.runFork(program)
 
setTimeout(() => {
  Effect.runFork(Fiber.interrupt(fiber))
}, 500)
```

### Cheatsheet

| Name             | Given          | To                    |
| ---------------- | -------------- | --------------------- |
| `runSync`        | `Effect<A, E>` | `A`                   |
| `runSyncExit`    | `Effect<A, E>` | `Exit<A, E>`          |
| `runPromise`     | `Effect<A, E>` | `Promise<A>`          |
| `runPromiseExit` | `Effect<A, E>` | `Promise<Exit<A, E>>` |
| `runFork`        | `Effect<A, E>` | `RuntimeFiber<A, E>`  |
