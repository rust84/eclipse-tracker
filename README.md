# 2026 Total Solar Eclipse Tracker

A single-file, self-contained web page that tracks the **total solar eclipse of
August 12, 2026** from the viewer's current location.

It computes the eclipse circumstances for wherever the user is (via the browser
Geolocation API) and shows:

- a live **countdown** to totality (or to maximum eclipse),
- a **Sun & Moon visualization** at the moment of maximum eclipse,
- a **100% / partial / 0%** verdict on whether the site will observe totality,
- a table of **local event times** (partial begin → totality → partial end).

No build step, no dependencies, no network calls. Open `index.html` in a browser.

---

## Features

| Feature | Description |
|---|---|
| **Location** | Uses `navigator.geolocation`; falls back to manual lat/lon inputs or a city preset selector. |
| **Totality verdict** | Banner + coverage %: 100% (inside the umbra), 0–100% (partial penumbra), or 0% (no eclipse). |
| **Countdown** | Ticks every 0.5 s toward totality start (or maximum eclipse if outside the path). |
| **Sun/Moon view** | SVG drawing both disks at their true relative size, offset to match the max coverage. |
| **Circumstances table** | Partial begin, totality begin/max/end, partial end, and totality duration — all in the user's local timezone. |
| **Live status** | Tracks the eclipse phase in real time on eclipse day (not-yet / partial / TOTALITY NOW / over). |

---

## How it works

### Besselian elements
The page uses NASA/Espanak **Besselian elements** for the 2026 Aug 12 total
eclipse (greatest eclipse 17:45:53 UT, 65.2°N, 25.2°W):

```
t0 = 18.0 TDT
n   x          y          d          l1         l2         mu
0   0.475593   0.771161   14.79667   0.537954   -0.008142  88.74776
1   0.5189288  -0.2301664 -0.012065  0.0000940  0.0000935  15.003093
2   -0.0000773 -0.0001245 -0.000003  -0.0000121 -0.0000121
3   -0.0000088  0.0000037
tan f1 = 0.0046141,  tan f2 = 0.0045911
```

Each element `a` is evaluated as a cubic polynomial
`a = Σ aₙ·tⁿ`, with `t = TDT_hours − 18.0`. A constant `ΔT = 70 s` converts
UT1 → TDT.

### Reduction to the observer
For an observer at geodetic latitude φ and longitude λ (east positive):

```
ψ      = atan(0.99664719 · tan φ)            # geocentric latitude
ρ      = 0.9983271 + 0.0016764·cos2φ − 0.0000035·cos4φ
ρcosψ, ρsinψ

ξ     = ρcosψ · sin(μ − λ)
η     = −ρcosψ·sin d·cos(λ−μ) + ρsinψ·cos d
ζ     = ρcosψ·cos d·cos(λ−μ) + ρsinψ·sin d
```

The observer's distance from the shadow axis in the fundamental plane is
`Δ = √((x−ξ)² + (y−η)²)`.

### Shadow radii and coverage
The penumbra/umbra radii projected to the observer's height above the
fundamental plane:

```
L1 = l1 − ζ·tan f1
L2 = l2 − ζ·tan f2
```

Obscuration (fraction of the Sun's disk covered):

```
obscuration = (L1 − Δ) / (L1 + L2)      when Δ < L1, else 0
```

- **Totality (100%)** when `Δ < −L2` (inside the umbra), or when computed
  coverage is `≥ 99%` — the latter accounts for edge-of-path sensitivity, since
  the ~294 km-wide umbra makes the exact boundary numerically delicate.
- **Partial** when `L2 ≤ Δ < L1`.
- **Nothing** when `Δ ≥ L1`.

### Contact times
`P1`/`P4` (partial begin/end) and `U2`/`U3` (totality begin/end) are found by
scanning the eclipse window (14:30–20:30 UT) for magnitude sign changes and
refining each crossing with bisection.

---

## Usage

1. Open `index.html` in a modern browser.
2. Grant location access (or choose a city / enter coordinates).
3. The page computes and displays everything automatically.

### City presets
- Reykjavík, Iceland
- Ísafjörður, Iceland
- London, UK (partial — outside the umbra)
- Madrid, Spain (edge of the path)
- Segovia, Spain (edge)
- Teruel, Spain (inside the path of totality)

---

## Notes & limitations

- **Edge-of-path accuracy**: Totality at a specific site depends sensitively on
  being inside the ~294 km-wide umbra. The page uses a `≥99%` coverage threshold
  to classify borderline sites (e.g. Madrid/Segovia) as total; results near the
  path edges are approximate.
- **Lunar-limb / solar-radius**: Uses the standard Besselian constants, not the
  higher-precision limb corrections (IQP method). Contact times are accurate to
  roughly a minute.
- **Times are approximate** and computed for sea level.
- The eclipse occurs **Aug 12, 2026** (the page is live on that day).

---

## Accuracy validation

The model was validated against published reference data:

| Location | Model | Reference (Wikipedia) |
|---|---|---|
| London, UK | partial begin 17:17, max 92% | 17:17:19, 91.42% |
| Teruel, Spain | totality 18:30:12–18:31:44 | ~18:31–18:33 (~1:34) |
| Segovia, Spain | partial begin 17:35 | 17:35:47 |
