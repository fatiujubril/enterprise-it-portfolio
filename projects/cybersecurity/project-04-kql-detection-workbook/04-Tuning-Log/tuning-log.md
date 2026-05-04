# Detection Rule Tuning Log

This log tracks all tuning decisions made during the detection engineering process.
Each entry documents what fired, whether it was a true or false positive, and what change was made.

---

## Log Format

| Date | Rule | Event That Fired | TP or FP | Tuning Action | New Threshold |
|---|---|---|---|---|---|
| - | - | - | - | - | - |

---

## Notes
- All tuning decisions are made based on observed lab data
- FP rate is measured over 48 hours per rule before and after tuning
- Exclusions are implemented using Sentinel Watchlists where applicable
