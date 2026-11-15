# Contributing

Short version; the full working agreement is in [DEVCONTEXT.md](DEVCONTEXT.md).

## Branches and commits

- Branch from `main`: `feat/<module>-<name>`, `fix/<module>-<name>`, `docs/<name>`.
- Keep branches short-lived. Rebase or merge `main` often.
- Commit messages: imperative and scoped, e.g. `metabolism: enforce hard particle cap`.

## Pull request checklist

- [ ] Builds on a clean checkout; tests pass locally and in CI
- [ ] Touches only the module(s) you own, or the owner reviewed it
- [ ] New logic has unit tests (bounds, determinism, boundary cases)
- [ ] **No change to shared contracts** without approval from both other devs
- [ ] No unbounded allocation or container in hot paths
- [ ] No Qt/Win32 includes in `brain/` or `metabolism/`
- [ ] Missing telemetry represented as `valid=false`, never `0`
- [ ] Tuned constants commented as initial/untuned or backed by a measurement
- [ ] Docs updated (architecture, interfaces, organism model, status board)
- [ ] No invented numbers; synthetic data labeled as synthetic

## Review expectations

- Review within a working day where possible; small PRs make this easy.
- Review for: correctness, boundedness, determinism, contract adherence, test quality.
- `main` must stay green. If you break it, revert first and fix after.

## Reporting issues

Include: Windows version, hardware, build preset, steps to reproduce, whether synthetic or real telemetry was used, relevant log lines.
