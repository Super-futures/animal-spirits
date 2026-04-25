# Animal Spirits

A live observatory mapping attention, stress propagation, and market signals across three regions.

Three axes — attention, narrative, and market — are read continuously and projected into a shared field. The system does not predict future states. It diagnoses current dynamics: what is present, what is moving, how those movements are accelerating, and whether the axes are aligning or diverging.

**Live:** https://super-futures.github.io/animal-spirits  
**Backend:** https://github.com/super-futures/animal-spirits-api  
**Data feed:** https://super-futures.github.io/animal-spirits-api/data/state.json — refreshed every 15 minutes

---

## The three axes

Three analytically independent signals, held alongside each other rather than collapsed into a single index. Each axis performs a distinct role in the system.

| Axis | Epistemic role | What it reads | Source |
|---|---|---|---|
| **Attention** | Compositional — the distribution of affect | What kinds of signals are present, and in what mix across anxiety, confidence, aspiration, constraint | Wikimedia Pageviews API (clustered and normalised per region) |
| **Narrative** | Dynamic — the propagation of stress | Where pressure is building, how it is moving, and how quickly it is accelerating | GDELT DOC 2.0 TimelineTone, sourcecountry-scoped |
| **Market** | Responsive — realised behavioural output | What has been expressed in price and macro conditions | Alpha Vantage ETF proxies (SPY, ISF.LON, NIFTYBEES.BSE) + FRED (VIX, high-yield spread, dollar index) |

The axes are deliberately not equivalent.

- **Attention** asks: what is present in the field  
- **Narrative** asks: what is moving through the field  
- **Market** asks: what has been realised  

Narrative is not a second measure of attention. It is the dynamic layer that traces how pressure moves through the attention field over time. It is currently operationalised as an anxiety-weighted stress signal — a proxy for stress propagation rather than a definition of narrative itself.

Each axis can lead, lag, or diverge from the others. These relationships — not the individual signals in isolation — are the primary object of observation.

---

## Three regions

United States, United Kingdom, India — three economies, three attention systems, three narrative pipelines.

---

## Reading the field

The system renders three collective states continuously — **Expansion**, **Contraction**, **Instability** (bull, bear, stag in the Keynesian register). These are not discrete categories but probabilities emerging from a latent field Ψ composed of the three axes.

Ψ is constructed from normalised signals and their rates of change, producing a continuous field from which regime probabilities emerge.

The field is dynamic:

- It incorporates **rate of change** (derivatives of signals)  
- It registers **divergence** between axes  
- It tracks **instability as tension, divergence, and entropy within the field**, rather than as a standalone class  

Signals are temporally smoothed and differentiated, allowing the system to register direction and acceleration rather than static levels alone.

Each panel shows a **posture** — Lean, Caution, or Pause — describing how the axes are holding together:

- **Lean** — signals are aligned; the field is coherent  
- **Caution** — signals are diverging; alignment is weakening  
- **Pause** — instability elevated; coherence has broken down  

The posture is interpretive. The probabilities are generative.

---

## The visual grammar

- **Attention blooms** — warm, diffuse. Size encodes magnitude; diffusion encodes uncertainty.  
- **Market cells** — cool, defined. Position encodes regime; size encodes conviction.  
- **Narrative arrows** — direction encodes stress direction; length encodes magnitude; change encodes acceleration.  
- **A ↔ M offset** — spatial distance between attention and market encodes relative lead–lag (phase offset), indicating which signal is ahead.  
- **Ψ contour** — boundary where expansion tips into contraction across the field.  

---

## What it is, what it isn’t

Animal Spirits is an observational instrument, not a predictive model or trading signal.

It treats economic and emotional behaviour as temporally entangled rather than causally linear: attention, narrative, and market signals co-evolve; none is privileged as the sole driver.

The system does not forecast. It renders the present as a structured field — including direction, acceleration, and instability — making visible how collective dynamics are forming in real time.

The interface is the argument — a continuous field rather than a discrete dashboard.

---

## Architecture

This repository is the frontend — a single `index.html` using D3 and Canvas, polling a static `state.json` feed.

The backend (animal-spirits-api) runs as a scheduled GitHub Actions workflow, composing state every 15 minutes from Wikimedia, Alpha Vantage, FRED, and GDELT, and committing to `data/state.json`.

There is no server. The commit history of `state.json` forms a growing time-series dataset of the field.

---

