# Animal Spirits

A live affective observatory that maps collective human mood against economic behaviour across three regions (United States, United Kingdom, India), rendered as a shared interpretive field rather than a dashboard of feeds.

**Live at:** [super-futures.github.io/animal-spirits](https://super-futures.github.io/animal-spirits/)

## What this is

Animal Spirits is a research-grade visualisation that treats attention, market, and narrative as three analytically independent axes of a single collective state. Its intellectual lineage runs through Keynes' "animal spirits" — the non-rational drivers of economic action — and has a direct precursor in Maurice Benayoun's 2011 work *e-Forecast*.

The project is a **mirror with instrumental affordances**. Not a trading signal. Not a predictive engine. A way of reading, in real time, how three heterogeneous affective signals align and misalign across three regions — and how that alignment shifts over hours and days.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  animal-spirits (this repo)             │
│                                                          │
│  index.html renders a live field visualisation:         │
│  - EMA-smoothed signals                                 │
│  - Derivatives and instability detection                │
│  - Latent field Ψ via radial basis functions           │
│  - Three posture states (Lean / Caution / Pause)       │
│                                                          │
│  Polls every 5 minutes:                                 │
│  ↓                                                       │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│       animal-spirits-api (separate repo)                │
│                                                          │
│  GitHub Actions cron, every 15 minutes:                 │
│    - Fetch attention (Wikimedia Pageviews, 4 clusters) │
│    - Fetch market (Alpha Vantage ETFs + FRED stress)   │
│    - Fetch narrative (GDELT DOC 2.0 TimelineTone)      │
│    - Compose unified state, commit to data/state.json  │
│                                                          │
│  Published at:                                          │
│  super-futures.github.io/animal-spirits-api/            │
│    data/state.json                                      │
└─────────────────────────────────────────────────────────┘
```

The [backend](https://github.com/super-futures/animal-spirits-api) is a scheduled job, not a server — no cold starts, no billing risk, no CORS dance. The frontend reads a static JSON file and does all the interpretive work locally.

## The three axes

| Axis | Source | What it reads |
|------|--------|---------------|
| **Attention** | Wikimedia Pageviews | Which anxiety/confidence/aspiration/constraint terms people are looking up today vs. their 30-day baseline |
| **Market** | Alpha Vantage (ETFs) + FRED | Regional equity momentum (SPY, ISF.LON, NIFTYBEES.BSE) blended with a global macro-stress backdrop (VIX, high-yield credit spread, dollar anomaly) |
| **Narrative** | GDELT DOC 2.0 | Average tone of news coverage matching anxiety keywords, per region, over the last 24 hours |

Each returns a scalar in approximately `[−1, +1]`. Negative = stress/contraction-coded, Positive = expansion-coded. Zero = no signal or neutral.

## Visual language

The interface is deliberately not a dashboard. Signals are rendered as pressure zones, radial gradients, and spatial offsets encoding temporal lag between axes.

- **Warm diffuse circles** = attention (intensity ∝ volume, spread ∝ uncertainty)
- **Cool defined rectangles** = market (position encodes regime)
- **Purple arrows** = narrative (length ∝ volume, direction ∝ tone velocity)
- **Offset between attention and market** = temporal lag (the signature of animal spirits leading or trailing material conditions)
- **RBF contour line** = Ψ = 0, the boundary between expansion and contraction regions of the latent field

## Current state

**v1.0** — first release using the unified backend state feed. Architecture simplified from three-source direct fetching to single static-JSON consumption.

Known next work (not urgent, not blocking):
- Vocabulary alignment pass (Lean/Caution/Pause as primary vocabulary, bull/bear as technical subtitle)
- Axis-key clarity (what each visual encoding *changes* when data changes)
- Ψ-contour label in the legend

## Running locally

This is a single HTML file. To preview:

```bash
# Python 3
python3 -m http.server 8000
# Then open http://localhost:8000
```

Or open `index.html` directly in a browser — the only caveat is that some browsers block `fetch` from `file://` URLs, so serving via a local HTTP server is more reliable.

## Deployment

- **GitHub Pages:** `Settings → Pages → Deploy from a branch → main / (root)`
- **Netlify:** connect the repo, no build step needed (static HTML)

## Attribution

- Keynes, J. M. (1936). *The General Theory of Employment, Interest and Money*. Chapter 12 — source of the "animal spirits" framing.
- Benayoun, M. (2011). *e-Forecast*. Hong Kong — conceptual precursor for real-time collective-mood visualisation.
- Leetaru, K. — GDELT Project, whose public computational-narrative infrastructure makes the narrative axis possible.

## Maintainer

Leon, at [Superfutures](https://github.com/super-futures) / Unitec (Te Pūkenga), Auckland.
