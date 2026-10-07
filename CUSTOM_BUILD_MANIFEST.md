# FileBrowser Quantum Custom v1.5.8

This tree is based on the official upstream stable tag:

- Repository: https://github.com/gtsteffaniak/filebrowser
- Upstream tag: `v1.5.8-stable`
- Upstream commit: `84024e258b4e71d32b8cce48c69ae5172a65fe08`
- Target: Windows amd64

## Custom changes

Only these six upstream files contain code changes from the custom build:

```text
backend/common/settings/config.go
backend/common/settings/structs.go
backend/database/share/extended.go
backend/http/share.go
frontend/src/components/prompts/Help.vue
frontend/src/components/sidebar/Sidebar.vue
```

The included `config-production-example.yaml` documents the custom configuration for the global share banner and sidebar behavior.

## Custom behavior

- Source-relative global fallback share banner / Open Graph image.
- Existing per-share custom banner remains higher priority.
- Help remains available, but the Official Docs link is removed.
- Version text remains visible but is not a GitHub hyperlink.
- No changes to original file contents, download behavior, disk/storage logic, delete logic, user scope, or database schema were intentionally made.

## Rebuild note

The Windows executable previously delivered with this project was:

```text
filebrowser-quantum-v1.5.8-custom-windows-amd64.exe
```

SHA-256:

```text
1dca554e47b179fd07a688c5e64aef7fc36e70d6c20601f3e3b16de67bde98ef
```

Before production deployment, rebuild from this tree with the pinned upstream commit and run the full backend/frontend tests in a Windows-capable build environment. Do not claim bit-for-bit reproducibility unless the same toolchain, frontend dependencies, build flags, and embedded assets are used.

## Custom author / maintainer

**Samuel Extehines Heydemans**

This attribution applies to the custom modifications in this repository. The upstream FileBrowser Quantum project and its original authors retain their original copyright and attribution.

## Upstream security update included

This candidate includes the `v1.5.8-stable` high-severity fix for TOTP/MFA re-enrollment. Anonymous callers can no longer replace an existing second factor using only the account password; replacing or resetting an existing factor requires an authenticated self or administrator session. First-time enrollment without MFA remains supported.

The release also includes the upstream dependency updates and Docker FFmpeg update from the release notes.

## Validation summary

- Go version: `go1.27.0`
- Backend: `go test ./...` passed.
- Frontend: 10 test files and 58 tests passed.
- Frontend production build: passed.
- Windows cross-build: passed (`PE32+ x86-64`).
- Note: `npm ci` reported four high-severity advisories in the upstream dependency tree; review with `npm audit` before changing locked dependencies. They were not changed as part of this custom patch.
