# 🔐 Recepio Backup Repository

> **مستودع النسخ الاحتياطية المشفرة والآلية لـ Recepio**
> **آخر تحديث:** 2026-07-19

---

## ✅ الاختبار والتوثيق

### نتائج الاختبار - 19 يوليو 2026

| المرحلة | الحالة | التفاصيل |
|---------|--------|----------|
| SSH Connection | ✅ نجح | الاتصال بالسيرفر 63.181.126.178 |
| Database Backup | ✅ نجح | 42 KB - 17 جدول |
| Compression | ✅ نجح | gzip - 6.4 KB |
| Encryption | ✅ نجح | GPG RSA-4096 |
| GitHub Upload | ✅ نجح | 8.7 KB مرفوع |
| Download | ✅ نجح | تحميل من GitHub |
| Decryption | ✅ نجح | فك التشفير |
| Extraction | ✅ نجح | استخراج الملفات |
| Data Integrity | ✅ نجح | MD5 checksum متطابق |

### الملفات المستعادة:

```
/tmp/restored_files/
├── database.sql.gz  (6.4 KB) - قاعدة البيانات كاملة
├── nginx-api.conf   (3.3 KB) - تكوين API
├── nginx-app.conf   (757 B)  - تكوين App
└── nginx-admin.conf (759 B)  - تكوين Admin
```

---

## ⚠️ تحذير أمني هام

```
⛔ لا ترفع أبداً:
  - مفاتيح SSH الخاصة
  - كلمات مرور قاعدة البيانات
  - JWT secrets
  - API tokens
  - أي ملف .env بقيم حقيقية

✅ فقط الملفات المشفرة بـ GPG تُرفع
✅ القيم الحقيقية في GitHub Secrets
```

---

## 📁 هيكل المستودع

```
recepio-backups/
│
├── 📁 encrypted/              # النسخ الاحتياطية المشفرة
│   ├── database_full_20260719.sql.gz.gpg
│   ├── recepio_20260719.sql.gz.gpg
│   ├── docker-compose.prod.yml.gpg
│   ├── server-configs_20260719.tar.gz.gpg
│   ├── recepio-backup-public.gpg
│   └── BACKUP-MANIFEST.md
│
├── 📁 configs/                 # التكوينات (بدون أسرار)
│   ├── 📁 docker/
│   │   ├── docker-compose.recepio.yml
│   │   ├── docker-compose.monitoring.yml
│   │   └── docker-compose.chatwoot.yml
│   ├── 📁 nginx/
│   │   ├── nginx-api.recepio.io.conf
│   │   ├── nginx-app.recepio.io.conf
│   │   └── nginx-admin.recepio.io.conf
│   ├── 📁 systemd/
│   │   ├── recepio-backup.service
│   │   └── recepio-backup.timer
│   └── env.example             # هيكل .env بدون قيم
│
├── 📁 keys/                    # المفاتيح العامة
│   ├── recepio-backup-public.gpg
│   └── README.md
│
├── 📁 manual/                  # نسخ يدوية
│   └── 20260719_144216/
│       ├── backup_*.tar.gz.gpg
│       └── MANIFEST.md
│
├── 📁 hourly/                  # نسخ كل ساعة (أوتوماتيك)
│   └── YYYYMMDD_HH/
│
├── 📁 realtime/                # نسخ كل 5 دقائق
│   └── LATEST
│
└── 📁 .github/workflows/       # GitHub Actions
    ├── backup.yml              # يومي 2:00 UTC
    ├── backup-hourly.yml       # كل ساعة
    ├── backup-realtime.yml     # كل 5 دقائق
    ├── backup-manual.yml       # يدوي
    └── restore-secrets.yml     # نشر الأسرار للسيرفر
```

---

## 🔐 نظام التشفير

### GPG Key Information

```
Key ID:       5029865814F77EC5
Email:        backup@recepio.io
Fingerprint:  7964DD3DBF0E92F913CC4DCD5029865814F77EC5
Algorithm:    RSA 4096-bit
Created:      2026-07-19
Expires:      لا ينتهي (permanent)
```

### مكان المفاتيح

| المفتاح | المكان | الاستخدام |
|---------|--------|-----------|
| **المفتاح العام** | `keys/recepio-backup-public.gpg` | للتشفير في GitHub |
| **المفتاح الخاص** | GitHub Secrets (`GPG_PRIVATE_KEY`) | لفك التشفير في Actions |
| **نسخة احتياطية** | USB آمن (Cold Storage) | للطوارئ |

---

## 🔧 GitHub Secrets المطلوبة

### إدارة من CLI:

```bash
# عرض جميع الـ Secrets
gh secret list --repo RecepioHQ/recepio-backups

# إضافة Secret جديد
echo "SECRET_VALUE" | gh secret set SECRET_NAME --repo RecepioHQ/recepio-backups
```

### قائمة الـ Secrets (8):

| Secret | الغرض |
|--------|-------|
| `GPG_PRIVATE_KEY` | مفتاح GPG الخاص للتشفير |
| `SSH_PRIVATE_KEY` | مفتاح SSH للوصول للسيرفر |
| `DB_PASSWORD` | كلمة مرور PostgreSQL |
| `JWT_SECRET` | مفتاح توقيع JWT |
| `ENCRYPTION_KEY` | مفتاح Fernet للبيانات |
| `RESTIC_PASSWORD` | كلمة مرور Restic |
| `B2_ACCOUNT_ID` | معرف Backblaze B2 |
| `B2_ACCOUNT_KEY` | مفتاح Backblaze B2 |

---

## 🚀 أنواع النسخ الاحتياطية

### 1. النسخ اليدوي (Manual)

```bash
# تشغيل يدوي
gh workflow run backup-manual.yml --repo RecepioHQ/recepio-backups -f backup_type=full

# أو من GitHub UI:
# Actions → Manual Backup Push → Run workflow
```

**المحتوى:**
- قاعدة البيانات كاملة
- تكوينات Nginx (3 مواقع)
- تكوينات Docker Compose
- خدمات Systemd

### 2. النسخ كل ساعة (Hourly)

```yaml
# الجدولة: كل ساعة (0 * * * *)
# المحتوى: Database + Nginx + Systemd + Docker Compose
# الاحتفاظ: 24 ساعة
```

### 3. النسخ اللحظي (Realtime)

```yaml
# الجدولة: كل 5 دقائق (*/5 * * * *)
# المحتوى: قاعدة البيانات فقط
# الاحتفاظ: ساعة واحدة
```

### 4. النسخ اليومي (Daily)

```yaml
# الجدولة: كل يوم 2:00 UTC (0 2 * * *)
# المحتوى: نسخة كاملة
# الاحتفاظ: 30 يوم
```

---

## 📋 كيفية الاستعادة

### الطريقة 1: من GitHub Actions

1. اذهب إلى: https://github.com/RecepioHQ/recepio-backups/actions
2. اختر **"Deploy Secrets to Server"**
3. أدخل IP السيرفر الجديد
4. اضغط **"Run workflow"**

### الطريقة 2: يدوياً من GitHub

```bash
# 1. تحميل النسخة
gh api repos/RecepioHQ/recepio-backups/contents/manual/BACKUP_NAME/backup.tar.gz.gpg \
  --jq '.content' | base64 -d > backup.tar.gz.gpg

# 2. فك التشفير
gpg --decrypt --output backup.tar.gz backup.tar.gz.gpg

# 3. استخراج الملفات
tar -xzf backup.tar.gz

# 4. استعادة قاعدة البيانات
gunzip -c database.sql.gz | psql -U recepio -d recepio

# 5. نسخ تكوينات Nginx
sudo cp nginx-*.conf /etc/nginx/sites-available/
sudo nginx -t && sudo systemctl reload nginx
```

### الطريقة 3: من سيرفر جديد

```bash
# 1. استنساخ المستودع
git clone https://github.com/RecepioHQ/recepio-backups.git
cd recepio-backups

# 2. فك التشفير
gpg --decrypt manual/LATEST/backup.tar.gz.gpg | tar -xzf -

# 3. استاعدة قاعدة البيانات
gunzip -c database.sql.gz | sudo docker exec -i recepio-postgres psql -U recepio recepio
```

---

## 🔄 إذا غيرت السيرفر

### كل ما تحتاجه:
1. **GitHub Account** - للوصول للمستودع والـ Secrets
2. **مفتاح USB** - يحتوي مفتاح GPG الخاص (احتياطي)
3. **IP السيرفر الجديد**

### الخطوات:

```bash
# 1. تشغيل الـ workflow
gh workflow run restore-secrets.yml \
  --repo RecepioHQ/recepio-backups \
  -f server_ip=NEW_SERVER_IP \
  -f environment=production

# 2. أو يدوياً من GitHub:
# Actions → Deploy Secrets to Server → Run workflow
# أدخل IP الجديد

# 3. الـ workflow سيتولى:
# - نشر جميع الأسرار
# - استيراد مفتاح GPG
# - إعادة تشغيل الخدمات
```

---

## 🛡️ الأمان

### مستويات الحماية:

| المستوى | التقنية | الاستخدام |
|---------|---------|-----------|
| **التشفير** | GPG RSA-4096 + AES-256 | النسخ الاحتياطية |
| **النقل** | SSH + TLS | الاتصال بالسيرفر |
| **التخزين** | GitHub Secrets | الأسرار الحساسة |
| **الوصول** | GitHub Actions permissions | التحكم في التنفيذ |

### قواعد مهمة:

1. ✅ **النسخ مشفرة دائماً** قبل الرفع
2. ✅ **No secrets in code** - أبداً
3. ✅ **الـ Secrets محمية** في GitHub
4. ✅ **مفتاح GPG الخاص** في مكان آمن خارج النظام
5. ✅ **Workflow permissions** محددة (contents: write)

---

## 📞 الطوارئ

### في حالة الفشل:

```
1. تحقق من GitHub Actions logs
2. تأكد من SSH connection
3. تحقق من GPG key valid
4. راجع GitHub Secrets
```

### استعادة كاملة:

```bash
# من USB
gpg --import /media/usb/private-key.asc

# تحميل آخر نسخة
gh api repos/RecepioHQ/recepio-backups/contents/manual \
  --jq '.[0].name'

# فك التشفير والاستعادة
gpg --decrypt backup.tar.gz.gpg | tar -xzf -
```

---

## 🔗 الروابط المهمة

| الرابط | الوصف |
|--------|-------|
| https://github.com/RecepioHQ/recepio-backups | المستودع |
| https://github.com/RecepioHQ/recepio-backups/actions | GitHub Actions |
| https://github.com/RecepioHQ/recepio-backups/settings/secrets/actions | GitHub Secrets |
| http://63.181.126.178 | السيرفر |

---

## 📊 الإحصائيات

```
المستودعات:        4 (recepio-backups, recepio-app, server-bootstrap, server-backup)
الـ Secrets:       8
الـ Workflows:     5
النسخ الاحتياطية:  ~10/day
التكرار:           كل 5 دقائق / كل ساعة / يومياً
التشفير:           GPG RSA-4096
```

---

## 📝 سجل التغييرات

### 2026-07-19
- ✅ إنشاء مستودع GitHub
- ✅ إضافة نظام النسخ الاحتياطي
- ✅ إعداد GPG Key RSA-4096
- ✅ إضافة 8 GitHub Secrets
- ✅ إنشاء 5 Workflows
- ✅ اختبار كامل - ناجح 100%
- ✅ توثيق شامل

---

**أنشئت:** 2026-07-19  
**الإصدار:** 2.0.0  
**آخر اختبار:** 2026-07-19 17:44 UTC  
**الحالة:** ✅ يعمل بشكل كامل
