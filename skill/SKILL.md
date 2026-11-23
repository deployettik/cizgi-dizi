---
name: cizgi-dizi
description: Instagram/TikTok için yapay zekâ seslendirmeli 2D çizgi dram dizisi (9:16 Reels, varsayılan ~3 dk) üretir. Yassı karakterler, kalın kontur, ağlayan/şaşıran yüzler, serif altyazı, "Bölüm N · …" başlığı, sonda "Devamı için takipte kalın". Türk dizisi kalıplarıyla (gizli kimlik, kayınvalide, paralel evren, değiştirilen bebek…) senaryo yazar, ElevenLabs ile her karakteri farklı sesle seslendirir, Remotion ile render eder. Proje C:\Users\hazal\cizgi-dizi. Kullanıcı "çizgi dizi", "animasyon dizi", "bölüm yaz", "dizi senaryosu", "oadambendegilim tarzı", "karakterli hikâye reels" dediğinde kullan.
---

# Çizgi Dizi

Proje: `C:\Users\hazal\cizgi-dizi`. Stack: Remotion 4 + React SVG karakterler + ElevenLabs TTS (v3, duygu etiketleri) + ffmpeg.

## Akış

1. **Fikir ve senaryo.** Önce `skill/senaryo-rehberi.md` dosyasını oku. Kalıp kataloğundan birini seç (ya da kullanıcının fikrini al) ve bölümü **varsayılan olarak maksimum sürede kur: 3 dakikanın hemen altı** (~175 sn, 40–45 replik, 3–4 sahne, 4 perde). Her 20–30 sn'de bir mini kanca olsun. Hazal kısa isterse 30–60 sn. Yeni bir dizide önce `diziler/<dizi>/sezon.md` sezon planını yaz. Sezon **en az 10 bölüm** olmalı. Örnek: `diziler/ornek-konak/sezon.md`. Senaryoyu kullanıcıya kısaca göster. Onay gelmeden seslendirme yapma, çünkü kredi harcar.
2. **Dizi dosyası.** `diziler/<dizi>/dizi.json` içinde hesap adı, karakterler (görünüm + ses) ve ses id'leri yer alır.
3. **Bölüm dosyası.** `diziler/<dizi>/bolum-01.json`. Sahneler → oyuncular → çekimler.
4. **Kontrol (kredisiz).** `npm run kontrol -- <dizi>` komutu dosyayı doğrular, süreyi ve harf/kredi tahminini verir.
5. **Seslendir.** `npm run seslendir -- <dizi> bolum-01` her repliği ElevenLabs ile seslendirir; ağız hareketini ve zamanlamayı çıkarır, `caption.txt` yazar. Önbellek `.cache/tts` içinde tutulur: aynı replik ikinci kez kredi harcamaz.
6. **Render.** `npm run render -- <dizi> bolum-01` şu üç dosyayı üretir: `out/<dizi>-bolum-01.mp4`, `-caption.txt` ve **`-kapak.png`** (Instagram kapak görseli). Kapakta dizi adı büyük harfle, kırmızı "BÖLÜM N" etiketi ve ağlayan ana karakter yer alır; arkada iki karakter durur. Kapağı ayrıca almak için: `npx remotion still Kapak out/k.png --props=out/props-<dizi>-<bolum>.json`. Öndeki ve arkadaki karakterler ile duygular `KapakProps` (`ana`, `yanlar`, `anaDuygu`, `yanDuygular`) üzerinden değiştirilebilir. Yazılar 3:4 profil ızgarası kırpımının (y 240–1680) içinde kalır.
7. **Doğrula.** Kare kolajı çıkar (`ffmpeg -vf "fps=1/2,scale=240:-1,tile=8x2"`) ve bak: yakın planda hedefin önünde biri var mı, karakter kadraj dışında mı? İstersen sesi Scribe ile yazıya döküp repliklerle karşılaştır.

Karakter tasarlarken `npm run kontrol -- <dizi>` çalıştır, sonra `npm run dev` ile Studio'da **Vitrin** kompozisyonunu aç. Tüm karakterler bütün duygu ve pozlarda sırayla görünür. Hızlı kare için: `npx remotion still Vitrin out/v.png --frame=110 --scale=0.5`.

## dizi.json

```json
{
  "ad": "Dizi adı", "hesap": "@hesap",
  "karakterler": {
    "zeynep": {
      "ad": "Zeynep",
      "gorunum": { "yas": "genc", "cinsiyet": "kadin", "ten": "#efcfb4", "sac": "at-kuyrugu", "sacRengi": "#5a3a26",
                   "gozRengi": "#6b4a2e", "ust": "#9fb4cf", "alt": "#4f5a6e", "yaka": true },
      "ses": { "id": "<elevenlabs voice id>", "perde": 0, "hiz": 1, "stabilite": 0.5 }
    }
  }
}
```

- `yas`: `cocuk` | `genc` | `yetiskin` | `yasli`. Çocuk küçük gövdeli ve iri kafalı çizilir; yaşlıda kırışıklık olur.
- `sac`: `kisa` | `dagnik` | `uzun` | `orgu` | `topuz` | `at-kuyrugu` | `kel` | `basortusu`. Başörtüsünde rengi `sacRengi` belirler.
- Giysi: `ust`, `alt`, `etek`, `elbise`, `desen` (`puantiye` | `cizgili` | `ekose`), `yaka`, `kisaKol`, `ayakkabi`.
- Yüz ve aksesuar: `sakal` (`biyik` | `kirli` | `sakal`), `gozluk`, `kolye` (ay kolye rengi, ör. `#e8c35a`).
- Bu hesapta Türkçe sesler var:
  - **Yunus** `fXhoW006nc5Wf8xkGVSy`: erkek
  - **Yasemin** `ywzrmJ3AgYiLqAeZAGrq`: kadın, derin; yaşlı kadın için `perde: -1`
  - **Meloxia** `axtmxCPnqPghs9C5SjJ8`: kadın
  - Çocuk sesi için **Jessica** `cgSgspJ2msm6clMCkdW9` ile `perde: +4` kullan.
  - Hazal'ın klonu `HMq7M1L1Ct9EHmk6DF1O`: yalnızca Hazal isterse.
  - Daha fazlası: `GET /v1/voices`.
  - Aynı sesi iki karaktere verirsen `perde` ±2 ile ayır.

## bolum-XX.json

```json
{
  "no": 1,
  "baslik": "Bölüm 1 · <birinci tekil itiraf>…",
  "kanca": "<ilk 3 sn ekranda büyük yazı, isteğe bağlı>",
  "aciklama": "<caption gövdesi>", "soru": "<yorum sorusu>", "etiketler": ["animasyondizi", "dram"],
  "sahneler": [{
    "mekan": "salon", "zaman": "gece", "zengin": false,
    "oyuncular": {
      "kerem": { "x": 540, "otur": true, "poz": "basini-tutuyor", "duygu": "sersem" },
      "asli":  { "x": 300, "derinlik": 0.3, "gizli": true }
    },
    "cekimler": [
      { "kim": "kerem", "soz": "Dün geceki kutlamada çok içmiştim...", "kamera": { "tip": "orta", "hedef": "kerem" }, "efekt": "karar" },
      { "kim": "asli", "soz": "…", "duygu": "aglama", "poz": "koruyor", "oyuncular": { "asli": { "gizli": false } } },
      { "sure": 1.0, "oyuncular": { "kerem": { "duygu": "saskin" } }, "kamera": { "tip": "cok-yakin", "hedef": "kerem" }, "efekt": "sarsinti" }
    ]
  }]
}
```

- **mekan**: `salon` | `yatak-odasi` | `mutfak` | `sokak` | `hastane` | `ofis`. `zaman`: `gunduz` | `gece`. `zengin: true` konak varyantıdır (damask duvar, avize, altın çerçeve).
- **Dünya** 1080×1920 boyutunda. `x` değeri 0–1080 arası ayak ortasıdır. Pratik aralık 150–930; karakterler arasında en az 200 bırak. `derinlik` 0 ön, 1 arka (küçük). Salonda kanepe x 150–930 arasındadır; `otur: true` olan karakter kanepeye oturur.
- **duygu**: `normal` | `mutlu` | `uzgun` | `aglama` (gözyaşı) | `korku` (ter, titreme) | `kizgin` | `saskin` | `sersem` | `supheli` | `utangac`. Duygu aynı zamanda ses tonunu seçer (v3 etiketi: `[crying]`, `[angry]`…). Elle vermek için `"ton": "[whispers]"`.
- **poz**: `ayakta` | `oturuyor` | `basini-tutuyor` | `kollari-bagli` | `isaret` | `koruyor` (kolunu baktığı yöne açar) | `eller-agizda` | `el-salliyor`.
- **yon**: `sag` | `sol`. Belirtilmezse karakter merkeze bakar. `isaret` ve `koruyor` bu yöne uzanır.
- **Çekim içindeki `oyuncular`** sahne durumunu o çekimden itibaren değiştirir: giriş için `gizli: false`, çıkış, duygu değişimi. `duygu` ve `poz` kısayolları konuşan kişiye uygulanır.
- **kamera**:
  - `tip`: `genis` | `iki` (`hedef` + `hedef2`) | `orta` | `yakin` | `cok-yakin`
  - `hareket`: `it` (varsayılan, yavaş zoom) | `cek` | `sabit` | `titrek`
  - Kamera verilmezse konuşana `orta` çekim yapılır.
  - **Yakın planda hedefin önünde (daha küçük derinlikte, yakın x'te) başka karakter durmasın.**
- **efekt**: `karar` (siyahtan açılış; ilk çekimde kullan) | `flas` (ani giriş veya ifşa) | `sarsinti` (şok).
- **sure**: sessiz çekimin uzunluğu; replikli çekimde replikten sonraki ek bekleme. Konuşmasız `soz` iç ses ya da yazı kartı olarak altyazıda görünür.

## Hesap

Instagram hesabı **@cizgidizikanali**; bu ad her `dizi.json` içinde `hesap` alanına yazılır. Profil fotoğrafı, ekranında ağlayan kahramanın olduğu eski tüplü bir televizyon; arka plan kırmızı ışınlı. Üretmek için: `npx remotion still Profil out/profil.png --props=out/props-<dizi>-<bolum>.json`. Ekrandaki karakter `yuz` ile, ifadesi `duygu` ile değişir.

## Notlar

- Altyazı serif (Lora 700), beyaz ve kalın koyu konturludur; referanstaki gibi ekranın %67'sinde durur. Filigran sol üstte `@hesap`. Sonda 2.2 sn'lik "Devamı için takipte kalın" kartı ve sallanan zil var.
- Müzik: bölüm JSON'una `"muzik": "muzik/dosya.mp3"` ekle (public/ altında, telifsiz). %12 seste döner.
- Süre: seslendirmeden sonra çıktıdaki "N çekim, X sn" satırına bak. X + 2,2 ≤ 178 olmalı (3 dk sınırı). Fazlaysa replik sil ve yeniden seslendir; önbellekteki replikler tekrar kredi yemez. Replikten sonraki boşluk varsayılan olarak 0,22 sn.
- Kredi: starter planda ayda 40k karakter. 3 dakikalık bir bölüm ~2000 harf, kısa bölüm ~300–500 harf tutar. `kontrol` komutu tahmini gösterir.
- v3 hata verirse `seslendir` otomatik olarak `eleven_multilingual_v2`'ye düşer (bu modelde etiket yok). Elle seçmek için `--model eleven_multilingual_v2`. Önbelleği yok saymak için `--yeniden`.
- Yeni mekân, poz ya da saç eklemek için: `src/mekan/Mekan.tsx`, `src/karakter/Karakter.tsx` (`POZ` el hedefleri, `IFADE` duyguları, `SacArka`/`SacOn`) ve `scripts/dogrula.mjs` listeleri. Hepsi birlikte güncellenmeli.
- Repo'ya ses ve video girmez (`public/diziler/*`, `out/` gitignore'da). `.env` asla commit edilmez.
- Gerçek dizilerin karakterlerini, adlarını ve repliklerini kopyalama; kalıbı al, hikâyeyi özgün yaz.
