---
layout: page
title: RPN Basics
---
There is no `=` button, but on a normal calculator what is the `=` button
anyway? Take the algebraic expression entered, assume it is the LHS and display
a result value for RHS? Honestly if we think about it, it is a strange
interface and behaviour. It only makes sense because it's what we know and it
translates directly from our written form to the machine input.

When we usually interact with a machine the flow is generally something like:
 1. Select operation (e.g. division)
 2. Enter arguments (e.g. dividend & divisor)
 3. Get result

With a standard calculator this looks something like:
 1. Enter 1st argument (e.g. dividend)
 2. Select operation (e.g. divide)
 3. Enter 2nd argument (e.g. divisor)
 4. Press equals/execute to get result

How did that become a thing?

In a RPN postfix system it looks like this:
 1. Enter 1st argument (e.g. dividend)
 2. Hit ENTER to push the value onto the stack
 3. Enter 2nd argument (e.g. divisor)
 4. Select operation (e.g. divide)
    * Two values are popped from the stack
    * Function called supplying popped values as arguments
    * Result is push back onto the top of the stack

This interface actually makes more sense as a human interface. Values first
then the operation we want to perform.

The Stack
---------
The standard 4 value stack in a RPN calculator conceptually looks like this:

| ID  | Description
|----:| ---
|   t | Bottom of stack, value is duplicated to *z* when stack is lifted
|   z |
|   y | Upper displayed value on models with two rows
|   x | Displayed value

Assuming a stack pre-populated with 9, 7, 5, 3 and we then run a calculation
for *128 / 2* the steps look something like this:

| ID | Start | 128 | ENTER | 2   | /  |
|---:|-------|-----|-------|-----|----|
|  t | 9     | 7   | 5     | 5   | 5  |
|  z | 7     | 5   | 3     | 3   | 5  |
|  y | 5     | 3   | 128   | 128 | 3  |
|  x | 3     | 128 | 128   | 2   | 64 |

When `ENTER` is pressed the stack is pushed and the previous value stays in the
`x` register and remains the displayed value, however the next number input
will replace what is in that register (*2* in this case).


Taking advantage of stack lift
------------------------------
Say you need to repeatedly run the same calculation but want the intermediary
values, for example compound interest. So interest of 1.25% on 130.

| ID | 1.0125 | ENTER  | ENTER  | ENTER  | 130    | *      | *      | *      |
|---:|--------|--------|--------|--------|--------|--------|--------|--------|
| t  | 9      | 7      | 5      | 1.0125 | 1.0125 | 1.0125 | 1.0125 | 1.0125 |
| z  | 7      | 5      | 1.0125 | 1.0125 | 1.0125 | 1.0125 | 1.0125 | 1.0125 |
| y  | 5      | 1.0125 | 1.0125 | 1.0125 | 1.0125 | 1.0125 | 1.0125 | 1.0125 |
| x  | 1.0125 | 1.0125 | 1.0125 | 1.0125 | 130    | 131.62 | 133.27 | 134.93 |

Every time we run the multiply operation the stack lifts duplicating `t` into
`z`, so we can repeatedly hit multiply to get `x = x * 1.0125`.


Values first
------------
The major advantage to RPN over traditional calculators is making use of the
stack and having a values first approach. Here's a real world example:

 * I have just filled up with 22.6L of petrol and my trip clock from the last
   fill reads 117 miles.

I want to compute that into either MPG or L/100km, normally I'd want to write
those down and then figure out the math to do this (algebraic thinking).
Instead let's just mash the values in and get to L/100km somehow.

| ID | Vals | x<>y | 0.62 | /      | /    | 100  | *     |
|---:|------|------|------|--------|------|------|-------|
|t   |      |      |      |        |      |      |       |
|z   |      |      | 22.6 |        |      |      |       |
|y   | 117  | 22.6 | 117  | 22.6   |      | 0.12 |       |
|x   | 22.6 | 117  | 0.62 | 188.71 | 0.12 | 100  | 11.98 |

 1. Enter the values
 2. OK seeing litres value, swap X and Y to get the miles value
 3. I know `0.62` is conversion enter it
 4. Divide and we now have km
 5. Divide again and it's L/km
 6. Multiply by 100 and have L/100km

Note it is a visual operation each time, the data is entered and visible, then
we decide what to do with it. Taking advantage of the stack as memory it's easy
to play around with and juggle units of one value while keeping the other
stored, ready for use in the next step.

Let's do the same again but MPG

| ID | Vals | 4.546 | /    | /     |
|---:|------|-------|------|-------|
|t   |      |       |      |       |
|z   |      | 117   |      |       |
|y   | 117  | 22.6  | 117  |       |
|x   | 22.6 | 4.546 | 4.97 | 23.53 |

Division & Subtraction
----------------------
It would first appear that the operations you need to think about, at least in
terms of the order of arguments is division and subtraction, however we can
actually safely ignore both once we have an idea what the expected value should
be.

In terms of subtraction if it should be reversed use `CHS` or `+/-` to
negate the value in the `x` register.

If the dividend and divisor were mixed up simply hit the inverse button `1/x`
to flip the value. Again this makes it easier to purely think in terms of
values and operations you want to perform on them. If you know a divide
operation for the two values on the top of your stack is required you likely
know if you are expecting fractional value less than 1 or not.

Stack operations
----------------
 * `x<>y` swaps the `x` and `y` registers
 * `Rv` rolls the entire stack down
   * `x` <- `y`
   * `y` <- `z`
   * `z` <- `t`
   * `t` <- `x`
 * `LSTx` push the last `x` value from before an operation back onto the stack
   * e.g. `27`, `ENTER`, `3`, `/`, `LSTx`; results in *9* in the `y` register
     and *3* in the `x` register.

### Insert values
#### Into `y`
 * Enter value followed by `x<>y`

#### Into `z`
 * Can lose current `z` and `t`, new val fills `z` and `t`:
   * Enter value; `ENTER`, `Rv`, `Rv`

| ID | Vals | 2 |ENT|Rv |Rv |
|---:|------|---|---|---|---|
|t   | 9    | 8 | 7 | 2 | 2 |
|z   | 8    | 7 | 6 | 7 | 2 |
|y   | 7    | 6 | 2 | 6 | 7 |
|x   | 6    | 2 | 2 | 2 | 6 |

 * Push current `z` into `t` and insert new `z`:
   * Enter value; `x<>y`, `Rv`, `x<>y`, `Rv`, `Rv`, `Rv`

| ID | Vals | 2 |x<>y|Rv |x<>y|Rv |Rv |Rv |
|---:|------|---|----|---|----|---|---|---|
|t   | 9    | 8 |  8 | 6 |  6 | 7 | 2 | 8 |
|z   | 8    | 7 |  7 | 8 |  8 | 6 | 7 | 2 |
|y   | 7    | 6 |  2 | 7 |  2 | 8 | 6 | 7 |
|x   | 6    | 2 |  6 | 2 |  7 | 2 | 8 | 6 |
