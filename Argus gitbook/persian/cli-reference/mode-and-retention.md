# mode و retention

## `argus-cli mode` — حالت فیلتر تجمیع

```bash
argus-cli mode                   # نمایش حالت فعلی و اندازهٔ پنجره
sudo argus-cli mode balanced     # performance | balanced | forensic
```

```
  ◆ FILTER MODE

  mode: balanced   (window 300s)
```

| حالت | پنجره | معنی |
|---|---|---|
| `performance` | ۳۶۰۰s | تجمیع تهاجمی |
| `balanced` (پیش‌فرض) | ۳۰۰s | تجمیع عادی |
| `forensic` | ۰ | بدون تجمیع — هر رویداد فردی |

**نکات مهم:**

- هیچ حالتی **ثبت را خاموش نمی‌کند** — فقط اندازهٔ پنجرهٔ تجمیع را عوض می‌کند.
- تغییر حالت خودش یک بلوک در زنجیره (`MODE_CHANGE`) و یک هشدار تولید می‌کند.
- دیمن درجای پنج ثانیه آن را در کار می‌گیرد (**بدون ری‌استارت**).
- تجمیع فقط وقتی رخ می‌دهد که یک قاعده در `/etc/argus/allowlist.conf` مسیر را نام برده
  باشد؛ پیش‌فرض خالی است.

---

## `argus-cli retention` — سیاست نگهداشت

```bash
argus-cli retention                        # نمایش سیاست فعلی
sudo argus-cli retention mode off          # off | archive | auto
sudo argus-cli retention threshold 80      # ۱..۹۹
sudo argus-cli retention archive-path /mnt/backup
```

```
  ◆ RETENTION POLICY

  mode:       off
  threshold:  85% full
  act on:     20% of eligible archives
  archive to: (unset)

  off      compress + seal only — nothing is ever removed (default)
  archive  move the oldest days to an external path — nothing destroyed
  auto     delete the oldest days — irreversible
  The current day and the last 7 days are always protected. Writing needs sudo.
```

| دستور | کار |
|---|---|
| `retention` | نمایش سیاست |
| `retention mode off\|archive\|auto` | تعیین حالت |
| `retention threshold N` | درصد دیسکی که حالت اقدام می‌کند (۱..۹۹) |
| `retention archive-path P` | مقصد حالت `archive` |

| حالت | رفتار |
|---|---|
| `off` (پیش‌فرض) | فشرده + مهر. هیچ‌چیز حذف نمی‌شود |
| `archive` | قدیمی‌ترین N٪ را به مسیر بیرونی منتقل می‌کند |
| `auto` | قدیمی‌ترین N٪ را حذف می‌کند (غیرقابل‌بازگشت) |

**قواعد ایمنی (همیشه):** امروز هرگز دست نمی‌خورد، ۷ روز آخر محفوظ است، و اقدام **پیش از
وقوع** در زنجیره ثبت می‌شود.

جزئیات کامل در [نگهداشت](../deployment/retention.md).

---

## تفاوت `mode` و `retention`

| | `mode` | `retention` |
|---|---|---|
| چه چیزی را کنترل می‌کند | تجمیع رویداد (حجم لاگ) | نگهداشت دیسک (فشرده/انتقال/حذف) |
| مقیاس زمانی | پنجرهٔ ثانیه‌ای | گذر ۱۰ دقیقه‌ای |
| خطر از دست رفتن داده | ندارد | `auto` دارد |
| فایل پیکربندی | `/etc/argus/mode` | `/etc/argus/retention.conf` |
| اعمال بدون ری‌استارت | بله (≤۵ ثانیه) | بله (≤۱۰ دقیقه) |
