# Samsung Galaxy W (ancora / GT-I8150) — Android 8.1

Hedef: **LineageOS 15.1**, yalnızca ARM32, mevcut bölüm düzeni ve
`/system/vendor` kullanan, Treble olmayan bir yükseltme cihazı.

**Durum: kaynak ağacı ön uyarlaması. Derlenmiş veya telefonda doğrulanmış ROM değildir.**
Bu paket flashlanabilir ZIP değildir. `boot.img`, `system.img`, kernel veya recovery
imajı içermez. Tam Android kaynakları ve kernel verilmediğinden ROM derlemesi
yapılmadı. Aşağıdaki engeller çözülmeden bu ağaçtan çalışan Oreo beklenmemelidir.

## Yapılan değişiklikler

- CM ürün tanımı yerine `lineage_ancora` ve LineageOS ortak telefon yapılandırması.
- Oreo derleyicisinin tanıdığı `cortex-a8` CPU varyantı; ARMv7-A/NEON ve 32 bit
  Binder korunuyor. `scorpion`, bu derleyicinin CPU varyant adı değildir.
- `dlmalloc` seçimi yerine düşük bellek yapılandırması ve Android Go varsayılanları.
- Android 8.1 varsayılan HAL sarmalayıcıları için ürün modülleri ve VINTF manifesti.
  Bunlar eski HAL'lerin ABI'sini veya davranışını otomatik dönüştürmez.
- Vendor `.so` dosyaları için gerçek prebuilt modüller; `obj/lib` kopyalama ve
  kamera makefile'ındaki elle `export_includes` oluşturma kaldırıldı.
- Modül adlarında platformla çakışmayı azaltan `ancora_blob_` öneki kullanıldı;
  kurulan dosya adları korunuyor. Kameranın bağlandığı `libseccameraadaptor` ve
  sesin bağlandığı `libaudioalsa` normal modül adlarını koruyor.
- Eski `rild` ve `libril.so` dosyaları korunuyor fakat kurulum listesinde değiller.
  Oreo'nun radyo arayüzünü sağlayamadıkları için radio HIDL ilan edilmiyor.
- Eski FUSE/sdcard ve dhcpcd servisleri çıkarıldı; depolamayı Oreo vold yönetiyor.
  Ayrı `/charger` servisi kullanılıyor. Otomatik `setup_fs` servisi kaldırıldı.
- Framework overlay'leri Oreo kaynak adlarına göre daraltıldı. Donanım tuşları
  Lineage SDK overlay yoluna taşındı; eski storage_list/Mms/CM ayarları çıkarıldı.
- CM12 SELinux kuralları arşivlendi; Oreo için başlangıç domain ve dosya etiketleri
  eklendi. Enforcing veya neverallow kontrolleri kapatılmadı. Politika tamamlanmadı.
- Orijinal 55 vendor dosyası değişmeden korundu. SHA-256 listesi ve ELF bağımlılık
  dökümü eklendi. Bölüm boyutları ve fstab blok aygıtları orijinal ağaçtan korundu.

## Derlemeyi ve çalışmayı engelleyen işler

| Bileşen | Gereken çalışma |
| --- | --- |
| Kernel | `kernel/samsung/msm7x30-common` ve `ancora_defconfig` yok. Kernel sürümü/config'i bu arşivlerden doğrulanamaz. Oreo için Binder/hwbinder/vndbinder, SELinux, seccomp/TSYNC, senkronizasyon, ağ ve FUSE gereksinimleri kaynak üzerinde değerlendirilmeli, gereken geri taşımalar yapılmalı. |
| Grafik | `gralloc.msm7x30`, `hwcomposer.msm7x30`, `copybit.msm7x30`, `memtrack.msm7x30`, `libgenlock` ve C2D sağlayan uyarlanmış msm7x30 kaynakları yok. Oreo uyumlu display-legacy dalı ve `project-path-for,qcom-display` eşlemesi gerekli. Adreno 205 blob'larının Bionic/libui sembolleri ayrıca kontrol edilmeli. HWC1 adaptörünün bu HAL sürümüyle çalıştığı doğrulanmalı. |
| Kamera | `cameraHAL.cpp` hâlâ kaldırılmış `ui/legacy/Overlay.h` API'sini kullanıyor. Framework tarafında ABI uyumlu Overlay/MemoryHeapPmem/CameraParameters desteği veya gerçek bir HAL yeniden yazımı gerekiyor. Sadece header kopyalamak ABI uyumluluğu sağlamaz. `TARGET_HAS_LEGACY_CAMERA_HAL1` bu eksikleri çözmez. |
| Telefon/modem | CM12 `SamsungRIL.java`, Oreo RIL API'sine uygun değil. Eski daemon/libril ise radio HIDL sunmuyor. Samsung protokolünü Oreo native RIL + radio HIDL'ye bağlayan gerçek bir uyarlama gerekiyor. Mevcut paketle arama/SMS/mobil veri beklenmemeli. |
| Ses | Legacy ALSA `audio.primary.msm7x30` ve yardımcı modüller arşivlerde yok. Oreo HAL API'sine uyarlanmış kaynak gerekli. `.conf` audio policy kullanımı seçildi; kayıt, çalma ve görüşme yolları ayrıca denenmeli. |
| Video | `libOmxCore`, `libOmxVdec`, `libOmxVenc`, `libmm-omxcore`, `libstagefrighthw`, `libc2dcolorconvert` kaynakları yok. Eski Stagefright/OMX uyarlaması gerekiyor. Eski codec listesi portlama amacıyla korunuyor, codec desteği doğrulanmış değil. |
| Eski binary/ABI | `rmt_storage` ve arşivlenen `rild`, ELF `EXEC` (non-PIE). `TARGET_NEEDS_NON_PIE_SUPPORT` bir fork özelliğidir; seçilen linker'da uygulanıp uygulanmadığı kontrol edilmeli. Eski libbinder/libutils/libui/libcamera_client sembolleri için hedef ROM üzerinde sembol analizi ve gerekirse gerçek shim/uyumluluk yamaları gerekli. |
| Wi-Fi/Bluetooth | BCM4329 Wi-Fi firmware'i vendor arşivinde yok; product'taki Broadcom firmware bağımlılığı gerekli. `dhd.ko` kernel ile üretilmeli. Oreo supplicant HIDL, NL80211 ve eski Broadcom sürücü/vendor desteği birlikte doğrulanmalı. Bluetooth için uygun `libbt-vendor` kaynağı gerekli. |
| SELinux | Yeni politika başlangıç düzeyinde. Tam kaynakta derlenmeli; cihazın AVC kayıtlarına göre HAL, sysfs ve remote-storage erişimleri tamamlanmalı. CM12'nin geniş izinleri kopyalanmamalı. |
| Depolama/boot | Mevcut bölüm boyutları değiştirilmedi. 5.5 MiB boot ve 7.5 MiB recovery sınırlarına kernel+ramdisk'in sığdığı kontrol edilmeli. fstab şifreleme/F2FS ve SD kart yolları cihazda denenmeli. |

`legacy-cm12/` altındaki dosyalar yalnızca referans içindir; aktif yapılandırmaya
eklenmez. Eski `extract-files.sh`/`setup-makefiles.sh` kullanılmamalıdır: eski
betikler blob listesi ve çıktı yolları açısından verilen vendor arşiviyle uyuşmuyordu.
Yeni dosya eşlemelerinin kaynağı `tools/vendor-files.json` dosyasıdır.

## Kaynak ağacına yerleştirme

Paketin `device/samsung/ancora` ve `vendor/samsung/ancora` dizinlerini aynı yollarla
tam bir LineageOS **lineage-15.1** kaynak ağacına kopyalayın. `tools/` ve bu belgeyi
de kaynak ağacının kökünde saklayın. Platform kaynaklarını edinmek için başlangıç:

```sh
repo init -u https://github.com/LineageOS/android.git -b lineage-15.1
repo sync
```

Bu komutlar cihaza özel eksik kernel/HAL kaynaklarını sağlamaz. Yukarıdaki bağımlılıkları
uygun ve birbiriyle uyumlu dallardan ayrıca eklemek gerekir. Repo adresi/dalı
verilmediği için rastgele kernel veya hardware deposu ekleyen local manifest üretilmedi.
Eski Android sürümlerine uygun derleme ortamı ve JDK 8 kullanılmalıdır.

Yerel kontroller:

```sh
python3 tools/generate-vendor.py
python3 tools/check-port.py
# Tam Lineage kaynakları eklendikten sonra ek yol kontrolleri:
python3 tools/check-port.py --android-root "$PWD"
```

Eksikler giderildikten sonra derlemeyi başlatmak için:

```sh
source build/envsetup.sh
lunch lineage_ancora-userdebug
mka bacon
```

Bu komutlar burada çalıştırılmış bir derlemenin sonucu değildir.
Statik kontrolün geçmesi kernel'in, C/C++ HAL'lerin, platform sembollerinin veya
SELinux politikasının derlendiği anlamına gelmez.

İlk cihaz denemelerinde sırasıyla boot/adb, ekran/dokunmatik, vold/depolama,
ses, Wi-Fi/Bluetooth, sensörler/GPS, kamera/video ve modem doğrulanmalıdır.
`adb logcat -b all`, `dmesg` ve varsa last_kmsg/pstore kayıtları sonraki uyarlamanın
temel girdileridir. Bu paket için donanım üzerinde doğrulanmış hiçbir özellik yoktur.

## Başvurulan LineageOS 15.1 kaynakları

- https://github.com/LineageOS/android_hardware_interfaces/tree/lineage-15.1
- https://github.com/LineageOS/android_frameworks_base/tree/lineage-15.1
- https://github.com/LineageOS/android_lineage-sdk/tree/lineage-15.1
- https://github.com/LineageOS/android_system_core/tree/lineage-15.1
- https://github.com/LineageOS/android_system_sepolicy/tree/lineage-15.1
- https://github.com/LineageOS/android_build/tree/lineage-15.1

Orijinal lisanslar korunmuştur; kapalı kaynak dosyaları kullanıcı tarafından
sağlanan arşivlerden alınmıştır.
