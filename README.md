# lau-lie-algebra

Lie algebras are the infinitesimal generators of continuous symmetry. Structure constants, the Killing form, root systems, Dynkin diagrams — this crate makes them concrete and computable.

The symmetries a hermit crab carries — bilateral, rotational — are Lie groups in disguise. Their Lie algebras are the tangent vectors at the identity: the "velocities" of symmetry.

## The math in 60 seconds

A **Lie algebra** 𝔤 is a vector space with an antisymmetric bracket [X,Y] = -[Y,X] satisfying the Jacobi identity [X,[Y,Z]] + [Y,[Z,X]] + [Z,[X,Y]] = 0. Structure constants cᵢⱼᵏ encode everything: [eᵢ, eⱼ] = cᵢⱼᵏeₖ.

Key results this crate verifies:

- **sl(2):** the fundamental 3-dimensional algebra, [H,E]=2E, [H,F]=-2F, [E,F]=H
- **so(3):** isomorphic to su(2), rotations in 3D
- **Killing form:** B(X,Y) = tr(ad(X)·ad(Y)) — non-degenerate iff semisimple
- **Cartan's criterion:** 𝔤 is semisimple ↔ Killing form is non-degenerate
- **Root systems:** vectors in a Euclidean space encoding the algebra's structure
- **Dynkin diagrams:** graphs classifying all simple Lie algebras (Aₙ, Bₙ, Cₙ, Dₙ, E₆, E₇, E₈, F₄, G₂)
- **BCH formula:** log(exp(X)·exp(Y)) as a series in nested commutators

References: Humphreys, *Introduction to Lie Algebras and Representation Theory* (1972)

## Quick start

```rust
use lau_lie_algebra::{LieAlgebra, StructureConstants, RootSystem, DynkinDiagram};

// Build sl(2) from its defining relations
let sl2 = LieAlgebra::sl2();
assert!(sl2.verify_jacobi());  // Jacobi identity holds

// Structure constants: [eᵢ, eⱼ] = cᵢⱼᵏ eₖ
let c = sl2.structure_constants();

// Killing form B(X,Y) = tr(ad(X)ad(Y))
let killing = sl2.killing_form();
assert!(killing.is_non_degenerate()); // sl(2) is semisimple

// Root system of sl(3) ≅ A₂
let roots = RootSystem::type_a(2);
let dynkin = DynkinDiagram::from_root_system(&roots);
// Two nodes connected by a single edge

// BCH: log(exp(X)·exp(Y)) for small X, Y
let x = vec![0.1, 0.0, 0.0];
let y = vec![0.0, 0.1, 0.0];
let z = sl2.bch(&x, &y, 4); // 4th order approximation
```

## Key types

| Type | What it is |
|------|-----------|
| `LieAlgebra` | A Lie algebra with basis, bracket, and structure constants |
| `StructureConstants` | The cᵢⱼᵏ tensors encoding the bracket on a basis |
| `KillingForm` | The symmetric bilinear form B(X,Y) = tr(ad(X)ad(Y)) |
| `RootSystem` | Vectors in Euclidean space encoding the algebra's weight structure |
| `DynkinDiagram` | A graph classifying a simple Lie algebra's type |
| `ExponentialMap` | exp: 𝔤 → G, mapping algebra elements to group elements |

## Contributing

[Open an issue](https://github.com/SuperInstance/lau-lie-algebra/issues) or PR. Natural extensions:

- Representation theory (weights, highest weight modules)
- Affine Lie algebras and Kac-Moody
- Applications to gauge theory and particle physics
