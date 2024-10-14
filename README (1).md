# Creating Effects

This pages records the Effect constructors

### Common

#### Effect.succeed

```typescript
import { Effect } from "effect"
 
const success = Effect.succeed(42)
```

`success` is an instance of `Effect<number, never, never>`

#### Effect.fail

```typescript
import { Effect } from "effect"
 
// Creating an effect that represents a failure scenario
const failure = Effect.fail(
  new Error("Operation failed due to network error")
)
```

`failure` is an instance of `Effect<never, Error, never>`

#### Effect.sync

```typescript
import { Effect } from "effect"
 
const log = (message: string) =>
  Effect.sync(() => {
    console.log(message) // side effect
  })
 
const program = log("Hello, World!")
```

The `program` has the type `Effect<void, never, never>`

#### Effect.try

```typescript
import { Effect } from "effect"
 
const parse = (input: string) =>
  Effect.try(
    () => JSON.parse(input) // This might throw an error if input is not valid JSON
  )
 
const program = parse("")
```

The `program` value has the type `Effect<any, UnknownException, never>`

or

```typescript
import { Effect } from "effect"
 
const parse = (input: string) =>
  Effect.try({
    try: () => JSON.parse(input), // JSON.parse may throw for bad input
    catch: (unknown) => new Error(`something went wrong ${unknown}`) // remap the error
  })
 
const program = parse("")
```

The `program` value has the type `Effect<any, Error, never>`

#### Effect.promise

```typescript
import { Effect } from "effect"
 
const delay = (message: string) =>
  Effect.promise<string>(
    () =>
      new Promise((resolve) => {
        setTimeout(() => {
          resolve(message)
        }, 2000)
      })
  )
 
const program = delay("Async operation completed successfully!")

```

The `program` value has the type `Effect<string, never, never>`

#### Effect.tryPromise

```typescript
import { Effect } from "effect"
 
const getTodo = (id: number) =>
  Effect.tryPromise(() =>
    fetch(`https://jsonplaceholder.typicode.com/todos/${id}`)
  )
 
const program = getTodo(1)
```

The `program` value has the type `Effect<Response, UnknownException, never>`

or

```typescript
import { Effect } from "effect"
 
const getTodo = (id: number) =>
  Effect.tryPromise({
    try: () => fetch(`https://jsonplaceholder.typicode.com/todos/${id}`),
    // remap the error
    catch: (unknown) => new Error(`something went wrong ${unknown}`)
  })
 
const program = getTodo(1)
```

The `program` value has the type `Effect<Response, Error, never>`

#### Effect.async

```typescript
import { Effect } from "effect"
import * as NodeFS from "node:fs"
 
const readFile = (filename: string) =>
  Effect.async<Buffer, Error>((resume) => {
    NodeFS.readFile(filename, (error, data) => {
      if (error) {
        resume(Effect.fail(error))
      } else {
        resume(Effect.succeed(data))
      }
    })
  })
 
const program = readFile("todos.txt")
```

#### Effect.suspend

```typescript
import { Effect } from "effect"
 
let i = 0
 
const bad = Effect.succeed(i++)
 
const good = Effect.suspend(() => Effect.succeed(i++))
 
console.log(Effect.runSync(bad)) // Output: 0
console.log(Effect.runSync(bad)) // Output: 0
 
console.log(Effect.runSync(good)) // Output: 1
console.log(Effect.runSync(good)) // Output: 2
```

### Cheatsheet

| Function                | Given                              | To                            |
| ----------------------- | ---------------------------------- | ----------------------------- |
| `succeed`               | `A`                                | `Effect<A>`                   |
| `fail`                  | `E`                                | `Effect<never, E>`            |
| `sync`                  | `() => A`                          | `Effect<A>`                   |
| `try`                   | `() => A`                          | `Effect<A, UnknownException>` |
| `try` (overload)        | `() => A`, `unknown => E`          | `Effect<A, E>`                |
| `promise`               | `() => Promise<A>`                 | `Effect<A>`                   |
| `tryPromise`            | `() => Promise<A>`                 | `Effect<A, UnknownException>` |
| `tryPromise` (overload) | `() => Promise<A>`, `unknown => E` | `Effect<A, E>`                |
| `async`                 | `(Effect<A, E> => void) => void`   | `Effect<A, E>`                |
| `suspend`               | `() => Effect<A, E, R>`            | `Effect<A, E, R>`             |
