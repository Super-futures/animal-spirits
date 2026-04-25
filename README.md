# Animal Spirits

A live observatory reading three signals of collective economic state across three regions.

Attention, narrative, and market are read continuously, smoothed, and projected into a shared field. The system does not predict. It traces what is moving, where, and whether those movements are aligned.

**Live:** [super-futures.github.io/animal-spirits](https://super-futures.github.io/animal-spirits)
**Backend:** [super-futures/animal-spirits-api](https://github.com/super-futures/animal-spirits-api)
**Data feed:** [state.json](https://super-futures.github.io/animal-spirits-api/data/state.json) — refreshed every 15 minutes.

---

## The three signals

| Signal | What it reads | Source |
|---|---|---|
| **Attention** | Collective search and reading activity across four affect clusters (anxiety, confidence, aspiration, constraint), combined into a single magnitude per region | Wikimedia Pageviews API (32 articles per region, z-scored against a 29-day baseline) |
| **Narrative** | The tone of news carrying the moment — currently derived from the anxiety cluster, as the most reactive channel | GDELT DOC 2.0 TimelineTone, sourcecountry-scoped |
| **Market** | Regional equity state, blended with global macro stress | Alpha Vantage ETF proxies (SPY, ISF.LON, NIFTYBEES.BSE) + FRED (VIX, high-yield spread, dollar index) |

Each signal can lead, lag, or diverge from the others. The relationships between them — not any one reading alone — are what the interface surfaces.

## Three regions

United States, United Kingdom, India.

## Reading the field

The system renders three collective states continuously — **Expansion**, **Contraction**, **Instability** (bull, bear, stag in the Keynesian register). These are not categories but probabilities emerging from a latent field Ψ composed of the three signals.

Each panel shows a **posture** — Lean, Caution, or Pause — describing how the signals are holding together:

- **Lean** — signals are aligned; the field is coherent
- **Caution** — signals are diverging; attention and markets are out of sync
- **Pause** — instability elevated; the field has lost coherence

The posture is interpretive. The probabilities are generative.

## The visual grammar

- **Attention blooms** — warm, diffuse. Size encodes magnitude; diffusion encodes uncertainty.
- **Market cells** — cool, defined. Position encodes regime; size encodes conviction.
- **Narrative arrows** — direction encodes tone; length encodes magnitude; change encodes acceleration.
- **A ↔ M offset** — spatial distance between attention and market encodes temporal lag.
- **Ψ contour** — boundary where expansion tips into contraction across the field.

## What it is, what it isn't

Animal Spirits is an observational instrument, not a predictive model or trading signal. It treats economic and emotional behaviour as temporally entangled rather than causally linear: attention, narrative, and market signals co-evolve; none is privileged as the sole driver.

The interface is the argument — a continuous field rather than a discrete dashboard.

The intellectual lineage runs through Keynes (animal spirits), Shiller (narrative economics), and traditions of affective mapping in media art and critical design. A formal paper on the three-axis framework is in preparation.

## Architecture

This repository is the frontend — a single `index.html` using D3 and Canvas, polling a static `state.json` feed every 5 minutes.

The backend ([animal-spirits-api](https://github.com/super-futures/animal-spirits-api)) runs as a scheduled GitHub Actions workflow, composing state every 15 minutes from Wikimedia, Alpha Vantage, FRED, and GDELT, and committing to `data/state.json`. There is no server. The commit history of `state.json` forms a growing time-series dataset.

## Run locally

```bash
git clone https://github.com/super-futures/animal-spirits.git
cd animal-spirits
python3 -m http.server 8000
```

Open `http://localhost:8000`.

## Status

**v1.1** — three signals, three regions, continuous regime.

Attention combines the four affect clusters via a root-mean-square, which preserves the magnitude of any cluster's activation without collapsing opposing surges into nothing. The compositional structure of attention exists upstream but is reduced to a scalar before entering the field — a more structured reading (carrying the four-cluster vector through Ψ, exposing the mix in the interface) is the central target for v2.

Narrative is currently derived from the anxiety cluster as a reactive-channel proxy. Future refinement will focus on how this signal is constructed rather than on averaging additional clusters into it.

Attention uses a static 30-day baseline; a rolling baseline is a near-term update.

## Maintainer

Leon — [Superfutures](https://github.com/super-futures), Auckland.
