# Hints

## Rung 1
**Hint 1 (cheap):** The account-info endpoint is meant to update only
   profile fields. Look at exactly which array is passed to
   `$this->guard()->user()->update(...)` — is it a validated allow-list or
   something looser?

## Rung 2
**Hint 2 (narrower):** `$request->except(['_token'])` is not an
   allow-list; it's a deny-list with a single entry. Every other request
   field — including ones `AccountInfoRequest` never validates — flows into
   mass assignment on the user model.

## Rung 3
**Hint 3 (narrow):** The user model exposes `password` as a fillable
   attribute (it must, for registration and the password-change flow to
   work). Whatever keys survive `except()` are written straight to the row.

## Rung 4
**Hint 4 (spicy):** Vulnerable line:
   ```php
   $result = $this->guard()->user()->update($request->except(['_token']));
   ```
   POST the account-info route with an extra `password=NewPass123!` field
   (plus a valid `_token`) and the model setter hashes and stores it — no
   current-password check anywhere in this path.

## Rung 5
**Hint 5 (giveaway):** Exploit: as any authenticated user (or via CSRF/
   XSS against one), send `POST /my-account` with the normal profile fields
   **and** `password=<attacker-chosen>`. The login credential is replaced
   silently while the legitimate change-password flow — the only place that
   verifies the old password — is bypassed completely. An attacker with any
   temporary foothold in the session then owns the account permanently.
