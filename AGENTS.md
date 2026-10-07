# AGENTS.md — Zenedu API docs conventions

This repository is the **Mintlify documentation site for the Zenedu public REST API (`/api/v1`)**.
These conventions apply to **any agent or human** editing this repo (Claude, ChatGPT/Codex, Cursor, …).

> This file and `CLAUDE.md` are excluded from the Mintlify build via `.mintignore`.
> They are internal conventions and must **never** be published or appear on the docs site.

---

## Repository structure

- **`docs.json`** — site config: theme, navigation, and the OpenAPI reference.
  The spec is referenced as `"openapi": "api-reference/openapi.json"` — a **relative path with NO leading slash**
  (a leading slash breaks spec loading on the hosted build).
- **`api-reference/openapi.json`** — the **hand-maintained** OpenAPI 3.1 spec and the single source of truth for
  every endpoint's parameters, request body, responses, and schemas. There is **no generation pipeline**.
- **`api-reference/**/<action>.mdx`** — one page per operation, frontmatter only:
  ```mdx
  ---
  title: "Short title"
  description: "One sentence."
  openapi: "METHOD /path"   # e.g. PATCH /api/v1/bot/{bot}/settings/email
  ---
  ```
- Guides (`index`, `authentication`, `errors`) are plain MDX.

---

## Navigation ordering rules (`docs.json`) — the important part

The sidebar must stay consistent and tidy. **Two hard rules:**

### Rule 1 — order every cluster by HTTP method

Within any group **or** subgroup, order pages by method so the colored method badges cluster together
and never interleave:

```
GET → POST → PUT → PATCH → DELETE
```

- Never mix PUT with PATCH (or any two methods). All GETs first, then all POSTs, etc.
- Within the same method, keep a sensible order
  (GET: `list` → `get` → auxiliary reads like `catalog`/`events`; POST: `create` → `duplicate` → uploads; …).
- This is grouping by **HTTP method, not by semantics** — e.g. `Activate` (PUT) and `Deactivate` (DELETE)
  intentionally land in different clusters. That is correct.

### Rule 2 — two levels: resource first, sub-resources nested

- A resource's **own** endpoints (its CRUD and direct actions) live at the **group's top level** as direct pages.
- Only **sub-resources** go into nested **subgroups**. Each subgroup follows Rule 1 internally.
- **Do not** wrap a resource's own endpoints in a subgroup named after the resource.

Example — `Products`:

```
Products
  list, get, subscribers, create, duplicate, …, activate, update, …, delete   ← product's OWN endpoints, method-sorted
  ▸ Lessons           ← sub-resource subgroup (method-sorted)
  ▸ Lesson materials
  ▸ Lesson video
  ▸ Sections
  ▸ Steps
```

Groups that use this pattern today: Products, Funnels, Offers, Subscribers, Broadcasts, Channels, Groups,
Affiliate, Mini-app, Web, Folders.

---

## Adding or changing an endpoint

1. Add/update the operation in `api-reference/openapi.json`
   (path + method + parameters + requestBody + responses + any new `components/schemas`).
2. Create `api-reference/<resource>/<action>.mdx` with the frontmatter shown above.
3. Add the page to `docs.json` navigation **in its method-sorted position (Rule 1)** and **at the correct level (Rule 2)**.
4. Verify: `npx mint validate` and `npx mint broken-links`. Preview locally with `npx mint dev` (http://localhost:3000).

> Note: `mint validate` always prints one warning — `Error validating OpenAPI file /docs.json … version must be a string`.
> It is a harmless CLI quirk (it mis-scans `docs.json`); it is unrelated to your changes. The live site is built from git on push.

---

## OpenAPI spec conventions (match the existing entries)

- OpenAPI **3.1**, a single spec file. `servers` is `https://app.zenedu.com`. Bearer auth (Laravel Sanctum);
  every request needs `Accept: application/json`.
- **Responses:** list endpoints use `allOf` of `#/components/schemas/PaginatedResponse` + `{ data: [Item] }`;
  a single item is `{ data: Item }`. Error responses reference `#/components/schemas/Error` **inline**
  (this spec does not use shared `components/responses`).
- **Status codes reflect the code** — 201 create, 202 queued, 204 delete; 401 unauthorized, 403 lacks access,
  404 not found, 409 conflict, 422 validation. Do not invent codes that the controller/action can't return.
- **`operationId`:** camelCase, unique across the whole spec (prefix with the resource, e.g. `listCoupons`, `createCoupon`).
- Define each schema **once** in `components/schemas`; reference shared schemas
  (`Bot`, `Subscriber`, `Offer`, `Product`, `Error`, `PaginatedResponse`, …) by `$ref`.
- **Authorization is described via 401/403 only** — there is no Pro middleware on `/api/v1`.
  Keep an existing `(Pro)` note if one is already there; do not add new `(Pro)` notes.

---

## Don't

- Don't publish these agent files or let them reach the build — they stay in `.mintignore`.
- Don't add a leading slash to the `openapi` path in `docs.json`.
- Don't reorder pages in a way that interleaves HTTP methods, and don't bury a resource's own endpoints in a same-named subgroup.
