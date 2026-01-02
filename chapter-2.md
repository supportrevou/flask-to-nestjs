# 📘 Bab 2: Kamus Istilah (The Rosetta Stone)

> **Tujuan Bab Ini:**
> Memetakan konsep yang sudah kamu kenal di Flask ke padanannya di NestJS. Kita tidak akan membahas *coding* mendalam dulu, tapi kita akan menyamakan persepsi istilah.

Seringkali, kebingungan saat belajar framework baru bukan karena konsepnya sulit, tapi karena **namanya berbeda**.

Kamu sudah tahu cara membuat API, menangani database, dan validasi data. Kamu hanya perlu tahu: *"Di NestJS, benda ini namanya apa?"*

Mari kita bedah satu per satu.

---

## 2.1. Titik Masuk Aplikasi (Entry Point)

Di mana aplikasi dimulai? Di mana server dijalankan?

### 🐍 Flask: `app.py`

Biasanya kamu memiliki satu file utama (sering disebut `app.py` atau `run.py`) tempat kamu menginisialisasi instans Flask.

```python
# app.py
from flask import Flask
app = Flask(__name__) # Inisialisasi

if __name__ == '__main__':
    app.run() # Jalankan server

```

### 🦁 NestJS: `main.ts`

Di NestJS, file utamanya standar bernama `main.ts`. Di sini kita membuat "Instance Aplikasi" menggunakan `NestFactory`.

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule); // Inisialisasi
  await app.listen(3000); // Jalankan server di port 3000
}
bootstrap();

```

---

## 2.2. Mengelompokkan Fitur (Organization)

Bagaimana cara agar kodingan tidak menumpuk di satu file?

### 🐍 Flask: `Blueprint`

Kamu menggunakan `Blueprint` untuk memisahkan fitur, misalnya memisahkan rute User dan rute Produk.

```python
# user_routes.py
user_bp = Blueprint('user', __name__)

```

### 🦁 NestJS: `Module`

Konsepnya sangat mirip. NestJS menggunakan `@Module` untuk membungkus Controller dan Service yang berkaitan. Bedanya, Module di NestJS wajib didaftarkan agar aplikasi mengenalinya.

```typescript
// user.module.ts
@Module({
  controllers: [UserController],
  providers: [UserService],
})
export class UserModule {}

```

> **Catatan:** Module adalah warga kelas satu di NestJS. Jika sebuah fitur tidak masuk ke dalam Module, fitur itu tidak akan jalan.

---

## 2.3. Menangani Request & Routing

Bagaimana cara mendefinisikan URL endpoint?

### 🐍 Flask: `@app.route`

Kamu menempelkan *decorator* langsung pada fungsi.

```python
@app.route('/users', methods=['GET'])
def get_users():
    return "List of users"

```

### 🦁 NestJS: `@Controller`

Routing dibagi dua tahap: Prefix di level Class, dan Method di level fungsi.

```typescript
@Controller('users') // 1. Prefix URL '/users'
export class UserController {
  
  @Get() // 2. Method GET. Jadi URL akhirnya: GET /users
  getUsers() {
    return "List of users";
  }
}

```

---

## 2.4. Logika Bisnis (Business Logic)

Di mana kita menulis kode "if-else", kalkulasi, dan panggilan database?

### 🐍 Flask: `View Function`

Di Flask, seringkali logika bisnis ditulis campur di dalam fungsi *route*.

```python
@app.route('/buy', methods=['POST'])
def buy_item():
    # Logika validasi, hitung harga, save DB campur di sini
    user = User.query.get(1)
    if user.balance < 1000:
        return "Gagal", 400
    ...

```

### 🦁 NestJS: `Service`

NestJS (dan best practice modern) memaksa pemisahan tugas. Controller hanya menerima request, lalu melempar tugas berat ke **Service**.

```typescript
// user.service.ts
@Injectable()
export class UserService {
  buyItem(userId: string) {
    // Logika bisnis murni di sini
    // Controller tidak boleh tau cara hitung harga
  }
}

```

---

## 2.5. Validasi Data (Validation)

Bagaimana memastikan data JSON yang dikirim client sudah benar?

### 🐍 Flask: `Marshmallow` / `Pydantic`

Kamu mungkin menggunakan library tambahan seperti Marshmallow untuk memvalidasi skema data.

### 🦁 NestJS: `DTO` + `class-validator`

NestJS menggunakan konsep **DTO (Data Transfer Object)**. Ini adalah Class sederhana yang diberi dekorator validasi.

```typescript
// create-user.dto.ts
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  name: string;

  @IsEmail()
  email: string;
}

```

Jika data yang dikirim tidak sesuai (misal email tidak valid), NestJS otomatis menolak request tersebut sebelum sampai ke Controller.

---

## 2.6. Keamanan & Middleware

Bagaimana cara memproteksi route (misal: hanya user login)?

### 🐍 Flask: Decorator (`@login_required`)

Kamu membuat fungsi wrapper (decorator) sendiri atau dari library `flask-login`.

```python
@app.route('/secret')
@login_required
def secret_page():
    pass

```

### 🦁 NestJS: `Guards`

NestJS punya konsep spesifik bernama **Guard**. Ini adalah Class yang bertugas mengizinkan atau menolak request.

```typescript
@UseGuards(AuthGuard) // Guard dipasang di sini
@Get('secret')
getSecretPage() {
  return "This is secret";
}

```

---

## 📝 Tabel Ringkasan (Cheat Sheet)

Simpan tabel ini sebagai referensi cepat saat kamu bingung.

| Konsep | Istilah di 🐍 Flask | Istilah di 🦁 NestJS |
| --- | --- | --- |
| **Inisialisasi** | `app = Flask(__name__)` | `NestFactory.create()` |
| **Grup Fitur** | `Blueprint` | `Module` |
| **Routing** | `@app.route` | `@Controller` |
| **Logika** | View Function | `Service` (Provider) |
| **Data Masuk** | `request.json` | `DTO` (Data Transfer Object) |
| **Validasi** | Marshmallow / Manual | `class-validator` / `Pipes` |
| **Database** | SQLAlchemy | TypeORM / Prisma |
| **Proteksi** | `@login_required` | `Guards` |
| **Testing** | `pytest` | `Jest` |

---

**Langkah Selanjutnya:**
Sekarang kita sudah punya "kamus bahasa"-nya. Di **Bab 3**, kita akan membahas hal yang paling sering ditanyakan developer Flask: **"Bagaimana cara menyusun folder project agar tidak berantakan?"** karena struktur folder NestJS sangat berbeda dengan Flask.
