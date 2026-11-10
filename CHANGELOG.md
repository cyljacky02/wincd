# Release notes

## v0.1.2 — pre-release

Fixes starting `wincd.ps1` with `powershell.exe -File` and adds the Traditional Chinese installation guide to the samples pack.

### Fixed

- In Windows PowerShell 5.1, `powershell.exe -File` now starts `wincd.ps1`. In v0.1.1 and earlier it failed before any command ran, because the default installation root could not be determined. Starting it with the call operator behaves as before, and an installed engine still refuses another `-Root`.

### Added

- `INSTALL.md` in `wincd-samples-0.1.2.zip`: the Traditional Chinese installation and update guide, identical to this repository's INSTALL.md. It covers choosing the installation root, for example `D:\WinCD`; `engine`, `inbox`, and `data` always stay under that root.

### Changed

- Engine version 0.1.2. Apart from the `wincd.ps1` fix, behavior, Deployment Profile rules, and data formats are identical to v0.1.1.
- The samples walkthrough links to the `INSTALL.md` inside the pack. The demos and the Deployment Profile reference are unchanged.

### Evaluation limitations

Unchanged from v0.1.1: native Windows/TUI acceptance is incomplete; no Windows Server matrix, production-workload, or real-F5 certification is claimed; no Authenticode signature or build provenance attestation is provided. Use the demo on a test machine only.

### Validation of these packages

- Automated suite (fake host and TEMP directories) on Windows PowerShell 5.1 with Pester 3.4.0: 1660 passed, 0 failed, 1 skipped. The skipped test compiles the monitor handler against IIS's Microsoft.Web.Administration and needs IIS on the test host.
- Both packages verified file-for-file against the built source. From the extracted engine, `status` succeeded through both the call operator and `powershell.exe -File`, and both demo ZIPs passed a read-only plan. The INSTALL.md commands ran as written in an elevated session: install, `status`, update, `status`.

This is not a production-readiness declaration.

### License

Apache License 2.0. Bundled YamlDotNet remains under its MIT license.

## v0.1.1 — pre-release

Adds a samples pack so new users have a runnable starting point. The engine is unchanged from v0.1.0 apart from its reported version.

### Added

- `wincd-samples-0.1.1.zip`, a separate download built from the same source as the engine:
  - `README.md`: a test-machine walkthrough (Traditional Chinese): deploy v1, deploy v2, roll back, clean up, with the equivalent command-line commands.
  - `demo/wincd-demo-v1.zip`, `demo/wincd-demo-v2.zip`: deployment content for project `wincd-demo` that needs no Tomcat, IIS, or F5 and writes only to `C:\WinCD-demo\`. Demonstrates Mirror deletion with a preserve list, a single-file mapping, an ordered wait, a one-shot command, and version labels.
  - `reference/order-api/`: the fully commented Deployment Profile reference, with alternatives for IIS sites, process services, checks, and F5 modes.

### Changed

- Engine version 0.1.1. Behavior, Deployment Profile rules, and data formats are identical to v0.1.0.

### Known issues

- In Windows PowerShell 5.1, starting `wincd.ps1` with `powershell.exe -File` fails before any command runs, because the default installation root cannot be determined. Use the call operator as documented; from cmd.exe, for example: `powershell.exe -NoProfile -Command "& 'C:\WinCD\engine\wincd.ps1' status; exit $LASTEXITCODE"`. v0.1.0 behaves the same.

### Evaluation limitations

Unchanged from v0.1.0: native Windows/TUI acceptance is incomplete; no Windows Server matrix, production-workload, or real-F5 certification is claimed; no Authenticode signature or build provenance attestation is provided. Use the demo on a test machine only.

### Validation of these packages

- Automated suite (fake host and TEMP directories): 1655 passed, 0 failed, on Windows PowerShell 5.1 with Pester 3.4.0. This includes deploying demo v1, deploying v2, and rolling back through the public commands in TEMP directories.
- Both packages verified file-for-file against the built source. From the extracted engine, a read-only `status` succeeded and both demo ZIPs passed a read-only plan.

This is not a production-readiness declaration.

### License

Apache License 2.0. Bundled YamlDotNet remains under its MIT license.

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
