# SaksıAğ — Proje Özeti & Devir Notu

> Bu dosya, projeyi yeni bir sohbette kaldığı yerden sürdürmek için hazırlanmış bir **bağlam devri**dir. Kararların *gerekçeleri* dahil edildi ki yeniden tartışılmasın.

**Son güncelleme:** 2026-07-21

---

## 1. Proje nedir?

**SaksıAğ** — balkonlardaki akıllı saksıları **LoRa mesh** ile birbirine bağlayan açık donanım bir **çok-afetli çevre ve afet ağı** fikri. Her saksı hem sensör hem yayıcı: toprağı ölçer, tozlaşmayı besler, yağmuru tutar ve altyapı çökünce afet mesajını balkondan balkona internetsiz taşır.

Tek cümlelik v1 tanımı: *"Yangın, sel ve sıcak dalgayı önden haber veren, afet sonrası internetsiz konuşabilen açık balkon ağı."*

Durum: **erken konsept.** Kod/donanım henüz yok; teknik değerler tahmini, pilot öncesi doğrulanacak.

Köken: Kullanıcının "insanlık, doğa ve tüm yaşam için önemli program" fikri taramasından çıktı; özgün fikirler 1 (toprak sağlığı), 4 (süngerkent), 6 (tozlaşma), 10 (çevre sensörü) + erken uyarı afet birleştirilerek doğdu.

---

## 2. Mimari (kararlaştırıldı)

- **LoRa (ESP32 · SX1262):** Omurga. Küçük paketleri km menzil, bina içinden. 868 MHz (TR/EU), ~%1 duty-cycle.
- **Bluetooth (BLE):** *Yardımcı* köprü — telefon ↔ düğüm (kurulum, eşleme, yerel veri boşaltma). Telefonlarda LoRa yok.
- **Güneş paneli + LiFePO4:** Bakımsız enerji; sıcağa/çevrime dayanıklı, yangın riski düşük.
- **Taban:** Meshtastic-uyumlu (mesh protokolü yeniden icat edilmez).

**Üç düğüm tipi (aynı donanım, rol firmware'de):** yaprak (algıla/yayınla) · yönlendirici (röle) · ağ geçidi (internete köprü, mahallede 1–3).

**Olay teyidi (kritik):** Tek düğümün yanlış alarmı yayılmaz; olay ancak N komşu onaylayınca "doğrulanmış" sayılır. Ucuz sensörün güvenilmezliği ağ mimarisiyle telafi edilir. *(Bu konsensüs mantığının detayı — kaç düğüm/süre/eşik — henüz açılmadı; olası bir sonraki adım.)*

**Mesaj önceliği:** P0 afet alarmı → P1 insan mesajı ("iyiyim/yardım") → P2 sensör özeti → P3 rutin telemetri.

**Kaba güç & maliyet (doğrulanacak):** ort. ~8–12 mA · panel 1–2 W · özerklik 3–5 gün · birim ~30–90 USD.

---

## 3. Kapsanan afetler (kalibrasyon ilkesi)

İlke: **mesh, tehlikeden hızlıysa önden uyarır.** Rüzgâr/su/ısıyla yayılan afetler dk–saat ölçeğinde ilerler; radyo bunları geçer.

- 🟢 Güçlü: sel, yangın, zehirli bulut, sıcak dalga, don
- 🟡 Kısmi: heyelan (nem+eğim), fırtına (basınç), doğalgaz kaçağı (yerel)
- **Deprem:** Yerel LoRa dalgayı geçemez (merkez üssünde uyarı yok). AMA internete bağlı ağ geçitleri devredeyken elektronik sinyal uzak mahallelere dalgadan hızlı ulaşır → uzaklaştıkça saniyelerce ön uyarı mümkün (Google'ın telefon-tabanlı sistemi gibi). **Ana vaat değil; ağ yoğunlaştıkça beliren, AFAD vb.'yi besleyen arka-plan katkı katmanı olarak konumlandırıldı.** (Kullanıcının açık talebi: öne çıkarma.)

**Paylaşımlı sensörler (zarafet):** BME680 (sıcak dalga+don+fırtına+hava), PM/PMS5003 (yangın+kirlilik), gaz (yangın+kaçak+kimyasal), su seviyesi (sel). İvmeölçer zaten var → deprem teyidi + heyelan eğimi çift görev.

---

## 4. "Neden saksı, düz kutu değil?" (temel gerekçe)

Form süs değil, stratejinin kendisi. Ağın tek gerçek sorunu: *yeterince çok düğümü doğru yerlere gönüllüce koydurmak.*

- **Truva atı:** İnsanlar bitki istiyor → sensör yoğunluğu bedava. Kutuyu kimse takmaz ("gözetleme").
- Sensörlerin çoğu zaten toprağı/hazneyi/bitkiyi/güneşi gerektirir; kutu hiçbirini doğal sağlamaz.
- **Bakım insan alışkanlığına biner:** sulama = ücretsiz canlılık kontrolü. (Sensör ağlarının 1 no'lu ölümü ihmaldir.)
- **Asimetri argümanı:** Saksının sorunu *çözülebilir mühendislik* (nem/korozyon → mühürlü bölme, kapasitif sensör, drenaj); kutunun sorunu *çözülemez benimseme.*

---

## 5. Öncelik (Etki × Yapılabilirlik) — v1 kapsamı

- 🟢 **v1 hemen:** yangın, sel, sıcak dalga algılama + çevre/hava.
- 🔵 **v1 killer / yatırım:** afet-mesh iletişimi; süngerkent/su tutma; (gaz v1.1; topluluk/zihinsel uzun vade).
- 🟡 **Hızlı dolgu:** toprak sağlığı, onarım hakkı (açık donanım), dil/bilgi (NFC).

**Yol haritası:** Faz 0 Yaşayan düğüm → Faz 1 Sokak mesh'i → Faz 2 Çok-tehlikeli mahalle → Faz 3 Kritik kütle (yoğunlukla deprem katkısı belirir).

---

## 6. Depo & altyapı durumu

- **GitHub:** https://github.com/TA3HRJ/saksiag (public) — hesap `TA3HRJ`, `gh` CLI ile bağlı.
- **Canlı sayfa (GitHub Pages):** https://ta3hrj.github.io/saksiag/ (main/kök, HTTPS zorunlu, push'ta otoyayın).
- **Yerel yol:** `C:\HAM\SaksiAg` — dal `main`, `origin/main` ile senkron.
- **Claude Artifact (kullanıcıya özel, private):** https://claude.ai/code/artifact/fa8de2e0-5641-461c-a012-a1a7f13eb802 — içerik `index.html` ile aynı.

**Depo içeriği:**
- `index.html` — tek sayfalık görsel konsept (canvas mesh animasyonu dahil, tema-duyarlı)
- `README.md` — proje özeti
- `LICENSING.md`, `CONTRIBUTING.md`
- `LICENSE` (AGPL-3.0), `LICENSES/CERN-OHL-S-2.0.txt`, `LICENSES/CC-BY-SA-4.0.txt`
- `.gitignore`, bu `OZET.md`

---

## 7. Lisanslama (kararlaştırıldı ve uygulandı)

Çok-lisanslı: **yazılım AGPL-3.0 · donanım CERN-OHL-S-2.0 · doküman CC-BY-SA-4.0.** Metinler resmi kaynaklardan birebir. GitHub `AGPL-3.0` olarak tanıyor.

Gerekçe (kullanıcının "ticariye dönerse?" sorusuna): Açık kaynak ≠ ticaret yasağı. Copyleft + telif sende + **marka ("SaksıAğ") saklı** → ticari kapı açık (çift lisans opsiyonu), rakip kapatıp koparamaz. **Önemli uyarı:** depo public olduğu için kamuya açıklama gerçekleşti → EU/TR'de patent yolu pratikte kapandı (ama bu "savunma amaçlı yayın"dır; başkası da patentleyemez). Katkı: şimdilik DCO; çift lisans ciddileşirse CLA gerekir. **Ben avukat değilim — ciddi ticari adımda IP vekili (TÜRKPATENT marka + CLA) önerildi.**

---

## 8. Açık sonraki adımlar (kullanıcı seçecek)

- Olay **teyit/konsensüs mantığını** netleştir (kaç düğüm, süre, eşik) — yangından depreme her şeyin ortak kritik parçası.
- Faz 0 için **somut tek-düğüm parça listesi + bağlantı şeması** (sipariş edilebilir düzey).
- **Güç bütçesini gerçek sayılarla** doğrula (sensör başına tüketim × duty-cycle → panel/batarya boyutu).
- Marka/logo notu, depo **topics** & açıklaması.
- (İstenirse) LICENSE'ı GitHub Pages/depoda görünür kılma zaten yapıldı.

---

## 9. Çalışma notları (yeni sohbet için)

- Kullanıcı **Türkçe** iletişim kuruyor; yanıtlar Türkçe.
- Oturumun ana çalışma dizini `C:\HAM\Plex`; **proje ise `C:\HAM\SaksiAg`** — git komutlarını `-C "C:/HAM/SaksiAg"` ile ya da o dizine geçerek çalıştır.
- Git Bash `gh api`'de baştaki `/`'ı dosya yoluna çevirir → endpoint'i **slash'sız** ver (`repos/...`).
- Git kimliği: `TA3HRJ` / `ta3hrj@gmail.com`. Commit'ler DCO ile imzalanıyor (`git commit -s`).
- Kullanıcının üslubu: dürüst sınırları/karşı-argümanları açıkça isteyen, mühendislik gerekçesi arayan biri. Abartıdan kaçın, "dürüst sınır" kutuları bu projenin imzası.
