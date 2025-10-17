# 🗳️ Example Voting App – 99Group DevOps Challenge

Halo! 👋  
Terima kasih sudah membuka repository ini.  
Ini adalah submission saya untuk **DevOps Internship Challenge**, di mana fokus utama adalah membangun aplikasi voting sederhana yang:

- Dapat dijalankan **secara lokal maupun dengan Docker**.  
- Menerapkan prinsip **containerization**, **multi-stage build**, dan **infrastructure automation (IaC)**.  
- Memiliki arsitektur **multi-service** dengan Redis, PostgreSQL, dan Worker (.NET).  

---

## 🚀 Cara Menjalankan Aplikasi

### 1️⃣ Jalankan dengan Docker (Direkomendasikan)

#### Prasyarat:
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

#### Langkah Menjalankan:
Clone Projek
```bash
# Clone repository
git clone <your-repo-url>
cd example-voting-app
```
Jalankan di Docker

```bash
# Jalankan seluruh service
docker compose up --build
```
Akses aplikasi:

- 🗳️ Voting page:
```bash
http://localhost:8080
```

- 📊 Result page:
```bash
http://localhost:8081
```

Hentikan semua container:
```bash
docker compose down
```

💡 Docker akan otomatis membangun image untuk setiap service dan menjalankannya dalam container terisolasi dengan network internal.

### 2️⃣ Jalankan Secara Manual (Tanpa Docker)

#### Prasyarat:

- Python 3.x

- Redis (port 6379)

- PostgreSQL (port 5432)

- Node.js 18+

- .NET SDK 7.0

#### Langkah:
```bash
# Jalankan Redis dan PostgreSQL secara manual

# Vote service
cd vote
pip install -r requirements.txt
python app.py

# Worker service (.NET)
cd worker
dotnet run

# Result service (Node.js)
cd result
npm install
node server.js
```

Akses:

http://localhost:8080
 → Voting Page

http://localhost:8081
 → Result Page

---

## ⚙️ Arsitektur & Komponen
```markdown
[Vote App (Python Flask)]  --->  [Redis Queue]  --->  [Worker (.NET)]  --->  [Postgres DB]
      │                                                           │
      └───────────────────────────────────────────────────────────┘
                             ↓
                   [Result App (Node.js)]
```

| Service  | Teknologi                     | Fungsi                               |
| -------- | ----------------------------- | ------------------------------------ |
| `vote`   | Python (Flask + Gunicorn)     | UI untuk memilih opsi                |
| `result` | Node.js (Express + Socket.io) | Menampilkan hasil voting real-time   |
| `worker` | .NET 7                        | Memproses dan menyimpan hasil voting |
| `redis`  | Redis Alpine                  | Menyimpan antrian suara sementara    |
| `db`     | PostgreSQL 15-alpine          | Penyimpanan hasil akhir              |
| `seed`   | Python (opsional)             | Mengisi data awal (profil: `seed`)   |

---

## 🗂️ Struktur Folder
```markdown
/vote               → Frontend voting (Python Flask)
/result             → Hasil voting (Node.js)
/worker             → Backend worker (.NET)
/seed-data          → Seeder opsional untuk populasi data awal
/docker-compose.yml → Orkestrasi container
/terraform/         → (Opsional) IaC setup untuk Task 3
```

---

## 🧠 Keputusan Teknis

- Multi-stage Dockerfile → image ringan dan cepat dibangun.

- Redis Queue → memisahkan input vote & proses simpan.

- PostgreSQL Volume → data tetap aman antar container restart.

- Network Separation (front-tier & back-tier) → isolasi & keamanan.

- Healthcheck Compose → memastikan dependency siap sebelum container lain berjalan.

---

## 📈 Potensi Peningkatan

- ✅ **Continuous Integration (CI):**  
  Saat ini sudah ada GitHub Actions untuk menjalankan `npm test` sebelum push atau pull request ke branch `main`.  
  Pipeline ini memastikan kode tetap stabil dan bebas error sejak tahap awal.

- 🚀 **Continuous Deployment (CD):**  
  Bisa ditingkatkan dengan otomatisasi build dan push image ke Docker Hub, lalu auto-deploy ke staging/production.

- 📊 **Observability:**  
  Integrasi Prometheus + Grafana + Loki untuk metrik, log, dan visualisasi performa container.

- 🧪 **Testing yang Lebih Lengkap:**  
  Tambahkan unit & integration test untuk setiap service (vote, result, worker).

- ⚡ **Performance Testing:**  
  Gunakan load testing (misalnya `k6` atau `Locust`) untuk optimasi throughput & latency.

- 🔐 **Security & Rate Limiting:**  
  Tambahkan autentikasi user dan pembatasan request untuk endpoint voting agar tidak disalahgunakan.
--- 

Notes
-

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to deal with them in Docker at a basic level.