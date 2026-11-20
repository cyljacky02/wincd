# Release notes

## v0.3.2 — pre-release

The disk space check before a deployment or rollback now counts only what that run needs. `KeepReleases` and `KeepBackups` no longer reserve disk space. Data formats are unchanged from v0.3.0, apart from one field dropped from the capacity report.

### Upgrading

- From v0.3.0 or v0.3.1: transaction records, frozen Plans and the Backup layout are unchanged (`schemaRevision: 7`). Replace the engine folder as INSTALL describes; projects do not need to be deployed again. As for every update, first close running TUIs and jobs and finish pending Recovery Runs.
- From v0.2.x: follow "Upgrading from v0.2.x" under v0.3.0 below. It applies unchanged.

### Changed

- The disk space check before the host is changed, and its repeat after StopOrder Exec steps and before Backup, counts only this run's peak need:
  - on the data volume: a new retained release copy (none when that release is already retained), extraction, this run's Backup, transaction records and a safety margin;
  - on each destination volume: what this run writes.

  Previously the check also required room for `KeepReleases − 1` more releases and `KeepBackups − 1` more backups. A large count such as 9999 therefore refused deployments with `DiskSpace` even when the disk could hold the deployment. Too little space for this run is still refused (`rejected`, field `DiskSpace`) before the host is changed. The check never deletes retained releases or backups to make room.
- A very large `KeepReleases` or `KeepBackups` now means, in effect, no automatic cleanup. That has two costs:
  - listing and selecting releases hash every retained copy again, so the release list and rollback get slower as copies accumulate;
  - every Backup holds the files a deployment overwrote or deleted as they were, plaintext configuration included. They accumulate under the data folder until you delete them.
- The `capacity` and `backupCapacity` reports in results and transaction records no longer have a `retentionBytes` field.
- Engine version 0.3.2. Deployment Profile rules, frozen Plans and the Backup layout are identical to v0.3.0; transaction records differ only by the dropped `retentionBytes` field, which no engine reads back.

### Evaluation limitations

Unchanged from v0.3.0: native Windows/TUI acceptance is incomplete; no Windows Server matrix, production-workload, or real-F5 certification is claimed; no Authenticode signature or build provenance attestation is provided. Use the demo on a test machine only.

### Validation of these packages

- Automated suite (fake host and TEMP directories) on Windows PowerShell 5.1 with Pester 3.4.0, in an elevated session: 1998 passed, 0 failed, 1 skipped. The skipped test compiles the monitor handler against IIS's Microsoft.Web.Administration and needs IIS on the test host. Nine tests that need a volume that generates 8.3 short names were inconclusive on the validation machine.
- Both packages were verified file-for-file against the built source, including the files inside both demo ZIPs.
- In an elevated session and a new dedicated root, the INSTALL.md commands ran as written. They installed `wincd-engine-0.3.1.zip` from v0.3.1 and ran `status`, then updated to `wincd-engine-0.3.2.zip` and ran `status`. Each ZIP matched its release's `SHA256SUMS`.
- From the updated engine, `status` succeeded both through the call operator and through `powershell.exe -File`. Both demo ZIPs passed a read-only plan.

This is not a production-readiness declaration.

### License

Apache License 2.0. Bundled YamlDotNet remains under its MIT license.

## v0.3.1 — pre-release

Fixes how rollback, deployment and retention cleanup decide which release is installed, makes restore check every recorded change before undoing any, and has `wincd` explain why an operation cannot run before asking for confirmation. Data formats are unchanged from v0.3.0.

### Upgrading

- From v0.3.0: transaction records, frozen Plans and the Backup layout are unchanged (`schemaRevision: 7`). Replace the engine folder as INSTALL describes; projects do not need to be deployed again. As for every update, first close running TUIs and jobs and finish pending Recovery Runs.
- From v0.2.x: follow "Upgrading from v0.2.x" under v0.3.0 below. It applies unchanged.

### Fixed

- Rollback, deployment and retention cleanup now use the release that is actually installed, as `status` reports it, instead of the release whose deployment finished last. Previously, after the host clock was set back, these could go wrong:
  - the default rollback could reinstall the release that was already running;
  - after a restore, an Overlay deployment could remove files based on the wrong release;
  - retention cleanup could delete the installed release's only retained copy.

  When the records do not show which release is installed, deploy and rollback are refused (field `Journal`) instead of guessing which files to remove, and retention cleanup deletes nothing. The read-only release list still shows `current` as before in that case.
- Restore and Recovery Runs check every mapping's recorded change before undoing any destination. Previously, a mismatch found in a later mapping stopped the restore after the earlier mappings had already been put back.
- An Exec Step is reported as running only after its start has been recorded. An Exec whose start could not be recorded is reported only as not started.

### Changed

- `wincd` asks for `Y` only when the operation can run as summarized. It now refuses before asking (`rejected`, exit code 1, host unchanged) when:
  - `f5 down|up` without `-Yes` or `-Force` gets a rejected preview, or a preview with Forceable warnings. The result keeps those `warnings` and names the explicit `-Force` command; `Y` never means Force. These refusals are not recorded as manual traffic switches.
  - `deploy` or `rollback` runs with `-Json` but without `-Yes`, and the release contains an Operator Pause (field `OperatorPause`). Use the TUI or `wincd` without `-Json`.
  - the default `rollback` finds no version to roll back to because the previous version's retained copy is damaged. The damaged copies are listed in `warnings`.
- Engine version 0.3.1. Deployment Profile rules and data formats are identical to v0.3.0.

### Evaluation limitations

Unchanged from v0.3.0: native Windows/TUI acceptance is incomplete; no Windows Server matrix, production-workload, or real-F5 certification is claimed; no Authenticode signature or build provenance attestation is provided. Use the demo on a test machine only.

### Validation of these packages

- Automated suite (fake host and TEMP directories) on Windows PowerShell 5.1 with Pester 3.4.0, in an elevated session: 1995 passed, 0 failed, 1 skipped. The skipped test compiles the monitor handler against IIS's Microsoft.Web.Administration and needs IIS on the test host. Nine tests that need a volume that generates 8.3 short names were inconclusive on the validation machine.
- Both packages were verified file-for-file against the built source, including the files inside both demo ZIPs.
- In an elevated session and a new dedicated root, the INSTALL.md commands ran as written. They installed `wincd-engine-0.3.0.zip` from v0.3.0 and ran `status`, then updated to `wincd-engine-0.3.1.zip` and ran `status`. Each ZIP matched its release's `SHA256SUMS`.
- From the updated engine, `status` succeeded both through the call operator and through `powershell.exe -File`. Both demo ZIPs passed a read-only plan.

This is not a production-readiness declaration.

### License

Apache License 2.0. Bundled YamlDotNet remains under its MIT license.

## v0.3.0 — pre-release

Adds the Operator Pause, keeps each Backup at the originals' own paths with a delete list and a guide for manual recovery, and fixes edge cases at go-live, in restore and in rollback selection. **Transaction and Plan data formats change again: read "Upgrading" before updating an existing installation.**

### Upgrading from v0.2.x

- Frozen Plans and transaction records now use `schemaRevision: 7`. v0.3.0 refuses to recover, preview a recovery, or switch traffic from records written by v0.2.x (revision 5, and revision 6, which only existed in development builds). v0.2.x engines likewise refuse revision 7 records and releases whose Orders contain `Pause`. Nothing is converted automatically.
- v0.3.0 cannot read a Backup in the v0.2.x layout. Before updating, finish every pending Recovery Run and every unfinished transaction with the engine that is installed now, as INSTALL describes.
- After updating, deploy each project once with v0.3.0.
- Some deployments that v0.2.x accepted are now refused before the host is changed:
  - a Backup path longer than 259 characters (247 for a folder). Install WinCD under a shorter root, such as `D:\WinCD`;
  - a directory mapping that would overwrite or delete a read-only file. File mappings already refused this.

### Added

- Operator Pause: write `Pause` as an entry anywhere in `StopOrder` or `StartOrder` to stop the deployment there until the Deployer chooses one of three options:
  - Continue: run the next entry.
  - Fail: the deployment fails at the Pause and restores automatically.
  - Interrupt: ends exactly like Ctrl+C. Traffic is set OFF and the project awaits a Recovery Run.

  How it behaves:
  - Only the exact word `Pause` counts; other casings are rejected with a hint.
  - It is not an approval: the engine checks nobody's permission and records the decision, the time, and how long the deployment waited.
  - The deploy and rollback summaries list every Pause. At a Pause, the screen shows the Order, the position, the steps before and after, and the time waited.
  - A Pause only runs interactively (TUI, or `wincd` without `-Json`). A non-interactive or `-Json` deploy or rollback of a release with a Pause is refused before any host change. `-Yes` does not skip a Pause.
  - Restore and Recovery Runs never stop at a Pause. A rollback stops at the Pauses of the release it deploys.
- Backup layout: each original is kept at its own path below the transaction's Backup folder. For example, `C:\app\lib\a.jar` is kept at `<DataRoot>\backups\<project>\<transaction>\C\app\lib\a.jar`. Beside it are:
  - `delete-list.txt`: the top-most paths the deployment created. It is a valid Delete List.
  - `README.txt`: how to put the Backup back by hand, including read-only paths and the junctions or symlinks to recreate.

  The engine never reads either file back: restore and Recovery Runs still follow the transaction record. Putting a Backup back by hand does not clear "awaiting recovery".
- `Get-WinCDRelease` results include `warnings`.

### Changed

- If the release's retained copy cannot be read just after the deployment goes live (missing, damaged, or held open by antivirus or backup software), the deployment completes live with a `Release Store:` warning. Previously, a deployment that had finished this way ended awaiting recovery with traffic OFF, and the Recovery Run then undid the healthy deployment.
- A damaged retained copy is left out of the release list and named in `warnings`, and every intact version can still be selected for rollback. Previously one damaged copy blocked rollback to every version.
  - Selecting the damaged version is refused.
  - When it shares a version label with another release, the label alone is still refused as repeated; choose by SHA-256.
  - Retention cleanup deletes nothing while any retained copy is damaged.
- Before Replace changes any destination, it refuses (and the deployment restores) when a file appeared after the Backup was taken where a Mirror would delete it or a write would overwrite it, because the Backup does not hold that file. This check does not cover:
  - a folder that appears empty;
  - files that appear after Replace starts (a failed Mirror deployment still removes them when it restores);
  - Delete Mappings, which delete whatever is at their listed paths.
- Engine version 0.3.0.

### Fixed

- Restore no longer resolves 8.3 short names in recorded delete paths a second time. Previously, if a short name came to point at a different folder between Backup and restore, a restore could leave behind a folder the deployment created, or put deleted content into the wrong folder while reporting "restored". It now restores the correct paths, or leaves the project awaiting recovery.
- A transaction record whose recorded paths would lead outside their mapping (for example `..` or a network path) leaves the project awaiting recovery instead of restoring anywhere.
- An Interrupt at an Operator Pause stays an interruption even when its outcome cannot be written to the transaction record.
- `powershell.exe -NonInteractive` counts as non-interactive before a release with a Pause is admitted.

### Evaluation limitations

Unchanged from v0.2.0: native Windows/TUI acceptance is incomplete; no Windows Server matrix, production-workload, or real-F5 certification is claimed; no Authenticode signature or build provenance attestation is provided. Use the demo on a test machine only.

### Validation of these packages

- Automated suite (fake host and TEMP directories) on Windows PowerShell 5.1 with Pester 3.4.0: 1965 passed, 0 failed, 0 skipped. Nine tests that need a volume that generates 8.3 short names were inconclusive on the validation machine.
- Both packages were verified file-for-file against the built source, including the files inside both demo ZIPs.
- In an elevated session and a new dedicated root, the INSTALL.md commands ran as written. They installed `wincd-engine-0.2.0.zip` from v0.2.0 and ran `status`, then updated to `wincd-engine-0.3.0.zip` and ran `status`.
- From the updated engine, `status` succeeded both through the call operator and through `powershell.exe -File`. Both demo ZIPs passed a read-only plan.

This is not a production-readiness declaration.

### License

Apache License 2.0. Bundled YamlDotNet remains under its MIT license.

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
