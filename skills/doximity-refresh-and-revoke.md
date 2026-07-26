---
name: Refresh and revoke a Doximity session
description: Keep a Doximity access token current with the refresh_token grant and cleanly end a session with token revocation and RP-initiated logout.
api: openapi/doximity-oauth-openapi.yml
operations: [token, revoke, logout]
---

# Refresh and revoke a Doximity session

Maintain and tear down an authenticated Doximity session obtained via the verify-member flow.

## Steps
1. **Refresh the access token** — `token` (POST /oauth/token) with HTTP Basic
   `client_id:client_secret`, `grant_type=refresh_token`, `refresh_token`, and optionally `scope`.
   Do this before `expires_in` (~1800s) elapses. An expired/used refresh token returns
   `invalid_grant` (400) — restart the Authorization Code flow.
2. **Revoke on sign-out** — `revoke` (POST /oauth/revoke) with HTTP Basic auth and `token=<the
   access or refresh token>` to immediately invalidate it.
3. **RP-initiated logout** — `logout` (GET /oauth/logout) with `id_token_hint`,
   `post_logout_redirect_uri`, and `state` to end the Doximity session and return the member to
   your app.

## Rules
- Client authentication at `/oauth/token` and `/oauth/revoke` is HTTP Basic (`client_secret_basic`);
  bad credentials return `invalid_client` (401).
- Never store `client_secret` unencrypted or expose it in a browser/mobile client.
- Refresh tokens rotate access; revoke both on logout to fully end the session.
