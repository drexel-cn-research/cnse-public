# Anonymous Receipt Identifier Generation

To verify survey completion for extra credit without linking any response to an individual student, each survey generated a random 8-digit receipt code on submission. Students copied this code into their homework submission; the code proved a survey had been completed without revealing which response belonged to which student, since the code itself carries no identifying information and is not stored alongside the response.

This document describes how that code was generated.

## Overview

Each assignment's survey used a secret, per-assignment reference number, generated once and known only to the researcher. Every receipt code issued for that assignment embeds this reference number, split across two positions and surrounded by random digits, along with a checksum. A receipt is valid only if it decodes to the correct reference number for that assignment and its checksum matches — a property a student cannot forge without knowing the secret reference number.

## Step 1: Generate the assignment's secret reference number

For each assignment, a two-digit reference number *N* was drawn once, uniformly at random, from the range **30–70**:

```
N = random_integer(0, 40) + 30
```

This number was generated fresh for each of the seven assignments and kept private — it does not appear anywhere in the public artifact repository. For the worked example below, assume **N = 42**.

## Step 2: Generate two range-bound random numbers

Two two-digit random numbers were then generated, each constrained relative to *N*:

- **Random₁**: a random integer from **0 to N − 1** (for N = 42: range 0–41)
- **Random₂**: a random integer from **N to 99** (for N = 42: range 42–99)

Both are formatted as two-digit strings, zero-padded if necessary (e.g., 7 → "07").

Because Random₁ is always strictly less than *N* and Random₂ is always greater than or equal to *N*, the pair brackets the secret reference number — a property used to help validate the receipt without publishing *N* itself.

## Step 3: Generate one single-digit random number

A third random number, **Random₃**, was drawn from **0–9** (a single digit, no padding).

## Step 4: Assemble the 8-digit code

The two digits of the reference number are split and inserted immediately after each of the two range-bound random numbers, rather than kept together — this avoids placing the reference number as a single recognizable block in the code.

| Position | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---|---|---|---|---|---|---|---|
| Field | Random₁ (tens digit) | Random₁ (ones digit) | Reference number, tens digit | Random₂ (tens digit) | Random₂ (ones digit) | Reference number, ones digit | Random₃ | Checksum |

## Step 5: Compute the checksum

The final digit is a simple checksum: the sum of the seven preceding digits, modulo 9.

```
checksum = (d1 + d2 + d3 + d4 + d5 + d6 + d7) mod 9
```

## Worked Example

Using N = 42:

| Step | Value |
|---|---|
| Reference number *N* | 42 → digits {4, 2} |
| Random₁ (drawn from 0–41) | 07 |
| Random₂ (drawn from 42–99) | 88 |
| Random₃ (drawn from 0–9) | 5 |
| Digits 1–7 | 0, 7, 4, 8, 8, 2, 5 |
| Digit sum | 0+7+4+8+8+2+5 = 34 |
| Checksum (34 mod 9) | 7 |
| **Final receipt code** | **07488257** |

## Validation

Given a submitted receipt code and the (privately held) reference number for that assignment, a receipt is accepted as valid only if both hold:

1. The digits at positions 3 and 6 reconstruct the correct reference number for that assignment.
2. The checksum at position 8 matches the digit sum (mod 9) of positions 1–7.

Because the reference number is never published and changes every assignment, a student cannot construct a valid-looking receipt without having actually received one from the survey tool.

In practice, this gives two levels of validation. A **quick, quasi-correctness check** can be done by inspection: knowing the reference number for the assignment, a grader can glance at positions 3 and 6 of a submitted code and immediately spot an obviously fabricated receipt (wrong digits in those two positions) without doing any arithmetic. **Full verification** additionally requires recomputing the checksum from positions 1–7 and confirming it matches position 8, which catches a code that gets the reference-number digits right by chance or copies them from a real receipt but alters the random digits. TAs were given a short validation script to perform full verification at grading time rather than checking by hand.
