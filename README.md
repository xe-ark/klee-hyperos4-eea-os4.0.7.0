# POCO X8 Pro (`klee`) — HyperOS 4.0 EEA (`OS4.0.7.0.XPJEUXM`) Custom Fastboot ROM

Bu depo, **POCO X8 Pro / Redmi Turbo 5 (`klee`, MT6899)** cihazı için **HyperOS 4.0 EEA (`OS4.0.7.0.XPJEUXM`, Android 17 / SDK 37, Linux `6.6.118`)** tabanlı özelleştirilmiş Fastboot ROM paketinin dokümantasyonunu, debloat listesini ve düzeltilmiş **OrangeFox R12.0 (`vendor_boot`)** yapılandırma notlarını içerir.

---

## Yapılan Değişiklikler

### 1. Sistem & Framework Yamaları
- **`framework.jar` / `services.jar` / `miui-services.jar`**: İmzalı/yama uyumlu sistem servis düzenlemeleri.
- **Native SBC XQ (Bluetooth Yüksek Kalite SBC) + `Settings.apk` Fix**:
  - `Settings.apk` (`classes3.dex` / DEX `040`) sıfırdan temiz derlenerek `SetupWizard` / `Ayarlar` durduruldu hatası giderildi.
  - `/system/lib64/sbc_xq` → `/apex/com.android.bt/lib64` overlay'i aktif edildi.
- **Sistem Başlatıcı & Ekran Görüntüsü Enjeksiyonu (`/product` + `system_ext`)**:
  - Stok `MiLauncherGlobal` (POCO Launcher) kaldırıldı.
  - **`HyperOS4_RustLauncherMod_v1.4`**: `MiuiHome.apk` (`com.miui.home`) ve `HyperLauncherExt.apk` (`com.miui.home.kashi.ext`) doğrudan `/product/priv-app/` altına, ilgili `privapp-permissions` XML dosyalarıyla enjekte edildi.
  - `system_ext/etc/init/init.miui.ext.rc` içinde `ro.product.brand=POCO` için `ro.miui.product.home` değeri `"com.miui.home"` olarak güncellendi.
  - **`HyperOS_ScreenshotV1.1`**: `MIUIScreenshot.apk` `/product/app/MIUIScreenshot/` altına enjekte edildi (`oat/` temizlendi).
- **Diğer Sistem Modülleri**:
  - `FreeStyle_CC_mod_V1`
  - `HyperOS_SecurityV8`
  - `HyperOS_ThemeManagerV7`
- **Debloat**: Toplam **58 adet** sistem/bloat uygulaması kaldırıldı (tam liste: `debloat_listesi.txt`).

### 2. Boot, Root & Bootloader Güvenliği
- **`init_boot.img`**: KernelSU Next gömülü (`kernelsu` modülü aktif, `u:r:ksu:s0`).
- **`lk.img` & `preloader.img`**: `lk_fenrir.bin` ve `preloader_fenrir.bin` (⚠️ **Asla stok `lk.img` veya `preloader_raw.img` flaşlamayın**).
- **`vbmeta.img`**: `Flags: 1` (`verification` & `verity` devre dışı).

### 3. Düzeltilmiş OrangeFox R12.0 Recovery (`vendor_boot.img`)
`OrangeFox-R12.0-klee-OS4.0.7-fixed.img` imajı stok `OS4.0.7.0` `vendor_ramdisk00` ve `dtb` korunarak `vendor_ramdisk01` üzerinde şu hatalar giderilerek hazırlanmıştır:
- **KeyMint `-33` / `/data` Şifre Çözme Fixi**: `prop.default` sürüm ve güvenlik yaması değerleri ROM ile eşitlendi (`ro.build.version.release=17`, `ro.build.version.security_patch=2026-10-01`, `ro.vendor.build.security_patch=2026-10-01`, `ro.vendor.build.version.release=15`). Metadata + User 0 Weaver FBE sorunsuz çözülür.
- **`/persist` Yazılabilirliği**: `twrp.flags` ve `init.recovery.keymint.rc` üzerindeki `ro,noload` kilidi kaldırıldı (`wipe` bayrağı eklenmeden!). OrangeFox ayarları (`/persist/.foxs`) kalıcı kaydedilir.
- **`fastbootd` & USB ConfigFS Fixi**: `init.rc` / `init.recovery.usb.rc` çakışması giderildi, `UDC "none"` temizliği eklendi; `prep-fastbootd.sh` ile `fastbootd` öncesi tüm dinamik `super` bölümleri unmount/unmap edilir ve eski `dm-*` symlinkleri temizlenir (`system_ext_a`, `odm_a`, `*_dlkm_a` dahil 9/9 bölüm çalışır).
- **`init.recovery.mt6899.rc`**: Eksik `import /init.recovery.project.rc` eklendi; `twrp.flags` içine `/init_boot`, `/mi_ext_image`, `/odm_image` eklendi; `/usb_otg` internal UFS LUN0 (`sda`) yerine `sdd`/`sde` yapıldı.

---

## İndirme Bağlantıları (Google Drive)
- **Fastboot ROM (`klee_eea_OS4.0.7.0_fastboot.zip`, `7.046.286.390` B / `~6.6 GB`)**:
  - https://drive.google.com/file/d/1T8byCaVyNgYRp-qwIlWojcxU2OwECowK/view?usp=drivesdk
- **Düzeltilmiş OrangeFox R12.0 (`OrangeFox-R12.0-klee-OS4.0.7-fixed.img`, `67.108.864` B)**:
  - https://drive.google.com/file/d/1EljlFDioPf1hHIndN5QNHJvRrfnfQKI8/view?usp=drivesdk
- **Geliştiriciler için Saf Recovery Ramdisk & Ayrı OrangeFox Reposu**:
  - GitHub: https://github.com/xe-ark/klee-orangefox-r12
  - Google Drive (`ofox_r12_klee_recovery_ramdisk.lz4`): https://drive.google.com/file/d/1ht35j7uJd6fcH4CFkghyIy1wE_30X-nA/view?usp=drivesdk

---

## İmajlar & Dosya Özeti
- **`klee_eea_OS4.0.7.0_fastboot.zip`**: Güncellenmiş `super.img` (`8.414.505.048` B, `lpmake --virtual-ab --sparse`), `vendor_boot.img` (düzeltilmiş OrangeFox), `init_boot.img` (KernelSU Next), `lk.img`, `preloader.img` ve tüm fastboot imajlarını içerir.
- **`OrangeFox-R12.0-klee-OS4.0.7-fixed.img`**: Gelecek sürümlerde doğrudan kullanılabilecek bağımsız düzeltilmiş OrangeFox imajı (`67.108.864` B).
