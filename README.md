# SwipeTune 🔥🎧

**Tinder tarzı şarkı keşif uygulaması.** Kart açılınca şarkı çalar. Sağa kaydırırsan beğenirsin, sola kaydırırsan geçersin. Beğendiklerin Spotify'daki **SwipeTune ❤️** playlistine otomatik eklenir, Shazam'daki gibi.

> *A Tinder-style music discovery web app with a hand-built recommendation engine (Thompson Sampling, hierarchical Bayesian priors, personalized PageRank over an artist graph). Pure client-side JavaScript: no backend, no build step.*

🔗 **Canlı:** https://i-asimov.github.io/swipetune/

---

## Özellikler
- 🎵 **Otomatik önizleme:** Her kartta 30 saniyelik önizleme çalar. Karta dokununca durur ya da devam eder.
- 👀 **Girişsiz mod:** Spotify hesabı olmadan da denenebilir, beğeniler cihazda saklanır.
- ❤️ **Shazam gibi playlist:** İlk beğenide `SwipeTune ❤️` playlisti oluşturulur, sonraki beğeniler oraya eklenir. Geri alırsan şarkı playlistten de çıkar.
- 🧠 **Kendi yazdığım öneri motoru:** "Algoritma beynin" paneli, sistemin seni nasıl modellediğini canlı gösterir.
- 🖼️ Gerçek albüm kapakları, uyum yüzdesi ve her öneri için "neden bu şarkı" açıklaması.
- 🔐 Spotify girişi **PKCE** ile yapılır. Backend ve client secret yok, şifre uygulamaya hiç gelmez.

## Öneri motoru
Spotify, yeni uygulamalar için hazır öneri servisini (`/recommendations`, audio features, related artists) 2024'te kapattı. Bu yüzden öneri sistemi sıfırdan yazıldı.

| Katman | Ne yapıyor |
|---|---|
| **Thompson Sampling** | Tür, sanatçı, saat dilimi×tür ve öneri kaynağı için ayrı Beta(α, β) kolları tutulur. Her kararda bu dağılımlardan örnek çekilerek keşif ile bilineni getirme dengelenir. |
| **Hiyerarşik Bayes** | Az verisi olan bir sanatçının önseli, türüne olan zevkten türetilir (shrinkage). Sevilmeyen bir türden tek bir beğeni, sanatçıyı favori yapmaz. |
| **Örtük sinyaller** | Karar süresi ve önizlemeyi dinleme ödüle katılır. Anında geçilen şarkı güçlü negatif, uzun düşünülüp geçilen zayıf negatif sayılır, dinleyip geçmek yarım puan alır. |
| **Zamanla unutma** | Her kaydırmada eski sinyaller γ = 0.992 ile sönümlenir (yaklaşık 85 kaydırmada yarı ömür). |
| **Sanatçı grafiği + PPR** | Aramalarda birlikte çıkan, featuring yapan ve birlikte beğenilen sanatçılardan bir ağ kurulur. Sevilen sanatçılardan başlayan *Personalized PageRank* ile komşu sanatçılar keşfedilir. Başlangıç kütlesi türler arasında dengelenir (çoklu ilgi), böylece tek bir tür ağı domine edemez. |
| **Kaynak meta-bandit** | Beş öneri yöntemi var: favori sanatçılar, ağ komşuları, tür tahmini, Spotify'da en çok dinlenenler ve keşif. Sistem kullanıcıda hangisinin işe yaradığını da öğrenir. |
| **Adaptif keşif** | Zevk henüz oturmadıysa ve beğeni oranı düşükse keşif artar. Zevk oturduysa bilinen sanatçılara dönülür. Bu, keşfin kendini besleyen bir döngüye girmesini önler. |
| **Sıralama** | Tahmin + çeşitlilik cezası (aynı sanatçı ya da tür üst üste gelmesin) + yenilik bonusu. Havuz her kaydırmada yeniden sıralanır ve budanır. |
| **Soğuk başlangıç** | Onboarding'de seçilen türler ve Spotify'da en çok dinlenen sanatçılar (`/me/top/artists`) başlangıç önseli olarak kullanılır. |

### Simülasyon sonucu
Sentetik bir kullanıcıyla (aslında Türkçe Rock ve Indie seviyor, onboarding'de yanlış tür seçiyor) 4 tekrar yapıldı:

| | Kaydırma 1–30 | Son 60 kaydırma |
|---|---|---|
| Beğeni oranı | ~%25 | **%58–78** |
| Rastgele öneri | ~%18 | ~%18 |

## Mimari
- **Keşif, kapaklar, önizlemeler → Deezer açık API'si** (JSONP, anahtar gerektirmez): sanatçı top şarkıları ve *benzer sanatçılar*. Benzer sanatçı verisi algoritmadaki sanatçı ağını besler.
- **Spotify → sadece giriş ve yazma.** Kullanıcının en çok dinlediği sanatçılar günde bir kez alınır. Bir şarkı beğenilince Spotify'da karşılığı bulunur ve playliste eklenir.
- **Neden?** Spotify geliştirme modunun istek kotası düşük. Keşfi Deezer'a taşımak Spotify isteklerini yaklaşık %95 azalttı. Spotify `429` dönerse üstel bekleme uygulanır ve beğeniler kuyruğa alınıp kota açılınca gönderilir.
- Bütün API cevapları `localStorage`'da 24 saat önbelleğe alınır.

## Teknik
- Tek dosya: `index.html` (HTML + CSS + vanilla JS). Framework ve build adımı yok.
- Spotify Web API: PKCE OAuth, `/me/top/artists`, `/search` (sadece beğenide eşleştirme), `/me/playlists`, `/playlists/{id}/items`, `/me/library`
- Deezer API (JSONP): `/search/artist`, `/artist/{id}/top`, `/artist/{id}/related`
- Durum `localStorage`'da tutulur: model, görülenler, beğeniler.

## Kurulum (kendi kopyan için)
1. [Spotify Developer Dashboard](https://developer.spotify.com/dashboard) → **Create app** (hesabın Premium olmalı)
2. Redirect URI: sitenin tam adresi, örneğin `https://<kullanıcı>.github.io/swipetune/`. **Web API**'yi işaretle.
3. `index.html` içindeki `CLIENT_ID` değerini kendi Client ID'nle değiştir.
4. GitHub Pages'i aç: **Settings → Pages → Deploy from branch → main / root**.
5. Spotify geliştirme modunda en fazla **5 test kullanıcısı** olabilir. Diğer kullanıcıları **User Management** sayfasından e-postalarıyla ekle.

## Kısayollar
`→` beğen · `←` geç · `↑` süper beğeni · `Z` geri al · `Space` çal/durdur
