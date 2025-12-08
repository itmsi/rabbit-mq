# Setup RabbitMQ dengan Docker Compose

Dokumentasi ini menjelaskan cara setup RabbitMQ menggunakan Docker Compose dengan network Traefik.

## Prasyarat

1. **Docker** versi 29.1.2 atau lebih baru
2. **Docker Compose** terinstall
3. **Network Traefik** sudah dibuat sebelumnya

## Langkah-langkah Setup

### 1. Pastikan Network Traefik Sudah Ada

Sebelum menjalankan RabbitMQ, pastikan network `traefik-network` sudah dibuat. Jika belum, buat dengan perintah:

```bash
docker network create traefik-network
```

Untuk memverifikasi network sudah ada:

```bash
docker network ls | grep traefik-network
```

### 2. Clone atau Download File Docker Compose

Pastikan file `docker-compose.yml` sudah ada di direktori kerja Anda.

### 3. Konfigurasi Environment Variables (Opsional)

File `docker-compose.yml` menggunakan default credentials:
- **Username**: `admin`
- **Password**: `admin123`

**⚠️ PENTING**: Untuk production, ubah password default dengan cara:

1. Buat file `.env` di direktori yang sama dengan `docker-compose.yml`:

```bash
cat > .env << EOF
RABBITMQ_DEFAULT_USER=your_username
RABBITMQ_DEFAULT_PASS=your_secure_password
EOF
```

2. Update `docker-compose.yml` untuk menggunakan file `.env`:

```yaml
environment:
  RABBITMQ_DEFAULT_USER: ${RABBITMQ_DEFAULT_USER:-admin}
  RABBITMQ_DEFAULT_PASS: ${RABBITMQ_DEFAULT_PASS:-admin123}
```

### 4. Jalankan RabbitMQ

Jalankan container dengan perintah:

```bash
docker compose up -d
```

Atau jika menggunakan versi lama Docker Compose:

```bash
docker-compose up -d
```

### 5. Verifikasi Container Berjalan

Cek status container:

```bash
docker compose ps
```

Atau:

```bash
docker ps | grep rabbitmq
```

### 6. Akses Management UI

Setelah container berjalan, akses Management UI melalui browser:

- **URL**: http://localhost:15672
- **Username**: `admin` (atau sesuai konfigurasi)
- **Password**: `admin123` (atau sesuai konfigurasi)

### 7. Test Koneksi dari Container Lain

Karena RabbitMQ menggunakan network `traefik-network`, container lain di network yang sama dapat mengakses RabbitMQ dengan:

- **Hostname**: `rabbitmq`
- **Port AMQP**: `5672`
- **Port Management**: `15672`

Contoh koneksi dari aplikasi lain di network yang sama:

```
amqp://admin:admin123@rabbitmq:5672/
```

## Port yang Digunakan

- **5672**: Port AMQP untuk koneksi aplikasi
- **15672**: Port Management UI untuk web interface

## Volume Data

Data RabbitMQ disimpan dalam Docker volumes:
- `rabbitmq_data`: Data persistent RabbitMQ
- `rabbitmq_logs`: Log files RabbitMQ

## Perintah Berguna

### Melihat Logs

```bash
docker compose logs -f rabbitmq
```

### Stop Container

```bash
docker compose stop
```

### Start Container

```bash
docker compose start
```

### Restart Container

```bash
docker compose restart rabbitmq
```

### Hapus Container (Data Tetap Tersimpan)

```bash
docker compose down
```

### Hapus Container dan Volume (Hapus Semua Data)

```bash
docker compose down -v
```

## Troubleshooting

### Container Tidak Bisa Start

1. Pastikan network `traefik-network` sudah ada:
   ```bash
   docker network inspect traefik-network
   ```

2. Cek port 5672 dan 15672 tidak digunakan aplikasi lain:
   ```bash
   sudo netstat -tulpn | grep -E '5672|15672'
   ```

### Tidak Bisa Akses dari Container Lain

1. Pastikan container lain juga menggunakan network `traefik-network`
2. Gunakan hostname `rabbitmq` untuk koneksi, bukan `localhost`

### Lupa Password

Jika lupa password, reset dengan:

```bash
docker compose exec rabbitmq rabbitmqctl change_password admin new_password
```

## Keamanan

1. **Ubah password default** sebelum production
2. **Gunakan firewall** untuk membatasi akses port dari luar
3. **Gunakan SSL/TLS** untuk koneksi production (konfigurasi tambahan diperlukan)
4. **Backup volume** secara berkala

## Informasi Versi

- **Docker**: 29.1.2 (build 890dcca)
- **RabbitMQ Image**: rabbitmq:3-management-alpine
- **Docker Compose Version**: 3.8

