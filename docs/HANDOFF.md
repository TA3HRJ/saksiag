# SaksıAğ — Proje Özeti & Devir Notu

> Bu dosya, projeyi yeni bir sohbette kaldığı yerden sürdürmek için hazırlanmış bir **bağlam devri**dir. Kararların *gerekçeleri* dahil edildi ki yeniden tartışılmasın.

**Son güncelleme:** 2026-09-24

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

**Olay teyidi (kritik):** Tek düğümün yanlış alarmı yayılmaz; olay ancak N komşu onaylayınca "doğrulanmış" sayılır. Ucuz sensörün güvenilmezliği ağ mimarisiyle telafi edilir. *(Ayrıntı: [`konsensus.md`](konsensus.md) — taslak, 2026-09-24. Özet: bağımsız konumdan k=3 oy / T / R; k canlı komşuya göre uyarlanır; yerel alarm konsensüsten muaf; yavaş tehlikeler oylamayla değil medyanla.)*

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
- **Yerel yol:** `C:\Claude Projects\saksi-ag` — dal `main`, `origin/main` ile senkron. (Eski yol `C:\HAM\SaksiAg` idi; taşındı.)
- **Claude Artifact (kullanıcıya özel, private):** https://claude.ai/code/artifact/fa8de2e0-5641-461c-a012-a1a7f13eb802 — içerik `index.html` ile aynı.

**Depo içeriği:**
- `index.html` — tek sayfalık görsel konsept (canvas mesh animasyonu dahil, tema-duyarlı)
- `README.md` — proje özeti
- `LICENSING.md`, `CONTRIBUTING.md`
- `LICENSE` (AGPL-3.0), `LICENSES/CERN-OHL-S-2.0.txt`, `LICENSES/CC-BY-SA-4.0.txt`
- `.gitignore`, `CLAUDE.md` (kalıcı proje talimatı), bu `docs/HANDOFF.md`
- `docs/konsensus.md` — olay teyidi tasarım taslağı (CC-BY-SA-4.0)
- `docs/konsensus.md` — olay teyidi tasarım taslağı (CC-BY-SA-4.0)

---

## 7. Lisanslama (kararlaştırıldı ve uygulandı)

Çok-lisanslı: **yazılım AGPL-3.0 · donanım CERN-OHL-S-2.0 · doküman CC-BY-SA-4.0.** Metinler resmi kaynaklardan birebir. GitHub `AGPL-3.0` olarak tanıyor.

Gerekçe (kullanıcının "ticariye dönerse?" sorusuna): Açık kaynak ≠ ticaret yasağı. Copyleft + telif sende + **marka ("SaksıAğ") saklı** → ticari kapı açık (çift lisans opsiyonu), rakip kapatıp koparamaz. **Önemli uyarı:** depo public olduğu için kamuya açıklama gerçekleşti → EU/TR'de patent yolu pratikte kapandı (ama bu "savunma amaçlı yayın"dır; başkası da patentleyemez). Katkı: şimdilik DCO; çift lisans ciddileşirse CLA gerekir. **Ben avukat değilim — ciddi ticari adımda IP vekili (TÜRKPATENT marka + CLA) önerildi.**

---

## 8. Açık sonraki adımlar (kullanıcı seçecek)

- ~~Olay teyit/konsensüs mantığı~~ → taslak yazıldı: `docs/konsensus.md`. Kullanıcı henüz gözden geçirmedi.
- Faz 0 için **somut tek-düğüm parça listesi + bağlantı şeması** (sipariş edilebilir düzey). Konsensüs taslağından gelen şartlar: sıcaklık sensörü **radyasyon kalkanında**; düğüm tüm ham okumaları + yerel tetikleri **zaman damgalı kaydetmeli** (yanlış tetik oranı λ Faz 0'da ölçülür); yeri ölçülecek bir zemin-seviyesi sel sensörü seçeneği.
- **Duty-cycle/band doğrulaması:** Meshtastic EU_868 ön ayarının alt bandı + BTK kısa menzilli cihaz kuralları. "~%1" varsayımı kontrol edilmedi.
- **Güç bütçesini gerçek sayılarla** doğrula (sensör başına tüketim × duty-cycle → panel/batarya boyutu).
- Marka/logo notu, depo **topics** & açıklaması.
- (İstenirse) LICENSE'ı GitHub Pages/depoda görünür kılma zaten yapıldı.

---

## 9. Çalışma notları (yeni sohbet için)

- Kullanıcı **Türkçe** iletişim kuruyor; yanıtlar Türkçe.
- Proje artık oturumun ana çalışma dizini: `C:\Claude Projects\saksi-ag`. Ayrıca `-C` ile yol vermek gerekmiyor.
- Git Bash `gh api`'de baştaki `/`'ı dosya yoluna çevirir → endpoint'i **slash'sız** ver (`repos/...`).
- Git kimliği: `TA3HX` / `136229226+TA3HRJ@users.noreply.github.com`, **yerel** olarak `.git/config`'te (global `.gitconfig` yok). Çağrı işareti 2026-09-16'da TA3HRJ → TA3HX oldu; GitHub hesap adı hâlâ `TA3HRJ`. Gerekçesi CLAUDE.md'de ve `a6e0b48` mesajında. **Commit'ler `git commit -s` ile atılır** (DCO, `CONTRIBUTING.md`; karar 2026-09-24). `f4d1574`, `f250daf`, `a6e0b48` imzasız kaldı, geçmiş yeniden yazılmadı — bkz. §10.
- Kullanıcının üslubu: dürüst sınırları/karşı-argümanları açıkça isteyen, mühendislik gerekçesi arayan biri. Abartıdan kaçın, "dürüst sınır" kutuları bu projenin imzası.

---

## 10. Oturum günlüğü

### 2026-09-06 — bakım oturumu

Kod/konsept değişmedi; yalnızca depo hijyeni.

- `OZET.md` → `docs/HANDOFF.md` olarak taşındı. Gerekçe: CLAUDE.md zaten oturum sonunda
  `docs/HANDOFF.md` güncellenmesini istiyordu, yani iki ayrı devir-notu konvansiyonu vardı.
  Tek dosyada birleştirildi.
- CLAUDE.md'deki "`OZET.md` git'te takipsiz" notu kaldırıldı — dosya `f4d1574` ile commit
  edilmişti, not eskimişti.
- Bu dosyadaki eskimiş olgular düzeltildi: yerel yol, çalışma dizini, git kimliği (§6, §9).

**Açık kalan / dikkat:** `CONTRIBUTING.md` DCO sign-off şart koşuyor; `ea8240f` imzalı ama
`f4d1574`, `f250daf` ve `a6e0b48` imzasız; `ea8240f`'teki sign-off adresi eski gmail adresi. Geçmişi yeniden yazmak
yerine bundan sonrasının `git commit -s` ile atılması ve adresin noreply olması yeterli —
ama bu bir karar, henüz verilmedi. *(2026-09-24'te verildi: bundan sonra `-s`.)*

### 2026-09-24 — konsensüs taslağı

Kullanıcı §8'den konsensüs mantığını seçti. Çıktı `docs/konsensus.md` (CC-BY-SA-4.0); README'deki
"Olay teyidi" paragrafından bağlandı. Varılan tasarım kararları ve gerekçeleri:

- **Konsensüs yalnızca bağımsız hataları çözer.** Ortak nedenli hatalarda (Sahra tozu, öğle
  güneşi, havai fişek) ağ yanlış sonuçta uzlaşır. Bunlara karşı savunma k değil, düğüm içi
  füzyon ve mekânsal biçim testi. Belgenin omurgası bu ayrım.
- **Yerel alarm konsensüsten muaf.** Tek dairenin yangını k balkona ulaşmayabilir; konsensüs
  yalnızca mahalleye *yayma* kararını verir.
- **k=3 gerekçesi hesapla konuldu:** Formül `C(M,k)·k·λ^k·T^(k−1)`, Monte Carlo ile doğrulandı.
  k=3 ancak düğüm başı yanlış şüphe haftada birin altındaysa yetiyor. Bu yüzden gürültülü düğüm
  karantinası zorunlu hâle geldi. k'nın üst sınırını yanlış alarm değil, gerçek olayın kaç
  balkona ulaştığı belirliyor; bu sayı bilinmiyor.
- **Oy "bağımsız konum" başına, düğüm kimliği başına değil.** Komşuluk hop'la değil, kurulumda
  telefondan atanan kaba hücreyle ölçülüyor. Kesin koordinat saklanmıyor; bu bilinçli bir
  gizlilik tercihi.
- **Lider yok:** Her düğüm oyları kendisi sayar; olay kimliği deterministik hash olduğu için
  aynı anda ilan eden iki düğüm aynı kimliği üretir.
- **Yavaş tehlikelerde (sıcak dalga, don, fırtına) oylama yok, komşu medyanı var.**

**Bu oturumda yakalanan tutarsızlıklar (README'ye dokunulmadı, belgede açıkça yazıldı):**
- README yangın için "CO" diyor ama gaz sensörü v1.1'de. v1'de bu bacağı BME680 VOC karşılıyor
  (§5.1).
- Sel "güçlü" olarak listelenmiş, ama balkon düğümü sokaktaki suyu göremez. Gördüğü yağış
  şiddeti. P0 sel alarmı ancak zemin seviyesindeki düğümlerden doğabilir (§5.2). README'deki
  sel iddiası yumuşatılmalı mı? Karar kullanıcıda.
- Deprem için ek gerekçe: LoRa ile oy toplamak saniyeler sürüyor ve GPS'siz saatler kayıyor.
  Önceki "arka plan katkısı" kararını güçlendiriyor, genişletmiyor.

**Tuzak:** Makinede `python` komutu Microsoft Store kısayoluna düşüyor. Çalışan yorumlayıcı
`C:\Users\Admin\AppData\Local\Programs\Python\Python313\python.exe`.

**DCO kararı verildi:** Kullanıcı bundan sonraki commit'lerin `git commit -s` ile atılmasını
onayladı. Sign-off, yerel kimlikten gelir (`TA3HX <136229226+TA3HRJ@...>`). Geçmiş yeniden
yazılmadı; eski imzasız commit'ler olduğu gibi kaldı. İlk imzalı commit'ler: `685eab8` (taşıma),
`0844acc` (konsensüs taslağı).

**Açık:** Taslak henüz gözden geçirilmedi.

### 2026-09-25 — README düzeltmeleri

Kullanıcı yukarıdaki iki tutarsızlığın README'de düzeltilmesini onayladı:
- Sel satırı ikiye ayrıldı: *şiddetli yağış → sel riski* 🟢 (hazne doluş hızı) ve *su baskını*
  🟡 (yalnızca zemin seviyesindeki düğümlerle). Tablonun altına deprem notu biçiminde bir
  "Sel notu" eklendi.
- Yangın sensörü `sıcaklık + PM2.5 + VOC (BME680) · CO v1.1` oldu. Aynı gerekçeyle zehirli
  bulut satırı da *(v1.1)* olarak işaretlendi, çünkü gaz sensörü v1.1'de.

Ardından kullanıcının isteğiyle `index.html` (canlı sayfa) de aynı biçimde düzeltildi: afet
tablosu, paylaşımlı sensörler paragrafı ve su seviyesi parça satırı. Ayrıca deprem kutusuyla aynı
stilde bir "Dürüst sınır: sel" kutusu eklendi. Kutu, konsensüs taslağına GitHub blob
bağlantısıyla gidiyor. Pages'te `.md` dosyasına göreli bağlantı güvenilir biçimde işlemediği
için göreli bağlantı kullanılmadı. HANDOFF §3'teki "🟢 Güçlü: sel" ifadesi tarihsel karar kaydı
olarak bırakıldı.
