# Prompt Review Update FileBrowser Quantum Custom

Saya punya instalasi custom **FileBrowser Quantum (FBQ)** di Windows. Kalau saya mengirim link release/update FBQ, jangan langsung meng-upgrade atau build. Tugas kamu adalah menilai update tersebut, lalu hanya membuat build baru jika memang aman dan diperlukan.

## Identitas proyek

- Repository upstream: `https://github.com/gtsteffaniak/filebrowser`
- Baseline sebelumnya: `v1.5.6-stable`
- Platform target: Windows 64-bit (`GOOS=windows`, `GOARCH=amd64`)
- Storage source utama: Windows local disk, contoh `D:\Your-Data`
- Akses eksternal: Cloudflare Tunnel + domain pribadi
- FFmpeg Windows tersedia dan dikonfigurasi melalui `integrations.media.ffmpegPath`

## Custom yang wajib dipertahankan

1. **Global share banner per source**
   ```yaml
   server:
     sources:
       - path: "D:\\Your-Data"
         name: "Your Data Source"
         config:
           defaultEnabled: true
           shareBanner: "Branding/Share-Banner.png"
   ```
   `shareBanner` relatif terhadap root source. Banner tersebut menjadi fallback `og:image` untuk public share. Custom banner dari Advanced options tetap harus mengalahkan fallback global.

2. **Share title tetap bawaan FBQ**
   - Jangan mengubah title otomatis seperti `Shared Files - Nama File/Folder`.
   - Fokus custom hanya pada banner/`og:image`.

3. **Sidebar**
   ```yaml
   frontend:
     disableHelp: false
     disableVersionText: false
     disableVersionLink: true
   ```
   Help tetap tampil, teks versi tetap tampil, tetapi teks versi tidak boleh menjadi link ke GitHub.

4. **Help**
   - Navigasi/shortcut Help tetap ada.
   - Link/paragraf Official Docs di dalam Help dihapus.

## Aturan keamanan

- Jangan mengubah logic disk/storage.
- Jangan mengubah operasi delete.
- Jangan mengubah scope user, permission, auth, atau public-share access tanpa alasan dan audit khusus.
- Jangan mengubah schema/database kecuali benar-benar diwajibkan upstream.
- Jangan menambahkan telemetry tersembunyi, downloader, reverse shell, credential collector, atau koneksi eksternal.
- Jangan menyentuh file asli pengguna.
- Download harus tetap mengambil file asli, bukan file hasil transcoding/cache.
- Jangan menjalankan command destruktif atau overwrite database/config produksi.
- Semua perubahan harus minimal, reversible, dan terdokumentasi.

## Prosedur setiap kali ada update

Saya akan memberikan link release, misalnya:

```text
<LINK RELEASE FBQ TERBARU>
```

Lakukan langkah berikut:

### 1. Verifikasi source

- Pastikan release adalah stable, bukan beta/alpha/nightly.
- Catat tag, commit, tanggal, dan parent commit.
- Jangan memakai `main` jika stable tag tersedia.

### 2. Analisis urgensi

Baca release notes dan source diff. Klasifikasikan:

- **URGENT**: auth bypass, unauthenticated access, path traversal, arbitrary file read/write, privilege escalation, public-share bypass, token/session vulnerability, database corruption, atau security fix penting.
- **RECOMMENDED**: bug fix penting, kompatibilitas Windows/FFmpeg/browser, stabilitas storage/share, atau perbaikan performa yang relevan.
- **OPTIONAL**: fitur baru yang tidak dipakai, UI, dokumentasi, atau optimasi kecil.
- **HOLD**: beta, breaking change, migrasi database berisiko, atau perubahan yang belum kompatibel dengan custom patch.

Jelaskan apakah aman tetap memakai versi sekarang jika update tidak urgent.

### 3. Audit area sensitif

Bandingkan versi lama dan baru untuk:

- authentication/session/token;
- user permissions dan scope;
- public share dan share password;
- download dan HTTP Range/`206 Partial Content`;
- filesystem/source resolver;
- delete/move/copy/upload;
- database dan migration;
- FFmpeg/media preview;
- frontend player/Plyr/native player;
- HTTP trusted headers dan Cloudflare compatibility.

### 4. Reapply custom patch

Jika update layak dipakai:

- ambil source dari stable tag;
- apply ulang global `shareBanner` fallback;
- apply ulang Help tanpa Official Docs;
- apply ulang version text/link behavior;
- pertahankan nama dan format config yang kompatibel;
- jangan menganggap patch lama otomatis cocok; periksa konflik manual.

### 5. Test wajib

Jalankan minimal:

```bash
gofmt -w <changed-go-files>
go test ./...
npm run test
npm run build
git diff --check
```

Audit diff final dan pastikan tidak ada perubahan tak terkait pada storage, delete, permission, scope, auth, database, atau download.

### 6. Build Windows

Jika semua aman:

```bash
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build ...
```

Verifikasi:

- binary adalah Windows PE32+ amd64;
- frontend ter-embed;
- binary tidak kosong;
- checksum SHA-256;
- source tag/commit benar;
- tidak ada file rahasia ikut terpaket.

### 7. Laporan akhir

Jawab dengan format:

```text
Release yang dianalisis:
Tag/commit:
Status: URGENT / RECOMMENDED / OPTIONAL / HOLD
Alasan:
Security fix:
Breaking changes:
Dampak ke custom kita:
Custom patch yang berhasil dipertahankan:
Test yang lolos:
Test yang tidak bisa dilakukan:
Risiko tersisa:
Rekomendasi deployment:
Langkah rollback:
SHA-256 binary:
```

Jika ada hal yang belum diverifikasi, katakan terus terang. Jangan pernah menyatakan “bebas bug 100%” atau “aman 100%”.

## Instruksi penting

Sebelum mengedit atau build, mulai dengan membaca link release yang saya berikan dan menjelaskan hasil analisisnya. Jika update tidak urgent, jangan membuat build baru tanpa alasan. Jika update urgent, tetap buat patch custom dan build hanya setelah test serta audit diff selesai.
