# Prompt — UAT Note

Use after a slice's tests are green, to prove it met the **business** need.
Green tests prove the code does what it was told; UAT proves it was told
the right thing.

---

```
Produce the UAT note for <VS-0xx — Name>.

Read first:
- docs/ba/09-UAT-STRATEGY.md
- docs/delivery/slices/<VS-0xx-SLICE-FILE>.md (acceptance criteria)
- docs/ba/07-PROFIT-CALCULATION-SPEC.md (if the slice touches a calculation)

Walk through the slice as the persona would (docs/product/PERSONAS.md),
using the UI, not the API. For EACH acceptance criterion record:

1. The criterion, quoted
2. The scenario walked through — the actual steps a real user would take
3. Expected result
4. Actual result
5. Pass / Fail
6. If Fail: which of these was at fault —
      wrong requirement | wrong formula | wrong data | wrong code
   Do not patch over it. Naming the layer is the point

Rules:
- Report failures honestly. A UAT note with no failures on a non-trivial
  slice is more suspicious than one with two
- Use synthetic data and label it as synthetic. Never present fabricated
  cost or fee figures as real or reconciled
- Where the slice produces a calculated number, verify it against a hand-
  worked example, not against what the code returned. Recomputing with the
  same code proves nothing
- If an acceptance criterion cannot be verified through the UI yet, say so
  and say why — do not silently drop it

Finish with:
- Overall pass / fail
- Anything that passed but felt wrong to use (this is worth recording even
  when it is not a defect)
- Which of the six depth criteria in docs/ba/09-UAT-STRATEGY.md this slice
  now demonstrates
```

---

## Notes

- Step 6's four-way split (requirement / formula / data / code) is the
  diagnostic habit the whole project is built around. Keep it.
- "Passed but felt wrong to use" catches usability problems that no
  acceptance criterion is phrased to catch.
