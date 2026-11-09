# Release notes

## v0.1.0 — pre-release

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
- No Authenticode signature or build provenance attestation is provided.

### Validation of this package

- Automated suite (fake host and TEMP directories): 1640 passed, 0 failed, on Windows PowerShell 5.1 with Pester 3.4.0.
- Package contents verified file-for-file against the built source; read-only `status` smoke test from the extracted package succeeded.

This is not a production-readiness declaration.

### License

Apache License 2.0. Bundled YamlDotNet remains under its MIT license.
