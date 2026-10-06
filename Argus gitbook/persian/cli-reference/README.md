# مرجع CLI

`argus-cli` رابط اپراتور ARGUS است. بیشتر دستورها **بدون `sudo`** کار می‌کنند؛ آن‌هایی
که فایل‌های محافظت‌شده را می‌خوانند یا می‌نویسند، به `sudo` نیاز دارند.

---

## فهرست دستورها

| دستور | کار |
|---|---|
| [`status`](status-and-stats.md) | وضعیت سپر، دیمن، رمز، مصرف دیسک |
| [`count`](status-and-stats.md) | تعداد بلوک‌ها: امروز/هفته/ماه/کل |
| [`stats`](status-and-stats.md) | تعداد بلوک‌های امروز |
| [`list`](list-and-filters.md) | ۱۰۰ بلوک آخر؛ با فیلترها |
| [`sensitive`](list-and-filters.md) | فقط رویدادهای مسیر حساس |
| [`block`](list-and-filters.md) | نمایش یک بلوک/بازه/فهرست |
| [`tail`](list-and-filters.md) | نمایش زنده |
| [`verify`](verify-and-integrity.md) | تأیید زنجیره (لینک + هش) |
| [`integrity`](verify-and-integrity.md) | گواهی یکپارچگی برای یک بازه |
| [`anomalies`](verify-and-integrity.md) | هشدارهای امروز |
| [`export`](export-and-siem.md) | خروجی JSON/CSV |
| [`mode`](mode-and-retention.md) | حالت فیلتر تجمیع |
| [`retention`](mode-and-retention.md) | سیاست نگهداشت |
| [`set-password`](shield-and-password.md) | تنظیم رمز حذف |
| [`change-password`](shield-and-password.md) | تغییر رمز حذف |
| [`release`](shield-and-password.md) | آزادسازی سپر (بدون حذف) |
| [`remove`](shield-and-password.md) | حذف کامل (لاگ می‌ماند) |
| `help <command>` | راهنمای تفصیلی یک دستور |

---

## راهنمای درون‌خطی

```bash
argus-cli help            # فهرست کامل
argus-cli help list       # راهنمای دستور list
argus-cli help retention  # راهنمای دستور retention
```

---

## گزینه‌های عمومی

| گزینه | کار |
|---|---|
| `--no-color` | خاموش‌کردن رنگ ANSI |
| `--date MM/DD/YYYY` | محدودکردن به یک روز |
| `--from MM/DD/YYYY` | از یک تاریخ |
| `--last N` | N بلوک آخر |

**متغیر محیطی:** `NO_COLOR=1` هم رنگ‌ها را خاموش می‌کند (استاندارد رایج). رنگ به‌طور
خودکار در پایپ (غیر TTY) و در `TERM=dumb` خاموش می‌شود، تا اسکریپت‌ها متن ساده ببینند.

---

## تکمیل خودکار

اسکریپت تکمیل bash در `/etc/bash_completion.d/argus-cli` نصب می‌شود. در یک شل در حال
اجرا:

```bash
source /etc/bash_completion.d/argus-cli
```

سپس `argus-cli <Tab>` دستورها و `argus-cli list --<Tab>` گزینه‌ها را پیشنهاد می‌دهد.
