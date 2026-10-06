# نصب

## پیش‌نیاز

[الزامات](requirements.md) را برآورده کنید. سپس:

---

## گام ۱ — ساخت

```bash
cd ~/argus_v15
make clean
make
```

هفت فایل ساخته می‌شود:

| فایل | نقش |
|---|---|
| `build/argus_id` | ابزار اثر انگشت ماشین |
| `build/camouflage_engine` | ابزار ساخت درخت طعمه |
| `kernel_module/argus_shield.ko` | ماژول سپر |
| `kernel_module/argus_sensor.bpf.o` | حسگر eBPF |
| `userspace/argus-cli` | CLI اپراتور |
| سایر باینری‌های `userspace/` | دیمن و resolver (نامشان داخلی است) |

---

## گام ۲ — نصب

```bash
sudo ./install.sh
```

نصب‌کننده این کارها را انجام می‌دهد:

| مرحله | چه چیزی |
|---|---|
| ۱ | باینری‌های کاربرفضا → `/usr/local/bin/` و **strip** شدن |
| ۲ | حسگر eBPF → `/usr/local/share/argus/` |
| ۳ | سورس‌های GPL → `/usr/local/share/argus/src/` |
| ۴ | فایل‌های پیکربندی → `/etc/argus/` |
| ۵ | `machine.id` (اثر انگشت ماشین) |
| ۶ | `embedded_salt` (مهر immutable، مجوز 600) |
| ۷ | درخت طعمه (اختیاری، با `ARGUS_DECOYS=1`) |
| ۸ | ماژول سپر → `/lib/modules/.../argus/` و بارگذاری |
| ۹ | واحدهای systemd + تکمیل خودکار bash |

---

## گام ۳ — اگر رمز حذف تنظیم نشده باشد

اگر `remove.key` نباشد، این پیام را می‌بینید:

```
[!] no removal password set, so the shield was NOT loaded.
```

این **عمدی** است: سپر بدون مسیر خروج هرگز مسلح نمی‌شود (وگرنه فقط با ریبوت پاک می‌شد).
دستورهای ادامه را همان‌جا چاپ می‌کند:

```bash
sudo argus-cli set-password      # رمز را ۱۰ بار وارد کنید
sudo modprobe argus_shield
sudo argus-cli service restart
```

---

## رفتار پیکربندی: نصب اول در برابر ارتقا

سه فایل پیکربندی **حالت اپراتور** هستند و در ارتقا **بازنویسی نمی‌شوند**:

| فایل | رفتار |
|---|---|
| `/etc/argus/mode` | فقط اگر نباشد ساخته می‌شود |
| `/etc/argus/allowlist.conf` | فقط اگر نباشد ساخته می‌شود |
| `/etc/argus/retention.conf` | فقط اگر نباشد ساخته می‌شود |

سه فایل دیگر (`export.conf`, `anchor.conf`, `alert.conf`) با هر نصب به‌روز می‌شوند.

منطق: ارتقا نباید بی‌سروصدا حالت فیلتر یا allowlistی که اپراتور از دادهٔ واقعی استخراج
کرده را پاک کند.

---

## گام ۴ — تأیید نصب

```bash
argus-cli status
```

خروجی مورد انتظار:

```
  SHIELD:    ◆ ACTIVE
  RELEASE:   ◆ locked
  DAEMON:    ◆ <pid>
  LICENSE:   ◆ ...
  PASSWORD:  ◆ SET
  STORAGE:   ◆ <n>% used
```

سپس صحت زنجیره:

```bash
sudo argus-cli verify
# ALL <N> BLOCKS VERIFIED — NO BREAKS
```

---

## بررسی‌های پس از نصب

```bash
# سرویس‌ها فعالند؟
argus-cli service status

# ماژول بار شده؟
lsmod | grep argus_shield

# حسگر همهٔ هوک‌ها را attach کرد؟
argus-cli logs --last 20 | grep hooks
# argus: sensor attached 21/21 hooks (0 skipped)
```

---

## تکمیل خودکار bash

نصب‌کننده یک فایل تکمیل bash به `/etc/bash_completion.d/argus-cli` می‌گذارد که در
پوسته‌های جدید خودکار منبع می‌شود. با زدن `argus-cli ` و فشردن `Tab`، زیردستورها
و پرچم‌های معتبر پیشنهاد می‌شوند:

```
$ argus-cli <Tab>
block        count        help         list         mode         retention    ...
$ argus-cli list --<Tab>
--external  --from      --last     --sensitive  --no-color
$ argus-cli retention mode <Tab>
archive  auto  off
```

اگر در نشست جاری فعال نشد:

```bash
source /etc/bash_completion.d/argus-cli
```

تکمیل فقط پرچم‌ها و آرگومان‌هایی را پیشنهاد می‌دهد که CLI واقعاً می‌پذیرد، پس یک
`Tab` سرگردان هرگز گزینه‌ای نامعتبر پیشنهاد نمی‌دهد.

---

## عیب‌یابی نصب

| مشکل | راه‌حل |
|---|---|
| `insmod: File exists` | ماژول از قبل بار است؛ اول `sudo argus-cli release` |
| سپر بار نمی‌شود | `ls /etc/argus/remove.key`؛ بدون آن عمداً بار نمی‌شود |
| `Operation not permitted` هنگام نصب | باینری قدیمی مهر immutable دارد؛ `sudo chattr -i /usr/local/bin/argus-cli` |
| دیمن بالا نمی‌آید | `argus-cli logs --last 30` |

بیشتر در [عیب‌یابی](troubleshooting.md).
