# راهنمای جامع سیستم Arch Linux USB

این راهنما شامل تمام ابزارها و قابلیت‌های نصب شده در سیستم Arch Linux USB شما است.

## 📑 فهرست مطالب
1. [سیستم ایمنی و بازیابی](#1-سیستم-ایمنی-و-بازیابی)
2. [سیستم به‌روزرسانی اتمی](#2-سیستم-به‌روزرسانی-اتمی)
3. [بهینه‌سازی‌های پیشرفته](#3-بهینه‌سازی‌های-پیشرفته)

## 1. سیستم ایمنی و بازیابی

### 1.1. مدیریت همگام‌سازی (Sync)
```bash
# اجرای دستی همگام‌سازی اجباری
enforced-sync

# فعال/غیرفعال کردن همگام‌سازی دوره‌ای
systemctl enable periodic-sync.timer    # فعال‌سازی
systemctl disable periodic-sync.timer   # غیرفعال‌سازی
systemctl status periodic-sync.timer    # مشاهده وضعیت
```

### 1.2. مانیتورینگ سلامت I/O
```bash
# مشاهده وضعیت مانیتورینگ
systemctl status io-health-monitor.service

# بررسی لاگ‌های سلامت
tail -f /var/log/io-health.log
```

### 1.3. مدیریت Snapshot و بازیابی
```bash
# ایجاد دستی snapshot
create-system-snapshot

# مدیریت سرویس snapshot خودکار
systemctl enable system-snapshot.timer     # فعال‌سازی
systemctl disable system-snapshot.timer    # غیرفعال‌سازی
systemctl status system-snapshot.timer     # مشاهده وضعیت

# مشاهده snapshots موجود
ls -l /persistent/snapshots/
```

### 1.4. تله‌متری و مانیتورینگ
```bash
# مشاهده متریک‌های عملکردی
cat /persistent/telemetry/performance-metrics.csv

# مدیریت سرویس تله‌متری
systemctl status performance-telemetry.timer
systemctl enable performance-telemetry.timer
systemctl disable performance-telemetry.timer
```

### 1.5. تشخیص قطع برق
```bash
# مشاهده وضعیت سرویس
systemctl status power-failure-detector.service

# بررسی لاگ‌های مرتبط
tail -f /var/log/power-events.log
```

## 2. سیستم به‌روزرسانی اتمی

### 2.1. مدیریت به‌روزرسانی
```bash
# به‌روزرسانی سیستم
atomic-update-manager update-system

# بازگشت به نسخه قبلی
atomic-update-manager rollback

# مشاهده وضعیت
atomic-update-manager status
```

### 2.2. مدیریت Snapshot‌های اتمی
```bash
# ایجاد snapshot قبل از به‌روزرسانی
atomic-snapshot pre-update

# نمایش اطلاعات snapshot‌ها
atomic-snapshot info

# چرخش دستی snapshot‌ها
atomic-snapshot rotate

# تنظیم تعداد حداکثر snapshot‌ها
atomic-snapshot set-max 5
```

### 2.3. بازیابی از Snapshot
```bash
# مشاهده لیست snapshot‌ها
atomic-recovery list

# بازیابی از snapshot
atomic-recovery recover /persistent/snapshots/atomic-snapshot-20251022-120000.tar.gz
```

## 3. بهینه‌سازی‌های پیشرفته

### 3.1. مدیریت ZSWAP
```bash
# پیکربندی مجدد ZSWAP
/usr/local/bin/configure-zswap

# مشاهده وضعیت فعلی
cat /proc/sys/vm/zswap*
cat /sys/module/zswap/parameters/*

# مدیریت سرویس
systemctl status configure-zswap.service
```

### 3.2. مدیریت Bcachefs
```bash
# اجرای بهینه‌سازی‌ها
/usr/local/bin/optimize-bcachefs

# مشاهده وضعیت
bcachefs fs show /dev/disk/by-label/ARCH_PERSIST
```

### 3.3. مدیریت Prefetch
```bash
# اجرای دستی
smart-prefetch on-login    # در زمان ورود
smart-prefetch periodic    # دوره‌ای

# مدیریت سرویس
systemctl status smart-prefetch.timer
systemctl enable smart-prefetch.timer
systemctl disable smart-prefetch.timer
```

### 3.4. پروفایل‌های سخت‌افزاری
```bash
# اجرای مجدد تشخیص و اعمال
hardware-profile-manager

# مشاهده پروفایل فعلی
cat /etc/hardware-profiles/current

# مدیریت سرویس
systemctl status hardware-profile.service
```

### 3.5. بهینه‌سازی I/O
```bash
# اجرای دستی بهینه‌سازی‌ها
advanced-write-optimizer

# مشاهده تنظیمات فعلی
cat /proc/sys/vm/dirty_*
cat /sys/block/sd*/queue/scheduler

# مدیریت سرویس
systemctl status io-optimizer.service
```

##  نکات مهم در مورد کرنل و درایورها

### مدیریت نسخه کرنل
سیستم به طور هوشمند تلاش می‌کند نسخه کرنل نصب شده با محیط Live هماهنگ باشد:
```bash
# نمایش نسخه کرنل فعلی
uname -r

# بررسی نسخه کرنل نصب شده
pacman -Q linux

# بررسی وضعیت ماژول‌های کرنل
lsmod
```

در صورت نیاز به تغییر نسخه کرنل:
1. از محیط Live با نسخه کرنل مورد نظر بوت کنید
2. اسکریپت نصب را اجرا کنید
3. تأیید کنید که می‌خواهید نسخه کرنل محیط Live نصب شود

### راه‌اندازی مجدد mkinitcpio
در صورت نیاز به بازسازی initramfs:
```bash
# برای نسخه کرنل خاص
mkinitcpio --kernel <kernel-version> -P

# برای همه نسخه‌های نصب شده
mkinitcpio -P
```

##  گزینه‌های بوت GRUB

سیستم شما دارای گزینه‌های بوت متنوعی است:

1. **Arch Linux USB (Automatic Profile)**
   - انتخاب خودکار پروفایل بر اساس RAM

2. **Arch Linux USB (Low Resource Mode - 2GB RAM)**
   - بهینه‌شده برای سیستم‌های با RAM کم

3. **Arch Linux USB (Medium Resource Mode - 2-8GB RAM)**
   - تنظیمات متعادل برای سیستم‌های معمولی

4. **Arch Linux USB (High Resource Mode - 8GB+ RAM)**
   - عملکرد بالا برای سیستم‌های قوی

5. **Arch Linux USB (Safe Mode)**
   - حالت امن با حداقل درایورها

6. **Arch Linux USB (Recovery Mode - Read Only)**
   - حالت بازیابی با فایل‌سیستم فقط-خواندنی

7. **Arch Linux USB (Snapshot Recovery)**
   - بازیابی از snapshot

## ⚙️ مدیریت کلی سیستم

### بررسی وضعیت تمام سرویس‌ها
```bash
# مشاهده وضعیت همه سرویس‌های بهینه‌سازی
systemctl status configure-zswap.service
systemctl status bcachefs-optimize.service
systemctl status smart-prefetch.timer
systemctl status hardware-profile.service
systemctl status io-optimizer.service
systemctl status final-optimizations.service
```

### بررسی لاگ‌های سیستم
```bash
# لاگ‌های سیستمی
journalctl -xe

# لاگ‌های به‌روزرسانی
tail -f /var/log/atomic-updates.log

# لاگ‌های بهینه‌سازی
tail -f /var/log/io-health.log
tail -f /var/log/prefetch.log
```

---
🔍 **نکته**: همیشه قبل از انجام تغییرات مهم، یک snapshot ایجاد کنید.