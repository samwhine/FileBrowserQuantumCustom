# FileBrowser Quantum Custom v1.5.8 — Samuel Extehines Heydemans

This tree is based on the official upstream stable tag:

- Repository: https://github.com/gtsteffaniak/filebrowser
- Upstream tag: `v1.5.8-stable`
- Upstream commit: `84024e258b4e71d32b8cce48c69ae5172a65fe08`
- Target: Windows amd64

## Custom changes

These upstream files contain code changes from the custom build:

```text
backend/common/settings/config.go
backend/common/settings/structs.go
backend/database/share/extended.go
backend/http/share.go
frontend/src/components/prompts/Help.vue
frontend/src/components/sidebar/Sidebar.vue
frontend/src/utils/upload.js
backend/http/static.go
```

The included `config-production-example.yaml` documents the custom configuration for the global share banner and sidebar behavior.

## Custom behavior

- Source-relative global fallback share banner / Open Graph image.
- Existing per-share custom banner remains higher priority.
- Upload resilience improvements: a 60-second stalled-progress window, bounded automatic retry with exponential backoff, and watchdog reset after each confirmed chunk response.
- The upload-fix remains user-configurable: `maxConcurrentUpload` and `uploadChunkSizeMb` continue to come from the user's File Loading settings; the custom code does not force a fixed concurrency or chunk size.
- Help remains available, but the Official Docs link is removed.
- Version text remains visible but is not a GitHub hyperlink.
- Sidebar displays the version string embedded in the EXE and can display configurable author attribution; the author link is optional and opens with safe external-link attributes.
- No changes to original file contents, download behavior, disk/storage logic, delete logic, user scope, or database schema were intentionally made.

## Rebuild note

The latest Windows executable delivered with this project is:

```text
filebrowser-quantum-v1.5.8-sidebar-author-upload-fix-windows-amd64.exe
```

SHA-256:

```text
3146d9fcec6f9c7ae3d12c83ca335d6db5247182d1b4976d30d9fa77f8b8d062
```

Before production deployment, rebuild from this tree with the pinned upstream commit and run the full backend/frontend tests in a Windows-capable build environment. Do not claim bit-for-bit reproducibility unless the same toolchain, frontend dependencies, build flags, and embedded assets are used.

## Custom author / maintainer

**Samuel Extehines Heydemans**

This attribution applies to the custom modifications in this repository. The upstream FileBrowser Quantum project and its original authors retain their original copyright and attribution.

The latest Windows binary embeds the display label `v1.5.8-sidebar-author-upload-fix`. The author attribution is displayed separately from the version and is configured through the frontend author fields; changing the display label during a future build does not change the upstream base version or custom behavior.

## Upstream security update included

This candidate includes the `v1.5.8-stable` high-severity fix for TOTP/MFA re-enrollment. Anonymous callers can no longer replace an existing second factor using only the account password; replacing or resetting an existing factor requires an authenticated self or administrator session. First-time enrollment without MFA remains supported.

The release also includes the upstream dependency updates and Docker FFmpeg update from the release notes.

## Validation summary

- Go version: `go1.27.0`
- Backend: `go test ./...` passed.
- Frontend: 10 test files and 58 tests passed.
- Frontend production build: passed.
- Windows cross-build: passed (`PE32+ x86-64`) for the sidebar-author-upload-fix build.
- Upload-fix frontend behavior was validated with the existing 58-test suite and a production asset build. Real-world browser/tunnel stress testing is still recommended with a small test batch before production rollout.
- Note: `npm ci` reported four high-severity advisories in the upstream dependency tree; review with `npm audit` before changing locked dependencies. They were not changed as part of this custom patch.
