# Open WebUI dengan Docker Compose

Konfigurasi Docker Compose untuk menjalankan [Open WebUI](https://github.com/open-webui/open-webui) bersama Playwright sebagai mesin pemuat halaman web. Data aplikasi disimpan secara persisten di direktori lokal `data/`.

## Layanan

| Layanan | Image | Fungsi |
| --- | --- | --- |
| `open-webui` | `ghcr.io/open-webui/open-webui:main` | Antarmuka web untuk berinteraksi dengan model AI |
| `playwright` | `mcr.microsoft.com/playwright:v1.60.0-noble` | Browser headless untuk mengambil konten halaman web |

Open WebUI tersedia di port `3000` secara default. Port host dapat diubah melalui variabel `PORT`.

## Prasyarat

- Linux (konfigurasi ini ditujukan untuk Debian atau turunannya)
- Docker Engine
- Docker Compose plugin (`docker compose`)
- Layanan model AI yang dapat diakses oleh Open WebUI, misalnya Ollama atau API yang kompatibel dengan OpenAI

Pastikan instalasi Docker sudah berjalan:

```bash
docker --version
docker compose version
```

## Konfigurasi

1. Clone repositori dan masuk ke direktorinya:

   ```bash
   git clone <URL_REPOSITORI>
   cd openwebui-debian
   ```

2. Buat file `.env` dan isi kunci rahasia aplikasi:

   ```dotenv
   WEBUI_SECRET_KEY=ganti-dengan-kunci-acak-yang-kuat
   ```

   Kunci dapat dibuat dengan OpenSSL:

   ```bash
   openssl rand -hex 32
   ```

   Jangan mengganti `WEBUI_SECRET_KEY` setelah aplikasi digunakan. Perubahan kunci dapat membuat data terenkripsi atau sesi yang sudah ada tidak dapat dibaca.

3. Opsional, tentukan port host saat menjalankan Compose:

   ```bash
   PORT=8080 docker compose up -d
   ```

   Jika `PORT` tidak ditentukan, aplikasi menggunakan `http://localhost:3000`.

## Menjalankan aplikasi

Jalankan semua layanan di latar belakang:

```bash
docker compose up -d
```

Periksa status container:

```bash
docker compose ps
```

Buka Open WebUI melalui:

```text
http://localhost:3000
```

Akun pertama yang didaftarkan biasanya menjadi administrator. Segera gunakan kata sandi yang kuat dan batasi akses jaringan jika layanan tidak dimaksudkan untuk publik.

## Operasi harian

Melihat log seluruh layanan:

```bash
docker compose logs -f
```

Melihat log Open WebUI saja:

```bash
docker compose logs -f open-webui
```

Menghentikan layanan tanpa menghapus data:

```bash
docker compose down
```

Memulai ulang layanan:

```bash
docker compose restart
```

## Memperbarui image

Konfigurasi menggunakan tag `main` untuk Open WebUI. Tarik image terbaru lalu buat ulang container:

```bash
docker compose pull
docker compose up -d
```

Sebaiknya backup direktori `data/` sebelum melakukan pembaruan.

## Data dan backup

Seluruh data persisten Open WebUI dipetakan dari container ke:

```text
./data/
```

Direktori tersebut dapat berisi database, unggahan, dan vector database. Untuk membuat backup yang konsisten, hentikan layanan terlebih dahulu:

```bash
docker compose down
tar -czf open-webui-data-backup.tar.gz data/
docker compose up -d
```

File `.env` tidak masuk ke Git. Simpan salinan `WEBUI_SECRET_KEY` di tempat aman karena backup data mungkin bergantung pada kunci tersebut.

## Menghubungkan model AI

Host Docker tersedia dari dalam container melalui alamat:

```text
host.docker.internal
```

Jika Ollama berjalan langsung di host, URL yang umumnya digunakan dari Open WebUI adalah:

```text
http://host.docker.internal:11434
```

Pastikan layanan model menerima koneksi dari jaringan Docker dan firewall host mengizinkannya. Konfigurasi koneksi model dapat dilakukan melalui panel administrator Open WebUI.

## Pemecahan masalah

### Port sudah digunakan

Gunakan port host lain:

```bash
PORT=8080 docker compose up -d
```

### Open WebUI tidak dapat terhubung ke model di host

- Gunakan `host.docker.internal`, bukan `localhost`, dari dalam container.
- Pastikan layanan model aktif dan mendengarkan pada interface yang dapat dijangkau Docker.
- Periksa log dengan `docker compose logs -f open-webui`.

### Playwright tidak siap

Periksa container dan log layanan:

```bash
docker compose ps playwright
docker compose logs playwright
```

Open WebUI menggunakan URL WebSocket internal `ws://playwright:3000`; port Playwright tidak dipublikasikan ke host.

## Keamanan

- Jangan commit `.env`, database, unggahan, atau backup ke repositori.
- Jangan publikasikan port aplikasi langsung ke internet tanpa HTTPS, autentikasi, dan pembatasan akses yang memadai.
- Gunakan reverse proxy seperti Nginx, Caddy, atau Traefik untuk deployment publik.
- Tinjau image baru dan lakukan backup sebelum pembaruan.

## Struktur proyek

```text
.
├── docker-compose.yml  # Definisi layanan Open WebUI dan Playwright
├── .env                # Rahasia lokal; tidak dilacak oleh Git
└── data/               # Data persisten Open WebUI; tidak dilacak oleh Git
```