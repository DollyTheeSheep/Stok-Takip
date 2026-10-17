# Stok Takip — Proje Notları

Kore ürünleri satan bir dükkan için iPad'de kullanılacak stok, satış ve rapor uygulaması.
Kullanıcıyla Türkçe konuş. Kullanıcı yazılımcı değil; açıklamaları sade tut.

## Yapı
- Tek dosya: `index.html` (HTML + CSS + JS bir arada, harici kütüphane yok). Bu yapıyı koru.
- Veri tarayıcıda `localStorage` içinde, anahtar: `stokTakipV1`.
- Tasarım iOS tarzında: alt sekme çubuğu, büyük başlık, gruplu listeler, otomatik karanlık mod.
- Yayınlandığı yer (Claude artifact): https://claude.ai/artifact/PJxjJ63DA4HC17v2QzVJ1X

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
1. **Cihazlar arası ortak veri.** Kullanıcı her cihazda aynı veriyi görmek istiyor. Konuşulan plan: girişte e-posta alanı + Google ile giriş (OAuth istemci kimliği, Google Cloud'da kullanıcı oluşturacak), veriler Drive'da tek bir JSON dosyasında (`drive.file` izni). Kullanıcı Google Cloud kurulumunu sonraya bıraktı; adım adım rehber gerekecek. Alternatif: farklı bir ücretsiz barındırma + basit bir veritabanı (ör. Firebase/Supabase).
2. **Yayınlama.** Drive girişi için uygulamanın sabit bir adreste (ör. GitHub Pages) yayınlanması gerekecek; o adres OAuth'ta yetkili kaynak olarak eklenmeli.
3. **Bilinen sorun:** Claude artifact olarak yayınlanan sürümde "Dışa aktar" (yedek indirme) çalışmıyor, çünkü orada indirme izni yok. Normal tarayıcıda / GitHub Pages'te çalışır.

## Kontrol
Değişiklikten sonra script'i sözdizimi için kontrol et ve mümkünse bir tarayıcıda (Playwright vb.) satış, raf doldurma, uyarı ve rapor akışlarını dene.
