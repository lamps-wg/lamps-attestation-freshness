# LAMPS Attestation Freshness slides

- `freshness-ietf126.tex`: IETF 126 deck, summarizes the changes from draft
  version `-05` to `-08`.
- `freshness-ietf127.tex`: IETF 127 deck, summarizes the changes from draft
  version `-08` to `-10` and introduces the passport model section proposed
  in PR #58.

Build the current deck with:

```sh
make
```

The `TEX` variable in the Makefile selects the deck. The sources use only
standard Beamer, TikZ, `booktabs`, and `array` packages.
