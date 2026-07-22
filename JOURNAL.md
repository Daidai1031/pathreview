## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/89

**Issue title:** API reference doc is missing the `POST /profiles` request body schema

**Tier:**  [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**

The API reference file `docs/API.md` documents the response shape for each
endpoint, but it never describes what data a client must send in the request
body for the two POST endpoints: `POST /profiles` and `POST /reviews`. Because of
this gap, a developer reading the docs can't tell which fields are required,
what types they should be, or what a valid example request looks like. A
successful fix adds request body schemas for both endpoints, with a description
for each field and realistic example values, so the documentation matches the
detail already provided for responses.

**Selection notes:**

- Tier fit:
This is my first contribution to a large codebase, so I'm
deliberately choosing a Tier 1 issue. It's docs-only and self-contained, which
matches where I am right now.

- Codebase readiness: I traced both endpoints to their request models:
`POST /profiles` uses ProfileCreate (`api/schemas/profile.py`), which accepts two
optional fields: github_username and portfolio_url (both strings). `POST /reviews`
uses ReviewCreate (`api/schemas/review.py`), which requires a single field,
profile_id (a UUID). I read these Pydantic classes directly, so the schemas I
document will match the fields the API actually accepts. Note: this is a
documentation-only change with no code behavior, so there is no unit test to
modify; my verification is checking the documented fields against these request
models.

- Scope and time: Estimated 2–3 hours per the issue, which fits
comfortably in the Weeks 8–9 window alongside my other commitments. I checked
the issue comments and the ledger's Claims count and I'm fine with 14 people are on it. The issue lists no blockers or dependencies.

- All boxes reasonably satisfied (with the test-file caveat noted
above), so this issue is a realistic, well-scoped Tier 1 choice for me.


**Branch name:** docs/89-add-post-request-body-schemas

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger