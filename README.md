# Otonom Tarım Robotu Kontrol Kartı (AgriRobot Stepper & CAN Bus Controller)

Bu depo, otonom domates toplama robotunun mekanik kol eklemlerini ve motor hareketlerini yüksek hassasiyetle yönetmek için tasarlanmış 4 katmanlı endüstriyel kontrol kartının donanım (PCB) dosyalarını içermektedir. Kart, NVIDIA Jetson Nano ile CAN Bus üzerinden haberleşerek otonom sistemin alt seviye motor kontrol (stepper) görevlerini üstlenir.

## 🚀 Öne Çıkan Özellikler

* **Ana Denetleyici (MCU):** Sistemin merkezinde yer alan STM32 serisi mikrodenetleyici[cite: 12].
* **Güç Altyapısı:** Yüksek akım taşıyabilen `XT30_IN` ve `XT30_OUT` konnektör bağlantıları[cite: 13, 14] ile sağlanan 24V DC güç yönetimi.
* **Haberleşme Arayüzleri:** Dış sistemlerle iletişim için modern USB Type-C portu[cite: 12] ve endüstriyel CAN Bus diferansiyel hatları.
* **Fiziksel ve Elektriksel Yapı:** 
  * Siyah lehim maskesi (solder mask) üzerine dizilmiş yüksek yoğunluklu SMD bileşenler (kapasitörler, dirençler, butonlar)[cite: 12].
  * Parazitleri önlemek ve ısıyı dağıtmak amacıyla tasarlanmış 4 katmanlı yapı.
  * Kartın dört köşesine yerleştirilmiş, elektriksel olarak topraklanmış (`Hole GND`) sağlam montaj delikleri[cite: 13, 14].
* **Güvenlik ve Üretim:** Yüksek akım hatlarında (24V/XT30) atlamaları ve lehim kısa devrelerini önlemek için 0.2mm - 0.3mm özel izolasyon (Clearance) kuralları uygulanmıştır.

## 📸 Tasarım Görselleri

Aşağıda Altium Designer üzerinden alınmış tasarım katmanları ve 3D model görselleri bulunmaktadır:

### 3D Kart Görünümü
*(Kompakt tasarım, USB-C arayüzü, XT30 güç girişleri ve STM32 yerleşimi)*
![3D Görünüm](docs/image_604a75.png) 

### Katman İncelemesi (Top & Bottom Layer)
*(Sol: Kırmızı üst katmandaki (Top Layer) yoğun sinyal yolları. Sağ: Mavi alt katmandaki (Bottom Layer) güç ve poligon dökümleri)*

| Üst Katman (Top Layer) | Alt Katman (Bottom Layer) |
| :---: | :---: |
| ![Top Layer](docs/image_604a5a.png) | ![Bottom Layer](docs/image_604a5d.png) |

## 📁 Depo İçeriği

* `CanBUS_stepper_STM32.PrjPcb`: Ana Altium proje dosyası
* `CanBUS_stepper_STM32.SchDoc`: Şematik devre tasarımı
* `CanBUS_stepper_STM32.PcbDoc`: 4 katmanlı PCB yönlendirme dosyası
* `CanBUS_stepper_STM32.RUL`: Tasarıma özel dışa aktarılmış üretim kuralları (DRC - Design Rule Check)
* `CAN_Bus_stepper.PcbDoc.pcbstackup`: Kartın fiziksel katman (Layer Stack) konfigürasyonu

## 🛠️ Üretim Notları (Fabrication)

Bu tasarım JLCPCB, PCBWay gibi standart dizgi ve üretim fabrikalarına tam uyumludur. Tasarımda minimum yol kalınlığı, matkap çapı (0.2mm minimum via deliği) ve lehim maskesi payları modern üretim standartlarına göre (Rule-Driven) optimize edilmiştir. Üretime göndermek için gereken Gerber ve NC Drill dosyalarını projeden doğrudan türetebilirsiniz.
