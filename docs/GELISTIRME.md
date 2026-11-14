# efpies — geliştirici notları

Kullanıcıya yönelik tanıtım için ana [README](../README.md) dosyasına bak. Bu dosya kodla çalışanlar içindir.

## Çalıştırma / derleme
```
npm install
npm start          # geliştirme
npm test           # testler (gerçek sistemi sadece okur; yazma testleri geçici kopyalarda)
npm run dist       # dist/efpies-Kurulum-<sürüm>.exe
```
Arayüz ekran görüntüleri (geçici veriyle): `npx electron test/snapshot.js <klasör>`
İkonu yeniden üretmek: `npx electron scripts/render-icon.js` (`assets/icon.svg` → `assets/icon.png`)

Derleme sorunları: `Cannot create symbolic link` → `%LOCALAPPDATA%\electron-builder\Cache\winCodeSign\*.7z` dosyasını aynı yerde `winCodeSign-2.6.0` klasörüne elle aç. `makensis ... could not open file` → yol 260 karakteri aşıyor; `subst Q: .` ile kısa sürücü harfinden derle.

**Önemli:** Kurulu efpies'i Claude masaüstü uygulamasının komut satırından başlatma. O ortamda `AppData` yazmaları başka bir klasöre yönlendirilir ve yönetici olarak çalışan kurulumlar (NVIDIA sürücüsü, Windows Update) dosyaları bulamaz ("Sistem belirtilen yolu bulamıyor").

## Yeni sürüm yayınlama (GitHub: oqqy-dev/efpies)
Kurulu efpies açılıştan 20 sn sonra ve 6 saatte bir `github.com/oqqy-dev/efpies/releases` adresine bakar. Yeni sürümü arka planda indirir; **oyun açıkken kurmaz**. Bir oyun kapandıktan sonra (pencere tepsideyse) ya da efpies kapanırken kurar.

**Anahtarla (önerilen):**
1. `package.json` içinde `version` değerini artır (kurulu sürümden büyük olmalı).
2. GitHub → Settings → Developer settings → Fine-grained tokens → yalnızca `oqqy-dev/efpies` için **Contents: Read and write** izinli anahtar.
3. PowerShell'de:
   ```
   $env:GH_TOKEN = "github_pat_..."
   powershell -ExecutionPolicy Bypass -File scripts\release.ps1
   ```
4. Sürüm açıklamasına yazılanlar uygulamada "Bu sürümde neler var" olarak görünür.

**Elle:** `npm run dist` → GitHub → Releases → *Create a new release* → etiket `v<sürüm>` → `dist` klasöründen `efpies-Kurulum-<sürüm>.exe`, `efpies-Kurulum-<sürüm>.exe.blockmap` ve `latest.yml` dosyalarını yükle → *Publish release*. Üç dosya da gerekli.

## Nasıl çalışır
| Parça | Yöntem |
|---|---|
| NVIDIA ayarları | `nvapi64.dll` (NvAPI DRS) PowerShell içinden C# ile; sadece oyunun kendi profiline yazılır |
| Sürücü güncellemesi | NVIDIA'nın sürücü arama servisi (`AjaxDriverService`), GPU kimliği için `ZenitH-AT/nvidia-data`; kurulum `-s -noreboot -noeula` |
| Windows Update | Windows Update Agent COM (`Microsoft.Update.Session`) |
| FPS | Intel PresentMon (ilk kullanımda onayla indirilir); ham kare verisi saklanmaz, histogramdan ortalama ve %1 düşük hesaplanır |
| Sıcaklık | GPU: `nvidia-smi`; işlemci: Windows termal bölge sayacı (dizüstülerde işlemci sıcaklığına yakın) |
| Yönetici işlemleri | `src/lib/ps.js` → `elevate()`: gerçek hata mesajını ve dönüş kodunu döndürür |
| Kendini güncelleme | `electron-updater` (GitHub sağlayıcısı) |

Önceki adla ("FPS Optimizer") alınmış yedekler ve orijinal değerler ilk açılışta otomatik taşınır.

## Yapı
```
src/main.js                 Ana süreç: IPC, izleme, güncelleme akışları, raporlar
src/lib/system.js           Donanım/Windows taraması
src/lib/games.js            Steam/Epic tarama, sürüm, izleme
src/lib/ops.js, apply.js    Tekil ayar işlemleri, yedek, orijinaller, geri alma
src/lib/nvdrs.js            NVIDIA profil ayarları (NvAPI)
src/lib/nvidia.js           Sürücü kontrolü, notların özeti, indirme, kurulum
src/lib/winupdate.js        Windows Update arama/kurulum/geçmiş
src/lib/monitor.js          Telemetri, oyun süreç izleyici, FPS oturumu
src/lib/perf.js             Oturumlar, olaylar, önce/sonra raporu
src/lib/guardian.js         Oyun güncellemesi sonrası tarama ve düzeltme
src/lib/selfupdate.js       efpies'in kendi güncellemesi
src/lib/profiles/           cs2, unreal, nvidiaRecs, windows, helpers, index
src/renderer/               Arayüz
scripts/release.ps1         Test + derleme + GitHub'a yayın
scripts/render-icon.js      SVG ikondan PNG üretimi
```
