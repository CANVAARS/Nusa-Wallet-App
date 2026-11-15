# Nusa Wallet

**Nusa Wallet** adalah aplikasi Android untuk ekosistem **Nusachain** (koin **NUX / Nusacoin**). Aplikasi ini menggabungkan dompet (Nusa Card), berita, toko, bursa tukar, dan direktori DApp serta layanan ekosistem dalam satu aplikasi mobile.

- **Package:** `org.nusacoin.nusawallet`
- **Versi:** 1.0 (`versionCode` 1)
- **Android minimum:** 9 (API 28) · **target:** API 34
- **Website ekosistem:** [nusachain.org](https://www.nusachain.org) · [nusacoin.org](https://www.nusacoin.org)

---

## Daftar Isi

- [Fitur](#fitur)
- [Direktori DApp & Layanan](#direktori-dapp--layanan)
- [Cara Kerja (Arsitektur)](#cara-kerja-arsitektur)
- [Struktur Proyek](#struktur-proyek)
- [Build & Menjalankan](#build--menjalankan)
- [Menambah DApp / Layanan Baru](#menambah-dapp--layanan-baru)
- [Izin Aplikasi](#izin-aplikasi)
- [Keamanan](#keamanan)
- [Catatan Pengembangan](#catatan-pengembangan)
- [Lisensi](#lisensi)

---

## Fitur

Aplikasi memiliki lima menu pada navigasi bawah, ditambah menu **Pengaturan**:

| Menu | Isi |
|---|---|
| **Dompet** | **Nusa Card** (*Nusacoin Wallet & Decrypter Suite*), tertanam langsung di dalam aplikasi |
| **Berita** | Artikel terbaru dari blog [nusacoin.org](https://www.nusacoin.org) (feed Blogger) |
| **Toko** | Toko Nusachain ([shop.nusachain.org](https://shop.nusachain.org/)) |
| **Tukar** | Pasar NUX/USDT di QuTrade |
| **DApps** | Direktori DApp dan layanan ekosistem (lihat bagian berikut) |
| **Pengaturan** | Tema gelap/terang dan bahasa (Indonesia/Inggris) |

### Nusa Wallet (koneksi ke DApp)

Tombol **Buka Nusa Wallet** di menu DApps membuka layar wallet native (UI yang sama dengan ekstensi Chrome *Nusa Wallet*):

- **Buat / impor wallet** (Bech32 `nu1q…` atau P2PKH `N…`) dengan password, lalu **cadangkan WIF** (wajib).
- **Kunci tersimpan terenkripsi** di penyimpanan privat aplikasi (`filesDir`, bukan cache): PBKDF2 (250.000 iterasi) + AES-GCM. Bertahan selama aplikasi terpasang. WIF yang terbuka hanya ada di memori dan dihapus saat **auto-lock** (default 15 menit) atau tombol *Kunci*.
- **Buka dengan biometrik** (opsional, Pengaturan wallet): password dibungkus kunci Android Keystore yang butuh sidik jari/wajah.
- **Saldo, kirim NUX, riwayat**, dan daftar **situs terhubung** (bisa dicabut).
- **Mode DApp**: halaman di domain `nusachain.org`, `nusachain.site`, dan `nusacoin.site` dibuka di peramban mini yang menyediakan **`window.nux`**. Setiap connect, kirim NUX, tulis data (OP_RETURN, maks 80 byte), dan tanda tangan pesan wajib disetujui di layar persetujuan.

### Nusa Card (menu Dompet)

- **Paper Wallet**: membuat kartu dompet kertas Nusacoin dan mengunduhnya sebagai PNG (disimpan ke folder *Download*).
- **Dekripsi**: mendekripsi private key.
- **Live Wallet**: **Saldo**, **Terima**, **Kirim**, dan **Riwayat** transaksi NUX (kirim NUX memakai dialog konfirmasi).
- **Scan QR** lewat kamera atau **Upload QR** dari galeri.

## Direktori DApp & Layanan

Menu **DApps** menampilkan **57 layanan terintegrasi**, dengan filter kategori: *Semua, Tools & Portal, Explorers, Mining Pools, Sumber Daya,* dan *Perkenalan*.

| Kategori | Isi |
|---|---|
| **Tools & Portal** | Blockspot.io Voting, VeriSign Mainnet, VeriSign Testnet, Nusa Card Tool, Paya Chain, Read This File, TA Render, Tanya Nusa, Blog Hash Generator |
| **Explorers** | Scan Nusacoin, Taobot Explorer, Chain Nusacoin, Nusacoin Main Explorer, Taobot Book, NusaScan, Block NusaScan, Testnet Explorer |
| **Mining Pools** | Nusacoin Official Pool, Nusacoin Java Pool, Taobot Pool, ALR8 Pool, Koinium Pool |
| **Sumber Daya** | Nusacoin Core (Windows & Linux), Nusacoin Wallet, Paper Wallet, Green Wallet, Dompet Web, Taobot Wallet App, CPU Miner, GPU Miner, Kosa Kata, Logo Drive, Musik Drive |
| **Perkenalan** | Individu, Bisnis, Developer Portal, Memulai Nusacoin, Visi Misi & Roadmap, Cara Kerja Nusacoin, Whitepaper (ID & EN) |

Selain itu tersedia kolom **link kustom** untuk membuka tautan sembarang melalui peramban mini aplikasi.

> Daftar ini mengikuti isi `app/src/main/assets/index.html`. Kumpulan tools lengkap ekosistem ada di [nusachain.org/Tools](https://www.nusachain.org/Tools).

## Cara Kerja (Arsitektur)

Nusa Wallet adalah **cangkang native Android (WebView)** yang memuat satu halaman web lokal.

```
┌───────────────────────────── Aplikasi Android ─────────────────────────────┐
│  MainActivity (WebView utama)                                               │
│   └─ memuat https://appassets.androidplatform.net/assets/index.html         │
│        (assets/index.html, Tailwind + Font Awesome)                         │
│        ├─ Tab Dompet  → Nusa Card (tertanam, dimuat via iframe srcdoc)      │
│        ├─ Tab Berita  → feed Blogger nusacoin.org                           │
│        ├─ Tab Toko    → iframe shop.nusachain.org                           │
│        ├─ Tab Tukar   → iframe qutrade.io (NUX/USDT)                        │
│        └─ Tab DApps   → direktori kartu layanan                             │
│                                                                             │
│  BrowserActivity  → peramban mini untuk semua link luar                     │
│  Links            → aturan pembukaan link (web / unduhan / skema lain)      │
│  FileSaver        → simpan unduhan data: (mis. kartu PNG) ke Download       │
└─────────────────────────────────────────────────────────────────────────────┘
```

Poin penting:

- **`assets/index.html`** disajikan lewat `WebViewAssetLoader` pada alamat `https://appassets.androidplatform.net/assets/`, sehingga tidak memakai akses `file://`.
- **Link luar** (di luar host aplikasi) dibuka di `BrowserActivity`, peramban mini terpisah. Tautan unduhan (`.rar .zip .7z .apk .exe .msi .dmg .tar .gz .pdf`) dan skema non-web (`mailto:`, `tg://`, `market:`, dll.) diserahkan ke aplikasi atau browser bawaan HP.
- **Situs yang menolak iframe** (`blockspot.io`, `qutrade.io`) otomatis dibuka sebagai halaman penuh di `BrowserActivity`; iframe diganti tombol "Buka lagi".
- **Unduhan** berupa `data:` URL (mis. kartu PNG) disimpan ke folder *Download* melalui `FileSaver` (MediaStore di Android 10+, izin penyimpanan di Android 9).
- **Cookie** diaktifkan agar sesi login di tab Toko tetap tersimpan setelah aplikasi ditutup.
- Aplikasi menampilkan peringatan jika **tidak ada koneksi internet**, karena sebagian besar konten dimuat online.

## Struktur Proyek

```
NusaWallet-Android/
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
├── .github/workflows/build.yml          # CI: build APK debug
└── app/
    ├── build.gradle.kts
    └── src/main/
        ├── AndroidManifest.xml
        ├── assets/
        │   ├── index.html               # NusaChain Mobile DApp (UI utama)
        │   ├── logo.png
        │   └── wallet/                     # Nusa Wallet (UI + engine, dipakai ulang dari ekstensi Chrome)
        │       ├── popup.html/js/css       # layar wallet & persetujuan DApp (sama dengan ekstensi)
        │       ├── background.js           # logika DApp & izin (sama dengan ekstensi)
        │       ├── lib/nusa-core.js        # kripto, transaksi, vault
        │       ├── shim.js                 # API chrome.* di atas jembatan native
        │       ├── engine.html             # halaman engine (WebView tersembunyi)
        │       ├── provider.js             # window.nux untuk DApp
        │       └── android.css, android-extras.js
        ├── java/org/nusacoin/nusawallet/
        │   ├── MainActivity.kt          # WebView utama, izin kamera, file chooser
        │   ├── BrowserActivity.kt       # Peramban mini untuk link luar
        │   ├── Links.kt                 # Aturan pembukaan link (+ daftar domain DApp)
        │   ├── WalletHost.kt            # Penyimpanan, engine, router pesan, auto-lock
        │   ├── WalletBridge.kt          # Jembatan JS (hanya halaman milik aplikasi)
        │   ├── WalletActivity.kt        # Layar wallet + persetujuan DApp
        │   ├── DappBridge.kt            # window.nux untuk DApp (origin whitelist)
        │   ├── BiometricVault.kt        # Buka kunci biometrik (Android Keystore)
        │   └── FileSaver.kt             # Simpan unduhan ke folder Download
        └── res/                         # Ikon, string, tema, warna
```

## Build & Menjalankan

### Prasyarat

- JDK **17**
- Android Studio (atau Gradle **8.2.1**)
- Android Gradle Plugin 8.2.2 · Kotlin 1.9.22 · `compileSdk` 34
- Dependensi: `androidx.webkit:webkit:1.9.0`

### Build lokal

1. Buka folder proyek di Android Studio dan tunggu **Gradle Sync** selesai.
2. Jalankan di emulator/perangkat (**Run ▶**), atau build APK debug:
   ```bash
   gradle assembleDebug
   ```
   APK berada di `app/build/outputs/apk/debug/`.

> Repositori ini tidak menyertakan *Gradle wrapper* (`gradlew`), jadi gunakan Android Studio atau Gradle 8.2.1 yang terpasang.

### Build otomatis (GitHub Actions)

Workflow `.github/workflows/build.yml` berjalan pada setiap `push` dan bisa dipicu manual (`workflow_dispatch`). Ia membangun `assembleDebug` dengan JDK 17 dan Gradle 8.2.1, lalu mengunggah artefak **`NusaWallet-debug`** (APK).

## Menambah DApp / Layanan Baru

Direktori DApp berada di tab `tab-dapps` pada `app/src/main/assets/index.html`.

1. Tambahkan kartu baru pada kategori yang sesuai (*Tools & Portal, Explorers, Mining Pools, Sumber Daya,* atau *Perkenalan*) dengan meniru kartu yang sudah ada.
2. Tombol kartu membuka tautan melalui peramban mini (`BrowserActivity`). Tautan unduhan akan diserahkan ke aplikasi/browser HP.
3. Jika DApp menolak tampil di dalam iframe, tambahkan host-nya ke `FRAME_BLOCKED_HOSTS` di `MainActivity.kt`.
4. Perbarui label jumlah layanan ("38 Layanan Terintegrasi") bila perlu.
5. Build ulang dan uji di perangkat.

## Izin Aplikasi

| Izin | Kegunaan |
|---|---|
| `INTERNET`, `ACCESS_NETWORK_STATE` | Memuat konten online dan mengecek koneksi |
| `CAMERA` (opsional) | Scan QR di Nusa Card |
| `WRITE_EXTERNAL_STORAGE` (hanya Android 9 ke bawah) | Menyimpan kartu PNG ke folder Download |

Kamera tidak wajib ada di perangkat (`required="false"`).

## Keamanan

- **Kunci Anda adalah tanggung jawab Anda.** Simpan private key dan cadangan di tempat aman; jangan dibagikan kepada siapa pun.
- Izin kamera hanya diberikan kepada **halaman milik aplikasi**; situs luar yang meminta kamera ditolak.
- Link luar dibuka di `BrowserActivity` yang **terpisah dari WebView dompet**: tanpa jembatan JavaScript, tanpa izin kamera, dan tanpa berbagi halaman dengan dompet.
- `usesCleartextTraffic="false"` dan `allowBackup="false"`; akses `file://` dan `content://` pada WebView dinonaktifkan.
- Periksa alamat tujuan dan jumlah sebelum mengonfirmasi pengiriman NUX.
- Unduh aplikasi, core wallet, dan miner hanya dari sumber resmi Nusacoin.

## Integrasi DApp (window.nux)

Di mode DApp, halaman mendapat `window.nux` dengan API yang **sama dengan ekstensi Chrome**, sehingga DApp cukup ditulis sekali:

```js
const { address, publicKey } = await window.nux.connect();
const bal  = await window.nux.getBalance();                          // NUX
const sig  = await window.nux.signMessage('halo');
const tx   = await window.nux.sendTransaction({ to: 'nu1q…', amount: 1.5 });   // { txid }
const note = await window.nux.sendData({ data: 'sha256-dokumen' });             // OP_RETURN, maks 80 byte -> { txid }
window.addEventListener('nux#initialized', () => { /* provider siap */ });
```

Keamanan jembatan:

- Provider hanya diinjeksi ke origin di `Links.DAPP_HOSTS` (aturan origin WebView). Untuk menambah domain, ubah daftar itu.
- Hanya frame utama yang dilayani; origin diambil dari Android (`sourceOrigin`), bukan dari data halaman.
- Halaman hanya bisa memanggil method `nux_*`. Private key tidak pernah dikirim ke DApp.
- Halaman wallet dan engine hanya memuat aset lokal; link luar dibuka di peramban mini terpisah.
- Layar wallet memakai `FLAG_SECURE` (tanpa screenshot/pratinjau recent apps).

## Catatan Pengembangan

- `assets/wallet/` (kecuali `shim.js`, `engine.html`, `provider.js`, `android*.{css,js}`) adalah salinan dari ekstensi Chrome. Perbaikan di satu sisi sebaiknya disalin ke sisi lain.
- Vault tidak ikut cadangan Android (`allowBackup="false"`): jika aplikasi di-uninstall atau datanya dihapus, wallet hilang. **Pengguna harus menyimpan WIF.**
- `MSG_MAGIC` pada `nusa-core.js` ditandai *asumsi* oleh penulisnya; samakan dengan `strMessageMagic` Nusacoin Core sebelum `signMessage` dipakai untuk verifikasi.
- Tab **Dompet** (Nusa Card tertanam, `NUSA_CARD_B64`) tetap ada untuk paper wallet, dekripsi, dan live wallet.

## Lisensi

Tentukan lisensi proyek di sini (mis. MIT) dan tambahkan file `LICENSE`.

---

**Nusa Wallet**: gerbang mobile menuju ekosistem Nusachain.
