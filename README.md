# CAN Bus Haberleşmeli Kapalı Çevrim Step Motor Sürücü Kartı (STM32F303)

CAN bus haberleşmeli, papatya zincirleme (daisy chain) bağlanabilen, **kapalı çevrim step motor sürücü kartı**. NEMA 17 motorun arkasına monte edilir; her motor kendi işlemcisi, sürücüsü ve enkoderi olan akıllı bir düğüm (node) olur. Güç ve CAN hattı tek bir kablo ile  bir sonraki karta aktarılır.



---


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


## PCB Görselleri







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

