# ضمیمه

مراجع تکمیلی برای جست‌وجوی سریع.

---

## صفحات این بخش

| صفحه | موضوع |
|---|---|
| [واژه‌نامه](glossary.md) | اصطلاحات فنی |
| [مرجع مسیرها](paths.md) | مسیر همهٔ فایل‌ها و باینری‌ها |
| [مجوز](license.md) | تفکیک GPL و اختصاصی |
| [تست‌ها](testing.md) | مجموعهٔ تست خودکار |
| [گزارش تست](test-report.md) | نتیجهٔ آخرین کمپین تست کامل |

---

## مرجع سریع دستورها

```bash
# ساخت و نصب
make
sudo ./install.sh

# روزمره
argus-cli status
argus-cli count
argus-cli list --external
argus-cli sensitive
argus-cli verify

# نگهداشت و حالت
argus-cli mode
argus-cli retention

# امنیت
sudo argus-cli release
sudo argus-cli remove

# تست
sudo tests/run_all.sh
```
