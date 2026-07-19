# 🔄 عملية الاستعادة - Recepio

> **دليل شامل لاستعادة النظام من النسخ الاحتياطية**

---

## 📋 جدول المحتويات

1. [نظرة عامة](#نظرة-عامة)
2. [المتطلبات](#المتطلبات)
3. [استعادة قاعدة البيانات](#استعادة-قاعدة-البيانات)
4. [استعادة الملفات](#استعادة-الملفات)
5. [استعادة السيرفر كاملاً](#استعادة-السيرفر-كاملاً)
6. [التحقق من الاستعادة](#التحقق-من-الاستعادة)
7. [استكشاف الأخطاء](#استكشاف-الأخطاء)

---

## نظرة عامة

### سيناريوهات الاستعادة

| السيناريو | الخطوات |
|-----------|---------|
| فقدان بيانات محدود | استعادة جدول/بيانات محددة |
| تلف قاعدة البيانات | استعادة آخر نسخة احتياطية |
| تلف السيرفر | إعادة بناء السيرفر من الصفر |
| كارثة كاملة | استعادة من Backblaze B2 |

### مصادر النسخ الاحتياطية

```
1. GitHub (يومياً)     → recepio-backups repo
2. Local (على السيرفر) → /opt/backups/
3. Backblaze B2        → recepio-server-backups bucket
4. USB Flash Drive     → Cold storage
```

---

## المتطلبات

### أدوات مطلوبة

| الأداة | الهدف |
|--------|-------|
| GPG | فك تشفير الملفات |
| PostgreSQL Client | استعادة DB |
| SSH | الوصول للسيرفر |
| Restic | استعادة من B2 |

### معلومات مطلوبة

```bash
# Server IP
SERVER_IP=63.181.126.178

# Database Connection
DB_HOST=localhost
DB_NAME=recepio
DB_USER=recepio
DB_PASSWORD=RecepioApp2026

# GitHub Repo
BACKUP_REPO=https://github.com/RecepioHQ/recepio-backups

# GPG Key
GPG_KEY_ID=recepio-backup@recepio.io
```

---

## استعادة قاعدة البيانات

### الطريقة 1: من GitHub (الموصى بها)

```bash
# 1. استنساخ مستودع النسخ الاحتياطية
git clone https://github.com/RecepioHQ/recepio-backups.git
cd recepio-backups

# 2. استيراد مفتاح GPG
gpg --import private-key.asc
# أو من GitHub Secret
echo "$GPG_PRIVATE_KEY" | gpg --import

# 3. فك تشفير آخر نسخة
LATEST_BACKUP=$(ls -t encrypted/db-*.gpg | head -1)
gpg --decrypt --output db-restore.sql.gz "$LATEST_BACKUP"

# 4. فك الضغط
gunzip db-restore.sql.gz

# 5. استعادة قاعدة البيانات
# إذا كانت DB موجودة:
PGPASSWORD='RecepioApp2026' psql -h localhost -U recepio -d recepio < db-restore.sql

# أو إذا تحتاج إنشاء DB جديدة:
PGPASSWORD='RecepioApp2026' psql -h localhost -U postgres -c "DROP DATABASE IF EXISTS recepio;"
PGPASSWORD='RecepioApp2026' psql -h localhost -U postgres -c "CREATE DATABASE recepio;"
PGPASSWORD='RecepioApp2026' psql -h localhost -U recepio -d recepio < db-restore.sql

# 6. التحقق
PGPASSWORD='RecepioApp2026' psql -h localhost -U recepio -d recepio -c "SELECT COUNT(*) FROM clinics;"
```

### الطريقة 2: من السيرفر مباشرة

```bash
# SSH للسيرفر
ssh ubuntu@63.181.126.178

# فك التشفير والاستعادة
cd /opt/recepio-backups
./scripts/decrypt-backup.sh encrypted/db-2024-01-15.sql.gz.gpg
./scripts/restore-backup.sh /tmp/recepio-decrypted/db-2024-01-15.sql.gz
```

### الطريقة 3: من Backblaze B2

```bash
# إعداد Restic
export RESTIC_REPOSITORY=b2:recepio-backups-archives
export RESTIC_PASSWORD="your-restic-password"
export B2_ACCOUNT_ID="your-b2-id"
export B2_ACCOUNT_KEY="your-b2-key"

# قائمة اللقطات
restic snapshots

# استعادة لقطة معينة
restic restore latest --target /tmp/restore

# فك التشفير والاستعادة
cd /tmp/restore
./scripts/restore-backup.sh encrypted/db-*.gpg
```

---

## استعادة الملفات

### استعادة .env

```bash
# من GitHub
git clone https://github.com/RecepioHQ/recepio-backups.git
cd recepio-backups

# فك التشفير
gpg --decrypt --output env-restore.tar.gz encrypted/env-*.tar.gz.gpg

# فك الضغط
tar -xzf env-restore.tar.gz

# نسخ إلى السيرفر
scp recepio/.env ubuntu@63.181.126.178:/opt/recepio-api-v2/.env

# إعادة تشغيل API
ssh ubuntu@63.181.126.178 "docker restart recepio-api"
```

### استعادة Nginx Configs

```bash
# فك التشفير
gpg --decrypt --output nginx-restore.tar.gz encrypted/nginx-*.tar.gz.gpg

# فك الضغط ونسخ
tar -xzf nginx-restore.tar.gz -C /tmp/
sudo cp -r /tmp/etc/nginx/* /etc/nginx/

# إعادة تشغيل Nginx
ssh ubuntu@63.181.126.178 "sudo nginx -t && sudo systemctl reload nginx"
```

---

## استعادة السيرفر كاملاً

### الخطوة 1: إنشاء سيرفر جديد

```bash
# AWS Console:
# 1. Launch Instance
# 2. Ubuntu 22.04 LTS
# 3. t3.large
# 4. 100GB GP3
# 5. Security Group: recepio-secure-sg
```

### الخطوة 2: إعداد السيرفر

```bash
# SSH
ssh ubuntu@NEW_SERVER_IP

# تحديث
sudo apt update && sudo apt upgrade -y

# تثبيت Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker ubuntu

# تثبيت أدوات
sudo apt install -y git gnupg postgresql-client
```

### الخطوة 3: استعادة Docker Containers

```bash
# استنساخ repo
git clone https://github.com/RecepioHQ/recepio-api-v2.git /opt/recepio-api-v2

# استعادة .env
# (من GitHub Secrets)

# تشغيل containers
cd /opt/recepio-api-v2
docker compose up -d
```

### الخطوة 4: استعادة قاعدة البيانات

```bash
# استعادة من النسخة الاحتياطية (انظر выше)
./restore-backup.sh encrypted/db-latest.sql.gz.gpg
```

### الخطوة 5: التحقق

```bash
# التحقق من الصحة
curl http://localhost:3001/healthz
curl https://api.recepio.io/healthz
```

---

## التحقق من الاستعادة

### قائمة التحقق

```bash
# 1. تحقق من الاتصال
curl https://api.recepio.io/healthz
# Expected: {"status":"ok","version":"2.0.0"}

# 2. تحقق من قاعدة البيانات
PGPASSWORD='RecepioApp2026' psql -h localhost -U recepio -d recepio -c "
SELECT 
  (SELECT COUNT(*) FROM clinics) as clinics,
  (SELECT COUNT(*) FROM clinic_users) as Users,
  (SELECT COUNT(*) FROM conversations) as conversations,
  (SELECT COUNT(*) FROM bookings) as bookings;
"

# 3. تحقق من Redis
redis-cli ping
# Expected: PONG

# 4. تحقق من Docker
docker ps
# Expected: recepio-api, recepio-postgres, recepio-redis

# 5. تحقق من SSL
curl -k https://api.recepio.io/healthz
# Expected: No SSL errors

# 6. تحقق من تسجيل الدخول
curl -X POST https://api.recepio.io/api/v1/auth/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@recepio.io","password":"TestPass123!"}'
# Expected: {"access_token":"...","refresh_token":"..."}
```

---

## استكشاف الأخطاء

### الخطأ: GPG decryption failed

```bash
# تأكد من الاستيراد الصحيح
gpg --list-keys

# إذا كان المفتاح غير موجود
gpg --import private-key.asc

# تأكد من الثقة
gpg --edit-key recepio-backup@recepio.io
gpg> trust
gpg> 5 (ultimate trust)
gpg> save
```

### الخطأ: Database connection refused

```bash
# تحقق من أن PostgreSQL يعمل
docker ps | grep postgres

# تحقق من السجلات
docker logs recepio-postgres

# تحقق من كلمة المرور
docker exec -it recepio-postgres psql -U postgres -c "\du"
```

### الخطأ: Backup file corrupted

```bash
# تحقق من سلامة الملف
gpg --verify backup.sql.gz.gpg

# إذا كان تالفاً
# استخدم نسخة سابقة
ls -la encrypted/db-*.gpg | sort -k10 | tail -5

# أو استعادة من B2
restic restore latest --target /tmp/restore
```

### الخطأ: Out of disk space

```bash
# تحقق من المساحة
df -h

# نظف Docker
docker system prune -a

# نظف النسخ الاحتياطية القديمة
find /opt/backups -name "*.gz" -mtim +7 -delete
```

---

## 📞 جهات الاتصال للطوارئ

| الحالة | الإجراء |
|--------|---------|
| فقدان بيانات عاجل | Telegram: @Tariq990 |
| فشل الاستعادة | GitHub Issues |
| خطر أمني | حدّث كل كلمات المرور فوراً |

---

## 📊 جدول الصيانة

| المهمة | التكرار |
|--------|---------|
| تحقق من النسخ الاحتياطية | أسبوعياً |
| اختبار الاستعادة | شهرياً |
| تحديث التوثيق | عند التغيير |
| تبديل كلمات المرور | كل 90 يوم |

---

**أنشئت:** 2026-07-19
**الإصدار:** 1.0.0
