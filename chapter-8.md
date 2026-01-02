# 📘 Bab 8: Testing & Quality Assurance (The Safety Net)

> **Quote of the Chapter:**
> *"Di Flask, testing seringkali menjadi renungan (afterthought). Di NestJS, testing adalah warga kelas satu. Setiap kali kamu membuat file baru, file test-nya otomatis dibuatkan."*

Jika kamu terbiasa menggunakan **`pytest`** di Flask, kamu mungkin punya hubungan benci-tapi-cinta dengan testing.

* **Cintanya:** `pytest` itu simpel dan *powerful*.
* **Bencinya:** Melakukan *mocking* pada dependensi global (seperti `from database import db`) seringkali rumit. Kamu harus menggunakan `unittest.mock.patch` yang kadang bikin pusing.

Kabar gembira: Di NestJS, testing jauh lebih terisolasi berkat **Dependency Injection**.

NestJS menggunakan **Jest** sebagai *test runner* default. Jest adalah "standar industri" di dunia JavaScript/TypeScript, sama seperti `pytest` di dunia Python.

---

## 8.1. Pytest vs Jest: Bedanya Apa?

Secara filosofi, keduanya mirip.

| Fitur | 🐍 Flask (`pytest`) | 🦁 NestJS (`Jest`) |
| --- | --- | --- |
| **File Ekstensi** | `test_*.py` atau `*_test.py` | `*.spec.ts` (untuk Unit) & `*.e2e-spec.ts` (untuk E2E) |
| **Grouping** | Class atau Function | `describe('Nama Group', () => { ... })` |
| **Test Case** | `def test_login():` | `it('should login', () => { ... })` |
| **Assertion** | `assert x == y` | `expect(x).toEqual(y)` |
| **Setup** | `@pytest.fixture` | `beforeEach(async () => { ... })` |

---

## 8.2. Unit Testing: Kekuatan Dependency Injection

Ingat di Bab 1 kita membahas bahwa Controller "meminta" Service lewat `constructor`? Inilah alasan utamanya.

Saat testing, kita bisa dengan mudah menukar **Service Asli** (yang konek ke database) dengan **Service Palsu (Mock)** tanpa mengubah kode Controller sedikitpun.

### Skenario

Kita mau ngetes `UsersController`. Kita tidak mau test ini menyentuh database beneran.

**Kode Controller (Yang mau dites):**

```typescript
constructor(private userService: UsersService) {} // <--- Dependensi

```

**Kode Test (`users.controller.spec.ts`):**

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

describe('UsersController', () => {
  let controller: UsersController;
  let fakeService: UsersService;

  beforeEach(async () => {
    // 1. Buat Module Pura-pura (Testing Module)
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        {
          provide: UsersService,
          // 2. Ganti Service Asli dengan Mock Object
          useValue: {
            findAll: jest.fn().mockReturnValue(['User Palsu 1', 'User Palsu 2']),
          },
        },
      ],
    }).compile();

    controller = module.get<UsersController>(UsersController);
  });

  // 3. Jalankan Test
  it('should return array of users', () => {
    expect(controller.findAll()).toEqual(['User Palsu 1', 'User Palsu 2']);
  });
});

```

**Perbandingan dengan Flask:**
Di Flask, kamu mungkin harus melakukan `patch('app.routes.get_users')`. Jika lokasi import berubah, test kamu error. Di NestJS, kita mengganti object-nya langsung di level module, jadi lebih aman dan bersih.

---

## 8.3. End-to-End (E2E) Testing

Unit test memeriksa satu bagian kecil. E2E test memeriksa keseluruhan aplikasi dari luar (HTTP Request).

Di Flask, kamu menggunakan `client = app.test_client()`.
Di NestJS, kita menggunakan library bernama **Supertest**.

NestJS biasanya menyimpan file E2E di folder terpisah bernama `test/`.

```typescript
// app.e2e-spec.ts
import * as request from 'supertest';
import { Test } from '@nestjs/testing';
import { AppModule } from './../src/app.module';

describe('AppController (e2e)', () => {
  let app;

  beforeAll(async () => {
    // Bangun aplikasi full (seperti main.ts)
    const moduleFixture = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it('/users (GET)', () => {
    return request(app.getHttpServer())
      .get('/users')       // Tembak URL
      .expect(200)         // Harapannya status 200
      .expect({            // Harapannya body JSON-nya ini
        data: [] 
      });
  });
});

```

Konsepnya 100% sama dengan `test_client` di Flask. Kamu mengirim request HTTP sungguhan ke aplikasi yang berjalan di memori.

---

## 8.4. Kenapa Ada Banyak File `.spec.ts`?

Setiap kali kamu menggunakan CLI (`nest generate service users`), NestJS otomatis membuat dua file:

1. `users.service.ts` (Kode asli)
2. `users.service.spec.ts` (File test)

**Jangan dihapus!**
NestJS mencoba mendisiplinkan developer. Dengan file test yang sudah dibuatkan kerangkanya, "hambatan mental" untuk mulai nulis test jadi berkurang. Kamu tinggal isi bagian `it(...)`.

---

## 8.5. Debugging Test

Di Python, kamu mungkin pakai `pdb` atau `print`.
Di Node.js/NestJS, `console.log` tetap berfungsi saat testing.

Untuk menjalankan test:

* **Run semua test:** `npm run test`
* **Run test yang berubah saja (Watch mode):** `npm run test:watch` (Sangat berguna saat coding!)
* **Run E2E test:** `npm run test:e2e`

---

## 📝 Ringkasan Bab 8

1. **Jest** adalah teman barumu. Sintaksnya beda dikit sama `pytest`, tapi logikanya sama.
2. **Dependency Injection** membuat Unit Testing di NestJS sangat menyenangkan karena *mocking* jadi mudah dan rapi.
3. **Automation:** NestJS CLI otomatis membuatkan file test (`.spec.ts`) untuk setiap komponen yang kamu buat.
4. **Tipe Test:**
* `*.spec.ts`: Unit Test (Tes fungsi/class terisolasi).
* `*.e2e-spec.ts`: Integration Test (Tes API dari URL).



---

**Langkah Selanjutnya:**
Kita sudah di penghujung panduan! Aplikasi sudah jadi, database konek, validasi aman, dan test sudah jalan. Langkah terakhir adalah: **Bagaimana cara menjalankannya di Production?**

Di **Bab 9 (Terakhir)**, kita akan membahas **Deployment**. Kita akan melihat bagaimana cara membungkus aplikasi NestJS agar siap dijalankan di server (Docker, PM2, atau Cloud), karena cara menjalankannya beda dengan `gunicorn` atau `uwsgi` di Flask.
