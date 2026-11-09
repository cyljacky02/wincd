# Release notes

## v0.1.0 — draft pre-release

Initial distribution candidate for the WinCD lightweight Deployment Engine and its Traditional Chinese TUI / PowerShell command-line Front-ends.

### Included

- One-project, one-host deployment, rollback, and interrupted-transaction recovery.
- File mappings and directory mappings with Overlay or Mirror behavior.
- Explicit Services, Steps, and ordered stop/start actions; one-shot commands, position-specific checks, and waits.
- Three-file YAML Deployment Profiles and an included YAML parser.
- Local traffic control through an IIS monitor endpoint or legacy health file, with explicitly configured Up/Down response bodies.
- Offline copy-folder installation; no online package manager required on the installed host.

### Evaluation limitations

- Native Windows/TUI acceptance for this exact candidate is not complete.
- No complete Windows Server validation matrix, production-workload validation, or real-F5 certification is claimed.
- Local traffic intent is not proof of application health, successful connection draining, or actual F5 member state.
- No automatic migration or compatibility with older development snapshots is promised.
- WinCD distribution licensing and final release review are pending. No runtime release has been published.

The draft is not a production-readiness declaration. The final release notes will record the validation actually completed for the published package.
