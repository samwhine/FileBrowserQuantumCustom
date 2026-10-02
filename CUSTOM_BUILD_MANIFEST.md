# FileBrowser Quantum Custom v1.5.6

This tree is based on the official upstream stable tag:

- Repository: https://github.com/gtsteffaniak/filebrowser
- Upstream tag: `v1.5.6-stable`
- Upstream commit: `5d9b4df2a21d1ba4a6af481181a7402cb5cbb5ca`
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
filebrowser-quantum-v1.5.6-global-banner-windows-amd64.exe
```

SHA-256:

```text
91b315180ce80ba16717cba518e0930e9813adbc532fb87bcf249b3fb5f94f68
```

Before production deployment, rebuild from this tree with the pinned upstream commit and run the full backend/frontend tests in a Windows-capable build environment. Do not claim bit-for-bit reproducibility unless the same toolchain, frontend dependencies, build flags, and embedded assets are used.

## Custom author / maintainer

**Samuel Extehines Heydemans**

This attribution applies to the custom modifications in this repository. The upstream FileBrowser Quantum project and its original authors retain their original copyright and attribution.
