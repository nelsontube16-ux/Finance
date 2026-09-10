# MF20006 — Lecture 1: Numbers and Computation

**Course:** Introduction to Computer Science, Fudan University
**Lecturer:** Hui Xu (xuh@fudan.edu.cn)
**Source material:** `l1numbersslides.pdf` (44 slides) + `l1numbersnotes.pdf` (8 pages)

Exam-oriented study guide. Sections 1–3 follow the slides; Section 4 is the hardware
deep dive; Section 5 is drills with worked answers; Section 6 is the one-page cram sheet.

---

## The single idea

A computer is a box of switches, and a switch is either off or on. So **everything** —
numbers, text, images, code — must be encoded as patterns of 0s and 1s. The lecture
answers three questions in order:

1. How do we encode **whole numbers**, including negatives? (slides 3–19)
2. How does a machine of physical switches **actually add** them? (slides 20–31)
3. How do we encode **fractions**, and what breaks? (slides 32–44)

---

## 1. Integers

### 1.1 Positional notation (slides 6–7)

Position encodes a power of the base:

```
123      = 1×10² + 2×10¹ + 3×10⁰
(123)ₙ   = 1×n²  + 2×n¹  + 3×n⁰       ← generalise to any base n
(1011)₂  = 1×2³ + 0×2² + 1×2¹ + 1×2⁰ = 11
```

One binary digit = one **bit**. Base 2 is used because hardware only has to distinguish
**two** voltage levels ("low" and "high"), which is robust against electrical noise;
distinguishing ten levels reliably is not.

**Decimal → binary:** divide by 2 repeatedly, read the remainders **backwards**.

```
11 ÷ 2 = 5 r 1
 5 ÷ 2 = 2 r 1
 2 ÷ 2 = 1 r 0
 1 ÷ 2 = 0 r 1      → read up → 1011
```

```
function to_binary(X):
    if X == 0: return "0"
    result = ""
    while X > 0:
        r = X mod 2
        result = str(r) + result      # PREPEND: remainders arrive least-significant first
        X = X div 2
    return result
```

### 1.2 Storage and fixed width (slides 8–9)

Memory is one unbroken ribbon of bits with no delimiters. Two conventions locate a number:

- **Start** — the **memory address**.
- **End** — an agreed **fixed length**: 8, 16, 32, 64 bits.

This is what a type declaration (`int x`) actually tells the machine. Fixed width implies a
hard ceiling: 8 bits = 2⁸ = 256 patterns = unsigned range 0…255. Exceed it and you get
**overflow**. Leading zeros are mandatory (3 = `00000011`).

### 1.3 Hexadecimal (slide 10)

Base 16, digits 0–9 then A–F. Used because **16 = 2⁴**, so **one hex digit = exactly four
bits**, with no arithmetic:

```
1011 0110  →  B6  →  0xB6
```

One byte is always two hex characters. Hex is pure shorthand for binary — a display
convention, not a storage format.

### 1.4 Binary quantifiers (slide 12)

| Prefix | Power | ≈ decimal |
|---|---|---|
| kilo (K) | 2¹⁰ = 1024 | thousand |
| mega (M) | 2²⁰ | million |
| giga (G) | 2³⁰ | billion |
| tera (T) | 2⁴⁰ | trillion |
| peta (P) | 2⁵⁰ | |
| exa (E)  | 2⁶⁰ | |
| zetta (Z)| 2⁷⁰ | |
| yotta (Y)| 2⁸⁰ | |

The divergence from decimal SI widens with each step (a "1 TB" drive shows as ~0.909 TiB).

### 1.5 Signed integers (slides 15–16)

Three candidate schemes, presented as an engineering argument:

| Scheme | Rule for −x | 4-bit example (−3) | Problem |
|---|---|---|---|
| **Sign-and-magnitude** | flip the leading bit | `1011` | two zeros (`0000`, `1000`) |
| **One's complement** | invert all bits; x + y = 2ⁿ − 1 | `1100` | still two zeros (`0000`, `1111`) |
| **Two's complement** | invert all bits **then add 1**; x + y = 2ⁿ | `1101` | **none — this is the standard** |

Equivalent negation rule: subtract 1 first, then invert. Both directions work.

**4-bit two's complement table:**

| Dec | −8 | −7 | −6 | −5 | −4 | −3 | −2 | −1 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Bin | 1000 | 1001 | 1010 | 1011 | 1100 | 1101 | 1110 | 1111 | 0000 | 0001 | 0010 | 0011 | 0100 | 0101 | 0110 | 0111 |

Properties to be able to state:

- All 2ⁿ patterns used exactly once; **exactly one zero**.
- Range is **asymmetric**: −2ⁿ⁻¹ … 2ⁿ⁻¹ − 1 (one more negative than positive, since 0
  occupies a non-negative slot).
- **The MSB carries weight −2ⁿ⁻¹**; all other bits are positive as usual. Fastest way to
  read a negative value by hand: `1011` = −8 + 0 + 2 + 1 = −5.

### 1.6 Why two's complement wins (slides 17–18)

**Subtraction becomes addition**, so only an adder circuit is ever built:

```
  2 − 5 = 2 + (−5)          7 − 5 = 7 + (−5)
     0010                      0111
   + 1011                    + 1011
   ------                    --------
     1101 = −3               1 0010 → drop carry → 0010 = 2
```

**Proof (slide 18).** "mod 2ⁿ" is the mathematical name for "keep n bits, discard overflow"
— which hardware does for free by having nowhere to put the extra bit. By definition the
stored pattern for −y holds the value 2ⁿ − y, so:

```
(x + (−y)) mod 2ⁿ
  = (x + 2ⁿ − y) mod 2ⁿ         substitute the definition
  = (x − y + 2ⁿ) mod 2ⁿ         regroup
  = (x − y) mod 2ⁿ              because 2ⁿ mod 2ⁿ = 0        ∎
```

**Clock intuition (same proof, no algebra).** An n-bit register is a clock with 2ⁿ numbers
on its face. On a 12-hour clock, winding back 5 hours equals winding forward 7, because
7 = 12 − 5. On a 4-bit (16-hour) clock, subtracting 5 equals adding 16 − 5 = 11 = `1011`
— which is exactly the two's complement of 5. The discarded carry bit *is* the lap around
the dial.

**Corollary — where it breaks:** the identity holds *mod 2ⁿ*. If the true result falls
outside −2ⁿ⁻¹ … 2ⁿ⁻¹−1 you get the right answer mod 2ⁿ, i.e. the wrong answer. That is
signed overflow (detection in §4.7).

---

## 2. Computation — the abstraction ladder (slides 20–31)

```
CPU
 └─ ALU (Arithmetic Logic Unit)
     └─ logic gates (AND / OR / NOT / XOR)
         └─ transistors  (switches)
             └─ semiconductors (doped silicon)
```

Each layer is built purely from the layer below. Nobody at the top thinks about electrons;
nobody at the bottom knows what a number is. Full treatment in §4.

---

## 3. Floating-point numbers

### 3.1 Fixed-point (slide 34)

Declare a permanent split, e.g. 4 integer bits + 4 fraction bits. Positions right of the
point are negative powers of 2:

```
(1000.0100)₂ = 2³ + 2⁻² = 8.25
(0010.1100)₂ = 2 + ½ + ¼ = 2.75
```

**Limitation:** one global, up-front trade-off. More integer bits = more range, less
precision; more fraction bits = the reverse. A program handling both 0.000001 and 1,000,000
cannot be served.

### 3.2 Floating-point (slide 35)

Let the binary point **move**: store the significant digits plus an exponent saying where
the point goes. This is scientific notation:

```
625 = 6.25 × 10²          (110.01)₂ = (1.1001)₂ × 2²
```

Requires a **standard** so encoder and decoder agree.

### 3.3 IEEE 754 (slide 36)

```
32-bit single precision:  ┌─┬────────┬───────────────────────┐
                          │s│ exp(8) │     mantissa (23)     │
                          └─┴────────┴───────────────────────┘
64-bit double precision:   1 sign + 11 exponent + 52 mantissa
```

**value = (−1)ˢ × 1.m × 2^(exp − 127)**   (bias 1023 for double)

Two design points that get examined:

- **The implicit leading 1.** Normalised binary always has exactly one non-zero digit left
  of the point, and in binary the only non-zero digit is 1 — so it is never stored. You get
  a 24th bit of precision free. (Exponent field all-zero signals a *subnormal*, where the
  implicit bit is 0 instead.)
- **Why the bias of 127?** The exponent must go negative. Storing `exponent + 127` keeps
  the field non-negative, which means **two floats can be compared by comparing their bit
  patterns as integers** — comparison and sorting hardware becomes nearly free. Exponent
  fields 0 and 255 are reserved for special values.

**Reserved patterns:**

| exp field | mantissa | Meaning |
|---|---|---|
| all 0s | 0 | ±zero |
| all 0s | ≠ 0 | subnormal (gradual underflow) |
| all 1s | 0 | ±Infinity |
| all 1s | ≠ 0 | NaN |

**Worked decode** — `0 10000110 10010000000000000000000`:
sign 0 → positive; exp = 134, 134 − 127 = **7**; mantissa → 1 + 2⁻¹ + 2⁻⁴ = **1.5625**;
value = 1.5625 × 2⁷ = **200**.

### 3.4 Non-uniform distribution (slide 37)

Ticks on the float number line cluster densely near 0 and spread apart toward the extremes.
The mantissa gives ~7 decimal digits of **relative** precision; the exponent scales that.

| Near | Gap between consecutive float32 values |
|---|---|
| 1.0 | ~1.2 × 10⁻⁷ |
| 1,000,000 | ~0.06 |
| > 2²⁴ = 16,777,216 | ≥ 2 — odd integers cannot be stored |

Floats trade **absolute** accuracy for **relative** accuracy. Correct for physics, wrong for
money — hence currency is stored as integer cents or a decimal type.

### 3.5 Encoding a decimal (slide 39) — 11.25

```
Integer part 11 — divide by 2, read remainders UP:
  11/2 = 5 r 1 ;  5/2 = 2 r 1 ;  2/2 = 1 r 0 ;  1/2 = 0 r 1   → 1011

Fractional part 0.25 — multiply by 2, read the emitted digits DOWN:
  0.25 × 2 = 0.5 → 0 ;  0.50 × 2 = 1.0 → 1                     → .01

So 11.25 = (1011.01)₂
Normalise:  1011.01 = 1.01101 × 2³      → exp = 3
Stored exponent: 3 + 127 = 130 = 10000010
Mantissa: drop the implicit leading 1 → 01101, pad to 23 bits

RESULT:  0 10000010 01101000000000000000000
```

(Integer part: divide, read up. Fraction: multiply, read down. Mirror images.)

### 3.6 Precision loss (slide 40)

**A fraction is exactly representable in binary iff its denominator, in lowest terms, is a
power of 2.**

- 0.5 = 2⁻¹ ✓ exact.  0.25, 0.75, 0.125 ✓ exact.
- 0.1 = 1/10 — the 5 in the denominator can never be divided by a power of 2, so
  0.1 = `0.0001100110011…`₂ repeating forever → **must be rounded**.

Hence `0.1 + 0.2 != 0.3`. Nothing is broken: 0.1 is stored as 0.100000001490116…, and three
separately-rounded values do not reconcile. Exactly analogous to 1/3 being unwritable in
decimal.

**Practical rules:** never compare floats with `==` (use `|a − b| < ε`); never store money
in a float; never use a float for an ID or a counter.

### 3.7 All data are bits (slides 42–43)

> What distinguishes one type of information from another is **not the bits themselves but
> the interpretation.**

The byte `01000001` is 65, or `'A'`, or a grey pixel, or a CPU instruction — context alone
decides. This is why types and file formats exist, and why memory-safety bugs are dangerous.

**ASCII** — 8 bits (7 used) = 128 characters:

- `'0'` = 0x30 = `0011 0000` = 48 → so `char − '0'` converts a digit character to its value.
- `'A'` = 0x41 = `0100 0001` = 65; `'a'` = 97. They differ by **32 = 2⁵** — case conversion
  is a single bit flip.
- 0x00–0x1F are invisible control codes (NUL, LF, CR, ESC, BEL).
- The digits occupy 0x30–0x39 **in order**, deliberately.

Note: the character `'5'` is stored as 53, not 5.

---

## 4. Hardware deep dive (slides 21–31, exam depth)

### 4.1 Silicon and doping

Silicon is **group IV**: 4 valence electrons, each shared with 4 neighbours in the crystal
— every bond satisfied, so **no mobile carriers** and poor conductivity. That controllable
mediocrity is the point.

**Doping** adds ~1 impurity atom in 10⁷:

| | Dopant group | Example | Adds | Majority carrier | Name |
|---|---|---|---|---|---|
| **N-type** | V (5 valence e⁻) | phosphorus, arsenic | a spare electron | **electrons** (negative) | donor |
| **P-type** | III (3 valence e⁻) | boron, gallium | a missing bond = **hole** | **holes** (positive) | acceptor |

Two common exam traps:

- **A hole is not a particle.** It is a vacancy; when a neighbouring electron hops in, the
  vacancy moves the other way, so it behaves as a positive carrier. Bookkeeping device.
- **Doped silicon is electrically neutral overall** — neutral atoms were added. N-type is
  not negatively charged; it merely has electrons free to move. Do not confuse *carrier
  type* with *net charge*.

### 4.2 The p–n junction

At a p/n interface, electrons diffuse across and recombine, leaving **immobile ionised
dopant atoms** and a carrier-free **depletion region** with a built-in field that opposes
further diffusion.

- **Forward bias** (p→+, n→−): depletion region narrows → **conducts**.
- **Reverse bias** (p→−, n→+): depletion region widens → **blocks**.

One-way conduction = a **diode**. It is also why a MOSFET's source and drain do not simply
short together: back-to-back junctions block until a channel is created.

### 4.3 The MOSFET

**M**etal–**O**xide–**S**emiconductor **F**ield-**E**ffect **T**ransistor. Terminals: gate,
source, drain (+ body). Structural key: **the gate is separated from the channel by an
insulating oxide (SiO₂) and is electrically connected to nothing it controls.**

Three consequences:

1. **No steady current into the gate** — it is a capacitor plate; control is by **electric
   field** (hence *field-effect*).
2. **Near-zero static power** and **high fan-out** — one output can drive many inputs,
   because those inputs sink no DC current. Without this you could not stack billions.
3. Switching still costs energy: the gate capacitance must be charged/discharged every
   transition, **P_dynamic ≈ ½CV²f**. This is why clock speeds stalled near 4 GHz in the
   mid-2000s — power scales with frequency and the heat became unmanageable.

**NMOS turn-on:** n-type source/drain in a **p-type** body. Gate above the **threshold
voltage V_th** → the field repels holes and attracts electrons to the surface, **inverting**
a thin layer to n-like → an **inversion channel** bridges source to drain. Gate low → no
channel → off. **PMOS** is the mirror: p-type source/drain in an n-type body, a *low* gate
forms a p-channel.

| | Body | Carriers | Conducts when gate is | Passes a strong |
|---|---|---|---|---|
| **NMOS** | p-type | electrons | **high** (1) | **0** (ground) |
| **PMOS** | n-type | holes | **low** (0) | **1** (VCC) |

**Why a transistor enables computation:** it is a switch thrown by a *voltage*, so one
circuit's output can be the next circuit's control input — signals commanding signals,
cascaded without limit.

### 4.4 NMOS logic (as taught in slides 24–26) vs. CMOS (as actually built)

The slides build NOT/AND/OR from NMOS + a pull-up load. Historically real (1970s NMOS
logic), but every chip since ~1980 uses **CMOS**. Know both — power questions hinge on it.

**Problem with NMOS-only logic:** when the pull-down NMOS conducts, the load resistor is
still tied to VCC, so a **direct VCC→ground path** burns power continuously while the output
sits low. Also gives a weak, slow pull-up.

**CMOS inverter:**

```
        VCC
         │
      ──┤ PMOS        A = 0 → PMOS on,  NMOS off → OUT = 1
   A ───┤             A = 1 → PMOS off, NMOS on  → OUT = 0
      ──┤ NMOS
         │            Never both on → NO static power path.
       Ground
             ├──── OUT
```

Power is consumed only *during transitions*. That single property is why CMOS won and why a
phone needs no fan.

**The CMOS recipe:** every gate = an NMOS **pull-down network** to ground + a PMOS
**pull-up network** to VCC, wired as **duals** (series ⇄ parallel).

| NMOS pull-down | Behaviour | Gate |
|---|---|---|
| two in **series** | pulls low only when **both** inputs high | **NAND** (4 transistors) |
| two in **parallel** | pulls low when **either** input high | **NOR** (4 transistors) |

CMOS is **naturally inverting**: NAND and NOR are the cheap primitives; **AND = NAND +
inverter = 6 transistors**. AND is genuinely *more expensive* than NAND in silicon — the
opposite of the truth-table intuition.

This also explains slide 25's garbled text: its AND is series NMOS (NAND behaviour) plus an
"output transistor" (the inverting stage) that restores AND. Slide 26's OR is parallel NMOS
(NOR) plus the same output stage.

### 4.5 Truth tables and Boolean algebra

| A | B | ¬A | A∧B | A∨B | A⊕B |
|---|---|---|---|---|---|
| 0 | 0 | 1 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 | 1 | 1 |
| 1 | 0 | 0 | 0 | 1 | 1 |
| 1 | 1 | 0 | 1 | 1 | **0** |

XOR = "one or the other, **but not both**" — it differs from OR only in the last row.

**Functional completeness — NAND alone builds everything:**

```
NOT A   = A NAND A
A AND B = (A NAND B) NAND (A NAND B)
A OR  B = (A NAND A) NAND (B NAND B)        ← De Morgan
```

Since any truth table can be written as a sum of products over {AND, OR, NOT}, and NAND
gives all three, **NAND is universal**. (So is NOR.)

**Laws to memorise:**

| Law | Form |
|---|---|
| Identity | A + 0 = A · 1 = A |
| Null | A + 1 = 1, A · 0 = 0 |
| Idempotent | A + A = A, A · A = A |
| Complement | A + Ā = 1, A · Ā = 0 |
| **De Morgan** | **¬(A·B) = Ā + B̄**, **¬(A+B) = Ā · B̄** |
| Distributive | A(B+C) = AB + AC, **and** A + BC = (A+B)(A+C) |
| Absorption | A + AB = A |

De Morgan is the most examined identity in the topic — it formalises "an inverting gate can
be redrawn with the inversions moved to the inputs," which is how NANDs become ANDs on paper.

**XOR facts:** `A ⊕ B = ĀB + AB̄`; `A ⊕ 0 = A`; `A ⊕ 1 = Ā`; `A ⊕ A = 0`; associative and
commutative. Deep reading: **XOR is a parity detector** — A⊕B⊕C = 1 iff an *odd* number of
inputs are 1.

### 4.6 Adders, derived rather than memorised

Method: **sum-of-products** — for each row whose output is 1, write the minterm; OR them.

**Half adder** (slide 28):

| A | B | Cout | S |
|---|---|---|---|
| 0|0|0|0|
| 0|1|0|1|
| 1|0|0|1|
| 1|1|1|0|

- S = ĀB + AB̄ = **A ⊕ B**
- Cout = **A · B**

Two gates — arithmetic from logic. "Half" = produces a carry but cannot accept one, so it
serves only bit position 0.

**Full adder** (slide 29): inputs A, B, Cin.

- Sum is 1 in rows 001, 010, 100, 111 — the rows with an **odd** number of 1s (parity):
  **S = A ⊕ B ⊕ Cin**
- Cout is 1 in rows 011, 101, 110, 111 — "at least two inputs are 1", the **majority
  function**:

```
Cout = AB + ACin + BCin        (SOP form, "any two agree")
     = AB + (A ⊕ B)·Cin        (circuit form — reuses the A⊕B wire from the sum)
```

Both forms are equal; the second is preferred because A⊕B already exists. Structure = **two
half adders + one OR**. Cost = **2 XOR + 2 AND + 1 OR = 5 gates**.

**Ripple-carry adder** (slide 30): half adder for bit 0, full adders above, carries chained.

- Stage k cannot start until stage k−1 delivers its carry → delay is **O(n)**. At ~2 gate
  delays per stage, a 32-bit adder ≈ **64 gate delays**.
- That chain is typically the CPU's **critical path**, and the critical path sets the
  maximum clock frequency.

**Carry-lookahead adder** — the standard answer to "how do you make addition faster?":

```
Generate:  Gᵢ = Aᵢ · Bᵢ           this bit makes a carry regardless of Cin
Propagate: Pᵢ = Aᵢ ⊕ Bᵢ           this bit passes an incoming carry through
           Cᵢ₊₁ = Gᵢ + Pᵢ·Cᵢ
```

Expanding the recurrence makes every carry a direct function of the inputs — **O(log n)**
depth at the cost of more gates. Classic time/area trade-off.

### 4.7 Two omissions that examiners like

**(a) The adder is also a subtractor — one control wire.**
Since A − B = A + ¬B + 1:

- Put an **XOR on each B input**, with control line **SUB** as the other operand.
  SUB = 0 → B passes unchanged → computes A + B. SUB = 1 → every B bit inverted.
- Feed **SUB into C₀** (the bottom carry-in). When SUB = 1 that supplies the crucial **+1**.

One control bit reconfigures the circuit, at a cost of n XOR gates. This is the concrete
cash value of slide 17's "reuse the adder".

**(b) Overflow detection — two different flags.**
The hardware computes identical bits either way; only interpretation differs, so it raises
two flags:

- **Carry flag (unsigned overflow)** = carry out of the MSB.
- **Overflow flag (signed overflow)** = **carry-in to MSB ⊕ carry-out of MSB**.
  Equivalently: two positives gave a negative, or two negatives gave a positive.

Worked 4-bit example: `0111 + 0001 = 1000`, i.e. 7 + 1 = −8. Carry *out* of the MSB is 0, so
unsigned is content (7+1=8 ✓). But carry *in* to the MSB is 1 ≠ 0 → **signed overflow**.

### 4.8 From adder to ALU

An **ALU** is parallel units — adder/subtractor, AND, OR, XOR, shifter — all fed the same
operands simultaneously, with a **multiplexer** selecting whose result to keep, driven by
the instruction's **opcode**.

A **2-to-1 multiplexer** is itself gates: `OUT = (S̄ · I₀) + (S · I₁)` — "if S is 0 take I₀,
else I₁."

Computing discarded results is deliberate: gates are cheap, time is expensive, so compute
everything in parallel and select afterwards. The ALU also emits **status flags**: Zero (an
OR-reduce + inverter), Negative (copy of the MSB), Carry, Overflow. Those flags are what
`if (a > b)` compiles into.

Vocabulary: all of the above is **combinational logic** — output is a pure function of
current inputs, no memory. Storing values (registers, RAM) needs **sequential logic**: gates
with feedback forming latches and flip-flops, clocked. If asked "why can't an adder
remember?" — it has no feedback path.

### 4.9 Von Neumann architecture (slide 31)

Five parts: **Input**, **Memory**, **CPU** (= **Control Unit** + **ALU**), **Output**.

**Stored-program concept:** instructions live in the **same memory as data**, in the same
format — numbers. Consequences:

- The machine is **general-purpose**: change memory, get a different machine, no rewiring.
- Programs can read and write other programs — compilers, loaders, OSes, JITs depend on it.
- Contrast **Harvard architecture** (separate instruction and data memory): faster and
  safer, less flexible; used in microcontrollers and, in hybrid form, in split L1 caches.

**Fetch–decode–execute cycle:**

1. **Fetch** — Control Unit puts the **Program Counter** on the address bus; memory returns
   the instruction; PC advances.
2. **Decode** — read the opcode, assert control lines (which ALU op, which registers).
3. **Execute** — operands flow from registers into the ALU; flags are set.
4. **Writeback** — result to a register or memory.
5. Repeat. A **branch** simply writes a new value into the PC — that is all a loop or an
   `if` physically is.

**Von Neumann bottleneck:** CPU speed far exceeds memory speed, and instructions and data
share one path. The entire memory hierarchy (registers → L1/L2/L3 → RAM → disk) exists to
hide this gap. That is the answer to "why do caches exist?"

---

## 5. Exercises with worked answers

### From the slides

**Slide 11 — decode / encode.**

| Question | Answer |
|---|---|
| `1111 1111` | **255** unsigned (a run of n ones = 2ⁿ−1); **−1** if read as signed |
| `1000 0000` | **128** unsigned; **−128** signed |
| 100 → binary | 64+32+4 = **`0110 0100`** (0x64) |
| 1024 → binary | 2¹⁰ = **`100 0000 0000`** — a 1 followed by ten 0s |

**Slide 19 — 16-bit signed range.** Range is −2ⁿ⁻¹ … 2ⁿ⁻¹−1:

- Largest = **32767** = `0111 1111 1111 1111`
- Smallest = **−32768** = `1000 0000 0000 0000`

**Slide 38 — decode three float32 values.** Split 1 / 8 / 23:

| Bits | Working | Value |
|---|---|---|
| `0 11111111 000…0` | exponent all 1s + zero mantissa → reserved | **+Infinity** |
| `0 10000000 011000…` | e = 128−127 = 1; 1.011₂ = 1.375 | **2.75** |
| `1 10000001 011000…` | sign 1; e = 129−127 = 2; 1.375 × 4 | **−5.5** |

(The first is a trick question testing the reserved patterns. All-1s exponent with a
*non-zero* mantissa would be NaN.)

**Slide 40 — which of 0.1, 0.2, 0.3, 0.4, 0.5 are exact?** Only **0.5** (= 2⁻¹). The others
all have a factor of 5 in the denominator, which no power of 2 divides.

### From the notes (§1.4)

**1) 16-bit signed max/min** → 32767 and −32768 (as above).

**2) Test whether an integer is a power of 2.**

```c
bool is_power_of_two(int x) { return x > 0 && (x & (x - 1)) == 0; }
```

A power of 2 has exactly one 1-bit; subtracting 1 clears it and sets every lower bit, so the
AND is zero:

```
 8 = 1000,  7 = 0111  →  8 & 7 = 0000  ✓
12 = 1100, 11 = 1011  → 12 & 11 = 1000 ✗
```

The `x > 0` guard matters: 0 passes the AND test but is not a power of 2.

**3) Which decimals are not exact in binary floating point?** Any whose denominator in
lowest terms is not a power of 2 — 0.1, 0.2, 0.3, 1/3, 0.7. Example: 0.1 = 1/10, and 10 = 2×5;
the factor 5 makes the expansion `0.0001100110011…`₂ repeat forever, so it must be rounded
at 23 bits.

**4) Which integers are not exact in float32?** float32 has 23 stored mantissa bits + 1
implicit = **24 bits of significand**. Every integer up to 2²⁴ = 16,777,216 is exact;
**16,777,217 is the first that is not** (it needs 25 bits and rounds down). Beyond 2²⁴ only
even numbers are representable; beyond 2²⁵ only multiples of 4; the gap doubles at each
power. This is §3.4's non-uniform number line, stated arithmetically.

### Extra drills

**a) `1101 0110` in 8-bit two's complement?** → **−42**. Two methods:

```
A (negate back):  invert → 0010 1001, +1 → 0010 1010 = 42, MSB was 1 → −42
B (weighted MSB): −128 + 64 + 16 + 4 + 2 = −42          ← faster under time pressure
```

**b) Why does `0.1 + 0.2 != 0.3` but `0.5 + 0.25 == 0.75`?** 0.5 = 2⁻¹ and 0.25 = 2⁻² are
single mantissa bits, stored exactly, and their sum is exactly representable. 0.1 and 0.2
are infinitely repeating in binary and must be rounded; two rounded inputs do not reconcile
with a separately-rounded 0.3.

**c) Build XOR from NAND only.** `A ⊕ B = (A NAND (A NAND B)) NAND (B NAND (A NAND B))`
— 4 NAND gates.

**d) Show 5 − 3 = 2 in 4-bit two's complement.** −3 = invert `0011` → `1100`, +1 → `1101`.
Then `0101 + 1101 = 1 0010`; discard the carry → `0010` = **2** ✓

---

## 6. Cram sheet

**Representation**
- Position ⇒ power of the base; bit = binary digit; base 2 because two voltage levels are noise-robust.
- Dec→bin: divide by 2, read remainders **up**. Fraction: multiply by 2, read digits **down**.
- 1 hex digit = 4 bits exactly (16 = 2⁴). 1 byte = 2 hex chars.
- Unsigned n bits: 0 … 2ⁿ−1. Signed (two's complement): **−2ⁿ⁻¹ … 2ⁿ⁻¹−1**.
- Two's complement: **invert then +1**; MSB weight is **−2ⁿ⁻¹**; one zero; subtraction = addition (mod 2ⁿ).
- Signed overflow = carry-in to MSB ⊕ carry-out of MSB. Unsigned overflow = carry out of MSB.

**Hardware**
- Ladder: semiconductor → transistor → gate → adder → ALU → CPU.
- N-type = donor, extra electrons. P-type = acceptor, holes. Doped Si is still neutral overall.
- NMOS conducts on gate **high**; PMOS on gate **low**. Gate is insulated → field control, no gate current.
- CMOS = dual pull-up (PMOS) / pull-down (NMOS) → no static power. NAND/NOR are cheap (4 T); AND/OR cost 6 T.
- NAND (or NOR) alone is **functionally complete**. De Morgan: ¬(AB) = Ā+B̄.
- Half adder: **S = A⊕B, C = A·B**. Full adder: **S = A⊕B⊕Cin, Cout = AB + (A⊕B)Cin**.
- Ripple-carry delay is O(n); carry-lookahead (G = AB, P = A⊕B, Cᵢ₊₁ = Gᵢ + PᵢCᵢ) is O(log n).
- Von Neumann: instructions and data in the same memory; fetch–decode–execute; a branch writes the PC.

**Floating point**
- float32 = 1 sign + 8 exponent (bias **127**) + 23 mantissa. float64 = 1 + 11 (bias 1023) + 52.
- value = (−1)ˢ × **1**.m × 2^(exp − bias); leading 1 is implicit; bias makes bitwise integer comparison work.
- exp all 1s: mantissa 0 → ±Inf, mantissa ≠ 0 → NaN. exp all 0s → zero / subnormal.
- Exact iff the denominator is a power of 2. Integers exact up to **2²⁴** in float32.
- Precision is **relative**, not absolute → gaps grow with magnitude → never use floats for money.
- All data are bits; **meaning comes from interpretation**. ASCII: `'0'` = 48, `'A'` = 65, case bit = 32.

---

## 7. Errata in the source slides

- **Slide 25 (AND)** — text reads "pulled down to VCC: Not A = 1", conflating two sentences
  and reusing the NOT gate's label. The notes are correct: the AND output goes **high** only
  when both inputs are high, via the output (inverting) stage.
- **Slide 36** — labels the exponent as bits "24–31" and the mantissa "1–23", numbering from
  the right with the sign as bit 32. Same layout as standard references, opposite numbering
  (most texts call the sign bit 31).
- **Slide 39** — the printed bit string has **36 characters**, four trailing zeros too many.
  The correct 32-bit encoding of 11.25 is `0 10000010 01101000000000000000000` (verified
  against the actual IEEE 754 encoding). The slide's first 32 characters are correct.
- **Slide 4** — the "digits are based on their number of angles" story is explicitly labelled
  a **myth** by the lecturer. Slide 5 gives the real lineage: Shang (1300 BC) → Brahmi
  (300 BC) → Hindu (Gwalior) → Sanskrit-Devanagari → Western/Eastern Arabic → 15th–16th
  century European forms.

## 8. Method note (slides 14 and 41)

The lecturer twice asks you to attempt these conversions with an LLM, hunt for an
**adversarial case** where it fails, verify against ground truth, then retry with a **coding
agent** that writes and executes a program.

The lesson: a language model doing long arithmetic token-by-token is pattern-matching; a
model that writes and runs ten lines of code is using a machine that is correct by
construction. **Move exact work to the tool that cannot be wrong.** The float case is the
better trap, because a wrong bit in position 19 of a confident, well-formatted 32-bit string
is invisible to the eye.

Ground-truth checkers named in the slides:
- Binary ↔ decimal: https://www.rapidtables.com/convert/number/binary-to-decimal.html
- IEEE 754: https://www.h-schmidt.net/FloatConverter/IEEE754.html
