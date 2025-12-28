---
draft: false
title: Representing Bits

params: 
    desc: It may be a wonder how bits can represent so much despite being a bunch of 0s and 1s. That thanks to encoding schemes.
    author: Andrew Nguyen 
---



A **bit** has two discrete and distinguishable states: ○ and │. For simplicity's sake, let's use `0` and `1` respectively. A 4-bit tuple is a nibble and an 8-bit tuple is a byte. With an external encoding scheme, it's possible to represent an \(n\)-bit tuple into something useful.



# {{< heading "Integers" >}}
For a quick method for encoding into an integer, use *positional codes*. Let \(b\) be the base of the number system (e.g., binary is \(2\) and hexadecimal is \(16\)). At position \(i\)—0-indexed, starting from the rightmost bit (the least significant bit)—the weight is \(b^i\). Then multiply the weight with the value at \(i\), and sum everything up.

If there are \(n\) bits in the bit string, the value set of unsigned integers is \(U = [0, 2^n - 1]\) and signed integers is \(S = [-2^{n - 1}, 2^{n - 1} - 1]\). There are three ways of encoding signed integers:
- **Two's complement**: Highest weight is a special \(-2^i\)
- Biased: Subtract \(2^{n - 1}\) from the unsigned representation
- Sign bit: Dedicate the MSB for positive (`0`) or negative (`1`)

Expansion is when the bit string grows. For unsigned integers, simply place `0`'s in the new positions. For signed, copy the MSB over (sign extension). Truncation is shrinking the bit string by literally discarding the extra bits. 



# {{< heading "Floating Point" >}}
There are two ways of representing floating point. Both have a starting sign bit.
- **Floating point**: \(\pm S \times 2^E\)
- Fixed point: \(a\) most significant bits are the integer, and the following \(b\) bits have a weight of \(2^-i\)

In floating point, after the sign bit are \(q\) *exponent* bits. Bit strings \((0^q)\) and \((1^q)\) are reserved, so **normalized floating point** only covers \([-(2^{q - 1} - 1), 2^{q - 1}]\).

\[
    \textrm{exponent value} = \operatorname{B2U}(E) - 2^{q - 1} - 1
\]

\(p\) *significand* bits come after. The weight is \(2^{-i}\), where indexing actually starts on the most significant side. The most significant bit (\(i = 0\)) is actually the hidden bit, which is implicitly a `1` in normalized floating point. All together, the value set of normalized floating point is \([2^{1 - 2^{q - 1}}, 2^{2^{q - 1} + 1})\).

\(p\) is precision. This concept leads to machine epsilon \(\epsilon\), the gap between 1 and \((1 \cdot 0^{p - 2} 1)\); it's \(2^{-(p - 1)}\). Unit in the last place \(\mathnormal{ulp}\) is the gap between any number and the next (i.e., \(\epsilon \times 2^E\)).

<!-- TODO: is this right -->
{{< subtext >}}
    The largest normalized floating point number is \((2 - 2^{-(p - 1)} \times 2^{2^q - 1})\).
{{< /subtext >}}

When a normalized floating point number that cannot be accurately represented is produced, something akin to rounding must be done. \(x_-\) is analogous to floor, and extraneous bits are truncated out. \(x_+\) is like ceiling by then also adding \((0 \cdot 0^(p - 2) 1)\) to the significand. 

{{< subtext >}}
    The default rounding mode is round-to nearest, with tiebreak to even. The tiebreak is whichever of \(x_-\) or \(x_+\) has a significand LSB of `0`.

    Whichever rounding mode is used will be applied for every operation.
{{< /subtext >}}

Other types of numbers other than normalized floating point can be represented through the reserved exponent bit strings \((0^q)\) and \((1^q)\):
<!-- TODO: shouldn't 1 - epsilon be the lowest normalized (2^q - 1) -->
- **Subnormal numbers**: when \(E = (0^q)\), numbers from \(1 - \epsilon\) to 0 can be achieved with an exponent of \(-(2^{q - 1} - 1)\) and a hidden bit of `0`
- **Infinites** and **Not a Number**: when \(E = (1^q)\), if \(S = (0^{p - 1})\), it's infinity; otherwise, it's `NaN`. 

{{< subtext >}}
    `NaN`'s propagate through operations
{{< /subtext >}}



# {{< heading "Characters" >}}
A C `char` type is a single byte encoded via ASCII. Unicode provided support for magnitudes more of characters. 

UTF-8 is a variable-length encoding that goes up to four bytes. If the first byte starts with `0`, the next 7 bits are direct from ASCII. `110`, `1110`, and `11110` means two, three, and four bytes respectively. Each continuation byte starts with `10`.

{{< subtext >}}
    Obtain the unicode code point by extracting the not-overhead bits and converting it into hex.
{{< /subtext >}}