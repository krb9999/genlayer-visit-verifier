# genlayer-visit-verifier

A [GenLayer](https://genlayer.com) Intelligent Contract that adjudicates a
simple real-world question: **did this guest actually stay at this hotel / visit
this attraction park?**

Someone submits a claim (guest or booking reference, venue, `hotel` or
`attraction`, date range, public evidence URL). Each validator fetches the
evidence page, reasons about it with its own LLM, and the network must agree on
one of:

| Verdict | Meaning |
| --- | --- |
| `CONFIRMED` | Evidence clearly shows this guest at this venue within the dates, not cancelled/refunded |
| `NOT_CONFIRMED` | Evidence contradicts the claim (wrong venue/dates, cancelled, no-show, wrong guest) |
| `INSUFFICIENT_EVIDENCE` | Page empty, unrelated, or too ambiguous |

Validators agree on the **verdict**; the wording of the reason may differ
(Equivalence Principle, `prompt_comparative`). The prompt treats the page as
untrusted data to resist prompt injection, and GenLayer's greyboxing (diverse
validator models) adds a second layer.

Example uses: affiliate commission qualification, supplier-vs-seller "did the
stay happen" disputes, loyalty or credit eligibility.

## Files

- `contracts/visit_verifier.py` - the contract (GenVM `v0.2.16`, pinned runner hash)
- `evidence/` - fake receipts for testing (`valid-stay`, `wrong-dates`, `cancelled`, `injection`)

## Run it in GenLayer Studio

1. Open <https://studio.genlayer.com>, upload `contracts/visit_verifier.py`.
2. Run and Debug -> Deploy (no constructor args).
3. `submit_claim` with: `ABC123`, `Hotel Del Coronado`, `hotel`, `2026-11-01`,
   `2026-11-03`, and a public URL to one of the files in `evidence/`
   (raw GitHub URL; the repo must be public for validators to fetch it).
4. Read the result with `get_claim(0)` or `is_confirmed(0)`.

Expected: `valid-stay` -> CONFIRMED; `wrong-dates` and `cancelled` ->
NOT_CONFIRMED; `injection` -> NOT_CONFIRMED (ignores the embedded instruction).

## Notes

- The header must pin `py-genlayer` to a hash; `:latest` is rejected outside debug mode.
- Real evidence is usually behind logins. A production version would have the
  supplier publish a signed/hashed receipt page, or restrict to known domains.
- Avoid putting real guest PII in public test evidence.
