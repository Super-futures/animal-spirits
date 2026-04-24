# Animal Spirits

A live affective observatory that reads three axes of collective state — **attention, market, and narrative** — across three regions — **United States, United Kingdom, India** — and renders them as a shared interpretive field rather than a dashboard of feeds.

**Live at:** [super-futures.github.io/animal-spirits](https://super-futures.github.io/animal-spirits/)

---

## What this is

Animal Spirits is a research-grade visualisation that treats attention, market, and narrative as three analytically independent axes of a single collective state. Its intellectual lineage runs through Keynes' *animal spirits* — the non-rational drivers of economic action, described in Chapter 12 of the *General Theory*.

The project is a **mirror with instrumental affordances**, not a predictive engine. A way of reading, in real time, how three heterogeneous affective signals align and misalign across three regions, and how that alignment shifts over hours and days.

## What you see

The interface renders the field as spatial geometry, not as a chart.

- **Warm diffuse circles** — attention (size = volume, diffusion = uncertainty)
- **Cool defined rectangles** — market (position = regime, size = conviction)
- **Purple arrows** — narrative (direction = tone, length = volume)
- **Offset between attention and market** encodes temporal lag: the signature of affect leading or trailing material conditions
- **Ψ-contour line** traces the boundary between expansion and contraction zones of the latent field
- **Posture readings** — Lean, Caution, Pause — describe the field's current invitation to action

Three per-region panels below the map show the underlying probability decomposition: expansion / contraction / instability, along with tension, entropy, and the A↔M temporal relationship.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  animal-spirits (this repo)              │
│                                                          │
│  index.html — a single self-contained HTML file:        │
│    - D3 + Canvas rendering                              │
│    - EMA smoothing, derivatives, instability detection  │
│    - Latent field Ψ via radial basis functions          │
│    - Lean / Caution / Pause posture derivation          │
│                                                          │
│  Polls every 5 minutes, flashes the freshness           │
│  indicator only when a new sample timestamp arrives.    │
│                        ↓                                 │
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│       animal-spirits-api (sibling repo)                  │
│                                                          │
│  GitHub Actions cron, every 15 minutes:                 │
│    - Fetch attention   (Wikimedia Pageviews)            │
│    - Fetch market      (Alpha Vantage ETFs + FRED)      │
│    - Fetch narrative   (GDELT DOC 2.0 TimelineTone)     │
│    - Compose unified state, commit data/state.json      │
│                                                          │
│  Published at:                                          │
│  super-futures.github.io/animal-spirits-api/            │
│      data/state.json                                    │
└─────────────────────────────────────────────────────────┘
```

The [backend](https://github.com/super-futures/animal-spirits-api) is a scheduled job, not a server — no cold starts, no billing risk, no CORS dance. The frontend reads a single static JSON file and does all the interpretive work locally.

## The three axes

| Axis | Source | What it reads |
|------|--------|---------------|
| **Attention** | Wikimedia Pageviews | Which economic-anxiety / confidence / aspiration / constraint terms people are looking up today, relative to their 30-day baseline |
| **Market** | Alpha Vantage (ETFs) + FRED | Regional equity momentum (SPY, ISF.LON, NIFTYBEES.BSE) blended with a global macro-stress backdrop (VIX, high-yield credit spread, dollar anomaly) |
| **Narrative** | GDELT DOC 2.0 | Average tone of news coverage matching anxiety keywords, per region, over the last 24 hours |

Each axis is normalised per region against its own rolling baseline and returns a signed scalar in approximately `[−1, +1]`. Negative = stress/contraction-coded. Positive = expansion-coded. Zero = neutral or no signal.

## Field dynamics

The frontend layers three further derivations on top of the unified state:

- **EMA smoothing** — each axis is smoothed with `α = 0.25` to damp per-tick noise without hiding genuine inflections
- **Latent field Ψ** — a weighted combination of the three axes projected spatially via radial basis functions
- **Regime probability decomposition** — expansion (*bull*), contraction (*bear*), instability (*stag*) are emergent probabilities derived from Ψ and its tension, not discrete thresholds
- **Lean / Caution / Pause posture** — collapses the probability decomposition into a plain-language stance toward action

The bull/bear/stag terminology is preserved as a technical subtitle because the literature uses it. The primary vocabulary throughout the UI is *expansion / contraction / instability* and, for posture, *Lean / Caution / Pause*.

## Freshness semantics

The small green dot next to the timestamp in the header has meaningful behaviour:

- **Dim** — the current sample is the same one we already saw
- **Bright flash** — a new state.json commit has been detected on the backend

The header text `field sampled · HH:MM UTC` shows the UTC timestamp of the backend commit, not the time of the browser poll. The page polls every 5 minutes, the backend refreshes every 15 minutes, so roughly one in three polls produces a new sample.

## Running locally

This is a single HTML file with no build step. To preview:

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000`.

You can also open `index.html` directly in a browser, but some browsers block `fetch` from `file://` URLs, so serving via a local HTTP server is more reliable.

## Deployment

- **GitHub Pages**: Settings → Pages → Deploy from a branch → `main` / root
- **Netlify**: connect the repo, no build command needed

## Attribution

- Keynes, J. M. (1936). *The General Theory of Employment, Interest and Money.* Chapter 12 — source of the "animal spirits" framing.
- Benayoun, M. (2011). *e-Forecast.* Hong Kong — conceptual precursor for real-time collective-mood visualisation.
- Leetaru, K. — [GDELT Project](https://www.gdeltproject.org/), whose public computational-narrative infrastructure makes the narrative axis possible.
- Wikimedia Foundation — Pageviews API.
- Alpha Vantage, FRED (St. Louis Fed) — market and macro-stress data.

## Versions

- **v1.0** (current) — unified state feed, Expansion/Contraction/Instability primary vocabulary, freshness indicator tied to real backend commits, Ψ-contour labelled in legend, honest axis-key grammar.
- v0.1 – v0.8 — prototype lineage, archived at [animalspirits-legacy](https://github.com/super-futures/animalspirits-legacy).

## Maintainer

Leon, at [Superfutures](https://github.com/super-futures), Auckland.
