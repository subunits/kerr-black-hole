# kerr-signal-geometry

Analysis of Kerr black hole geodesic trajectories using signal geometry tools.

Each Boyer-Lindquist coordinate — t(λ), r(λ), θ(λ), φ(λ) — is treated as a scalar time series and fed through two companion tools: the [AVS Ground Tool](https://github.com/subunits/avsp) for augmented vector space analysis, and the [AVS Kähler Extension](https://github.com/subunits/kahler-ts) for Hermitian geometry and persistent homology.

---

## Black hole parameters

| Parameter | Value |
|---|---|
| Spin a/M | 0.7 |
| ISCO radius | 3.3931 M |
| Geodesic energy E | 0.8964 |
| Angular momentum L | 2.5865 |
| Samples | 300 (affine step h = 0.5) |
| Units | G = c = M = 1 |

---

## Files

    src/kerr_isco.py        RK4 Kerr geodesic integrator (Python 3, stdlib only)
    src/avsp_Main.hs        AVS Ground Tool v3 (upstream, unmodified)
    src/kahler_isco.hs      AVS Kähler Extension patched with ISCO signals
    data/isco_all.csv       All four coordinates in one file
    data/isco_r_coord.csv   Radial coordinate r(λ)
    data/isco_t_coord.csv   Coordinate time t(λ)
    data/isco_theta_coord.csv  Polar angle θ(λ)
    data/isco_phi_coord.csv Azimuthal angle φ(λ)
    results/kahler_results.md  Full analysis output and physical interpretation
    black_hole_soft.x3d     X3D scene — softened Kerr black hole (a/M = 0.7)

---

## Requirements

- Python 3 — stdlib only (`math`, `csv`)
- GHC 9.x — `base` only, no Cabal or Stack required

---

## Usage

    # Compile and run AVS ground tool on radial coordinate
    ghc -O src/avsp_Main.hs -o avsp
    LANG=C.UTF-8 ./avsp data/isco_r_coord.csv

    # Compile and run Kähler extension on all four ISCO signals
    ghc -O src/kahler_isco.hs -o kahler_isco
    LANG=C.UTF-8 ./kahler_isco

    # Regenerate trajectory CSVs from the integrator
    python3 src/kerr_isco.py

    # Full pipeline via Makefile
    make all

---

## Pipeline

    kerr_isco.py
        RK4 integration of Kerr geodesic equations
        Boyer-Lindquist coordinates t, r, θ, φ
        Spin a/M = 0.7, ISCO radius r = 3.3931 M
        ↓
    data/*.csv  (300 samples per coordinate)
        ↓
    avsp_Main.hs
        AVS augmentation (lag window 3, derivOrder 1, rollingWin 4)
        kNN search, OLS regression, anomaly detection, Shannon entropy
        Distance matrix (L2 and cosine)
        ↓
    kahler_isco.hs
        Complexification R^n → C^(n/2)
        Hermitian metric H = g + iω
        Kähler condition verification
        Curvature — discrete dω over triples
        Chern proxy — Σ ω(pᵢ, pⱼ)
        Symplectic form matrix
        Holomorphic kNN (Hermitian distance)
        Vietoris-Rips persistent homology

---

## Key results

### Chern proxy — symplectic integral Σ ω(pᵢ, pⱼ)

| Signal | Chern proxy | vs sine (1037.38) | Physical meaning |
|---|---|---|---|
| sine (ref) | 1037.38 | 100% | Pure periodic baseline — ω accumulates without cancellation |
| ISCO-θ | 987.57 | 95% | Near-sinusoidal polar oscillation, monotone phase |
| ISCO-r | 934.59 | 90% | Oscillatory radial perturbation, monotone accumulation |
| ISCO-φ | 512.45 | 49% | Azimuthal advance; 2π wrapping causes partial cancellation |
| lorenz | 436.26 | 42% | Chaotic Lorenz attractor — partial ω cancellation |
| ISCO-t | 23.56 | 2% | Linear coordinate time; near-collinear lag vectors, near-zero ω |

### Curvature dω

All four ISCO coordinates show constant dω ≈ 0.0325 across all triples — uniform to three significant figures. The embedding is uniformly curved (not Lagrangian), consistent with a circular geodesic at fixed r_ISCO in a curved spacetime. For comparison, the Lorenz attractor shows wildly varying dω across triples. The near-constant value here is the Kähler signature of orbital regularity.

### Symplectic matrix (ISCO-r, first 4 points)

    ω(u,v)      t=3      t=4      t=5      t=6
    t=3         0.000   +0.440   +0.870   +1.278
    t=4        -0.440    0.000   +0.440   +0.870
    t=5        -0.870   -0.440    0.000   +0.440
    t=6        -1.278   -0.870   -0.440    0.000

Uniform increment Δω = 0.440 between consecutive steps — the orbital frequency of the ISCO encoded as symplectic phase advance in C^k.

### Persistent homology

H0 drops from 15 to 1 as ε grows — two synthetic clusters merging cleanly. H1 = 0 throughout — no topological loops in the ISCO signal manifold. Entropy rises monotonically from 0.0 to 3.25 bits as the Vietoris-Rips complex fills in.

---

## X3D scene

`black_hole_soft.x3d` is a pure X3D 3.3 scene of the Kerr black hole at spin a/M = 0.7. Open it in [X_ITE](https://create3000.github.io/x_ite/) or Instant Reality. The scene uses `<Torus>` geometry for volumetric accretion disk rings with differential rotation driven by `<TimeSensor>` and `<OrientationInterpolator>` nodes wired via `<ROUTE>`.

---

## Extending

To analyse other orbital regimes, edit `src/kerr_isco.py`:

- Plunging orbit: set r < r_isco, remove the eps_r stabilisation
- Eccentric orbit: initialise with non-circular E and L values
- Different spin: change a = 0.7 — r_ISCO is computed automatically
- Photon sphere: set r = 1.5 * r_s (approximate)

Feed any resulting CSV into avsp directly: `./avsp my_signal.csv`

---

## Upstream tools

- avsp: [github.com/subunits/avsp](https://github.com/subunits/avsp) — AVS Ground Tool v3. `avsp_Main.hs` is unmodified upstream source.
- kahler-ts: [github.com/subunits/kahler-ts](https://github.com/subunits/kahler-ts) — Kähler extension. `kahler_isco.hs` patches the signal corpus with the four ISCO coordinates.

---

## Licence

Public domain — Unlicense. No warranty expressed or implied.
