# Sistem Pengelolaan SPPT Desa

Paket ini berisi antarmuka HTML responsif, backend Google Apps Script, dan data demo. Semua menu utama pada HTML dapat dicoba dengan data demo. Data demo disimpan di penyimpanan browser (localStorage), jadi belum dibagikan ke perangkat lain. Untuk database terpusat, gunakan Code.gs pada satu spreadsheet dan hubungkan antarmuka ke endpoint sesuai bagian **Integrasi aplikasi**.

## Isi paket

- `aplikasi-pengelolaan-sppt.html` — aplikasi browser, login demo, dashboard, data SPPT, pembagian, pembayaran penarikan bertahap per SPPT, status Dibayar Sebagian, pembagian pembayaran per nama pemilik di akun penarik, catatan nominal dan status lunas/belum per pemilik, laporan, pengajuan/persetujuan pindah, akun penarik, CSV, cetak, dan cadangan JSON.
- `Code.gs` — backend JSON Google Apps Script untuk satu Google Spreadsheet. Menyediakan validasi role di server, sesi login, CRUD SPPT, penarikan, permintaan pindah, dashboard, laporan, impor, dan log aktivitas.
- `README-PETUNJUK.md` — panduan ini.

## Coba demo lokal

1. Buka `aplikasi-pengelolaan-sppt.html` di browser modern.
2. Masuk dengan `admin` / `admin123`, atau `penarik1`, `penarik2`, `penarik3` dengan password `123456`.
3. Data contoh memuat 24 SPPT dengan beberapa status. Gunakan menu SPPT untuk pencarian, detail, pengeditan admin, atau pencatatan penarik. Menu Pembagian SPPT membagi menurut dusun. Penarik dapat mengajukan pindah dari tombol **Ajukan pindah** pada daftar SPPT. Admin memutuskan permintaan di **Permintaan Pindah SPPT**.
4. Demo berada pada browser dan profil yang sama. Tombol **Unduh cadangan data demo** membuat backup JSON.

## Siapkan database Google Sheets dan backend

1. Buat satu Google Spreadsheet untuk desa. Salin ID di antara `/d/` dan `/edit` pada URL spreadsheet.
2. Buka **Ekstensi → Apps Script** dari spreadsheet tersebut.
3. Ganti isi `Code.gs` dengan file Code.gs dari paket ini, lalu simpan.
4. Pilih fungsi `setup` dari dropdown fungsi dan tekan **Jalankan**. Berikan izin Apps Script. Fungsi ini menyiapkan sheet `USERS`, `SPPT`, `PENARIKAN`, `PINDAH_SPPT`, `RIWAYAT_AKTIVITAS`, `PENGATURAN`, membuat akun demo, menambahkan kolom JUMLAH_PEMILIK, PEMILIK_BERSAMA, dan JUMLAH_TERBAYAR pada SPPT serta JUMLAH_DIBAYAR dan NAMA_PEMBAYAR pada PENARIKAN, dan menambahkan 24 baris SPPT contoh jika sheet SPPT masih kosong. Jalankan setup() lagi setelah memperbarui backend agar header baru ditambahkan. Apps Script terikat ke spreadsheet aktif; ID disimpan otomatis di Script Properties.
5. Uji login lewat deployment setelah deploy. Akun awal: `admin` / `admin123`; `penarik1` sampai `penarik3` / `123456`. Segera ubah password demo sebelum dipakai dengan data asli.
6. Tekan **Deploy → Deployment baru → Aplikasi web**. Pilih **Jalankan sebagai: Saya**. Pilihan akses harus sesuai kebijakan organisasi Google Workspace Anda. Opsi siapa pun membuat endpoint dapat dijangkau publik; tetap semua fungsi data memerlukan token sesi yang diterbitkan setelah login. Jangan bagikan URL spreadsheet.
7. Deploy dan salin URL yang berakhiran `/exec`. Setelah perubahan Code.gs, buat versi deployment baru atau edit deployment yang ada.

## Integrasi aplikasi

> **Status koneksi paket ini:** Versi HTML terbaru mendukung login, memuat daftar SPPT/akun/penarikan/perpindahan dari Sheets, serta menulis pendataan, pembagian, penarikan, pengajuan dan keputusan pindah, pengelolaan akun, dan impor CSV melalui backend. Simpan URL Web App pada menu Pengaturan di tiap perangkat lalu keluar dan masuk kembali memakai akun Sheets. Bukti foto belum diunggah ke Drive; hanya nama file yang dicatat.

Apps Script mendukung permintaan `POST` berisi JSON. Contoh: `{"action":"login","username":"admin","password":"admin123"}` menghasilkan token. Untuk aksi selanjutnya, kirim properti `token` dan `action`. Aksi yang disediakan: `getSPPT`, `getSPPTById`, `getSPPTByPenarik`, `saveSPPT`, `updateSPPT`, `deleteSPPT`, `savePenarikan`, `getPenarikan`, `ajukanPindah`, `getPermintaanPindah`, `setujuiPindah`, `tolakPindah`, `getDashboardAdmin`, `getDashboardPenarik`, `getLaporan`, `importSPPT`, `exportSPPT`, `saveUser`, `listUsers`.

Contoh objek untuk `saveSPPT` (nama properti dapat huruf besar/kecil):

```json
{"action":"saveSPPT","token":"TOKEN","data":{"TAHUN_PAJAK":"2026","NOP":"33.07.010.001.0001.0","NAMA_WAJIB_PAJAK":"SUKIRNO","NAMA_YANG_HARUS_DITARIK":"SITI","ID_PENARIK":"P1","PAJAK_TERUTANG":125000}}
```

Backend menyimpan hash SHA-256 password, memeriksa role dari sesi server (tidak percaya role dari browser), membatasi akses penarik ke tugasnya, menolak duplikat kombinasi tahun pajak + NOP, dan mengunci pembaruan agar pemindahan mengubah baris SPPT yang sama. URL Web App dapat diuji dengan `doGet` untuk memeriksa status inisialisasi.

## Catatan penggunaan

- CSV impor menggunakan header seperti `TAHUN_PAJAK,NOP,NAMA_WAJIB_PAJAK,NAMA_YANG_HARUS_DITARIK,ALAMAT,RT,RW,DUSUN,PAJAK_TERUTANG`. Ekspor CSV menggunakan struktur kolom SPPT.
- Data SPPT mendukung `NO_PERSIL`, `LOKASI_BLOK`, `JUMLAH_PEMILIK`, dan `JUMLAH_TERBAYAR`; catatan penarikan menyimpan `JUMLAH_DIBAYAR`. Setelah mengganti Code.gs, jalankan `setup()` lagi agar header baru ditambahkan pada sheet tanpa menghapus kolom/data lama. Pembayaran dicatat gabungan per SPPT; tiap transaksi menambah total pembayaran, status menjadi `Dibayar Sebagian` sampai lunas, dan laporan menampilkan total dibayar serta sisa tagihan. Pembagian otomatis menunjukkan nominal bagian tiap pemilik; pembayarannya tetap dicatat sebagai satu nilai gabungan untuk SPPT.
- Unggah foto bukti ke Google Drive belum diimplementasikan; aplikasi hanya mencatat nama berkas pada demo.
- Untuk beban data besar, tambahkan paginasi dan unggah bukti langsung ke Drive sebelum operasional; endpoint saat ini membaca sheet ke memori untuk menyusun hasil.
- Mode demo tidak cocok untuk data identitas pajak nyata karena kredensial contoh dan localStorage bukan penyimpanan terpusat yang aman.
