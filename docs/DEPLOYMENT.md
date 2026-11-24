# Panduan Deployment (VPS 2 core / 2 GB RAM)

Acuan: PRD bagian 7 (non-fungsional) dan 8 (arsitektur). Target: Ubuntu LTS, Nginx, PHP-FPM 8.3, PostgreSQL 16, tanpa Redis (antrean di database).

> Dokumen ini belum diuji di VPS sungguhan. Uji di staging sebelum dipakai produksi, dan koreksi bila ada langkah yang berbeda.

## 1. Paket dasar

```bash
sudo apt update && sudo apt install -y nginx postgresql php8.3-fpm php8.3-cli \
  php8.3-pgsql php8.3-intl php8.3-mbstring php8.3-xml php8.3-zip php8.3-gd \
  php8.3-curl supervisor git unzip certbot python3-certbot-nginx
```

Pasang Composer dan Node.js (LTS) sesuai petunjuk resmi masing-masing.

## 2. Swap 2 GB

```bash
sudo fallocate -l 2G /swapfile && sudo chmod 600 /swapfile
sudo mkswap /swapfile && sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
echo 'vm.swappiness=10' | sudo tee /etc/sysctl.d/99-swap.conf
```

## 3. PostgreSQL

Buat database dan pengguna khusus aplikasi (ganti kata sandi).

```bash
sudo -u postgres psql -c "CREATE USER namche WITH PASSWORD 'GANTI_INI';"
sudo -u postgres psql -c "CREATE DATABASE namche_app OWNER namche;"
```

Tuning ringan di `/etc/postgresql/16/main/conf.d/namche.conf`:

```
shared_buffers = 256MB
effective_cache_size = 768MB
work_mem = 8MB
maintenance_work_mem = 64MB
max_connections = 40
```

Restart: `sudo systemctl restart postgresql`.

## 4. PHP-FPM (pm = ondemand)

Di `/etc/php/8.3/fpm/pool.d/www.conf`:

```
pm = ondemand
pm.max_children = 8
pm.process_idle_timeout = 20s
pm.max_requests = 500
```

Di `/etc/php/8.3/fpm/php.ini`: `memory_limit = 256M`, `upload_max_filesize = 5M`, `post_max_size = 8M`, `opcache.enable = 1`.

## 5. Aplikasi

```bash
sudo mkdir -p /var/www/namche_app && sudo chown $USER /var/www/namche_app
git clone https://github.com/abdulrahman777-id/namche_app /var/www/namche_app
cd /var/www/namche_app
composer install --no-dev --optimize-autoloader
cp .env.example .env && php artisan key:generate
npm ci && npm run build
```

Isi `.env` produksi: `APP_ENV=production`, `APP_DEBUG=false`, `APP_URL`, kredensial `DB_*`, `QUEUE_CONNECTION=database`. File `.env` tidak boleh masuk Git. Lalu:

```bash
php artisan migrate --force
php artisan storage:link
php artisan config:cache && php artisan route:cache && php artisan view:cache
sudo chown -R www-data:www-data storage bootstrap/cache
```

Unggahan pengguna disimpan di disk privat (di luar webroot), bukan `public`.

## 6. Nginx dan HTTPS

Server block minimal (`/etc/nginx/sites-available/namche_app`):

```nginx
server {
    server_name sekolah.example.sch.id;
    root /var/www/namche_app/public;
    index index.php;
    client_max_body_size 8M;

    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    location / { try_files $uri $uri/ /index.php?$query_string; }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }
    location ~ /\.(?!well-known).* { deny all; }
}
```

Aktifkan situs, lalu pasang sertifikat: `sudo certbot --nginx -d sekolah.example.sch.id`.

## 7. Worker antrean (satu worker) dan scheduler

`/etc/supervisor/conf.d/namche-worker.conf`:

```
[program:namche-worker]
command=php /var/www/namche_app/artisan queue:work --sleep=3 --tries=3 --max-time=3600 --memory=128
user=www-data
autostart=true
autorestart=true
numprocs=1
stopwaitsecs=3600
```

Cron scheduler (untuk pembentukan sesi harian dan kirim ulang pesan gagal):

```
* * * * * www-data cd /var/www/namche_app && php artisan schedule:run >> /dev/null 2>&1
```

Setelah deploy ulang, jalankan `php artisan queue:restart`.

## 8. Backup `pg_dump`

Skrip `/usr/local/bin/namche-backup.sh` (jalankan harian lewat cron). Retensi minimal 14 hari, salin ke luar server, uji restore tiap bulan.

```bash
#!/usr/bin/env bash
set -euo pipefail
DIR=/var/backups/namche; mkdir -p "$DIR"
F="$DIR/namche_$(date +%F).sql.gz.gpg"
sudo -u postgres pg_dump namche_app | gzip | gpg --batch --yes --symmetric \
  --passphrase-file /root/.namche-backup-pass -o "$F"
find "$DIR" -name 'namche_*.gpg' -mtime +14 -delete
# Salin ke luar server, mis. rclone copy "$DIR" remote:namche-backup
```

Restore: `gpg -d berkas.sql.gz.gpg | gunzip | sudo -u postgres psql namche_app` (pada database kosong).

## 9. Pemantauan

Pantau uptime, ukuran antrean `jobs`/`failed_jobs`, dan status perangkat Fonnte (mulai Fase 6). Pakai Sentry atau log Laravel untuk error.

## 10. Rilis berikutnya

```bash
git pull && composer install --no-dev --optimize-autoloader
npm ci && npm run build
php artisan migrate --force
php artisan config:cache && php artisan route:cache && php artisan view:cache
php artisan queue:restart
```
