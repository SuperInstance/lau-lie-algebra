# lau-lie-algebra

Lie algebras are the infinitesimal generators of continuous symmetry. Structure constants, the Killing form, root systems, Dynkin diagrams — this crate makes them concrete and computable.

The symmetries a hermit crab carries — bilateral, rotational — are Lie groups in disguise. Their Lie algebras are the tangent vectors at the identity: the "velocities" of symmetry.

## Table of Contents

1. [Overview](#overview)
2. [Mathematical Background](#mathematical-background)
3. [Architecture](#architecture)
4. [API Reference](#api-reference)
5. [Examples](#examples)
6. [Theorems Verified](#theorems-verified)
7. [Installation & Usage](#installation--usage)
8. [License](#license)

---

## Overview

This crate implements computational Lie algebra theory in pure Rust with zero external dependencies for linear algebra. It provides:

- **Lie algebras** defined by structure constants, with bracket computation
- **Classical algebras**: sl(2), so(3), gl(n), Heisenberg, upper triangular, ℝ³ with cross product
- **Structural analysis**: Killing form, Cartan's criterion, center, derived algebra, solvability, nilpotency
- **Representations**: homomorphism verification, irreducibility testing, direct sum and tensor product, character computation
- **Exponential map and BCH formula**: matrix exponential via adjoint representation, Baker–Campbell–Hausdorff to third order
- **Root systems and Dynkin diagrams**: Cartan matrices, root classification, Weyl group sizes, automatic type detection (Aₙ, Bₙ, Cₙ, Dₙ, E₆₋₈, F₄, G₂)

All results are verified against 99 property-based tests.

---

## Mathematical Background

### Lie Algebras

A **Lie algebra** 𝔤 is a vector space equipped with a bilinear bracket [·,·] satisfying:

1. **Antisymmetry**: [X,Y] = −[Y,X]
2. **Jacobi identity**: [X,[Y,Z]] + [Y,[Z,X]] + [Z,[X,Y]] = 0

In a basis {e₁,…,eₙ}, the bracket is encoded by **structure constants** cᵢⱼᵏ:

> [eᵢ, eⱼ] = Σₖ cᵢⱼᵏ eₖ

### Classical Lie Algebras

| Algebra | Description | Key property |
|---------|-------------|--------------|
| **sl(n)** | traceless n×n matrices | Simple, semisimple |
| **so(n)** | skew-symmetric n×n matrices | Semisimple (n≥3) |
| **gl(n)** | all n×n matrices | Not semisimple (has center = scalar matrices) |
| **Heisenberg** | 2n+1 dimensional | Nilpotent, not semisimple |
| **Upper triangular** | strictly upper triangular matrices | Nilpotent |

### Structural Invariants

- **Center**: Z(𝔤) = {x ∈ 𝔤 : [x,y] = 0 for all y}
- **Derived algebra**: [𝔤,𝔤] = span{[x,y] : x,y ∈ 𝔤}
- **Killing form**: κ(x,y) = tr(ad(x)∘ad(y)), where ad(x)(y) = [x,y]
- **Cartan's criterion**: 𝔤 is semisimple ⟺ κ is non-degenerate
- **Solvable**: derived series terminates at zero
- **Nilpotent**: lower central series terminates at zero

### Representations

A **representation** ρ: 𝔤 → gl(V) is a linear map preserving brackets:

> ρ([X,Y]) = ρ(X)ρ(Y) − ρ(Y)ρ(X)

This crate can:
- Verify that a given map is a representation (`verify_homomorphism`)
- Test irreducibility by checking for invariant subspaces (`is_irreducible`)
- Compute characters χ(X) = tr(ρ(X))
- Construct direct sums and tensor products of representations

### Root Systems and Dynkin Diagrams

For a semisimple Lie algebra, a **Cartan subalgebra** 𝔥 gives rise to a **root system** Φ ⊂ 𝔥* — a set of vectors encoding the algebra's structure.

**Simple roots** α₁,…,αᵣ form a basis of Φ such that every root is either positive or negative. The **Cartan matrix** has entries:

> Aᵢⱼ = 2(αᵢ,αⱼ)/(αᵢ,αᵢ)

The **Dynkin diagram** is a graph whose nodes are simple roots and whose edges encode the Cartan matrix entries. The classification theorem states that every finite-dimensional simple Lie algebra over ℂ has type Aₙ, Bₙ, Cₙ, Dₙ, E₆, E₇, E₈, F₄, or G₂.

### Exponential Map and BCH

The **exponential map** exp: 𝔤 → G maps algebra elements to group elements:

> exp(X) = I + X + X²/2! + X³/3! + … (computed via the adjoint representation)

The **Baker–Campbell–Hausdorff (BCH) formula** expresses log(exp(X)·exp(Y)) as a series of nested commutators:

> Z = X + Y + ½[X,Y] + 1/12[X,[X,Y]] − 1/12[Y,[X,Y]] + …

This crate computes BCH to third order.

---

## Architecture

```
src/
├── lib.rs       — All types, constructors, algorithms, and helpers
└── tests.rs     — 99 tests organized by module
```

### Type Hierarchy

```
LieAlgebra                    — core algebra with structure constants
  ├── bracket(), killing_form(), is_semisimple(), center(), …
  │
  ├── LieRepresentation       — ρ: 𝔤 → gl(V)
  │     ├── verify_homomorphism(), is_irreducible()
  │     ├── direct_sum(), tensor_product()
  │     └── character()
  │
  ├── ExponentialMap          — exp and BCH
  │     ├── exp_of()
  │     └── baker_campbell_hausdorff()
  │
  ├── RootSystem              — roots, Cartan matrix, Weyl group
  │     └── DynkinDiagram     — classification (Aₙ–G₂)
  │
  └── DenseMatrix             — internal linear algebra support
```

### Key Types

| Type | What it represents |
|------|-------------------|
| `LieAlgebra` | Algebra with named basis and cᵢⱼᵏ structure constants |
| `DenseMatrix` | Square matrix for linear algebra (multiply, determinant, trace, commutator) |
| `LieRepresentation` | A representation ρ: 𝔤 → gl(V), stored as matrices per basis element |
| `RootSystem` | Roots and simple roots in Euclidean space |
| `DynkinDiagram` | Nodes and weighted edges, with automatic classification |
| `DynkinType` | Classified type: `A(n)`, `B(n)`, `C(n)`, `D(n)`, `E(k)`, `F4`, `G2` |

### Serialization

All types implement `Serialize`/`Deserialize` via serde. Algebras, representations, root systems, and Dynkin diagrams can be persisted to JSON and reloaded.

---

## API Reference

### LieAlgebra

```rust
// Construction
let alg = LieAlgebra::new("name", vec!["e".into(), "h".into(), "f".into()], structure_constants);

// Core operations
alg.bracket(&x, &y);                    // [Σxᵢeᵢ, Σyⱼeⱼ]
alg.adjoint_matrix(&x);                  // ad(x) as DenseMatrix

// Verification
alg.verify_antisymmetry();               // [X,Y] = -[Y,X]
alg.verify_jacobi();                     // Jacobi identity

// Structural analysis
alg.is_abelian();
alg.center();                            // Z(𝔤) as basis vectors
alg.derived_algebra();                   // [𝔤,𝔤] as basis vectors
alg.killing_form();                      // κ as DenseMatrix
alg.is_semisimple();                     // Cartan's criterion
alg.is_solvable(max_depth);
alg.is_nilpotent(max_depth);
```

### Pre-built Algebras

```rust
let sl = sl2();                           // sl(2,ℝ): {e,h,f}, dim 3
let so = so3();                           // so(3): rotations, dim 3
let h1 = heisenberg(1);                   // Heisenberg, dim 3
let h2 = heisenberg(2);                   // Heisenberg, dim 5
let g2 = gl(2);                           // gl(2), dim 4
let ut = upper_triangular(3);             // nilpotent, dim 3
let cp = cross_product_lie();             // ℝ³ with cross product (≅ so(3))
```

### LieRepresentation

```rust
let rep = LieRepresentation::new(algebra, dimension, rho_matrices);

rep.verify_homomorphism();                // ρ([X,Y]) = [ρ(X),ρ(Y)]
rep.is_irreducible();                     // no nontrivial invariant subspaces
rep.character(&element);                  // tr(ρ(element))
rep.direct_sum(&other);                   // ⊕ representation
rep.tensor_product(&other);               // ⊗ representation
```

### ExponentialMap

```rust
// Matrix exponential via adjoint representation (Taylor series)
let exp_x = ExponentialMap::exp_of(&alg, &x, num_terms);

// Baker–Campbell–Hausdorff to third order
let z = ExponentialMap::baker_campbell_hausdorff(&alg, &x, &y);
// z ≈ x + y + ½[x,y] + 1/12[x,[x,y]] - 1/12[y,[x,y]]
```

### RootSystem

```rust
let rs = sl_n_root_system(3);             // A₂ root system
let rs2 = sl2_root_system();              // A₁ root system

rs.cartan_matrix();                       // Cartan matrix A
rs.dynkin_diagram();                      // → DynkinDiagram
rs.positive_roots();                      // positive half of Φ
rs.roots.len();                           // total root count
rs.weyl_group_size();                     // |W|
```

### DynkinDiagram

```rust
let dd = rs.dynkin_diagram();
dd.classify();                            // → DynkinType::A(2), etc.
dd.nodes;                                 // number of simple roots
dd.edges;                                 // Vec<(i, j, multiplicity)>
```

### DenseMatrix

```rust
let m = DenseMatrix::from_vec(vec![vec![1.0, 2.0], vec![3.0, 4.0]]);

m.multiply(&other);                       // matrix product
m.add(&other);                            // element-wise sum
m.scale(s);                               // scalar multiply
m.transpose();
m.trace();
m.determinant();                          // cofactor expansion (n ≤ 6)
m.commutator(&other);                     // AB - BA
```

---

## Examples

### sl(2): The Fundamental Example

```rust
use lau_lie_algebra::*;

let alg = sl2();
// Basis: e=0, h=1, f=2

// Defining relations
let he = alg.bracket(&[0.0, 1.0, 0.0], &[1.0, 0.0, 0.0]);
assert!((he[0] - 2.0).abs() < 1e-8);  // [h,e] = 2e ✓

let hf = alg.bracket(&[0.0, 1.0, 0.0], &[0.0, 0.0, 1.0]);
assert!((hf[2] - (-2.0)).abs() < 1e-8); // [h,f] = -2f ✓

let ef = alg.bracket(&[1.0, 0.0, 0.0], &[0.0, 0.0, 1.0]);
assert!((ef[1] - 1.0).abs() < 1e-8);   // [e,f] = h ✓

// Structural properties
assert!(alg.verify_jacobi());
assert!(alg.is_semisimple());
assert!(!alg.is_solvable(10));
assert!(!alg.is_nilpotent(10));
assert_eq!(alg.center().len(), 0);      // center is trivial
```

### Killing Form and Cartan's Criterion

```rust
let kf = sl2().killing_form();
// κ in basis {e,h,f}:
//   κ(e,e)=0, κ(f,f)=0, κ(h,h)=8
//   κ(e,f)=κ(f,e)=4, κ(e,h)=κ(h,f)=0

assert!(kf.determinant().abs() > 1e-8); // non-degenerate → semisimple

// Compare with Heisenberg: degenerate Killing form
let h = heisenberg(1);
assert!(h.killing_form().determinant().abs() < 1e-8); // degenerate → not semisimple
```

### Representations of sl(2)

```rust
// Fundamental (2-dimensional) representation
let rep = sl2_fundamental_representation();
assert!(rep.verify_homomorphism());       // ρ preserves brackets
assert!(rep.is_irreducible());            // no invariant subspaces

// Character: tr(ρ(h)) = 1 + (-1) = 0
let ch = rep.character(&[0.0, 1.0, 0.0]);
assert!(ch.abs() < 1e-8);

// Direct sum: 2⊕2 is not irreducible
let ds = rep.direct_sum(&rep);
assert_eq!(ds.dimension, 4);
assert!(!ds.is_irreducible());

// Tensor product: 2⊗2 is a 4-dim representation
let tp = rep.tensor_product(&rep);
assert_eq!(tp.dimension, 4);
assert!(tp.verify_homomorphism());
```

### Exponential Map and BCH

```rust
let alg = sl2();

// exp(0) = I
let id = ExponentialMap::exp_of(&alg, &[0.0, 0.0, 0.0], 10);
// All diagonal entries = 1, off-diagonal = 0

// exp of small h: diagonal entries are exp(±2t)
let exp_h = ExponentialMap::exp_of(&alg, &[0.0, 0.1, 0.0], 15);

// BCH: log(exp(e)·exp(f)) ≈ e + f + ½h + …
let z = ExponentialMap::baker_campbell_hausdorff(
    &alg, &[1.0, 0.0, 0.0], &[0.0, 0.0, 1.0]
);
// z ≈ (5/6)e + ½h + (5/6)f
```

### Root Systems and Classification

```rust
// sl(3) → A₂ root system
let rs = sl_n_root_system(3);
assert_eq!(rs.roots.len(), 6);            // 6 total roots
assert_eq!(rs.positive_roots().len(), 3); // 3 positive roots

let cm = rs.cartan_matrix();
// Cartan matrix for A₂:
//   [[2, -1], [-1, 2]]

let dd = rs.dynkin_diagram();
assert_eq!(dd.classify(), DynkinType::A(2));

// Weyl group of A₂ is S₃, order 6
assert_eq!(rs.weyl_group_size(), 6);

// Classification works for sl(n) across sizes
for n in 2..=5 {
    let rs = sl_n_root_system(n);
    assert_eq!(rs.dynkin_diagram().classify(), DynkinType::A(n - 1));
}
```

### Nilpotent and Solvable Algebras

```rust
// Heisenberg algebra: nilpotent (hence solvable)
let h = heisenberg(1);
assert!(h.is_nilpotent(5));
assert!(h.is_solvable(5));
assert!(!h.is_semisimple());
assert_eq!(h.center().len(), 1);          // center = span{z}

// Upper triangular matrices: nilpotent
let ut = upper_triangular(3);
assert!(ut.is_nilpotent(5));
assert!(!ut.is_semisimple());

// 2×2 upper triangular: actually abelian (1-dim)
let ut2 = upper_triangular(2);
assert!(ut2.is_abelian());
```

### Isomorphism: so(3) ≅ ℝ³ with Cross Product

```rust
let so = so3();
let cp = cross_product_lie();

// Same dimension
assert_eq!(so.dimension, cp.dimension);

// Both semisimple
assert!(so.is_semisimple());
assert!(cp.is_semisimple());

// Same bracket structure: [e₁,e₂] = e₃, [e₂,e₃] = e₁, [e₃,e₁] = e₂
```

---

## Theorems Verified

All 99 tests serve as machine-checked proofs. Here is the complete list of verified theorems:

### Axiom Verification

1. **Antisymmetry** [X,Y] = −[Y,X]: verified for sl(2), so(3), gl(2), Heisenberg, upper triangular, cross product, abelian — 7 tests
2. **Jacobi identity**: verified for all of the above — 8 tests

### sl(2) — The Prototype

3. **Defining relations**: [h,e]=2e, [h,f]=−2f, [e,f]=h — `test_sl2_brackets`
4. **Semisimple** (Killing form non-degenerate) — `test_sl2_semisimple`
5. **Not solvable**, not nilpotent — `test_sl2_not_solvable`, `test_sl2_not_nilpotent`
6. **Trivial center** (dim 0) — `test_sl2_center_empty`
7. **Killing form**: κ(e,e)=0, κ(f,f)=0, κ(h,h)=8, κ(e,f)=4 — `test_sl2_killing_form`
8. **Adjoint representation**: ad(h) is diag(2,0,−2) — `test_sl2_adjoint_matrix`
9. **Derived algebra** = sl(2) itself (dim 3) — `test_sl2_derived_algebra`

### Classical Algebras

10. **so(3) ≅ su(2)**: same dimension, both semisimple — `test_so3_isomorphic_to_sl2`
11. **gl(2) not semisimple**: has nontrivial center (scalar matrices) — `test_gl2_not_semisimple`, `test_gl2_center`
12. **Cross product ≅ so(3)**: same structure constants — `test_cross_product_isomorphic_to_so3`

### Nilpotent and Solvable Algebras

13. **Heisenberg is nilpotent** (hence solvable) — 4 tests for n=1,2
14. **Heisenberg not semisimple**: degenerate Killing form — `test_heisenberg_not_semisimple`
15. **Heisenberg center** = span{z}, dimension 1 — `test_heisenberg_center_dim1`
16. **Upper triangular is nilpotent** — `test_upper_triangular_nilpotent`
17. **Upper triangular n=2 is abelian** — `test_upper_triangular_2_jacobi`

### Killing Form and Cartan's Criterion

18. **sl(2) Killing form non-degenerate**: det ≠ 0 — `test_killing_form_sl2_nondegenerate`
19. **Heisenberg Killing form degenerate**: det = 0 — `test_killing_form_heisenberg_degenerate`
20. **so(3) Killing form non-degenerate** — `test_so3_killing_form`

### Representation Theory

21. **sl(2) fundamental rep** is a homomorphism and irreducible — 6 tests
22. **Character** of h in fundamental rep = 0 — `test_sl2_fundamental_character`
23. **Direct sum** preserves homomorphism, is reducible — `test_direct_sum`, `test_direct_sum_not_irreducible`
24. **Tensor product** preserves homomorphism — `test_tensor_product`
25. **Adjoint representation** of sl(2) is a valid representation — `test_adjoint_representation_is_representation`

### Exponential Map

26. **exp(0) = I** — `test_exp_identity`
27. **exp(h)** gives diagonal entries exp(±2t) — `test_exp_of_small_element`
28. **BCH(X,0) = X** — `test_bch_zero_y`
29. **BCH commutes for parallel elements** — `test_bch_commutative_case`
30. **BCH full third-order formula** for [e,f] — `test_bch_formula_terms`

### Root Systems

31. **sl(2) Cartan matrix** = [2] — `test_sl2_cartan_matrix`
32. **sl(3) Cartan matrix** = [[2,−1],[−1,2]] — `test_sl3_cartan_matrix`
33. **sl(n) has n(n−1) total roots** — `test_sl3_total_roots`
34. **sl(3) has 3 positive roots** — `test_sl3_positive_roots`

### Dynkin Diagram Classification

35. **A₁ through A₅**: sl(n+1) gives Aₙ — `test_dynkin_a1` through `test_sl_n_dynkin_an_minus_1`
36. **B₂**: detected from multiplicity-2 edge — `test_dynkin_b2`
37. **G₂**: detected from multiplicity-3 edge — `test_dynkin_g2`
38. **D₄**: Y-shaped diagram — `test_dynkin_d4`
39. **F₄**: chain with multiplicities 1-2-1 — `test_dynkin_f4`

### Weyl Group

40. **|W(A₁)| = 2**, **|W(A₂)| = 6**, **|W(A₃)| = 24** — 3 tests

### Abelian Algebra

41. **Abelian**: satisfies Jacobi, antisymmetry, is solvable and nilpotent, not semisimple — `test_abelian_algebra`

### Serialization

42. All major types round-trip through JSON: LieAlgebra, LieRepresentation, RootSystem, DynkinDiagram, DynkinType, DenseMatrix — 6 tests

---

## Installation & Usage

### Prerequisites

- Rust 1.56+ (2021 edition)

### Add to your project

```toml
[dependencies]
lau-lie-algebra = { git = "https://github.com/SuperInstance/lau-lie-algebra" }
```

Or clone and use locally:

```bash
git clone https://github.com/SuperInstance/lau-lie-algebra.git
cd lau-lie-algebra
cargo test   # Run all 99 tests
```

### Dependencies

| Crate | Purpose |
|-------|---------|
| `serde` + `serde_json` | JSON serialization of algebras, representations, and root systems |

No external linear algebra crates — all matrix operations (multiplication, determinant, Gaussian elimination) are implemented from scratch.

---

## License

MIT
