# 📘 Bab 7: Middleware, Guards, & Interceptors (The Bodyguards)

> **Quote of the Chapter:**
> *"Di Flask, kamu menggunakan `@login_required` dan `before_request` untuk segala hal. Di NestJS, setiap tugas punya 'petugas' spesifiknya sendiri."*

Salah satu pertanyaan terbesar saat pindah ke NestJS: *"Di mana saya menaruh logika cek token login? Di Middleware? Di Guard? Atau di Interceptor?"*

Di Flask, batasannya kabur. Kamu bisa melakukan validasi user di `before_request`, di dalam route, atau pakai decorator kustom.

Di NestJS, ada **Siklus Hidup Request (Request Lifecycle)** yang ketat. Bayangkan request user seperti tamu yang masuk ke gedung perkantoran. Mereka harus melewati beberapa pos pemeriksaan:

1. **Middleware:** Resepsionis (Mencatat log, mengubah header).
2. **Guards:** Satpam (Cek ID Card/Auth).
3. **Interceptors:** Bagian Packing (Membungkus data sebelum keluar).

Mari kita bedah satu per satu.

---

## 7.1. Middleware: Si Resepsionis

Fungsinya mirip dengan `before_request` di Flask. Middleware berjalan **paling awal**, bahkan sebelum NestJS tahu route mana yang akan dituju.

Cocok untuk: Logger, Helmet (Security Headers), CORS, atau memproses body parser.

### 🐍 Flask (`before_request`)

```python
@app.before_request
def log_request():
    print(f"Ada request masuk ke: {request.path}")

```

### 🦁 NestJS (Middleware)

Kita membuat class yang mengimplementasikan `NestMiddleware`.

```typescript
// logger.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`Ada request masuk ke: ${req.originalUrl}`);
    next(); // Lanjut ke pos berikutnya (WAJIB DIPANGGIL)
  }
}

```

*Cara pasangnya:* Middleware didaftarkan di `Module` menggunakan method `configure()`.

---

## 7.2. Guards: Si Satpam (Auth) 👮

Ini adalah pengganti `@login_required`. Tugas Guard hanya satu: Mengembalikan `true` (boleh lewat) atau `false` (DILARANG!).

Guard berjalan **setelah** Middleware tapi **sebelum** Controller.

### 🐍 Flask (`@decorator`)

```python
def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if not request.headers.get('Authorization'):
            abort(401) # Tendang
        return f(*args, **kwargs)
    return decorated_function

@app.route('/admin')
@login_required # Pasang di sini
def admin_page(): ...

```

### 🦁 NestJS (Guard)

Kita membuat class dengan interface `CanActivate`.

```typescript
// auth.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    // Logika cek token sederhana
    const token = request.headers['authorization'];
    
    return token === 'rahasia123'; // Return true jika valid, false jika tidak
  }
}

```

**Cara Pakainya:**
Pasang decorator `@UseGuards` di Controller.

```typescript
@Controller('admin')
@UseGuards(AuthGuard) // <--- Pasang satpam di sini
export class AdminController {
    // ...
}

```

---

## 7.3. Interceptors: Si Bagian Packing 🎁

Pernahkah kamu ingin mengubah response sebelum dikirim ke user? Misalnya, membungkus semua data dalam format `{ "data": ... }` atau menyembunyikan field `password`.

Di Flask, kamu mungkin menggunakan `after_request`. Di NestJS, kita pakai **Interceptor**.

### 🐍 Flask (`after_request`)

```python
@app.after_request
def wrap_response(response):
    # Modifikasi JSON response secara manual
    # Agak ribet karena harus parse JSON dulu, ubah, lalu dump lagi
    return response

```

### 🦁 NestJS (Interceptor)

Interceptor bisa "mencegat" request saat masuk **DAN** saat keluar (response). Ini sangat powerful.

Contoh: Kita ingin semua response API otomatis dibungkus object `data`.

```typescript
// response.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { map } from 'rxjs/operators';

@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    // 'next.handle()' adalah proses menjalankan Controller
    // '.pipe(map(...))' adalah proses memodifikasi hasil return Controller
    
    return next.handle().pipe(
      map(data => ({ 
        statusCode: 200,
        data: data,
        timestamp: new Date()
      }))
    );
  }
}

```

**Hasilnya:**
Controller kamu me-return: `['Budi', 'Siti']`.
Yang diterima User:

```json
{
  "statusCode": 200,
  "data": ["Budi", "Siti"],
  "timestamp": "2023-10-27T10:00:00Z"
}

```

Controller tetap bersih, format response terstandarisasi otomatis!

---

## 7.4. Tabel Pembagian Tugas

Biar tidak bingung kapan pakai yang mana, ingat tabel ini:

| Tugas | Fitur Flask | Fitur NestJS | Kenapa? |
| --- | --- | --- | --- |
| **Logging Request** | `before_request` | **Middleware** | Karena perlu dijalankan paling awal. |
| **Cek Login/Role** | Decorator (`@login_required`) | **Guard** | Karena logic-nya cuma Butuh Ya/Tidak (Boolean). |
| **Validasi Input** | Manual / Marshmallow | **Pipe** (Bab 5) | Karena spesifik memeriksa argumen fungsi. |
| **Ubah Response** | `after_request` | **Interceptor** | Karena bisa memanipulasi data hasil return. |
| **Handle Error** | `@app.errorhandler` | **Exception Filter** | (Bonus) Menangkap error global. |

---

## 📝 Ringkasan Bab 7

1. **Jangan Campur Aduk:** NestJS memisahkan logika berdasarkan *kapan* logika itu dijalankan.
2. **Middleware:** Pintu gerbang utama (Logging, CORS).
3. **Guard:** Pos keamanan (Authentication & Authorization). Jika Guard bilang `false`, request langsung ditolak (403 Forbidden).
4. **Interceptor:** Jalur keluar. Tempat kamu membedah dan memodifikasi hasil akhir sebelum dikirim ke user.

---

**Langkah Selanjutnya:**
Kita sudah membahas hampir semua aspek teknis koding. Tapi ada satu hal non-teknis yang sangat penting dalam pengembangan software modern: **Testing**.
Developer Flask biasanya menggunakan `pytest`. Di **Bab 8**, kita akan melihat bagaimana NestJS memanjakan developernya dengan **Jest** yang sudah terintegrasi sejak awal instalasi.
