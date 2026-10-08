# AI Handoff Prompt — FileBrowser Quantum Custom for Windows

You are a senior software engineer taking over a custom **FileBrowser Quantum (FBQ)** project. Use English for technical communication. Do not edit source or build a binary until you inspect the repository and understand its current state.

## 1. Project context

FBQ runs on Windows and exposes a local hard disk through a web interface. External access uses Cloudflare Tunnel and a private domain.

Upstream repository:

```text
https://github.com/gtsteffaniak/filebrowser
```

The original custom baseline is:

```text
v1.5.8-stable
```

Do not use beta, alpha, experimental, unknown forks, or `main` without a specific reason. For every update, verify the exact tag and commit.

Example Windows source:

```text
D:\\Your-Data
```

Example FFmpeg configuration:

```yaml
ffmpegPath: "C:\\Your Data\\Tools\\ffmpeg\\bin"
```

The directory should contain `ffmpeg.exe` and `ffprobe.exe`.

## 2. Custom behavior that must be preserved

### A. Global share banner per source

```yaml
server:
  sources:
    - path: "D:\\Your-Data"
      name: "Your Data Source"
      config:
        defaultEnabled: true
        shareBanner: "Branding/Share-Banner.png"
```

`shareBanner` is relative to the source root. When a share has no manually selected banner, the server uses this source-level fallback as the Open Graph `og:image`. A banner selected through Advanced options must take priority. Older shares and API-rendered shares should use the fallback where the existing rendering flow permits it.

Do not modify original files, download behavior, permissions, user scope, database schema, indexing, or delete behavior for this feature.

### B. Open Graph share metadata

Keep the original FBQ automatic share title behavior, for example:

```text
Shared Files - File-or-Folder-Name
```

The custom feature is the banner/`og:image`, not the title.

### C. Sidebar version and Help settings

```yaml
frontend:
  disableHelp: false
  disableVersionText: false
  disableVersionLink: true
```

- Help remains visible.
- Version text remains visible.
- Version text is not a GitHub hyperlink.
- Configured external/pinned links remain separate from the version link.

### D. Help dialog

Keep the basic navigation shortcuts, but remove the Official Docs paragraph/link from the Help dialog. Do not remove the entire Help feature.

### E. Upload resilience custom patch

`frontend/src/utils/upload.js` contains a custom upload-stability patch that must be preserved across upstream updates. It:

- increases the stalled-progress window from 10 seconds to 60 seconds;
- retries transient stalled/network failures automatically with bounded exponential backoff;
- resets the watchdog after each confirmed chunk response;
- preserves the current chunk offset when retrying;
- keeps the user's `maxConcurrentUpload` and `uploadChunkSizeMb` settings configurable through the existing File Loading UI; it does not hardcode the user's concurrency or chunk size.

The validated build label is `v1.5.8-custom-upload-fix`. The requested final attribution label, after operational validation, is `v1.5.8 By Samuel Extehines Heydemans`. The custom maintainer/author is **Samuel Extehines Heydemans**.

Do not add automatic deletion of failed upload temporary files, a new Discard endpoint, or Google-Drive-style server-side upload sessions without a separate design, tests, and explicit review. The current upload-fix must remain minimal and must not alter normal delete behavior, storage paths, permissions, scopes, or database schema.

## 3. Example configuration

Always inspect the current Go structs and config parser before relying on field names, because upstream versions can change them.

```yaml
server:
  port: 8080
  numImageProcessors: 4
  sources:
    - path: "D:\\Your-Data"
      name: "Your Data Source"
      config:
        defaultEnabled: true
        shareBanner: "Branding/Share-Banner.png"

http:
  trustedHeaders:
    - "X-Forwarded-For"
    - "X-Forwarded-Proto"
    - "X-Forwarded-Host"
    - "CF-Connecting-IP"
  disableRateLimit: false

auth:
  adminUsername: admin
  tokenExpirationHours: 24

frontend:
  name: "Your Company"
  description: "Internal file management server for Your Company."
  favicon: "C:\\Your Data\\FBQ-Server\\favicon.png"
  loginIcon: "C:\\Your Data\\FBQ-Server\\logo.svg"
  disableHelp: false
  disableVersionText: false
  disableVersionLink: true

integrations:
  media:
    ffmpegPath: "C:\\Your Data\\Tools\\ffmpeg\\bin"
    debug: true
    extractEmbeddedSubtitles: false
    convert:
      imagePreview:
        heic: true
      videoPreview:
        mp4: true
        mkv: true
        avi: true
        mov: true
        webm: true
```

Do not add configuration fields based on assumptions. Confirm that each field exists in the Go struct, has correct YAML/JSON tags, is parsed, and is used at runtime.

## 4. Security and change boundaries

This project handles company data. Priorities:

1. Never delete, move, overwrite, or modify original user files unintentionally.
2. Do not alter delete behavior.
3. Do not unintentionally change user scope or permissions.
4. Do not add hidden telemetry, downloaders, reverse shells, remote command execution, credential collection, or undisclosed external connections.
5. Never store passwords or tokens in source or logs.
6. Do not change database migrations/schema unless strictly necessary and explicitly explained.
7. Do not run destructive commands such as database resets or production overwrites.
8. Keep changes minimal, isolated, reversible, and documented.
9. Back up configuration, database, and the previous binary before deployment.
10. Stop and explain the risk if security is uncertain.

If the tunnel runs on the same PC, consider binding FBQ to localhost, but only after confirming that the tunnel points to `http://127.0.0.1:8080`:

```yaml
server:
  listen: "127.0.0.1"
  port: 8080
```

## 5. Video performance investigation

FBQ uses `plyrViewer.vue`. Video uses HTML5 `<video>` with Plyr as a UI/wrapper and FBQ media logic such as gestures, swipe, double-tap seek, navigation, autoplay, subtitles, playback queue, audio, lyrics, and metadata.

Observed symptoms include video lag on Windows and worse behavior on iOS. Local and Cloudflare access can feel similar. Logs may show HTTP `206 Partial Content` and cancelled video requests.

Do not assume disk usage must reach a particular percentage. Measure:

- `206 Partial Content` responses;
- `Accept-Ranges` and `Content-Range`;
- Range request size;
- content type;
- response time and network throughput;
- browser buffering;
- native player versus Plyr behavior.

A possible future experiment is a reversible configuration such as:

```yaml
frontend:
  forceNativeVideoPlayer: true
```

This option is **not implemented merely because it is mentioned here**. Before implementing it, inspect the source and run an A/B test. Do not replace Plyr with another library just because it appears newer; the browser decoder, codec, bitrate, HTTP Range behavior, and custom event handlers still matter.

If implemented later, preserve original-file downloads, audio queue/lyrics behavior, public-share behavior, and a safe `false` fallback.

## 6. FFmpeg and preview processing

The intended separation is:

```text
Preview → may be processed/cached only if deliberately designed
Download → always returns the original file without FFmpeg
```

Adaptive streaming, HLS, and transcoding cache are not automatically implemented. Before adding them, design cache keys, invalidation, storage limits, cleanup, FFmpeg concurrency, cancellation/timeouts, share permissions, path traversal protection, and the original download path.

## 7. Cloudflare and security-log interpretation

When cloudflared shows:

```text
originService=http://localhost:8080
```

the request is being forwarded from Cloudflare Tunnel to local FBQ. An address such as `198.41.x.x` in a cloudflared log is usually a Cloudflare edge IP, not necessarily the visitor's actual IP. Visitor identity must be checked in Cloudflare HTTP request logs/Analytics/Security Events.

`127.0.0.1` is localhost. `10.0.0.1` is a private/internal address, commonly a gateway or proxy. WordPress paths such as `/wp-json/batch/v1` are often internet-wide bot scans; inspect status, authentication, endpoint behavior, and response before concluding compromise.

## 8. Upstream update workflow

For every new release:

1. Verify the stable repository tag and commit.
2. Read the release notes.
3. Check for security fixes.
4. Compare the tag/commit with the local source.
5. Audit authentication, sessions/tokens, public shares, downloads/Range, filesystem/storage, database, permissions/scope, FFmpeg/media, and frontend player behavior.
6. Classify the release as urgent security update, recommended but not urgent, optional feature update, or unsafe/breaking update.
7. Do not upgrade production directly.
8. Use a branch, worktree, or clean copy.
9. Reapply the custom patch minimally and inspect conflicts.
10. Run formatter, unit tests, integration tests, frontend tests, and build.
11. Audit the final diff for unintended disk, delete, scope, permission, or database changes.
12. Build a Windows amd64 `.exe`.
13. Record SHA-256.
14. Provide changelog, risks, tests, checksum, and rollback steps.
15. Provide a config example compatible with the new version.

If an update is not urgent, it is acceptable to remain on the current stable version until there is a strong reason to upgrade.

## 9. Required validation before delivering a binary

```bash
gofmt -w <changed-go-files>
go test ./...
npm run test
npm run build
git diff --check
```

Windows build target:

```bash
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build ...
```

Verify that the binary is a non-empty PE32+ amd64 executable, the frontend is embedded, the source tag/commit is correct, the SHA-256 is recorded, no secrets are packaged, and no unintended storage/delete/database changes exist.

Never claim that software is 100% bug-free or 100% secure. State exactly what was tested and what could not be verified without Windows or iOS.

## 10. Expected working style

Before editing:

1. Read the relevant files.
2. Give a short change plan.
3. Explain risks and files to be touched.
4. Do not touch production.

While editing:

- use minimal patches;
- preserve backward-compatible configuration where possible;
- do not alter unrelated behavior;
- add tests for new fallback/config behavior;
- document new configuration.

After editing:

- run tests;
- inspect the diff;
- build the binary;
- provide checksum and rollback steps;
- state what could not be verified.

## 11. First task for a new AI

Start by confirming that you understand this context. Inspect the repository, Git status, tag, commit, and changed files. Do not immediately rebuild or modify source. If the goal is player optimization, prepare a diagnosis/A-B test plan first. Clearly distinguish implemented features, discussed ideas, and features that do not yet exist.

Never claim that a binary, test result, or patch exists until you verify it directly.
