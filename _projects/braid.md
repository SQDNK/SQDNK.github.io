---
layout: page
title: braid theory
description: formally prove braid equivalence algorithms
importance: 3
category: work
---

<a href="https://textiles-lab.github.io/">the lab</a>.

The idea to view braid groups as a case of Artin groups was used in this
<a href="https://textiles-lab.github.io/publications/2021-braid-transfer-plan/">research paper</a>
which presents the "first complete, discrete representation of the machine’s loop-tangling process".

define braid groups as a case of Artin groups
First we introduce `artin_tits` by inductively constructing its relation
and creating a presented group over this relation. Then we introduce `braid_group`
as a special case of `artin_tits`.

the braid group:
/--
Construct the braid group as a special case of Artin-Tits.

Input:
`n : ℕ` the number of strands in the braid group

Output:
`artin_tits (fin (n - 1)) (λ i j, if abs ((i : ℤ) - j) = 1 then 3 else 2)`
a presented group on braid group relations.

Details:

- The `λ` function is saying as follows: for any generators
  `i,j`, the relation is `i * j = j * i` and the entry is `(i,j) = 2`; for generators
  `i, i+1`, there is a second relation `i * (i+1) * i = (i+1) * i * (i+1)` and the
  entry is `(i,i+1) = 3`. These entries are the length of the `s * t * ...` sequence
  in the relation, excluding the `1` at the end.
- In the `λ` function, `i,j` are being cast from `fin (n-1)` to `ℤ` so subtraction
  can be done in `ℤ` and not `fin (n-1)`, where it is subtraction mod `n-1`.
  We do this in case of something like the following:

  `def test (a : fin 5) := a - (2 : fin 5)`

  `#eval test 0` which evaluates to `3 ≣ -2 mod 5` since `-2 = 5(-1) + 3`, rather than `2`.
  --credit the person.

- The type of the index is `fin (n-1)`, which has `n-1` terms/elements.
  An element of `fin n` is a natural number `x` and a proof that `x` is less
  than `n`, so these elements are just the numbers `0, 1, 2, ..., n-1`.  
   --credit this person too.
  -- This code was written by Lean Zulip user @Kyle Miller.
  -/
  abbreviation braid_group (n : ℕ) :=
  artin_tits (fin (n - 1))
  (λ i j, if abs ((i : ℤ) - (j : ℤ)) = 1 then 3 else 2)

reference: @Kyle Miller (possibly reached at <a href="https://leanprover.zulipchat.com/">lean forum</a>).
