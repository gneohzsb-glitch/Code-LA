# تست‌ها

ARGUS یک مجموعهٔ تست خودکار دارد که با یک دستور اجرا می‌شود:

```bash
cd ~/argus_v15/tests
sudo ./run_all.sh
```

`run_all.sh` ابتدا هر نصب در حال اجرا را از مسیر رمزدار آزادسازی پایین می‌آورد (مجموعه‌های
سپر به کنترل انحصاری ماژول کرنل نیاز دارند)، همهٔ مجموعه‌ها را به ترتیب درست اجرا می‌کند، و
در پایان رمز، ماژول و دیمن را برمی‌گرداند. **هیچ ریبوتی در کار نیست — هرگز.**

> رمز حذف را می‌توان با `ARGUS_PW=<password>` بازنویسی کرد.

مجموعه به‌ازای هر ادعا یک خط `PASS`/`FAIL` چاپ می‌کند و در پایان یا `ALL SUITES PASSED` را
نشان می‌دهد یا `FAILURES PRESENT`. اگر هر مجموعه‌ای شکست بخورد، کد خروج غیرصفر می‌شود؛ پس
مستقیماً در CI قابل استفاده است.

---

## پوشش مجموعه

**۴۴ مجموعه** و **۶۳۱ ادعا** وجود دارد. در ادامه بر اساس بخشی از محصول که هر گروه
می‌آزماید، دسته‌بندی شده‌اند.

### سپر و ماژول کرنل

| مجموعه | چه چیزی را اثبات می‌کند |
|---|---|
| `test_shield.sh` | کشتن دیمن، `rmmod`، `rmmod -f`، و مسیر آزادسازی با رمز |
| `test_shield_reduction.sh` | سپر دقیقاً ۴ هوک نصب می‌کند — نه بیشتر |
| `test_cli_shield.sh` | تعامل CLI ↔ سپر و گزارش وضعیت |

### یکپارچگی و زنجیرهٔ هش

| مجموعه | چه چیزی را اثبات می‌کند |
|---|---|
| `test_verify.sh` | ویرایش، دوباره‌لینک، و حذف، هر سه کشف می‌شوند |
| `test_aggregation.sh` | تجمیع، بازیابی پس از crash، و تغییر حالت |
| `test_timestamps.sh` | دقت timestamp بلوک |
| `test_anchor_hmac.sh` | HMAC لنگر روزانه، و اینکه تغییر کلید/بازه بدون ری‌استارت اعمال می‌شود |

### رفتار حسگر

| مجموعه | چه چیزی را اثبات می‌کند |
|---|---|
| `test_no_flood.sh` | ضد فیدبک: ARGUS هرگز خروجی خودش را ثبت نمی‌کند |
| `test_noise_filter.sh` | فیلتر نویز و پرچم مسیر حساس |
| `test_list_filters.sh` | `list --external` بدون سقف، و `sensitive` |
| `test_encryption.sh` | رمزنگاری لاگ: رمزگشایی شفاف و سازگاری |
| `test_gcm.sh` | کدک AES-GCM هر ویرایشی در ciphertext، tag یا طول را رد می‌کند |
| `test_retention.sh` | فشرده‌سازی بی‌اتلاف، مهر immutable، و حالت‌های نگهداشت |

### دسترسی، مجوزها و پیکربندی

| مجموعه | چه چیزی را اثبات می‌کند |
|---|---|
| `test_permissions.sh` | هر فایلی که محصول می‌سازد، حالت و مالک درست را دارد |
| `test_password_kdf.sh` | رمز حذف به‌صورت هش کند KDF ذخیره می‌شود، نه متن ساده |
| `test_group_access.sh` | اعضای گروه `argus` می‌توانند بدون `sudo` شواهد را بخوانند |
| `test_sudo_hint.sh` | CLI به‌جای رد خشک، راهنمای `sudo`/گروه چاپ می‌کند |
| `test_machine_id_stable.sh` | `machine.id` در بازبیلدها پایدار می‌ماند |
| `test_release_no_ssh_kill.sh` | مسیر آزادسازی هرگز نشست SSH اپراتور را نمی‌کشد |
| `test_config_reread.sh` | تغییرات پیکربندی زنده خوانده می‌شوند |
| `test_upgrade_preserves_config.sh` | به‌روزرسانی، پیکربندی اپراتور را حفظ می‌کند |
| `test_version.sh` | یک رشتهٔ نسخهٔ واحد در همه‌جا گزارش می‌شود |

### هشدارها و شواهد

| مجموعه | چه چیزی را اثبات می‌کند |
|---|---|
| `test_alerts_chain.sh` | هشدارها در زنجیره ثبت می‌شوند و از ری‌استارت جان سالم به‌در می‌برند |
| `test_evidence_native.sh` | قالب شواهد بومی خودتوصیف و دست‌نخورده است |
| `test_no_decoy_leak.sh` | مطالب پوششی/فریب هرگز محتوای واقعی را لو نمی‌دهند |
| `test_exporters.sh` | صادرکننده‌های SIEM رکوردهای خوش‌ساخت منتشر می‌کنند |
| `test_remove_preserves_keys.sh` | `remove` کلیدها را نگه می‌دارد (مخرب؛ آخر از همه اجرا می‌شود) |

### مجوز و بیلدر

| مجموعه | چه چیزی را اثبات می‌کند |
|---|---|
| `test_argus_id.sh` | `argus-id` شناسهٔ ماشین را درست استخراج می‌کند |
| `test_license_valid.sh` | مجوز معتبر تأیید می‌شود |
| `test_license_expired.sh` | مجوز منقضی رد می‌شود |
| `test_license_tampered.sh` | payload یا امضای دست‌کاری‌شده رد می‌شود |
| `test_license_wrong_machine.sh` | مجوز بسته‌شده به ماشین دیگر رد می‌شود |
| `test_license_replay.sh` | nonce بازپخش‌شده رد می‌شود |
| `test_license_key_rotation.sh` | چرخش کلید رعایت می‌شود |
| `test_license_grace_period.sh` | پنجرهٔ مهلت طبق طراحی رفتار می‌کند |
| `test_license_runtime_verify.sh` | دیمن در حال اجرا مجوز را دوباره تأیید می‌کند |
| `test_license_antitamper.sh` | اشکال‌زدا (debugger) متصل به verifier کشف و رد می‌شود |
| `test_builder.sh` | `argus-builder` یک بستهٔ مهرشده و قابل نصب می‌سازد |
| `test_license_integration.sh` | `install.sh --check-only` بستهٔ درست را می‌پذیرد |

### سازگاری و ساختار

| مجموعه | چه چیزی را اثبات می‌کند |
|---|---|
| `test_backward_compatibility.sh` | قالب‌های قدیمی روی دیسک همچنان تأیید می‌شوند |
| `test_source_separation.sh` | سورس GPL و اختصاصی از هم جدا می‌مانند |
| `test_capillary.sh` | لبه‌های آرگومان، پیکربندی خصمانه، CPU بیکار، و خاموشی سریع |

### بهداشت اطلاعات (v15.5)

| مجموعه | چه چیزی را اثبات می‌کند |
|---|---|
| `test_no_name_leak.sh` | هیچ خروجی اپراتوری، نام یونیت/باینری/مسیر داخلی را نمی‌گوید |
| `test_codes.sh` | شکست‌ها کد `ARG-nnn` دارند، و هر کد مستند شده است |

---

## نکات چند مجموعهٔ برگزیده

### `test_verify.sh`

یک زنجیرهٔ ساختگی برای یک تاریخ گذشته می‌سازد (پس به لاگ زنده دست نمی‌زند) و چهار حالت را
چک می‌کند:

| حالت | نتیجهٔ مورد انتظار |
|---|---|
| زنجیرهٔ سالم | `ALL N BLOCKS VERIFIED — NO BREAKS` |
| محتوا ویرایش‌شده | `BAD HASHES` |
| بلوک دوباره‌لینک‌شده | `BROKEN LINKS` |
| بلوک حذف‌شده از میانه | `BROKEN LINKS` |

### `test_retention.sh`

- فایل `retention.conf` نصب شده و پیش‌فرض `off` است
- `argus-cli retention` سیاست را گزارش می‌کند
- مقادیر نامعتبر (حالت ناشناخته، آستانهٔ ۰ یا ۱۵۰) رد می‌شوند
- یک مقدار نوشته‌شده در پیکربندی می‌ماند و بازگردانی می‌شود
- قواعد ایمنی در سورس دیمن وجود دارند: `off` پیش از هر حذف برمی‌گردد، پنجرهٔ ۷ روزه محفوظ
  است، و اقدام پیش از وقوع در زنجیره ثبت می‌شود

این تست هرگز حالت `auto`/`archive` را تنظیم نمی‌کند، پس هیچ دادهٔ واقعی در خطر نیست.

### `test_list_filters.sh`

- سرصفحهٔ سرشماری `list --external` حاضر است
- شمارش هر IP با مجموع گزارش‌شده می‌خواند
- مجموع درون پنجره‌ای مستقل از لاگ خام قرار می‌گیرد که دو طرفِ خواندن CLI گرفته شده است
  (لاگ زنده است، پس چک تساوی دقیق ذاتاً شکننده می‌بود)
- هیچ سقف ثابتی در سورس `cli.c` نمانده
- `sensitive` فقط بلوک‌های پرچم‌دار را نشان می‌دهد
- `list --sensitive` و `sensitive` با هم می‌خوانند

### `test_capillary.sh`

یک پاس عمداً خصمانه روی لبه‌ها:

- CLI روی آرگومان‌های خالی، بسیار بلند، یا بدشکل هرگز crash نمی‌کند
- یک پیکربندی خصمانه (`INTERVAL=0`) دیمن را قفل نمی‌کند
- CPU بیکار در یک بازهٔ اندازه‌گیری‌شده ثابت می‌ماند (بدون busy loop)
- `systemctl stop` سریع برمی‌گردد (بدون توقف در خاموشی)

### `test_license_antitamper.sh`

verifier مقدار `TracerPid` خودش را می‌خواند؛ اگر اشکال‌زدا یا tracer متصل باشد حالت
`TAMPERED` را بالا می‌آورد و مجوز مرگبار تلقی می‌شود. مجموعه تأیید می‌کند که اجرای پاک
`VALID` می‌ماند و اجراهای `strace`/`gdb` رد می‌شوند. این دفاع لایه‌ای است، نه تضمین — حد
واقعی را در ضمیمهٔ مجوز ببینید.

---

## اجرای تک‌تک

```bash
sudo ./test_shield.sh
sudo ./test_shield_reduction.sh
./test_cli_shield.sh
./test_verify.sh
./test_aggregation.sh          # دیمن خاموش
./test_no_flood.sh 20
./test_timestamps.sh
sudo ./test_retention.sh
sudo ./test_noise_filter.sh
sudo ./test_list_filters.sh
sudo ./test_encryption.sh
sudo ./test_gcm.sh
sudo ./test_capillary.sh
```

> برخی تست‌ها به دیمن در حال اجرا نیاز دارند و برخی به دیمن خاموش؛ `run_all.sh` خودش ترتیب
> را رعایت می‌کند. اجرای دستی یک مجموعه خارج از ترتیب ممکن است پیش‌شرطش را نقض کند.

---

## ساخت هارنس تست

بعضی تست‌ها یک هارنس C جداگانه می‌سازند که سورس واقعی را لینک می‌کند:

```bash
gcc -O2 -Wall -I userspace/daemon -I common \
    -o /tmp/argus_ret_test tests/ret_test.c \
    userspace/daemon/retention.c \
    userspace/daemon/alert_engine.c \
    userspace/daemon/storage.c \
    userspace/daemon/blockchain.c \
    userspace/daemon/hasher.c \
    userspace/daemon/ipcache.c \
    userspace/daemon/encryptor.c -lz -lpthread -lcrypto
```

هارنس نگهداشت، تابع واقعی `retention_compress_past_days()` را روی یک روز ساختگی اجرا می‌کند
و نتیجه را بازرسی می‌کند — بدون نیاز به ری‌استارت دیمن.

برای نتایج آخرین کمپین کامل، [گزارش تست](test-report.md) را ببینید.
