<p align="center">
  <img src="docs/logo.svg" width="128" alt="Logo Module Viewer Simulation">
</p>

<h1 align="center">Module Viewer Simulation</h1>

<p align="center">
  <b>Lihat, bedah, dan simulasikan module root tanpa memasangnya.</b><br>
  Versi <b>1.0.0</b> · codename <b>R</b> · Android &amp; Windows · 🌐 Indonesia / English
</p>

<p align="center">
  <a href="https://github.com/NOSENSEGMWK/Module-Viewer-Simulation-/releases/latest">⬇️ Unduh</a> ·
  <a href="#fitur">Fitur</a> ·
  <a href="#cara-pakai">Cara pakai</a> ·
  <a href="#versi-windows-100-w">Windows</a> ·
  <a href="#batasan--catatan">Batasan</a> ·
  <a href="README.en.md">🇬🇧 English</a>
</p>

<p align="center">
  Dibuat oleh <b>Towa</b> with AI (<b>Claude</b>)
</p>

---

## Tentang

Memasang module root yang belum dikenal itu berisiko: satu baris `rm -rf` atau `dd` ke partisi yang salah bisa membuat
perangkat tidak mau menyala. **Module Viewer Simulation (MVS)** membantu kamu *melihat dulu* apa yang dilakukan sebuah
module sebelum memasangnya sungguhan. Semua skrip module hanya berjalan di mini-shell simulasi di dalam memori aplikasi.

- ✅ **Tidak butuh root**
- ✅ **Tidak ada yang dijalankan di perangkat/komputermu**: module hanya diekstrak ke penyimpanan privat aplikasi
- ✅ **Tanpa internet**: aplikasi tidak punya akses internet sendiri, dan WebUI module berjalan dengan jaringan diblokir
- ✅ **Tanpa dependensi luar**: UI dan simulator murni HTML/JavaScript di dalam WebView
- ✅ **Dua bahasa**: Bahasa Indonesia dan English, bisa diganti kapan saja di Pengaturan

## Fitur

### 📦 Memeriksa module
| Fitur | Keterangan |
|---|---|
| **Pilih module `.zip`** | Mendukung format module Magisk / KernelSU / APatch (butuh `module.prop`). Ukuran zip maksimal 350 MB, dengan perlindungan *zip-slip* dan batas ukuran hasil ekstrak. |
| **Kartu informasi** | Nama, versi, pengembang, deskripsi, banner, ukuran, jumlah file, dan penanda **WEBUI** / **AKSI**. |
| **Tombol Aksi** | Module yang punya `action.sh` mendapat tombol **Aksi** di sebelah kiri tombol WebUI, seperti di manajer root. Skripnya dijalankan di simulator dengan keluaran bergaya terminal dan kode keluar. |
| **Proses instalasi (simulasi)** | Log bergaya terminal KSU: `ui_print`/`echo` dari `customize.sh` ikut tampil, perintah lain ditandai `[SIM]`, dan baris berbahaya diberi label tingkat risiko. Tidak ada yang benar-benar dijalankan. |
| **Lihat isi** | Pohon file lengkap + penampil teks bernomor baris. File biner ditandai, ELF bisa dibuka di analyzer. |
| **Ikon APK** | File `.apk` di dalam module tampil dengan ikon dan label tersendiri. |

### 🛡️ Analisis keamanan statis
Memindai pola berbahaya pada skrip tanpa mengeksekusinya, lalu memberi **skor dan tingkat risiko**
(*aman · ditinjau · rendah · sedang · tinggi · kritis*):

- Operasi destruktif ke **partisi** (`dd`, `flash_image`, `fastboot`, format, tulis ke `/dev/block/...`), dengan penilaian dampak per partisi (bootloader, boot, modem, nvram, sistem, dst.)
- Penghapusan / perubahan izin direktori penting (`rm -rf`, `chmod -R`, `find -delete`, `chown`)
- Unduh lalu jalankan (`curl … | sh`), kemungkinan *reverse shell*, koneksi jaringan mentah
- Obfuskasi: `base64 -d`, `eval`, rangkaian hex, nama perintah yang disusun dari variabel, kata sensitif yang dipecah
- Fork bomb, `setenforce 0`, `remount rw`, `pm install/uninstall`, `reboot`
- **Pelacak variabel**: `X=/system; rm -rf "$X"` tetap terdeteksi karena nilai variabelnya diuraikan
- Kode WebUI (`.js`/`.html`) ikut dipindai

### ⚙️ Native Binary Analyzer
Pembaca file **ELF** untuk binary di dalam module (ARM64, ARM32, x86, x86-64), langsung dari daftar file atau dari temuan analisis.

### 🌐 WebUI module
WebUI bawaan module dibuka **di dalam aplikasi** dengan API `ksu.*` palsu:
`exec` / `spawn` (disimulasikan oleh mini-shell), `moduleInfo`, `toast`, `fullScreen`, serta API aplikasi terpasang ala KernelSU:
`listPackages`, `listUserPackages`, `listSystemPackages`, `listAllPackages`, `getPackagesInfo`, `getPackagesIcons`, dan ikon aplikasi lewat `ksu://icon/<paket>`.
Jaringan diblokir. Tombol **←** di pojok kiri atas untuk kembali.

### 📱 Simulasi Android
Perangkat palsu dengan nilai yang bisa **dibaca dan ditulis** lewat `/sys` dan `/proc` virtual, termasuk CPU (cluster, governor, frekuensi), GPU, thermal, baterai, memori, properti sistem, dan perintah `dumpsys` / `getprop` / `settings`.

| Chipset | Status |
|---|---|
| **MediaTek Helio G99 (MT6789)**, Mali-G57 | Bawaan |
| **Qualcomm Snapdragon 695 5G (SM6375)**, Adreno 619 | <img alt="BETA" src="https://img.shields.io/badge/-BETA-d32f2f?style=flat-square"> |

Bisa memilih root manager yang disimulasikan (**KernelSU / Magisk / APatch**), nama perangkat, dan codename.

### 🔁 Simulasi daemon <img alt="BETA" src="https://img.shields.io/badge/-BETA-d32f2f?style=flat-square">
`post-fs-data.sh`, `service.sh`, dan `boot-completed.sh` dijalankan sebagai **proses sungguhan di penjadwal simulator** dengan **waktu virtual**:

- Urutan boot nyata, `system.prop` dipasang, dan `sys.boot_completed` berubah menjadi `1` pada waktunya
- Daemon hidup terus: `while true; do …; sleep N; done &`, `nohup`, `setsid`, `( … ) &`, fungsi loop, pidfile
- Proses & sinyal: `$!`, `$$`, `pidof`, `pgrep`, `pkill`, `kill`, `trap`, `wait`, `ps`, `top`, `/proc/<pid>`
- Pemicu: `inotifyd`, aplikasi di depan (`dumpsys window`), layar nyala/mati, status charger
- Kontrol: kecepatan waktu 1x/10x/60x/600x, lompat waktu, jeda, reboot, log per proses, riwayat proses yang sudah keluar, logcat, peringatan *busy-loop*
- Bisa dimatikan lewat pengaturan **Simulasikan daemon?**

### 🌍 Bahasa & pembaruan
- **Bahasa Indonesia / English** di Pengaturan, berlaku langsung di seluruh aplikasi (UI, hasil analisis, log instalasi, layar daemon, dan Native Binary Analyzer). Nama, deskripsi, isi file, dan keluaran skrip milik module sengaja tidak diterjemahkan.
- **Cek pembaruan**: Pengaturan memuat tombol yang membuka [halaman proyek](https://github.com/NOSENSEGMWK/Module-Viewer-Simulation-) dan [rilis terbaru](https://github.com/NOSENSEGMWK/Module-Viewer-Simulation-/releases/latest) di browser. Aplikasi sendiri tidak terhubung ke internet.

## Cara pakai

### Android
1. Unduh APK terbaru dari halaman **[Releases](https://github.com/NOSENSEGMWK/Module-Viewer-Simulation-/releases/latest)** lalu pasang (Android 8.0 / API 26 ke atas; izinkan pemasangan dari sumber ini bila diminta).
2. Buka aplikasi → **Pilih module** → pilih file `.zip` module.
3. Gunakan tombol **Lihat isi**, **Proses instalasi**, **Hasil analisis**, **Simulasi daemon**, **Aksi** (bila ada `action.sh`), dan **Buka** (WebUI).
4. Ikon ⚙️ di pojok kanan atas untuk pengaturan (bahasa, chipset, root manager, simulasi daemon, nama perangkat, cek pembaruan).

### Versi Windows 1.0.0 (W)
Tersedia di **[Releases](https://github.com/NOSENSEGMWK/Module-Viewer-Simulation-/releases/latest)** sebagai `ModuleViewerSim_Windows_1.0.0-W.zip`.

1. Ekstrak zip, klik dua kali `ModuleViewerSim.exe` (terbuka lewat Microsoft Edge mode aplikasi). Bila `.exe` diblokir antivirus, buka `ModuleViewerSim.html` langsung di Edge/Chrome. Isinya sama.
2. Tekan **Pilih module** (atau `Ctrl+O`), atau **seret file `.zip`** ke jendela.
3. Tampilan mengikuti ukuran jendela (minimum 420 × 520). Pintasan: `Esc` / `Alt+←` / tombol mouse "mundur" untuk kembali.

Syarat: Windows 10/11 64-bit, Microsoft Edge atau Google Chrome terbaru. `.exe` belum ditandatangani penerbit, jadi SmartScreen mungkin menampilkan peringatan ("More info" → "Run anyway").

## Batasan & catatan

- **Analisis = pencocokan pola teks, bukan jaminan keamanan.** Hasil "aman" tidak berarti module pasti aman; baca kode module sebelum memasangnya sungguhan.
- **Mini-shell bukan Android asli.** Perilaku sebagian perintah, angka sensor, dan tabel frekuensi adalah data simulasi.
- **Snapdragon 695 dan simulasi daemon masih BETA**: tabel frekuensi/level dan sebagian node adalah perkiraan yang masuk akal, bukan dump dari perangkat asli. Laporan bug sangat membantu.
- **Binary ELF native tidak dieksekusi.** Binary yang dijalankan di latar belakang hanya muncul sebagai proses dengan label "ELF native".
- **Ikon aplikasi di WebUI bersifat generik** (huruf + warna per paket), bukan logo asli aplikasi.
- **Terjemahan English** dibuat dengan mencocokkan teks yang tampil dengan kamus; teks yang belum ada di kamus tetap tampil dalam Bahasa Indonesia. Koreksi terjemahan sangat diterima.
- Di Windows, WebUI module yang dipecah menjadi banyak file ES-module (`import './x.js'`) mungkin tidak berjalan; WebUI satu file / satu bundle berjalan normal.

## Dibangun dengan

- **Android:** Kotlin + WebView; UI dan simulator (mini-shell, penjadwal proses, sistem berkas virtual) ditulis dengan HTML/JavaScript murni
- **Windows:** satu file HTML mandiri + launcher `.exe` kecil (PE64) yang membuka Edge mode aplikasi, WebUI module berjalan di `iframe` ber-sandbox dengan jaringan diblokir
- **Build:** GitHub Actions menghasilkan APK release dan paket Windows secara otomatis

## Kredit

Dibuat oleh **Towa** with AI (**Claude**, model AI dari Anthropic). Rancangan, arahan, dan pengujian oleh Towa; kode dikembangkan bersama Claude.

## Disclaimer

Proyek ini tidak berafiliasi dengan, dan tidak didukung oleh, Magisk, KernelSU, APatch, MediaTek, Qualcomm, Google, atau Microsoft.
Semua nama merek dan merek dagang adalah milik pemiliknya masing-masing dan hanya disebut untuk menjelaskan kompatibilitas.
Gunakan dengan risiko sendiri. Aplikasi ini memeriksa dan mensimulasikan, **bukan** menjamin keamanan atau keberhasilan pemasangan module.
