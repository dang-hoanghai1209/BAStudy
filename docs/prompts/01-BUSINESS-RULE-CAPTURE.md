# Prompt — Business Rule Capture

Use when an unanswered business question is blocking a slice. The output is
a rule written into `docs/ba/08-BUSINESS-RULES.md`, not code.

---

```
A business rule needed by <VS-0xx> is unanswered. Help me decide it
properly, then record it.

Question: <the question, stated precisely — e.g. mandatory question #7:
is COGS taken at time-of-sale price or current price?>

Read first:
- docs/ba/08-BUSINESS-RULES.md
- docs/ba/07-PROFIT-CALCULATION-SPEC.md
- docs/ba/03-DOMAIN-GLOSSARY.md
- docs/architecture/HISTORICAL-DATA-POLICY.md

Do this:
1. State what is actually being asked, in business terms, without jargon.
2. Give each plausible answer, and for each: what it means for the seller,
   what it means for the data model, and what it makes impossible later.
3. Say which real-world practice is most common, and be explicit about
   whether that is established fact or your inference.
4. Give a worked numeric example showing the answers producing DIFFERENT
   results. If they never differ, say so — the question may not matter.
5. Recommend one, with the reasoning stated.
6. List everything downstream this decision constrains.

Do NOT write code. Do NOT edit docs/ba/08-BUSINESS-RULES.md until I confirm
the decision.

Once I confirm, write the rule in this format:

### BR-0xx — <short title>
Statement: <unambiguous, testable>
Rationale: <why>
Decided: <date> by <who>
Affects: <entities, calculations, slices>
Test: <the test that proves it>
```

---

## Notes

- Step 4 is the highest-value step. Two answers that never produce
  different numbers are not worth a decision meeting.
- "Statement" must be testable. "COGS should be accurate" is not a rule;
  "COGS is the cost record whose effective range covers the order date" is.
- Never let this prompt drift into implementation. The output is a rule.
