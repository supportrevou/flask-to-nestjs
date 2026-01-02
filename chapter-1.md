# 📘 Bab 1: Perubahan Mindset (The Paradigm Shift)

> **Quote of the Chapter:**
> *"Tantangan terbesar pindah dari Flask ke NestJS bukanlah sintaksnya, melainkan filosofinya."*

Selamat datang di perjalanan migrasi ke NestJS!

Jika kamu terbiasa dengan Flask, minggu pertama menggunakan NestJS mungkin akan terasa sedikit mengejutkan. Rasanya seperti pindah dari rumah pribadi yang bebas direnovasi sesuka hati (Flask), ke sebuah apartemen modern yang memiliki aturan tata letak yang ketat (NestJS).

Di Flask, kamu memegang kendali penuh. Kamu bebas menentukan lokasi file database, struktur folder, hingga library yang ingin digunakan. Fleksibilitas ini menyenangkan, tapi juga menantang.

Di NestJS, kamu masuk ke dalam kerangka kerja yang sudah matang. Struktur dasarnya sudah ditentukan. Tugas kamu bukan lagi membangun pondasi dari nol, melainkan mengisi bagian-bagian yang sudah disediakan dengan logika bisnis aplikasi.

Bab ini membahas **tiga perubahan pola pikir utama** agar transisimu berjalan mulus.

---

## 1.1. Dari "Kebebasan Penuh" ke "Struktur Baku"

Perbedaan paling mencolok adalah filosofi dasar framework: **Micro-framework (Flask)** vs **Opinionated Framework (NestJS)**.

### 🐍 Flask: Fleksibilitas Tinggi

Flask memberikan kanvas kosong. Kamu bisa membuat aplikasi dalam satu file saja, atau memecahnya menjadi ratusan file dengan gaya arsitektur sesukamu.

* **Kelebihan:** Sangat cepat untuk membuat *prototype* atau MVP.
* **Tantangan:** Tidak ada standar baku. Jika bergabung ke proyek orang lain, kamu perlu waktu untuk mempelajari "gaya" struktur folder mereka.

### 🦁 NestJS: Konvensi di atas Konfigurasi

NestJS menyediakan struktur yang jelas dan seragam (terinspirasi dari Angular).

* **Routing** punya tempat sendiri (Controller).
* **Logika Bisnis** punya tempat sendiri (Service).
* **Konfigurasi** punya tempat sendiri (Module).
* **Kelebihan:** **Rapi dan Terprediksi**. Kamu bisa pindah ke tim NestJS manapun, dan strukturnya pasti sama. Kamu akan langsung tahu di mana mencari kode *User Registration*.

> **Analogi:**
> * **Flask:** Kamu merakit sendiri komponen-komponennya (seperti LEGO curah).
> * **NestJS:** Kamu mengikuti panduan perakitan yang sudah disediakan agar hasilnya kokoh dan standar (seperti LEGO Technic).
> 
> 

---

## 1.2. Dari Functional ke Object-Oriented (OOP)

Developer Python/Flask biasanya sangat nyaman menggunakan fungsi (`def`). Di NestJS, **hampir semuanya adalah Class**. NestJS sangat bergantung pada konsep *Object Oriented Programming* (OOP).

### Perbandingan Kode

**🐍 Flask (Gaya Fungsional)**
Fungsi berdiri sendiri dan didekorasi langsung.

```python
@app.route('/kucing')
def cari_kucing():
    return "Meong"

```

**🦁 NestJS (Gaya OOP)**
Route (method) harus hidup di dalam sebuah Class Controller.

```typescript
@Controller('kucing')
export class KucingController {
  
  @Get()
  cariKucing() {
    return 'Meong';
  }
}

```

💡 **Mindset Shift:**
Hindari berpikir: *"Saya akan buat fungsi untuk login."*
Mulailah berpikir: *"Saya akan buat **Class AuthService** yang memiliki **method login**."*

---

## 1.3. Dependency Injection (DI): Konsep Paling Penting

Ini sering menjadi bagian yang paling membingungkan bagi developer Python.

### ❌ Cara Lama (Direct Import)

Di Python, kita terbiasa mengimpor dependensi secara langsung.

```python
# user_routes.py
from database import db  # <--- Hard dependency (diimpor langsung)

def get_user():
    return db.query(...)

```

*Masalah:* Kode sulit di-test (unit testing) karena terikat erat dengan file `database.py`.

### ✅ Cara Baru (Dependency Injection)

Di NestJS, kita **"meminta"** dependensi melalui `constructor`. Framework yang akan menyediakannya untuk kita.

```typescript
// user.controller.ts
@Controller('users')
export class UserController {
  // "Nest, tolong sediakan UserService untuk saya gunakan."
  constructor(private userService: UserService) {} 

  @Get()
  getUser() {
    // Framework sudah menyiapkan userService, tinggal pakai.
    return this.userService.findAll();
  }
}

```

> **Analogi Restoran:**
> Bayangkan NestJS sebagai pelayan restoran.
> 1. Kamu (Controller) duduk di meja.
> 2. Kamu bilang: "Saya butuh sendok (Service)."
> 3. Pelayan mengambilkan sendok dan memberikannya ke tanganmu.
> 
> 
> Kamu tidak perlu lari ke dapur mencari sendok sendiri. Ini membuat kodemu bersih dan mudah di-test.

---

## 1.4. Type Safety: Mencegah Error Sejak Dini

* **Python (Dynamic Typing):** Error seringkali baru ketahuan saat aplikasi dijalankan (*Runtime Error*).
* **TypeScript (Static Typing):** Error ketahuan saat kamu sedang menulis kode (*Compile Time Error*).

**Saran Penting:**
Jangan malas mendefinisikan Tipe Data atau DTO (*Data Transfer Object*).

* 🔴 **Hindari:** Menggunakan `any`. Ini membuat TypeScript kehilangan fungsinya.
* 🟢 **Lakukan:** Definisikan bentuk datanya, misal `createProductDto`.

Meskipun terlihat lebih banyak yang harus diketik, ini akan menyelamatkanmu dari *debugging* berjam-jam karena salah tipe data.

---

## 📝 Ringkasan Bab 1

| Aspek | 🐍 Flask (Kebiasaan Lama) | 🦁 NestJS (Kebiasaan Baru) |
| --- | --- | --- |
| **Filosofi** | Bebas, tentukan sendiri. | Terstruktur, ikuti pola framework. |
| **Gaya Koding** | Dominan fungsi (`def`). | Dominan Class (`class`) & OOP. |
| **Manajemen Library** | Import langsung di file. | Injeksi melalui `constructor`. |
| **Pengecekan Error** | Saat aplikasi berjalan (*Runtime*). | Saat menulis kode (*Compile time*). |

---

**Langkah Selanjutnya:**
Sekarang setelah mindset kita selaras, mari kita masuk ke **Bab 2** untuk menerjemahkan istilah-istilah teknis di Flask (seperti Blueprints, Marshmallow, SQLAlchemy) ke padanannya di NestJS.
