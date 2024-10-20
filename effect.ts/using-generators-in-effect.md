# Using Generators in Effect

## Using Effect.gen

```typescript
import { Effect } from "effect"
 
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
 
// Assembling the program using a generator function
const program = Effect.gen(function* () {
  // Retrieve the transaction amount
  const transactionAmount = yield* fetchTransactionAmount
 
  // Retrieve the discount rate
  const discountRate = yield* fetchDiscountRate
 
  // Calculate discounted amount
  const discountedAmount = yield* applyDiscount(
    transactionAmount,
    discountRate
  )
 
  // Apply service charge
  const finalAmount = addServiceCharge(discountedAmount)
 
  // Return the total amount after applying the charge
  return `Final amount to charge: ${finalAmount}`
})
 
// Execute the program and log the result
Effect.runPromise(program).then(console.log) // Output: "Final amount to charge: 96"
```

## Comparing Effect.gen with async/await

{% tabs %}
{% tab title="Using Effect.gen" %}
```typescript
const addServiceCharge = (amount: number) => amount + 1
 
const applyDiscount = (
  total: number,
  discountRate: number
): Effect.Effect<number, Error> =>
  discountRate === 0
    ? Effect.fail(new Error("Discount rate cannot be zero"))
    : Effect.succeed(total - (total * discountRate) / 100)
 
const fetchTransactionAmount = Effect.promise(() => Promise.resolve(100))
 
const fetchDiscountRate = Effect.promise(() => Promise.resolve(5))
 
export const program = Effect.gen(function* () {
  const transactionAmount = yield* fetchTransactionAmount
  const discountRate = yield* fetchDiscountRate
  const discountedAmount = yield* applyDiscount(
    transactionAmount,
    discountRate
  )
  const finalAmount = addServiceCharge(discountedAmount)
  return `Final amount to charge: ${finalAmount}`
})
```
{% endtab %}

{% tab title="Using async/await" %}
```typescript
const addServiceCharge = (amount: number) => amount + 1
 
const applyDiscount = (
  total: number,
  discountRate: number
): Promise<number> =>
  discountRate === 0
    ? Promise.reject(new Error("Discount rate cannot be zero"))
    : Promise.resolve(total - (total * discountRate) / 100)
 
const fetchTransactionAmount = Promise.resolve(100)
 
const fetchDiscountRate = Promise.resolve(5)
 
export const program = async function () {
  const transactionAmount = await fetchTransactionAmount
  const discountRate = await fetchDiscountRate
  const discountedAmount = await applyDiscount(
    transactionAmount,
    discountRate
  )
  const finalAmount = addServiceCharge(discountedAmount)
  return `Final amount to charge: ${finalAmount}`
}U
```
{% endtab %}
{% endtabs %}

## Error handling

```typescript
import { Effect } from "effect"
 
const program = Effect.gen(function* () {
  console.log("Task1...")
  console.log("Task2...")
  yield* Effect.fail("Something went wrong!")
  console.log("This won't be executed")
})
 
Effect.runPromise(program).then(console.log, console.error)
/*
Output:
Task1...
Task2...
(FiberFailure) Error: Something went wrong!
*/
```





