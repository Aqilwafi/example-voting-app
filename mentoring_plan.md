# 🧩 Monitoring Plan – Example Voting App

Dokumen ini menjelaskan rencana monitoring untuk aplikasi voting sederhana ini. Tujuannya adalah memastikan semua komponen aplikasi dapat terpantau dengan baik di lingkungan produksi, serta tim bisa segera mengetahui dan menanggapi setiap gangguan.

---

## 🛠️ Tools yang Digunakan

| Tool | Fungsi | Alasan Penggunaan |
|------|---------|-------------------|
| **Prometheus** | Mengumpulkan dan menyimpan metrik dari setiap service (vote, worker, result, redis, db). | Ringan, efisien, dan integrasi mudah dengan Docker. |
| **Grafana** | Menampilkan metrik dari Prometheus dalam bentuk grafik dan dashboard interaktif. | Visualisasi real-time untuk memudahkan analisis dan debugging. |
| **Alertmanager** | Mengirimkan notifikasi otomatis jika terjadi anomali. | Terintegrasi langsung dengan Prometheus dan bisa dikirim ke Slack/email. |
| **Loki** | Mengumpulkan log dari tiap container (vote, worker, result). | Untuk menelusuri error secara cepat tanpa harus masuk ke tiap container. |

---

## 📊 Metrik yang Dipantau

Aplikasi ini terdiri dari beberapa komponen:  
- **Vote (frontend API untuk input suara)**  
- **Worker (proses backend untuk membaca dari Redis dan menulis ke DB)**  
- **Result (frontend untuk menampilkan hasil)**  
- **Redis & DB (penyimpanan sementara dan permanen)**  

### 1. Kesehatan Infrastruktur
| Metrik | Tujuan | Contoh Query (PromQL) |
|--------|---------|----------------------|
| CPU Usage | Mengetahui apakah container overload. | `rate(container_cpu_usage_seconds_total[5m])` |
| Memory Usage | Mendeteksi memory leak atau lonjakan. | `container_memory_usage_bytes` |
| Container Status | Mengecek apakah ada service yang restart/berhenti. | `container_last_seen` |

### 2. Kinerja Aplikasi
| Komponen | Metrik | Penjelasan |
|-----------|--------|------------|
| **Vote Service** | HTTP Request Rate & Error Rate | Memastikan endpoint `/vote` tidak overload dan tidak banyak error 5xx. |
| **Worker Service** | Queue Depth (Redis) | Jika antrean terlalu dalam, berarti worker tidak cukup cepat memproses suara. |
| **Result Service** | Response Latency | Mengecek seberapa cepat hasil vote ditampilkan ke pengguna. |

### 3. Pengalaman Pengguna
| Metrik | Penjelasan |
|--------|-------------|
| Response Time | Waktu respon rata-rata dari Vote dan Result. |
| Error Rate | Persentase permintaan yang gagal. |
| Throughput | Jumlah request sukses per detik. |

---

## 🚨 Sistem Peringatan (Alerts)

Contoh beberapa alert yang bisa diterapkan di **Alertmanager**:

### 1. Error Rate Tinggi
```yaml
- alert: HighErrorRate
  expr: rate(http_requests_total{status=~"5.."}[2m]) > 0.05
  for: 2m
  labels:
    severity: critical
  annotations:
    summary: "Error rate tinggi pada Vote Service"
    description: "Lebih dari 5% permintaan ke /vote gagal dalam 2 menit terakhir."
```

---

### Contoh Prometheus Setup (pseudocode)
```yaml
scrape_configs:
  - job_name: 'vote'
    static_configs:
      - targets: ['vote:80']

  - job_name: 'worker'
    static_configs:
      - targets: ['worker:80']

  - job_name: 'result'
    static_configs:
      - targets: ['result:80']

  - job_name: 'redis'
    static_configs:
      - targets: ['redis:6379']
```
