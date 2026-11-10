# WinCD

WinCD is a Windows deployment and rollback tool with a Traditional Chinese console TUI and a PowerShell command-line interface. Both use the same Deployment Engine. Human approvals and coordination across machines remain outside the tool.

This is the **public distribution repository**: user documentation, release notes, and downloadable runtime packages. Development history is maintained separately. Runtime packages contain readable PowerShell and supporting source files; they are not opaque executables.

## Downloads and release status

Download runtime ZIPs and their `SHA256SUMS` from [GitHub Releases](https://github.com/cyljacky02/wincd/releases). GitHub's automatically generated **Source code** ZIP/tarball contains this documentation repository, **not** the installable engine.

The current release is **v0.1.1**, a **pre-release**, with two packages:

- `wincd-engine-0.1.1.zip`: the engine.
- `wincd-samples-0.1.1.zip` (optional): a demo you can deploy (v1, then v2) and roll back on a test machine without Tomcat, IIS, or F5, plus the fully commented Deployment Profile reference. See section 4 of [INSTALL](INSTALL.md).

A pre-release is intended for evaluation, not a claim of production readiness; see the [release notes](CHANGELOG.md) for its validation scope and limitations.

## Capabilities

- Deploy selected file and directory mappings for one project on one Windows host.
- Manage Windows services, IIS sites/application pools, and declared process services in an explicit stop/start order.
- Run declared one-shot commands, checks, and fixed waits at their specified positions.
- Back up changed files, restore after failures, recover interrupted transactions, and roll back using retained deployment content.
- Express local traffic intent through an IIS-hosted F5 monitor endpoint or a legacy health file. This is not proof of application health, traffic draining, or the load balancer's actual member state.

## Requirements

- Windows PowerShell **5.1** (`powershell.exe`).
- .NET Framework **4.6.2 or later**.
- An elevated local-administrator account for mutating operations.
- IIS ASP.NET 4.x only when using the optional IIS monitor endpoint.

The YAML parser is included, so an installed host does not need internet access or additional PowerShell modules. Current native/TUI acceptance and the Windows Server validation matrix are incomplete; this repository does not claim production-workload or real-F5 certification.

See [installation and updates](INSTALL.md) (Traditional Chinese), [release notes](CHANGELOG.md), and [release integrity policy](RELEASE-POLICY.md).

## Licensing

WinCD is licensed under the [Apache License, Version 2.0](LICENSE). Each release also attaches a copy as `LICENSE`.

Bundled third-party components retain their own licenses: YamlDotNet is MIT-licensed, and its notice is included in the runtime package at `engine/WinCD/Lib/YamlDotNet/LICENSE.txt`.

Do not upload real deployment configuration, credentials, logs, or production data to this public repository.
