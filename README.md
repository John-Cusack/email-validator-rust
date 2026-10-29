# email-validator-rust

Experimental Rust port lane for
[python-email-validator](https://github.com/JoshData/python-email-validator)
(syntax validation + normalization). The Unlicense (public domain), per the
upstream CC0 license; see `NOTICE` for attribution.

## Lane outcome (2026-10-29): `no_gain` — no Rust port scoped

Tier-1 pure-Python fixes measured first on a frozen 2000-address workload
(Ryzen 9 5950X, CPython 3.12.3, email-validator 2.3.0, min-of-7 fresh-process
wall + worker self-RSS both arms, `check_deliverability=False`):

| Track | Stock | Tier-1 | Speedup |
|---|---|---|---|
| mixed-2000 (representative) | 112.8 ms | 35.3 ms | 3.20x |
| ascii-valid-5000 | 296.2 ms | 46.1 ms | 6.43x |
| idn-1000 | 104.0 ms | 106.1 ms | 0.98x (idna-bound, unchanged) |

Tier-1 (upstream-plausible, suite-identical — 306/306 + 468/468 differential
with exact messages): skip the `idna.decode`/`idna.encode` roundtrip for
pure-ASCII domains without Punycode labels (2.03x mixed isolated); vectorize
per-character checks to C-speed scans; cache per-call module imports;
ASCII fast path for the split loop.

Kill math against the tier-1 baseline: warmed cProfile attribution of the
representative track gives email_validator-own 39.7%, idna-shared 16.7%,
rest (C `re`, `unicodedata`, builtins) 43.6% — infinite-speedup ceiling of
an own-code-only kernel **1.66x < 2x**. The IDN track cannot move without
reimplementing the shared `idna` dependency (a different package) while
keeping byte-exact `idna` error strings. Honest `no_gain`.

Full evidence: rustsmith `data/rs-port-email-validator/` + ADR-065
([rustsmith](https://github.com/John-Cusack/rustsmith) PR body carries the
bench table and capture counts).
