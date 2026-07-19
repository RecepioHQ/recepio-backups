# 🔐 Recepio Backup Repository

> **مستودع النسخ الاحتياطية المشفرة لـ Recepio**

---

## ⚠️ تحذير أمني هام

```
⛔ لا ترفع أبداً:
  - مفاتيح SSH الخاصة
  - كلمات مرور قاعدة البيانات
  - JWT secrets
  - API tokens
  - أي ملف .env

✅ فقط الملفات المشفرة بـ GPG/AGE تُرفع
```

---

## 📁 هيكل المستودع

```
recepio-backups/
├── encrypted/          # النسخ الاحتياطية المشفرة
│   ├── db-YYYY-MM-DD.sql.gz.gpg
│   ├── env-encrypted.enc
│   └── keys-backup.tar.gz.gpg
│
├── keys/              # المفاتيح العامة فقط
│   ├── public-key.asc
│   └── backup-key.pub
│
├── scripts/           # سكربتات الباكب
│   ├── encrypt-backup.sh
│   ├── decrypt-backup.sh
│   └── restore-backup.sh
│
├── reports/           # تقارير الباكب
│   └── backup-report-*.json
│
└── logs/              # سجلات الباكب
    └── backup-*.log
```

---

## 🔧 المتطلبات

### على السيرفر:
```bash
# تثبيت GPG
sudo apt install gnupg

# تثبيت age (بديل أحدث)
sudo apt install age

# تثبيت restic (للنسخ الاحتياطية المتقدمة)
sudo apt install restic
```

### على الجهاز المحلي:
```bash
# GPG for Windows
# حمّل من: https://gpg4win.org/

# أو استخدم Git Bash المدمج
```

---

## 🚀 الاستخدام

### 1. إنشاء مفتاح GPG

```bash
# إنشاء مفتاح جديد
gpg --full-generate-key

# اختر:
# - RSA and RSA
# - 4096 bits
# - expiration: 1y
# - name: Recepio Backup Key

# تصدير المفتاح العام
gpg --armor --export recepio-backup@recepio.io > keys/public-key.asc

# تصدير المفتاح الخاص (احفظه في مكان آمن جداً!)
gpg --armor --export-secret-keys recepio-backup@recepio.io > private-key.asc
```

### 2. إنشاء نسخة احتياطية مشفرة

```bash
# من السيرفر
./scripts/encrypt-backup.sh
```

### 3. فك التشفير والاستعادة

```bash
# فك التشفير
./scripts/decrypt-backup.sh encrypted/db-2024-01-15.sql.gz.gpg

# الاستعادة
./scripts/restore-backup.sh decrypted/db-2024-01-15.sql.gz
```

---

## 📋 جدول النسخ الاحتياطية

| النوع | التكرار | الاحتفاظ |
|-------|---------|----------|
| قاعدة البيانات | يومياً | 30 يوم |
| ملفات .env | عند التغيير | غير محدود |
| مفاتيح SSH | عند التغيير | غير محدود |
| تكوين Docker | عند التغيير | غير محدود |

---

## 🔄 سير العمل

### على السيرفر (كل ليلة):
```bash
# 1. نسخة احتياطية من قاعدة البيانات
pg_dump recepio | gzip > /tmp/db-backup.sql.gz

# 2. تشفير النسخة
gpg --encrypt --recipient recepio-backup@recepio.io \
    --output encrypted/db-$(date +%Y-%m-%d).sql.gz.gpg \
    /tmp/db-backup.sql.gz

# 3. رفع إلى GitHub
git add encrypted/*.gpg
git commit -m "backup: db $(date +%Y-%m-%d)"
git push
```

---

## 🛡️ الأمان

### قواعد التشفير:
1. **GPG Encryption**: AES-256 + RSA-4096
2. **Symmetric Encryption**: age-encryption
3. **Key Rotation**: كل 90 يوم

### مكان تخزين المفاتيح الخاصة:
- USB محمي بكلمة مرور (Primary)
- Safe deposit box (Secondary)
- **لا تُخزن أبداً على السيرفر أو GitHub**

---

## 📞 الطوارئ

في حالة فقدان الوصول:

1. استخدم المفتاح الخاص من USB
2. استعد آخر نسخة من GitHub
3. فك التشفير واستعد قاعدة البيانات

---

## 🔗 الروابط

- **GitHub Repo**: https://github.com/RecepioHQ/recepio-backups
- **Server**: 63.181.126.178
- **API**: https://api.recepio.io

---

**أنشئت:** 2026-07-19
**الإصدار:** 1.0.0
