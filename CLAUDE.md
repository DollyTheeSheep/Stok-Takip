# Stok Takip — Proje Notları

Kore ürünleri satan bir dükkan için iPad'de kullanılacak stok, satış ve rapor uygulaması.
Kullanıcıyla Türkçe konuş. Kullanıcı yazılımcı değil; açıklamaları sade tut.

## Yapı
- Tek dosya: `index.html` (HTML + CSS + JS bir arada, harici kütüphane yok — GIS script tag'i hariç). Bu yapıyı koru.
- Veri tarayıcıda `localStorage` içinde (anahtar: `stokTakipV1`), girişliyse aynı veri Google Drive'da da tutulur (bkz. Google Drive senkronizasyonu).
- Tasarım iOS tarzında: alt sekme çubuğu, büyük başlık, gruplu listeler, otomatik karanlık mod.
- **Canlı site (asıl kullanım yeri):** https://dollytheesheep.github.io/Stok-Takip/ — repo: https://github.com/DollyTheeSheep/Stok-Takip (main dalı, GitHub Pages ile yayında). `git push origin main` sonrası birkaç dakika içinde günceller.
- Claude artifact sürümü de var ama artık ikincil: https://claude.ai/artifact/PJxjJ63DA4HC17v2QzVJ1X — orada Google script'i CSP tarafından engellendiği için Drive senkronizasyonu çalışmaz, sadece localStorage ile çalışır.

## Google Drive senkronizasyonu
- Google Cloud projesi: "Stok Takip" (ID: `molten-verve-511916-m9`), hesap: ddkandogan@gmail.com. Drive API etkin.
- OAuth onay ekranı "External" + "Testing" modunda; test kullanıcısı olarak sadece ddkandogan@gmail.com eklendi (100 kullanıcıya kadar test kullanıcısı eklenebilir, Google Cloud Console > Google Auth Platform > Audience).
- OAuth istemci kimliği (Web application, adı "Stok Takip Web"): yetkili JavaScript kaynakları `https://dollytheesheep.github.io` ve `http://localhost:8731` (yerel test için). Yeni bir adrese taşınırsa Google Cloud Console > Clients'tan bu kaynak listesine eklenmeli.
- Çalışma şekli: `index.html` içindeki `GOOGLE_CLIENT_ID` sabiti ile Google Identity Services (GIS) token client kurulur, `drive.file` izniyle sadece uygulamanın oluşturduğu `stok-takip-verisi.json` dosyasına erişilir. Giriş yapılınca Drive'daki dosya indirilir (yoksa mevcut yerel veriyle oluşturulur); her `save()` çağrısında (satış, ürün ekleme/düzenleme, raf doldurma, "Gördüm" vb.) 1.2 sn gecikmeyle Drive'a da yazılır (`scheduleDrivePush`).
- Sayfa açılışında daha önce giriş yapılmışsa sessizce yeniden bağlanmaya çalışır; bu kullanıcı etkileşimi içermediğinden tarayıcı popup'ı engelleyebilir — bu yüzden 4 saniyelik zaman aşımı var (`driveEnsureToken`), süre dolunca sessizce normal "Google ile giriş yap" durumuna döner (sonsuz "Senkronize ediliyor…" kalmaz).
- Birden fazla cihaz aynı anda yazarsa son yazan kazanır (basit "son yazan kazanır", çakışma birleştirme yok) — tek kişilik bir dükkan için yeterli kabul edildi.

## Veri modeli
```js
{
  products: [{ id, name, cat, cost, price, depot, shelf, min, dMin, ack, ackD }],
  // depot = depodaki adet, shelf = raftaki adet
  // min = raf uyarı limiti, dMin = depo uyarı limiti
  // ack / ackD = raf / depo uyarısı "Gördüm" denmiş mi
  sales: [{ id, date /* ISO */, total, items: [{ id, name, cat, qty, price, cost, total }] }]
}
```
Eski kayıtlarda `dMin`, `items[].cat`, `items[].cost` olmayabilir; `norm()` ve `itemInfo()` bunu tamamlıyor.

## Sekmeler ve kurallar
- **Satış:** Ürün kartlarına dokunarak sepete ekleme, adet ayarlama, toplam. Satışta önce raftan, raf yetmezse depodan düşer.
- **Ürünler:** "+ Ürün ekle" butonu formu açar (form varsayılan gizli). "Düzenle" formu o ürünün kartının altında açar. "Raf doldur" yazılan adedi depodan rafa taşır.
- **Uyarılar:** Raf ve depo için ayrı bölümler. Stok limitin altına düşünce (<= limit) uyarı çıkar; "Gördüm" denmemiş uyarı varsa alttaki zil simgesi kırmızı yanıp söner ve sayı rozeti gösterir. Stok limitin üstüne çıkınca "gördüm" işareti sıfırlanır.
- **Rapor:** "Satış raporu" — Günlük / Haftalık (pazartesi başlar) / Aylık + tarih seçip "Rapor oluştur". Ciro, tahmini kâr, işlem sayısı, adet, önceki döneme göre % değişim, saatlik/günlük çubuk grafik (dokununca detay), kategori halka grafiği, en çok satan 10 ürün. "Stok durumu" — raf/depo adet ve maliyet/satış değerleri, son satışlar, yedek dışa/içe aktarma (JSON).

## Kullanıcının istediği ama henüz yapılmayanlar
- **OAuth onay ekranı "Testing" modunda.** Şu an sadece ddkandogan@gmail.com giriş yapabilir. Kullanıcı ileride başka bir Google hesabıyla da girmek isterse (ör. çalışanı için), o e-postayı Google Cloud Console > Audience > Test users'a eklemek gerekir. "Production"a geçmek Google doğrulamasından geçmeyi gerektirir (onlarca kullanıcı için ihtiyaç yok).
- **Bilinen sorun:** Claude artifact olarak yayınlanan sürümde Google girişi ve "Dışa aktar" (yedek indirme) çalışmıyor, çünkü orada hem Google script'i hem indirme izni engelli. Asıl kullanım yeri artık GitHub Pages adresi.

## Kontrol
Değişiklikten sonra script'i sözdizimi için kontrol et ve mümkünse bir tarayıcıda (Playwright vb.) satış, raf doldurma, uyarı ve rapor akışlarını dene.
