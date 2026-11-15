# Hints

## Rung 1
**Cheap:** In `action.yml`, look at the `branch` step. Variables are built
   from `github.ref`, `github.base_ref`, and the PR head ref — all of which an
   attacker can influence via branch names. The output lines then run through
   `eval`.
