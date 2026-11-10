# Installation and updates

These instructions describe the v0.1.1 runtime packages, published as a pre-release under the [Apache License 2.0](LICENSE). Read the [release notes](CHANGELOG.md) for their acceptance limitations before using them.

## 1. Download and verify

From the same [GitHub Release](https://github.com/cyljacky02/wincd/releases), download:

- `wincd-engine-0.1.1.zip`
- `SHA256SUMS`
- optionally `wincd-samples-0.1.1.zip` (see [section 4](#4-optional-try-the-samples-pack))

In Windows PowerShell 5.1, calculate each ZIP's SHA-256 and compare it with the entry for the identical filename in `SHA256SUMS`. Stop if they do not match.

```powershell
Get-FileHash .\wincd-engine-0.1.1.zip -Algorithm SHA256
Get-FileHash .\wincd-samples-0.1.1.zip -Algorithm SHA256
Get-Content .\SHA256SUMS
```

A checksum confirms matching bytes; an unsigned checksum downloaded beside a package is not independent proof of publisher identity. See [release integrity](RELEASE-POLICY.md).

## 2. Install into a new, dedicated directory

Use an **elevated Windows PowerShell 5.1** session. The following is for a fresh installation only; do not extract over an existing engine or an active deployment.

```powershell
Unblock-File .\wincd-engine-0.1.1.zip
New-Item C:\WinCD -ItemType Directory -ErrorAction Stop
icacls C:\WinCD /inheritance:r /grant:r '*S-1-5-32-544:(OI)(CI)F' '*S-1-5-18:(OI)(CI)F' '*S-1-5-32-545:(OI)(CI)RX'
if ($LASTEXITCODE -ne 0) { throw 'Failed to restrict the installation directory permissions.' }
Expand-Archive .\wincd-engine-0.1.1.zip -DestinationPath C:\WinCD
& C:\WinCD\engine\wincd.ps1 status
```

The ZIP contains one top-level `engine` directory. The layout is:

```text
C:\WinCD\engine\  Runtime files, entry point wincd.ps1
C:\WinCD\inbox\   Incoming deployment ZIPs, created when needed
C:\WinCD\data\    Private state, logs, backups, and retained deployment content
```

The installation root can be another dedicated local directory, but the runtime folder must remain named **engine**. Never grant untrusted users write access to the engine. Deployment content and backups can contain plaintext credentials; do not publish or broadly share `inbox` or `data`.

`status` is read-only: it does not create the data layout, repair permissions, or provision IIS. Mutating commands prepare their required directories and enforce their own preconditions.

## 3. Open the TUI

```powershell
& C:\WinCD\engine\wincd.ps1
```

The console interface is in Traditional Chinese. Supplying a verb instead invokes the command-line interface. Start `wincd.ps1` with the PowerShell call operator (`&`) as shown; `powershell.exe -File` cannot start it in v0.1.1 (see the [release notes](CHANGELOG.md)). Do not point evaluation runs at production services, directories, or traffic endpoints.

The optional `install-monitor` command changes IIS configuration and permissions and requires additional host preparation. Do not run it as a routine installation check. A public monitor endpoint reports local traffic intent, not application health or confirmed load-balancer state.

## 4. Optional: try the samples pack

`wincd-samples-0.1.1.zip` is a separate download built from the same source as the engine. It is not part of the engine; do not extract it into the `engine` directory.

| Path in the ZIP | Contents |
| --- | --- |
| `README.md` | Walkthrough in Traditional Chinese: deploy v1, deploy v2, roll back, and clean up, with the equivalent command-line commands. |
| `demo/wincd-demo-v1.zip`, `demo/wincd-demo-v2.zip` | Deployment content for project `wincd-demo`. Needs no Tomcat, IIS, or F5 and writes only to `C:\WinCD-demo\`. |
| `reference/order-api/` | The fully commented Deployment Profile reference. It targets `D:\Tomcat9`, `C:\nginx`, and an IIS monitor site, so it cannot be deployed on a clean test machine; copy from it when writing your own profile. |

Use the demo **on a test machine only**: it runs as an administrator, makes `C:\WinCD-demo\site` match each release it deploys (Mirror deletes files that release does not contain, except the preserve list), and leaves the `wincd-demo` project's records in the WinCD `data` directory, because WinCD has no command to delete them. The walkthrough's installation step corresponds to sections 1 and 2 above.

## Updates

Stop all WinCD TUI sessions and jobs and resolve unfinished recovery work before considering an update. Verify the new package and its documented compatibility, replace the **entire engine directory**, and preserve `inbox` and `data` separately. Do not overlay files or assume that an older engine can read newer profiles or saved transaction state.

v0.1.1 changes only the reported engine version. Deployment Profile rules and saved data formats are identical to v0.1.0, so updating from v0.1.0 is the engine-directory replacement described above. Neither version promises migration or compatibility with earlier development snapshots. Changes to the optional IIS handler may require a separately planned monitor installation/update; ordinary runtime operations do not silently repair or upgrade that installation.
