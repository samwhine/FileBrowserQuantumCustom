# FileBrowser Quantum Custom — Update Review Prompt

I maintain a custom **FileBrowser Quantum (FBQ)** installation on Windows. When I send you an FBQ release or update link, do not immediately upgrade or build anything. First assess the update, then build only if it is safe and necessary.

## Project identity

- Upstream repository: `https://github.com/gtsteffaniak/filebrowser`
- Previous baseline: `v1.5.6-stable`
- Target platform: Windows 64-bit (`GOOS=windows`, `GOARCH=amd64`)
- Primary storage: Windows local disk, for example `D:\\Your-Data`
- External access: Cloudflare Tunnel and a private domain
- FFmpeg is available on Windows and configured through `integrations.media.ffmpegPath`

## Custom behavior that must be preserved

### 1. Global share banner per source

```yaml
server:
  sources:
    - path: "D:\\Your-Data"
      name: "Your Data Source"
      config:
        defaultEnabled: true
        shareBanner: "Branding/Share-Banner.png"
```

`shareBanner` is relative to the source root. It is the fallback `og:image` for public shares. A banner selected manually through Advanced options must always take priority over the global fallback.

### 2. Original FBQ share title

- Keep the original automatic share title behavior, such as `Shared Files - File-or-Folder-Name`.
- The custom change concerns the banner/`og:image`, not the title.

### 3. Sidebar settings

```yaml
frontend:
  disableHelp: false
  disableVersionText: false
  disableVersionLink: true
```

Help remains available, version text remains visible, and the version text is not a GitHub hyperlink.

### 4. Help dialog

- Keep the built-in navigation shortcuts.
- Remove the Official Docs paragraph/link from the Help dialog.
- Do not remove the entire Help feature merely to remove that link.

## Security and change restrictions

- Do not alter disk or storage logic.
- Do not alter delete behavior.
- Do not change user scope, permissions, authentication, or public-share access without a specific reason and dedicated audit.
- Do not change the database schema unless upstream requires it and the migration is understood.
- Do not add hidden telemetry, downloaders, reverse shells, credential collectors, or undisclosed external connections.
- Do not modify users' original files.
- Downloads must continue to return the original files, not transcoded or cached derivatives.
- Do not run destructive commands or overwrite production databases/configuration.
- Keep changes minimal, reversible, and documented.

## Procedure for every upstream update

I will provide a release link, for example:

```text
<NEW FBQ RELEASE LINK>
```

### 1. Verify the source

- Confirm that the release is stable, not beta/alpha/nightly.
- Record the tag, commit, date, and parent commit.
- Do not use `main` when a stable tag is available.

### 2. Assess urgency

Read the release notes and source diff, then classify the update:

- **URGENT**: authentication bypass, unauthenticated access, path traversal, arbitrary file read/write, privilege escalation, public-share bypass, token/session vulnerability, database corruption, or an important security fix.
- **RECOMMENDED**: important bug fixes, Windows/FFmpeg/browser compatibility, storage/share stability, or relevant performance fixes.
- **OPTIONAL**: unused features, UI changes, documentation, or minor optimizations.
- **HOLD**: beta releases, breaking changes, risky database migrations, or changes incompatible with the custom patch.

Explain whether it is safe to remain on the current version when the update is not urgent.

### 3. Audit sensitive areas

Compare the old and new versions for:

- authentication, sessions, and tokens;
- user permissions and scope;
- public shares and share passwords;
- downloads and HTTP Range/`206 Partial Content`;
- filesystem/source resolution;
- delete, move, copy, and upload operations;
- database and migrations;
- FFmpeg/media previews;
- frontend player, Plyr, and native-player behavior;
- trusted HTTP headers and Cloudflare compatibility.

### 4. Reapply the custom patch

If the update is worth adopting:

- start from the stable tag;
- reapply the global `shareBanner` fallback;
- reapply the Help change that removes Official Docs;
- reapply the version text/link behavior;
- preserve compatible configuration names and formats;
- never assume the old patch applies cleanly—inspect conflicts manually.

### 5. Required tests

Run at minimum:

```bash
gofmt -w <changed-go-files>
go test ./...
npm run test
npm run build
git diff --check
```

Audit the final diff and confirm that there are no unrelated changes to storage, delete, permissions, scope, authentication, database, or downloads.

### 6. Windows build

If the result is safe:

```bash
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build ...
```

Verify:

- the binary is Windows PE32+ amd64;
- the frontend is embedded;
- the binary is non-empty;
- a SHA-256 checksum is recorded;
- the source tag/commit is correct;
- no secrets are packaged.

### 7. Final report format

```text
Release analyzed:
Tag/commit:
Status: URGENT / RECOMMENDED / OPTIONAL / HOLD
Reason:
Security fixes:
Breaking changes:
Impact on our custom patch:
Custom changes preserved:
Tests passed:
Tests not available:
Remaining risks:
Deployment recommendation:
Rollback steps:
SHA-256:
```

Be honest about anything that was not verified. Never claim that software is 100% bug-free or 100% secure.

## Important instruction

Before editing or building, read the release link I provide and explain your analysis. If the update is not urgent, do not create a new build without a clear reason. If it is urgent, reapply the custom patch and build only after testing and auditing the final diff.
