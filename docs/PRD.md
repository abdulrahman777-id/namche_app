**PRODUCT REQUIREMENTS DOCUMENT**

**Aplikasi Sekolah SMAN 6 Yogyakarta**

Website + PWA berbasis Laravel dan PostgreSQL (namche_app)

| **Informasi**      | **Isi**                                                                                                   |
|--------------------|-----------------------------------------------------------------------------------------------------------|
| Versi              | 1.0 (draf siap eksekusi)                                                                                  |
| Tanggal            | 24 November 2026                                                                                          |
| Repositori         | abdulrahman777-id/namche_app                                                                              |
| Tim                | 2 orang pengembang                                                                                        |
| Server             | VPS 2 core / 2 GB RAM                                                                                     |
| Cakupan MVP        | Data induk, jadwal, presensi per jam, jurnal mengajar, perizinan, notifikasi WhatsApp (Fonnte), dashboard |
| Dokumen pendamping | docs/RANCANGAN.md di repositori (rancangan diskusi awal)                                                  |

*Dokumen ini ditulis agar dapat dipakai langsung sebagai acuan eksekusi bertahap oleh Claude (Claude Code di Claude Desktop). Setiap persyaratan diberi kode (FR-xx) dan kriteria penerimaan, dan Lampiran B berisi urutan perintah per fase.*

1\. Ringkasan dan tujuan

1.1 Latar belakang

SMAN 6 Yogyakarta memiliki sekitar 860 siswa (24 rombel), 56 guru dan 30 karyawan. Presensi, jurnal mengajar dan perizinan masih perlu dipusatkan dalam satu sistem agar kehadiran siswa dan kehadiran guru mengajar dapat dipantau, dan agar orang tua mendapat informasi tepat waktu. Data induk saat ini berasal dari ekspor Excel Dapodik.

1.2 Tujuan produk

- Memusatkan data induk guru, karyawan, siswa, orang tua dan kelas, dengan impor berulang dari Excel Dapodik.

- Mencatat presensi siswa **setiap jam pelajaran** oleh guru yang mengajar, lengkap dengan **jurnal mengajar** (materi dan kemajuan kelas).

- Menyediakan alur perizinan yang jelas: izin harian oleh orang tua, izin per jam oleh siswa, lengkap dengan persetujuan berjenjang.

- Memberi Kepala Sekolah dan Waka Kurikulum dashboard kehadiran siswa dan supervisi kehadiran guru mengajar.

- Mengirim notifikasi WhatsApp otomatis melalui gateway Fonnte, dengan halaman pengaturan yang dapat dikelola admin.

- Dapat dipasang sebagai PWA untuk siswa dan orang tua (mobile-first).

1.3 Indikator keberhasilan

| **Indikator**                                                       | **Target**                      |
|---------------------------------------------------------------------|---------------------------------|
| Sesi pelajaran yang presensinya terisi di hari yang sama            | ≥ 95% setelah 1 bulan pemakaian |
| Waktu guru mengisi presensi + jurnal satu kelas                     | ≤ 2 menit                       |
| Pengajuan izin yang diproses (disetujui/ditolak) dalam 1 hari kerja | ≥ 90%                           |
| Notifikasi WhatsApp terkirim berhasil                               | ≥ 95% dari yang dijadwalkan     |
| Halaman presensi pada ponsel kelas menengah (jaringan 4G)           | Termuat ≤ 3 detik               |

1.4 Di luar lingkup (MVP)

- Modul nilai/rapor, SPP/keuangan, PPDB, perpustakaan, e-learning.

- Presensi dengan QR/GPS/biometrik, serta mode offline dengan sinkronisasi antrean (PWA hanya cache baca).

- Impor jadwal dari aSc Timetables (ditunda; fase jadwal memakai input manual dan impor CSV generik lebih dulu).

- Aplikasi mobile native dan integrasi API Dapodik langsung.

2\. Pengguna dan role

Role adalah jabatan. Hak yang bergantung pada kelas, jadwal atau tanggal ditentukan lewat **penugasan** (wali kelas, guru BK, waka, guru piket). Satu pengguna dapat memegang beberapa role.

| **Role**             | **Deskripsi dan hak utama**                                                                                                                                                                     |
|----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Super Admin          | Seluruh sistem, akun, pengaturan (termasuk Fonnte), impor, audit log.                                                                                                                           |
| Admin/TU             | Kelola data induk, impor Dapodik, kelola jadwal pelajaran dan jadwal piket, reset password, tugaskan guru pengganti.                                                                            |
| Kepala Sekolah       | Hanya lihat: dashboard dan laporan seluruh sekolah, jurnal mengajar, catatan BK.                                                                                                                |
| Wakil Kepala Sekolah | Per bidang (Kurikulum, Kesiswaan, Sarpras, dst.; bidang dapat ditambah lewat data). Waka Kurikulum: melihat jurnal dan supervisi kehadiran guru. Waka Kesiswaan: menyetujui dispensasi sekolah. |
| Wali Kelas           | Kelas yang diampu: rekap harian, koreksi presensi, menyetujui izin harian.                                                                                                                      |
| Guru                 | Mengisi presensi dan jurnal pada sesi yang diajar; menyetujui izin per jam pada jam yang diajar.                                                                                                |
| Guru Piket           | Guru yang bertugas piket pada tanggal itu: menyetujui izin per jam. Ditentukan lewat jadwal piket.                                                                                              |
| Guru BK              | Siswa di kelas yang ditugaskan: riwayat presensi dan catatan BK (rahasia).                                                                                                                      |
| Siswa                | Melihat riwayat presensi sendiri, mengajukan izin per jam.                                                                                                                                      |
| Orang Tua            | Melihat data dan presensi anak, mengajukan izin harian.                                                                                                                                         |

3\. Log keputusan produk

Keputusan berikut sudah disepakati dalam diskusi dan menjadi dasar dokumen ini.

| **\#** | **Keputusan**                                                                                                    | **Catatan**                             |
|--------|------------------------------------------------------------------------------------------------------------------|-----------------------------------------|
| K1     | Stack: Laravel (versi stabil terbaru), PostgreSQL, Blade + Livewire + Tailwind, Filament untuk panel admin.      | Tim 2 orang, satu codebase              |
| K2     | Presensi dicatat per jam pelajaran oleh guru pengajar, bersama jurnal mengajar.                                  | Jam pelajaran seragam untuk semua kelas |
| K3     | Koreksi presensi setelah hari berlalu hanya oleh wali kelas (tercatat di audit log).                             |                                         |
| K4     | Penugasan Guru BK per kelas.                                                                                     | Seperti wali kelas                      |
| K5     | Waka dipisah per bidang; bidang dapat ditambah belakangan.                                                       | Berbasis data                           |
| K6     | Kepala Sekolah dan Waka Kurikulum boleh melihat jurnal mengajar untuk supervisi.                                 |                                         |
| K7     | Izin harian diajukan orang tua, disetujui wali kelas. Dispensasi sekolah disetujui Waka Kesiswaan.               |                                         |
| K8     | Izin per jam (tidak mengikuti pelajaran) diajukan siswa; guru pengajar dan guru piket keduanya wajib menyetujui. | Dikonfirmasi pemilik produk             |
| K9     | Guru piket: jadwal tetap per hari, dengan penggantian per tanggal oleh TU.                                       | Menutup kedua opsi                      |
| K10    | Login orang tua = NIS anak (kolom NIPD); siswa = NISN; guru = email/NIP. Password awal acak dan wajib diganti.   | NIS mudah ditebak, jadi password wajib  |
| K11    | Notifikasi memakai WhatsApp via Fonnte; token dan nomor pengirim diatur nanti lewat halaman pengaturan.          | Nomor khusus sekolah tersedia           |
| K12    | Impor Dapodik (guru dan siswa) berupa upsert; 18 guru yang belum ada di file diinput manual.                     | Kunci: NISN, NIK                        |
| K13    | Impor jadwal aSc Timetables ditunda; sementara input manual + impor CSV.                                         | File contoh belum ada                   |

4\. Persyaratan fungsional

Prioritas: **P0** wajib MVP, **P1** penting dan dikerjakan setelah P0 selesai. Setiap FR memiliki kriteria penerimaan (KP) yang dapat diuji.

4.1 Autentikasi dan akun

FR-01 Login dan pembuatan akun (P0)

Pengguna masuk dengan username sesuai jenisnya. Akun dibuat otomatis saat impor, dengan password awal acak.

**Kriteria penerimaan:**

- Orang tua login dengan NIS anak, siswa dengan NISN, guru/staf dengan email atau NIP.

- Password awal acak per akun; login pertama memaksa penggantian password (minimal 8 karakter).

- Pembatasan percobaan login (rate limiting) dan penguncian sementara setelah gagal berulang.

- Admin dapat mereset password dan mencetak daftar password awal per kelas (PDF), yang hanya dapat diunduh sekali setelah pembuatan atau reset.

- Satu NIS = satu akun orang tua. Siswa tanpa NIS ditandai di laporan impor dan tidak dibuatkan akun orang tua sampai NIS diisi.

FR-02 Role, permission dan penugasan (P0)

Role dan permission memakai spatie/laravel-permission. Hak berbasis kelas/jadwal dicek lewat Policy dan tabel penugasan.

**Kriteria penerimaan:**

- Seeder membuat semua role dan permission sesuai Bagian 6.

- Admin dapat menugaskan wali kelas, guru BK, waka (dengan bidang) dan guru piket per tahun ajaran/tanggal.

- Pergantian tahun ajaran tidak membutuhkan perubahan role; cukup penugasan baru.

- Uji otomatis membuktikan: orang tua tidak dapat membuka data anak orang lain; guru tidak dapat mengisi sesi guru lain; wali kelas hanya mengoreksi kelasnya.

4.2 Data induk

FR-03 Sekolah, tahun ajaran, semester (P0)

Pengaturan identitas sekolah, tahun ajaran aktif dan semester aktif.

**Kriteria penerimaan:**

- Hanya satu tahun ajaran dan satu semester yang aktif pada satu waktu.

- Data presensi dan penugasan terikat tahun ajaran, sehingga arsip tahun lalu tetap dapat dilihat.

FR-04 Guru dan karyawan (P0)

CRUD guru dan karyawan, termasuk akun login, status kepegawaian dan tugas tambahan.

**Kriteria penerimaan:**

- Form tambah/ubah guru manual (untuk 18 guru yang belum ada di Dapodik) dan **template Excel** unduhan dengan kolom yang sama seperti impor.

- NIK unik; NIP dan NUPTK boleh kosong (guru honor).

- Daftar dapat dicari dan disaring (status, tugas tambahan).

FR-05 Siswa, orang tua, kelas, rombel (P0)

CRUD siswa dan orang tua, kelas per tahun ajaran, dan penempatan siswa (rombel).

**Kriteria penerimaan:**

- NISN unik dan 10 digit; disimpan sebagai teks (tidak kehilangan angka 0 di depan).

- Siswa dapat dipindahkan antar kelas; riwayat penempatan tersimpan.

- Orang tua dibuat dari data ibu, lalu ayah atau wali bila ada. Nomor WhatsApp orang tua dikelola di FR-16.

4.3 Impor data

FR-06 Impor Excel Dapodik (guru dan siswa) (P0)

Unggah file ekspor Dapodik, pratinjau, lalu terapkan. Dapat diulang tiap semester tanpa menggandakan data.

**Kriteria penerimaan:**

- Membaca format sebenarnya: header guru di baris ke-5; header siswa dua tingkat (baris ke-5 dan 6) dengan grup Data Ayah, Data Ibu dan Data Wali.

- Upsert berdasarkan NISN (siswa) dan NIK (guru). Mengimpor ulang file yang sama tidak mengubah jumlah data.

- Halaman pratinjau menampilkan jumlah baris baru, berubah, tidak berubah dan bermasalah, dengan laporan error per baris yang dapat diunduh.

- Kelas dan penempatan siswa dibentuk otomatis dari kolom **Rombel Saat Ini** (24 rombel).

- Saran role awal (wali kelas, waka, dll.) dari kolom **Tugas Tambahan**, ditampilkan sebagai usulan yang harus dikonfirmasi admin.

- Hanya kolom yang dibutuhkan yang diimpor (Lampiran A). Kolom sensitif lain tidak disimpan.

- Impor berjalan sebagai job antrean dan tidak melebihi memori VPS 2 GB (baca per potongan/streaming).

FR-07 Impor jadwal (P1)

Jadwal diinput manual dan dapat diimpor dari CSV generik. Impor aSc Timetables ditunda sampai file contoh tersedia.

**Kriteria penerimaan:**

- Format CSV: kelas, hari, jam ke-, mata pelajaran, guru; dengan pratinjau dan laporan error.

- Kolom kode_asc pada guru, kelas dan mapel disiapkan di skema agar impor aSc kelak tidak membutuhkan migrasi besar.

4.4 Jadwal dan piket

FR-08 Jam pelajaran, jadwal pelajaran, jadwal piket (P0)

Satu tabel jam pelajaran global. Jadwal pelajaran per kelas per hari. Jadwal piket tetap per hari dengan penggantian per tanggal.

**Kriteria penerimaan:**

- Admin/TU dapat membuat dan mengubah jam pelajaran, jadwal pelajaran dan jadwal piket.

- Penggantian piket per tanggal menimpa jadwal tetap pada tanggal tersebut saja.

- Sistem menolak jadwal yang membuat satu guru mengajar dua kelas pada jam yang sama.

4.5 Presensi dan jurnal

FR-09 Pembentukan sesi pembelajaran (P0)

Jadwal pelajaran dijabarkan menjadi sesi per tanggal. Presensi dan jurnal menempel pada sesi.

**Kriteria penerimaan:**

- Sesi dibuat otomatis (terjadwal harian) untuk hari efektif; hari libur dapat ditandai sehingga tidak membentuk sesi.

- Guru pengganti dapat ditugaskan per sesi oleh TU atau Waka Kurikulum; guru aktual tercatat terpisah dari guru terjadwal.

FR-10 Pengisian presensi per jam (P0)

Guru memilih sesi pada hari ini dan mengisi status setiap siswa dalam satu layar.

**Kriteria penerimaan:**

- Status: hadir, terlambat, sakit, izin, izin tidak mengikuti pelajaran, alpa. Sakit, izin dan izin tidak mengikuti pelajaran dihitung tidak hadir **berizin**; alpa terpisah.

- Bawaan semua siswa **hadir**; guru hanya mengubah yang berbeda. Siswa dengan izin yang disetujui sudah terisi otomatis dan ditandai.

- Satu sesi menyimpan satu set presensi; waktu pengisian (diisi_pada) tercatat untuk supervisi.

- Guru hanya dapat mengisi pada hari sesi (batas H+1 dapat diatur). Setelah itu hanya wali kelas yang dapat mengoreksi.

- Halaman termuat dan tersimpan cepat untuk kelas 36 siswa di ponsel 4G.

FR-11 Jurnal mengajar (P0)

Pada sesi yang sama guru mengisi jurnal: materi, kegiatan, kemajuan kelas dan catatan.

**Kriteria penerimaan:**

- Materi wajib diisi sebelum sesi ditandai selesai; kolom lain opsional.

- Kepala Sekolah dan Waka Kurikulum dapat melihat jurnal semua guru; wali kelas dapat melihat jurnal kelasnya.

FR-12 Koreksi oleh wali kelas dan audit (P0)

Wali kelas mengoreksi presensi kelasnya, termasuk setelah batas guru.

**Kriteria penerimaan:**

- Setiap koreksi menyimpan nilai lama, nilai baru, pelaku dan alasan di audit log.

- Rekap harian wali kelas menggabungkan semua jam dalam satu tampilan per siswa.

4.6 Perizinan

FR-13 Izin harian (sakit, izin, dispensasi) (P0)

Orang tua mengajukan melalui PWA.

**Kriteria penerimaan:**

- Formulir: jenis, tanggal mulai-selesai, alasan, lampiran foto (ukuran dan tipe dibatasi).

- Sakit dan izin disetujui **wali kelas**; dispensasi sekolah disetujui **Waka Kesiswaan**.

- Status pengajuan: diajukan, disetujui, ditolak, dibatalkan; penolakan wajib disertai catatan.

- Setelah disetujui, semua sesi pada rentang tanggal terisi status izin saat guru membuka presensi.

- Izin susulan boleh diajukan hingga H+3 (dapat diatur).

FR-14 Izin per jam (tidak mengikuti pelajaran) (P0)

Siswa mengajukan melalui PWA untuk satu atau beberapa jam pada suatu tanggal.

**Kriteria penerimaan:**

- Formulir: tanggal, jam pelajaran yang dipilih (dari jadwal kelasnya), alasan.

- Persetujuan wajib dari **guru pengajar** pada tiap jam yang diajukan **dan** dari **guru piket** pada tanggal itu (satu persetujuan piket mencakup semua jam di pengajuan).

- Status jadi disetujui hanya jika seluruh persetujuan yang disyaratkan lengkap; satu penolakan membatalkan jam terkait.

- Setelah disetujui, sesi terkait terisi status "izin tidak mengikuti pelajaran".

- Guru melihat antrean pengajuan di dashboard masing-masing; guru piket melihat semua pengajuan hari itu.

4.7 Notifikasi WhatsApp (Fonnte)

FR-15 Pengiriman notifikasi (P0)

Semua pesan masuk antrean dan dikirim oleh worker melalui API Fonnte.

**Kriteria penerimaan:**

- Kejadian (masing-masing dapat diaktifkan/dinonaktifkan): siswa alpa; izin harian disetujui/ditolak; izin per jam disetujui/ditolak; pengajuan izin baru ke penyetuju; pengumuman penting.

- Antrean memakai tabel notifikasi_wa dengan status menunggu, terkirim, gagal, dan menyimpan respons Fonnte.

- Jeda acak antar pesan (rentang dapat diatur) dan jam kirim yang diizinkan (mis. 06.00–20.00); di luar jam itu pesan menunggu.

- Pesan gagal dicoba ulang otomatis terbatas, lalu ditandai gagal; admin dapat mengirim ulang manual.

- Panggilan: POST https://api.fonnte.com/send dengan header Authorization berisi token, parameter target, message dan countryCode. Detail parameter harus dicek ke dokumentasi Fonnte terkini saat implementasi.

FR-16 Nomor WhatsApp orang tua dan verifikasi (P0)

Data Dapodik tidak memuat nomor HP orang tua, sehingga nomor dikumpulkan di aplikasi.

**Kriteria penerimaan:**

- Saat login pertama, orang tua wajib mengisi nomor WhatsApp, lalu memverifikasinya dengan kode yang dikirim melalui Fonnte.

- Admin dapat mengisi atau mengimpor nomor orang tua lewat Excel dan menandainya sebagai terverifikasi.

- Nomor dinormalisasi ke format internasional (62…) sebelum disimpan dan dikirim.

- Notifikasi ke orang tua hanya dikirim ke nomor terverifikasi.

FR-17 Halaman pengaturan Fonnte (P0)

Halaman khusus Super Admin untuk mengelola integrasi Fonnte. Token dan nomor pengirim diisi nanti oleh pemilik; aplikasi harus berjalan tanpa token (pesan menunggu atau tertahan dengan peringatan).

**Kriteria penerimaan:**

- Menyimpan **token** terenkripsi (cast terenkripsi Laravel); tidak pernah ditampilkan penuh kembali (hanya penanda tersimpan dan 4 karakter terakhir).

- Pengaturan: aktif/nonaktif global dan per kejadian, kode negara bawaan (62), rentang jeda antar pesan, jam kirim yang diizinkan.

- Tombol **Cek status perangkat** (terhubung/terputus) memanggil API Fonnte dan menampilkan hasil.

- Tombol **Kirim pesan uji** ke nomor yang dimasukkan.

- **Templat pesan** per kejadian dengan placeholder {nama_siswa}, {kelas}, {tanggal}, {jam}, {mapel}, {status}, {alasan} dan pratinjau.

- **Log pengiriman** dengan filter status dan tombol kirim ulang untuk yang gagal.

- Seluruh perubahan pengaturan tercatat di audit log.

4.8 Dashboard, supervisi dan BK

FR-18 Dashboard kehadiran (P0)

Ringkasan kehadiran siswa untuk Kepala Sekolah, Waka, wali kelas dan Guru BK sesuai cakupannya.

**Kriteria penerimaan:**

- Persentase kehadiran per kelas dan angkatan untuk hari ini, minggu ini, bulan ini, dengan filter rentang tanggal dan tahun ajaran.

- Daftar siswa dengan alpa berulang (ambang dapat diatur) dan tren kehadiran.

- Ekspor rekap ke Excel dan PDF.

- Kueri agregat memakai indeks yang tepat dan tetap responsif pada ±1,3 juta baris presensi.

FR-19 Supervisi kehadiran guru mengajar (P0)

Untuk Kepala Sekolah dan Waka Kurikulum.

**Kriteria penerimaan:**

- Tabel jadwal terhadap sesi terisi dengan kategori: terisi tepat waktu, terisi terlambat, belum diisi, guru pengganti.

- Dapat disaring per guru, kelas dan tanggal; dapat dibuka ke jurnal terkait.

- Daftar sesi belum diisi bagi TU/Waka untuk ditindaklanjuti.

FR-20 Modul Guru BK (P1)

Guru BK melihat dan mencatat bimbingan untuk siswa di kelas yang ditugaskan.

**Kriteria penerimaan:**

- Daftar siswa dengan pola ketidakhadiran bermasalah di kelas yang ditugaskan.

- Catatan BK (jenis, isi, tindak lanjut) hanya terbaca oleh Guru BK yang bersangkutan dan Kepala Sekolah.

- Penugasan BK per kelas per tahun ajaran dikelola admin.

4.9 PWA dan pengumuman

FR-21 PWA untuk siswa dan orang tua (P0)

Aplikasi dapat dipasang di layar utama dan nyaman di ponsel.

**Kriteria penerimaan:**

- Manifest, ikon, splash screen dan service worker aktif di HTTPS; skor Lighthouse PWA terpenuhi.

- Mode offline berupa cache tampilan dan data terakhir (hanya baca); aksi tulis membutuhkan koneksi dan menampilkan pesan jelas jika offline.

- Tampilan mobile-first dan dapat dipakai dengan satu tangan untuk tugas utama (lihat presensi, ajukan izin).

FR-22 Pengumuman (P1)

Admin atau Waka mengirim pengumuman ke kelompok (semua, kelas, angkatan).

**Kriteria penerimaan:**

- Tampil di PWA; pengiriman WhatsApp bersifat opsional per pengumuman.

5\. Model data

Tabel dan kolom memakai bahasa Indonesia snake_case (konsisten dengan istilah domain). Semua tabel memiliki id, created_at, updated_at; data induk memakai soft delete. Zona waktu aplikasi Asia/Jakarta, locale id.

| **Tabel**                      | **Kolom kunci dan catatan**                                                                                                      |
|--------------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| users                          | id, name, username (unik), email nullable, password, must_change_password, tipe (guru/karyawan/siswa/orang_tua), terakhir_login. |
| sekolah                        | nama, npsn, alamat, kontak.                                                                                                      |
| tahun_ajaran / semester        | nama, tanggal_mulai, tanggal_selesai, aktif (hanya satu aktif).                                                                  |
| guru                           | user_id, nama, nik (unik), nip, nuptk, jk, tempat/tanggal_lahir, status_kepegawaian, tugas_tambahan, hp, email, kode_asc.        |
| karyawan                       | user_id nullable, nama, jabatan, kontak.                                                                                         |
| siswa                          | user_id, nama, nisn (unik, teks), nipd/nis (teks), nik, jk, tempat/tanggal_lahir, alamat ringkas, hp, sekolah_asal, aktif.       |
| orang_tua                      | user_id, siswa_id, hubungan (ayah/ibu/wali), nama, pekerjaan, no_wa, no_wa_terverifikasi_pada.                                   |
| kelas                          | tahun_ajaran_id, tingkat, nama (mis. X E-1), kode_asc.                                                                           |
| rombel                         | kelas_id, siswa_id, tanggal_mulai, tanggal_selesai (riwayat penempatan).                                                         |
| wali_kelas / penugasan_bk      | guru_id, kelas_id, tahun_ajaran_id.                                                                                              |
| bidang_waka / penugasan_waka   | bidang_waka(nama); penugasan_waka(guru_id, bidang_id, periode).                                                                  |
| jadwal_piket / piket_pengganti | jadwal_piket(guru_id, hari); piket_pengganti(guru_id, tanggal) menimpa jadwal tetap.                                             |
| mata_pelajaran                 | kode, nama, kode_asc.                                                                                                            |
| jam_pelajaran                  | urutan (jam ke-), jam_mulai, jam_selesai, jenis (pelajaran/istirahat). Global.                                                   |
| jadwal_pelajaran               | kelas_id, mata_pelajaran_id, guru_id, hari, jam_pelajaran_id; unik per guru+hari+jam.                                            |
| sesi_pembelajaran              | jadwal_pelajaran_id, tanggal, guru_id_aktual, status (belum/terisi/selesai), diisi_pada, ditutup_pada.                           |
| presensi                       | sesi_id, siswa_id, status (enum), keterangan, dari_izin_id nullable, diubah_oleh; unik (sesi_id, siswa_id).                      |
| jurnal_mengajar                | sesi_id (unik), materi, kegiatan, kemajuan_kelas, catatan.                                                                       |
| pengajuan_izin                 | siswa_id, jenis (sakit/izin/dispensasi/per_jam), tanggal_mulai, tanggal_selesai, alasan, lampiran, status, diajukan_oleh.        |
| pengajuan_izin_jam             | pengajuan_izin_id, sesi_id atau (tanggal, jam_pelajaran_id), guru_pengajar_id, status.                                           |
| persetujuan_izin               | pengajuan_izin_id, peran_penyetuju (wali_kelas/waka_kesiswaan/guru_pengajar/guru_piket), user_id, keputusan, catatan, waktu.     |
| catatan_bk                     | siswa_id, guru_bk_id, jenis, isi, tindak_lanjut, tanggal.                                                                        |
| pengumuman                     | judul, isi, sasaran, kirim_wa, dibuat_oleh.                                                                                      |
| pengaturan                     | kunci (unik), nilai (terenkripsi untuk rahasia). Menyimpan konfigurasi Fonnte.                                                   |
| templat_wa                     | kejadian, isi, aktif.                                                                                                            |
| notifikasi_wa                  | kejadian, target (nomor), pesan, status, respons, percobaan, dijadwalkan_pada, terkirim_pada, relasi opsional (siswa/pengajuan). |
| audit_log                      | user_id, aksi, objek (tipe+id), nilai_lama, nilai_baru, alasan, ip, waktu.                                                       |

**Indeks wajib:** presensi(sesi_id), presensi(siswa_id, sesi_id), sesi_pembelajaran(tanggal, jadwal_pelajaran_id), sesi_pembelajaran(guru_id_aktual, tanggal), rombel(kelas_id, siswa_id), notifikasi_wa(status, dijadwalkan_pada). Volume presensi ±7.000 baris per hari.

6\. Matriks role dan permission

Matriks diimplementasikan sebagai seeder permission agar dapat diubah tanpa mengubah kode. Tanda ✓ berarti diizinkan; "kelas" berarti hanya untuk kelas yang ditugaskan; "sesi" hanya untuk sesi yang diajarnya; "Kur" dan "Kes" berarti hanya Waka bidang Kurikulum atau Kesiswaan; "diri" dan "anak" berarti data sendiri atau anaknya.

| **Permission**                    | **SA** | **TU** | **KS** | **Waka** | **Wali** | **Guru** | **Piket** | **BK** | **Siswa** | **Ortu** |
|-----------------------------------|--------|--------|--------|----------|----------|----------|-----------|--------|-----------|----------|
| Kelola akun dan reset password    | ✓      | ✓      |        |          |          |          |           |        |           |          |
| Impor Dapodik / kelola data induk | ✓      | ✓      |        |          |          |          |           |        |           |          |
| Kelola jadwal dan piket           | ✓      | ✓      |        |          |          |          |           |        |           |          |
| Lihat data induk semua            | ✓      | ✓      | ✓      | ✓        |          |          |           |        |           |          |
| Isi presensi dan jurnal           |        |        |        |          |          | sesi     |           |        |           |          |
| Koreksi presensi                  | ✓      |        |        |          | kelas    |          |           |        |           |          |
| Lihat presensi semua              | ✓      | ✓      | ✓      | ✓        |          |          |           |        |           |          |
| Lihat presensi kelas / anak       |        |        |        |          | kelas    | sesi     |           | kelas  | diri      | anak     |
| Lihat jurnal semua (supervisi)    | ✓      |        | ✓      | Kur      | kelas    |          |           |        |           |          |
| Ajukan izin harian                |        |        |        |          |          |          |           |        |           | ✓        |
| Ajukan izin per jam               |        |        |        |          |          |          |           |        | ✓         |          |
| Setujui izin harian               |        |        |        |          | kelas    |          |           |        |           |          |
| Setujui dispensasi sekolah        |        |        |        | Kes      |          |          |           |        |           |          |
| Setujui izin per jam              |        |        |        |          |          | sesi     | ✓         |        |           |          |
| Catatan BK                        |        |        | ✓      |          |          |          |           | kelas  |           |          |
| Pengaturan Fonnte dan templat WA  | ✓      |        |        |          |          |          |           |        |           |          |
| Dashboard seluruh sekolah         | ✓      | ✓      | ✓      | ✓        |          |          |           |        |           |          |
| Audit log                         | ✓      |        | ✓      |          |          |          |           |        |           |          |

Singkatan: SA Super Admin, TU Admin/TU, KS Kepala Sekolah, Waka Wakil Kepala Sekolah, Wali Wali Kelas, BK Guru BK, Ortu Orang Tua.

7\. Persyaratan non-fungsional

| **Aspek**                | **Persyaratan**                                                                                                                                                                                             |
|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Performa                 | Halaman presensi ≤ 3 detik pada 4G; respons API umum ≤ 500 ms (p95) pada beban 100 pengguna aktif bersamaan; lonjakan jam masuk (±800 siswa dan guru dalam 10 menit) tidak membuat server kehabisan memori. |
| Sumber daya              | Berjalan di VPS 2 core / 2 GB: PHP-FPM pm=ondemand, PostgreSQL shared_buffers ±256 MB, satu worker antrean, swap 1–2 GB, antrean di database (tanpa Redis).                                                 |
| Keamanan                 | HTTPS wajib; password di-hash; rate limiting login; CSRF; otorisasi Policy di semua endpoint; unggahan dibatasi tipe/ukuran dan disimpan di luar webroot; header keamanan; dependensi diperbarui.           |
| Privasi (UU PDP)         | Minimisasi data: hanya kolom yang dibutuhkan yang diimpor (Lampiran A). Catatan BK dan data pribadi dibatasi aksesnya. Tidak ada data nyata di Git, log atau data uji.                                      |
| Ketersediaan dan backup  | pg_dump harian terenkripsi dan disalin ke luar server, retensi minimal 14 hari, uji restore tiap bulan. Target pemulihan ≤ 4 jam.                                                                           |
| Observabilitas           | Log aplikasi terpusat, pelacakan error (Sentry atau log Laravel), pemantauan uptime, dan peringatan jika antrean atau perangkat Fonnte bermasalah.                                                          |
| Kompatibilitas           | Chrome/Edge/Firefox dan Safari terkini; Android 8+ dan iOS 16.4+ (web push iOS hanya setelah dipasang ke layar utama).                                                                                      |
| Aksesibilitas dan bahasa | Antarmuka berbahasa Indonesia; kontras dan ukuran sentuh sesuai praktik ponsel; format tanggal dd-mm-yyyy.                                                                                                  |
| Kualitas kode            | Pest/PHPUnit untuk aturan bisnis; Laravel Pint; analisis statis (Larastan) pada CI; migration dan seeder dapat dijalankan dari nol.                                                                         |

8\. Arsitektur dan konvensi teknis

8.1 Keputusan arsitektur

- Monolit Laravel. Livewire untuk antarmuka interaktif; Filament untuk panel admin (data induk, impor, pengaturan, log); halaman guru/siswa/orang tua memakai Blade + Livewire kustom yang ringan.

- Logika bisnis di kelas Action/Service (mis. ImporSiswaAction, BukaSesiAction, SetujuiIzinAction), bukan di controller atau komponen.

- Otorisasi dengan Policy dan permission Spatie; pengecekan penugasan (wali kelas, BK, piket) di Policy.

- Antrean berbasis database untuk impor, notifikasi WhatsApp dan pembentukan sesi; scheduler Laravel (cron) untuk pembentukan sesi harian dan kirim ulang pesan gagal.

- Rahasia (token Fonnte) disimpan dengan enkripsi aplikasi (APP_KEY); .env tidak masuk Git.

8.2 Struktur proyek (usulan)

> app/Actions \# logika bisnis per kasus penggunaan
>
> app/Models \# Eloquent: Siswa, Guru, Kelas, SesiPembelajaran, Presensi, ...
>
> app/Policies \# otorisasi berbasis penugasan
>
> app/Imports \# impor Excel (Dapodik, template guru, jadwal CSV)
>
> app/Services/Fonnte \# klien API Fonnte + antrean notifikasi
>
> app/Filament \# resource panel admin dan halaman pengaturan Fonnte
>
> app/Livewire \# komponen presensi, izin, dashboard
>
> database/migrations, database/seeders (role, permission, jam pelajaran)
>
> tests/Feature, tests/Unit
>
> docs/ \# PRD, rancangan, panduan operasional

8.3 Konvensi

- Penamaan domain berbahasa Indonesia (Siswa, Guru, Presensi, SesiPembelajaran); istilah teknis tetap bahasa Inggris.

- Semua perubahan skema lewat migration; seeder idempoten. Kunci identitas (NISN, NIK, NIPD) bertipe string.

- Commit kecil dengan pesan jelas; satu fase = satu cabang kerja. Jangan memasukkan file Excel Dapodik atau data pribadi ke repositori.

- Data uji memakai factory/seeder dengan data fiktif, bukan data siswa dan guru yang sebenarnya.

9\. Roadmap dan kriteria selesai

Estimasi untuk 2 orang. Fase 1–4 menghasilkan MVP presensi yang dapat diujicobakan.

| **Fase** | **Cakupan (FR)**                                                                                                    | **Kriteria selesai**                                                                               | **Estimasi** |
|----------|---------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|--------------|
| 0        | Setup proyek Laravel + PostgreSQL, Filament, Livewire, Tailwind, CI, deployment VPS (Nginx, PHP-FPM, HTTPS, backup) | Aplikasi kosong berjalan di staging dengan HTTPS dan pipeline CI hijau                             | 3–4 hari     |
| 1        | FR-01, FR-02, FR-03, kerangka PWA                                                                                   | Login berbagai tipe akun, role/permission/penugasan, audit log, tes otorisasi lulus                | 1 minggu     |
| 2        | FR-04, FR-05, FR-06                                                                                                 | Impor kedua file Dapodik sukses dan idempoten; 24 kelas terbentuk; template guru manual berfungsi  | 1,5 minggu   |
| 3        | FR-07, FR-08                                                                                                        | Jam, jadwal, piket dapat dikelola; deteksi bentrok guru; impor CSV jadwal                          | 1 minggu     |
| 4        | FR-09, FR-10, FR-11, FR-12                                                                                          | Guru mengisi presensi + jurnal; wali kelas mengoreksi; audit tercatat                              | 1,5 minggu   |
| 5        | FR-13, FR-14                                                                                                        | Alur izin harian dan per jam lengkap dengan persetujuan berjenjang dan pengisian otomatis presensi | 1 minggu     |
| 6        | FR-15, FR-16, FR-17                                                                                                 | Notifikasi via Fonnte, halaman pengaturan, verifikasi nomor orang tua, log dan kirim ulang         | 1 minggu     |
| 7        | FR-18, FR-19, FR-20, FR-21, FR-22                                                                                   | Dashboard, supervisi guru, modul BK, PWA lengkap, ekspor laporan                                   | 1–1,5 minggu |
| 8        | Uji coba terbatas, pelatihan, perbaikan, rilis bertahap                                                             | Uji coba 2–3 kelas selama 2 minggu tanpa isu kritis; panduan pengguna                              | 1–2 minggu   |

9.1 Definition of Done (berlaku untuk setiap fase)

- Seluruh kriteria penerimaan FR pada fase tersebut terpenuhi dan diuji (tes otomatis untuk aturan bisnis dan otorisasi).

- Migration dan seeder berjalan dari database kosong; tidak ada data pribadi di repositori.

- Pint dan analisis statis lulus; tidak ada peringatan keamanan dependensi tingkat tinggi.

- Dokumentasi singkat fase (apa yang dibangun, cara menjalankan, keputusan penting) ditambahkan di docs/.

10\. Risiko dan mitigasi

| **Risiko**                                                                           | **Dampak**              | **Mitigasi**                                                                                                                                                     |
|--------------------------------------------------------------------------------------|-------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Nomor pengirim WhatsApp dibatasi/diblokir karena Fonnte memakai WhatsApp tidak resmi | Notifikasi berhenti     | Nomor khusus sekolah, jeda dan jam kirim terkontrol, hindari pesan massal sekaligus, pantau status perangkat, siapkan notifikasi dalam aplikasi sebagai cadangan |
| Nomor WhatsApp orang tua tidak tersedia di Dapodik                                   | Notifikasi tidak sampai | Pengisian wajib saat login pertama dengan verifikasi kode; isi manual/impor oleh admin                                                                           |
| NIS mudah ditebak (nomor urut)                                                       | Akses akun tidak sah    | Password awal acak wajib diganti, rate limiting, reset oleh admin                                                                                                |
| Lonjakan beban jam masuk di VPS 2 GB                                                 | Lambat atau gagal       | Endpoint presensi ringan, indeks, worker tunggal, swap, uji beban sebelum rilis                                                                                  |
| Format ekspor Dapodik berubah                                                        | Impor gagal             | Pemetaan kolom berdasarkan nama header, laporan error jelas, tes dengan file contoh                                                                              |
| Data pribadi bocor lewat Git atau log                                                | Pelanggaran privasi     | .gitignore untuk \*.xlsx dan .env, data uji fiktif, log tanpa data sensitif                                                                                      |
| Guru tidak disiplin mengisi presensi                                                 | Data tidak lengkap      | Dashboard supervisi, pelatihan, pengingat, dan dukungan pimpinan                                                                                                 |
| Ketergantungan 2 orang pengembang                                                    | Bus factor rendah       | Dokumentasi di repositori, konvensi jelas, tes otomatis                                                                                                          |

11\. Hal yang ditunda atau perlu diisi nanti

- **File ekspor aSc Timetables** (XML/Excel): menentukan pemetaan impor jadwal (FR-07). Dikerjakan setelah file tersedia.

- **Token dan nomor pengirim Fonnte:** diisi pemilik lewat halaman pengaturan (FR-17); tidak disimpan di kode atau repositori.

- **Nilai bawaan yang dapat diubah** setelah uji coba: batas pengisian guru (H+1), batas izin susulan (H+3), ambang alpa berulang, jam kirim WhatsApp.

- **Bidang Waka tambahan** selain Kurikulum, Kesiswaan dan Sarpras: ditambah lewat data.

- **Pemeliharaan setelah rilis:** penanggung jawab server dan aplikasi belum ditetapkan.

Lampiran A. Pemetaan kolom Dapodik

Berdasarkan struktur file ekspor yang sebenarnya (SMAN 6 Yogyakarta, tanggal unduh 24 November 2026). Pencocokan kolom memakai **nama header**, bukan posisi.

A.1 Daftar guru (52 kolom, header di baris ke-5)

| **Kolom Dapodik**                               | **Disimpan di**                      | **Catatan**                                         |
|-------------------------------------------------|--------------------------------------|-----------------------------------------------------|
| Nama                                            | guru.nama                            |                                                     |
| NIK                                             | guru.nik                             | Kunci upsert; teks                                  |
| NIP, NUPTK                                      | guru.nip, guru.nuptk                 | Boleh kosong; teks                                  |
| JK, Tempat Lahir, Tanggal Lahir                 | guru.jk, tempat_lahir, tanggal_lahir |                                                     |
| Status Kepegawaian, Jenis PTK                   | guru.status_kepegawaian, jenis_ptk   | PNS, PPPK, Guru Honor Sekolah, Tenaga Honor Sekolah |
| Tugas Tambahan                                  | guru.tugas_tambahan                  | Dasar usulan role wali kelas/waka                   |
| Alamat Jalan, RT, RW, Desa/Kelurahan, Kecamatan | guru.alamat (ringkas)                | Opsional                                            |
| HP, Email                                       | guru.hp, guru.email                  | Email untuk login                                   |

**Tidak diimpor:** NPWP, Nama Wajib Pajak, Bank, Nomor Rekening, Rekening Atas Nama, No KK, Karpeg, Karis/Karsu, SK dan TMT pengangkatan, data pasangan, nama ibu kandung, lintang/bujur, dan kolom kepegawaian lain yang tidak dibutuhkan aplikasi.

A.2 Daftar peserta didik (66 kolom, header dua tingkat baris ke-5 dan 6)

| **Kolom Dapodik**                           | **Disimpan di**                     | **Catatan**                                  |
|---------------------------------------------|-------------------------------------|----------------------------------------------|
| Nama                                        | siswa.nama                          |                                              |
| NISN                                        | siswa.nisn                          | Kunci upsert; wajib teks 10 digit            |
| NIPD                                        | siswa.nis                           | Username orang tua; 1 siswa kosong, ditandai |
| NIK                                         | siswa.nik                           | Teks                                         |
| JK, Tempat Lahir, Tanggal Lahir, Agama      | siswa.\*                            |                                              |
| Alamat, RT, RW, Dusun, Kelurahan, Kecamatan | siswa.alamat (ringkas)              | Opsional                                     |
| HP                                          | siswa.hp                            | 46 kosong; bukan nomor orang tua             |
| Data Ayah / Ibu / Wali: Nama, Pekerjaan     | orang_tua.nama, pekerjaan, hubungan | Ayah kosong pada 11 siswa; wali pada 833     |
| Rombel Saat Ini                             | kelas.nama + rombel                 | 24 rombel (X E-1…8, XI F-1…8, XII F-1…8)     |
| Sekolah Asal                                | siswa.sekolah_asal                  | Opsional                                     |

**Tidak diimpor:** penghasilan orang tua, NIK orang tua, KPS/KIP/KKS/PIP, nomor rekening, No KK, nomor ijazah dan ujian, akta lahir, data fisik (berat/tinggi/lingkar kepala), kebutuhan khusus, lintang/bujur, jarak rumah. Bila suatu hari dibutuhkan, tambahkan secara sadar dengan persetujuan pemilik produk.

A.3 Statistik file contoh

| **Hal**                  | **Nilai**                                                                                                                                             |
|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| Guru dalam file          | 38 PTK (25 PNS, 9 PPPK, 3 Guru Honor, 1 Tenaga Honor); 18 guru lain diinput manual                                                                    |
| Tugas tambahan           | Wali kelas 18, guru wali 14, Waka Kurikulum 1, Waka Kesiswaan 1, Waka Sarpras 1, lainnya 3; 6 dari 24 kelas belum punya wali kelas; Guru BK tidak ada |
| Siswa dalam file         | 860 (381 L, 479 P) dalam 24 rombel, isi 35–37 per rombel                                                                                              |
| Duplikasi dan kekosongan | NISN, NIK siswa dan NIK guru tanpa duplikat; NIPD kosong 1; NIP kosong 4; NUPTK kosong 1                                                              |

Lampiran B. Panduan eksekusi di Claude Desktop

B.1 Persiapan sekali di awal

1.  Selesaikan akses GitHub agar Claude dapat melakukan push ke repositori namche_app (hubungkan GitHub di claude.ai dan pasang Claude GitHub App pada repositori).

2.  Buka repositori namche_app di Claude Desktop (Claude Code) dan pastikan Claude membaca dokumen ini serta docs/RANCANGAN.md.

3.  Simpan file Excel Dapodik **di luar repositori** (mis. folder terpisah); jangan di-commit. Berikan jalurnya ke Claude hanya saat mengerjakan Fase 2.

4.  Siapkan VPS (Ubuntu LTS), domain/subdomain, dan akses SSH untuk Fase 0. Token Fonnte tidak diperlukan sampai Fase 6.

B.2 Aturan kerja untuk Claude

- Kerjakan **satu fase per sesi/cabang**. Sebelum mengubah apa pun, baca PRD ini dan ringkas rencana fase itu; tunggu persetujuan untuk keputusan di luar PRD.

- Tulis tes untuk aturan bisnis dan otorisasi pada tiap FR; fase dianggap selesai hanya jika kriteria penerimaan dan Definition of Done terpenuhi.

- Jangan pernah memasukkan data pribadi, file .xlsx atau rahasia (token, APP_KEY) ke Git. Gunakan data fiktif untuk uji.

- Jika sesuatu di PRD ambigu atau bertentangan dengan kenyataan data, hentikan dan tanyakan, jangan menebak.

- Di akhir fase: jalankan seluruh tes, perbarui docs/, buat commit dengan pesan jelas, dan laporkan apa yang selesai, apa yang tidak, dan keputusan yang perlu dipastikan.

B.3 Perintah per fase (salin ke Claude)

| **Fase** | **Perintah**                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Fase 0   | Baca docs/PRD (dokumen ini) dan docs/RANCANGAN.md. Kerjakan Fase 0: buat proyek Laravel (versi stabil terbaru) dengan PostgreSQL, Livewire, Tailwind, Filament, Pest, Pint dan Larastan; konfigurasi zona waktu Asia/Jakarta dan locale id; tambah GitHub Actions untuk tes dan lint; siapkan dokumen deployment ke VPS 2 GB (Nginx, PHP-FPM ondemand, PostgreSQL tuning, swap, supervisor untuk queue worker, cron scheduler, backup pg_dump). Tunjukkan rencana dulu. |
| Fase 1   | Kerjakan Fase 1: FR-01, FR-02, FR-03. Buat tabel users dengan tipe akun dan must_change_password, login multi-username (NIS/NISN/email/NIP), rate limiting, spatie/laravel-permission dengan seeder role dan permission sesuai Bagian 6, tabel penugasan (wali kelas, BK, waka, piket), audit log, dan kerangka PWA. Sertakan tes otorisasi.                                                                                                                            |
| Fase 2   | Kerjakan Fase 2: FR-04, FR-05, FR-06. Bangun CRUD guru, karyawan, siswa, orang tua, kelas, rombel di Filament. Buat importer Dapodik sesuai Lampiran A (header dua tingkat, upsert NISN/NIK, pratinjau, laporan error, job antrean hemat memori). File contoh ada di \[JALUR FILE\]; jangan salin ke repositori. Buat juga template Excel guru untuk input manual 18 guru.                                                                                              |
| Fase 3   | Kerjakan Fase 3: FR-07, FR-08. Jam pelajaran global, jadwal pelajaran dengan deteksi bentrok guru, jadwal piket tetap dengan penggantian per tanggal, dan impor CSV jadwal. Siapkan kolom kode_asc tetapi jangan membuat importer aSc.                                                                                                                                                                                                                                  |
| Fase 4   | Kerjakan Fase 4: FR-09 sampai FR-12. Pembentukan sesi harian (scheduler), halaman presensi dan jurnal untuk guru (mobile-first, bawaan hadir), batas waktu pengisian, guru pengganti, koreksi wali kelas dengan audit log. Sertakan uji beban sederhana untuk halaman presensi.                                                                                                                                                                                         |
| Fase 5   | Kerjakan Fase 5: FR-13 dan FR-14. Izin harian oleh orang tua (wali kelas/Waka Kesiswaan menyetujui) dan izin per jam oleh siswa (guru pengajar dan guru piket wajib menyetujui), pengisian otomatis presensi, dan antrean persetujuan.                                                                                                                                                                                                                                  |
| Fase 6   | Kerjakan Fase 6: FR-15, FR-16, FR-17. Klien API Fonnte, antrean notifikasi dengan jeda dan jam kirim, templat pesan, verifikasi nomor WhatsApp orang tua, dan halaman pengaturan Fonnte di Filament (token terenkripsi, cek status perangkat, pesan uji, log dan kirim ulang). Aplikasi harus tetap berjalan tanpa token. Cek dokumentasi Fonnte terkini sebelum menulis klien.                                                                                         |
| Fase 7   | Kerjakan Fase 7: FR-18 sampai FR-22. Dashboard kehadiran, supervisi guru mengajar, modul Guru BK, PWA lengkap (manifest, service worker, cache baca), pengumuman, ekspor Excel/PDF.                                                                                                                                                                                                                                                                                     |
| Fase 8   | Kerjakan Fase 8: audit keamanan (gunakan security review), uji beban jam masuk, panduan pengguna per role, panduan operasional (backup/restore, rotasi token), dan rencana uji coba 2–3 kelas.                                                                                                                                                                                                                                                                          |

*Dokumen ini adalah dokumen hidup. Perubahan keputusan dicatat di Bagian 3 beserta tanggalnya.*
