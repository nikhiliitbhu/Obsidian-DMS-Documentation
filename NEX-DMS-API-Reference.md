# DMS · Integration Reference — **ISKCON Noida Expressway (NEX)**

Everything the NEX landing-page team needs to pull categories, festivals, donors,
birthdays, and donation redirects from **NEX's** DMS backend.

- **Backend:** shared multi-tenant DMS (one codebase, own container + DB per temple)
- **This tenant (NEX):** container `isk-nex-prd-dms` on host `whf-gnw-prod-nex-prod`
  (`13.204.247.135`), database `isk_nex_prod_donation`, public domain
  **`seva.nex.myiskcon.org`**
- **Reference client:** iskconnoida.org (TanStack Start) + companion Expo app —
  same API contract, different tenant

> This is the NEX-specific edit of the generic DMS API Reference. Every value below
> is NEX's own — do **not** reuse ISKCON Noida's `iic.iskconnoida.org` URL or key.

---

## §1 Overview

NEX runs on the same Donation Management System (DMS) backend as the other ISKCON
temples — same API contract, own container and database. You don't need to reuse the
DMS repository; build the landing page in whatever stack you like and call the
endpoints below directly.

Verified against the live backend at `routes/api/v1/routes.php` and
`app/Providers/RouteServiceProvider.php` (API prefix `api`, version group `v1`).

## §2 Authentication & base URL

> **✅ Ready-to-use credentials (already provisioned & verified working 2026-07-07).**
> The NEX service account and token already exist — you do **not** need to mint anything.
> Drop these straight into the landing page's build env:
> ```
> VITE_API_URL=https://seva.nex.myiskcon.org/api/v1
> VITE_API_KEY=1|ziDdUNe2egBQuQMp3eoZB2XsPw0bbhuiegNzvoecde262d4d
> ```
> Backing account: `landing@nex.myiskcon.org` (DMS user id 37) with `category.manage`.
> Confirmed returning live `data` from `/categories`, `/donations`, and `/birthdays`.
> Internal-team note: this token authenticates as that account — keep it to internal
> deploys; if it's ever exposed publicly, rotate it (see the mint command below).

> **Correction to the generic PDF.** The generic reference calls this a "static API
> key" pulled from an env var. That is **not** how this backend works. Verified against
> the live code (`config/auth.php`, `app/Http/Middleware/UseApiGuard.php`,
> `AuthController@token`): the API uses **Laravel Sanctum**, and the "key" is a
> **personal access token minted by logging in** — there is no `API_KEY` in NEX's
> `.env` (a live `grep` of NEX's container `.env` on 2026-07-07 returned only
> `APP_URL`, no key/token var).

**Exactly how the reference frontend connects** (verified against the real repo
`IskconNoida/iskconnoida.org` — `frontend/src/main/utils/handler.ts` and
`frontend/.env.example`): it reads **two env vars** and sends the key as a bearer token
on every call. There is **no runtime login** — a pre-minted token is baked into the build.

```
VITE_API_URL = https://seva.nex.myiskcon.org/api/v1   # NOTE: includes /api/v1
VITE_API_KEY = 1|ziDdUNe2egBQuQMp3eoZB2XsPw0bbhuiegNzvoecde262d4d                    # sent as: Authorization: Bearer ${VITE_API_KEY}
```

- `VITE_API_URL` **must end in `/api/v1`** — the code calls `VITE_API_URL + "/categories"`,
  so the prefix lives in the env value (Noida's is `https://iic.iskconnoida.org/api/v1`).
- `VITE_API_KEY` is a **static, pre-minted Sanctum token** embedded client-side. You only
  need NEX ops to hand you that one token string (below).

```
GET https://seva.nex.myiskcon.org/api/v1/categories
Authorization: Bearer 1|ziDdUNe2egBQuQMp3eoZB2XsPw0bbhuiegNzvoecde262d4d
```

> **How to get NEX's token (ops).** It is issued per user account, stored in NEX's DB
> (`personal_access_tokens` table), not in any env file. Two ways:
> 1. **Log in a dedicated read-only service account** via the `POST /login` call above
>    (create that account first in NEX's DMS admin panel; give it only the roles
>    needed to read categories/donors/birthdays).
> 2. **Mint one directly on the box:**
>    ```bash
>    ssh -i ~/.ssh/nex ubuntu@13.204.247.135 \
>      "sudo docker exec isk-nex-prd-dms php artisan tinker --execute=\
>      \"echo \Vanguard\User::where('email','SERVICE@nex')->first()->createToken('landing-page')->plainTextToken;\""
>    ```
>    (Production access — expect an approval prompt. Treat the printed token as a
>    secret.)

> **Required permission on the token account (verified 2026-07-07).** The token account
> must hold the permission **`category.manage`** — the `/categories` endpoint enforces it
> (`CategoryController` → `CheckPermissions` → `403 "Forbidden."`); `/donations` and
> `/birthdays` enforce none. Confirmed end-to-end on NEX: service account
> `landing@nex.myiskcon.org` (user id 37, role `User` + `category.manage`) returns live
> `data` from all three endpoints. This matches how Noida's landing account is scoped.
> Grant it with: `Vanguard\User::find(37)->permissions()->syncWithoutDetaching([9]);`
> (permission id 9 = `category.manage`).

> **⚠ Security — read this before shipping.** These `/api/v1` endpoints sit behind the
> `auth` + `verified` middleware with the Sanctum guard. A token minted by
> `createToken()` here carries the **full abilities of that user account** (`*`), not a
> read-only "public data" scope. So a token embedded in a browser bundle (the way the
> reference frontend ships `VITE_API_KEY` / `EXPO_PUBLIC_API_KEY`) is a live,
> full-access user credential readable in devtools. Mitigate by either:
> - **fetching server-side** (proxy the DMS calls from your own backend, keep the token
>   off the client), or
> - **restricting the token's abilities** to read-only (`createToken('landing', ['read'])`)
>   *and* confirming the controllers actually enforce that ability, or
> - using a **strictly least-privilege service account** whose role can only read the
>   three public lists.
>
> Do **not** reuse another tenant's (e.g. Noida's) token, and do not commit the token.

The "log in" experience your visitors get is a separate redirect into the DMS web app
itself (see §5) — different from this token, which is your app's own service credential.

## §3 Endpoints

All read-only (GET), returning JSON as `{ data: ..., meta?: ... }`. Paths are
relative to `https://seva.nex.myiskcon.org/api/v1`.

### `GET /categories` — list categories
Umbrella object for donation causes, festivals/events, and one-off info content.
Everything that renders as a "card" is a category.

**Query filter**
- `?filter[category_type]=service` — donation categories / causes
- `?filter[category_type]=festival` — festivals & events

> Confirm the exact `category_type` strings **NEX's** admin panel uses before
> filtering — `service`/`festival` are conventions, not an enforced enum.

**Response (illustrative)**
```json
{
  "data": [
    {
      "id": "42",
      "title": "Annakut Seva",
      "img": "https://.../annakut.jpg",
      "description": "<p>Sponsor a plate...</p>",
      "footer_img": "https://.../footer.png",
      "footer_text": "Every offering matters",
      "category_type": "service",
      "donate_link": "https://seva.nex.myiskcon.org/donate/42",
      "date_from": "2026-08-10",
      "date_to": "2026-08-10"
    }
  ]
}
```

### `GET /categories/{id}` — category detail
Single-category detail, reused for donation-category, event/festival, and CSR pages
alike. Same shape, wrapped as `{ data: <category> }`.

### `GET /donations` — list **donors**
Naming quirk: despite the path, this returns **donor records** (people, for a "wall of
gratitude" carousel), not transactions or amounts. Name your client function for what
it returns (e.g. `getDonors()`).

```json
{
  "data": [
    {
      "id": 104,
      "name": "Radha D.",
      "avatar": "https://.../avatar.jpg",
      "category": "Annakut Seva",
      "sub_category": "",
      "city": "Noida",
      "created_at": "2026-06-21T10:04:00Z"
    }
  ]
}
```

### `GET /birthdays` — list birthdays · paginated
Returns `{ data: [...], meta: { last_page, ... } }`. No server-side "get everything" —
walk `?page=2`, `?page=3` … up to `meta.last_page` and concatenate client-side.

**Item shape**
```json
{ "id": 7, "name": "...", "avatar": "...", "birthday": "1998-08-03", "city": "..." }
```

## §4 Data models

**Category**

| field | type | notes |
|---|---|---|
| id | string | stable, tenant-scoped — not globally unique |
| title | string | |
| img | string | image URL |
| description | string | rich text; render with a markdown/HTML renderer |
| footer_img | string | |
| footer_text | string | |
| category_type | string | e.g. `service`, `festival` — confirm NEX's values |
| donate_link | string? | present only when the category accepts donations |
| date_from | string | ISO date; used to sort/filter upcoming festivals |
| date_to | string | |

**Donor** (from `/donations`)

| field | type | notes |
|---|---|---|
| id | number | |
| name | string | |
| avatar | string | |
| category | string | which cause they gave to |
| sub_category | string | |
| city | string | can be empty — handle blank gracefully |
| created_at | string | ISO timestamp |

**Birthday** (from `/birthdays`)

| field | type | notes |
|---|---|---|
| id | number | |
| name | string | |
| avatar | string | |
| birthday | string | ISO date |
| city | string | |

## §5 Donate & login redirects

Separate from the JSON API — sending visitors into NEX's DMS web app to donate or log in.

1. **Per-category donate link** — every category can carry a `donate_link`, a
   ready-to-use URL into that category's donation page. Render it directly; for NEX
   these point at `https://seva.nex.myiskcon.org/...`.

2. **"Log in to DMS" link** — a static link to NEX's DMS web-app root:

   ```
   https://seva.nex.myiskcon.org/
   ```

   NEX is a **single-domain** tenant, so you do **not** need the reference site's
   `DMS_URL_BY_HOST` / `toDmsHost()` hostname-mapping helper (that solves a
   two-domain problem specific to ISKCON Noida). Just hardcode the URL above.

## §6 Tenant-specific gotchas (NEX)

- **Base URL:** `https://seva.nex.myiskcon.org/api/v1` — not `iic.iskconnoida.org`.
- **DMS web-app root:** `https://seva.nex.myiskcon.org/`.
- **Hardcoded category IDs:** the reference homepage pins its campaign banner to
  category id `216` (Noida's dataset). Find NEX's equivalent category in NEX's admin
  panel and use that ID.
- **`category_type` values:** confirm the exact strings NEX's categories are tagged
  with before filtering.
- **Skip the two-domain mapping:** NEX serves one domain — don't port
  `DMS_URL_BY_HOST` / `toDmsHost()`.
- **Client-exposed token (not a public key):** the "key" is a Sanctum user token with
  full account abilities (§2), not a read-only public key. If you render client-side it
  ships in the JS bundle — prefer server-side fetching or a least-privilege service
  account with restricted abilities.
- **Clone-drift caution:** NEX is a clone tenant that has historically drifted behind
  Noida on migrations/permissions. If an endpoint 500s on NEX but works on Noida,
  suspect tenant drift, not the API contract.

## §7 Quick-start client

Framework-agnostic TypeScript wrapper covering all five endpoints. Swap in your env
var names; keep the key out of source control.

```ts
// dms-client.ts  (NEX)
const API_URL = import.meta.env.VITE_API_URL;  // https://seva.nex.myiskcon.org/api/v1  (must include /api/v1)
const API_KEY = import.meta.env.VITE_API_KEY;  // static Sanctum token, sent as Bearer — same as the reference frontend

async function fetchJson(path: string) {
  const url = `${API_URL.replace(/\/+$/, "")}/${path.replace(/^\/+/, "")}`;
  const res = await fetch(url, {
    headers: { Authorization: `Bearer ${API_KEY}` },
  });
  if (!res.ok) throw new Error(`DMS request failed ${res.status} for ${url}`);
  return res.json();
}

// categories, optionally filtered by category_type
export const getCategories = (categoryType?: string) =>
  fetchJson(categoryType ? `/categories?filter[category_type]=${categoryType}` : "/categories")
    .then((res) => res?.data ?? []);

export const getCategoryById = (id: string) =>
  fetchJson(`/categories/${id}`).then((res) => res?.data ?? null);

// note: path is "/donations" but the payload is donor records
export const getDonors = () =>
  fetchJson("/donations").then((res) => res?.data ?? []);

export const getBirthdays = async () => {
  const first = await fetchJson("/birthdays");
  let all = first.data;
  for (let page = 2; page <= first.meta.last_page; page++) {
    const next = await fetchJson(`/birthdays?page=${page}`);
    all = [...all, ...next.data];
  }
  return all;
};
```

**Usage**
```ts
const festivals       = await getCategories("festival");
const donationCauses  = await getCategories("service");
const allDonors       = await getDonors();
const allBirthdays    = await getBirthdays();
const annakutDetail   = await getCategoryById("42");
// render annakutDetail.donate_link directly as the "Donate" button href
```

`.env` for the landing page (these are the **real var names** the `iskconnoida.org`
frontend uses — `frontend/.env.example`):
```
VITE_API_URL=https://seva.nex.myiskcon.org/api/v1
VITE_API_KEY=1|ziDdUNe2egBQuQMp3eoZB2XsPw0bbhuiegNzvoecde262d4d
```

---

*Compiled from the live DMS backend (`routes/api/v1/routes.php`,
`RouteServiceProvider.php`, `config/auth.php`, `UseApiGuard.php`, `AuthController@token`)
and NEX infra topology, verified 2026-07-07. Every value is NEX's own and embedded here
**except the Sanctum token** (§2), which is a per-account credential minted at login and
must be issued by NEX ops — not a static env key. A live check of NEX's container `.env`
confirmed no API key/token var exists there.*
