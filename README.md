# Günlük AI Brifing

Her sabah RSS kaynaklarını tarar, Gemini (ücretsiz katman) ile Türkçe özetler ve Telegram'dan gönderir.
GitHub Actions üzerinde çalıştığı için bilgisayarının açık olması gerekmez. Hepsi ücretsizdir.

```
RSS kaynakları -> tekrar eleme (sent.json) -> Gemini özet + önem puanı -> Telegram
```

Gemini çalışmazsa sistem çökmez; başlık ve linkleri özetsiz gönderir.

## Kurulum

### 1. Gemini API anahtarı
1. https://aistudio.google.com/app/apikey adresine Google hesabınla gir.
2. "Create API key" ile anahtar oluştur ve bir yere not et. Faturalandırma açman gerekmez.

### 2. Telegram botu
1. Telegram'da **@BotFather**'ı aç, `/newbot` yaz, adımları izle. Sana bir **bot token** verir.
2. Oluşturduğun bota gir ve `/start` yaz (bunu yapmazsan bot sana mesaj atamaz).
3. Chat ID'ni öğrenmek için tarayıcıda şu adresi aç (TOKEN yerine kendi token'ını yaz):
   `https://api.telegram.org/botTOKEN/getUpdates`
   Çıktıda `"chat":{"id": 123456789` şeklindeki sayı senin **chat ID**'ndir.

### 3. Bilgisayarında dene (VS Code terminali)
```powershell
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt

$env:GEMINI_API_KEY="..."
$env:TELEGRAM_BOT_TOKEN="..."
$env:TELEGRAM_CHAT_ID="..."

python briefing.py --ping       # Telegram bağlantısı çalışıyor mu?
python briefing.py --dry-run    # mesajı ekranda göster, gönderme
python briefing.py              # gerçek gönderim
```
Anahtarları bir dosyaya veya koda yazma; sadece terminal oturumunda ortam değişkeni olarak ver.

### 4. GitHub'a yükle
1. GitHub'da **public** bir repo aç (örn. `daily-ai-briefing`). Public repoda Actions dakikaları ücretsizdir.
2. Bu klasörü o repoya yükle.
3. Repo > Settings > Secrets and variables > Actions > **New repository secret** ile üç secret ekle:
   `GEMINI_API_KEY`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`
4. Repo > Settings > Actions > General > Workflow permissions: **Read and write permissions** seç (sent.json'u kaydetmek için).
5. Actions sekmesi > "Günlük Brifing" > **Run workflow** ile elle bir kez çalıştırıp test et.

Public repoda kodun ve `sent.json` içindeki link listesi herkese açık olur; anahtarların ise Secrets'ta gizli kalır.

## Ayarlar

- **Saat:** `.github/workflows/daily.yml` içindeki `cron: "7 5 * * *"` satırı. Değerler UTC'dir, Türkiye saati = UTC + 3. Örneğin 07:30 için `30 4 * * *`.
  GitHub zamanlanmış görevleri birkaç dakika (bazen daha fazla) geciktirebilir.
- **Konular ve kaynaklar:** `config.yaml`. Kaynak eklemek için `feeds` listesine RSS adresi yaz.
  `keywords` verirsen sadece o kelimeleri içeren kayıtlar alınır.
- **Brifing uzunluğu:** `show_per_topic` ve `min_importance`.
- **Gemini modeli:** `config.yaml` içindeki `gemini.models`. Ücretsiz modeller zamanla değişebilir; hata alırsan
  AI Studio'dan güncel model adlarını kontrol et.

## Sorun giderme

- **Mesaj gelmiyor:** Actions sekmesinde son çalışmanın logunu aç. `Telegram hatası 400/403` ise bota `/start` yazdığından ve chat ID'nin doğru olduğundan emin ol.
- **"Okunamayan kaynaklar" uyarısı:** O sitenin RSS adresi değişmiş olabilir; `config.yaml`'dan düzelt veya çıkar.
- **Özet yok, sadece başlıklar:** Gemini kotası dolmuş veya model adı eskimiş olabilir; log'daki `[uyarı] Gemini` satırlarına bak.
- **60 gün işlem olmazsa:** GitHub, hareketsiz repolarda zamanlanmış çalışmaları durdurabilir. `sent.json` her gün güncellendiği için
  normalde sorun olmaz; yine de Actions sekmesinden zaman zaman kontrol et.

## Gizlilik notu

Ücretsiz Gemini katmanında gönderilen içerikler Google tarafından ürün geliştirmede kullanılabilir. Bu projede sadece
herkese açık haber başlıkları ve özetleri gönderildiği için sorun olmaz; kişisel bilgi eklemeyin.
