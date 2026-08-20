---
name: "uret"
description: "Görsel ve video üretir — işi en ucuz yetkin modele yönlendirir, sağlayıcı API'sine gider, çıktıyı tek bir düz klasöre kaydeder ve yanına promptunu içeren JSON kaydı bırakır. \"Görsel üret\", \"video oluştur\", \"thumbnail yap\", \"reklam görseli\", \"logo/afiş\", \"ürün fotoğrafı\", \"reels\", \"şunu canlandır\", \"varyasyon çıkar\" gibi her üretim isteğinde kullan. Ödemeli video çağrısı öncesi maliyeti söyler ve onay bekler. API anahtarı gerektirir."
---

# Üret — Görsel ve Video

Tek komut: işi en ucuz yetkin modele yönlendirir, üretir, tek klasöre kaydeder,
promptu asla kaybetmez.

> Mimari, RoboNuggets'ın ücretsiz `/generate` build guide'ından uyarlandı
> (skool.com/robonuggets). Model kimlikleri ve fiyatlar o rehberin yazıldığı
> tarihe ait — **kullanmadan önce sağlayıcının sayfasından doğrula.**

## Dört adım
**1. Yönlendir** → iş için modeli ve onu çalıştıran en ucuz sağlayıcıyı seç,
o modelin reçete dosyasını oku.
**2. Referansları hazırla** → `refs/` içinden gerçek görselleri yükle.
**3. Üret** → reçeteye göre API'yi çağır, async ise bekle, dosyayı düz klasöre kaydet.
**4. Kaydet** → yanına JSON kaydı yaz: hangi prompt, model, ayar.

## Klasör
```
uret/
├── models/          her model için bir reçete dosyası
├── generations/     TÜM çıktılar burada, DÜZ, alt klasör YOK
│   └── refs/        logo, yüz, stil referansları
└── ayar.json        klasör yolu + sağlayıcı tercihleri
```
`.env` anahtarları **git'e girmez.**

## ÖN KOŞUL — ilk çalıştırmada
Claude'un kendi içinde görsel/video üretimi **yok**; iş dış sağlayıcıya gider.

Üç toplayıcı, tek anahtarla çok modele erişim:

| Sağlayıcı | Anahtar | Neden |
|---|---|---|
| Kie AI (kie.ai) | `KIE_API_KEY` | Popüler modellerde çoğu zaman en ucuz rota |
| fal.ai | `FAL_KEY` | Hızlı, geniş katalog, en iyi dokümantasyon — ilk yedek |
| WaveSpeed (wavespeed.ai) | `WAVESPEED_API_KEY` | İkinci yedek |

**Kimlik doğrulama üç farklı şekilde — en çok takılınan yer:**
- Google AI Studio → anahtar **URL'de**: `...:generateContent?key={KEY}`
- fal.ai → başlıkta **`Authorization: Key {FAL_KEY}`**
- Kie AI → başlıkta **`Authorization: Bearer {KIE_KEY}`**

Anahtarları **ortam değişkeninde** tut. **Anahtarı isteme, görme, dosyaya yazma** —
sadece `[ -n "$FAL_KEY" ]` ile var mı diye bak. Yoksa: neyin gerektiğini,
nereden alınacağını söyle ve **dur**. Sahte çıktı üretme.

Sandbox'ın o API'ye erişimi olmayabilir. İlk çağrıdan önce ucuz bir istekle
bağlantıyı test et; erişim yoksa açıkça söyle. "Muhtemelen çalışır" deme.

## Reçete dosyası — `models/<model>.md`
Her model için bir kez, on dakika:
```
| Model ID | Provider | Method (Sync/Async) | Type | API key | Docs | Cost |
## Endpoint      POST https://...
## Request       (dokümandaki tam JSON gövdesi)
## Response      (dosya nerede: base64 alanı mı, URL mi; async ise
                  status endpoint'i ve "bitti" diyen alan)
## Notes         (rate limit, boyut sınırı, içerik kuralları)
```
**Async kalıbı** (video modellerinin çoğu): POST → task id → 10-15 sn'de bir
durum sor → tamam deyince dosya URL'i gelir → **hemen indir** (URL'ler saatler
içinde ölür) → kaydet → log yaz.

Sync mi async mi olduğunu yanlış bilmek, ilk denemenin "takılmış" görünmesinin
bir numaralı sebebidir.

## Model seçimi
İkisiyle başla: **ucuz bir görsel modeli** + **bir video modeli**. Biri sync biri
async olduğu için iki kalıbı da öğretir.

| İş | Ne ara |
|---|---|
| Günlük görsel, referansla çalışma | Ucuz, hızlı, referans desteği güçlü görsel modeli |
| **Görselin içinde okunaklı metin** (tabela, afiş, menü, UI) | Metin konusunda iyi olan görsel modeli — bu ayrı bir yetenek |
| Genel video | Makul fiyatlı, hareketi iyi video modeli |
| Başlangıç karesinden yüksek kalite video | Kaliteli ama yavaş video modeli |
| Referans görselleri canlandırma | Reference-to-video destekleyen model |

`models/` içinde reçetesi olmayan modeli çağırma. "Model not found" hatası
alırsan sağlayıcının sayfasından kimliği tazele ve reçeteyi güncelle —
sistemin tek bakım işi budur.

## Maliyet (rehberin tarihine ait kaba değerler — doğrula)
| İş | Aralık |
|---|---|
| Taslak görsel, ucuz model | $0.01 – $0.03 · varsayılanın bu, burada özgürce dene |
| Kaliteli görsel, üst model | $0.05 – $0.15 · sadece finaller |
| Video, saniye başı | $0.20 – $0.35 · 10 sn = $2–3.5 · maliyet kapısı bu yüzden var |

## KURALLAR — sistemin asıl değeri bunlar

**1. Videodan önce fiyat söyle, onay bekle.** Model, süre, çözünürlük ve
beklenen tutarı yaz, dur. *Fiyat söylemek onay değildir. Bir onay = bir çalıştırma.*

**2. Ucuza taslak, pahalıya final.** Ucuz modelde dene. Kullanıcı bir favori
seçtiğinde aynı promptu kaliteli modelde tekrarla. Çöpe gidecek taslaklara
premium ödeme.

**3. Logo ve yüz asla kelimeyle tarif edilmez.** Tarif edilen logo her seferinde
yanlış gelir — yanlış şekil, yanlış renk, uydurma detay. Gerçek dosyayı
`generations/refs/` içinde tut ve API çağrısına referans olarak geçir.
**Dosya yoksa dur ve kullanıcıdan iste.**

**4. Tek düz klasör.** Her çıktı aynı klasöre, alt klasör yok. Klasörlemek
düzenli hissettirir ama kütüphaneyi okuyan her aracı bozar ve zaten altı ay
sonra sistemi hatırlamazsın.
İsimlendirme: `{proje}_{aciklama}_{zaman}.{uzanti}`

**5. Sağlayıcı değişimini gizleme.** Hangi rotanın çalıştığını ve neden
o rotaya düşüldüğünü söyle.

**6. Aynı anda tek üretim.** Rate limit yersin.

**7. Her kayıttan sonra sidecar log.** Aynı isim + `.json`:
```json
{ "model": "...", "prompt": "API'ye giden tam metin",
  "refs": ["refs/logo.png"], "params": {"aspect":"16:9","size":"2K"},
  "created": "2026-08-11T09:41:00Z" }
```
Üç hafta sonra "bunu hangi prompt üretmişti" sorusunun tek cevabı bu.

## Prompt yazımı — kalite farkı burada
Zayıf: *"kahve reklamı görseli"*. İyi prompt beşini içerir:
**özne** (ne, hangi durumda, hangi açıdan) · **ortam** · **ışık** (sert/yumuşak,
yön, saat) · **kamera** (lens hissi, derinlik, kadraj) · **stil** (film/render/
illüstrasyon, palet, doku).
Ayrıca **negatif** yaz: watermark, bozuk el, istenmeyen metin, aşırı doygunluk.
Video için ek: kamera hareketi (sabit / yavaş kaydırma / dolly-in), süre,
sahnedeki hareket.

En/boy: 1:1 feed · 9:16 reels/story · 16:9 YouTube/web · 4:5 dikey feed.

Kampanya tutarlılığı için `uret/stiller/<isim>.md` içinde stil kartı tut
(palet, ışık, lens, doku, negatifler) ve her promptun sonuna ekle.

## Galeri
Kullanıcı isterse `generations/` klasörünü okuyan tek dosyalık bir bento/masonry
duvar üret: en yeni üstte, videolar hover'da sessizce oynar, tıklayınca büyür,
filtre/sekme yok. Klasör düz olduğu için galeri hiç güncellenmez — yeni üretimler
kendiliğinden görünür.

## Sınırlar
- Onaysız para harcama.
- Üretemiyorsan söyle; placeholder dosya oluşturma.
- Gerçek kişilerin benzerliğini üretme, marka logolarını taklit etme.
- Telifli bir eserin stilini birebir kopyalamaya çalışma.
- Yayına gidecek içerikte yapay zeka etiketi gerekip gerekmediğini hatırlat.
- Türkçe yaz.
