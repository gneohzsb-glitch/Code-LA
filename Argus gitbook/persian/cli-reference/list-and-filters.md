# list و فیلترها

## `argus-cli list`

پیش‌فرض: ۱۰۰ بلوک آخر.

```bash
argus-cli list                   # ۱۰۰ بلوک آخر
argus-cli list --last 20         # ۲۰ بلوک آخر
argus-cli list --from 09/30/2026 # یک روز مشخص
```

| گزینه | کار |
|---|---|
| `--last N` | N بلوک آخر |
| `--external` | سرشماری بلوک‌های دارای IP بیرونی (بدون سقف) |
| `--sensitive` | فقط رویدادهای مسیر حساس |
| `--from MM/DD/YYYY` | یک روز مشخص |
| `--no-color` | خاموش‌کردن رنگ |

بلوکی که مسیر حساسی باز کرده، نشان `[SENSITIVE]` می‌گیرد.

---

## `argus-cli list --external`

سرشماری کامل بلوک‌هایی که یک IP بیرونی دارند، **بدون سقف**، مرتب از پرترافیک‌ترین:

```bash
argus-cli list --external
```

```
  ◆ 12148 BLOCKS WITH EXTERNAL IP — 4 DISTINCT

  10.0.2.2:        6668 blocks
  185.125.188.55:  3164 blocks
  91.189.91.83:    168 blocks
  ...
```

> **تاریخچه:** پیش‌تر این دستور روی ۱۰۰۰۰ تطابق و ۲۵۶ IP یکتا قفل می‌شد و مجموع ناقص
> گزارش می‌کرد. اکنون در یک گذر، هر بلوک را در یک جدول درج باز می‌شمارد و گذر اضافهٔ
> `wc -l` را حذف می‌کند (روز ۱۰۹ مگابایتی ≈ ۰٫۶ ثانیه).

---

## `argus-cli sensitive`

فقط رویدادهای مسیر حساس، به‌صورت یک دستور مستقل:

```bash
argus-cli sensitive
argus-cli sensitive --last 10
argus-cli sensitive --from 09/30/2026
```

این معادل `argus-cli list --sensitive` است؛ هر دو از همان تابع مشترک استفاده می‌کنند و
خروجی یکسان می‌دهند.

«مسیر حساس» یعنی دسترسی به `/etc/shadow`، کلیدهای SSH، `/root/`، `/etc/argus`، درخت
ماژول، و نوشتن روی فایل‌های پیکربندی سیستم.

---

## `argus-cli block`

نمایش یک بلوک مشخص، یک بازه، یا یک فهرست:

```bash
argus-cli block 1                # یک بلوک
argus-cli block 5-10             # بازه
argus-cli block 1,3,7            # فهرست
argus-cli block 42 --date 09/30/2026
```

---

## `argus-cli tail`

نمایش زندهٔ بلوک‌های جدید:

```bash
argus-cli tail
argus-cli tail --from 09/30/2026
# Ctrl+C برای خروج
```

---

## خواندن روزهای فشرده

همهٔ دستورهای خواندن، روزهای فشرده (`.log.gz`) را هم می‌خوانند. `verify` روی یک روز
فشرده کار می‌کند و خودش گزارش می‌دهد که فایل فشرده است.

```bash
argus-cli list --from 09/30/2026    # اگر آن روز فشرده باشد، باز هم کار می‌کند
```

---

## مثال‌های کاربردی

```bash
# پرترافیک‌ترین IPهای بیرونی
argus-cli list --external | head -20

# همهٔ دسترسی‌ها به فایل‌های حساس امروز
argus-cli sensitive

# بلوک‌های یک روز مشخص
argus-cli list --from 09/30/2026 | head -50

# بازرسی یک بلوک خاص
argus-cli block 42
```
