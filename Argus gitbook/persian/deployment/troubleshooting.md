# عیب‌یابی

## وضعیت سرویس‌ها

```bash
argus-cli service status
argus-cli logs --last 30
```

پیام‌های سالم در ژورنال:

```
argus: sensor attached 21/21 hooks (0 skipped)
argus: shield protecting pid <N>
argus: daemon active (pid=<N>)
```

---

## پیام‌های کرنل سپر

```bash
sudo dmesg | grep -i "argus shield"
```

---

## مسیر لاگ

```bash
sudo ls -l <ARGUS_STORE>/$(date -u -d '+12600 seconds' '+%Y/%B')/
```

روزهای گذشته به `.log.gz` تبدیل شده‌اند. برای خواندن دستی:

```bash
sudo zcat <ARGUS_STORE>/2026/September/30.log.gz | tail -5
```

> تاریخ در نام پوشه با **وقت ایران (UTC+3:30)** حساب می‌شود، نه UTC.

---

## مشکلات رایج

### سپر بار نمی‌شود

```bash
ls /etc/argus/remove.key          # باید وجود داشته باشد
sudo dmesg | tail -5              # دلیل رد شدن را می‌گوید
```

سپر **بدون** `remove.key` عمداً بار نمی‌شود. برای ساختش:

```bash
sudo argus-cli set-password
sudo modprobe argus_shield
```

### `insmod: File exists`

ماژول از قبل بار است. اول آزاد کنید:

```bash
sudo argus-cli release
```

### نصب: `Operation not permitted`

باینری قدیمی مهر immutable دارد:

```bash
sudo chattr -i /usr/local/bin/argus-cli
sudo ./install.sh
```

### دیمن بالا نمی‌آید یا گیر کرده

```bash
argus-cli service status
sudo dmesg | grep -i 'argus shield' | tail -5
```

اگر `argus-cli service status` یک فرآیند دیمن اضافه نشان داد (یتیم):

```bash
sudo argus-cli service restart
```

### `verify` بلوک‌های رمزشده را رد می‌کند

اگر `ENCRYPT=1` است، `verify` را با `sudo` اجرا کنید (salt فقط root خواندنی است):

```bash
sudo argus-cli verify
```

اگر بدون root اجرا کنید، تعداد بلوک‌های خوانده‌نشده گزارش می‌شود:

```
◆ 237 ENCRYPTED BLOCK(S) SKIPPED
```

### دیسک پر شده / هشدار Storage

```bash
argus-cli status | grep STORAGE
sudo grep -A2 'Storage' /var/log/argus_alerts.log | tail -6
sudo du -sh <ARGUS_STORE>/
```

سیاست پیش‌فرض: **هیچ‌چیز حذف نمی‌شود، فقط فشرده.** اگر باز هم جا کم آمد:

```bash
argus-cli retention                      # سیاست فعلی
sudo argus-cli retention mode archive    # انتقال به رسانهٔ بیرونی
sudo argus-cli retention archive-path /mnt/backup
```

(آرشیوهای مهرشده را قبل از جابجایی دستی با `chattr -i` باز کنید.)

### سپر مسلح است ولی نمی‌توانم سرویس را ری‌استارت کنم

باید کار کند. `systemctl restart` از PID 1 می‌آید و رد نمی‌شود. اگر قفل شده:

```bash
sudo argus-cli service restart
argus-cli logs --last 20
```

### `kill -9` روی دیمن خطا نمی‌دهد ولی دیمن نمی‌میرد

این **رفتار درست** است. سپر سیگنال را بی‌اثر می‌کند بدون اینکه خطا برگرداند — عمدی، تا
مهاجم نشانه‌ای نبیند.

---

## آمار زندهٔ زنجیره

```bash
sudo tail -f <ARGUS_STORE>/2026/October/01.log | cut -c1-160
```

---

## اگر همه‌چیز شکست خورد

مسیر بازگشت همیشه باز است:

```bash
sudo argus-cli release             # سپر آزاد + دیمن متوقف
sudo argus-cli service restart
```

هیچ سناریویی وجود ندارد که فقط با ریبوت باز شود.
