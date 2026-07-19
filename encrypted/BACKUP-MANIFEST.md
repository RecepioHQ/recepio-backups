# 📦 Full Backup Manifest - 2026-07-19

## Encrypted Files (في encrypted/)

| الملف | الحجم | المحتوى |
|-------|-------|---------|
| `database_full_20260719.sql.gz.gpg` | ~7 KB | قاعدة البيانات كاملة |
| `docker-compose.prod.yml.gpg` | ~2 KB | Docker Compose الإنتاج |
| `server-configs_20260719.tar.gz.gpg` | ~3 KB | Nginx + Systemd configs |
| `recepio_20260719.sql.gz.gpg` | ~7 KB | نسخة سابقة |

## Unencrypted Configs (في configs/)

| المجلد | المحتوى |
|--------|---------|
| `docker/` | Docker Compose (بدون أسرار) |
| `nginx/` | Nginx configs (بدون SSL certs) |
| `systemd/` | Systemd services |
| `env.example` | هيكل .env (بدون قيم) |

## للاستعادة

```bash
# فك التشفير
gpg --decrypt database_full_20260719.sql.gz.gpg | gunzip > database.sql
gpg --decrypt docker-compose.prod.yml.gpg > docker-compose.prod.yml
gpg --decrypt server-configs_20260719.tar.gz.gpg | tar -xzf -
```

## ملاحظات

- القيم الحقيقية محفوظة في GitHub Secrets
- GPG Private Key في GitHub Secrets + USB آمن
- SSL Certificates يتم تجديدها من Let's Encrypt
