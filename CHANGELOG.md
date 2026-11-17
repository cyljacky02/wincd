# Release notes

## v0.2.0 — pre-release

Adds the Update write mode and Delete Mappings, runs Mappings in declared order, and fixes how failed or interrupted transactions are recorded and reported. **Transaction and Plan data formats change: read "Upgrading" before updating an existing installation.**

### Upgrading from v0.1.x

- Frozen Plans and transaction records now use `schemaRevision: 5`. v0.2.0 refuses to recover, preview a recovery, or switch traffic from records written by v0.1.x. v0.1.x engines likewise refuse v0.2.0 records. Nothing is converted automatically.
- Before updating, finish every pending Recovery Run with the engine that is installed now, as INSTALL describes.
- After updating, deploy each project once with v0.2.0.

### Added

- `Mode: Update` for directory mappings: adds or replaces the files the release folder contains and never deletes anything in the destination. Use it when only changed files are delivered. Rollback does not remove files that Update added in a newer version.
- Delete Mappings (`Type: Delete`, `From` naming a TXT list in the release): remove the listed local files or folders. Paths that no longer exist are skipped. Deleted content is backed up and restored if the same deployment fails, but rollback does not bring it back. The engine refuses to delete its own installation or data folders, network paths, or content behind junctions or symlinks.
- The summary shown before a deployment (TUI and `wincd deploy`) lists each path to delete, whether it currently exists, and the total size. It warns when rollback will be incomplete because of Update or Delete Mappings.
- The samples pack's Deployment Profile reference shows Update and Delete Mappings.

### Changed

- Mappings run top to bottom in declared order. Each mapping works from what the mappings before it left. Backup and capacity checks still happen before any write, and restore undoes the mappings in reverse order.
- After a restore, the decision whether traffic is switched back is made in one place. A recovery preview predicts exactly the warnings that the Recovery Run then records.
- `wincd rollback <version>`: the engine itself selects the retained release. The rollback summary shows its SHA-256 and the risk notices.
- Engine version 0.2.0.

### Fixed

- A deployment whose automatic restore succeeded is no longer left awaiting recovery when the intent of a phase or step could not be written to the transaction record (for example, while another process briefly held the file). Previously such a step stayed "running" and was treated as interrupted with an unknown outcome. The project blocked new deployments, kept traffic OFF, and needed a Recovery Run that repeated the whole restore. Such a step is now recorded as failed and not started. The transaction ends as restored, and traffic resumes when `F5.AutoUp` allows it. A step that was genuinely interrupted after it started (crash or Ctrl+C) is still treated as having an unknown outcome.
- A one-shot command that failed before it started (for example, because its staged file had changed) is no longer reported as if it had run and needed its effects checked.
- Writing the traffic gate creates a missing legacy health file folder.
- A frozen `KeepReleases` that is not a positive number keeps every retained release instead of removing them.
- Status, journal, and TUI labels:
  - A restored transaction shows one consistent label.
  - Entries left by a run that is no longer running show as unfinished.
  - A Manual Traffic Switch preview shows as previewed, not rejected.
  - Warnings say that traffic is switched back only once the project is no longer awaiting recovery.
- `wincd deploy` shows the local host name in its summary. `wincd recover` prints recovery-preview warnings once.

### Evaluation limitations

Unchanged from v0.1.2: native Windows/TUI acceptance is incomplete; no Windows Server matrix, production-workload, or real-F5 certification is claimed; no Authenticode signature or build provenance attestation is provided. Use the demo on a test machine only.

### Validation of these packages

- Automated suite (fake host and TEMP directories) on Windows PowerShell 5.1 with Pester 3.4.0: 1907 passed, 0 failed, 0 skipped.
- Both packages were verified file-for-file against the built source, including the files inside both demo ZIPs.
- In an elevated session and a new dedicated root, the INSTALL.md commands ran as written. They installed `wincd-engine-0.1.2.zip` from v0.1.2 and ran `status`, then updated to `wincd-engine-0.2.0.zip` and ran `status`.
- From the updated engine, `status` succeeded both through the call operator and through `powershell.exe -File`. Both demo ZIPs passed a read-only plan.

This is not a production-readiness declaration.

### License

Apache License 2.0. Bundled YamlDotNet remains under its MIT license.

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
