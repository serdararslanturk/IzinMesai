# İzin & Mesai Yönetim Sistemi — Çalışma Özeti (devam için)

> Bu dosyayı yeni bir sohbete yükleyip kaldığın yerden devam edebilirsin.
> Yeni sohbette önce **"github al"** de, sonra bu özeti dikkate alarak çalış.

## Proje
- **Tek dosya:** `C:\ClaudeProjeler\izin-mesai\izin-mesai-yonetim-sistemi_1.html` (HTML/JS SPA, ~500KB, tek dosyada CSS+JS+veri).
- **GitHub:** `serdararslanturk/IzinMesai` — son commit için `git log` (7655b13 sonrası: 15 Eylül mobil görünüm).
- **Kullanıcı:** Serdar (Türkçe). ACR şirketi için **izin + fazla mesai + harcırah** yönetimi.

## İş akışı (önemli kurallar)
- **"github al"** → `powershell.exe -ExecutionPolicy Bypass -File C:/ClaudeProjeler/al.ps1 -proje izin-mesai` (pull).
- **"github ver"** → `C:/ClaudeProjeler/yukle.ps1 -proje izin-mesai -mesaj "..."` (commit + push).
  - **Commit mesajında ÇİFT TIRNAK KULLANMA** — PowerShell argümanını bozuyor (commit oluşmaz, "Everything up-to-date" der). Tırnak gerekiyorsa `cd /c/ClaudeProjeler/izin-mesai && git commit -F` ya da heredoc ile yap.
  - Otomatik push yok; sadece "github ver" deyince.
- **"browserda göster"** → statik sunucu: `cd /c/ClaudeProjeler/izin-mesai && python -m http.server <port>` (arka planda). **Portlar sürekli ölüyor (exit 4)** — her seferinde yeni port dene (8765→8775 kullanıldı, sıradaki ~8776). Mobil test: `resize_window` preset mobile (375px); dar ekran eşiği 900px. Sonra `preview_start` ile `http://localhost:<port>/izin-mesai-yonetim-sistemi_1.html?f=N` (cache-bust `?f=N` artır) aç.
- **Test/doğrulama:** Ekran görüntüsü yerine `javascript_tool` ile DOM/JS kontrolü yap (screenshotlar sık başarısız). Edit-hook sürekli `file://`/`data:` **stale sekme** açıyor — test sonuçları tuhaf gelirse o sekmeyi kapat, **seed** sekmesinde taze reload et.
- **Test verisini geri al**, test için asla `kaydet()` çağırma; aktif kullanıcıyı **AHMET SERDAR ARSLANTÜRK (`p_A262`, üst yönetim)** olarak bırak.
- Sandbox'ta SQL Server çalışmaz; statik `python -m http.server` ile servis edilir. Python 3.12 kurulu.

## Sabit test verisi (`testKayitlariHazirla()` kod içinde)
- **ECE ER (`p_A016`)**, yöneticisi **`p_A262` = AHMET SERDAR**. Her açılışta ECE ER için **2 izin + 2 mesai + 2 harcırah — hepsi "onay bekliyor"** oluşturulur; her reload'da sıfırlanıp yeniden pending gelir. AHMET SERDAR'ın **Bekleyen onaylar** kuyruğuna düşer → onay/ret/revize testleri buradan.
- **İK kullanıcısı:** ÖYKÜ ARSAL ÇELEBİ (`rol==='ik'`) — Ayarlar ve personel kartı düzenleme için gerekir.

## Mimari anahtarlar
- Global `S`; `ciz()`/`cizGovde()` `#page`'i çizer; modallar `modalAc(html, genis)`; `modalKapat()`.
- Router: `S.sayfa` → view fonksiyonu; `SAYFALAR` menü dizisi; `navCiz()`; `git(k)`.
- **Yetki:** `seviye()` (0 personel, 1 yönetici, 2 üst yönetim, 3 İK), `gercekSeviye`, `takvimKisiler()` (veri kapsamı: sv≥2 tümü, sv1 kendi+alt kadro, else kendi). **Rol yetki matrisi** `S.ayar.yetki` (Ayarlar→Genel'de düzenlenir; `sayfaErisir()`/`islemYetki()`; İK kilitli tam yetki; `YETKI_VARSAYILAN`).
- **Persist:** `depoYaz()` → `{ayar, personel, izin, mesai, harcirah, aktifId}` (window.storage; harcirah dahil).
- Onay durumları: `BEKLEYEN`, `GECERLI`, `DURUM_AD`; `onayKuyrugu()` (izinler+mesailer+harcirahlar).

## Harcırah modülü
- Kayıt: `harcirahFormu(pid)` — Personel, **Ülke** (200 ülke, `ULKELER`; yazarak-aranan liste `harcirahUlkeAra`/`trNorm`; tıklayınca üzerine yaz `this.select()`; default Türkiye), **Şehir** (Türkiye→81 il `TR_ILLER`, yurtdışı→serbest metin), **Görev tarihleri** (izin gibi aralık takvimi `htakvim*`), açıklama.
- Kapsam: `harcirahKapsam(ulke)` = Türkiye→`yurtici`, diğer→`yurtdisi`.
- Fonksiyonlar: `harcirahHesap`, `harcirahAralikTopla`, `harcirahGunlukTarife`, `harcirahAyPayi`, `harcirahCakismasi`, `harcirahOzet`, `harcirahSatiri`, `harcirahDetay`, `harcirahKarar`, `harcirahRedAc`, `harcirahRevizeAc/Kaydet`, `harcirahIptal`, `harcirahSil`, `harcirahDonemEkle/Sil/Yaz`.
- Kayıt alanları: `{id,pid,ulke,sehir,kapsam,bas,bit,gun,gunluk,dovis,tutar,durum,gecmis,...}`. `DOVIZ_SIM={TL:'₺',EUR:'€'}`.

## Bu oturumda tamamlananlar (hepsi push'landı, `7655b13`)
1. **Harcırah modülü** (baştan) — yukarıdaki tüm parçalar.
2. **Rapordan onay**: Mesai listesi (`vMesaiListe`) Detay sekmesinde inline Onayla/Reddet; Aylık drilldown → kişi "Kayıtlar" → `msKisiAyMesai` modalı (Onayla/Reddet/Detay).
3. **Reddet → "Red/Revize"**: onay veren görünümünde buton adı; reddet ekranında "Revize et…"; revizede **eski/yeni yan yana** (izin/mesai/harcırah üçünde de).
4. **Personel ret/revize bildirimi**: "Yönetici işlemleri" kartı + **"Gördüm"** (mesai+harcırah; izinde zaten vardı). Bayraklar: `redGoruldu/redTarih`, `revizeliOnay/revizeGoruldu/revizeTarih`.
5. **Mesai/Harcırah kişisel sayfası** (`vMesailerim`): tek **Toplam kutusu** (üstte birleşik toplam, altında `turOzetHtml` → "Mesai: … Harcırah: …"); **aylık kutucuklar** (her ay total + Mesai:/Harcırah:); aya tıkla → **ayrı pencere/modal** (`kisiAyAc`→`kisiAyDetay` modalAc). `kisiAylikVeri` per-ay veri.
6. **Detay ekranlarından Sil kaldırıldı**; satırlarda sadece **Revize · Detay**; **Geri çek** revize modalı içine taşındı ("Talebi geri çek").
7. **Harcırah ay sınırı bölünmesi** ("kısmi" rozeti).
8. **Yıllık ücret** (dinamik): personel kartında **yıl bazlı ücret listesi** (`p.ucretler={yil:tutar}`; `ucretYillarCiz`/`ucretTaslakGir`/`ucretYilEkle/Sil`; migration eski `p.ucret`→`ilkYil()` yılına). Mesai **kendi yılının ücretinden** hesaplanır; yıl ücreti yoksa **"ücret bekleniyor"** (kayıt yapılır), girilince otomatik. Yardımcılar: `ucretHam/ucretTanimli/aylikUcret/gunlukUcret/saatlikUcret(pid,yil)`, `mesaiYili(m)`, `mesaiTutar` **null dönebilir**, `mtutar(m)` (toplamlar) / `mtutarYazi(m)` (gösterim).
9. **Harcırah geçerlilik dönemleri** (güne göre böl): `S.ayar.harcirahDonemleri`; Ayarlar→Mesai'de tablo (başlangıç/bitiş tarih + yurtiçi/yurtdışı tutar+döviz; "+ Dönem ekle"). Görev günleri o güne denk gelen dönemin tarifesinden; dönem sınırını aşan görev **güne göre bölünür**; ay payı da böler. Tarifesiz gün → `eksik` uyarı + kayıt engeli. Migration: eski tek tarife → açık-uçlu tek döneme.

## 15 Eylül 2026 oturumu — mobil görünüm (push'landı)
1. **Üst şerit 4 sekme** (dar ekran ≤900px): İzinlerim · Mesai/Harcırah · Bekleyen onaylar (sayı rozetli) · Raporlar. `BIRINCIL`, `NAV_SINIF`; yetkiye göre 3–4 sekme.
2. **Raporlar sayfası** (`raporlar`, `vRaporlar`, `raporSayfalari()`, `RAPOR_ACIKLAMA`): yalnız dar ekranda sekmesi var (`mobiltab`); erişilebilen raporları kart olarak listeler. Rapor içindeyken Raporlar sekmesi seçili kalır. Raporlar ⚙ panelinde artık listelenmez.
3. İzinlerim'de sarı "Bekleyen onayları aç" kutusu ve "Kimler izinde" kartı mobilde kaldırıldı (onay kutusu masaüstünde duruyor). Onay kutusu harcırah sayısını da yazıyor.
4. **Mesai/Harcırah** ana düğmeleri mobilde yan yana (`.anabtnlar`).
5. **İzin taleplerim düğmesi** (mobil): İzin talebi'nin yanında, rozetli; listeyi pencerede açar (`izinTaleplerimAc`). Ortak parçalar: `izinTaleplerimVeri/Ozet/Govde`; Gördüm sonrası `taleplerPenceresiTazele()`. Mobilde sayfadaki İzin Taleplerim kartı gizli (`yalnizgenis`).
6. Kart görünümünde sticky işlem sütununun sol çizgisi kaldırıldı (`.ptbl tr.kart td.str`).
7. **İzin hakları raporu mobil tablo** (`vRapor`, `.mtbl`): dar ekranda kart yerine kişi başına tek satır; sütunlar Personel (altında kıdem) · Dept. · D.B. · Kul. · Kalan · +Gel. · H.S. · Sonr. (tarih + gün). Başlıkta arama (`#rapAraKutuM`, `rapAra(v, kutu)`) ve departman seçimi (`baslikSecici('rapDeptM',...)`, masaüstüyle ortak `rapF.dept`); açılan liste `position:fixed` + `th.mdept{z-index:5}`. Sütun payları Personel %25 / Dept. %17 (geniş dar-ekranda şişmesin). Satıra dokun → `kisiKayitlari`. Mobilde kişi sayısı + sıralama şeridi gizli (`.hd.rapust`). Masaüstü tablo aynen.
8. **İzin takvimi mobilde ızgara** (`vTakvim`/`takvimTablo`): eskiden dar ekranda ızgara gizliydi (yalnız liste). Artık gösteriliyor; yana kayar, isim sütunu 96px ve sabit. Kaydırma yeri yeniden çizimde korunur, ilk açılış/dönem değişince bugüne gider (`takvimKaydirmaOku/Yaz`, `.calwrap[data-k]`). Okuma şeridi metni dar ekranda "dokunun". Altındaki kayıt listesi (`takvimListesi`) mobilde duruyor.

## Sırada / olası işler
- Mobilde İzinlerim'de 5 özet kartından sonuncusu tek başına kalıyor — tam genişliğe yaymak önerildi (bekliyor).
- İzin taleplerim penceresine onaylı/geçmiş izinleri eklemek (önerildi, bekliyor).
- Harcırahı **Mesai listesi raporuna** eklemek (rapor şu an mesai-only).
- Kadro/ücret sayfasında **yıl seçici** (şu an S.yil'e göre gösteriyor).
- İzin tarafında yıllık-ücret benzeri bir ihtiyaç varsa gözden geçirme.

## Doğrulanmış örnek değerler
- 2026 mesai (ECE ER, 3 saat, fazla ×1,5) = **1.100 ₺**; 2027 mesai (yıl ücreti yok) = **"ücret bekleniyor"**.
- Harcırah 30 Ara 2026 – 2 Oca 2027, iki dönemli tarife (2026:1000, 2027:1200 ₺) → **4.400 ₺** (Aralık 2.000 / Ocak 2.400).
