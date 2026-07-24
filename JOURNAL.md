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

---

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** 

**Reproduction summary:**
I ran the API locally and exercised both POST endpoints through the OpenAPI UI at
`localhost:8000/docs`. `POST /profiles` turned out to take `multipart/form-data`
with an undocumented `resume_file` upload, while `POST /reviews` takes JSON — yet
`docs/API.md` describes both with identical one-line entries and no request format
at all, so the two are indistinguishable to a reader.

**PLAN.md link:** 

**Walkthrough video (recommended):**

**Blockers or open questions:**
The issue body says the response schemas are already documented, but `docs/API.md`
has neither requests nor responses — I need to confirm with the maintainer whether
responses are in scope. Separately, two of the error paths I tested return 500 for
what is clearly invalid client input; that looks like a bug in the handlers'
exception handling, but it is a code change and I don't think it belongs in this
docs issue.

---

### Reproduction detail

Everything below was observed against the API running locally at
`http://localhost:8000`. Bearer tokens are redacted.

#### What `docs/API.md` says today

The Profiles and Reviews sections in full:

```
### Profiles

`POST /profiles` — Create a profile with resume and GitHub username.
`GET /profiles/{profile_id}` — Retrieve a profile.
`DELETE /profiles/{profile_id}` — Delete a profile and associated data.

### Reviews

`POST /reviews` — Request a new portfolio review for a profile.
`GET /reviews/{review_id}` — Retrieve a completed review.
`GET /reviews` — List reviews for the authenticated user (paginated).
```

One line per endpoint. No content type, no field names, no required/optional
information, no example request, and no mention that a bearer token is needed.
A reader cannot construct a valid request from this.

#### `POST /profiles` takes `multipart/form-data`, not JSON

```
curl -X 'POST' \
  'http://localhost:8000/profiles' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: multipart/form-data' \
  -F 'github_username=daidai1031' \
  -F 'portfolio_url=https://www.daidingrdesigns.com/' \
  -F 'resume_file='
```

`200 OK`:

```json
{
  "id": "39539042-b11e-4fa1-9f7a-068398cba5eb",
  "user_id": "cc258f35-98a3-457b-b1bb-6fec6c95f9a9",
  "github_username": "daidai1031",
  "portfolio_url": "https://www.daidingrdesigns.com/",
  "created_at": "2026-07-24T05:31:40.029680Z",
  "resume_filename": null
}
```

Three things here appear nowhere in `docs/API.md`:

1. The content type is `multipart/form-data`. The doc gives no reason to expect
   this; the default assumption for a POST endpoint is JSON.
2. There is a third field, `resume_file`, a file upload. The doc mentions "resume"
   in prose but never names the field or says it is a file.
3. The endpoint requires `Authorization: Bearer <token>` from `POST /auth/login`.

Source of truth: `create_profile_endpoint` in `api/routes/profiles.py`, whose
parameters are declared as `Form(default=None)` and `UploadFile = File(default=None)`.
`ProfileCreate` is constructed inside the handler; it is not the request body model.

#### `POST /reviews` takes `application/json`

Request body, marked **required** in the OpenAPI schema:

```json
{
  "profile_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

With a `profile_id` that exists, `200 OK` and the review is created with
`status: "pending"`; the analysis then runs in the background:

```json
{
  "id": "4be36d29-a966-418d-ac6b-080e0268081d",
  "profile_id": "39539042-b11e-4fa1-9f7a-068398cba5eb",
  "status": "pending",
  "sections": null,
  "overall_score": null,
  "error_message": null,
  "created_at": "2026-07-24T06:06:45.606040Z",
  "updated_at": "2026-07-24T06:06:45.606046Z"
}
```

Source of truth: `create_review_endpoint` in `api/routes/reviews.py` takes
`data: ReviewCreate`; `ReviewCreate` in `api/schemas/review.py` declares one
required field, `profile_id: UUID`.

#### Error responses — observed, not assumed

| Scenario | Status | Response body |
| --- | --- | --- |
| `POST /profiles` with no bearer token | 401 | `{"detail": "Not authenticated"}` |
| `POST /profiles` with a `.docx` resume | 422 | `{"detail": "Resume must be a PDF or Markdown file"}` |
| `POST /profiles` with `github_username` over 255 chars | 500 | `{"detail": "Failed to create profile"}` |
| `POST /reviews` with a nonexistent `profile_id` | 500 | `{"detail": "Failed to create review"}` |

The 401 also carries a `www-authenticate: Bearer` response header. Swagger labels
all four as *Undocumented*, so they are missing from the OpenAPI schema as well as
from `docs/API.md`.

**On the two 500s.** Both are caused by invalid client input and would normally be
4xx. In `api/routes/profiles.py`, an over-length `github_username` arrives as an
unconstrained `Form` string and only fails when `ProfileCreate(...)` is constructed
inside the handler; that `ValidationError` is swallowed by the handler's generic
`except Exception` and re-raised as a 500. `api/routes/reviews.py` does the same
for a `profile_id` with no matching row.

The `.docx` case is the instructive contrast: that check raises `HTTPException`
explicitly, and the `except HTTPException: raise` branch passes it through intact,
so it correctly surfaces as 422. Two error paths in the same function, handled
differently.

Fixing the 500s is a code change and outside the scope of #89. I will document the
behavior as it currently stands, flag the discrepancy in the PR, and suggest a
separate issue.

#### Why this counts as a reproduction

The two POST endpoints take different content types — one form-data with a file
upload, one JSON — and `docs/API.md` describes them with structurally identical
one-line entries that omit the distinction entirely. A developer working from the
documentation alone cannot tell them apart, and the natural guess for `/profiles`
is wrong. All four error responses are undocumented as well.