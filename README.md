# famexus-calendar

Famexus Calendar service. Google Calendar integration. Each user connects their Google account with OAuth 2.0; refresh tokens are encrypted at rest and events are synced on a schedule using incremental sync tokens. Only reachable from the gateway.

Laravel 13 on PHP 8.5, MySQL 8 (database `famexus_calendar`), Redis for queues and cache.

## Run it

The usual way is through [famexus-infra](../famexus-infra), which starts every service, MySQL, Redis and Mailpit with docker-compose.

To run this service on its own:

```sh
composer setup          # composer install, copy .env, generate key, migrate
composer dev            # serves on http://localhost:8002
```

Without the infra stack you need MySQL and Redis yourself, or set `DB_CONNECTION=sqlite` and `QUEUE_CONNECTION=sync` / `CACHE_STORE=file` in `.env`.

Check it is up:

```sh
curl http://localhost:8002/up
curl http://localhost:8002/api/v1/ping
```

`/api/v1/hello` only answers requests carrying a gateway-signed JWT, so try it through the gateway: `curl http://localhost:8000/api/v1/hello`.

## JWT keys

Requests arrive from the gateway with a short-lived RS256 JWT. Put the gateway's public key at `storage/keys/jwt-public.pem` (see the gateway README for how to generate it). Keys are git-ignored.

## Quality

```sh
composer test           # Pest
composer lint           # Pint (check only) + PHPStan level 6 via Larastan
composer format         # Pint, fixes formatting
```

GitHub Actions runs the same checks on every push to `main` and every pull request (`.github/workflows/ci.yml`).

## API

REST/JSON under `/api/v1`. The OpenAPI spec lives in [`docs/openapi.yaml`](docs/openapi.yaml); keep it in step with the routes.

## Roadmap

- [ ] JWT verification middleware (gateway public key)
- [ ] Google OAuth connect/disconnect, encrypted refresh tokens
- [ ] Scheduled incremental sync (Redis queue) into a local events table
- [ ] Endpoints for upcoming events and dates for the dashboard
- Note: a public launch needs Google app verification for the calendar scope (allow several weeks)
