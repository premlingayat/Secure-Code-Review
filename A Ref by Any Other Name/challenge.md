**Company:** Streamline DevOps — publishers of small, widely-used composite
GitHub Actions.

**Project:** **RefNamer** — a composite GitHub Action that normalizes the
current Git ref into clean branch/tag names and exposes them as outputs
(`base_ref_branch`, `head_ref_branch`, `ref_branch`, `tag`, `current_branch`).
Teams embed it in CI workflows for builds, deploy gates, and artifact naming,
so its output is often trusted by later steps.

**Task:** A CodeQL-driven advisory flagged this Action as exploitable: a ref
name chosen by an attacker can cause **arbitrary command execution on any CI
runner that uses the Action**. Audit the bash run blocks and find where an
attacker-influenced value is handed to the shell.

**Scope note:** Focus on `action.yml` — the composite-run bash steps named
`branch` and `tag`. There is one file in this challenge.
