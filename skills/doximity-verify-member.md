---
name: Verify and identify a Doximity member
description: Run the OAuth 2.0 Authorization Code + PKCE flow to authenticate a Doximity medical professional and read their verified identity claims.
api: openapi/doximity-oauth-openapi.yml
operations: [authorize, token, userinfo]
---

# Verify and identify a Doximity member

Use this to let a verified U.S. medical professional sign in with Doximity and to read their
identity/profile claims. Doximity is an identity provider — it authenticates and identifies members
against the Doximity medical database; it does not allow broad querying of the directory.

## Prerequisites
- Register at https://www.doximity.com/developers/api_signup to obtain `client_id` / `client_secret`.
- PKCE is **required** (S256). Generate a `code_verifier` and its `code_challenge` per request.
- All requests must use HTTPS.

## Steps
1. **Start authorization** — `authorize` (GET https://auth.doximity.com/oauth/authorize).
   Send `client_id`, `response_type=code`, `redirect_uri`, `scope=openid profile:read:basic`
   (add `profile:read:email` etc. as needed), a unique `state`, `code_challenge`, and
   `code_challenge_method=S256`. Redirect the member's browser here.
2. **Handle the callback** — On approval Doximity redirects to `redirect_uri` with `code` and the
   echoed `state`. Verify `state` matches; on denial you receive `error=access_denied`
   (see errors/doximity-problem-types.yml).
3. **Exchange the code** — `token` (POST /oauth/token) with HTTP Basic `client_id:client_secret`,
   `grant_type=authorization_code`, `code`, `redirect_uri`, and the `code_verifier`. You receive
   `access_token`, `id_token` (RS256 JWT), `refresh_token`, and `expires_in` (~1800s).
4. **Validate the id_token** — verify the RS256 signature against the JWKS at
   `/.well-known/jwks.json`, and check `iss`, `aud`, `exp`, and `nonce`.
5. **Read claims** — `userinfo` (GET /oauth/userinfo) with `Authorization: Bearer <access_token>`.
   Returns `sub`, `name`, `credentials`, `specialty`, and any scope-gated claims.

## Rules
- Only request scopes you were granted; requesting an ungranted scope yields `invalid_scope`.
- Missing scope on `userinfo` returns `insufficient_scope` (403) — request the right `profile:read:*`.
- Respect the 5000 req/hour limit; back off on `429` until `X-Rate-Limit-Reset`
  (rate-limits/doximity-rate-limits.yml).
