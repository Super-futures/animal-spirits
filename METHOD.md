# Animal Spirits — Methodological Note

**Leon Tan** · Propensities / Unitec Institute of Technology

Prepared for **RSD15 — *Data in Relation***, Valentino Castle, Turin, 13–16 October 2026.

Describes deployed version v2.2. Figures derived from 89.9 days of continuous logging, 15 May – 13 August 2026 (1,173 refresh events, 3,519 region-observations).

**→ [The live instrument](https://propensities.github.io/animal-spirits/)**
**→ [Frontend repository](https://github.com/propensities/animal-spirits)** — interface, coupling model, visual grammar
**→ [Backend repository](https://github.com/propensities/animal-spirits-api)** — sources, normalisation, and the logged record (`data/history.jsonl`)

---

## Contents

1. [What the instrument observes](#1-what-the-instrument-observes)
2. [Signal construction and normalisation](#2-signal-construction-and-normalisation)
3. [Temporal resolution](#3-temporal-resolution)
4. [Coupling metrics](#4-coupling-metrics)
5. [Regime assignment](#5-regime-assignment)
6. [Comparability across regions](#6-comparability-across-regions)
7. [What the instrument supports, and what it does not](#7-what-the-instrument-supports-and-what-it-does-not)
8. [Record and future direction](#8-record-and-future-direction)

---

## 1. What the instrument observes

Animal Spirits reads three signals per region — attention (A), market (M), and narrative (N) — and renders the *relationship between them* rather than any signal individually. Attention and narrative together constitute an expressive field (E); market constitutes a material field. The instrument is observational: it describes the current state of coupling between these fields. It does not forecast, and no claim of causal direction is made in either direction.

This note documents how the signals are constructed, how coupling is computed, how regime is assigned, and what the resulting readings can and cannot support.

---

## 2. Signal construction and normalisation

Each axis is normalised independently, by a method appropriate to its own distribution, into a signed scalar in approximately (−1, +1). Normalisation happens in the backend; the interface renders pre-normalised state.

### 2.1 Attention — Wikimedia Pageviews API

32 articles per region (8 terms in each of four affect clusters: anxiety, confidence, aspiration, constraint), 96 articles in total, read from `en.wikipedia` at daily granularity.

Each article's most recent complete day is z-scored against its own preceding 30-day series. Cluster value is the mean z-score across available terms. The regional scalar is the mean of weighted cluster z-scores, squashed through a hyperbolic tangent:

```
weights:  anxiety −1.0 · confidence +1.0 · aspiration +0.5 · constraint −0.7
composite = Σ(weight × cluster_z) / n_clusters_available
A = tanh(composite / 1.5)
```

Per-term z-scoring is deliberate. It measures attention *unusual relative to that term's own baseline* rather than absolute traffic, so that background popularity is factored out and deviation is retained. A term that is always heavily read contributes nothing unless its readership moves.

### 2.2 Market — Alpha Vantage ETF proxies with FRED macro backdrop

```
M = clip(0.55 × local_equity + 0.45 × global_stress)
```

Local equity uses index-tracking ETFs rather than the indices themselves, because the data tier used covers equities but not index endpoints: `SPY` (S&P 500), `ISF.LON` (FTSE 100), `NIFTYBEES.BSE` (Nifty 50). Tracking error is well under 1%. The scalar compares the mean of the most recent five daily returns against the standard deviation of prior returns in the series, squashed via `tanh(ratio / 2.0)` — a measure of recent movement relative to that market's own recent volatility, not of price level.

Global stress is drawn from three FRED series — `VIXCLS`, `BAMLH0A0HYM2` (US high-yield option-adjusted spread), and `DTWEXBGS` (broad trade-weighted dollar) — each z-scored against its own 60-observation history:

```
stress = tanh((−0.40·z_vix − 0.40·z_credit − 0.20·|z_dollar|) / 1.5)
```

**This backdrop is not regionally differentiated.** See §6.1.

### 2.3 Narrative — GDELT DOC 2.0, TimelineTone

One query per region, filtered by `sourcecountry` (US, UK, IN), over a one-day timespan, restricted to economic stress terms:

```
("recession" OR "unemployment" OR "inflation" OR "crisis"
 OR "layoffs" OR "bankruptcy")
```

The most recent numeric tone value is normalised:

```
N = tanh(clip(raw_tone / 5.0, −1, +1))
```

GDELT tone is used with its native sign — negative tone on these terms indicates narrative stress; no inversion is applied. Because the query is restricted to stress vocabulary, N measures *the tone of stress discourse*, not general economic media sentiment. This is a deliberate choice: stress narratives propagate faster and respond sooner than other economic discourse, which makes them the more reactive carrier. It is not a neutral read of media tone, and should not be interpreted as one.

---

## 3. Temporal resolution

The three axes do not share a sampling rate, and the effective resolution of the composite is set by the slowest of them.

| Axis | Source granularity | Distinct values per day (observed, median) | Null rate |
|---|---|---|---|
| Attention | daily | 7 | 0.1 % |
| Market | daily closes | 3 | 0.0 % |
| Narrative | intraday | 5 | **49.9 %** |

The backend runs on a fifteen-minute schedule, but observed cadence over 89.9 days is a median gap of **89.8 minutes** (mean 110.5; 90th percentile 208.1), yielding **13.0 refresh events per day** — approximately 14% of nominal. This reflects scheduled-workflow throttling on the hosting platform rather than source failure. No outage exceeded nine hours across the full period, and the ninety-day window is complete.

**The consequence for interpretation is significant and is stated plainly: attention and market both derive from daily-granularity sources.** Sampling more frequently than daily produces repeated values, not additional information — market values repeat between consecutive observations 83% of the time, attention 20%. Only narrative carries genuine intraday variation, and it is absent from roughly half of all observations.

Readings should therefore be understood as **daily-resolution observations of coupling, refreshed opportunistically**, not as real-time measurement. The interface's continuous animation renders state changes and visual dynamics; it does not indicate that new information has arrived.

---

## 4. Coupling metrics

All coupling computation occurs in the interface, from the normalised scalars, over a rolling 28-observation buffer per region. Signals are first smoothed by exponential moving average (α = 0.25) and z-scored within the buffer window.

The expressive field is a weighted composite of attention, narrative, and narrative velocity (ΔN):

```
E = 0.40·z(A) + 0.35·z(N) + 0.25·z(ΔN)
```

E compresses internal structure: distinct configurations of its three components can produce identical E. **Expressive divergence** is computed alongside it as the mean pairwise absolute difference between the three components, and surfaced separately, so that an internally tense expressive field reading as stable in composite remains visible. Vectorised E is a future direction.

### 4.1 Alignment

Magnitude-weighted proximity of expressive and material fields:

```
proximity = 1 − min(1, |E − z(M)| / 2)
weight    = σ(1.5·(|E| + |z(M)|) − 1.5)
C_align   = proximity·weight + 0.5·(1 − weight)
```

The magnitude weighting is necessary because two signals both near zero are trivially proximate. Without it, a quiescent system reads as strongly aligned. The weighting pulls such cases toward a neutral 0.5 rather than a spurious 1.0.

### 4.2 Synchrony

Pearson correlation of first differences of E and M across the buffer window. **Synchrony is symmetric**: it measures co-movement, not direction of influence. It is named accordingly, and no directional or causal reading should be drawn from it. Directional coupling — Granger causality or lagged regression — requires longer windows than the current record supports and is identified as a future direction.

### 4.3 Lag

A lead–lag search across offsets τ ∈ [−6, +6] observations, selecting the τ maximising Pearson correlation between E and M. The reported metric is offset magnitude weighted by correlation strength at that offset:

```
C_lag = (|τ_best| / 6) × max(0, r_at_τ_best)
```

The confidence weighting is essential: a large apparent offset supported by weak correlation yields a low C_lag rather than a strong lag claim. Sign is retained separately (negative = expressive field leads; positive = material field leads).

**The interpretive limit is set by §3.** Because attention and market both originate in daily sources, the buffer contains at most a few distinct values per day, and the lead–lag search cannot resolve phase structure finer than the underlying daily granularity. C_lag should be read as an indication of daily-scale phase relationship, corroborated or not, over approximately the preceding fortnight of distinct observations. It is not a measurement of intraday lead time, and no such claim is made.

### 4.4 Instability

```
I = min(1, [(1 − C_align) + (1 − |C_sync|) + C_lag] / 3)
```

Instability is not a fourth signal but a composite of coupling breakdown: fields distant from each other, moving without correlation, and phase-offset simultaneously.

---

## 5. Regime assignment

Regime is assigned by **proximity to named attractors** in (C_align, C_sync) space, not by threshold conditions:

| Attractor | C_align | C_sync |
|---|---|---|
| expansion | 0.75 | +0.55 |
| contraction | 0.75 | −0.55 |
| instability | 0.25 | 0.00 |

A region's regime label is that of the nearest attractor by Euclidean distance. This matters for the visual argument: because assignment is continuous rather than thresholded, position in coupling space and displayed label express the same underlying quantity, and a region approaching a boundary is legible as approaching it rather than switching abruptly. The attractor coordinates are design decisions grounded in the coupling model, not empirically calibrated positions — regional calibration against the accumulated record is a future direction.

Instability is a condition of the assemblage rather than a regime alongside the other two: the state in which expressive and material fields decouple across alignment, synchrony, and phase simultaneously.

---

## 6. Comparability across regions

The same model, weights, and attractor positions are applied to all three regions. This is a deliberate commitment to treating no region as baseline. It also carries specific and unequal costs, stated here rather than minimised.

### 6.1 The market stress backdrop is not regionally differentiated

All three FRED series are US instruments, and the identical stress scalar contributes 45% of every region's market value. UK and India market readings are therefore 45% composed of US financial conditions. Some apparent inter-regional market coupling is a property of this construction rather than an observation of the world. Region-native stress instruments are a priority future direction.

A related failure mode: when a regional equity fetch fails but the macro series succeed, that region falls back to a value composed entirely of the global backdrop. The axis continues to report as live.

### 6.2 Attention is read in English for all three regions

All regions are read from `en.wikipedia`. Wikimedia serves Hindi, Bengali, Tamil and other editions through the same API on identical terms; they are not sampled.

This omits less than might be assumed. Approximately 85–92% of Wikipedia pageviews originating in India already go to the English edition (Wikimedia Foundation, 2017; figure not precisely pinned, but direction confirmed). Adding the major Indian-language editions would therefore recover a modest fraction of Indian Wikipedia traffic, not a majority of it.

The asymmetry lies elsewhere — not in *which* edition is read, but in *who reads at all*:

| | Pageviews, July 2026 | Population | Per capita | Internet penetration |
|---|---|---|---|---|
| US | ~3,000 m | ~349 m | 8.6 | ~95 % (2024) |
| UK | ~739 m | ~69.9 m | 10.6 | ~95 % (2024) |
| India | ~545 m | ~1,470 m | 0.4 | ~70 % (2025) |

Per head of population, English Wikipedia reaches the US roughly 23 times more intensively than India, and the UK roughly 28 times. For the US and UK, `en.wikipedia` approximates the general reading public. For India it indexes a substantially smaller and more anglophone stratum — English-speakers in India outnumber the UK's entire population, but remain a small fraction of 1.4 billion — whose economic position is not representative.

The India attention signal should be read accordingly. Multilingual sampling remains a worthwhile refinement, but it addresses the smaller of the two problems.

### 6.3 Narrative coverage is filtered by outlet location

GDELT's `sourcecountry` filter selects by where a publication is based, not by audience, language, or subject matter.

GDELT does not publish active per-country outlet counts, so source density cannot be stated directly. Independent assessment is nonetheless consistent: the UK Office for National Statistics (2020) finds Western and particularly US media over-represented within GDELT, introducing a Western reporting perspective into event reporting. Benchmarking work (Hong et al., 2025) reports systematic under-representation of Indian sources, and higher extraction accuracy for English-language articles (~56%) than non-English (~53%). Network analysis of 140 million articles across 37,802 GDELT domains identifies the US as the primary hub of global news diffusion, with the UK and India following (Alipour et al., 2024).

The narrative axis therefore reads a media field whose structure is itself Western-centred, before any query is applied.

### 6.4 Equity proxies represent different fractions of their markets

SPY, ISF.LON and NIFTYBEES track indices of differing breadth relative to their national markets.

Household equity participation also differs substantially, though by less than an order of magnitude:

| | Direct holdings | Including funds, pensions, managed accounts |
|---|---|---|
| US | ~21 % of families | ~58 % of families |
| UK | ~21 % of adults | ~37–39 % of adults |
| India | ~5.3 % of households | ~9.5–10 % of households |

Sources: Federal Reserve Board Survey of Consumer Finances (2022); FCA Financial Lives (2022) and ONS Wealth in Great Britain; SEBI Investor Survey 2025, published January 2026. Definitions are not fully commensurable — the UK broad figure excludes mandatory workplace pensions while the US figure includes 401(k) and IRA holdings — so the comparison indicates magnitude, not precision.

The ratio is roughly four-fold on direct holdings and up to six-fold on broad participation. The channel between collective affect and realised market behaviour is therefore not structurally equivalent across the triad even where the arithmetic applied to it is identical.

### 6.5 Axis scales are not identical

Narrative is bounded at ±0.762 by the `clip → tanh` sequence, while attention and market reach ±1. Coupling metrics assume comparable ranges; the asymmetry mildly attenuates narrative's contribution to E.

---

## 7. What the instrument supports, and what it does not

**Supported.** A daily-resolution reading of whether expressive and material fields in a given region are aligned, co-moving, and phase-offset; comparison of that condition across three regions on identical terms; and observation of change in that condition over a ninety-day rolling record.

**Not supported.** Prediction of market direction or magnitude. Attribution of cause in either direction between affect and market behaviour. Intraday lead time. Claims about non-anglophone economic affect in any region. Any reading of a single axis as a measure of sentiment.

The instrument is a mirror with instrumental affordances rather than a diagnostic device. Its contribution is the rendering of coupling as a legible, navigable visual argument — the proposition that the relationship between expressive and material fields is itself the object worth observing.

---

## 8. Record and future direction

The backend has logged region-level state continuously since 15 May 2026, retained as a ninety-day rolling window. This record is the basis for the next stage of the work: empirical calibration of attractor positions per region, characterisation of dwell time and transition frequency between regimes, and assessment of whether the coupling model behaves equivalently across the three contexts or requires regional specification.

Named future directions, in order of priority: region-native market stress instruments; multilingual attention sampling; per-region provenance reporting including fallback disclosure; narrative decomposition by affect cluster rather than aggregation under stress terms; and vectorised expressive field.

---

## References

Alipour, S., Di Marco, N., Avalle, M., Etta, G., Cinelli, M., & Quattrociocchi, W. (2024). The drivers of global news spreading patterns. *Scientific Reports, 14*(1), Article 1519. https://doi.org/10.1038/s41598-024-52076-6

Federal Reserve Board. (2022). *Survey of Consumer Finances (SCF)*. Board of Governors of the Federal Reserve System.

Financial Conduct Authority. (2022). *Financial Lives 2022 survey*. FCA.

Hong, D., Fu, Z., Zhang, X., & Pan, Y. (2025). Research on the development and application of the GDELT event database. *Data, 10*(10), Article 158. https://doi.org/10.3390/data10100158

Office for National Statistics. (2020). *Global Database of Events, Language and Tone (GDELT): Data quality note*. ONS.

Securities and Exchange Board of India. (2026). *Investor survey 2025*. SEBI.

Wikimedia Foundation. (2017, October 27). *Just how many people are reading Wikipedia in your country, and what language are they using?* Wikimedia Diff.

World Bank. (2025). *Individuals using the Internet (% of population)* [Data set]. World Bank Open Data.

---

## Access

The instrument is live at **[propensities.github.io/animal-spirits](https://propensities.github.io/animal-spirits/)**.

The logged record — every observation described in this note — is public at **[`data/history.jsonl`](https://github.com/propensities/animal-spirits-api/blob/main/data/history.jsonl)**, alongside the current state at **[`data/state.json`](https://github.com/propensities/animal-spirits-api/blob/main/data/state.json)**. Source modules for all three axes are in [`sources/`](https://github.com/propensities/animal-spirits-api/tree/main/sources).

---

*Prepared August 2026 · Propensities · Unitec Institute of Technology*
