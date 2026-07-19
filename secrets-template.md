# 🔐 قائمة المفاتيح والأسرار - Recepio

> **آخر تحديث:** 2026-07-19
> **إصدار:** 1.0.0

---

## ⚠️ تحذير أمني هام

```
⛔ هذا الملف للتوثيق فقط!
⛔ لا تحفظ القيم الحقيقية هنا أبداً!
⛔ استخدم GitHub Secrets لتخزين القيم الحقيقية
```

---

## 📊 جدول المفاتيح

| الاسم | الغرض | التخزين | ملاحظات |
|-------|-------|---------|---------|
| `SSH_PRIVATE_KEY` | مفتاح SSH للسيرفر | GitHub Secret | مفتاح خاص |
| `GPG_PRIVATE_KEY` | مفتاح GPG للتشفير | GitHub Secret + USB | غير مشفر في GitHub |
| `DB_PASSWORD` | كلمة مرور PostgreSQL | GitHub Secret | |
| `JWT_SECRET` | مفتاح توقيع JWT | GitHub Secret | 64+ حرف عشوائي |
| `ENCRYPTION_KEY` | مفتاح تشفير البيانات | GitHub Secret | Fernet key |
| `RESTIC_PASSWORD` | كلمة مرور Restic | GitHub Secret | |
| `B2_ACCOUNT_ID` | معرف Backblaze B2 | GitHub Secret | |
| `B2_ACCOUNT_KEY` | مفتاح Backblaze B2 | GitHub Secret | |

---

## 🔧 كيفية إضافة Secrets في GitHub

### 1. الذهاب إلى Settings
```
Repository → Settings → Secrets and variables → Actions
```

### 2. إضافة Secret جديد
```
Click "New repository secret"
Name: SECRET_NAME
Value: secret_value_here
Click "Add secret"
```

---

## 📋 قائمة الـ GitHub Secrets المطلوبة

### Database
```yaml
SECRET_NAME: DB_PASSWORD
DESCRIPTION: كلمة مرور PostgreSQL
VALUE: RecepioApp2026 (غيّر!)
```

### Security
```yaml
SECRET_NAME: JWT_SECRET
DESCRIPTION: مفتاح توقيع JWT tokens
VALUE: (أدخل مفتاح عشوائي 64+ حرف)
```

```yaml
SECRET_NAME: ENCRYPTION_KEY
DESCRIPTION: مفتاح تشفير Fernet للبيانات الحساسة
VALUE: (أدخل Fernet key من: python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())")
```

```yaml
SECRET_NAME: GPG_PRIVATE_KEY
DESCRIPTION: المفتاح الخاص بـ GPG للتشفير
VALUE: (أدخل ناتج: gpg --armor --export-secret-keys recepio-backup@recepio.io)
```

### Backup
```yaml
SECRET_NAME: RESTIC_PASSWORD
DESCRIPTION: كلمة مرور مستودع Restic
VALUE: (أدخل كلمة مرور قوية)
```

```yaml
SECRET_NAME: B2_ACCOUNT_ID
DESCRIPTION: معرف حساب Backblaze B2
VALUE: (من Backblaze B2 dashboard)
```

```yaml
SECRET_NAME: B2_ACCOUNT_KEY
DESCRIPTION: مفتاح تطبيق Backblaze B2
VALUE: (من Backblaze B2 dashboard)
```

### Server Access
```yaml
SECRET_NAME: SSH_PRIVATE_KEY
DESCRIPTION: مفتاح SSH الخاص للوصول للسيرفر
VALUE: (محتوى ملف recepio-secure-key.pem)
```

---

## 🗝️ تخزين المفاتيح المحلية

### 1. USB Flash Drive (المفضل)
```
1. أشّر USB في الكمبيوتر
2. انسخ المجلد: recepio-backups/private/
3. استخدم VeraCrypt لتشفير USB
4. احفظ في مكان آمن بعيداً عن الكمبيوتر
```

### 2. Safe Deposit Box
```
- انسخ المفاتيح إلى USB آخر
- خزنه في Safe Deposit Box بالبنك
- حدّث كل 90 يوم
```

### 3. Paper Backup (Cold Storage)
```
- اطبع المفاتيح على ورق
- استخدم QR codes للنسخ السريع
- خزن في خزنة آمنة
```

---

## 🔄 دورة تبديل المفاتيح

| المفتاح | الدورة |
|---------|--------|
| JWT_SECRET | كل 90 يوم |
| ENCRYPTION_KEY | كل 180 يوم |
| DB_PASSWORD | كل 180 يوم |
| GPG Private Key | كل 365 يوم |
| SSH Key | كل 365 يوم |
| API Keys | حسب الطلب |

---

## 📞 جهات الاتصال للطوارئ

| الدور | الاسم | التطبيق |
|-------|-------|---------|
| المالك | Tariq990 | Telegram |
| Infrastructure | ZCode Agent | GitHub |

---

## 🔐 مفتاح GPG العام

```
Current Key ID: 5029865814F77EC5
Email: backup@recepio.io
Fingerprint: 7964DD3DBF0E92F913CC4DCD5029865814F77EC5
Algorithm: RSA 4096
Created: 2026-07-19
Expires: لا ينتهي (permanent)
```

### الملفات
- **المفتاح العام:** `keys/recepio-backup-public.gpg`
- **المفتاح الخاص:** محفوظ في `Desktop/recepio-backup-private-20260719.gpg` (للنقل إلى USB)

---

## 📝 تعليمات إنشاء GPG Key

```bash
# 1. إنشاء مفتاح جديد
gpg --full-generate-key
# اختر: RSA and RSA, 4096 bits, 1 year expiration
# Name: Recepio Backup
# Email: recepio-backup@recepio.io

# 2. تصدير المفتاح العام
gpg --armor --export recepio-backup@recepio.io > keys/public-key.asc

# 3. تصدير المفتاح الخاص (لـ USB)
gpg --armor --export-secret-keys recepio-backup@recepio.io > private-key.asc

# 4. نسخ إلى USB
cp private-key.asc /media/usb/secure/

# 5. حذف من الكمبيوتر
shred -u private-key.asc
```

---

## ✨ أفضل الممارسات

1. **لا تخزن المفاتيح في الكود أبداً**
2. **استخدم GitHub Secrets للقيم الحساسة**
3. **غيّر كلمات المرور دورياً**
4. **احتفظ بنسخ احتياطية خارجية**
5. **استخدم مصادقة ثنائية على GitHub**
6. **راقب الوصول عبر Audit Logs**

---

**أنشئت بواسطة:** ZCode Agent
**آخر تحديث:** 2026-07-19
