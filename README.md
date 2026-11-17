<p align="center"><img src="docs/logo.svg" width="128" alt="logo"></p>

# Module Viewer Simulation (MVS) 1.0 — codename R

Aplikasi Android (dan Windows) untuk **melihat dan mensimulasikan** module root (Magisk / KernelSU / APatch) **tanpa memasangnya**.
Isi module hanya diekstrak ke penyimpanan privat aplikasi. Skrip module hanya berjalan di mini-shell simulasi dalam memori, **tidak pernah di perangkat**.

## Fitur

- **Pilih module `.zip`** lalu lihat proses instalasi bergaya terminal KSU (`ui_print`/`echo` dari `customize.sh`, perintah lain ditandai `[SIM]`).
- **Lihat isi** module: pohon file + penampil teks bernomor baris.
- **Analisis keamanan statis**: pola `rm`/`dd`/`chmod`/format ke partisi penting, `curl | sh`, obfuskasi, variabel yang disusun, dan lain-lain, lengkap dengan skor risiko.
- **Native Binary Analyzer**: pembaca ELF (ARM64, ARM32, x86, x86-64) untuk binary di dalam module.
- **WebUI module** dibuka di dalam aplikasi dengan API `ksu.*` palsu (perintah dicatat/disimulasikan, jaringan diblokir), termasuk `listPackages`, `getPackagesInfo`, dan ikon aplikasi (`ksu://icon/<paket>`, `getPackagesIcons`). Tombol **←** di pojok kiri atas untuk kembali.
- **Simulasi Android**: perangkat palsu dengan CPU/GPU/thermal/baterai/memori yang bisa dibaca dan ditulis lewat `/sys` dan `/proc` virtual.
  - **MediaTek Helio G99 (MT6789)** — bawaan.
  - **Qualcomm Snapdragon 695 (SM6375)** — **BETA**.
- **Simulasi daemon** — **BETA**: `service.sh`/`post-fs-data.sh`/`boot-completed.sh` dijalankan sebagai proses sungguhan di penjadwal simulator dengan waktu virtual (`while true` + `sleep`, `nohup`, `trap`, `kill`, `wait`, `inotifyd`, pidfile, deteksi aplikasi di depan). Bisa dimatikan lewat pengaturan **Simulasikan daemon?**.

> Analisis hanya mencocokkan pola teks. Bukan jaminan keamanan; baca kode module sebelum memasangnya sungguhan.

## Versi Windows

Folder [`windows/`](windows) membangun:

- `ModuleViewerSim.html` — aplikasi satu file, buka di Edge/Chrome.
- `ModuleViewerSim.exe` — launcher kecil yang menulis HTML ke `%TEMP%` lalu membuka Edge mode aplikasi.
- `ModuleViewerSim_Windows_<versi>-W.zip` — keduanya + `BACA-DULU.txt`. Edisi Windows berlabel **1.0.0 (W)**: tampilan fluid mengikuti ukuran jendela, pintasan keyboard/mouse, dan zona lepas file .zip (gaya di `windows/windows.css`). Ikon `.exe` ada di `windows/icon.ico` (dibuat oleh `windows/make_icon.py`, butuh Pillow; hasilnya sudah di-commit sehingga CI tidak memerlukannya). Ukuran minimum jendela 420 × 520.

```
python3 windows/build_windows.py     # hasil di dist/
```

`.exe` belum ditandatangani penerbit, jadi SmartScreen/antivirus bisa memberi peringatan; file `.html` yang sama bisa dipakai langsung. Module tidak pernah dijalankan di komputer; WebUI-nya berjalan di iframe ber-sandbox tanpa jaringan.

## Build lewat GitHub Actions

1. Buat repo kosong di GitHub, lalu dari folder ini:
   ```
   git init
   git add .
   git commit -m "Module Viewer Simulation 1.0"
   git branch -M main
   git remote add origin https://github.com/<user>/<repo>.git
   git push -u origin main
   ```
2. Buka tab **Actions** → workflow **Build Release**. Hasilnya di **Artifacts**:
   - `ModuleViewerSim-android-release` — APK release bertanda tangan + `SHA256SUMS.txt`
   - `ModuleViewerSim-windows` — zip versi Windows
3. **Rilis otomatis**: beri tag lalu push, GitHub Release dibuat dengan APK dan zip Windows terlampir.
   ```
   git tag v1.0.0
   git push origin v1.0.0
   ```

## Struktur

```
app/                          proyek Android (Kotlin + WebView)
  src/main/assets/ui/         UI (HTML/JS): index.html, sim.js (simulator), shim.js (API ksu), elf*.js, a64/a32/x86.js
  src/main/java/.../          ModuleManager, ModuleAnalyzer, MainActivity, WebUiActivity
windows/                      versi Windows: analyzer.js, desktop.js, bridge.js, build_windows.py
.github/workflows/build.yml   CI: APK release + Windows + GitHub Release
CHANGELOG.md                  riwayat perubahan
```

Syarat build lokal: JDK 17, Android SDK (compileSdk 34), Gradle 8.7 → `gradle assembleRelease`.
