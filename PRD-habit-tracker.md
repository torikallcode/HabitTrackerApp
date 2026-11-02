# PRD: Habit Tracker (Personal, Offline-First Mobile App)

> Dokumen ini adalah instruksi untuk AI agent. Bangun aplikasi sesuai spesifikasi di bawah. Jika ada hal yang ambigu, pilih opsi paling sederhana yang konsisten dengan dokumen ini, lalu catat keputusan itu di `DECISIONS.md`.

---

## 1. Ringkasan Produk

Aplikasi mobile habit tracker untuk penggunaan **pribadi (satu pengguna)**, **100% offline**, tanpa backend, tanpa akun, tanpa hosting, dan tidak untuk dikomersialkan. Setiap habit memiliki heatmap kontribusi bergaya GitHub. Halaman beranda menampilkan satu heatmap gabungan yang terisi berdasarkan penyelesaian seluruh habit pada hari tersebut.

## 2. Tujuan & Non-Tujuan

### Tujuan
- Menambah, mengubah, mengarsipkan, dan menghapus habit dengan cepat.
- Mencentang habit harian dengan 1-2 ketukan.
- Heatmap per habit dan heatmap gabungan di beranda (gaya GitHub contributions).
- Semua data tersimpan lokal di perangkat, bisa di-backup dan di-restore secara manual.
- Dapat dipasang dan dipakai di HP pribadi tanpa hosting dan tanpa store.

### Non-Tujuan (JANGAN dibangun)
- Login, registrasi, sinkronisasi cloud, backend, API eksternal.
- Multi-user, sosial, leaderboard, iklan, analytics pihak ketiga, monetisasi.
- Publikasi ke Play Store / App Store.

## 3. Platform & Rekomendasi Teknologi

**Asumsi platform utama: Android** (paling mudah dipasang langsung via APK tanpa store). Kode tetap harus lintas-platform sehingga iOS bisa dipakai nantinya.

| Kebutuhan | Pilihan | Alasan |
|---|---|---|
| Framework | **React Native + Expo (SDK terbaru, managed workflow)** | Satu codebase, tooling matang, sangat dikuasai AI agent, build APK mudah |
| Bahasa | **TypeScript (strict)** | Aman dari bug tipe data tanggal/skema |
| Navigasi | **Expo Router** | File-based routing, sederhana |
| Database lokal | **expo-sqlite** (+ **Drizzle ORM** opsional) | Offline, andal, query agregasi per tanggal mudah |
| State | **Zustand** | Ringan, minim boilerplate |
| Heatmap | **react-native-svg** (komponen custom) | Kontrol penuh atas grid ala GitHub, tanpa dependensi berat |
| Tanggal | **date-fns** | Ringan, bebas masalah mutasi |
| Notifikasi (opsional) | **expo-notifications** (lokal saja) | Pengingat harian tanpa server |
| Backup/Restore | **expo-file-system + expo-sharing + expo-document-picker** | Ekspor/impor file JSON |
| Haptics | **expo-haptics** | Umpan balik saat mencentang |
| Lint/format | ESLint + Prettier | Konsistensi |

**Alternatif yang diterima** jika agent lebih cocok: Flutter + Drift (SQLite) + `fl_chart`/CustomPainter. Pilih salah satu stack saja, jangan campur.

### Cara Distribusi ke HP Pribadi (tanpa hosting)
1. Build APK lokal: `eas build --platform android --profile preview --local` (atau `npx expo prebuild` lalu build dengan Android Studio / `./gradlew assembleRelease`).
2. Pindahkan file `.apk` ke HP (USB/Drive/kabel), aktifkan "Install dari sumber tidak dikenal", lalu pasang.
3. Aplikasi berjalan penuh offline setelah terpasang.
4. Sertakan `README.md` berisi langkah build dan install ini.
5. (iOS, opsional) Pasang via Xcode dengan Apple ID gratis (masa berlaku 7 hari) atau Apple Developer Program.

## 4. Pengguna & User Stories

Pengguna tunggal: pemilik aplikasi.

- Sebagai pengguna, saya ingin menambah habit dengan nama, warna, dan ikon agar mudah dibedakan.
- Sebagai pengguna, saya ingin mencentang habit untuk hari ini dengan satu ketukan.
- Sebagai pengguna, saya ingin mencentang/mengoreksi hari-hari sebelumnya jika lupa.
- Sebagai pengguna, saya ingin melihat heatmap tiap habit untuk mengetahui konsistensi saya.
- Sebagai pengguna, saya ingin melihat heatmap gabungan di beranda yang menunjukkan hari saya menyelesaikan semua habit.
- Sebagai pengguna, saya ingin data saya tidak hilang, sehingga saya butuh backup dan restore manual.

## 5. Fitur & Kebutuhan Fungsional

### 5.1 Beranda (Home)
1. **Header**: sapaan singkat + tanggal hari ini.
2. **Heatmap Gabungan** (gaya GitHub):
   - Grid kolom = minggu, baris = hari (Sen-Min atau Min-Sab, ikuti pengaturan; default Senin awal minggu).
   - Rentang tampilan default: 12 bulan terakhir (scroll horizontal, otomatis menggulir ke hari ini). Label bulan di atas, label hari di kiri.
   - Warna sel dihitung dari **rasio penyelesaian** hari itu (lihat 5.4).
   - Ketuk sel membuka bottom sheet berisi: tanggal, daftar habit yang dijadwalkan pada hari itu beserta status centang, dan bisa langsung diubah.
3. **Ringkasan statistik** (kartu kecil): streak hari sempurna saat ini, streak terpanjang, jumlah hari sempurna bulan ini.
4. **Daftar Habit Hari Ini**: tiap item menampilkan ikon, nama, dan tombol centang. Progress "3/5 selesai" ditampilkan.
5. Tombol **"+" (FAB)** untuk menambah habit.

### 5.2 Manajemen Habit
- **Tambah/Ubah habit** dengan field:
  - Nama (wajib, maks 60 karakter)
  - Deskripsi (opsional)
  - Warna (palet preset ± 10 warna; dipakai untuk heatmap habit tersebut)
  - Ikon/emoji (picker sederhana)
  - Jadwal: **setiap hari** (default) atau **hari tertentu** dalam seminggu (multi-pilih Sen-Min)
  - Pengingat (opsional): jam notifikasi lokal harian
- **Arsipkan** habit (disembunyikan dari daftar hari ini, riwayat tetap ada dan tetap dihitung di hari-hari lampau).
- **Hapus** habit (dengan konfirmasi; menghapus seluruh riwayat habit itu).
- **Urutkan ulang** habit (drag & drop atau tombol naik/turun).

### 5.3 Halaman Detail Habit
- Judul, ikon, warna.
- **Heatmap habit** (gaya GitHub, 12 bulan, warna sesuai warna habit; sel terisi = selesai, kosong = tidak; intensitas biner, atau 2 level jika diperlukan).
- Statistik: streak saat ini, streak terpanjang, total hari selesai, persentase penyelesaian 30 hari terakhir.
- Ketuk sel heatmap untuk toggle selesai/tidak pada tanggal tersebut (tanggal masa depan tidak bisa diubah).
- Tombol ubah, arsipkan, hapus.

### 5.4 Aturan Perhitungan Heatmap Gabungan (PENTING)
Untuk setiap tanggal `D`:
- `scheduled(D)` = habit yang **aktif pada D** (sudah dibuat pada/ sebelum D, belum diarsipkan pada D) **dan** dijadwalkan pada hari-dalam-minggu dari D.
- `done(D)` = habit di `scheduled(D)` yang dicentang pada D.
- `ratio(D) = done(D) / scheduled(D)`; jika `scheduled(D)` kosong maka sel berstatus **"tidak ada data"** (warna paling netral, tidak dihitung putus streak).

Level warna (4 tingkat + kosong), seperti GitHub:
| Level | Kondisi |
|---|---|
| 0 | ratio = 0 atau tidak ada data |
| 1 | 0 < ratio ≤ 0.33 |
| 2 | 0.33 < ratio ≤ 0.66 |
| 3 | 0.66 < ratio < 1 |
| 4 | ratio = 1 (**semua habit selesai**, "hari sempurna") |

Catatan: sesuai kebutuhan pengguna, level penuh (4) hanya muncul jika **semua** habit hari itu selesai. Level di bawahnya memberi gambaran progres parsial.

**Streak gabungan** = jumlah hari berturut-turut dengan ratio = 1 (hari tanpa jadwal dilewati, tidak memutus streak). Hari ini yang belum lengkap tidak memutus streak sampai hari berganti.

Perhitungan histori harus mengikuti **kondisi saat itu**: menambah habit baru tidak boleh mengubah warna hari-hari sebelum habit itu dibuat. Karena itu simpan `created_at` dan `archived_at`, dan jadwal habit bila diubah harus berlaku sejak tanggal perubahan (lihat model data, `habit_schedule_history`, atau cukup simpan snapshot sederhana dan catat keputusan di `DECISIONS.md`).

### 5.5 Pengaturan
- Hari awal minggu (Senin/Minggu).
- Tema: Terang / Gelap / Ikuti sistem.
- Jam pergantian hari (day cutoff, default 00:00; opsi sampai 04:00 untuk yang tidur larut).
- Ekspor data (JSON) dan Impor data (JSON, dengan konfirmasi "ganti semua data").
- Pengingat global on/off.
- Tentang aplikasi + versi.

### 5.6 Backup & Restore
- Ekspor: menghasilkan file `habit-tracker-backup-YYYYMMDD-HHmm.json` berisi versi skema, habit, dan seluruh log; dibagikan lewat share sheet (Drive, WhatsApp, file manager).
- Impor: pilih file JSON, validasi skema dan versi, tampilkan ringkasan (jumlah habit & log), minta konfirmasi, lalu timpa data dalam satu transaksi. Jika file tidak valid, data lama tidak boleh berubah.

## 6. Model Data (SQLite)

Semua tanggal disimpan sebagai string lokal `YYYY-MM-DD` (bukan timestamp UTC) agar tidak bergeser karena zona waktu.

```sql
CREATE TABLE habits (
  id            TEXT PRIMARY KEY,          -- UUID
  name          TEXT NOT NULL,
  description   TEXT,
  color         TEXT NOT NULL,             -- hex
  icon          TEXT,                      -- emoji / nama ikon
  schedule_days TEXT NOT NULL,             -- JSON array 0-6, mis. [1,2,3,4,5]; semua hari = [0..6]
  reminder_time TEXT,                      -- 'HH:mm' atau NULL
  sort_order    INTEGER NOT NULL DEFAULT 0,
  start_date    TEXT NOT NULL,             -- YYYY-MM-DD, habit mulai dihitung
  archived_at   TEXT,                      -- YYYY-MM-DD atau NULL
  created_at    TEXT NOT NULL
);

CREATE TABLE habit_logs (
  habit_id  TEXT NOT NULL REFERENCES habits(id) ON DELETE CASCADE,
  date      TEXT NOT NULL,                 -- YYYY-MM-DD
  completed INTEGER NOT NULL DEFAULT 1,
  created_at TEXT NOT NULL,
  PRIMARY KEY (habit_id, date)
);

CREATE INDEX idx_logs_date ON habit_logs(date);

CREATE TABLE settings (
  key   TEXT PRIMARY KEY,
  value TEXT NOT NULL
);
```

Aturan:
- Mencentang = insert/upsert log. Membatalkan centang = hapus baris log.
- Aktifkan `PRAGMA foreign_keys = ON`.
- Gunakan sistem migrasi berversi sejak awal (`PRAGMA user_version` atau Drizzle migrations).
- Pertimbangan perubahan jadwal: jika jadwal habit diubah, disarankan menambah tabel `habit_schedule_history(habit_id, effective_from, schedule_days)` agar riwayat heatmap tidak berubah mundur. Jika ingin lebih sederhana, boleh gunakan jadwal terkini untuk semua hari, namun dokumentasikan keterbatasan itu.

## 7. Arsitektur & Struktur Proyek

```
habit-tracker/
├─ app/                      # Expo Router (screens)
│  ├─ (tabs)/index.tsx       # Beranda
│  ├─ (tabs)/settings.tsx
│  ├─ habit/[id].tsx         # Detail habit
│  └─ habit/new.tsx          # Form tambah/ubah
├─ src/
│  ├─ db/                    # koneksi, skema, migrasi, repository
│  ├─ domain/                # logika murni: streak, rasio, level heatmap (WAJIB unit test)
│  ├─ components/            # Heatmap, HabitItem, StatCard, ColorPicker, dll
│  ├─ store/                 # Zustand
│  ├─ utils/                 # tanggal, backup/restore
│  └─ theme/                 # warna, tipografi, spacing
├─ __tests__/
├─ README.md
├─ DECISIONS.md
└─ package.json
```

Prinsip:
- Logika perhitungan (rasio, level, streak) berada di `src/domain` sebagai **fungsi murni** tanpa ketergantungan UI/DB, sehingga mudah diuji.
- Komponen `Heatmap` bersifat generik: menerima `Map<tanggal, level>` + palet warna + rentang tanggal, sehingga dipakai ulang untuk beranda dan detail habit.
- Query agregasi heatmap beranda dilakukan dengan satu/dua query SQL terindeks, bukan N+1 per hari.

## 8. Kebutuhan Non-Fungsional

- **Offline penuh**: tidak ada permintaan jaringan sama sekali; jangan tambahkan izin INTERNET jika tidak diperlukan.
- **Performa**: buka aplikasi < 2 detik; heatmap 12 bulan render lancar (≥ 55 FPS saat scroll); centang responsif < 100 ms (optimistic UI).
- **Keandalan data**: semua penulisan dalam transaksi; data bertahan setelah aplikasi ditutup paksa.
- **Privasi**: tidak ada telemetry, crash reporter pihak ketiga, atau analytics.
- **Aksesibilitas**: kontras warna memadai, target sentuh ≥ 44 px, label aksesibilitas pada tombol centang dan sel heatmap, mendukung dark mode.
- **Bahasa UI**: Bahasa Indonesia (siapkan string terpusat agar mudah diubah).
- **Penanganan tanggal**: semua operasi hari memakai tanggal lokal perangkat dan `day cutoff`; pastikan benar saat pergantian hari, DST/zona waktu berubah (Asia/Makassar = UTC+8, tanpa DST).

## 9. Desain & UX

- Gaya bersih, minimalis, mirip nuansa GitHub; sel heatmap kotak kecil dengan sudut sedikit membulat dan jarak antar-sel 2-3 px.
- Palet heatmap gabungan: 5 tingkat dari abu-abu netral ke hijau (atau satu warna aksen pilihan). Heatmap per habit memakai 2 tingkat dari warna habit tersebut (kosong/netral vs. warna penuh).
- Legenda "Kurang ▢▢▢▢▢ Lebih" di bawah heatmap.
- Animasi centang halus + haptic ringan.
- Navigasi bawah: **Beranda** dan **Pengaturan** (daftar habit & detail diakses dari beranda).
- Empty state yang ramah saat belum ada habit ("Mulai dengan menambah habit pertamamu").

## 10. Alur Utama

1. **Pertama kali buka** → empty state → tambah habit → kembali ke beranda dengan habit tampil di daftar hari ini.
2. **Centang habit** → log tersimpan → progres "x/y" berubah → sel hari ini di heatmap gabungan berubah level seketika → detail habit ikut diperbarui.
3. **Koreksi hari lalu** → buka detail habit atau ketuk sel di heatmap beranda → toggle → heatmap & streak dihitung ulang.
4. **Backup** → Pengaturan → Ekspor → share file JSON.
5. **Restore** → Pengaturan → Impor → pilih file → konfirmasi → data diganti.

## 11. Edge Case yang Wajib Ditangani

- Tidak ada habit terjadwal pada suatu hari → sel netral, tidak memutus streak.
- Habit baru dibuat hari ini → tidak mempengaruhi tanggal sebelum `start_date`.
- Habit diarsipkan → tidak dihitung di hari setelah `archived_at`, tetap dihitung di hari sebelumnya.
- Habit dihapus → seluruh log terhapus, heatmap gabungan dihitung ulang.
- Tanggal masa depan tidak bisa dicentang.
- Ganti hari saat aplikasi terbuka → UI menyegarkan "hari ini" otomatis (listener app foreground / timer).
- Nama habit duplikat → diperbolehkan, tetapi beri peringatan ringan.
- File impor rusak/versi tidak dikenal → tampilkan pesan error, data lama aman.

## 12. Pengujian

- **Unit test (wajib)** untuk `src/domain`: perhitungan rasio, level heatmap, streak (termasuk hari tanpa jadwal, habit baru, habit diarsipkan, pergantian hari/cutoff), serta validasi impor.
- **Test repository DB** dengan SQLite in-memory/mock: CRUD habit, upsert/hapus log, migrasi.
- **Pengujian manual** di perangkat nyata: checklist skenario di bagian 10 dan 11.

## 13. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Dapat menambah, mengubah, mengarsipkan, dan menghapus habit.
- [ ] Mencentang habit hari ini memperbarui daftar, progres, dan kedua heatmap seketika.
- [ ] Heatmap beranda mencapai level 4 hanya jika semua habit terjadwal hari itu selesai.
- [ ] Heatmap tiap habit tampil di halaman detail dan bisa dikoreksi per tanggal.
- [ ] Menambah habit baru tidak mengubah tampilan hari-hari sebelum habit itu dibuat.
- [ ] Data tetap ada setelah aplikasi ditutup dan dibuka ulang, termasuk tanpa internet/mode pesawat.
- [ ] Ekspor lalu impor menghasilkan data yang identik.
- [ ] Dark mode berfungsi.
- [ ] APK dapat dibangun secara lokal dan dipasang di HP sesuai `README.md`.
- [ ] Seluruh unit test lulus.

## 14. Tahapan Pengerjaan (Milestone)

1. **M1 - Fondasi**: inisialisasi Expo + TypeScript, routing, tema, koneksi SQLite, migrasi, ESLint/Prettier.
2. **M2 - Domain & Data**: skema DB, repository, fungsi murni rasio/level/streak + unit test.
3. **M3 - CRUD Habit**: form tambah/ubah, daftar hari ini, centang, arsip, hapus, urutan.
4. **M4 - Heatmap**: komponen `Heatmap` (SVG), heatmap gabungan di beranda, bottom sheet detail hari.
5. **M5 - Detail Habit**: heatmap per habit, statistik, koreksi per tanggal.
6. **M6 - Pengaturan & Backup**: tema, awal minggu, day cutoff, ekspor/impor JSON.
7. **M7 - Polish**: pengingat lokal, haptics, animasi, aksesibilitas, empty state.
8. **M8 - Rilis Pribadi**: build APK, `README.md` (langkah build/install), uji di HP nyata.

Setelah tiap milestone: jalankan test dan lint, pastikan aplikasi berjalan, lalu lanjut. Jangan menambah fitur di luar dokumen ini tanpa mencatatnya di `DECISIONS.md`.

## 15. Ide Masa Depan (di luar cakupan saat ini)

Catatan/komentar per hari, target mingguan (mis. 3x/minggu), habit numerik (mis. minum 8 gelas), widget layar utama Android, kategori/tag habit, statistik mingguan/bulanan, kunci aplikasi (PIN/biometrik).
