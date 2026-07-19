---
tags: [ops, reference]
aliases: [Development, Local Setup, Getting Started]
updated: 2026-06-19
---

# Local Development Setup

How to get the DMS running locally. It's a standard **Laravel 9 / PHP `^8.0.2`** app (on [[architecture-overview#Built on Vanguard|Vanguard]]); the notes below reflect *this* repo's actual config.

## Prerequisites

- **PHP 8.0.2+** with `ext-json` + usual Laravel extensions (mbstring, openssl, pdo, tokenizer, xml, ctype, bcmath)
- **Composer**
- **MySQL** (production uses RDS `iic-prd-db` — see [[infrastructure]])
- **Node + npm** (front-end assets via Laravel Mix)
- **Redis** — optional locally; defaults below avoid it

## First-time setup

```bash
# 1. Install PHP deps (auto-copies .env.example -> .env, runs package:discover,
#    and key:generate on create-project)
composer install

# 2. Ensure .env exists and has an app key
cp -n .env.example .env
php artisan key:generate

# 3. Configure .env — see below

# 4. Create your DB, then migrate + seed
php artisan migrate
php artisan db:seed        # Countries, Roles, Permissions, Settings, User, SqlDirectory

# 5. Front-end assets
npm install
npm run dev                # or: npm run watch

# 6. Serve
php artisan serve          # http://127.0.0.1:8000
```

> [!note] Where the schema comes from
> The base schema is one large migration — `database/migrations/2018_09_24_234659_create_donation_base.php` — followed by incremental migrations. `SqlDirectorySeeder` loads SQL from `database/seeders/sql/`. See [[domain-model]].

## `.env` you must set

`.env.example` ships **production** defaults (`APP_ENV=production`, `APP_DEBUG=false`). For local work:

```dotenv
APP_ENV=local
APP_DEBUG=true
APP_URL=http://127.0.0.1:8000

# Database — NOT in .env.example, add the standard Laravel keys
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=dms
DB_USERNAME=root
DB_PASSWORD=

# Keep from .env.example — app-level encryption depends on these
APP_KEY=...                # php artisan key:generate
ENCRYPTION_KEY=base64:...  # used with CIPHER
CIPHER=AES-256-CBC
```

> [!danger] Two gotchas
> 1. **`DB_*` keys are missing from `.env.example`** — add them yourself (`config/database.php` reads them).
> 2. The committed `APP_KEY` / `ENCRYPTION_KEY` are **placeholders** — never reuse them in a real deployment.

### Per-temple / feature config

The shared codebase is customized per temple entirely through `.env` — full table in [[infrastructure#Per-temple configuration]]. Common keys: `NATIVE_TEMPLE_CITY`, `RECEIPT_PREFIX`, `USER_PREFIX`, `PINBOT_API_KEY`, `PAYMENT_REDIRECT_URL`, `PUSHER_APP_*`.

## Drivers (local-friendly defaults)

`.env.example` already uses safe local defaults: `BROADCAST_DRIVER=log`, `CACHE_DRIVER=array`, `QUEUE_DRIVER=sync`, `SESSION_DRIVER=database`. With `SESSION_DRIVER=database` you need the `sessions` table — created by the base migration, so a completed `migrate` covers it.

## Plugins

Three local Composer **path packages** in `plugins/` load automatically on `composer install`: `ActivityLog` · `Announcements` · `LaravelCountries` (see [[architecture-overview#Plugins]]).

## Branches / deployment

Work flows `development` → `uat` → `main`; Jenkins deploys. Details in [[infrastructure#Deploy pipeline]].

## See also
[[architecture-overview]] · [[domain-model]] · [[infrastructure]] · [[api-reference]]
