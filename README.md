# CAN Bus Haberleşmeli Kapalı Çevrim Step Motor Sürücü Kartı (STM32F303)

CAN bus haberleşmeli, papatya zincirleme (daisy chain) bağlanabilen, **kapalı çevrim step motor sürücü kartı**. NEMA 17 motorun arkasına monte edilir; her motor kendi işlemcisi, sürücüsü ve enkoderi olan akıllı bir düğüm (node) olur. Güç ve CAN hattı tek bir kablo ile  bir sonraki karta aktarılır.



---
## İçindekiler

- [Özellikler](#özellikler)
- [Donanım Özeti](#donanım-özeti)
- [PCB Görselleri](#pcb-görselleri)
- [Pin Atamaları](#pin-atamaları)
- [CAN Protokolü](#can-protokolü)
- [Papatya Zinciri Bağlantısı](#papatya-zinciri-bağlantısı)
- [Depo Yapısı](#depo-yapısı)
- [Derleme ve Yükleme](#derleme-ve-yükleme)
- [İlk Çalıştırma (Bring-up)](#ilk-çalıştırma-bring-up)
- [Test Planı](#test-planı)
- [Güvenlik Uyarıları](#güvenlik-uyarıları)
- [Bilinen Sorunlar ve Yapılacaklar](#bilinen-sorunlar-ve-yapılacaklar)
- [Lisans](#lisans)

---

## Özellikler

| Özellik | Değer |
|---|---|
| Mikrodenetleyici | **STM32F303** (ARM Cortex-M4F, dahili bxCAN, USB FS) — TODO: tam model, örn. `STM32F303CBT6` |
| Step motor sürücüsü | TMC2209 (StealthChop2, SpreadCycle, StallGuard4, UART ile yapılandırma) |
| Motor akımı | ≤ 1,92 A rms (Rsense = 100 mΩ, yalnızca yazılımla ayar) |
| Enkoder | MT6701, 14 bit (16 384 sayım/tur ≈ 0,022°), I²C |
| CAN alıcı-verici | TCAN3414, 1 Mbit/s, DIP anahtarla seçilebilir 120 Ω sonlandırma |
| Besleme | 5–29 V DC (XT30), önerilen kullanım 24 V'a kadar |
| Güç geçişi | IN → OUT arası 12 A, giriş sigortası + TVS |
| Düğüm sayısı | En fazla 31 (5 bit düğüm kimliği, 0 = yayın) |
| USB | USB-C: programlama, seri konsol, USB ↔ CAN köprüsü |
| Genişleme | Qwiic (I²C) ve AUX (2 G/Ç) konnektörleri |
| İzleme | VIN ölçümü, NTC ile kart sıcaklığı, MCU sıcaklığı |
| Mekanik | ≈ 42 × 42 mm, 31 × 31 mm montaj delikleri, Ø3,3 mm |

## Donanım Özeti

```
DC kaynak ──► XT30 IN ──► Sigorta (12 A) + TVS ──► VIN_BUS ──► XT30 OUT (sonraki düğüm)
                                                     │
                                                     ├──► TMC2209 (VS)
                                                     └──► Diyot-VEYA ──► 3,3 V buck ──► STM32F303, TCAN3414, MT6701
USB-C 5 V ───────────────────────────────────────────┘

CAN_H / CAN_L : XT30 IN ◄──► XT30 OUT (doğrudan), TCAN3414 hatta "dal" olarak bağlı
STM32F303 ◄─ CAN TX/RX ─► TCAN3414
STM32F303 ── STEP/DIR/EN ──► TMC2209 ◄── UART ──► STM32F303
STM32F303 ◄── I²C ──► MT6701 (kartın alt yüzünde, motor mili ekseninde)
```

- **Yalnızca USB ile besleme:** İşlemci çalışır, motor sürücüsü enerjisiz kalır (yapılandırma ve test için).
- **Diyot-VEYA:** VIN_BUS ve USB 5 V, Schottky diyotlarla birleştirilir; VIN takılıp çıkarılırken MCU sıfırlanmaz.
- **Kapalı çevrim:** Enkoder sayımı mikroadıma çevrilir, iç konum sayacı ile fark eşiği aşarsa sayaç enkoder değerine eşitlenir; kaçırılan adımlar otomatik telafi edilir.
- **Adım üretimi:** STM32 donanım zamanlayıcıları ile yapılır. TODO: kullanılan TIM ve kanal bilgisi.
- **ADC:** VIN bölücüsü ×11 (29 V → 2,64 V), 12 bit ADC, 3,3 V referans.

### Ana Bileşenler

TODO: Kendi BOM'unuza göre doğrulayın / güncelleyin.

| Ref. | Parça | Not |
|---|---|---|
| U? | STM32F303xx | TODO: tam parça numarası ve kılıf |
| U? | TMC2209-LA | Alt ped ısıl via |
| U? | TCAN3414DRBR | CAN alıcı-verici |
| IC? | MT6701 (I²C varyantı) | Alt yüz, mil ekseni |
| U? | 3,3 V buck (≥ 40 V giriş) | TODO: kullandığınız entegre |
| J? | XT30PW(2+2)-M ×2 | IN / OUT |
| J? | USB-C | Programlama / CAN köprüsü |
| J? | Motor konnektörü (Milli-Grid 4 pin) | A1, B1, A2, B2 |
| J? | JST-SH 4 pin ×2 | Qwiic ve AUX |
| SW? | 1 konumlu DIP anahtar | CAN TERM (120 Ω) |

Tam liste için: [`hardware/bom/`](hardware/bom/)

## PCB Görselleri

| 3B görünüm (üst) | Üst katman (kırmızı) | Alt katman (mavi) |
|---|---|---|
| <img src="docs/images/pcb_3d_top.png" width="250"> | <img src="docs/images/pcb_top_layer.png" width="250"> | <img src="docs/images/pcb_bottom_layer.png" width="250"> |

- Katman sayısı: TODO (2 / 4 katman)
- Bakır ağırlığı: TODO
- Kart kalınlığı / yüzey kaplama: TODO
- Tasarım aracı: TODO (KiCad / Altium / EasyEDA)

Tasarım notları:

- IN → OUT arasındaki 12 A yolu, birden çok katmanda paralel bakır dolgu ve çok sayıda via ile sağlanmıştır.
- GND dönüş yolu aynı genişlikte tutulmuştur; montaj delikleri GND'ye bağlıdır.
- CAN_H / CAN_L çifti birlikte ve kısa yönlendirilmiştir; ESD diyodu hatta yakındır.
- MT6701'in altından ve yakınından yüksek akımlı iz geçmemelidir (manyetik ölçümü bozar).
- XT30 etiketleri: `XT30_IN` (sol), `XT30_OUT` (sağ).

## Pin Atamaları

TODO: STM32F303 için kendi pin atamanızı doldurun.

| Sinyal | STM32 pini | Yön | Açıklama |
|---|---|---|---|
| CAN_TX | TODO (örn. PB9) | Ç | TCAN3414 TXD (bxCAN) |
| CAN_RX | TODO (örn. PB8) | G | TCAN3414 RXD (bxCAN) |
| CAN_STB | TODO | Ç | TCAN3414 STB (0 = normal) |
| STEP | TODO | Ç | TMC2209 STEP (zamanlayıcı çıkışı) |
| DIR | TODO | Ç | TMC2209 DIR |
| TMC_EN | TODO | Ç | TMC2209 EN (aktif-düşük) |
| TMC_UART TX/RX | TODO | G/Ç | PDN_UART (TX 1 kΩ üzerinden) |
| DIAG | TODO | G | StallGuard kesmesi |
| INDEX | TODO | G | TMC2209 INDEX |
| MS1 / MS2 | TODO | Ç | UART adresi |
| SDA / SCL | TODO | G/Ç | MT6701 + Qwiic, 4,7 kΩ pull-up |
| VIN_SENSE | TODO (ADC) | G | 47 kΩ / 4,7 kΩ bölücü (×11) |
| NTC | TODO (ADC) | G | 10 kΩ NTC, B = 3380 |
| SW1 / SW2 | TODO | G | Butonlar |
| LED1 / LED2 | TODO | Ç | Programlanabilir LED'ler |
| AUX1 / AUX2 | TODO | G/Ç | AUX konnektörü (alt yüz) |
| USB D− / D+ | PA11 / PA12 | USB | USB-C |
| BOOT0 | TODO | G | DFU / bootloader girişi |
| SWDIO / SWCLK | PA13 / PA14 | — | Programlama |

> **Not:** STM32F303'te CAN ve USB varsayılan olarak PA11/PA12 pinlerini paylaşır. Her ikisini birden kullandığınız için CAN'ın alternatif pinlerde (örn. PB8/PB9) olması gerekir. Kendi seçiminizi yukarıya yazın. TODO

> **Not:** AUX1/AUX2 sırasını kartınızdaki serigrafiye göre burada açıkça belirtin. TODO

## CAN Protokolü

11 bitlik standart CAN kimliği:

```
CAN_ID = (NodeID << 6) | MsgType
          5 bit          6 bit
```

- `NodeID`: 1–31, `0` = yayın (tüm düğümler)
- `MsgType`: 0–31 komut, 32–63 telemetri
- Çok baytlı değerler **little-endian**
- RTR bitli çerçeve, ilgili değerin geri gönderilmesini ister
- Hız: **1 Mbit/s**

Seçilmiş mesajlar:

| Tip | Anlam | Veri |
|---|---|---|
| 1 | Konum (derece) | double |
| 2 | Konum (adım) | int64 |
| 3 | Hız (°/s) | float |
| 4 / 27 | Çalışma / tutma akımı (%) | uint16 |
| 5 | Etkinleştir | 0 / 1 |
| 6 | Acil durdurma | herhangi |
| 11 | Mikroadım | uint16 |
| 13 | Kapalı çevrim tipi (0 = açık, 1 = kapalı) | int16 |
| 33 | Enkoder açısı | double |
| 34 | Giriş gerilimi | float |
| 38 | Kart (NTC) sıcaklığı | float |
| 39 | Hata kodu | uint16 |

Örnek: Düğüm 3'ü 90° konumuna götür → `ID = (3 << 6) | 1 = 0xC1`, veri = `00 00 00 00 00 80 56 40` (double 90,0).

TODO: Firmware'inizde uyguladığınız tüm komut ve telemetri mesajlarını `docs/` altında ayrı bir dosyada listeleyin.

### USB ↔ CAN seri köprü biçimi

USB seri port (115200 baud) üzerinden satır tabanlı metin:

```
<CAN_ID onaltılık> <RTR> <16 onaltılık karakter veri>
```

Örnekler:

```
05 0 0100000000000000      # tüm düğümleri etkinleştir
43 0 0000C84200000000      # düğüm 1'i 100 °/s ile döndür
```

TODO: Köprü özelliği firmware'inizde uygulandıysa belirtin.

## Papatya Zinciri Bağlantısı

```
DC kaynak ─► [Düğüm 1] ─► [Düğüm 2] ─► [Düğüm 3] ─► ... ─► [Düğüm N]
              TERM: AÇIK   TERM: KAPALI  TERM: KAPALI        TERM: AÇIK
                 ▲
               USB ── Bilgisayar (herhangi bir düğüm USB↔CAN köprüsü olur)
```

- Her düğüm, tek bir XT30 (2+2) kablosuyla (+, −, CAN_H, CAN_L) zincirlenir.
- **120 Ω sonlandırma yalnızca hattın iki ucundaki düğümlerde açık olmalıdır** (DIP anahtar). Enerjisiz hatta CAN_H–CAN_L arası ≈ 60 Ω ölçülmelidir.
- Her düğüme benzersiz bir kimlik (1–31) atanmalıdır.
- 12 A geçiş akımı ilk kartın IN–OUT yolundan geçer; kablo kesiti en az 18 AWG olmalıdır.

## Depo Yapısı

TODO: Kendi klasör yapınıza göre düzenleyin.

```
.
├── hardware/
│   ├── schematic/        # Şematik dosyaları ve PDF
│   ├── pcb/              # PCB dosyaları
│   ├── gerber/           # Gerber ve delik dosyaları
│   └── bom/              # BOM ve CPL
├── firmware/
│   └── stm32f303/        # Düğüm yazılımı
├── software/
│   └── python/           # Python çok eksenli kontrol örnekleri
├── docs/
│   ├── images/           # README görselleri
│   └── Proje_Raporu.pdf  # Proje raporu
├── LICENSE
└── README.md
```

## Derleme ve Yükleme

TODO: Kullandığınız ortama göre doldurun.

### Gereksinimler

- TODO: STM32CubeIDE / PlatformIO / Arduino_Core_STM32 sürümü
- TODO: ST-Link (SWD) veya USB DFU aracı

### Adımlar

```bash
# Örnek (PlatformIO)
git clone https://github.com/<kullanici>/<depo>.git
cd <depo>/firmware/stm32f303
pio run -t upload
```

## İlk Çalıştırma (Bring-up)

İlk enerjilendirme **akım sınırlı laboratuvar kaynağı** ile yapılmalıdır. Motor, 8. adıma kadar bağlanmamalıdır.

| # | Adım | Beklenen |
|---|---|---|
| 1 | Görsel kontrol (köprü, yön, eksik eleman) | Hata yok |
| 2 | Enerjisiz kısa devre kontrolü (VIN–GND, 3V3–GND, CAN_H–CAN_L) | VIN, 3V3 > 1 kΩ; CAN_H–L ≈ 120 Ω (TERM açık) |
| 3 | Yalnızca USB ile enerji | 3V3 = 3,30 V ± %3; 3V3 LED yanar |
| 4 | Yazılım yükle (LED yakıp söndür + seri çıktı) | COM portu görünür, LED yanıp söner |
| 5 | I²C ve UART kontrolü | MT6701 bulunur; TMC2209 bu adımda yanıt vermez (VIN yok) |
| 6 | VIN uygula (12 V, akım sınırı 100 mA) | Boşta akım düşük; VIN ölçümü ≈ 12 V |
| 7 | Sürücü sürüm kontrolü | TMC2209 sürümü `0x21`, hata bayrağı yok |
| 8 | Motoru tak (enerji kesikken), %30 akım, düşük hız | Pürüzsüz dönüş |
| 9 | Enkoder (mıknatıs takılı, mili elle çevir) | Açı 0–360° sürekli, sıçrama yok |
| 10 | CAN (iki düğüm) | Komutlar iki düğümde çalışır |

> Motor elle veya yük tarafından hızlı döndürülürse jeneratör gibi davranıp VIN_BUS'a gerilim basar. Laboratuvar kaynakları genellikle akım yutamaz.

## Test Planı

| Kod | Test | Başarı ölçütü |
|---|---|---|
| T1 | Giriş gerilimi ve VIN ölçümü (5–29 V) | Hata ≤ ±%3; VIN < 6 V'ta sürücü kapalı |
| T2 | USB/VIN geçişleri | VIN takılıp çıkarılırken MCU sıfırlanmaz |
| T3 | 3,3 V kalitesi, boşta tüketim | 3,3 V ± %3; dalgalanma < 50 mVpp |
| T4 | Haberleşme (UART, I²C, CAN) | İki düğüm görünür |
| T5 | Enkoder çözünürlüğü ve gürültüsü | σ ≤ 2 sayım |
| T6 | Mikroadım–enkoder doğrusallığı | 1 turda hata ≤ 0,5° |
| T7 | Hareket profili (ivme/hız) | Ayarın ± %10'u |
| T8 | Azami hız | Yazılım/motor sınırı açıklanır |
| T9 | Adım kaçırma: açık / kapalı çevrim | Kapalı çevrimde son hata ≤ 0,2° |
| T10 | StallGuard ile sensörsüz referanslama | 10 tekrarda saçılım ≤ ±1° |
| T11 | Isıl test ve hata kodları | NTC < 100 °C |
| T12 | CAN fiziksel katman ve sonlandırma | 60 Ω; diferansiyel ≈ 2 V |
| T13 | Çok düğüm, gecikme, eşzamanlılık | Yayın kayması < 1 ms |
| T14 | Güç geçişi ve gerilim düşümü | 10 A'de IN–OUT düşümü < 0,15 V |
| T15 | Robotik gösterim (3 eksen) | 5 poz × 10 tekrar, hata ≤ 0,5° |

### Test Sonuçları

TODO: Yaptığınız testlerin sonuçlarını, grafiklerini ve notlarını buraya ekleyin.

| Test | Durum | Sonuç / Not |
|---|---|---|
| T1 | ⬜ | |
| T2 | ⬜ | |
| T3 | ⬜ | |
| ... | ⬜ | |

## Güvenlik Uyarıları

- Tüm testlere **akım sınırlı laboratuvar kaynağı** ile başlayın; akü (LiPo vb.) yalnızca kart doğrulandıktan sonra ve sigortalı kabloyla kullanın.
- Motor kablosunu ve XT30 kablolarını **enerji varken takıp çıkarmayın.**
- Hızlı döndürülen motor VIN_BUS'a gerilim basabilir; ağır yüklerin hızlı yavaşlatılmasında dikkatli olun.
- 24 V / 1,5 A civarında TMC2209 ve soğutucu 70 °C'yi aşabilir; çalışırken dokunmayın.
- Robot kolu testlerinde hareket alanında el bulundurmayın; acil durdurma (yayın, tip 6) her zaman hazır olsun.
- Neodim mıknatısları yutulma riskine karşı saklayın.

## Bilinen Sorunlar ve Yapılacaklar

- [ ] TODO: Pin atama tablosunu tamamla
- [ ] TODO: Firmware'i yayınla ve derleme talimatlarını yaz
- [ ] TODO: Python denetleyici örneklerini ekle
- [ ] TODO: Test sonuçlarını ve grafikleri ekle
- [ ] TODO: Kart revizyon notlarını (hatalar, düzeltmeler) ekle
- [ ] Bölünmüş (split) sonlandırma
- [ ] Ters kinematik ile kartezyen komut
- [ ] ROS 2 köprüsü

## Lisans

TODO: Lisansınızı seçin ve `LICENSE` dosyasını ekleyin (örn. GPL-3.0). Donanım dosyaları için ayrı bir lisans kullanacaksanız (örn. CERN-OHL-S) burada belirtin.

### Kullanılan Veri Sayfaları

- Analog Devices / Trinamic — TMC2209 Datasheet
- Texas Instruments — TCAN3413 / TCAN3414 Datasheet
- MagnTek — MT6701 Datasheet
- STMicroelectronics — STM32F303 Datasheet ve Reference Manual (RM0316)
- ISO 11898-1/-2 — Controller Area Network (CAN)
- IPC-2221 — Generic Standard on Printed Board Design

---

**Geliştirici(ler):** TODO
**Ders / Dönem:** Robotik Uygulamalar — 2026–2027 Güz
