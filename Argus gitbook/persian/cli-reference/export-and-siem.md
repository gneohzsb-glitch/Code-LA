# export و SIEM

## `argus-cli export`

خروجی بلوک‌های یک روز در `$HOME`:

```bash
argus-cli export --format json
argus-cli export --format csv
argus-cli export --format csv --from 09/30/2026
```

| گزینه | کار |
|---|---|
| `--format json` | خروجی JSON |
| `--format csv` | خروجی CSV (با ستون `flags`) |
| `--from MM/DD/YYYY` | یک روز مشخص |

---

## ستون `flags` در CSV

خروجی CSV یک ستون `flags` واقعی دارد که برای **مسیریابی در SIEM** استفاده می‌شود:

| مقدار | معنی |
|---|---|
| `1` | رویداد مسیر حساس (`[SENSITIVE]`) |
| `2` | خلاصهٔ تجمیع با شمارش بالای سقف (ناهنجاری) |
| `0` | رویداد عادی |

> **تاریخچه:** پیش‌تر سرصفحه هشت‌ستونه بود ولی بدنه خط JSON خام می‌نوشت، پس سرصفحه و
> سطرها هیچ‌وقت نمی‌خواندند. این در v15.1 رفع شد.

---

## فوروارد زنده

علاوه بر خروجی دستی، دیمن رویدادها را به‌صورت زنده به یک مقصد می‌فرستد. مقصد در
`/etc/argus/export.conf` و **بدون ری‌استارت** قابل تغییر است (config watcher آن را
دوباره می‌خواند).

### مسیرهای syslog (RFC5424)

```ini
DESTINATION=FILE:/var/log/argus_export.log
DESTINATION=TCP:siem.example.com:514
DESTINATION=UDP:siem.example.com:514
```

رویدادهای مسیر حساس با تگ **`ARGUS_SENSITIVE`** و اولویت هشدار (`<12>`) ارسال می‌شوند،
تا بدون تجزیهٔ JSON قابل مسیریابی باشند.

### صادرکننده‌های JSON (v15.4 — Phase 6)

سه مقصد HTTP-محور که هر بلوک را به‌صورت یک **POST با بدنهٔ JSON** می‌فرستند
(libcurl؛ در صورت `https://` روی TLS):

```ini
# Webhook عمومی
DESTINATION=HTTP:https://collector.example.com/ingest
TOKEN=<bearer-token>          # اختیاری → Authorization: Bearer

# Splunk HEC
DESTINATION=SPLUNK:https://splunk.example.com:8088/services/collector/event
TOKEN=<hec-token>             # → Authorization: Splunk <token>

# Elasticsearch
DESTINATION=ELASTIC:https://es.example.com:9200/argus/_doc
TOKEN=<api-key>               # اختیاری → Authorization: ApiKey

TLS_VERIFY=1                  # 0 فقط برای گواهی self-signed آزمایشگاهی
```

| مقصد | قالب بدنه |
|---|---|
| `HTTP:` | `{"host":..,"ts":..,"event":<block>}` |
| `SPLUNK:` | `{"time":..,"host":..,"sourcetype":"argus","event":<block>}` |
| `ELASTIC:` | `{"@timestamp":"<RFC3339>","host":{"name":..},"event":<block>}` |

`<block>` همان شیء JSON زنجیره است (پس همان چیزی که هش شده به SIEM می‌رسد). اگر یک POST
شکست بخورد، دیمن متوقف نمی‌شود و بلوک بعدی دوباره تلاش می‌کند.

---

## لنگر بیرونی (timestamp anchoring)

سر زنجیره به‌طور دوره‌ای به مقصد بیرونی فرستاده می‌شود (`/etc/argus/anchor.conf`).
اگر بعداً کسی کل زنجیره را بازنویسی کند، هش لنگرشده دیگر با سر زنجیره نمی‌خواند.

```ini
INTERVAL=300
DESTINATION=FILE:/var/log/argus_anchor.log
FORMAT=JSON
```

---

## نمونهٔ یکپارچه‌سازی

### ارسال به یک SIEM مبتنی بر syslog

```ini
DESTINATION=TCP:siem.internal:601
```

### استخراج رویدادهای حساس از CSV

```bash
argus-cli export --format csv --from 09/30/2026 \
  | awk -F, 'NR==1 || $<flags_col>==1'
```

### استخراج allowlist از دادهٔ واقعی

```bash
argus-cli export --format csv --from 09/01/2026 \
  | awk -F, '{print $<comm>","$<path>}' | sort | uniq -c | sort -rn | head -20
```

سپس هر جفت را بررسی کنید و اگر واقعاً یک daemon است که با خودش حرف می‌زند، به
`allowlist.conf` اضافه کنید.

---

## انواع بلوک در خروجی

SIEM می‌تواند انواع بلوک را با تگ مسیریابی کند:

| نوع بلوک | منبع |
|---|---|
| رویداد عادی | هر syscall ثبت‌شده |
| `AGGREGATE` | خلاصهٔ تجمیع |
| `AGGREGATE-RECOVERED` | بازیابی sidecar یتیم |
| `CONFIG` | تغییر allowlist/حالت/پنجره |
| `MODE_CHANGE` | تغییر حالت فیلتر |
| `MAINTENANCE` | اقدام نگهداشت |

جزئیات در [زنجیرهٔ هش](../architecture/blockchain.md).
