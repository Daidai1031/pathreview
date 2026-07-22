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

**Branch name:** docs/89-add-post-request-body-schemas

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger