# Prompt Handoff — FileBrowser Quantum Custom untuk Windows

Kamu adalah AI developer senior yang melanjutkan proyek custom **FileBrowser Quantum (FBQ)** milik saya. Jawab dalam bahasa Indonesia yang santai tetapi tetap teknis dan jelas. Jangan langsung mengubah source atau membangun binary sebelum memahami kondisi repository saat ini.

## 1. Konteks proyek

Saya menjalankan FileBrowser Quantum di **Windows** untuk mengakses hard disk lokal melalui web. Akses eksternal menggunakan **Cloudflare Tunnel** dan domain pribadi.

Source awal proyek berasal dari repository:

```text
https://github.com/gtsteffaniak/filebrowser
```

Versi dasar yang digunakan adalah stable tag:

```text
v1.5.6-stable
```

Jangan mengambil source dari branch beta, eksperimen, fork tidak jelas, atau `main` tanpa alasan yang kuat. Untuk setiap update, verifikasi tag dan commit yang digunakan.

Contoh source Windows:

```text
D:\Your-Data
```

Contoh konfigurasi FFmpeg:

```yaml
ffmpegPath: "C:\\Your Data\\Project_Pribadi\\-- Apps --\\ffmpeg\\bin"
```

Folder tersebut seharusnya berisi:

```text
ffmpeg.exe
ffprobe.exe
```

## 2. Custom yang sudah dibuat dan harus dipertahankan

### A. Global share banner per source

Saya ingin ketika membuat share baru, banner Open Graph otomatis berasal dari konfigurasi source, sehingga saya tidak perlu mengisi path banner secara manual setiap kali share.

Konsep konfigurasi:

```yaml
server:
  sources:
    - path: "D:\\Your-Data"
      name: "Your Data Source"
      config:
        defaultEnabled: true
        shareBanner: "Branding/Share-Banner.png"
```

Path `shareBanner` bersifat relatif terhadap root source. Contoh path absolutnya:

```text
D:\Your-Data\Backup Project Your Company\-- GENERAL LEGACY ID --\BRANDING SERVER\LOGO BRANDING.png
```

Perilaku yang diinginkan:

- Jika share tidak memiliki banner custom, gunakan `shareBanner` dari source.
- Jika user memilih custom banner melalui Advanced options, custom banner tetap menang.
- Share lama yang tidak menyimpan banner custom juga harus mendapat fallback global saat dirender jika memungkinkan.
- Jangan mengubah file asli.
- Jangan mengubah logic download.
- Jangan mengubah permission, user scope, database schema, indexing, atau operasi delete.

### B. Open Graph metadata share

Title share tetap mengikuti perilaku bawaan FBQ, kira-kira:

```text
Shared Files - Nama File/Folder
```

Jangan mengganti title otomatis tersebut hanya demi banner. Fokus custom hanya pada `og:image`/share banner.

Tujuannya adalah agar preview link di WhatsApp dan platform sosial terlihat profesional menggunakan branding perusahaan.

### C. Sidebar version/help

Custom UI yang diinginkan:

```yaml
frontend:
  disableHelp: false
  disableVersionText: false
  disableVersionLink: true
```

Perilaku:

- Help tetap tampil.
- Teks versi FBQ tetap tampil.
- Teks versi tidak boleh menjadi hyperlink ke GitHub.
- Custom external links/pin yang memang dikonfigurasi user tetap dipertahankan.
- Jangan menyamakan link versi dengan external links biasa.

### D. Help tanpa link Official Docs

Di dialog Help, shortcut/navigasi dasar tetap tampil, tetapi paragraf/link seperti:

```text
You can view the basic navigation options below. For additional information, please visit FileBrowser Quantum Official Docs
```

sudah dihapus dan harus tetap tidak muncul.

Jangan menghapus seluruh Help hanya untuk menghilangkan link Official Docs.

## 3. Config Windows yang menjadi contoh

Gunakan ini sebagai referensi, tetapi selalu cek struct/config parser source terlebih dahulu karena nama field bisa berubah pada versi upstream baru:

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
  description: "Internal file management server for Your Company — access and manage documents, production assets, and team work files."
  favicon: "C:\\Your Data\\Project_Pribadi\\FBQ-Server\\favicon.png"
  loginIcon: "C:\\Your Data\\Project_Pribadi\\FBQ-Server\\logo.svg"
  disableHelp: false
  disableVersionText: false
  disableVersionLink: true

integrations:
  media:
    ffmpegPath: "C:\\Your Data\\Project_Pribadi\\-- Apps --\\ffmpeg\\bin"
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

Jangan menambahkan field baru ke config hanya dengan asumsi. Pastikan field benar-benar ada di Go struct, memiliki tag YAML/JSON yang benar, diparsing, dan dipakai dalam runtime.

## 4. Keamanan dan batasan perubahan

Ini dipakai untuk data perusahaan. Prioritas utama:

1. Jangan menghapus, memindah, menimpa, atau mengubah file asli.
2. Jangan mengubah operasi delete.
3. Jangan mengubah user scope atau permission secara tidak sengaja.
4. Jangan menambahkan telemetry, downloader, reverse shell, remote command, credential collection, atau koneksi eksternal tersembunyi.
5. Jangan menyimpan password/token di source atau log.
6. Jangan mengubah database migration/schema kecuali benar-benar diperlukan dan dijelaskan dulu.
7. Jangan menjalankan command destruktif seperti `rm -rf`, reset database, atau overwrite config produksi tanpa konfirmasi eksplisit.
8. Semua perubahan harus minimal, terisolasi, reversible, dan terdokumentasi.
9. Backup config, database, dan binary lama sebelum deployment.
10. Jika ada keraguan terkait keamanan, berhenti dan jelaskan risikonya.

Untuk Cloudflare Tunnel, pertimbangkan agar FBQ hanya listen di localhost jika tunnel berjalan di PC yang sama:

```yaml
server:
  listen: "127.0.0.1"
  port: 8080
```

Jangan menerapkan perubahan ini tanpa memastikan tunnel memang mengarah ke `http://127.0.0.1:8080`.

## 5. Isu performa video yang sedang didiskusikan

FBQ menggunakan komponen `plyrViewer.vue`. Video dirender menggunakan elemen HTML5 `<video>`, dengan Plyr sebagai UI/wrapper serta logic tambahan FBQ seperti:

- custom gesture/touch;
- swipe;
- double-tap seek;
- previous/next navigation;
- autoplay;
- subtitle;
- playback queue;
- audio/lyrics/metadata logic.

Setting user FBQ memiliki opsi native player, tetapi pada public shared files setting user tidak selalu tersedia sehingga public share dapat tetap memakai Plyr.

Gejala:

- Preview video terasa lag di Windows.
- Di iOS biasanya lebih parah.
- Akses lokal dan Cloudflare Tunnel terasa sama.
- Disk usage Windows kadang hanya sekitar 3%.
- Log menunjukkan HTTP `206 Partial Content` dan beberapa request video dibatalkan.

Jangan langsung menyimpulkan disk harus dipaksa bekerja 50%. Ukur dulu:

- status `206 Partial Content`;
- `Accept-Ranges`;
- `Content-Range`;
- ukuran request Range;
- Content-Type;
- waktu response;
- network send throughput;
- browser buffering;
- native player versus Plyr.

### Ide eksperimen yang belum boleh dianggap sudah diterapkan

Pertimbangkan opsi config baru, misalnya:

```yaml
frontend:
  forceNativeVideoPlayer: true
```

Perilaku yang diinginkan jika opsi ini benar-benar diimplementasikan:

- Semua video user login memakai native HTML5 player.
- Semua video public share juga memakai native HTML5 player.
- Audio tetap memakai logic/Plyr FBQ agar queue, lyrics, album art, dan playback mode tidak rusak.
- Download tetap mengambil file asli.
- Tidak ada transcoding otomatis hanya karena player diganti.
- Opsi harus reversible dengan `false`.

Tetapi sebelum mengimplementasikan, lakukan inspeksi source dan A/B test. Jangan mengganti semua Plyr atau memakai Video.js/Vidstack hanya karena terlihat lebih modern. Library baru tetap memakai decoder browser yang sama dan tidak otomatis memperbaiki codec, HTTP Range, atau bitrate.

## 6. FFmpeg dan preview video

FFmpeg sudah tersedia di Windows. Konsep yang dibahas:

```text
Preview video → boleh diproses/cache jika benar-benar diperlukan
Download      → selalu file asli tanpa FFmpeg
```

Namun fitur adaptive streaming/HLS atau transcoding cache **belum boleh dianggap sudah ada**. Jangan mengimplementasikannya tanpa desain lengkap untuk:

- cache key;
- invalidasi saat file berubah;
- batas storage;
- cleanup;
- concurrency FFmpeg;
- cancel/timeout;
- permission public share;
- password share;
- path traversal prevention;
- original download path.

Prioritas investigasi performa:

1. Bandingkan Plyr dan native `<video controls playsinline>`.
2. Verifikasi HTTP Range.
3. Verifikasi codec/container/bitrate.
4. Cek custom touch listener yang memakai `passive: false`.
5. Baru pertimbangkan remux/transcode/cache/HLS.

## 7. Pemahaman log Cloudflare dan keamanan

Jika log cloudflared menunjukkan:

```text
originService=http://localhost:8080
```

maka request diteruskan dari Cloudflare Tunnel ke FBQ lokal.

IP seperti `198.41.x.x` pada log cloudflared biasanya adalah IP edge Cloudflare, bukan otomatis IP visitor asli.

IP `127.0.0.1` berarti localhost/proxy lokal. IP `10.0.0.1` adalah alamat private/internal, biasanya gateway atau proxy. Identitas visitor asli perlu dilihat dari Cloudflare HTTP request logs/Analytics/Security Events, bukan hanya log cloudflared.

Path seperti berikut biasanya menunjukkan scanner WordPress otomatis:

```text
/wp-json/batch/v1
/wp/v2/posts/999999
/wordpress/wp-json/...
/blog/wp-json/...
```

Jika berasal dari IP eksternal dan berulang, itu kemungkinan internet-wide bot scan. Pastikan status, endpoint, auth, dan response sebenarnya sebelum menyimpulkan compromise.

## 8. Workflow jika ada update upstream

Jika saya mengirim link release baru, lakukan workflow berikut:

1. Verifikasi repository dan tag stable.
2. Baca release notes.
3. Cek apakah ada security fix.
4. Bandingkan commit/tag dengan source lokal.
5. Audit perubahan pada:
   - auth;
   - session/token;
   - public share;
   - download/Range;
   - storage/filesystem;
   - database;
   - permissions/scope;
   - FFmpeg/media;
   - frontend player.
6. Tentukan status:
   - urgent security update;
   - recommended but not urgent;
   - optional feature update;
   - unsafe/breaking update.
7. Jangan langsung upgrade produksi.
8. Buat branch/worktree atau salinan kerja.
9. Reapply custom patch secara minimal.
10. Jalankan formatter, unit test, integration test, frontend test, dan build.
11. Audit diff untuk memastikan tidak ada perubahan disk/delete/scope yang tidak disengaja.
12. Build Windows `.exe` 64-bit.
13. Hitung SHA-256.
14. Berikan changelog, risiko, test result, checksum, dan langkah rollback.
15. Sediakan config YAML yang sesuai versi baru.

Jika release tidak urgent, rekomendasikan tetap memakai versi stabil yang sedang berjalan sampai ada alasan kuat untuk update.

## 9. Validasi wajib sebelum binary diserahkan

Minimal lakukan:

```bash
gofmt -w <changed-go-files>
go test ./...
npm run test
npm run build
git diff --check
```

Untuk build Windows:

```bash
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build ...
```

Lalu verifikasi:

- file `.exe` tidak kosong;
- format Windows PE32+ amd64;
- source tag/commit tepat;
- frontend sudah ter-embed;
- checksum SHA-256;
- tidak ada file sensitif ikut terpaket;
- tidak ada perubahan tidak sengaja pada database/storage/delete logic.

Jangan menyatakan “100% bebas bug” atau “100% aman”. Jelaskan cakupan test dan keterbatasan verifikasi.

## 10. Cara bekerja yang diharapkan

Sebelum mengedit:

1. Baca file yang relevan.
2. Tampilkan rencana perubahan singkat.
3. Jelaskan risiko dan file yang akan disentuh.
4. Jangan menyentuh produksi.

Saat mengedit:

- gunakan patch minimal;
- pertahankan kompatibilitas config lama jika memungkinkan;
- jangan mengubah behavior yang tidak terkait;
- tulis test untuk fallback/config baru;
- gunakan nama field yang jelas;
- dokumentasikan config baru.

Setelah mengedit:

- jalankan test;
- periksa diff;
- build binary;
- berikan checksum;
- berikan langkah instalasi dan rollback;
- sebutkan bagian yang belum bisa diverifikasi tanpa menjalankan Windows/iOS secara langsung.

## 11. Tugas pertama AI yang menerima prompt ini

Mulai dengan:

1. Konfirmasi bahwa kamu memahami custom FBQ di atas.
2. Inspeksi repository dan cek apakah path/source masih tersedia.
3. Cek status git, tag, commit, dan file changed.
4. Jangan langsung rebuild atau mengubah source.
5. Jika tujuan saya adalah optimasi player, buat diagnosis/A-B test plan terlebih dahulu.
6. Bedakan dengan jelas antara:
   - fitur yang sudah benar-benar diimplementasikan;
   - ide yang baru didiskusikan;
   - fitur yang belum dibuat.

Jangan mengarang bahwa binary, test, atau patch tertentu sudah ada jika belum memverifikasinya langsung.
