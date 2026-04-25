# Animal Spirits

A live observatory mapping attention, stress propagation, and market signals across three regions.

Three axes — attention, narrative, and market — are read continuously and projected into a shared field. The system does not predict. It traces what is present, what is moving, and whether those movements are aligned.

**Live:** [super-futures.github.io/animal-spirits](https://super-futures.github.io/animal-spirits)
**Backend:** [super-futures/animal-spirits-api](https://github.com/super-futures/animal-spirits-api)
**Data feed:** [state.json](https://super-futures.github.io/animal-spirits-api/data/state.json) — refreshed every 15 minutes.

---

## The three axes

Three analytically independent signals, held alongside each other rather than collapsed into a single index. Each axis performs a distinct role in the system.

| Axis | Epistemic role | What it reads | Source |
|---|---|---|---|
| **Attention** | Compositional — the distribution of affect | What kinds of signals are present, and in what mix across anxiety, confidence, aspiration, constraint | Wikimedia Pageviews API (32 articles per region, z-scored against a 29-day baseline) |
| **Narrative** | Dynamic — the propagation of stress | Where pressure is building, how it is moving, and how quickly it is accelerating. Currently operationalised as an anxiety-weighted stress signal | GDELT DOC 2.0 TimelineTone, sourcecountry-scoped |
| **Market** | Responsive — realised behavioural output | What has been expressed in price and macro conditions | Alpha Vantage ETF proxies (SPY, ISF.LON, NIFTYBEES.BSE) + FRED (VIX, high-yield spread, dollar index) |

The axes are deliberately not equivalent.

- **Attention** asks: what is present in the field
- **Narrative** asks: what is moving through the field
- **Market** asks: what has been realised

Narrative is not a second measure of attention. It is the dynamic layer that traces how pressure moves through the attention field over time. Each axis can lead, lag, or diverge from the others — the relationships between them are the primary object of observation.

## Three regions

United States, United Kingdom, India — three economies, three attention systems, three narrative pipelines.

## Reading the field

The system renders three collective states continuously — **Expansion**, **Contraction**, **Instability** (bull, bear, stag in the Keynesian register). These are not categories but probabilities emerging from a latent field Ψ composed of the three axes.

Each panel shows a **posture** — Lean, Caution, or Pause — describing how the axes are holding together:

- **Lean** — signals are aligned; the field is coherent
- **Caution** — signals are diverging; attention and markets are out of sync
- **Pause** — instability elevated; the field has lost coherence

The posture is interpretive. The probabilities are generative.

## The visual grammar

- **Attention blooms** — warm, diffuse. Size encodes volume; diffusion encodes uncertainty.
- **Market cells** — cool, defined. Position encodes regime; size encodes conviction.
- **Narrative arrows** — direction encodes stress direction; length encodes magnitude; change encodes acceleration.
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

**v1.1** — three axes, three regions, continuous regime.

The narrative axis is intentionally derived from an anxiety-weighted signal — not as a measure of sentiment, but as a proxy for stress propagation, the fastest and most reactive channel through which pressure moves in the system. Future refinement will focus on how stress is derived (weighting, modulation), not on averaging affect clusters into narrative, which would collapse the distinction between compositional and dynamic axes.

Attention currently uses a static 30-day baseline; a rolling baseline is a near-term update.

## Maintainer

Leon — [Superfutures](https://github.com/super-futures), Auckland.
