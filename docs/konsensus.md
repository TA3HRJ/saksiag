# Olay teyidi (komşu konsensüsü) — tasarım taslağı

> ⚠️ **Durum:** Tasarım taslağı. Buradaki bütün eşikler, pencereler ve yarıçaplar **başlangıç
> tahminidir**; hiçbiri ölçülmedi. Faz 0 (tek düğüm) yanlış tetik oranını, Faz 1 (sokak mesh'i)
> gerçek olayların kaç balkona ulaştığını ölçecek ve bu sayılar ona göre değişecek.

İlke (README'den): *Tek düğümün yanlış alarmı yayılmaz; olay ancak birkaç komşu onaylayınca
"doğrulanmış" sayılır.* Bu belge "birkaç", "komşu", "aynı anda" ve "onay" kavramlarının
ne anlama geldiğini tanımlıyor.

---

## 1. Konsensüs neyi çözer, neyi çözmez

Bu bölüm belgenin en önemli kısmı. Parametreler bunun üstüne kuruluyor.

**Çözdüğü: bağımsız hatalar.** Bozuk ya da kaymış bir sensör, saksının yanında içilen sigara, tek
balkondaki mangal, sulamadan sonraki nem sıçraması. Bunlar balkondan balkona ilişkisizdir.
Aynı dakikalarda üç ayrı balkonda rastlantıyla üst üste gelme olasılıkları çok küçüktür (§4).

**Çözmediği: ortak nedenli hatalar.** Bazı olaylar bütün düğümleri *aynı anda ve aynı yönde*
yanıltır:

- **Toz taşınımı** (ör. Sahra tozu): Bölgedeki bütün PM sensörleri birlikte yükselir.
- **Öğle güneşi:** Güney cepheli bütün balkonlarda sıcaklık 10+ °C fazla okunur.
- **Bayram havai fişeği, anız yakma dumanı:** Geniş alanda gerçek duman var ama yangın yok.

Bu durumlarda komşu sayısını artırmak işe yaramaz. Ağ yanlış sonuçta uzlaşır, üstelik daha
güvenli görünür. Bunlara karşı konsensüs değil iki ayrı savunma gerekiyor:

1. **Düğüm içi çok-sensör füzyonu.** Yangın şüphesi yalnızca PM'e dayanmaz. PM artışı, VOC/gaz
   artışı ve sıcaklık artış hızı birlikte aranır. Toz tek başına PM'i yükseltir, VOC'yi yükseltmez.
2. **Mekânsal biçim testi.** Nokta kaynaklı olay (yangın) komşuların *bir kısmında*, kaynaktan
   uzaklaştıkça azalarak görünür. Bölgesel olay (toz, sıcaklık) *hemen hepsinde* ve birbirine
   yakın düzeyde görünür. R içindeki canlı düğümlerin büyük çoğunluğu (başlangıç tahmini %70'ten
   fazlası) ve komşu hücreler de aynı anda şüphedeyse olay "yangın" olarak etiketlenmez.
   "Bölgesel partikül / sıcaklık" etiketi alır ve P2 olarak yayılır. *Bu sezgisel bir kural;
   ucuz ve güvenilir bir biçim testi açık bir konu (§10).*

**Ters yönde bir sınır: çok yerel gerçek olay bastırılır.** Tek bir dairedeki yangının dumanı k
bağımsız balkona ulaşmayabilir. O durumda konsensüs gerçek olayı da süzer. Bu yüzden yerel alarm
katmanı konsensüsten **muaftır** (§2). Balkondaki saksı, duman dedektörü gibi kendi sahibini her
durumda uyarır. Konsensüs yalnızca *mahalleye yayma* kararını verir.

---

## 2. Üç alarm katmanı

| Katman | Kim karar verir | Nereye gider | Öncelik / menzil |
|---|---|---|---|
| **Yerel** | Düğümün kendisi | Kendi LED/buzzer'ı + eşleşmiş telefon (BLE) | Yayın yok |
| **Şüphe** | Düğümün kendisi | Yakın komşular | P0 kuyruğu, **kısa menzil** (hop 1–2) |
| **Doğrulanmış olay** | Oyları sayan herhangi bir düğüm | Bütün mesh + ağ geçidi | P0, geniş menzil |

- *Şüphe* mesajı bir alarm değil, konsensüsün girdisidir. Hızlı olmalı, bu yüzden P0 kuyruğunda
  gider. Ama uzağa gitmemeli, çünkü oylar yereldir ve kanal ortaktır (§7).
- **Teyitsiz şüphe:** Düğümün yeterince canlı bağımsız komşusu yoksa (§3) teyit mümkün
  değildir. Şüphe ağ geçidine ulaşırsa haritada "teyitsiz" olarak gösterilir. P0 olarak
  yayılmaz. Seyrek bölgede sessiz kalmaktansa belirsizliği açıkça göstermek tercih edildi.

---

## 3. Temel kural

> T zaman penceresi içinde, R yarıçapında, aynı türde en az **k bağımsız konumdan** şüphe
> duyulursa olay **doğrulanmış** sayılır.

**Bağımsız konum, düğüm kimliği değildir.** Aynı balkondaki iki saksının hataları ortaktır
(aynı güneş, aynı sulayan kişi, aynı mangal). Bu yüzden aynı kurulum/sahip ya da birbirine çok
yakın (başlangıç tahmini <15 m) düğümler **tek oy** sayılır. Bu bilgi kurulumda telefonla (BLE)
atanır.

**Komşuluk hop sayısıyla değil, mesafeyle ölçülür.** LoRa'da tek atlama kilometreleri
aşabilir. "1 hop uzakta" bilgisi "yakında" anlamına gelmez. Her düğüme kurulumda telefonun
konumundan **kaba bir hücre** (ör. geohash, ~150 m) atanır. Kesin koordinat saklanmaz; bu hem
yeterli hem de kimin nerede oturduğunu yaymamak için bilinçli bir tercih.

**k, canlı komşu sayısına göre uyarlanır.** Her düğüm, R içindeki bağımsız konumlardan son
heartbeat'i duyulanları "canlı" sayar (kendisi dahil):

| Canlı bağımsız konum (R içinde) | k |
|---|---|
| ≥ 4 | 3 |
| 2–3 | 2 |
| 1 (yalnız) | teyit yok → *teyitsiz şüphe* |

Az komşulu bölgede k'yı düşürmek yanlış alarmı patlatmaz, çünkü aynı bölgede M de küçüktür.
Tablodaki M=10, k=2 satırına bakın (§4).

---

## 4. Neden k = 3? (sayılar)

**Model:** Her düğüm, gerçek olay yokken birbirinden **bağımsız** biçimde λ oranında yanlış
şüphe üretiyor. R içinde M bağımsız konum var ve pencere T = 10 dk. k farklı konumun T içinde
rastlantıyla üst üste gelme oranı yaklaşık `C(M,k) · k · λ^k · T^(k−1)`. Formül Monte Carlo
simülasyonuyla kontrol edildi ve %20 içinde tuttu. Simülasyon, alarmdan sonraki kilitlenmeyi
de hesaba kattığı için biraz daha düşük çıkıyor.

**Beklenen yanlış *ağ* alarmı / yıl (T = 10 dk):**

| Düğüm başı yanlış şüphe λ | M | k = 2 | k = 3 | k = 4 |
|---|---|---|---|---|
| ayda 1 | 10 | 0,25 | 0,0002 | ~0 |
| ayda 1 | 20 | 1,1 | 0,002 | ~0 |
| ayda 1 | 50 | 6,9 | 0,04 | 0,0001 |
| haftada 1 | 10 | 4,7 | 0,02 | ~0 |
| haftada 1 | 20 | 20 | 0,18 | 0,001 |
| haftada 1 | 50 | 130 | 3 | 0,05 |
| günde 1 | 20 | 960 | 60 | 2,4 |
| günde 1 | 50 | 6 200 | 1 000 | 110 |

**Tablodan çıkanlar:**

- **k = 2** yalnızca çok seyrek bölgede kabul edilebilir. Bu yüzden §3'teki uyarlama k=2'yi
  yalnızca az komşu olduğunda kullanıyor.
- **k = 3** yeterli, ama **tek bir şartla:** düğüm başı yanlış şüphe haftada birin altında
  kalmalı. Günde bir yanlış şüphe üreten düğümlerle hiçbir makul k kurtarmaz. Bu yüzden
  gürültülü düğüm karantinası (§8) isteğe bağlı değil, zorunlu.
- **k = 4** daha güvenli görünüyor ama bedeli var: gerçek bir duman bulutunun 4 *bağımsız*
  balkona ulaşması gerekir. Seyrek bölgede bu, gerçek olayı kaçırmak demek. **k'nın üst sınırını
  yanlış alarm değil, gerçek olayın kaç balkona ulaştığı belirler.** Bu sayı bilinmiyor, Faz 1'de
  ölçülecek.
- M, *bütün ağ* değil, R içindeki konum sayısıdır. Ağ büyüdükçe R içindeki yoğunluk da artar.
  Faz 3 yoğunluğunda k'nın sabit mi kalacağı yoksa M'ye oranla mı büyüyeceği açık bir soru.

> **Dürüst sınır:** Tablonun tamamı *bağımsızlık* varsayımına dayanıyor. §1'deki ortak nedenli
> hatalarda gerçek oran bu tablodakinden kat kat yüksek olur. O hatalara karşı savunma k değil,
> füzyon ve biçim testidir.

---

## 5. Tehlike başına parametreler (başlangıç tahmini)

Tehlikelerin hepsi aynı kuralla doğrulanmıyor. Hızlı ve nokta kaynaklı tehlikeler **oylamayla**,
yavaş ve bölgesel tehlikeler **dayanıklı istatistikle** (medyan) izleniyor.

| Tehlike | Düğüm içi tetik (füzyon) | Kalıcılık | T | R | Yöntem | Çıktı |
|---|---|---|---|---|---|---|
| **Yangın / duman** | PM2.5 sıçraması **ve** (VOC artışı **veya** sıcaklık artış hızı) | 60 sn | 10 dk | ~500 m | oylama, k=3 | P0 + biçim testi |
| **Zehirli bulut** *(v1.1)* | gaz sensörü eşiği + artış hızı | 60 sn | 10 dk | ~500 m | oylama, k=3 | P0 |
| **Şiddetli yağış → sel riski** | hazne doluş hızı (yağış şiddeti) | 10 dk | 30 dk | ~1 km | oylama, k=3 | P2 "şiddetli yağış" |
| **Sel (su baskını)** | zemin seviyesi su sensörü | 2 dk | 30 dk | ~1 km | oylama, k=2 | P0 |
| **Sıcak hava dalgası** | — | — | gün | mahalle | komşu **medyanı** | P2 bülten |
| **Don** | — | — | saat | mahalle | komşu medyanı | P2 |
| **Fırtına** | basınç düşüş hızı | — | saat | geniş | komşu medyanı | P2 |
| **Deprem** | ivmeölçer | — | 2–5 sn | geniş | ağ geçidi tarafında | §5.3 |

### 5.1 Yangın: v1 donanımında CO yok

README yangın sensörü olarak "sıcaklık + PM2.5 + CO" diyor. Oysa malzeme listesinde gaz
sensörü v1.1'de. v1'de "gaz" bacağı BME680'in VOC ölçümüyle (gaz direnci) sağlanıyor. VOC,
CO'ya özgü değil ve yavaş tepki veriyor. Füzyon kuralı bu yüzden VOC **veya** sıcaklık artış
hızı diyor. v1.1'de CO eklenince kural sıkılaşmalı.

### 5.2 Sel: balkondan su baskını görülmez

Düğümlerin çoğu üst katlarda. Balkondaki bir saksı sokaktaki suyu ölçemez. Yağmuru ölçebilir,
çünkü haznenin doluş hızı yağış şiddetini verir. Bu yüzden sel iki ayrı olaya bölündü:

- **Şiddetli yağış:** Çok düğüm görür, oylamayla doğrulanır. Bir *risk* bildirir, P2.
- **Su baskını:** Yalnızca zemin seviyesinde (giriş kat, bahçe, bodrum girişi) konumlanmış
  düğümler görür. Bunlar az olacağı için k=2. P0 çıktısı yalnızca buradan doğar.

Yani sel için "güçlü" iddiası, zemin seviyesinde düğüm bulunmasına bağlı. Bu, düğüm
yerleşimine dair bir gereklilik ve Faz 1'de sınanmalı.

### 5.3 Deprem: mesh konsensüsü ön uyarı için kullanılmaz

Önceki kararla uyumlu (arka plan katkısı, öne çıkarılmaz). Buraya bir ek gerekçe geliyor. Deprem
teyidi saniye ölçeğinde bir pencere ister. LoRa üzerinden oy toplamak ise tek atlamada bile
~0,6 sn sürüyor (§7), üstüne kuyruk ve rastgele bekleme ekleniyor. Saati GPS'siz düğümlerin zaman
damgaları da saniyeler kayabilir. Bu yüzden:

- Ön uyarıya katkı ancak **internete bağlı ağ geçitlerinde** mümkün. Bunun için geçidin kendi
  ivme kaydını ya da geçide doğrudan bağlı düğümlerin tetiklerini sunucu tarafında birleştirmek
  gerekir. Bu, ulusal sistemleri besleyen bir veri akışıdır; SaksıAğ'ın kendi alarmı değildir.
- Mesh içindeki konsensüs deprem için yalnızca **afet sonrası teyit** işini görür ("bu sarsıntı
  mahallenin geneli miydi, benim saksıma çarpan biri miydi?"). Bu teyidin saniyelerce
  gecikmesinin bir sakıncası yok.

### 5.4 Medyan da ortak hatayı düzeltmez

Sıcak dalga ve don için medyan, tek tük arızalı sensörü görmezden gelir. Ama güney cepheli
balkonların çoğunluğu güneşte ısınıyorsa medyan da yanlış çıkar. Bu, konsensüsle değil
donanımla çözülür: sıcaklık sensörü radyasyon kalkanı içinde, gölgede olmalı. Bu not Faz 0
parça listesine gidecek.

---

## 6. Durum makinesi ve mesajlar

Oyları sayan bir lider yok. **Her düğüm duyduğu şüpheleri kendisi sayar.** Böylece tek arıza
noktası oluşmaz ve ağ geçidi ya da internet çökse de teyit çalışır. Afet senaryosunun asıl
şartı da budur.

```
NORMAL ──(yerel tetik, kalıcılık süresince sürdü)──▶ ŞÜPHE
   ▲                                                  │  yerel alarm + ŞÜPHE mesajı (hop 1–2)
   │                                                  ▼
   └──(T süresince yeni tetik yok)──────────────── ŞÜPHE
                                                      │
          (T içinde, R içinde ≥ k bağımsız konumdan aynı tür ŞÜPHE duyuldu)
                                                      ▼
                                               DOĞRULANMIŞ ── OLAY mesajı (geniş menzil)
                                                      │
                             (R içinde şüphedeki konum < k, T_bitiş boyunca)
                                                      ▼
                                                  SONA ERDİ
```

Düğüm, kendisi şüphede olmasa bile oy sayabilir. Rüzgâr üstündeki bir balkon dumanı görmez ama
altındakilerin oylarını duyar.

**Mesajlar** (hepsi *kenar-tetiklemeli*: yalnızca durum değişince gönderilir, periyodik değil):

| Mesaj | Alanlar (taslak) | Menzil |
|---|---|---|
| `SUPHE` | tür · hücre · zaman · şiddet (1 bayt) · konum-kimliği | hop 1–2 |
| `OLAY` | tür · hücre · olay-kimliği · oy sayısı · ilk şüphe zamanı | geniş |
| `BITTI` | olay-kimliği | geniş |

- **İptal mesajı yok.** Şüphe T sonunda kendiliğinden düşer. Bu bir paket tasarrufu.
- **Olay kimliği deterministiktir:** `hash(tür, hücre, zaman kovası)`. k eşiğine aynı anda
  ulaşan iki düğüm aynı kimliği üretir, böylece çift alarm oluşmaz. Kova sınırına denk gelen
  nadir çakışmaları ağ geçidi (varsa) birleştirir.
- Aynı tür ve hücre için `OLAY` duyan düğüm, T_tutma süresince yeniden ilan etmez.

---

## 7. Yayın süresi bütçesi

Meshtastic LongFast ön ayarında (SF11 · 250 kHz · CR 4/5 · 16 sembol önsöz) 40–60 baytlık bir
paketin yayını, Semtech formülüne göre **~0,56–0,68 sn** sürüyor (hesaplandı, ölçülmedi).
Daha kısa ön ayarlarda bu süre ~0,2 sn'ye (SF9) ya da ~0,05 sn'ye (SF7) iniyor, ama menzil de
kısalıyor.

- Asıl darboğaz düğüm başına duty-cycle değil, **ortak kanal**. Menzildeki herkes aynı havayı
  paylaşıyor. Olay anında R içinde 20 düğüm şüphe gönderse ve her paket ~3 kez iletilse
  toplam ~36 sn kanal süresi harcanır. 10 dakikalık pencerede bu ~%6 kullanım demek. Kabul
  edilebilir.
- Aynı düğümler şüpheyi **periyodik** gönderseydi (ör. dakikada bir), aynı pencerede kanal
  doyardı ve P1 insan mesajlarına yer kalmazdı. Kenar tetikleme ve kısa hop bu yüzden zorunlu.
- **Doğrulanacak:** HANDOFF ~%1 duty-cycle varsayıyor. Meshtastic'in EU_868 ön ayarının
  kullandığı alt bandın izin verdiği duty-cycle ve ERP ile Türkiye'deki (BTK) kısa menzilli
  cihaz kuralları henüz kontrol edilmedi. Sonuç bu bütçeyi değiştirebilir.

---

## 8. Gürültülü düğüm karantinası

§4'teki tablo, düğüm başı yanlış şüphenin düşük kalmasını *şart* koşuyor. Kaymış bir PM
sensörü ya da hep güneşte kalan bir saksı bu şartı bozar.

- Her düğüm, son 7 günde **doğrulanmış olaya dönüşmeyen** şüphelerini sayar. Sayı eşiği
  (başlangıç tahmini: 3) geçerse düğüm **kendi oyunu susturur**. Yerel alarmı çalışmaya devam
  eder.
- Bu öz-denetim güvenilir, çünkü ölçüt dışarıdan gelen kanıta dayanıyor: komşulardan olay
  duyulmadı. Sensör bozuk olsa bile düğüm bunu sayabilir.
- Komşular da aynı sayımı konum-kimliği başına tutar. Susması gereken ama susmayan bir düğümün
  oyunu kendileri yok sayar.
- Susturulan düğüm sahibine LED ya da telefon üzerinden "sensörü kontrol et" der. Bakım, sulama
  alışkanlığına biner (README'deki saksı gerekçesi burada da işe yarıyor).

---

## 9. Güvenlik ve sorumluluk sınırı

- **Kötü niyetli düğüm:** Konsensüs *arızaya* karşı koruma sağlar, *saldırıya* karşı sağlamaz.
  Kanal anahtarını bilen biri k sahte konum kimliği uydurup sahte P0 alarmı üretebilir (Sybil
  saldırısı). Düğüm başına imza, kurulumda telefonla kayıt ve ağ geçidi tarafında tutarlılık
  denetimi gibi seçenekler var; hiçbiri henüz seçilmedi.
- **Mesaj bir gözlemdir, emir değildir.** Doğrulanmış olay bile "X hücresinde 4 bağımsız düğüm
  duman gördü" biçiminde iletilir. SaksıAğ "tahliye edin" demez. Eylem talimatı yetkili
  kurumların (AFAD, itfaiye) işidir. Bu hem hukuki hem ahlaki bir sınır: ucuz sensörlerle
  kurulmuş bir topluluk ağı resmi uyarı sistemi gibi davranmamalı.

---

## 10. Açık sorular (ölçümle kapanacak)

| Soru | Nerede ölçülür |
|---|---|
| Düğüm başı yanlış şüphe oranı λ, sensör ve tehlike türüne göre | **Faz 0.** Tek düğüm aylarca ham veri kaydeder. Faz 0'ın doğal çıktısı. |
| Gerçek bir duman bulutu kaç bağımsız balkona ulaşıyor? (k'nın üst sınırı) | Faz 1, kontrollü duman denemesi |
| Mekânsal biçim testi için ucuz ve güvenilir bir kural | Faz 1 verisi |
| Sel için zemin seviyesi düğüm yoğunluğu ne olmalı? | Faz 1 yerleşim planı |
| EU_868 alt bandı / BTK duty-cycle ve ERP sınırları | Mevzuat taraması |
| Düğüm kimlik doğrulaması (Sybil) | Faz 2 öncesi karar |
| Faz 3 yoğunluğunda k sabit mi, M'ye oranlı mı? | Faz 2–3 verisi |

**Faz 0'a doğrudan etkisi:** Tek düğüm yalnızca "yaşıyor mu?" sorusunu yanıtlamakla kalmamalı.
Bütün sensörlerin ham okumalarını ve yerel tetiklerini zaman damgasıyla kaydetmeli. λ ancak
böyle ölçülür, ve §4'teki bütün tablo λ'ya bağlı.

---

*Bu belge [CC-BY-SA-4.0](../LICENSES/CC-BY-SA-4.0.txt) lisanslıdır (bkz. [`LICENSING.md`](../LICENSING.md)).*
