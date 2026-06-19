---
tags: [api, reference]
aliases: [API, API Reference, Endpoints]
updated: 2026-06-19
---

# API Reference

The DMS exposes a JSON API under `routes/api/` (with a versioned `api/v1/routes.php`). Auth is **Laravel Sanctum** tokens — see [[auth-rbac]]. Controllers live in `app/Http/Controllers/Api/`.

> [!note] Verify before relying
> Routes below are transcribed from `routes/api/api.php` + `v1/routes.php` as of 2026-06-19. Endpoints evolve — confirm against the route files (`php artisan route:list`) before integrating.

## Auth

| Method | Path | Handler |
|--------|------|---------|
| POST | `login` | `Auth\AuthController@token` / `AuthApiController@login` |
| POST | `login/social` | `Auth\SocialLoginController@index` |
| POST | `logout` | `Auth\AuthController@logout` |
| POST | `register` | `Auth\RegistrationController@index` |
| POST | `password/remind` | `Auth\Password\RemindController@index` |
| POST | `password/reset` | `Auth\Password\ResetController@index` |
| POST | `email/resend`, `email/verify` | `Auth\VerificationController` |

## Account & users

| Method | Path | Purpose |
|--------|------|---------|
| GET / PATCH | `me`, `me/details`, `me/details/auth` | Own profile |
| POST / DELETE / PUT | `me/avatar`, `me/avatar/external` | Own avatar |
| GET | `me/sessions` | Own sessions |
| PUT / POST / DELETE | `me/2fa`, `me/2fa/verify` | Own 2FA |
| GET | `users`, `users/{userId}` | List / show users |
| POST/PUT/DELETE | `users/{user}/avatar`, `…/2fa`, `…/sessions` | Manage a user |
| GET | `stats` | `StatsController@index` |

## Donations & catalog

| Method | Path | Handler |
|--------|------|---------|
| GET | `categories`, `categories/{category}` | `CategoryController` |
| POST | `category`, `sub-category`, `category-data` | `CategoryController` |
| GET | `donations` | `DonationController` (list) |
| GET | `birthdays` | `BirthdayController` |
| POST | `users-list`, `users` | `CategoryController` user lookups |
| POST | `log_in` | `CashController@login` (portal/cash auth) |

See [[donation-flow]] for how donations are created and [[payments]] for gateways.

## Sadhna app

A separate mobile-app surface (spiritual-practice tracking) under `Api/Sadhna/`. Auth uses phone/OTP.

| Method | Path | Handler |
|--------|------|---------|
| POST | `register`, `check-phone`, `check-email` | `Sadhna\Auth\RegisterController` |
| POST | `login`, `login-otp`, `verify-otp` | `Sadhna\Auth\LoginController` |
| GET / PATCH | `me` | `Sadhna\Auth\ProfileController` |
| POST | `avatar`, `logout`, `logout-all` | `Sadhna\Auth\ProfileController` |
| (apiResource) | `practices` | `Sadhna\SadhnaController` |
| GET / POST / PUT | `preferences` | `Sadhna\PreferencesController` |
| GET | `graph-data` | `Sadhna\SadhnaController@graphData` |

Backed by `sadhna_daily_practices` + `sadhna_user_preferences` ([[domain-model#Entity reference]]).

## Filtering

List endpoints use **`spatie/laravel-query-builder`** — expect `?filter[...]`, `?sort=`, `?include=` query params (see that package's docs).

## See also
[[auth-rbac]] · [[donation-flow]] · [[payments]] · [[architecture-overview]]
