# Representations
**Bit** is a portmanteau of ==binary== and ==digit==. It has two discrete and distinguishable states: ○ and │. It may initially seem that there is no way to get numbers or characters, but it is possible with an external encoding scheme. 

## Bit Vectors
Say the bit is a set {○, │}. By performing a chain of Cartesian products with itself, we can create bit tuples. A 4-bit tuple is known as a *nibble*, whereas an 8-bit tuple is called a *byte*. Additionally, we may come up with a set of symbols to shorthand each permutation (e.g., hexadecimal's A is \[│, ○, │, ○\]).

-# We tend to ditch set notation and translate ○ to 0 and │ to 1.

## Positional Codes
This is how bit vectors become values. Each bit has a weight $2^i$, where $i$ is position starting from the right—the least significant bit. Multiply the bits by their respective weight then sum it all. The base may change (e.g., $16^i$ in hexadecimal).

## Integers
Let $n$ be the number of bits in the bit string, and $i$ start at 0. Unsigned integers have a value set $U = [0, 2^n - 1]$. Signed integers' value set $S = [-2^{n - 1}, 2^{n - 1} - 1]$. There are three ways of encoding signed integers:
- Two's complement: Make the highest weight $-2^i$
- Biased: Subtract $2^{n - 1}$ from the unsigned value of the same bit vector
- Sign bit: Dedicate the most significant bit to indicate positive `0` or negative `1`

The expansion of an integer bit vector uses the sign extension technique. If it is unsigned, simply place `0`s. Signed copies the MSB. Truncation simply cuts off any bits past $k$ bits. This new number is just modulo $2^k$ for unsigned, and is simply translated to signed if need be.

## Floating Point
There are two ways of representing floating point. Both have a starting sign bit.
- Fixed point: a bits serve as the integer, followed by b bits that go into negative powers of 2.
- Floating point: representing ±S × 2^E. This is the common method.

The q exponent bits come after the sign bit. The value is B2U(E) - (2^(q - 1) - 1). Since E reserves bit strings (0^q) and (1^q) for special floating point values, its value set in **normalized floating point** is \[-(bias - 1), bias\]. 

The p significand bits follow in the format (b~0~ · b~1~ ···). The positional code is 2^-i. b~0~ is actually the hidden bit; it is implicitly in the bit string as a 1 in normalized floating point. Put together, the value set of normalized floating point is \[1 × 2^(1 - bias), 2^(bias + 1)).

p is precision. Machine epsilon $\epsilon$ is the gap between 1 and (1 · 0^(p - 2) 1). The value is 2^(-(p - 1)). Unit in the last place $\mathnormal{ulp}$ is the gap between any number and the next normalized floating point number. It is defined as $ε \times 2^E$.

-# The maximum normalized floating point number technically is (1 · 1^(p - 1)) × 2^bias. Realistically, this is (2 - 2^(-(p - 1))) × 2^bias.

There are other types of numbers that can be represented using the reserved exponent bit strings:
- **Subnormal numbers** are values 1 - ε %% is this right %% to 0. E must have the bit string (0^q). This sets the exponent to -(bias - 1) and the hidden bit to 0 (gradual underflow).
- **Infinities** and **Not a Number** are achieved with an E bit string (1^q). An S bit string of (0^(p - 1)) gets infinity; anything else gets NaN.

-# NaNs propagate through operations.

$x_-$ is analogous to floor. It is obtained by truncating the significand bits until it fits. $x_+$ is like ceiling. It is obtained by truncating then adding (0 · 0^(p - 2) 1) to the significand %% special number types use special rules that may not fit these algorithms; intrinsically, they're sensical.  %%. If performing operations on floating points, rounding will be applied to the result after each operation.

-# The default rounding mode is round-to-nearest, with tiebreak to even. The tiebreak is whichever x~-~ or x~+~ has a significand lsb 0. %% even; odd is lsb 1 %%

## Chars
An 8-bit, signed or unsigned, quantity. The only difference between signed and unsigned is the type. Originally, there were only 128 encodings via ASCII. Then Unicode blew that number out of the water. 

UTF-8 is a variable-length encoding that goes up to four bytes. If the first byte starts with `0`, the next 7 bits are direct from ASCII. `110`, `1110`, and `11110` means two, three, and four bytes respectively. Each continuation byte starts with `10`.

%% getting the unicode code point by looking at the actual bits and converting this new bit string into hex %%