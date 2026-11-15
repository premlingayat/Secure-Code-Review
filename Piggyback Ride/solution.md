# Vulnerability brief

## Account update mass assignment

The vulnerable version of `MyAccountController` updated account attributes using
`$request->except(['_token'])`. This passed essentially all submitted fields to
the model, rather than only fields validated for the account-information form.
If `password` was mass-assignable, an authenticated user could submit a new
password through that route without going through the current-password
verification flow, potentially taking over the account persistently.

The fix uses `$request->validated()` as the update data, restricting the
attributes to those accepted by the form's validation rules.

The reviewed challenge snapshot already contains this fix; this describes the
vulnerable upstream version and its remediation.

- GitHub Security Advisory: [GHSA-xpv2-hrfc-hw62 (CVE-2026-54175)](https://github.com/advisories/GHSA-xpv2-hrfc-hw62)
- Fix commit: [ffe9a43](https://github.com/Laravel-Backpack/CRUD/commit/ffe9a43ed006d0667e52997036c05ac0bc06cf91)
