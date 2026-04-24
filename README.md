# Animal Spirits

A live affective observatory mapping collective mood against economic behaviour across three regions.

Three axes — attention, market, narrative — read continuously and projected into a shared field. The system doesn't predict. It traces what's moving, where, and whether it's moving together.

**Live:** [super-futures.github.io/animal-spirits](https://super-futures.github.io/animal-spirits/)
**Backend:** [super-futures/animal-spirits-api](https://github.com/super-futures/animal-spirits-api)
**Data feed:** [state.json](https://super-futures.github.io/animal-spirits-api/data/state.json), refreshed every 15 minutes.

---

## The three axes

Three analytically independent signals, held alongside each other rather than collapsed into one dial.

| Axis | What it reads | Source |
|---|---|---|
| **Attention** | Collective search and reading behaviour across four affect clusters — anxiety, confidence, aspiration, constraint | Wikimedia Pageviews API (32 articles per region, z-scored against 29-day baseline) |
| **Market** | Regional equity state, blended with global macro stress | Alpha Vantage ETF proxies (SPY, ISF.LON, NIFTYBEES.BSE) + FRED (VIX, high-yield spread, dollar index) |
| **Narrative** | Tone of the story carrying the moment | GDELT DOC 2.0 TimelineTone, sourcecountry-scoped |

Each axis can lead, lag, or diverge from the others. The relationship between them is where the reading lives.

## Three regions

United States, United Kingdom, India. Three economies, three attention economies, three narrative pipelines.

## Reading the field

The system renders three collective states continuously — **Expansion**, **Contraction**, **Instability** (bull, bear, stag in the Keynesian register). These aren't categorical labels but probabilities emerging from a latent field Ψ composed of the three axes.

Each panel shows a **posture** — Lean, Caution, or Pause — describing not the regime itself but how the axes are holding together:

- **Lean** — signals are moving together; the field is coherent
- **Caution** — signals are out of sync; attention and markets have diverged
- **Pause** — instability elevated; the field has lost its shape

The posture is human-scale. The probabilities are the machinery.

## The visual grammar

- **Attention blooms** — warm, diffuse. Size encodes volume, diffusion encodes uncertainty.
- **Market cells** — cool, defined. Position encodes regime, size encodes conviction.
- **Narrative arrows** — direction encodes tone, length encodes volume, acceleration encodes contagion risk.
- **A ↔ M offset** — spatial distance between attention and market forms encodes temporal lag.
- **Ψ contour** — a faint boundary across the map where expansion tips into contraction.

## What it is, what it isn't

Animal Spirits is a mirror with instrumental affordances, not a predictor or a trading signal engine. It treats emotional and economic behaviour as temporally entangled, not causally linked in one direction. The interface is the argument.

The intellectual lineage runs through Keynes on animal spirits, Shiller on narrative economics, and the tradition of data-driven affective mapping in media art and critical design. A research paper formalising the three-axis framework is in preparation.

## Architecture

This repo is the frontend — a single `index.html`, D3 + Canvas, polling the static `state.json` feed every 5 minutes. No build step. No framework.

The backend ([animal-spirits-api](https://github.com/super-futures/animal-spirits-api)) is a scheduled GitHub Actions job that composes state every 15 minutes by pulling from Wikimedia, Alpha Vantage, FRED, and GDELT, and commits the result to `data/state.json`. No server. The commit history of `state.json` is a growing time-series dataset.

## Run locally

```bash
git clone https://github.com/super-futures/animal-spirits.git
cd animal-spirits
python3 -m http.server 8000
```

Open `http://localhost:8000`. The page will read from the live backend feed.

## Status

**v1.1** — three axes, three regions, continuous regime. The narrative axis is currently wired for the anxiety cluster; the other three clusters (confidence, aspiration, constraint) are defined in code but not yet live. Attention uses a static 30-day baseline — rolling baseline is a near-term upgrade.

## Maintainer

Leon, at [Superfutures](https://github.com/super-futures), Auckland.
