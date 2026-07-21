# 🪴 SaksıAğ

**Binlerce balkon saksısı, tek bir dayanıklı sinir ağı.**

SaksıAğ; balkonlardaki akıllı saksıları **LoRa mesh** ile birbirine bağlayan açık donanım bir çevre ve afet ağı fikridir. Her saksı hem bir sensör hem de bir yayıcıdır: toprağı ölçer, tozlaşmayı besler, yağmuru tutar — ve baz istasyonları çöktüğünde afet mesajını balkondan balkona taşır. İnternet gerekmez.

> ⚠️ **Durum:** Bu bir **erken konsept taslağıdır.** Teknik değerler (menzil, band, güç bütçesi, maliyet) tahminidir ve bir pilot öncesi doğrulanmalıdır.

🌐 **Canlı konsept sayfası:** <https://ta3hrj.github.io/saksiag/> — ya da depodaki [`index.html`](index.html) dosyasını yerelde açın.

---

## Neden saksı — düz bir kutu değil?

Form bir süs değil, stratejinin kendisidir. Bir sensör ağının tek gerçek zorluğu teknoloji değil, **yeterince çok düğümü doğru yerlere gönüllüce koydurmaktır.**

| Boyut | Düz kutu | Saksı |
|-------|----------|-------|
| Yayılma / yoğunluk | Kimse gönüllü takmaz | İnsanlar zaten alıp balkona koyuyor → yoğunluk bedava |
| Sensör yerleşimi | Toprak/su/bitki yok | Toprak · hazne · bitki · güneşli konum doğal gelir |
| Bakım / canlılık | Unutulur, sessizce ölür | Her gün sulanır → ücretsiz bakım + canlılık kontrolü |
| Toplumsal kabul | Kameramsı, dirençli | Sıcak, kişisel, zararsız |
| Değer / misyon | Salt maliyet | Bitki + toprak + su tasarrufu; misyonu cisimleştirir |

**Asimetri:** Saksının sorunu *çözülebilir bir mühendislik sorunudur* (nem/korozyon → mühürlü bölme, kapasitif sensör, drenaj). Kutunun sorunu ise *çözülemez bir benimseme sorunudur.*

---

## Mimari

Doğru işi doğru radyoya yaptırmak üzerine kurulu:

- **LoRa (ESP32 · SX1262):** Omurga. Düğümden düğüme küçük paketleri kilometrelerce, bina içinden geçirerek taşır. 868 MHz (TR/EU).
- **Bluetooth (BLE):** Yardımcı köprü. Telefon ↔ düğüm (kurulum, eşleme, yerel veri boşaltma).
- **Güneş paneli + LiFePO4:** Bakımsız enerji. Sıcağa/çevrime dayanıklı, yangın riski düşük kimya.

**Üç düğüm tipi (aynı donanım, farklı rol):** yaprak düğüm (algılar/yayınlar) · yönlendirici (röle) · ağ geçidi (internete köprü, mahallede 1–3).

**Olay teyidi:** Tek düğümün yanlış alarmı yayılmaz; olay ancak birkaç komşu aynı anda onaylayınca "doğrulanmış" sayılır. Ucuz sensörlerin güvenilmezliği ağ mimarisiyle telafi edilir.

**Mesaj önceliği (LoRa %1 duty-cycle bütçesi):** `P0` afet alarmı → `P1` insan mesajı → `P2` sensör özeti → `P3` rutin telemetri.

---

## Kapsanan afetler

İlke: **mesh, tehlikenin kendisinden hızlıysa önden uyarır.** Rüzgârla, suyla, ısıyla yayılan afetler dakikalar–saatler ölçeğinde ilerler; radyo atlaması bunları geçer.

| Afet | Mesh önden uyarır mı? | Sensör |
|------|------------------------|--------|
| Sel / taşkın | 🟢 Güçlü | su seviyesi + yağmur + nem |
| Yangın | 🟢 Güçlü | sıcaklık + PM2.5 + CO |
| Zehirli bulut / kimyasal | 🟢 Güçlü | gaz (CO / NO₂ / VOC) |
| Sıcak hava dalgası | 🟢 Fazlasıyla | sıcaklık + nem |
| Aşırı soğuk / don | 🟢 Evet | sıcaklık |
| Heyelan | 🟡 İyi | nem + eğim (ivmeölçer) |
| Fırtına / dolu | 🟡 Birkaç dk | barometrik basınç |
| Doğalgaz kaçağı | 🟡 Yerel güvenlik | metan / LPG |
| Deprem | 🟡 İnternet geçidiyle uzağa | ivmeölçer → ön uyarı (uzak) + afet-sonrası teyit |

**Deprem notu (dürüst konumlandırma):** Yerel LoRa atlaması sismik dalgayı geçemez; merkez üssünde uyarı yoktur. Ama internete bağlı ağ geçitleri devredeyken elektronik sinyal uzak mahallelere dalgadan hızlı ulaşır — uzaklaştıkça artan saniyelerce ön uyarı mümkün. Bunu **ana vaat değil**, ağ yoğunlaştıkça beliren ve ulusal sistemleri (AFAD, telefon-tabanlı uyarı) besleyen bir **arka-plan katkı katmanı** olarak konumlandırıyoruz.

---

## Malzeme listesi (v1)

Meshtastic-uyumlu bir taban üzerine kurulur — mesh protokolü yeniden icat edilmez.

- ESP32 + SX1262 (LoRa omurga)
- Kapasitif toprak nem sensörü
- MEMS ivmeölçer (sarsıntı + heyelan eğimi)
- BME680 (sıcaklık + nem + basınç + VOC)
- PM sensörü / PMS5003 (duman + kirlilik)
- Su seviyesi sensörü (ultrasonik)
- LiFePO4 hücre + şarj kontrol
- Güneş paneli (1–2 W)
- 3D-baskı gövde
- *v1.1:* gaz sensörü (MQ/MiCS), NFC/QR bilgi etiketi

**Kaba güç & maliyet:** ort. ~8–12 mA · panel 1–2 W · özerklik 3–5 gün · birim ~30–90 USD *(pilot öncesi doğrulanacak)*.

---

## Yol haritası

- **Faz 0 — Yaşayan düğüm:** Tek saksı; bir bitki bakımı altında aylarca bakımsız yaşıyor mu?
- **Faz 1 — Sokak mesh'i:** 10–20 balkon; çok-atlamalı yayılım + komşu konsensüsü yangın/sel'de çalışıyor mu?
- **Faz 2 — Çok-tehlikeli mahalle:** İnternet geçidi, harita, telefon uygulaması.
- **Faz 3 — Kritik kütle:** Açık donanım dosyaları, topluluk; yoğunlukla deprem katkısı belirir.

---

## Katkı

Bu depo erken bir konsept aşamasındadır; fikir, geri bildirim ve teknik itirazlar memnuniyetle karşılanır — bir Issue ya da Pull Request açın. Katkı kuralları ve DCO imzası için [`CONTRIBUTING.md`](CONTRIBUTING.md).

## Lisans

Çok-lisanslı bir projedir — ayrıntılar için [`LICENSING.md`](LICENSING.md):

- **Yazılım / firmware:** [AGPL-3.0-or-later](LICENSE)
- **Donanım tasarımları:** [CERN-OHL-S-2.0](LICENSES/CERN-OHL-S-2.0.txt)
- **Doküman & içerik:** [CC-BY-SA-4.0](LICENSES/CC-BY-SA-4.0.txt)

"SaksıAğ" adı ve logosu bu lisansların dışında, proje sahibince saklı tutulur.

---

*İnsanlık, doğa ve tüm yaşam formları için.*
