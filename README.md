# SSH VPS — auto-connect ke VPS lewat HP & Windows

Dua aplikasi, satu tujuan: begitu dibuka, langsung masuk ke shell VPS kamu tanpa
ngetik apa-apa lagi.

| | Android | Windows |
|---|---|---|
| File hasil | `sshvps.apk` | `sshvps.exe` |
| Auto-connect | ya, saat app dibuka | ya, saat exe dijalankan |
| Simpan password | AES-256 di Android Keystore | DPAPI (terikat akun Windows) |
| Shortcut perintah | tombol chip, bisa ditambah/edit | menu `Ctrl+]` |
| Copy-paste | teks bisa diblok & disalin, ada tombol Copy/Paste | seleksi mouse seperti console biasa |

---

## 1. Cara dapat APK + EXE (GitHub Actions — tanpa install apa pun)

1. Buat akun / login ke <https://github.com>.
2. Klik **New repository** → nama bebas (mis. `sshvps`) → **Private** boleh → **Create**.
3. Di halaman repo baru, klik **uploading an existing file**.
4. Ekstrak `sshvps-project.zip`, lalu **drag semua isinya** (folder `android`, `windows`,
   `.github`, `README.md`) ke halaman upload GitHub → **Commit changes**.
   > Penting: yang diupload adalah *isi* zip, bukan foldernya. Di root repo harus
   > kelihatan folder `android`, `windows`, dan `.github`.
5. Buka tab **Actions**. Build jalan otomatis (± 4–7 menit pertama kali).
6. Kalau sudah hijau ✅, buka run tersebut → bagian **Artifacts** di bawah:
   - `sshvps-apk` → berisi `sshvps.apk`
   - `sshvps-exe` → berisi `sshvps.exe`
7. Selain itu tiap build juga dibuatkan **Release** (tab Releases) — dari HP lebih gampang
   download APK-nya dari sini, tinggal tap.

### Pasang APK di HP
Buka file APK → Android akan minta izin **"Install unknown apps"** untuk browser/file
manager kamu → izinkan → Install. (APK ditandatangani dengan debug key, jadi bisa
langsung dipasang; tidak bisa dipublish ke Play Store apa adanya.)

---

## 2. Cara pakai — Android

1. Buka app → **Tambah server**.
2. Isi `root@namadomain.com` (boleh juga `root@103.20.11.5:2222`) dan password.
3. Centang **"Langsung konek saat aplikasi dibuka"** → Simpan.
4. Selesai. Mulai sekarang tiap buka app, langsung masuk shell.

Di layar terminal:

- **Chip di atas keyboard** = shortcut perintah. Tap = langsung jalan.
  Tahan (long-press) untuk edit/hapus, tap **＋ shortcut** untuk nambah.
- **Baris tombol** `Ctrl+C`, `Ctrl+D`, `Tab`, `↑`, `↓`, `ESC`, `Enter` untuk kontrol shell.
- **Copy**: blok teks langsung di layar output, atau pakai menu ⋮ →
  *Copy output terakhir* / *Copy semua output*.
- **Paste**: tombol clipboard di kiri kolom input.
- Menu ⋮ juga punya: sambung ulang, bersihkan layar, perbesar/perkecil teks,
  dan reset host key (kalau VPS di-reinstall).

Ganti server: tekan Back untuk kembali ke daftar server. Tahan salah satu kartu server
untuk edit / hapus / ganti mana yang auto-connect.

## 3. Cara pakai — Windows

Taruh `sshvps.exe` di mana saja (mis. Desktop), lalu dobel-klik.

- Pertama kali: dia tanya alamat SSH + password, lalu simpan.
- Berikutnya: dobel-klik = langsung masuk VPS.
- Saat sesi jalan, tekan **Ctrl+]** untuk membuka menu shortcut perintah.

Opsi baris perintah:

```
sshvps                     konek otomatis ke server default
sshvps root@domain.com     konek ke alamat tertentu
sshvps --menu              pilih dari daftar server
sshvps --add               tambah server baru
sshvps --list              lihat daftar server
sshvps --run "df -h"       jalankan 1 perintah lalu keluar
sshvps --config            tampilkan lokasi file config
```

Config disimpan di `%APPDATA%\sshvps\config.json`.
Password di dalamnya dienkripsi DPAPI — file itu **hanya bisa dibuka oleh akun Windows
kamu di komputer itu**. Kalau file-nya dicopy ke PC lain, password-nya tidak terbaca.

---

## 4. Catatan keamanan (baca sebentar)

Menyimpan password **root** di HP/laptop itu praktis tapi berisiko: siapa pun yang bisa
buka perangkat kamu, bisa masuk ke VPS. Yang sudah dilakukan aplikasi ini:

- Password tidak disimpan sebagai teks polos (Android Keystore / Windows DPAPI).
- Host key server di-pin. Kalau berubah, koneksi ditolak/diperingatkan — ini
  perlindungan dari penyadapan (MITM).

Yang sebaiknya kamu lakukan di sisi VPS:

- Aktifkan kunci layar/biometrik di HP.
- Kalau nanti mau lebih aman: bikin user non-root + SSH key, lalu matikan login
  password (`PasswordAuthentication no` di `/etc/ssh/sshd_config`). Bilang saja kalau
  mau saya tambahkan dukungan SSH key ke kedua aplikasi.

---

## 5. Build sendiri (opsional)

**Android** — butuh Android Studio (atau Android SDK + JDK 17):

```bash
cd android
./gradlew assembleRelease
# hasil: app/build/outputs/apk/release/app-release.apk
```

**Windows exe** — butuh Go 1.21+:

```bash
cd windows
go mod tidy
GOOS=windows GOARCH=amd64 go build -ldflags="-s -w" -o sshvps.exe .
```

## Struktur

```
android/    project Android (Kotlin, JSch)
windows/    client Go untuk Windows
.github/    workflow build otomatis (APK + EXE)
```
