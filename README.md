# AgriRobot Stepper & CAN Bus Controller

Bu depo,  4 katmanlı endüstriyel kontrol kartının donanım (PCB) dosyalarını içermektedir. Kart, NVIDIA Jetson Nano ile CAN Bus üzerinden haberleşerek otonom sistemin alt seviye motor kontrol (stepper) görevlerini üstlenir.

## 🚀 Öne Çıkan Özellikler

* **Ana Denetleyici (MCU):** Sistemin merkezinde yer alan STM32 serisi mikrodenetleyici[cite: 12].
* **Güç Altyapısı:** Yüksek akım taşıyabilen giriş ve çıkış  konnektör bağlantıları ile sağlanan 24V DC güç yönetimi.
* **Haberleşme Arayüzleri:** Dış sistemlerle iletişim için modern USB Type-C portu ve endüstriyel CAN Bus diferansiyel hatları.
* **Fiziksel ve Elektriksel Yapı:** 
  * Siyah lehim maskesi (solder mask) üzerine dizilmiş yüksek yoğunluklu SMD bileşenler (kapasitörler, dirençler, butonlar)[cite: 12].
  * Parazitleri önlemek ve ısıyı dağıtmak amacıyla tasarlanmış 4 katmanlı yapı.
  * Kartın dört köşesine yerleştirilmiş, elektriksel olarak topraklanmış (`Hole GND`) sağlam montaj delikleri[cite: 13, 14].
* **Güvenlik ve Üretim:** Yüksek akım hatlarında (24V) atlamaları ve lehim kısa devrelerini önlemek için 0.2mm - 0.3mm özel izolasyon (Clearance) kuralları uygulanmıştır.

## 📸 Tasarım Görselleri

Aşağıda Altium Designer üzerinden alınmış tasarım katmanları ve 3D model görselleri bulunmaktadır:

### 3D Kart Görünümü
*(Kompakt tasarım, USB-C arayüzü, XT30 güç girişleri ve STM32 yerleşimi)*
<img width="777" height="781" alt="image" src="https://github.com/user-attachments/assets/caf8cd30-6403-45bb-b8f6-ff11acbb20eb" />


### Katman İncelemesi (Top & Bottom Layer)
*(Sol: Kırmızı üst katmandaki (Top Layer) yoğun sinyal yolları. Sağ: Mavi alt katmandaki (Bottom Layer) güç ve poligon dökümleri)*

| Üst Katman (Top Layer) | Alt Katman (Bottom Layer) |
---
 <img width="833" height="775" alt="image" src="https://github.com/user-attachments/assets/a428175d-5b4a-45dc-b919-a9afb29c8bd1" />
  ---
  <img width="787" height="807" alt="image" src="https://github.com/user-attachments/assets/8e99f5b5-e2a6-4992-bcf6-03d49de00c46" />
 

## 📁 Depo İçeriği

* `CanBUS_stepper_STM32.PrjPcb`: Ana Altium proje dosyası
* `CanBUS_stepper_STM32.SchDoc`: Şematik devre tasarımı
* `CanBUS_stepper_STM32.PcbDoc`: 4 katmanlı PCB yönlendirme dosyası
* `CanBUS_stepper_STM32.RUL`: Tasarıma özel dışa aktarılmış üretim kuralları (DRC - Design Rule Check)
* `CAN_Bus_stepper.PcbDoc.pcbstackup`: Kartın fiziksel katman (Layer Stack) konfigürasyonu

## 🛠️ Üretim Notları (Fabrication)

Bu tasarım JLCPCB, PCBWay gibi standart dizgi ve üretim fabrikalarına tam uyumludur. Tasarımda minimum yol kalınlığı 0.127, matkap çapı (0.2mm minimum via deliği) ve lehim maskesi payları modern üretim standartlarına göre (Rule-Driven) optimize edilmiştir. Üretime göndermek için gereken Gerber ve NC Drill dosyalarını projeden doğrudan türetebilirsiniz.
