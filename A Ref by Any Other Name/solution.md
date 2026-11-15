# Security review solution

## Vulnerability

The `branch` and `tag` steps in [`files/action.yml`](./files/action.yml) take branch and tag refs from GitHub event data. Those refs can be chosen by an attacker who can create a branch, tag, or pull request. The steps escape the values with `printf "%q"` but then pass them to `eval` when writing outputs. `eval` parses the resulting string as shell code, allowing crafted ref names to execute commands on the GitHub Actions runner.

This is command injection (CWE-77). Successful exploitation can compromise the runner and expose workflow credentials or secrets, depending on the workflow's permissions and event context.

## References

- [GitHub Security Advisory GHSA-gq52-6phf-x2r6 (CVE-2025-54416)](https://github.com/tj-actions/branch-names/security/advisories/GHSA-gq52-6phf-x2r6)
- [Fix commit e497ceb](https://github.com/tj-actions/branch-names/commit/e497ceb8ccd43fd9573cf2e375216625bc411d1f)

The fix replaces each `eval`-based output with a direct `printf` using a quoted value argument, so the shell does not re-interpret the ref as code.
