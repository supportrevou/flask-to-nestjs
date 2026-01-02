# 📘 Bab 6: Database & ORM (The Memory)

> **Quote of the Chapter:**
> *"Data yang valid di Bab 5 akan hilang jika komputer dimatikan. Saatnya kita simpan permanen ke Database."*

Di dunia Flask, standar emas untuk database adalah **SQLAlchemy**. Ia adalah ORM (*Object Relational Mapper*) yang menerjemahkan kode Python menjadi SQL.

Di dunia NestJS, pemain utamanya adalah **TypeORM** (atau **Prisma**). Kita akan fokus pada **TypeORM** karena integrasinya sangat erat dengan NestJS dan konsepnya paling mirip dengan SQLAlchemy.

Mari kita lihat bagaimana transisi dari `db.Model` ke `@Entity()`.

---

## 6.1. Koneksi Database (The Setup)

Di Flask, kamu biasanya menginisialisasi ekstensi `db` di `app.py` atau `extensions.py`.

### 🐍 Flask (SQLAlchemy Setup)

```python
# app.py
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://root:pass@localhost/db'
db = SQLAlchemy(app)

```

### 🦁 NestJS (TypeORM Setup)

Di NestJS, koneksi database diatur di **Root Module** (`AppModule`). Kita mengimpor `TypeOrmModule`.

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './users/user.entity';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',            // Jenis DB (bisa postgres, sqlite, dll)
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'password',
      database: 'test_db',
      entities: [User],         // Daftar tabel/entity
      synchronize: true,        // ⚠️ Hati-hati: Ini auto-create table (mirip db.create_all)
    }),
    UsersModule,
  ],
})
export class AppModule {}

```

> **Catatan:** `synchronize: true` sangat berguna saat *development* karena tabel dibuat otomatis sesuai kodingan kita. Tapi **JANGAN** nyalakan ini di *production* karena bisa menghapus data jika skema berubah.

---

## 6.2. Model vs Entity

Bagaimana mendefinisikan tabel User? Ternyata sangat mirip!

### 🐍 Flask (Model)

```python
class User(db.Model):
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50))
    is_active = db.Column(db.Boolean, default=True)

```

### 🦁 NestJS (Entity)

Bedanya hanya penggunaan Decorator.

```typescript
// user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity() // Menandakan ini adalah tabel database
export class User {
  @PrimaryGeneratedColumn() // Auto Increment ID
  id: number;

  @Column({ length: 50 })
  name: string;

  @Column({ default: true })
  isActive: boolean;
}

```

---

## 6.3. The Repository Pattern (Konsep Baru)

Ini perbedaan terbesarnya.

* **Di Flask (Active Record):** Model itu pintar. Model bisa mencari dirinya sendiri.
* `User.query.all()` (Model `User` langsung punya method query).


* **Di NestJS (Data Mapper):** Entity itu "bodoh". Dia cuma data. Yang pintar adalah **Repository**.
* Kamu butuh "Perantara" (Repository) untuk mengambil Entity.



Untuk menggunakan Repository, kita harus meng-inject-nya ke dalam Service.

### Setup di Module

Pertama, beri tahu Module bahwa kita mau pakai entity `User`.

```typescript
// users.module.ts
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])], // <--- Daftarkan di sini
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}

```

### Penggunaan di Service

Sekarang, mari kita inject Repository ke `UsersService`.

```typescript
// users.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>, // <--- "Nest, pinjam alat buat akses tabel User dong"
  ) {}

  findAll() {
    return this.usersRepository.find(); // SELECT * FROM user
  }
}

```

---

## 6.4. Kamus Operasi CRUD

Berikut adalah panduan translasi query dari SQLAlchemy ke TypeORM Repository.

| Operasi | 🐍 Flask (SQLAlchemy) | 🦁 NestJS (TypeORM Repository) |
| --- | --- | --- |
| **Select All** | `User.query.all()` | `this.repo.find()` |
| **Select One** | `User.query.get(1)` | `this.repo.findOneBy({ id: 1 })` |
| **Where** | `User.query.filter_by(name='A').first()` | `this.repo.findOneBy({ name: 'A' })` |
| **Create/Insert** | `u = User(name='A')`<br>

<br>`db.session.add(u)`<br>

<br>`db.session.commit()` | `const u = this.repo.create({ name: 'A' })`<br>

<br>`return this.repo.save(u)` |
| **Update** | `u.name = 'B'`<br>

<br>`db.session.commit()` | `await this.repo.update(id, { name: 'B' })` |
| **Delete** | `db.session.delete(u)`<br>

<br>`db.session.commit()` | `await this.repo.delete(id)` |

> **Info:** Di NestJS (TypeORM), `save()` bersifat *asynchronous* (mengembalikan Promise). Jadi kita harus menggunakan `await`.

---

## 6.5. Relasi Antar Tabel (Relationship)

Relasi One-to-Many di Flask dan NestJS juga mirip secara konsep.

### 🐍 Flask

```python
# Parent
items = db.relationship('Item', backref='owner')
# Child
owner_id = db.Column(db.Integer, db.ForeignKey('user.id'))

```

### 🦁 NestJS

```typescript
// User Entity (Parent)
@OneToMany(() => Item, (item) => item.owner)
items: Item[];

// Item Entity (Child)
@ManyToOne(() => User, (user) => user.items)
owner: User;

```

Bedanya, di NestJS kamu harus mendefinisikan relasi di **kedua sisi** entity secara eksplisit menggunakan decorator `@OneToMany` dan `@ManyToOne`.

---

## 📝 Ringkasan Bab 6

1. **Konfigurasi Terpusat:** Koneksi database diatur sekali di `app.module.ts` menggunakan `TypeOrmModule.forRoot`.
2. **Entity = Model:** Gunakan decorator `@Entity()` pengganti `db.Model`.
3. **Repository Pattern:** Kita tidak query langsung ke Class Entity. Kita menggunakan objek `Repository` yang di-inject ke Service.
4. **Async/Await:** Interaksi database di Node.js bersifat *non-blocking*. Jangan lupa kata kunci `await` saat memanggil database, atau data tidak akan kembali!

---

**Langkah Selanjutnya:**
Sekarang kita punya aplikasi lengkap: Router (Controller), Logika (Service), Validasi (DTO), dan Database (TypeORM).

Tapi, apa yang terjadi jika ada kode yang error? Atau bagaimana cara kita mengamankan endpoint tertentu agar tidak bisa diakses sembarang orang? Di **Bab 7**, kita akan membahas **Middleware, Guards (Auth), dan Interceptors**. Ini adalah versi "Advanced" dari Flask Decorators.
