# Search grammar report

## V6-informed update, 2026-11-13

The engine now supports a typed all/any filter operator and a causal trigger
window spanning the current and up to three previous M1 bars, including
mutation, crossover, canonical cache keys and executable-signal tests. This is
still a bounded trigger-plus-filters model; it is not the general typed tree or
multi-event sequence grammar requested by V6. The earlier baseline description
below refers to the pre-upgrade implementation.

## Current baseline

The current genome represents one trigger, a list of filter conditions, and
separate entry, exit, risk, and session fields. Entry conditions are evaluated
as a trigger combined with filters; the representation does not support a
general typed expression tree, OR nodes, event sequences, or explicit
temporal-condition nodes.

The existing condition catalog provides named, parameterized indicator and
price-action predicates. `GenomeFactory.Repair` removes invalid roles and
duplicate filter kinds/timeframes, clamps parameters, and enforces the
configured condition count and a six-indicator ceiling. Existing condition
signal tests cover causality for selected trigger kinds, including Heikin-Ashi
in the short smoke-test codebase, but this is not a grammar-wide proof.

## V6 acceptance status

| Requirement | Status |
|---|---|
| Generate/mutate/crossover typed expression trees | Not implemented |
| AND/OR and bounded temporal sequence nodes | Not implemented |
| Canonical serialization for expression trees | Not applicable to current grammar |
| Semantic simplification and contradiction detection | Not implemented |
| Static rejection of impossible entry logic | Not implemented |
| Max six indicator families | Enforced by genome repair |
| Bounded condition count | Enforced by `min_conditions` / `max_conditions` |
| Causal named-condition calculations | Implemented condition-by-condition; not proven for every catalog item |

No measured search result can demonstrate structural tree evolution because
the current model does not represent that grammar.
