# 📘 Bab 4: Membedah Kode: Flask vs NestJS (Side-by-Side)

> **Tujuan Bab Ini:**
> Kita akan membuat satu fitur sederhana: **"Mencari User berdasarkan ID"**. Kita akan membandingkan bagaimana kode ini ditulis di Flask, dan bagaimana kode yang sama dipecah di NestJS.

Teori sudah cukup. Mari kita lihat kodenya.

Skenario kita hari ini: Kita ingin membuat endpoint `GET /users/:id`. Jika user ditemukan, kembalikan datanya. Jika tidak, kembalikan error.

---

## 4.1. Cara Flask (The All-in-One Way)

Di Flask, biasanya kita menulis *routing* dan *logika* di tempat yang sama (atau minimal berdekatan).

```python
# user_routes.py
from flask import Blueprint, jsonify

user_bp = Blueprint('user', __name__)

# Data pura-pura (Mock Data)
users = [
    {'id': 1, 'name': 'Budi'},
    {'id': 2, 'name': 'Siti'}
]

@user_bp.route('/users/<int:id>', methods=['GET'])
def get_user(id):
    # 1. Logika Bisnis (Mencari data)
    user = next((u for u in users if u['id'] == id), None)
    
    # 2. Logika Response
    if not user:
        return jsonify({'message': 'User not found'}), 404
    
    return jsonify(user)

```

**Analisis Flask:**

* Sederhana dan ringkas.
* Parameter URL didefinisikan dengan `<int:id>`.
* Kita harus manual membungkus return dengan `jsonify`.

---

## 4.2. Cara NestJS (The Separated Way)

Di NestJS, kode di atas **HARUS** dipecah menjadi dua file utama: **Service** (Logika) dan **Controller** (Routing).

Mengapa? Agar *Single Responsibility Principle* terjaga.

### Langkah 1: Buat Service (Si Koki) 🍳

Tugas Service hanya satu: Mengolah data. Dia tidak peduli soal HTTP, JSON, atau status code.

```typescript
// users.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';

@Injectable()
export class UsersService {
  private users = [
    { id: 1, name: 'Budi' },
    { id: 2, name: 'Siti' },
  ];

  findOne(id: number) {
    const user = this.users.find((u) => u.id === id);
    
    // NestJS punya Error Helper bawaan
    if (!user) {
      throw new NotFoundException('User not found'); 
    }
    
    return user;
  }
}

```

> **Perhatikan:** Kita menggunakan decorator `@Injectable()`. Ini memberitahu NestJS: *"Hei, class ini bisa di-inject ke class lain lho!"*

### Langkah 2: Buat Controller (Si Pelayan) 🤵

Tugas Controller hanya satu: Menerima request, panggil Service, lalu kembalikan hasil.

```typescript
// users.controller.ts
import { Controller, Get, Param, ParseIntPipe } from '@nestjs/common';
import { UsersService } from './users.service';

@Controller('users')
export class UsersController {
  
  // DEPENDENCY INJECTION TERJADI DI SINI 👇
  constructor(private readonly usersService: UsersService) {}

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    // Panggil si Koki (Service)
    return this.usersService.findOne(id);
  }
}

```

**Analisis NestJS:**

1. **Dependency Injection:** Perhatikan `constructor`. Kita tidak melakukan `new UsersService()`. Kita hanya mendefinisikannya di parameter, dan NestJS yang akan menyiapkannya.
2. **Decorators:**
* `@Get(':id')`: Sama seperti `<int:id>` di Flask.
* `@Param('id')`: Cara mengambil nilai `id` dari URL.
* `ParseIntPipe`: Otomatis mengubah string "1" dari URL menjadi number `1`. Di Flask, ini mirip `<int:id>`.


3. **Return Value:** Kita tidak perlu `jsonify()`. NestJS otomatis mengubah object/array menjadi JSON saat dikirim ke client.

---

## 4.3. Tabel Translasi Sintaks

Ini adalah panduan cepat untuk menerjemahkan sintaks yang sering kamu pakai.

| Fitur | 🐍 Kode Flask | 🦁 Kode NestJS |
| --- | --- | --- |
| **Definisi Route** | `@app.route('/users')` | `@Controller('users')` |
| **HTTP Method** | `methods=['POST']` | `@Post()` |
| **URL Variable** | `/users/<id>` | `/users/:id` |
| **Ambil Param** | `def get_user(id):` | `findOne(@Param('id') id: string)` |
| **Ambil Body** | `request.json` | `@Body() createDto: CreateUserDto` |
| **Ambil Query** | `request.args.get('page')` | `@Query('page') page: number` |
| **Status Code** | `return ..., 201` | `@HttpCode(201)` (atau manual via `Res`) |
| **Return JSON** | `jsonify(data)` | `return data` (Otomatis) |
| **Error 404** | `return ..., 404` | `throw new NotFoundException()` |

---

## 4.4. "Kenapa NestJS Harus Sepanjang Itu?"

Kamu mungkin bertanya: *"Di Flask cuma 10 baris, di NestJS jadi 2 file dan 20 baris. Apa untungnya?"*

Bayangkan skenarionya berubah. Sekarang, selain dari HTTP Endpoint, kamu juga ingin memanggil fungsi `findOne` tersebut lewat **CLI Command** (Terminal) atau lewat **Cron Job** (Jadwal otomatis).

* **Di Flask (Cara Lama):** Kamu akan kesulitan karena logikanya tersangkut di dalam fungsi `@app.route`. Kamu harus refactor kode dulu.
* **Di NestJS:** `UsersService` berdiri sendiri.
* Untuk Web? Panggil dari `UsersController`.
* Untuk CLI? Panggil dari `ConsoleService`.
* Untuk Cron? Panggil dari `CronService`.



Logika bisnis (`UsersService`) bisa dipakai ulang di mana saja tanpa terikat pada HTTP Request. Inilah yang disebut **Scalability**.

---

## 📝 Ringkasan Bab 4

1. **Pemisahan Tugas:** Jangan campur urusan HTTP (Controller) dengan urusan Logika (Service).
2. **Pipes:** NestJS punya alat bantu seperti `ParseIntPipe` untuk mengubah tipe data URL secara otomatis.
3. **Otomatisasi JSON:** Kamu tidak perlu mengubah data jadi JSON secara manual, cukup `return object`.
4. **Error Handling:** Gunakan `Exception` bawaan NestJS (seperti `NotFoundException`) untuk melempar error HTTP standar.

---

**Langkah Selanjutnya:**
Di kode NestJS di atas, kita masih menggunakan *mock data* (array biasa). Di dunia nyata, data datang dari user dalam format JSON. Bagaimana kita memvalidasinya agar user tidak mengirim data sampah?

Di **Bab 5**, kita akan membahas **DTO (Data Transfer Object)**, salah satu fitur paling *powerful* di NestJS yang akan membuat pengguna Flask iri.
