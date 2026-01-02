# 📑 Executive Summary: Panduan Migrasi Flask ke NestJS

## 1. Pendahuluan

Dokumen ini merangkum strategi transisi bagi software engineer yang berpindah dari ekosistem **Python/Flask** ke **TypeScript/NestJS**. Migrasi ini bukan sekadar perubahan sintaks bahasa, melainkan perubahan fundamental dalam **filosofi arsitektur**.

Flask menawarkan kebebasan (*Micro-framework*), sementara NestJS menawarkan struktur dan standarisasi (*Opinionated Framework*). Tujuan utama migrasi ini biasanya adalah untuk mencapai **skalabilitas**, **keteraturan kode (maintainability)**, dan **keamanan tipe data (type safety)** pada aplikasi berskala besar.

---

## 2. Pergeseran Paradigma (Core Paradigm Shifts)

Terdapat tiga perubahan mentalitas utama yang harus diadopsi:

1. **Struktur vs. Kebebasan:**
* **Flask:** Mengandalkan konvensi tim (struktur folder bebas).
* **NestJS:** Mengandalkan konvensi framework (Modular Architecture). Kode dikelompokkan berdasarkan *Domain/Fitur* (misal: UserModule, OrderModule), bukan jenis file.


2. **OOP vs. Fungsional:**
* **Flask:** Dominan menggunakan fungsi (`def`) dan decorator lepas.
* **NestJS:** Wajib menggunakan `Class`, `Interface`, dan prinsip SOLID. Semua komponen (Controller, Service) adalah Class.


3. **Dependency Injection (DI):**
* **Flask:** Mengimpor dependensi secara langsung (`from db import session`).
* **NestJS:** "Meminta" dependensi melalui *Constructor Injection*. Hal ini membuat kode terlepas dari ketergantungan erat (*loosely coupled*) dan sangat mudah untuk diuji (*testable*).



---

## 3. Peta Translasi Konsep (The Rosetta Stone)

Tabel berikut memetakan komponen utama Flask ke padanannya di NestJS:

| Komponen | 🐍 Flask (Python) | 🦁 NestJS (TypeScript) | Deskripsi |
| --- | --- | --- | --- |
| **Routing** | `@app.route` | `@Controller` | Menangani request HTTP. |
| **Logic** | View Function | `Service` | Menangani logika bisnis. |
| **Grouping** | `Blueprint` | `Module` | Mengelompokkan fitur. |
| **Validation** | Marshmallow / Pydantic | `DTO` + `class-validator` | Validasi input data. |
| **Database** | SQLAlchemy (`db.Model`) | TypeORM (`@Entity`) | Definisi tabel database. |
| **DB Query** | `User.query.all()` | `repository.find()` | Akses data (Repository Pattern). |
| **Auth/Guard** | `@login_required` | `Guards` | Proteksi endpoint. |
| **Testing** | `pytest` | `Jest` | Unit & Integration Testing. |
| **Entry Point** | `app.py` | `main.ts` | Titik awal aplikasi. |

---

## 4. Keuntungan Strategis NestJS

Mengapa upaya migrasi ini layak dilakukan?

1. **Type Safety (TypeScript):** Mencegah *runtime error* yang umum terjadi di Python (seperti `NoneType object has no attribute`). Error terdeteksi saat *coding*, bukan saat aplikasi berjalan.
2. **Arsitektur Standar:** Struktur NestJS yang baku memudahkan *onboarding* anggota tim baru. Developer tidak perlu menebak di mana logika diletakkan.
3. **Ekosistem Terintegrasi:** Fitur seperti Testing (Jest), Validasi, dan Dokumentasi (Swagger) sudah terintegrasi secara *native*, mengurangi kebutuhan konfigurasi library pihak ketiga.
4. **Skalabilitas:** Pemisahan tegas antara Controller dan Service memudahkan pengembangan aplikasi menjadi microservices di masa depan.

---

## 5. Catatan Deployment & Production

Perbedaan signifikan terjadi pada fase *deployment*:

* **Proses Build:** NestJS (TypeScript) memerlukan langkah **kompilasi** (`npm run build`) sebelum dijalankan. Flask (Python) tidak.
* **Artifact:** Yang dijalankan di server production adalah folder `dist/` (hasil kompilasi JavaScript), bukan folder `src/`.
* **Process Manager:** Menggantikan **Gunicorn** (Python) dengan **PM2** (Node.js) untuk manajemen proses dan *clustering*.

## 6. Kesimpulan

Transisi ke NestJS memiliki kurva pembelajaran di awal, terutama terkait konsep *Dependency Injection* dan *Type System*. Namun, investasi waktu ini terbayar dengan menghasilkan basis kode yang lebih **kokoh**, **mudah dipelihara**, dan **minim bug** untuk jangka panjang.

Panduan ini (Bab 1-9) dirancang untuk memandu pengembang langkah demi langkah melalui transisi tersebut dengan hambatan seminimal mungkin.
