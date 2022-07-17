---
layout: post
title: HP-16C Exponential Program
---
The 16C (or DM16L) is a computer scientist model and not a general scientific
calculator, it does not include an exponent function, either arbitrary
y<sup>x</sup> or x². The correct thing to do here is obviously use a suitable
device, but for fun I've implemented an exponential program on this model. This
post documents some of that journey.

The simple way
--------------
The really easy and naive way to do this is to repeatedly multiply, the way we
all know and understand as a basic principle. This is actually a pretty easy
program to write and understand, for *a* to the power of *b* we store *b-1* in
`I` then use **DSZ** (Decrement `I` skip next instruction if `I` is zero) to
multiply *a* by itself the required amount of times.

This is clearly – slow as shit, becoming even more obvious running it on an
80's pocket calculator (even if a fancy one).

##### Implementation:
    LBL E // y^x x>=2
    1
    -
    STO I
    Rv
    ENT
    LBL 0
    *
    LST X
    DSZ
    GTO 0
    x<>y
    RTN

Exponentiation by squaring
--------------------------
There is a *O*(log n) solution to exponent calculation, which I will not go
into as [wikipedia][1] explains it well. From here on it will be expected
you're familiar with the tail-recursive implementation of the algorithm.

This program will only work while the calculator is in integer mode, it will
not work in floating point mode.

We can boil the algorithm down to something like this:
 1. y = 1
 2. n = n/2
 3. n was odd? (check the carry bit after a shift-right)
   * y = x * y
 4. x = x * x
 5. (n > 1) ?
   * GOTO 2
 6. x = x * y

##### Implementation:
    LBL E // Main Exponent Program y^x integer only x>=1
    F? 0  // {
    GTO 0 //   If flag 0 is set jump to loop otherwise set flag for later entry
    SF 0  // }

    1     // {
    STO I //   Store 1 in `I` (y)
    Rv    // }

    LBL 0 // Start of Label 0 (Main loop)
    1     // { If n = 1 go to the end section (2)
    x=y?  //
    GTO 2 //
    Rv    // }

    SR    // { n/2 by shifting right, run odd subroutine if carry flag set
    F? 4  //
    GSB 1 // }

    x<>y  // { x = x * x; then loop
    ENTER //
    *     //
    x<>y  //
    GTO E // }

    LBL 1 // { n is odd subroutine
    x<>I  //   Get the y value from register `I`
    x<>y  //   y = x * y; store it, restore stack and return
    *     //
    LSTx  //
    x<>y  //
    x<>I  //
    RTN   // }

    LBL 2 // { Subroutine for ending
    CF 0  //   Clear flag 0
    Rv    //   x = x * y; then return
    x<>I  //
    *     //
    RTN   // }

[1]: https://en.wikipedia.org/wiki/Exponentiation_by_squaring
