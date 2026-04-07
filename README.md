# 🥚 NdokkMu — Sistem Manajemen Distributor Telur

Sistem manajemen penjualan telur berbasis web dengan Firebase Firestore. Terdiri dari 3 file HTML:

- `index.html` — Dashboard publik + login
- `admin.html` — Panel admin (input kulak, jual, transfer, grafik FIFO)
- `agen.html` — Panel agen (input jual, return, grafik FIFO)

---

## 🔧 Setup Firebase Firestore

### 1. Firestore Rules
Buka Firebase Console → Firestore → Rules, lalu paste rules berikut:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Settings (public read, admin write)
    match /settings/{doc} {
      allow read: if true;
      allow write: if request.auth != null && 
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role in ['admin','superuser'];
    }

    // Users (read by authenticated, write by admin)
    match /users/{userId} {
      allow read: if true;
      allow write: if request.auth != null && 
        (request.auth.uid == userId || 
         get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role in ['admin','superuser']);
    }

    // Transaksi Kulak (admin only)
    match /transaksi_kulak/{doc} {
      allow read, write: if request.auth != null && 
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role in ['admin','superuser'];
    }

    // Transaksi Jual Admin
    match /transaksi_jual/{doc} {
      allow read, write: if request.auth != null && 
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role in ['admin','superuser'];
    }

    // Transaksi Transfer ke Agen
    match /transaksi_transfer/{doc} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && 
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role in ['admin','superuser'];
    }

    // Transaksi Jual Agen
    match /transaksi_jual_agen/{doc} {
      allow read: if request.auth != null && 
        (resource.data.agen_id == request.auth.uid || 
         get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role in ['admin','superuser']);
      allow write: if request.auth != null && 
        (request.resource.data.agen_id == request.auth.uid || 
         get(/databases/$(database)/documents/users/$(request.auth.uid)).data.role in ['admin','superuser']);
    }

    // Transaksi Return
    match /transaksi_return/{doc} {
      allow read: if request.auth != null;
      allow write: if request.auth != null;
    }
  }
}
```

### 2. Struktur Koleksi Firestore

```
firestore/
├── users/
│   └── {uid}/
│       ├── nama: string
│       ├── email: string
│       ├── role: "admin" | "superuser" | "agen"
│       ├── alamat: string
│       ├── whatsapp: string
│       └── lokasi: string (untuk admin)
│
├── settings/
│   └── admin/
│       ├── stok_kg: number
│       ├── nilai_stok: number
│       └── moving_average: number
│
├── transaksi_kulak/
│   └── {docId}/
│       ├── tanggal: string (YYYY-MM-DD)
│       ├── qty: number
│       ├── harga_beli: number
│       ├── catatan: string
│       └── created: timestamp
│
├── transaksi_jual/
│   └── {docId}/
│       ├── tanggal: string
│       ├── qty: number
│       ├── harga_jual: number
│       ├── catatan: string
│       ├── agen_id: string (opsional, jika transfer)
│       └── created: timestamp
│
├── transaksi_transfer/
│   └── {docId}/
│       ├── agen_id: string
│       ├── agen_nama: string
│       ├── tanggal: string
│       ├── qty: number
│       ├── harga_jual: number
│       ├── catatan: string
│       └── created: timestamp
│
├── transaksi_jual_agen/
│   └── {docId}/
│       ├── agen_id: string
│       ├── agen_nama: string
│       ├── tanggal: string
│       ├── qty: number
│       ├── harga_jual: number
│       ├── catatan: string
│       ├── is_return: bool (opsional)
│       └── created: timestamp
│
└── transaksi_return/
    └── {docId}/
        ├── agen_id: string
        ├── agen_nama: string
        ├── tanggal: string
        ├── qty: number
        ├── harga_return: number
        ├── catatan: string
        └── created: timestamp
```

### 3. Buat Akun Admin Pertama

1. Buka Firebase Console → Authentication → Add user
2. Masukkan email & password admin
3. Catat UID-nya
4. Buka Firestore → koleksi `users` → tambah dokumen dengan ID = UID tadi
5. Isi field:
   ```
   nama: "Nama Admin"
   email: "admin@example.com"
   role: "admin"
   alamat: "Alamat Gudang"
   whatsapp: "081234567890"
   ```

---

## 📂 Struktur File

```
ndokkmu/
├── index.html    → Dashboard publik + login
├── admin.html    → Panel admin
├── agen.html     → Panel agen
└── README.md     → Dokumentasi ini
```

---

## 🚀 Deploy ke GitHub Pages

1. Buat repository baru di GitHub
2. Upload semua file ke repo
3. Buka Settings → Pages → pilih branch `main` → folder `/root`
4. Akses via `https://username.github.io/nama-repo/`

> **Catatan:** GitHub Pages mendukung file statis HTML/JS. Firebase sudah di-load via CDN sehingga tidak perlu build step.

---

## 💡 Fitur Utama

| Fitur | Admin | Agen |
|-------|-------|------|
| Dashboard KPI | ✅ | ✅ |
| Input Kulak | ✅ | ❌ |
| Input Jual | ✅ | ✅ |
| Transfer ke Agen | ✅ | ❌ |
| Return ke Admin | ❌ | ✅ |
| FIFO Costing | ✅ | ✅ |
| Moving Average | ✅ | ✅ |
| Grafik Harga | ✅ | ✅ |
| Export Excel | ✅ | ✅ |
| Kelola Akun Agen | ✅ | ❌ |

---

*NdokkMu © 2025*
