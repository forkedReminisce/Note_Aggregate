# Modifying Bits
There are a couple ways to modify the underlying bits other than simply changing the value to represent. However, it is probably best to discuss byte ordering first; this is how the bits are arranged across multiple memory locations. **Little-endian** has the least-significant byte stored in the lowest memory address; **Big-endian** puts the most-significant byte there.

## Boolean Algebra
You can use the operations `AND`, `OR`, and `XOR` on two bit vectors to create a new bit vector with the desired qualities. If one of the bit vectors are shorter, it is expanded to match the length of the other. Given the operation, the following is done one position at a time.
- `AND` places a `1` if the two bits are identical, and `0` otherwise
- `OR` places a `1` if at least one bit has a `1`, and `0` if neither do
- `XOR` places a `1` if **only** one bit has a `1`, and `0` otherwise
- `NOT` only takes one bit vector. It switches the digit of each bit.

## Shifts
Shifts shift all the bits of a bit vector a direction k times and places a particular digit in all the now-freed spaces.

A **logical left shift** shifts to the left and places `0`s in the freed spots; a logical right shift functions the same but shifts to the right. An **Arithmetic right shift** is particularly useful for signed values as the holes are now filled with whatever the most significant bit *was*.

%%rotation is when you take the bits that fell off and put it in the freed spots%%

## Integer Arithmetic
**Unsigned addition** `x ⊕(n, u) y` starts at `x` on the number wheel and moves `y` times clockwise. If it goes past the break, there is an unsigned carry. The sum is **(x + y) mod 2^n**. The **additive inverse** of a number `⊖(n, u)` is whatever generates a sum of 0 via unsigned carry; 0 has a special case in that its additive inverse is itself. **Unsigned subtraction** `x ⊖(n, u) y` is addition between `x` and the additive inverse of `y`. 

**Signed addition** `x ⊕(n, s) y` functions like unsigned addition, except that if `y` is negative, move counter-clockwise. A signed overflow is the unsigned carry equivalent; if the sum is too big, keep subtracting 2^n until within the range; too small and keep adding 2^n. The **additive inverse** `⊖(n, s)` is simply the negation; 0 and -2^n-1 are themselves. Signed subtraction has the same definition as unsigned subtraction.

[signed overflow is if x and y are positive and the result is negative or vise-versa]: #
-# Another way of calculating the additive inverse is to `NOT` the bit vector and add 1 to it.

**Multiplication** `⊙` is iterated addition; always start at 0 on the number wheel %% what %%. The product for unsigned is **xy mod 2^n**; U2Sc that for signed. Strength reduction is breaking a factor into powers of 2 then adding the left shifts.

-# The number of left shifts is the power of 2 you wish to multiply the value by. 

Right shifts can be used when dividing by powers of 2. However, right shifts alone will produce the floor ⎣x / 2^k⎦ of the quotient. To get the ceiling ⎡x / 2^k⎤ of a signed quotient, do `x + (1 << k) - 1` before the arithmetic right shift. This is particularly useful for negative quotients.

%% floors and ceiling use biased (as seen in signed representation) method. do i need the formulas for floor and ceiling? %%

## Recursive Doubling
Despite the name, recursion is usually not employed in this technique. Regardless, it is a divide and conquer strategy that does work in parallel by using shifts and sometimes masks. For each division, the shift and mask is changed by a factor of 2. The work is combined with an associative operator (e.g., addition). If a mask is used, it only is present on one side.

Structured permutations is like recursive doubling, except shifts are on both sides, in opposite directions. 