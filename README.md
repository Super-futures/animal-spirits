# Animal Spirits

*A real-time interface for reading the constitutive dynamics of collective economic affect.*

---

## What it is

Animal Spirits observes the expressive and material dimensions of collective economic life across three regions simultaneously — US/Europe, UK, and India. It does not predict markets. It does not aggregate sentiment scores. It reads the **coupling** between distributed addressed expression and realised economic behaviour, and surfaces the regime that emerges from that coupling.

The project takes seriously Keynes' choice of the word *animal* in "animal spirits." Not *human* spirits — *animal*: the animate substrate of collective life, prior to and underneath the specific configurations that get called rational or irrational. Markets are not contaminated by affect; they are constituted through it. Animal Spirits makes that constitutive process visible.

---

## Theoretical grounding

The model draws on four intellectual lineages, each doing specific work that the others cannot:

**Keynes (1936)** — named the affective substrate of economic decision-making as an irreducible constitutive condition, not a residual error. In the *General Theory*, animal spirits are "a spontaneous urge to action rather than inaction" — not the irrational residual in otherwise rational markets, but the condition that makes action under genuine uncertainty possible at all. Without them, expected-utility calculation under radical uncertainty cannot generate decision. The animal spirits are what markets run on, not what contaminates them.

**Bakhtin** — all utterance is addressed. The dialogic structure of language anticipates and is shaped by the response of an other, and this structure is operative whether or not a co-present interlocutor exists. News cycles, market commentary, financial media, and social platforms instantiate addressed expression at scale: every story is shaped by an imagined reception, every price signal is read against anticipated response. Collective economic affect is not merely reflected in distributed expression — it is constituted through it. The other is baked into the language.

**Collins (2004)** — interaction ritual chain theory specifies the mechanism by which emotional energy is produced in successful communicative encounters, accumulated through ritual chains, and discharged into action. Markets are ritual chains at scale: attention, narrative, and market behaviour form a chain through which affect is deposited, held, and discharged. Stag is collective failed ritual — high expressive activity, no integration, no accumulation. The spring does not load.

**DeLanda (2016)** — assemblage theory. Every human assemblage has material and expressive components. Neither reduces to the other; relations between assemblages are external, not internal. The economic affect-narrative-market system is a cybernetic assemblage: expressive components (attention, narrative) coupled to material components (market behaviour), with regime states as emergent attractors and phase transitions as deterritorialisation events. No scale is privileged — individuals and collectives are equally real assemblages constituted through the same mechanism at different resolutions.

Together these lineages support a single claim: **collective economic affect is constituted through distributed addressed expression, and the conditions of that constitution are readable from the expressive and material infrastructure that produces it.** Contagion is not transmission — it is constitutive cascade, the dialogic field reorganising. Stag is not a third regime alongside bull and bear — it is the condition in which the constitutive process is active but failing to integrate.

---

## Architecture

The codebase is organised into six explicitly separated layers:

```
Layer 0 — Configuration      (constants, weights, attractor centres)
Layer 1 — Signal Acquisition  (raw signal accessors — swappable)
Layer 2 — Signal Processing   (RegionProcessor — all coupling computation)
Layer 3 — Synthesis           (state cache, synthetic fallback)
Layer 4 — Rendering           (map, coupling planes, panels — reads state only)
Layer 5 — Data Acquisition    (single fetch — state.json via GitHub Pages)
Layer 6 — Initialisation      (boot, tick, resize)
```

**To change data sources:** replace Layer 1 accessors only. Processing and rendering layers are untouched. This is the v3 pathway for Google Trends Alpha, richer sentiment pipelines, and vectorised attention.

---

## The model

### Three axes

**Attention (A)** — expressive, compositional. Computed by the backend from Wikimedia pageviews across four affect cluster terms (anxiety, confidence, aspiration, constraint). Enters the expressive field as a scalar; compositional structure is not currently exposed by the backend API. Vectorised A is a v3 direction.

**Market (M)** — material, responsive. Computed by the backend from Yahoo Finance indices (via Alpha Vantage) and FRED stress indicators. The realised behavioural discharge of collectively constituted affect — the collapse of the expressive field into decision.

**Narrative (N)** — expressive, dynamic. Computed by the backend from GDELT TimelineTone per region. Tone captures the valence of propagating expression — stress or relief moving through the media ritual chain; velocity (ΔN) captures activation, the rate at which that expression is propagating. Narrative is not a peer to attention; it is the channel through which affect propagates and deposits.

### Expressive field

```
E = w_A·A_z + w_N·N_z + w_V·ΔN_z
```

where all components are z-scored over the same rolling window W. E is a scalar — it compresses internal structure. **Expressive divergence** (displayed as *expr. div.*) exposes this compression:

```
expr_div = mean(|A_z − N_z|, |A_z − ΔN_z|, |N_z − ΔN_z|)
```

High expr. div. with stable E indicates an internally tense expressive field: the components are diverging even though the net field is stable. Displayed in panels and affects bloom diffusion on the map. Does not yet enter coupling computation — vectorised E is a v3 direction.

### Coupling metrics

Three metrics characterise the relationship between the expressive field (E) and the material field (M_z):

**(a) Alignment (C_align)** — magnitude-weighted proximity.

```
rawProximity = 1 − |E − M_z| / 2
magWeight    = sigmoid((|E| + |M_z|) · 1.5 − 1.5)
C_align      = rawProximity · magWeight + 0.5 · (1 − magWeight)
```

When both signals are near zero, alignment is suppressed toward neutral (≈0.5) rather than reading as falsely well-aligned. High alignment requires both signals to be meaningfully elevated and proximate.

**(b) Synchrony (C_sync)** — co-movement of changes.

```
C_sync = Pearson(ΔE_{t−W:t}, ΔM_{t−W:t})
```

Named *synchrony*, not feedback: this metric is symmetric. It measures whether expressive and material changes co-move, not which is driving which. Positive synchrony = expansion co-movement. Negative synchrony = contraction co-movement. Directional coupling is a v3 direction.

**(c) Lag (C_lag)** — confidence-weighted phase offset.

```
bestTau, bestR = argmax_τ Pearson(E_{t−τ}, M_t)  for τ ∈ [−τ_max, τ_max]
C_lag          = (|bestTau| / τ_max) · max(0, bestR)
```

C_lag encodes *confident phase offset*: near-zero when the best-fit lag is weakly corroborated, non-zero only when a phase offset is both large and well-supported. Lag sign is preserved separately (negative = E leads, positive = M leads) and encoded visually as ring colour on the coupling planes.

### Ψ — rendering instrument

```
Ψ = 0.5·C_align + 0.35·max(0, C_sync) − 0.15·C_lag
```

Ψ is a rendering parameter, not a descriptive summary. It drives the spatial displacement of attention and market blobs on the map. The primary state representation is the position vector **(C_align, C_sync, C_lag, I)**. Ψ is not displayed in panels.

### Instability

```
I = clip((1 − C_align) + (1 − |C_sync|) + C_lag, 0, 1) / 3
```

Instability is deterritorialisation: breakdown of coupling between expressive and material fields simultaneously across all three dimensions. It is not a third regime — it is a condition of the assemblage, present in any regime to varying degrees.

### Regime

Regime is assigned by proximity to named attractors in **(C_align, C_sync)** space:

| Attractor | C_align | C_sync | Reading |
|-----------|---------|--------|---------|
| Expansion | 0.75 | +0.55 | Fields reinforcing — high alignment, positive co-movement |
| Contraction | 0.75 | −0.55 | Fields contracting — high alignment, negative co-movement |
| Instability | 0.25 | 0.00 | Fields decoupled — low alignment, weak synchrony |

The label is assigned to the nearest attractor. Visual position in coupling space and computed regime label speak the same language — both are continuous-space readings, not threshold conditions.

---

## Visual grammar

**Attention bloom** — warm amber, radial gradient. Radius encodes intensity. Outer ring encodes C_lag magnitude; ring colour encodes lag direction (amber = E leading, blue = M leading).

**Market rectangle** — cool blue, rounded rectangle. Size encodes M_z magnitude.

**Narrative arrows** — purple, directional. Direction encodes propagation direction; number and length encode volume and velocity.

**A↔M offset** — dashed line between bloom and rectangle. Presence and dash pattern encode decoupling.

**Instability ring** — amber dashed ring around the region. Appears when I > 0.45.

**Expressive divergence halo** — subtle dashed halo on attention bloom. Appears when internal expressive components (A, N, ΔN) diverge significantly.

**Coupling plane** — 2D canvas showing position in (C_align × C_sync) space. Ring around each dot encodes C_lag. Three named attractor regions softly shaded. Inset on map (all regions) and per-region below panels.

**Regime tilt strip** — horizontal gradient in controls. Cursor driven by mean C_sync across regions.

---

## Data sources

| Axis | Source | Method | Status |
|------|--------|--------|--------|
| Market (M) | Yahoo Finance (via Alpha Vantage + FRED) | Backend — GitHub Actions | Live |
| Attention (A) | Wikimedia Pageviews API | Backend — 4 cluster × 3 region terms | Live |
| Narrative (N) | GDELT TimelineTone | Backend — per-region tone scalar | Live |

Live status indicated by A● M● N● badge in the header. When a source is unavailable the corresponding signal falls back to the synthetic model.

**Backend:** GitHub Actions workflow in `Super-futures/animal-spirits-api`. Runs on a schedule, computes and normalises all three axes, and writes `data/state.json` to the repo. The API repo has GitHub Pages enabled, serving `state.json` at `https://super-futures.github.io/animal-spirits-api/data/state.json` with permissive CORS headers. The frontend makes a single cache-busted fetch to this URL — all signal processing happens in the workflow.

---

## Known limitations and v3 directions

**Scalar attention (current):** A enters E as a scalar (RMS of four clusters). The compositional structure of the affect field does not enter coupling computation. Expressive divergence (expr. div.) partially exposes this tension.

**v3 direction — vectorised attention:** A enters E as a 2D vector (valence × accumulation), with the four clusters as named quadrants. Each cluster carries its own coupling signature to the market field. Requires Google Trends Alpha or a richer sentiment pipeline via institutional credentials.

**Symmetric synchrony (current):** C_sync is symmetric co-movement. It does not distinguish which field is driving which.

**v3 direction — directional coupling:** Granger causality or lagged regression over longer windows.

**Scalar E (current):** Distinct expressive configurations can produce identical E values. Expressive divergence (expr. div.) partially exposes this.

**v3 direction — vectorised E:** E as a 2D or 3D vector preserving internal structure through to coupling computation. Regime dynamics become cluster-aware.

---

## Deployment

- **Frontend:** `super-futures.github.io/animalspirits` (GitHub Pages — `Super-futures/animalspirits`)
- **Backend:** `super-futures.github.io/animal-spirits-api/data/state.json` (GitHub Pages — `Super-futures/animal-spirits-api`)
- **Stack:** Vanilla HTML/JS/D3/Canvas · Python/GitHub Actions · GitHub Pages

---

## Version history

| Version | Description |
|---------|-------------|
| v0.2 | Four affect clusters, three regions, simulated data, temporal layering per cluster |
| v0.8 | Three axes (A/M/N), three regions, live data, regime probabilities from Ψ |
| v1.0 | Signal processing layer, regime dynamics, global headline |
| v1.1 | Legibility pass — posture vocabulary, axis key, tightened panels |
| v2.0 | Coupling-based architecture — expressive/material separation, C_align/C_sync/C_lag, attractor-space regime, six-layer clean separation |

---

*Superfutures · v2.0*
