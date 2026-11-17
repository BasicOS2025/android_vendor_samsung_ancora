# Samsung Galaxy W (ancora) vendor — LineageOS 15.1

Android 8.1 için başlangıç vendor uyarlaması. **Derlenmiş veya cihazda doğrulanmış değildir.**

Tam LineageOS `lineage-15.1` kaynak ağacında bu depoyu `vendor/samsung/ancora` yoluna yerleştirin. Ayrı bir Oreo uyumlu device ağacı, kernel ve msm7x30 HAL kaynakları gereklidir. Bu depo yalnızca vendor dosyalarını içerir.

Kullanıcının sağladığı CM12 arşivindeki 55 proprietary dosya değiştirilmeden korunmuştur. 43 prebuilt modül ve 10 dosya kopyalama girdisi tanımlanmıştır. Eski non-PIE `rild` ve `libril.so` saklanır, ancak kurulum listesine eklenmez; Oreo radio HIDL uyarlaması gereklidir.

Makefile'ları bu depo kökünde yeniden üretmek için:

```sh
python3 tools/generate-vendor.py
```

`tools/vendor-files.json` dosya eşlemelerini, `tools/blob-sha256.json` orijinal dosyaların SHA-256 değerlerini içerir. `ELF-DEPENDENCIES.txt` bağımlılık dökümüdür; ABI uyumluluğu testi değildir.

Ayrıntılı bağımlılıklar ve engeller: [PORTING-8.1.md](PORTING-8.1.md). Bu belge birleşik device+vendor paketini de anlatır; `tools/check-port.py` gibi birleşik pakete ait yollar bu vendor deposunda bulunmaz. Bu aşamada derleme veya boot başarısı iddia edilmez.
