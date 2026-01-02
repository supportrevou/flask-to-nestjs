# 📘 Bab 9: Deployment & Production (The Launch)

> **Quote of the Chapter:**
> *"Jangan pernah menjalankan `npm run start:dev` di server production. Itu sama dosanya dengan menjalankan Flask pakai `app.run(debug=True)` di live server."*

Selamat! Aplikasi NestJS kamu sudah jalan, databasenya oke, test-nya lolos. Sekarang saatnya kita taruh di server agar bisa diakses dunia.

Di dunia Flask, kamu mungkin terbiasa dengan kombinasi **Nginx + Gunicorn + Systemd**.
Di dunia NestJS, kombinasinya adalah **Nginx + PM2 + Node**.

Mari kita lihat perbedaannya.

---

## 9.1. Langkah Wajib: The Build Process 🏗️

Ini perbedaan terbesar.

* **Python:** Bahasa *interpreted*. Kamu upload file `.py`, install pip, lalu jalan.
* **TypeScript:** Bahasa yang **harus dikompilasi** menjadi JavaScript agar bisa dimengerti oleh Node.js.

Sebelum deploy, kamu harus menjalankan perintah:

```bash
npm run build

```

Apa yang terjadi?

1. NestJS akan membaca semua kode TypeScript kamu di folder `src/`.
2. Ia akan menerjemahkan (transpile) menjadi JavaScript standar.
3. Hasilnya disimpan di folder baru bernama **`dist/`** (Distribution).

📂 **Folder `dist/` inilah aplikasi kamu yang sebenarnya.** Saat di production, kamu tidak lagi menjalankan file di `src/`, tapi file di `dist/`.

---

## 9.2. Menjalankan Aplikasi: Gunicorn vs Node

Bagaimana cara menyalakan servernya?

### 🐍 Flask (Gunicorn)

Flask bawaan (`app.run`) itu lemah, cuma buat development. Di production, kita butuh WSGI server.

```bash
gunicorn -w 4 app:app

```

### 🦁 NestJS (Node)

Sama, `npm run start:dev` itu berat karena dia memantau perubahan file (hot-reload). Di production, kita jalankan hasil build tadi.

Perintah manualnya:

```bash
node dist/main.js

```

*Artinya: "Node, tolong jalankan file main.js yang ada di folder dist."*

---

## 9.3. Menjaga Tetap Hidup: PM2 🛡️

Kalau Gunicorn bertugas mengatur *worker process*, di Node.js kita menggunakan **PM2 (Process Manager 2)**.

Kenapa butuh PM2?

1. **Auto Restart:** Kalau aplikasi crash, PM2 akan menghidupkannya lagi dalam hitungan milidetik.
2. **Cluster Mode:** Node.js itu *single-threaded*. Jika servermu punya 4 CPU core, Node.js cuma pakai 1. PM2 bisa meng-copy aplikasimu ke 4 core tersebut (mirip worker Gunicorn).

**Cara Install & Pakai:**

```bash
# 1. Install PM2 di server global
npm install pm2 -g

# 2. Jalankan aplikasi (Ganti Gunicorn dengan ini)
pm2 start dist/main.js --name "my-nest-app"

```

---

## 9.4. Docker: Bahasa Universal 🐳

Jika kamu tim Docker, transisinya sangat mudah. Perbedaannya hanya di `Dockerfile`.

### 🐍 Flask Dockerfile

```dockerfile
FROM python:3.9
COPY . .
RUN pip install -r requirements.txt
CMD ["gunicorn", "app:app"]

```

### 🦁 NestJS Dockerfile

Kita butuh langkah *build* di dalam Docker.

```dockerfile
# Gunakan image Node versi LTS (Long Term Support)
FROM node:18-alpine

WORKDIR /app

# 1. Copy file dependency dulu (biar cache layer optimal)
COPY package*.json ./

# 2. Install semua library
RUN npm install

# 3. Copy sisa kode source
COPY . .

# 4. 🔥 PENTING: Lakukan Build
RUN npm run build

# 5. Jalankan file hasil build (bukan source ts)
CMD ["node", "dist/main"]

```

> **Pro Tip:** Untuk production yang serius, gunakan "Multi-stage Build" agar image Docker-nya kecil (membuang file TypeScript source yang tidak perlu). Tapi Dockerfile di atas sudah cukup untuk memulai.

---

## 9.5. Environment Variables

Di Flask, kamu mungkin pakai `python-dotenv` untuk membaca file `.env`.
Di NestJS, prinsipnya sama.

Pastikan di server production kamu:

1. Sudah ada file `.env`.
2. Atau variable sudah di-set di sistem operasi (Environment server).

NestJS (`@nestjs/config`) akan otomatis membacanya lewat `process.env.NAMA_VARIABLE`.

---

## 📝 Ringkasan Bab 9

| Tahap | 🐍 Flask | 🦁 NestJS |
| --- | --- | --- |
| **Persiapan Code** | Tidak ada (langsung run). | `npm run build` (Wajib). |
| **Folder Executable** | `app.py` / root folder. | `dist/` folder. |
| **Server Runner** | Gunicorn / uWSGI. | `node dist/main.js`. |
| **Process Manager** | Systemd / Supervisor. | PM2. |
| **Reverse Proxy** | Nginx. | Nginx (Sama saja). |

---

# 🎓 Kata Penutup: Selamat Lulus!

Kamu baru saja menyelesaikan panduan migrasi dari Flask ke NestJS.

**Apa yang telah kamu pelajari?**

1. **Mindset:** Berpindah dari kebebasan (Flask) ke struktur (NestJS).
2. **Arsitektur:** Memecah kode menjadi Controller, Service, dan Module.
3. **Database:** Menggunakan Repository Pattern dan TypeORM.
4. **Keamanan:** Menggunakan DTO, Guards, dan Middleware.
5. **Quality:** Menulis Unit Test dengan Jest.
6. **Production:** Melakukan Build dan Deploy dengan PM2/Docker.

**Saran Terakhir:**
NestJS memiliki kurva belajar yang curam di awal (*steep learning curve*). Minggu pertama akan terasa berat dan penuh error.

* *"Kenapa error dependency not resolved?"* (Biasanya lupa masukin ke Module).
* *"Kenapa datanya null?"* (Biasanya lupa `await`).

Tapi begitu kamu melewati fase itu, kamu akan merasakan produktivitas yang luar biasa, kode yang rapi, dan tidur yang nyenyak karena Type Safety.

*Happy Coding, Ex-Flask Developer!* 🚀

---
