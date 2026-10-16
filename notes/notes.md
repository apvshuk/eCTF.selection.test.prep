## Bitwise Operators in C

- 6 operators in C for bitwise manipulation; only applicable to integral operands, i.e: char, int, short, long.
- 6 operators are:
  - "&" bitwise AND
  - "|" bitwise inclusive OR
  - "^" bitwise exclusive OR
  - "<<" left shift
  - ">>" right shift
  - "~" one's complement (unary)

- Integers in C are always stored as binary in memory. Bitwise operators operate directly on those bits. Before a bitwise operation, both operands are promoted to a common type (usually int). Any narrower type gets zero-padded on the left to match the width. Bitwise operators compare corresponding bits one-to-one.


- Zero-padding on the left is harmless: 0 & (anything) = 0, so the added bits produce 0 and don't affect the result. Common use: masking — AND-ing with a number whose bits are 1 where you want to keep and 0 where you want to zero out.

```
AND (&) - 0 & (anything) becomes 0; 1 & 1 is 1, only
0 & 0 = 0
0 & 1 = 0
1 & 0 = 0
1 & 1 = 1

OR (|) - if atleast one of the bits is 1, it becomes 1
0 | 0 = 0
0 | 1 = 1
1 | 0 = 1
1 | 1 = 1

EXLUSIVE (^) - different bits: 1, same bits: 0
0 | 0 = 0
0 | 1 = 1
1 | 0 = 1
1 | 1 = 0
```

- The shift operators << and >> perform left and right shifts of their left operand by the number
of bit positions given by the right operand, which must be non-negative. Thus x << 2 shifts
the value of x by two positions, filling vacated bits with zero; this is equivalent to
multiplication by 4. Right shifting an unsigned quantity always fits the vacated bits with zero.
Right shifting a signed quantity will fill with bit signs (''arithmetic shift'') on some machines
and with 0-bits (''logical shift'') on others.

- The unary operator ~ yields the one's complement of an integer; that is, it converts each 1-bit
into a 0-bit and vice versa

> **[Note]** One must distinguish the bitwise operators & and | from the logical operators && and ||, which imply left-to-right evaluation of a truth value. For example, if x is 1 and y is 2, then x & y is zero while x && y is one.
> **[Note]** My machine - int variables - 32 bits. Subject to change with machine. A number stored as binary may not occupy all 32 bit positions; the remaining "vacant" positions, on the left are padded with 0's. When using UNARY operator, these "hidden" 0's flip to 1's. So keep that in mind when calculating the UNARY of a number, use the BIT SIZE of the variable. Answer of UNARY of a number is a function of BIT SIZE of the variable type.