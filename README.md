# Animal Spirits

*A real-time interface for reading the constitutive dynamics of collective economic affect.*

---

## What it is

Animal Spirits observes the expressive and material dimensions of collective economic life across three regions simultaneously — US, UK, and India. It does not predict markets. It does not aggregate sentiment scores. It reads the **coupling** between distributed addressed expression and realised economic behaviour, and surfaces the regime that emerges from that coupling.

The project takes seriously Keynes' choice of the word *animal* in "animal spirits." Not *human* spirits — *animal*: the animate substrate of collective life, prior to and underneath the specific configurations that get called rational or irrational. Markets are not contaminated by affect; they are constituted through it. Animal Spirits makes that constitutive process visible.

---

## Theoretical grounding

The model draws on four intellectual lineages, each doing specific work that the others cannot:

**Keynes (1936)** — named the affective substrate of economic decision-making as an irreducible constitutive condition, not a residual error. In the *General Theory*, animal spirits are "a spontaneous urge to action rather than inaction" — not the irrational residual in otherwise rational markets, but the condition that makes action under genuine uncertainty possible at all. Without them, expected-utility calculation under radical uncertainty cannot generate decision. The animal spirits are what markets run on, not what contaminates them.

**Bakhtin** — all utterance is addressed. The dialogic structure of language anticipates and is shaped by the response of an other, and this structure is operative whether or not a co-present interlocutor exists. News cycles, market commentary, financial media, and social platforms instantiate addressed expression at scale: every story is shaped by an imagined reception, every price signal is read against anticipated response. Collective economic affect is not merely reflected in distributed expression — it is constituted through it. The other is baked into the language.

**Collins (2004)** — interaction ritual chain theory specifies the mechanism by which emotional energy is produced in successful communicative encounters, accumulated through ritual chains, and discharged into action. Markets are ritual chains at scale: attention, narrative, and market behaviour form a chain through which affect is deposited, held, and discharged. Instability is collective failed ritual — high expressive activity, no integration, no accumulation. The spring does not load.

**DeLanda (2016)** — assemblage theory. Every human assemblage has material and expressive components. Neither reduces to the other; relations between assemblages are external, not internal. The economic affect-narrative-market system is a cybernetic assemblage: expressive components (attention, narrative) coupled to material components (market behaviour), with regime states as emergent attractors and phase transitions as deterritorialisation events. No scale is privileged — individuals and collectives are equally real assemblages constituted through the same mechanism at different resolutions.

Together these lineages support a single claim: **collective economic affect is constituted through distributed addressed expression, and the conditions of that constitution are readable from the expressive and material infrastructure that produces it.** Contagion is not transmission — it is constitutive cascade, the dialogic field reorganising. Instability is not a third regime alongside expansion and contraction — it is the condition in which the constitutive process is active but failing to integrate.

---

## Ontological mapping

The three observable axes map cleanly onto DeLanda's material/expressive distinction:

| Component | Axis | Source |
|-----------|------|--------|
| **Expressive** | Attention (A) — compositional | Wikimedia Pageviews (en.wikipedia) |
| **Expressive** | Narrative (N) — dynamic, propagating | GDELT TimelineTone |
| **Material** | Market (M) — realised behaviour | Alpha Vantage ETF proxies + FRED |

Attention and narrative together compose the **expressive field (E)** — the addressed, dialogic substrate through which collective affect circulates. Market is the **material field (M)** — the realised behavioural discharge.

This mapping is not metaphor. It is the operational form of the assemblage: two distinct ontological components, irreducible to each other, coupled through computable relations.

---

## Architecture

The codebase is organised into six explicitly separated layers:

```
Layer 0 — Configuration       (constants, weights, attractor centres)
Layer 1 — Signal Acquisition  (raw signal accessors — swappable)
Layer 2 — Signal Processing   (RegionProcessor — coupling computation)
Layer 3 — Synthesis           (state cache, synthetic fallback)
Layer 4 — Rendering           (map, ternary plot, panels — reads state only)
Layer 5 — Data Acquisition    (single fetch — state.json via GitHub Pages)
Layer 6 — Initialisation      (boot, tick, resize)
```

**To change data sources:** replace Layer 1 accessors only. Processing and rendering layers are untouched. This is the v3 pathway for Google Trends Alpha, richer sentiment pipelines, and vectorised attention.

---

## The model

### The expressive field

The expressive components combine into a single field:

```
E = w_A · A_z  +  w_N · N_z  +  w_V · ΔN_z
```

where all components are z-scored over the same rolling window W. E is a scalar — it compresses internal expressive structure. **Expressive divergence** (displayed in panels as *expr. div.*) exposes this compression:

```
expr_div = mean( |A_z − N_z|, |A_z − ΔN_z|, |N_z − ΔN_z| )
```

High expr. div. with stable E indicates an internally tense expressive field: the components are diverging even though the net field reads stable. This affects bloom diffusion on the map but does not yet enter coupling computation directly — vectorised E is a v3 direction.

### Coupling metrics

Three metrics characterise the relationship between the expressive field (E) and the material field (M_z):

**(a) Alignment (C_align)** — magnitude-weighted proximity of E and M.

```
rawProximity = 1 − |E − M_z| / 2
magWeight    = sigmoid( (|E| + |M_z|) · 1.5 − 1.5 )
C_align      = rawProximity · magWeight  +  0.5 · (1 − magWeight)
```

When both signals are near zero, alignment is suppressed toward neutral (≈0.5) rather than reading as falsely well-aligned. High alignment requires both fields to be meaningfully elevated and proximate.

**(b) Synchrony (C_sync)** — symmetric co-movement of changes in E and M.

```
C_sync = Pearson( ΔE_{t−W:t}, ΔM_{t−W:t} )
```

Named *synchrony*, not feedback: this metric is symmetric. It measures whether expressive and material changes co-move, not which is driving which. Positive synchrony reads as expansion-direction co-movement; negative synchrony reads as contraction-direction co-movement. Directional coupling is a v3 direction.

**(c) Lag (C_lag)** — confidence-weighted phase offset.

```
bestTau, bestR = argmax_τ Pearson( E_{t−τ}, M_t )   for τ ∈ [−τ_max, τ_max]
C_lag          = (|bestTau| / τ_max) · max(0, bestR)
```

C_lag encodes *confident phase offset*: near-zero when the best-fit lag is weakly corroborated, non-zero only when a phase offset is both large and well-supported. Lag sign is preserved separately (negative = E leads, positive = M leads) and encoded visually as ring colour.

### Instability

```
I = min( 1, [ (1 − C_align) + (1 − |C_sync|) + C_lag ] / 3 )
```

Instability is deterritorialisation: breakdown of coupling between expressive and material fields simultaneously across all three dimensions. It is not a third regime alongside expansion and contraction — it is a condition of the assemblage, a measure of how decoupled the components have become.

### Three attractors

Regime is assigned by proximity to named attractors in **(C_align, C_sync)** space:

| Attractor | C_align | C_sync | Reading |
|-----------|---------|--------|---------|
| Expansion | 0.75 | +0.55 | E and M aligned and co-moving in the expansion direction |
| Contraction | 0.75 | −0.55 | E and M aligned and co-moving in the contraction direction |
| Instability | 0.25 | 0.00 | E and M decoupled — neither aligned nor co-moving |

The label is assigned to the nearest attractor. Visual position in coupling space and computed regime label speak the same language — both are continuous-space readings, not threshold conditions.

### Ψ — rendering instrument

```
Ψ = 0.5·C_align + 0.35·max(0, C_sync) − 0.15·C_lag
```

Ψ is a rendering parameter, not a descriptive summary. It drives the spatial displacement of expressive and material blooms on the map. The primary state representation is the position vector **(C_align, C_sync, C_lag, I)**. Ψ is not displayed in panels.

---

## Visual grammar — three complementary layers

Three layers, each operating at a different scale of the assemblage:

### 1. Geographic layer (map)

Spatial expression of the per-region assemblage components in real-world geography.

- **Attention bloom** — warm amber radial gradient. Radius encodes A intensity. Outer ring encodes C_lag magnitude; ring colour encodes lag direction (amber = E leads, blue = M leads).
- **Market rectangle** — cool blue rounded rectangle. Size encodes M_z magnitude.
- **Narrative arrows** — purple, directional. Direction encodes propagation direction; number and length encode volume and velocity.
- **A↔M offset** — dashed line between bloom and rectangle. Presence and dash pattern encode decoupling.
- **Instability ring** — amber dashed ring around the region. Appears when I > 0.45.
- **Expressive divergence halo** — subtle dashed halo on attention bloom. Appears when internal expressive components (A, N, ΔN) diverge significantly.

### 2. Relational layer (ternary attractor plot)

A small ternary plot inset in the bottom-right of the map shows all three regions in **coupling space**. Three vertices, three attractors:

- **Top vertex** — expansion
- **Bottom-right vertex** — contraction
- **Bottom-left vertex** — instability

Each region is plotted by its **proximity to the three attractors**, computed as inverse-square distance weights from its actual (C_align, C_sync) position to each attractor coordinate. Position in the triangle reads directly as attractor proximity.

**Encoding:**
- **Position** — barycentric weight by proximity to each attractor
- **Colour** — region identity (US, UK, India each get a distinct hue)
- **Size** — instability magnitude I
- **Outer ring** — C_lag magnitude with directional colour (blue = M leads, amber = E leads)

**Function:** The ternary plot is the inter-regional assemblage view. Three regions clustered near one vertex = global assemblage synchronized in that regime. Three regions spread across vertices = regional fragmentation, deterritorialisation at the inter-regional scale. Region colour stays constant; position changes — so trajectories are easy to track.

The geometry is the model: three attractors define a triangular coupling space, and each national assemblage occupies a position determined by its actual coupling state. There are no axes to interpret — the named vertices *are* the navigational system.

### 3. Analytical layer (per-region panels)

A three-column grid of per-region panels below the map. Each panel displays:

- **Coupling metrics** — C_align, C_sync (bars with neutral geometry, no regime colour bleed)
- **Lead-lag** — C_lag with confidence weighting
- **Instability** — I magnitude
- **Narrative velocity** — N and ΔN
- **Sparklines** — A, M, N history (rolling window)
- **Cluster composition** — display only, not yet coupled (v3 direction)

Panels provide analytical depth per region; the ternary plot provides relational simultaneity across regions; the map provides geographic and signal expression. No layer duplicates another — each does specific work.

### Controls

**Regime tilt strip** — horizontal gradient (expansion → instability → contraction). Cursor driven by mean C_sync across regions, encoding global synchrony state.

---

## Data sources

| Axis | Source | Method | Granularity | Availability |
|------|--------|--------|-------------|--------------|
| Attention (A) | Wikimedia Pageviews API — `en.wikipedia` | 32 articles per region (8 terms × 4 affect clusters), per-term z-score against own 30-day baseline | daily | 99.9 % |
| Market (M) | Alpha Vantage ETF proxies (`SPY`, `ISF.LON`, `NIFTYBEES.BSE`) blended with FRED macro-stress series | 0.55 × local equity + 0.45 × global stress | daily closes | 100 % |
| Narrative (N) | GDELT DOC 2.0 TimelineTone | one query per region, `sourcecountry` filtered, restricted to economic stress vocabulary | intraday | 50.1 % |

Availability measured across 1,173 refresh events, May–August 2026.

Live status indicated by A● M● N● badge in the header. When a source is unavailable, the corresponding signal falls back to the synthetic model.

**Backend:** GitHub Actions workflow in [`propensities/animal-spirits-api`](https://github.com/propensities/animal-spirits-api). Computes and normalises all three axes and writes `data/state.json`, served via GitHub Pages at `https://propensities.github.io/animal-spirits-api/data/state.json`. The frontend makes a single cache-busted fetch to this URL every 15 minutes — all signal processing happens upstream.

The workflow is scheduled at 15-minute intervals; observed cadence runs closer to a 90-minute median, reflecting scheduled-workflow throttling on the hosting platform. See *Temporal resolution* below.

---

## Temporal resolution

The three axes do not share a sampling rate, and the effective resolution of the composite is set by the slowest of them. Attention derives from daily Wikimedia pageview data; market from daily closes, cached for six hours. Only narrative carries genuine intraday variation, and it is present in roughly half of all observations.

Readings are therefore best understood as **daily-resolution observations of coupling, refreshed opportunistically**. The interface animates continuously; that animation renders state rather than indicating that new information has arrived. `C_lag` indicates a daily-scale phase relationship, corroborated or not, and is not a measurement of intraday lead time.

Processing buffers currently advance on the render tick rather than on data arrival. Advancing one step per `state.json` update, with back-fill from `history.jsonl`, would give `W_BUF` and `τ_max` defined durations — a v3 direction.

---

## Comparability across regions

The same model, weights, and attractor coordinates are applied to all three regions — a deliberate commitment to treating no region as baseline. Three consequences follow.

The macro-stress backdrop is drawn from US instruments (`VIXCLS`, `BAMLH0A0HYM2`, `DTWEXBGS`) and contributes 45% of every region's market value, so the three regional market signals share a common component.

Attention is read from `en.wikipedia` for all three regions. Wikimedia serves Hindi, Bengali, Tamil and other editions through the same API; they are not currently sampled. The effect is asymmetric — for the US and UK, `en.wikipedia` approximates the general reading public; for India it indexes an anglophone subset.

Narrative is restricted to economic stress vocabulary, so N measures the tone of stress discourse rather than general economic media tone. GDELT's `sourcecountry` filter selects by outlet location rather than audience, language, or subject.

Axis scales are not identical: N is bounded at ±0.762 by the `clip → tanh` sequence, while A and M reach ±1.

A full account of method, parameters, and observed availability is given in the accompanying methodological note.

---

## Known limitations and v3 directions

**Scalar attention (current):** the four affect clusters are combined in the backend into a single weighted composite — `Σ(weight × cluster_z) / n`, weights anxiety −1.0, confidence +1.0, aspiration +0.5, constraint −0.7 — before `state.json` is written. The compositional structure of the affect field therefore does not reach coupling computation, and `getClusterBreakdown()` returns `null`. Expressive divergence partially exposes this tension.

**v3 — vectorised attention:** A enters E as a 2D vector (valence × accumulation), with the four clusters as named quadrants. Each cluster carries its own coupling signature to the material field. Requires Google Trends Alpha or a richer sentiment pipeline via institutional credentials.

**Symmetric synchrony (current):** C_sync is symmetric co-movement. It does not distinguish which field is driving which.

**v3 — directional coupling:** Granger causality or lagged regression over longer windows.

**Scalar E (current):** Distinct expressive configurations can produce identical E values. Expressive divergence partially exposes this.

**v3 — vectorised E:** E as a 2D or 3D vector preserving internal structure through to coupling computation. Regime dynamics become cluster-aware.

**Static attractor coordinates (current):** Attractor positions are configured constants. They do not adapt to regional variation in baseline coupling.

**v3 — region-specific attractors:** Each region's attractor positions calibrated from its own historical distribution.

---

## Deployment

- **Frontend:** `propensities.github.io/animal-spirits/` (GitHub Pages — [`propensities/animal-spirits`](https://github.com/propensities/animal-spirits))
- **Backend:** `propensities.github.io/animal-spirits-api/data/state.json` (GitHub Pages — [`propensities/animal-spirits-api`](https://github.com/propensities/animal-spirits-api))
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
| v2.1 | Quadrant inset restored as relational layer between map and panels |
| v2.2 | Ternary attractor plot replaces cartesian quadrant — barycentric region positioning, region-identity dot colours, geometry directly expresses the three-attractor regime model |

---

*Propensities · v2.2*
