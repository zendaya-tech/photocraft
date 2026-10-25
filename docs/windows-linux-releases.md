# Photozen Windows and Linux releases

The **Release** workflow builds Windows x64 and Linux x86_64 from the selected
committed Git ref. Local edits must be committed before they can be included.
Windows code signing is optional; packages are unsigned when no credentials exist.

## Run the workflow

Open **Actions > Release > Run workflow**, select the source branch, and run it.
The optional `version` input must match `[workspace.package] version` in
`Cargo.toml`; it checks the selected source and does not override its version.

Enable **publish** to make the result installable through Zendaya Launcher.
With this option disabled, the workflow leaves a draft for review. Pushes to the
`release` branch or to a matching `vX.Y.Z` tag build and publish automatically.
The tag must match the Cargo version. A published version cannot be overwritten;
bump the source version for the next release.

The workflow builds both the desktop application and its CLI with HEIF support,
using the existing packaging scripts. Release fonts are checked out at the same
immutable `craft-fonts` revision as the previous release configuration; their
licenses are included by the packaging scripts.

| Platform | Downloads |
| --- | --- |
| Windows x64 | `photocraft-<version>-windows-x64.msi`, `photocraft-<version>-windows-x64-portable.zip` |
| Linux x86_64 | `photocraft-<version>-linux-x86_64.{AppImage,AppImage.zsync,deb,rpm,tar.gz}` |
| Checksums | `SHA256SUMS.txt` |

The portable ZIP includes `photocraft.exe` and `photocraft-cli.exe`. Packaging
checks their PE architecture and GUI/console subsystem and smoke-tests the CLI.
The Linux packaging script also smoke-tests the CLI. Every expected package must
exist and be nonempty before the final job uploads files and publishes the release.

## Launcher availability

Zendaya Launcher's Photozen catalog entry points to `zendaya-tech/photocraft`.
It accepts both `photozen-*` and the retained `photocraft-*` asset names. The
workflow keeps the packaging scripts' `photocraft-*` names for compatibility.

The launcher ignores drafts. Its default channel also ignores prereleases such
as `-rc.1`. Publish a stable version with the packages and `SHA256SUMS.txt`, then
refresh the launcher's release list.

## Optional Windows signing

Configure repository Actions secrets for either the PFX method
(`WINDOWS_CERTIFICATE` and `WINDOWS_CERTIFICATE_PASSWORD`) or the Azure method
(`AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`,
`AZURE_SIGNING_ENDPOINT`, `AZURE_SIGNING_ACCOUNT`, `AZURE_CERT_PROFILE`).
Signing is implemented in `packaging/windows/sign.ps1`. SignPath requires a
separate integration; signing in to its website does not enable signing in CI.
