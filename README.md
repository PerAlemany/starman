# 👨‍🚀 Starman

> *"There's a starman waiting in the sky"* — David Bowie, 1972

A skill for [Claude Code](https://claude.ai/code) / [OpenCode](https://opencode.ai) that activates mission-control precision mode. No hedging. No speculation. Just confirmed facts, explicit unknowns, and structured reasoning — the way you'd want your systems diagnosed at 2am.

## 📦 Installation

```bash
npx skills add https://github.com/PerAlemany/starman --skill starman
```

---

## What it does

Starman replaces vague, hedge-filled responses with a structured reasoning framework borrowed from aerospace protocols:

```
STATUS:    confirmed state — facts only
ANOMALY:   what's wrong — precise
ANALYZE:   root cause candidates, tradeoffs
GO:        proceed — criteria met
PROCEDURE: ordered steps — atomic, verifiable
VERIFY:    how to confirm success
```

Uncertainty is never hidden. If something is unknown, it's labeled `UNKNOWN:`. If something is assumed, it's labeled `HYPOTHESIS:`. No "it might be" or "this could potentially".

---

## Activation

Say any of these to activate:

| Phrase | Example |
|--------|---------|
| `starman` | `starman. production db throwing FK violations` |
| `starman mode` | `starman mode. k8s oomkilling pods, no code changes` |
| `mission control` | `mission control — kafka vs rabbitmq, 10k msgs/sec` |
| `astronaut mode` | `astronaut mode on. cloudwatch costs tripled` |
| `precision mode` | `precision mode — redis dropping writes every 40min` |
| `talk like an astronaut` | `talk like an astronaut. react app re-renders 200x` |

To deactivate: `abort starman` or `normal mode`.

---

## Example

**❌ Without Starman:**

> Your React component might be re-rendering because of a few possible reasons. It could be that you have an inline object prop that's creating a new reference on each render, or perhaps the parent component is re-rendering and passing new props. You might want to look into `useMemo` or `React.memo`, which could potentially help depending on your specific situation.

**✅ With Starman — nominal:**

```
STATUS:    Component re-renders on every parent render cycle.
ANOMALY:   Re-render triggered even when parent props are stable.
ANALYZE:   Inline object prop creates new reference on each render.
           Shallow comparison sees new ref → re-render triggered.
           Options:
             - useMemo: low footprint, preserves component API (preferred)
             - Move object outside component: simpler, less flexible
GO:        useMemo — minimal change, stable deps array available.
PROCEDURE: 1. Wrap object in `useMemo` with stable deps array
           2. Run React DevTools Profiler
VERIFY:    Zero re-renders on stable parent renders confirmed in Profiler.
           → VERIFY fails: inspect deps array for unstable refs, return to ANALYZE.
```

**⚡ With Starman — terse** (`/starman terse`):

```
Inline object prop → new ref each render → re-render.
useMemo preferred — minimal footprint, preserves API.
GO: useMemo with stable deps.
1. Wrap in `useMemo`  2. Profiler
→ Zero re-renders. Fails: check deps for unstable refs.
```

---

## Two modes

| Mode | What changes |
|------|-------------|
| `nominal` (default) | 📋 Full structure. Section labels visible. |
| `terse` | ⚡ Same precision and completeness. No labels. Dense. |

Switch with `/starman nominal` or `/starman terse`.
