# 📘 Bab 3: Arsitektur & Struktur Project (The Blueprint)

> **Quote of the Chapter:**
> *"Di Flask, kamu mengelompokkan file berdasarkan fungsinya (Routes, Models). Di NestJS, kamu mengelompokkan file berdasarkan fiturnya (Users, Products)."*

Saat kamu pertama kali menjalankan perintah `nest new my-project`, kamu akan diberikan sekumpulan file dan folder. Jangan panik.

Di Flask, struktur folder seringkali "datar" atau dikelompokkan berdasarkan *jenis file* (folder `templates`, folder `static`, folder `views`).

Di NestJS, struktur folder dikelompokkan berdasarkan **Domain** atau **Fitur**. Ini disebut dengan **Modular Architecture**.

Mari kita bedah anatominya.

---

## 3.1. Struktur Folder: Flask vs NestJS

Mari kita bandingkan struktur project untuk aplikasi toko online sederhana.

### 🐍 Gaya Flask (Layer-based)

Biasanya dipisah berdasarkan *layer* teknis.

```text
my-flask-app/
├── app.py
├── models.py          <-- Semua model DB ada di sini
├── routes.py          <-- Semua route ada di sini
├── services.py        <-- Semua logic ada di sini
└── schemas.py

```

*Masalah:* Ketika aplikasi membesar, `models.py` akan menjadi sangat panjang dan sulit dikelola.

### 🦁 Gaya NestJS (Feature-based)

Dipisah berdasarkan fitur bisnis. Setiap fitur punya "rumah" sendiri.

```text
my-nest-app/
├── src/
│   ├── app.module.ts       <-- Akar aplikasi
│   ├── main.ts             <-- Entry point
│   │
│   ├── users/              <-- MODUL USER (Semua ttg user di sini)
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   ├── users.module.ts
│   │   └── dto/
│   │
│   └── products/           <-- MODUL PRODUK (Semua ttg produk di sini)
│       ├── products.controller.ts
│       ├── products.service.ts
│       └── products.module.ts

```

*Keuntungan:* Jika ada masalah di fitur "Produk", kamu hanya perlu fokus ke folder `products`. Kamu tidak perlu menyentuh folder `users`.

---

## 3.2. Anatomi Sebuah Module

Di Bab 2 kita sudah menyinggung bahwa Module adalah pengelompokan fitur. Tapi bagaimana cara kerjanya?

Bayangkan sebuah Module (`users.module.ts`) sebagai sebuah **Kotak Perkakas (Toolbox)**. Agar kotak ini berguna, kita harus mendaftarkan isinya.

```typescript
// users.module.ts
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  controllers: [UsersController], // Siapa yang menerima request HTTP?
  providers: [UsersService],      // Siapa yang mengerjakan logika bisnis?
  exports: [UsersService]         // Apa yang boleh dipakai oleh module lain?
})
export class UsersModule {}

```

### Penjelasan 3 Komponen Utama:

1. **Controllers:** "Resepsionis". Pintu depan untuk menerima tamu (request).
2. **Providers:** "Pekerja". Di sinilah Service, Repository, atau Helper diletakkan.
3. **Exports:** "Barang Dagangan". Jika module lain (misal: AuthModule) butuh data user, maka `UsersService` harus di-export dari sini.

> ⚠️ **Kesalahan Umum Eks-Flask:**
> Lupa mendaftarkan Service ke dalam `providers`.
> Jika kamu membuat `UsersService` tapi lupa memasukkannya ke `providers: [UsersService]`, NestJS akan error: *"Nest can't resolve dependencies"*.
> Di Flask, cukup import file-nya, kode jalan. Di NestJS, kamu harus **mendaftarkannya**.

---

## 3.3. Menggabungkan Semuanya: The Root Module

Semua module kecil (Users, Products) harus lapor ke bos besar: **AppModule**.

NestJS membangun aplikasi dalam bentuk struktur pohon (*tree*). `AppModule` adalah batangnya, dan module lainnya adalah cabangnya.

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { UsersModule } from './users/users.module';
import { ProductsModule } from './products/products.module';

@Module({
  imports: [       // <--- DAFTARKAN SEMUA MODUL DI SINI
    UsersModule,
    ProductsModule,
  ],
})
export class AppModule {}

```

Jika kamu membuat module baru tapi lupa memasukkannya ke `imports` di `AppModule`, fitur tersebut tidak akan pernah aktif.

---

## 3.4. Nest CLI: Sahabat Terbaikmu

Sebagai pengguna Flask, kamu mungkin terbiasa membuat file secara manual: *klik kanan -> New File -> `routes.py*`.

Di NestJS, **JANGAN BUAT FILE MANUAL**. Gunakan CLI (*Command Line Interface*).

Kenapa? Karena CLI tidak hanya membuatkan file, tapi juga **otomatis menyambungkan kabel-kabelnya**.

### Contoh Magic CLI:

Jika kamu ingin membuat fitur baru bernama `orders`.

Jalankan perintah ini di terminal:

```bash
nest generate resource orders

```

Apa yang dilakukan NestJS secara otomatis?

1. 📂 Membuat folder `src/orders`.
2. 📄 Membuat file Controller, Service, Module, DTO, dan Entity.
3. 🧪 Membuat file testing `.spec.ts`.
4. 🔌 **Otomatis mengedit** `app.module.ts` dan mengimpor `OrdersModule`.

Dalam 1 detik, kamu sudah punya kerangka fitur CRUD (Create, Read, Update, Delete) yang siap pakai. Di Flask, ini butuh waktu 15-30 menit *setup* manual.

---

## 📝 Ringkasan Bab 3

| Konsep | Penjelasan Simpel |
| --- | --- |
| **Feature-based Structure** | Kelompokkan file berdasarkan topik (User, Order), bukan jenis file. |
| **Module (`@Module`)** | Tempat mendaftarkan Controller dan Service agar saling kenal. |
| **AppModule** | Induk dari semua module. Tempat semua fitur disatukan. |
| **Nest CLI** | Alat ajaib untuk membuat file dan struktur otomatis (`nest g resource ...`). |

---

**Langkah Selanjutnya:**
Sekarang kita sudah punya struktur project yang rapi. Tapi bagaimana cara Controller dan Service berbicara satu sama lain? Di **Bab 4**, kita akan masuk ke bagian paling menarik: **Hands-on Coding**. Kita akan menulis kode untuk membuat API CRUD pertama kita.
