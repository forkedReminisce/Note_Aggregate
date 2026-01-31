---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



The fundamental building block of digital logic circuits are digital logic gates. One would implement a simple boolean function (e.g., `NOT`, `AND`, `OR`). 

<!-- TODO: table of boolean notation and schematic -->



# {{< heading "Combinational Logic Circuits" >}}
This type of circuit has no cycles. That is to say that the result of the function is not referenced in any way the next time the function is run.

One such combinational circuit is the **ripple-carry adder**. This is where addition takes place. These are chained full adders, each of which comprise of two half-adders. A single FA handles a single digit in the addition. It receives a digit from each argument and a carry, and out comes the sum and a carry to give to the next FA.

{{< subtext >}}
    Subtraction takes the additive inverse of the second argument and gives the first FA a carry of `1`.
{{< /subtext >}}

<!-- TODO: schematic here -->