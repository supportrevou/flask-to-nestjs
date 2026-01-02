# 📘 Bab 5: Validasi Data & DTO (The Gatekeeper)

> **Quote of the Chapter:**
> *"Jangan biarkan data sampah masuk ke aplikasimu. Di Flask, kamu menyaring sampah secara manual. Di NestJS, kamu menyewa satpam untuk melakukannya."*

Dalam pengembangan API, prinsip utamanya adalah: **Jangan pernah percaya input dari user.**

Di Python/Flask, kita sering berurusan dengan data JSON mentah. Kita harus memeriksa satu per satu: *"Apakah field 'email' ada? Apakah formatnya benar? Apakah 'age' itu angka?"*

Di NestJS, kita menggunakan konsep bernama **DTO (Data Transfer Object)** yang dikombinasikan dengan library `class-validator`. Hasilnya? Kode controller yang bersih tanpa `if-else` untuk validasi.

Mari kita lihat bedanya.

---

## 5.1. Masalah: Validasi Manual di Flask 🐍

Bayangkan kita membuat fitur **Register User**. User harus mengirim `email` (wajib, format email) dan `password` (wajib, min 6 karakter).

Di Flask, tanpa library tambahan (seperti Pydantic/Marshmallow), kodenya akan terlihat seperti ini:

```python
# flask_app.py
@app.route('/register', methods=['POST'])
def register():
    data = request.json
    
    # 1. Cek apakah field ada
    if 'email' not in data or 'password' not in data:
        return jsonify({'error': 'Missing fields'}), 400
    
    # 2. Cek tipe data dan format
    if not isinstance(data['email'], str) or '@' not in data['email']:
        return jsonify({'error': 'Invalid email'}), 400
        
    if len(data['password']) < 6:
        return jsonify({'error': 'Password too short'}), 400
        
    # 3. Barulah masuk logika bisnis...
    return create_user(data)

```

**Masalahnya:**
Controller kita penuh dengan kode defensif (pengecekan error). Logika bisnis inti malah tertimbun oleh validasi.

---

## 5.2. Solusi: DTO di NestJS 🦁

Di NestJS, kita memisahkan definisi "Bentuk Data" ke dalam file terpisah yang disebut **DTO**.

Pikirkan DTO sebagai **Formulir Pendaftaran**. Sebelum masuk ke ruangan Controller, user harus mengisi formulir ini dengan benar.

### Langkah 1: Buat File DTO

Kita menggunakan *decorator* untuk menetapkan aturan. Ini mirip dengan **Pydantic** di Python modern.

```typescript
// create-user.dto.ts
import { IsEmail, IsNotEmpty, MinLength, IsString } from 'class-validator';

export class CreateUserDto {
  @IsEmail() // Harus format email
  @IsNotEmpty() // Tidak boleh kosong
  email: string;

  @IsString()
  @MinLength(6, { message: 'Password dikit banget, minimal 6 huruf dong' })
  password: string;
}

```

> **Catatan:** Kamu perlu menginstall library-nya dulu:
> `npm install class-validator class-transformer`

### Langkah 2: Pasang di Controller

Sekarang, perhatikan betapa bersihnya Controller kita.

```typescript
// users.controller.ts
import { Body, Controller, Post } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  
  @Post('register')
  // Perhatikan tipe data 'body' adalah CreateUserDto 👇
  register(@Body() body: CreateUserDto) {
    
    // TIDAK PERLU VALIDASI MANUAL!
    // Jika kode sampai di baris ini, berarti data SUDAH PASTI VALID.
    // Email pasti format email, password pasti >= 6 char.
    
    return { message: 'User valid', data: body };
  }
}

```

---

## 5.3. The Magic Switch: ValidationPipe 🪄

Ada satu hal penting. Secara default, NestJS **tidak** mengaktifkan validasi otomatis ini (karena alasan performa). Kita harus menyalakan "saklar"-nya sekali saja di `main.ts`.

Kita menggunakan sesuatu yang disebut **Pipe**. Pipe adalah kode yang berjalan saat data sedang "mengalir" dari Request menuju Controller.

```typescript
// main.ts
import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // 🔥 AKTIFKAN VALIDASI GLOBAL DI SINI
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true, // Fitur keren: Hapus field yang tidak ada di DTO
    forbidNonWhitelisted: true // Atau, tolak request jika ada field asing
  }));
  
  await app.listen(3000);
}
bootstrap();

```

### Apa itu `whitelist: true`?

Ini fitur keamanan favorit.
Jika user iseng mengirim data:

```json
{
  "email": "test@mail.com",
  "password": "123456",
  "isAdmin": true  // <--- User mencoba hack jadi admin
}

```

Karena field `isAdmin` tidak ada di definisi `CreateUserDto`, NestJS akan otomatis **menghapus** field tersebut sebelum sampai ke Controller. Aman!

---

## 5.4. Tabel Perbandingan Validasi

| Fitur | 🐍 Flask (Manual/Pydantic) | 🦁 NestJS (class-validator) |
| --- | --- | --- |
| **Definisi Schema** | Class Pydantic atau Marshmallow Schema. | Class TypeScript dengan Decorators. |
| **Lokasi Validasi** | Seringkali dipanggil manual di dalam route. | Otomatis dijalankan sebelum masuk Controller. |
| **Sanitasi Data** | Manual (misal `pop` field yang tidak diinginkan). | Otomatis via `whitelist: true`. |
| **Error Response** | Harus format return JSON manual. | Otomatis melempar `400 Bad Request` dengan pesan detail. |

---

## 📝 Ringkasan Bab 5

1. **DTO (Data Transfer Object):** Class sederhana untuk mendefinisikan bentuk data yang kita harapkan dari user.
2. **Decorators:** Gunakan `@IsEmail()`, `@IsString()`, `@MinLength()` untuk memberi aturan validasi tanpa menulis logika `if`.
3. **ValidationPipe:** "Satpam" global yang mencegat request. Jika data tidak sesuai DTO, request ditolak otomatis dengan status 400.
4. **Cleaner Code:** Controller kamu sekarang hanya fokus menerima data valid dan memanggil Service, bukan sibuk mengecek data.

---

**Langkah Selanjutnya:**
Data user sudah valid, tapi data itu masih hilang saat server dimatikan. Kita butuh database.
Di **Bab 6**, kita akan membahas **Database & TypeORM**. Bagaimana cara menghubungkan NestJS ke database (MySQL/PostgreSQL) tanpa menulis query SQL manual, mirip seperti SQLAlchemy di Flask.
