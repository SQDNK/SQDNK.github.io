---
layout: page
title: braid theory
description: formally prove braid equivalence algorithms
importance: 3
category: work
---

This was my project when I was an intern at <a href="https://textiles-lab.github.io/">the CMU Textiles Lab</a>.

The general idea is to define braid groups as a case of Artin groups. This was used in this
<a href="https://textiles-lab.github.io/publications/2021-braid-transfer-plan/">research paper</a>
which presents the "first complete, discrete representation of the machine’s loop-tangling process".

First we introduce `artin_tits` by inductively constructing its relation
and creating a presented group over this relation. Then we introduce `braid_group`
as a special case of `artin_tits`.

/-
How we implement the Artin-Tits group and then the braid group:

Let `F` be a finite set. Define a square matrix `m` to be the Coxeter matrix
over `F`. For `a,b ∈ F` and `n ≥ 2`, the entry of the matrix. (break?) we set the notation
`∏ (a,b : n) = (ab)^{n / 2}` if `n` is even and `∏ (a,b : n) = (ab)^{(n-1) / 2}(a)`
--is there an extra `a` here?  
if `n` is odd, where `∏` is the product, versus `Π` which is a Pi type. -- maybe
say it's a clarification on notation? --def or checkable equality? -- power notation?
right of the equality is a word of the free group. ∏ is a word? doesn't do anything
for the matrix implementation. what's the size of the matrix.

In the implementation below, `m` is of type `S → S → ℕ` where each `S` is an index
into the matrix. So for `s,t` the indices, `m s s = 1`, `m s t ≥ 2`. -- could
be implied in the `S → S → ℕ`?
-- don't know how big S is. how does S relate to F?

The group `A` is called an Artin-Tits group if it is presented by -- types first then
predicate: `for...` first then
`A = ⟨ F | ∏ (s,t : n₁) = ∏ (t,s : n₂) for s,t ∈ F, s ≠ t (unnecessary? may be a
red flag because seems unnecessary?), and n₁,n₂ entries`
`of the matrix (which matrix?), n₁,n₂ ≠ ∞ (unnecessary since matrix cannot contain
infinity anyways)⟩`. explain notation ⟨ generators | relations ⟩. can just say
entries are two or larger? which entries are `n₁, n₂`: they are the entries of (s,t)
and (t,s). notice constraint `n1 n2` are the same, so
the matrix is symmetric.
This is saying `s * t * s * t ... = t * s * t * s ...` where the number of `s`
and `t`'s depend on the matrix entry. The length of either side of the equation
are the same. -- n₁ n₂ should be the same. Note that the indices into the matrix and the generators themselves
are the same. That is because the generators are implemented as the numbers
`0, 1, 2, ...`. For example, in a 3-braid group, the generators are `0, 1` and
the matrix looks like
`[ 1 3 ]`
`[ 3 1 ]`  
where `(0,0) = 1`, `(0,1) = 3`, `(1,0) = 3`, and `(1,1) = 1`. `()` not introduced
as index, in fact, index is introduced above as ∏ (s,t : n₁).
go hard on the example: F is a set {0,1}, the relations are..., the matrix is ...
given matrix this over finite set that, write out presentation.

The braid group is a special case of the Artin-Tits group where the entry in the
matrix is `3` if the indices are next to each other and `2` if they are not.
Diagonal entries are `1`.
-/

/- --------------------------- Artin-Tits group ----------------------------- -/

/--
Construct a word `x` of the free group on `S`, denoted `x : free_group S`, that is
the alternating sequence `s * t * s * t ... * 1 ` of some length `n : ℕ`.

Input:
`S : Type*` the type of the index into the Coxeter matrix. `Type*` is an arbitrary
type. For braid groups, the type is `fin (n-1)` (described in `braid_group`).
`N` the length of the word
`S` the row (?) index into the Coxeter matrix -- (?) say what this means, also a todo list
`S` the col (?) index into the Coxeter matrix

Output:
`x : free_group S` a word of the free group on the type `S`.

Details:

- `ℕ` includes `0`.
- The `1` at the end is ignored and is only used to build the initial sequence of
  length `0` (?) as a base case for this recursive function.
- The method `free_group.of` maps a type `S` to the free group on `S`. So
  `free_group.of s` is an element (word) of the free group on `S`.
- (This function might be building the number of `s`'s and `t`'s to be equal to the entry
  `n` rather than `n/2` if it's even and `(n-1)/2` if it's odd. (?))
- `0 _ _ ` is a 0-length word. The `_ _` are implicit variables of type `S` (?).
- `(n+1) s t := ...` constructs the `free_group.of s * alt n t s` if
  the input is greater than `0` (?). This is recursively constructing the sequence.  
  -/
  -- This code was written with the help of @Kyle Miller.
  ```
  def alt {S : Type*} : ℕ → S → S → free*group S
  | 0 * \_ := 1
  | (n+1) s t := free_group.of s * alt n t s
  ```

/--
Construct the Artin-Tits relation.

Input:
`S : Type*` the type of the index into the Coxeter matrix. `Type*` is an
arbitrary type.
`m : S → S → ℕ` the Coxeter matrix. The term `m s t` takes in the indices
`s t` and outputs the entry of the matrix.

Output:
`set (free_group S)` the set containing the Artin-Tits relation `t * s * ... = s * t * ...`
where both sequences are the same length.

Details:

- We use `finset` for finite sets and `set` for everything else.
  -/
  -- This code was written with the help of @Kyle Miller.
  ```
  def artin_tits_rels (S : Type*) (m : S → S → ℕ) :
  set (free_group S) :=
  {g | ∃ s t, g = (alt (m s t) s t)⁻¹ * (alt (m t s) t s)}
  ```

/--
Construct the Artin-Tits groups. Traditionally we require that
`∀ s t, m s t = m t s` but the hypothesis isn't used in the definition.

Input:
`S : Type*` the type of the index into the Coxeter matrix. `Type*` is an
arbitrary type.
`m : S → S → ℕ` the Coxeter matrix. The term `m s t` takes in the indices
`s t` and outputs the entry of the matrix.

Output:
`presented_group (artin_tits_rels S m)` the presented group with the relations
`artin_tits_rels`. --output is a value, not a type. this is fine

Details:

- `abbreviation` is used to make the definitions reducible so Lean can see
  that they're groups (\_\*\*may need to elaborate).
  -/
  -- This code was written with the help of @Kyle Miller.
  ```
  abbreviation artin_tits (S : Type_) (m : S → S → ℕ) :=
  presented_group (artin_tits_rels S m)
  ```

/- --------------------------- Braid group ----------------------------- -/
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
  -- This code was written with the help of @Kyle Miller.
  -/
  ```
  abbreviation braid_group (n : ℕ) :=
  artin_tits (fin (n - 1))
  (λ i j, if abs ((i : ℤ) - (j : ℤ)) = 1 then 3 else 2)
  ```

reference: @Kyle Miller (possibly reached at <a href="https://leanprover.zulipchat.com/">lean forum</a>).
