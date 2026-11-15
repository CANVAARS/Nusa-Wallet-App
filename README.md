# Nusa Wallet (Android)

Aplikasi Android untuk ekosistem **Nusacoin (NUX) / Nusachain**. Aplikasi ini berupa *shell* WebView berbasis Kotlin yang memuat halaman HTML dari folder `assets`, sehingga tampilan dan fitur dapat diubah cukup dengan mengedit file HTML.

- Package: `org.nusacoin.nusawallet`
- Min SDK 28 (Android 9), target/compile SDK 34
- Build: Gradle (Kotlin DSL), JDK 17

## Fitur Utama

| Menu | Keterangan |
|------|------------|
| **Dompet** | Membuka `features/Nusa-Card.html` (Nusa Card Wallet: paper wallet, dekripsi private key, live wallet). |
| **DApps** | Direktori ekosistem (Tools, Explorer, Pool, Sumber Daya, Pengantar) + bagian **Fitur Aplikasi**. |
| **Toko** | Membuka `features/shop.html`; portal Seller/Admin lewat `features/login.html`. |
| **Pembaruan** | Cek rilis terbaru di GitHub dan unduh/instal APK dari dalam aplikasi. |

Aplikasi membutuhkan koneksi internet (font, ikon, dan layanan Nusachain dimuat dari web).

## Struktur Proyek

```
NusaWallet-Android/
├── .github/workflows/build.yml      # Build APK otomatis
├── app/
│   ├── build.gradle.kts
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── assets/
│       │   ├── index.html           # Antarmuka utama (DApps, pengaturan, dsb.)
│       │   ├── logo.png
│       │   └── features/            # Halaman fitur (HTML)
│       │       ├── Nusa-Card.html   # Tab Dompet (file khusus)
│       │       ├── shop.html        # Tab Toko (file khusus)
│       │       ├── login.html       # Portal Seller/Admin (file khusus)
│       │       └── README.txt
│       └── java/org/nusacoin/nusawallet/
│           ├── MainActivity.kt      # WebView utama, navigasi tab
│           ├── BrowserActivity.kt   # Browser dalam aplikasi untuk link web
│           ├── Features.kt          # Daftar fitur otomatis dari assets/features
│           ├── Links.kt             # Aturan pembukaan link
│           ├── FileSaver.kt         # Simpan file ke folder Download
│           ├── UpdateChecker.kt     # Cek rilis GitHub
│           └── ApkUpdater.kt        # Unduh & instal APK
├── build.gradle.kts
├── settings.gradle.kts
└── gradle.properties
```

## Cara Build

### Lewat GitHub Actions (disarankan)
1. Push proyek ke repositori GitHub.
2. Workflow **Build APK** jalan otomatis pada setiap push (atau jalankan manual lewat *Run workflow*).
3. Unduh APK dari **Artifacts → NusaWallet-debug**.

### Lokal
Butuh JDK 17 dan Gradle 8.2.1:

```bash
gradle assembleDebug
# hasil: app/build/outputs/apk/debug/*.apk
```

### Build rilis bertanda tangan
Build rilis ditandatangani bila environment variable berikut tersedia: `KEYSTORE_FILE`, `KEYSTORE_PASSWORD`, `KEY_ALIAS` (default `nusawallet`). Versi dan repo diisi lewat properti Gradle `-PappVersionName`, `-PappVersionCode`, `-PappReleaseDate`, dan `-PgithubRepo` (`pemilik/repo`, dipakai pengecek pembaruan).

## Menambah Konten

### 1. Menambah link/tools ke direktori DApps (cara termudah)
Edit `app/src/main/assets/index.html`, cari `DAPPS_TAMBAHAN`, lalu tambahkan satu blok per kartu:

```js
{
    judul: 'Nama Tool',
    deskripsi: 'Deskripsi singkat (Indonesia).',
    deskripsiEN: 'Short description (English).',
    url: 'https://contoh.com/Tool.html',
    kategori: 'tools',      // tools | explorers | pools | resources | intro
    label: 'Dokumen',       // teks kecil di pojok kartu (opsional)
    ikon: 'fa-certificate', // ikon Font Awesome (opsional)
    tombol: 'Buka Alat'     // teks tombol (opsional)
},
```

Kartu otomatis ikut pencarian, filter kategori, dan hitungan layanan. Hanya URL `http/https` yang diterima.

Tools yang sudah terdaftar di bagian ini (diambil dari direktori Tools Nusachain): Certificate, Notaris, Split UTXO, Traceability, Kas RT Koperasi, Tiket Event, Voting, Angpao THR, Berburu Harta Karun, Tanda Tangan Wallet, Pengirim Massal, Univ-Chain, Donasi, Jejak Revisi DED, Kwitansi Nusachain, dan Pencatatan Waris.

### 2. Menambah fitur bawaan (HTML di dalam aplikasi)
Taruh file `.html` di `app/src/main/assets/features/`. File otomatis muncul di tab DApps bagian **Fitur Aplikasi** tanpa mengubah kode Kotlin.

- Atur judul, ikon, deskripsi, dan urutan lewat tag meta di dalam `<head>`:

  ```html
  <title>Nama Fitur</title>
  <meta name="nusa:title" content="Nama Fitur">
  <meta name="nusa:title-en" content="Feature Name">
  <meta name="nusa:desc" content="Deskripsi singkat">
  <meta name="nusa:desc-en" content="Short description">
  <meta name="nusa:icon" content="fa-solid fa-star">
  <meta name="nusa:order" content="10">
  ```
- Tanpa meta, judul diambil dari `<title>`.
- File berawalan `_` diabaikan. `shop.html`, `login.html`, dan `Nusa-Card.html` adalah file khusus dan tidak tampil sebagai kartu.
- Gunakan link relatif antar file (`href="login.html"`), bukan URL penuh.
- Tag meta harus berada di 16 KB pertama file.
- Setelah menambah atau mengganti file, build ulang APK karena folder ini ikut dikemas.

## Izin Aplikasi

| Izin | Kegunaan |
|------|----------|
| `INTERNET`, `ACCESS_NETWORK_STATE` | Memuat layanan Nusachain dan cek koneksi |
| `CAMERA` | Scan QR |
| `WRITE_EXTERNAL_STORAGE` | Menyimpan file ke Download (perangkat lama) |
| `REQUEST_INSTALL_PACKAGES` | Memasang pembaruan APK |

## Keamanan

- Halaman lokal disajikan lewat `WebViewAssetLoader` (`https://appassets.androidplatform.net/`), tanpa JavaScript bridge pada daftar fitur.
- Link web biasa dibuka di browser dalam aplikasi; file unduhan (`.apk`, `.zip`, `.pdf`, dll.) dan skema lain diserahkan ke aplikasi/browser HP.
- **Private key tidak pernah boleh dibagikan.** Simpan cadangan paper wallet di tempat aman dan offline.

## Pemecahan Masalah

- **Fitur baru tidak muncul:** pastikan file `.html` ada di `assets/features/`, namanya tidak diawali `_`, lalu build ulang APK.
- **Build gagal setelah menambah HTML:** cek log workflow (`--stacktrace`). Sebagai alternatif, daftarkan tool lewat `DAPPS_TAMBAHAN` tanpa file baru.
- **Halaman kosong:** aplikasi membutuhkan internet; periksa koneksi.
