# WargaHub P04 — UX, RecyclerView, Fragment, dan Navigasi

Proyek aplikasi Android native berbasis **Kotlin** dan **XML Views (ViewBinding)** untuk memenuhi tugas **Praktikum 04 Pemrograman Mobile (PTI23043)** di Program Studi Teknologi Informasi, UIN Antasari Banjarmasin. Aplikasi ini mensimulasikan portal warga (**WargaHub**) dengan alur tiga layar (*Beranda*, *Daftar Laporan*, dan *Rincian Laporan*) yang menampilkan 15 laporan masalah lingkungan fiktif menggunakan `RecyclerView`, `ListAdapter`, `DiffUtil`, dan `AndroidX Navigation Component`.

## 1. Identitas Mahasiswa & Praktikum

| Komponen | Keterangan |
| :--- | :--- |
| **Nama Mahasiswa** | Muhammad Zaini Arsyad |
| **NIM** | 240104040057 |
| **Kelas** | TI24B |
| **Mata Kuliah** | Praktikum Pemrograman Mobile (PTI23043) — 1 SKS |
| **Semester / Pertemuan** | Semester V / Pertemuan 4 (Praktikum 04) |
| **Dosen Pengampu** | Muhayat, M.IT |
| **Nama Proyek & Package** | `WargaHubP04` — `id.antasari.p04.wargahub` |
| **Rekan Penguji Usability (Tahap 6.J)** | Aby Wahyudi (`240104040190`) |

## 2. Spesifikasi Lingkungan & Alat Pengembangan

| Parameter Teknis | Konfigurasi Proyek & Perangkat Uji |
| :--- | :--- |
| **IDE & JDK** | Android Studio dengan Embedded JDK (JetBrains Runtime / JBR 21) |
| **Android Gradle Plugin (AGP)** | `9.2.1` (Kotlin bawaan AGP) |
| **Gradle Wrapper** | `9.4.1` (`gradle/wrapper/gradle-wrapper.properties`) |
| **Konfigurasi SDK** | `compileSdk = 37`, `targetSdk = 36`, `minSdk = 26` (Android 8.0+), Java Target `VERSION_11` |
| **Pendekatan Antarmuka** | Native XML Views + `viewBinding = true` (Tanpa Jetpack Compose & tanpa plugin Safe Args) |
| **Perangkat Uji Utama** | Emulator **Pixel 7** — Android 17 (API 37), Versi Aplikasi `1.0` (`versionCode = 1`) |
| **Kondisi Uji Tambahan** | Orientasi *Portrait* & *Landscape*, Skala Font `1.0x`, `1.3x`, dan `1.5x`, Simulasi Layar Kecil (`360x640 dp`), serta Layanan Aksesibilitas **TalkBack** |

### Daftar Dependensi (`app/build.gradle.kts`)
- `androidx.core:core-ktx:1.19.1` — Ekstensi Kotlin untuk API inti Android (`bundleOf`, `isVisible`, `WindowInsetsCompat`).
- `androidx.appcompat:appcompat:1.8.0` — Kompatibilitas `AppCompatActivity`, `Toolbar`, dan tema `Theme.AppCompat.Light.NoActionBar`.
- `androidx.activity:activity-ktx:1.13.0` — Dukungan komponen Activity dan `enableEdgeToEdge()`.
- `androidx.recyclerview:recyclerview:1.4.0` — Komponen daftar efisien berbasis daur ulang View (`RecyclerView`, `ListAdapter`, `DiffUtil`, `StateRestorationPolicy`).
- `androidx.fragment:fragment-ktx:1.8.9` — Pengelolaan `Fragment` modern dengan konstruktor layout `Fragment(R.layout...)` dan `FragmentContainerView`.
- `androidx.navigation:navigation-fragment-ktx:2.9.5` — Pengelolaan `NavHostFragment`, `NavController`, dan grafik navigasi XML (`nav_graph.xml`).
- `androidx.navigation:navigation-ui-ktx:2.9.5` — Sinkronisasi otomatis judul `Toolbar` dan tombol navigasi *Up* dengan `AppBarConfiguration`.

## 3. Struktur Direktori Pengumpulan & Proyek

```text
P04_240104040057_Muhammad_Zaini_Arsyad/
├── README.md                                 # Dokumentasi utama proyek dan panduan penggunaan
├── docs/                                     # Dokumen laporan praktikum, sketsa alur, dan log validasi
│   ├── Laporan_P04.docx                      # Laporan resmi praktikum (Sketsa, Analisis, Matriks Uji, Refleksi)
│   ├── HASIL_VALIDASI.md                     # Rekapitulasi matriks pengujian aktual T01–T16 & uji teman
│   ├── SKETSA_ALUR.svg                       # Diagram sketsa alur 3 layar (Beranda, Daftar, Rincian)
│   ├── hasil_uji_akhir.json                  # Rekap data pengujian terstruktur beserta hash APK
│   ├── gradle-sync.txt                       # Bukti log keberhasilan Gradle Sync
│   ├── build-log.txt                         # Bukti log build APK utama
│   ├── build-invalid-id-test.txt             # Bukti log build pengujian cabang ID tidak ditemukan (-99L)
│   ├── build-ux-validation.txt               # Bukti log build & lint setelah penambahan layout-land
│   ├── lint-results-debug.txt                # Laporan pemeriksaan kode Android Lint (0 error)
│   ├── talkback-service.txt                  # Bukti status layanan aksesibilitas TalkBack pada perangkat uji
│   └── talkback-hierarchy.xml                # Dump hirarki UI aksesibilitas saat pengujian TalkBack
├── screenshots/                              # 26 bukti tangkapan layar skenario T01–T16 & evaluasi UX
│   ├── 01_beranda.png ... 26_pulihkan_akhir.png
└── WargaHubP04/                              # Root proyek Android Studio
    ├── build.gradle.kts                      # Konfigurasi Gradle tingkat root
    ├── settings.gradle.kts                   # Pengaturan modul (:app) dan repositori
    ├── gradle.properties                     # Konfigurasi JVM dan properti AndroidX
    ├── gradlew & gradlew.bat                 # Skrip eksekusi Gradle Wrapper
    ├── gradle/wrapper/                       # File JAR dan konfigurasi Gradle Wrapper
    └── app/
        ├── build.gradle.kts                  # Konfigurasi modul :app, SDK, ViewBinding, dan dependensi
        └── src/main/
            ├── AndroidManifest.xml           # Registrasi tunggal MainActivity dan tema Theme.WargaHubP04
            ├── java/id/antasari/p04/wargahub/
            │   ├── MainActivity.kt           # Wadah utama, konfigurasi Toolbar, NavController, dan WindowInsets
            │   ├── data/
            │   │   ├── Laporan.kt            # Model data class laporan (id: Long, judul, kategori, deskripsi, status)
            │   │   └── LaporanContoh.kt      # Objek singleton penyedia 15 data laporan fiktif dan fungsi cari(id)
            │   ├── adapter/
            │   │   └── LaporanAdapter.kt     # Implementasi ListAdapter, ViewHolder, DiffUtil, dan contentDescription
            │   └── ui/
            │       ├── HomeFragment.kt       # Layar Beranda dengan aksi navigasi menuju daftar laporan
            │       ├── ListFragment.kt       # Layar Daftar Laporan, RecyclerView, dan mode Uji Daftar Kosong
            │       └── DetailFragment.kt     # Layar Rincian Laporan berdasarkan argumen laporan_id (Long)
            └── res/
                ├── layout/
                │   ├── activity_main.xml     # Layout wadah utama (Toolbar + FragmentContainerView)
                │   ├── fragment_home.xml     # Layout layar Beranda dengan ScrollView
                │   ├── fragment_list.xml     # Layout layar Daftar Laporan (Portrait) dengan FrameLayout & emptyContainer
                │   ├── item_laporan.xml      # Layout kartu baris laporan individual pada RecyclerView
                │   └── fragment_detail.xml   # Layout layar Rincian Laporan dengan ScrollView
                ├── layout-land/
                │   └── fragment_list.xml     # Variasi layout Daftar Laporan khusus Landscape (Perbaikan UX Tahap 6.J)
                ├── navigation/
                │   └── nav_graph.xml         # Peta navigasi 3 Fragment, 2 action, dan argumen laporan_id (default -1L)
                ├── values/
                │   ├── strings.xml           # Seluruh teks statis dan template berparameter (%1$d, %1$s)
                │   ├── colors.xml            # Palet warna resmi (p04_navy, p04_slate, p04_page, p04_white)
                │   ├── dimens.xml            # Standar jarak padding halaman (16dp) dan celah antarelemen (12dp)
                │   ├── styles.xml            # Gaya tipografi (Text, Title) dan tombol (Button minHeight 48dp)
                │   └── themes.xml            # Tema terang Theme.WargaHubP04 (NoActionBar)
                └── values-night/
                    └── themes.xml            # Konfigurasi tema identik agar kontras tetap konsisten di mode malam
```

## 4. Cara Membuka, Membangun (Build), dan Menjalankan Proyek

### A. Melalui Android Studio
1. Buka **Android Studio**, pilih menu **File > Open**.
2. Arahkan ke direktori `WargaHubP04` yang berada di dalam folder pengumpulan ini, lalu klik **OK**.
3. Tunggu hingga proses **Gradle Sync** selesai tanpa pesan kesalahan. Pastikan **Gradle JDK** pada *Settings > Build, Execution, Deployment > Build Tools > Gradle* menggunakan **Embedded JDK (JBR 21)** atau JDK yang kompatibel.
4. Nyalakan emulator yang mendukung minimal API 26 (direkomendasikan API 36/37) atau hubungkan perangkat fisik Android dengan *USB Debugging* aktif.
5. Pilih konfigurasi **`app`** pada toolbar atas, lalu klik tombol **Run (`▶`)**.

### B. Melalui Terminal / Windows PowerShell
Buka terminal pada direktori `WargaHubP04/` (tempat file `settings.gradle.kts` dan `gradlew.bat` berada), lalu jalankan perintah berikut:

```powershell
# Windows PowerShell — Membangun APK Debug dan menjalankan pemeriksaan Android Lint
.\gradlew.bat assembleDebug lintDebug

# macOS / Linux
./gradlew assembleDebug lintDebug
```

- **Lokasi File APK Debug**: `WargaHubP04/app/build/outputs/apk/debug/app-debug.apk`
- **Lokasi Laporan Lint**: `WargaHubP04/app/build/reports/lint-results-debug.html`

## 5. Ringkasan Fitur Inti & Arsitektur Teknis

1. **Arsitektur *Single-Activity* dengan Tiga *Fragment***:
   - Aplikasi hanya mendaftarkan satu `MainActivity` di dalam `AndroidManifest.xml`.
   - `MainActivity` mengatur `Toolbar` kustom, menangani *system bars* dan *display cutout* satu kali pada level *root* menggunakan `WindowInsetsCompat`, serta menghubungkan `NavController` dengan `NavigationUI` (`setupActionBarWithNavController`).
   - Perpindahan layar antara `HomeFragment`, `ListFragment`, dan `DetailFragment` berlangsung di dalam `FragmentContainerView` (`@id/navHostFragment`) sesuai definisi pada `res/navigation/nav_graph.xml`.
2. **Manajemen Daftar Efisien dengan `RecyclerView`, `ListAdapter`, dan `DiffUtil`**:
   - `LaporanAdapter` mewarisi `ListAdapter<Laporan, LaporanHolder>` dengan objek `DiffUtil.ItemCallback` (`Pembanding`) yang membandingkan identitas unik (`a.id == b.id`) dan kesamaan isi (`a == b`).
   - Pemulihan posisi gulir (*scroll state*) saat rotasi atau kembali dari halaman rincian dijaga menggunakan `RecyclerView.Adapter.StateRestorationPolicy.PREVENT_WHEN_EMPTY`.
   - Setiap baris laporan (`item_laporan.xml`) dirancang sebagai satu target fokus aksesibilitas utuh (`clickable="true"`, `focusable="true"`, `minHeight="72dp"`), sementara ketiga `TextView` di dalamnya diberi `importantForAccessibility="no"` dan dirangkum melalui `binding.root.contentDescription` agar pembaca layar (**TalkBack**) membacakan informasi baris secara jelas tanpa pengulangan.
3. **Siklus Hidup *ViewBinding* yang Aman dari Kebocoran Memori (*Memory Leak*)**:
   - Pada ketiga `Fragment`, variabel `_binding` dibuat *nullable*, diinisialisasi di dalam `onViewCreated` menggunakan `Binding.bind(view)`, dan wajib dikosongkan (`_binding = null`) pada `onDestroyView()`.
   - Khusus pada `ListFragment`, referensi `binding.rvLaporan.adapter = null` dan `laporanAdapter = null` juga dilepas di dalam `onDestroyView()` agar objek `View` maupun *lambda callback* tidak tertahan di memori ketika `Fragment` berpindah ke *back stack*.
4. **Pengiriman Argumen Navigasi & Perlindungan *Back Stack***:
   - Saat baris laporan dipilih, `ListFragment` mengirim ID laporan bertipe `Long` (`bundleOf(DetailFragment.ARG_ID to laporan.id)`) melalui aksi `R.id.action_list_to_detail` setelah memverifikasi `nav.currentDestination?.id == R.id.listFragment` (mencegah *crash* akibat ketukan ganda/cepat).
   - `DetailFragment` membaca argumen `laporan_id` dengan nilai *default* `-1L`, lalu memanggil `LaporanContoh.cari(id)`.
   - Tombol *Kembali ke Daftar* pada `DetailFragment` memanggil `findNavController().navigateUp()` (bukan `navigate()` ke `ListFragment` baru) sehingga tidak menumpuk duplikat halaman daftar di *back stack*.
5. **Pengelolaan Keadaan Tampilan (*Empty State* & *Error State*)**:
   - **Mode Daftar Kosong**: Menekan tombol *Uji Daftar Kosong* mengubah *flag* `contohKosong` menjadi `true`, mengirimkan `emptyList()` ke `submitList()`, menyembunyikan `rvLaporan`, menampilkan `emptyContainer` beserta pesan petunjuk pemulihan, dan mengubah teks tombol menjadi *Tampilkan Data Contoh*. Nilai `contohKosong` disimpan di dalam `onSaveInstanceState` (`KEY_KOSONG = "contoh_kosong"`) agar tetap bertahan saat rotasi layar.
   - **Cabang Laporan Tidak Ditemukan**: Apabila ID yang diterima `DetailFragment` tidak ditemukan di `LaporanContoh` (`laporan == null`), layar menampilkan pesan aman `@string/detail_missing` dan menyembunyikan elemen kategori, status, serta deskripsi tanpa menggunakan operator paksa `!!`.

## 6. Batas Data Contoh (Simulasi Lokal)

- Seluruh 15 laporan masalah lingkungan (*Lampu jalan padam*, *Sampah menumpuk*, dan *Drainase tersumbat* pada zona latihan 1 s.d. 15) dibentuk secara lokal di memori melalui ekspresi `(1..15).map { ... }` di dalam `LaporanContoh.kt` dengan status `"Simulasi"`.
- Aplikasi pada Praktikum 04 ini **tidak menggunakan database lokal** (seperti SQLite/Room), **tidak menyimpan perubahan data secara permanen**, **tidak memiliki fitur CRUD** (tambah/ubah/hapus laporan), dan **tidak memerlukan izin jaringan (`INTERNET`)**.
- Munculnya kembali ke-15 laporan contoh setelah aplikasi ditutup paksa (*force-stop*) dan dibuka ulang dari *launcher* terjadi karena objek `LaporanContoh` menginisialisasi ulang daftar statis yang sama dari kode sumber, bukan karena adanya penyimpanan persisten.

## 7. Hasil pengujian dan dokumentasi

Build dan lint berhasil; 0 error dan 8 peringatan nonfatal. Implementasi dan skenario
fungsional didokumentasikan dalam docs/Laporan_P04.docx serta docs/HASIL_VALIDASI.md.
Lima layout utama ditambah satu variasi landscape tersedia. Variasi landscape
memperbesar ruang daftar pada layar pendek; bukti sebelum/sesudah ada pada screenshot 15 dan 16.

Uji Aby Wahyudi pada 9 November 2026: 5,30 detik dari Beranda sampai rincian zona 10,
kembali ke daftar tanpa bantuan, nilai akhir 5/5. Penguji sudah mengenal aplikasi.
Tidak ada data waktu sebelum perbaikan untuk perbandingan.

TalkBack: penguji mengonfirmasi pembacaan lengkap, aktivasi rincian dengan klik dua kali,
dan kembali ke daftar. Gulir saat TalkBack aktif belum berhasil pada emulator.

Folder screenshots memuat bukti teknis asli. Screenshot fokus/navigasi TalkBack
tidak menjadi bukti audio; hasil audio berasal dari konfirmasi manual penguji.

## 8. Pengumpulan

Sertakan README, docs/Laporan_P04.docx, screenshot, source dan Gradle Wrapper.
ZIP mengecualikan .gradle, .idea, .kotlin, build, local.properties dan kredensial.
Jangan mengeluarkan gradle/wrapper.
#   P 0 4 _ 2 4 0 1 0 4 0 4 0 0 5 7 _ M u h a m m a d _ Z a i n i _ A r s y a d  
 #   P 0 4 _ 2 4 0 1 0 4 0 4 0 0 5 7 _ M u h a m m a d _ Z a i n i _ A r s y a d  
 