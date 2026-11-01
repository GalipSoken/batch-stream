# 1. Batch ve stream nedir

[← İçindekiler](../README.md)

## Batch veri

Batch işleme, **sınırlı (bounded)** bir veri kümesini bir seferde, planlı bir zamanda işlemektir. Veri önce bir yerde birikir (dosya, tablo, nesne deposu), sonra bir iş (job) onu baştan sona okur, dönüştürür ve hedefe yazar. İşin başı ve sonu bellidir.

**Temel özellikler**

- **Gecikme:** Dakikalar ile saatler arası; sonuç ancak iş bitince görünür.
- **Tetikleme:** Zamanlayıcı (her gece 02:00, her saat başı) veya bir olay (dosya geldi).
- **Tekrar çalıştırılabilirlik:** Hata olursa iş aynı girdiyle yeniden koşulur; doğru kurulursa sonuç aynı çıkar (idempotent).
- **Verim (throughput):** Büyük hacmi verimli işler; kaynak sadece iş sürerken kullanılır.
- **Doğruluk:** Tüm veri elde olduğu için geç gelen kayıt, sıralama gibi sorunlar yoktur.

**Tipik kullanımlar:** Gün sonu raporları, regülatöre giden dosyalar, aylık bilanço hesapları, model eğitimi, DWH yüklemeleri (gece ETL), geçmişe dönük yeniden hesaplama (backfill).

## Stream veri

Stream işleme, **sınırsız (unbounded)** ve sürekli akan bir olay dizisini, olaylar geldikçe işlemektir. Ortada "tüm veri" diye bir şey yoktur; sistem 7/24 açık kalır ve her olayı milisaniyeler ile saniyeler içinde işler.

**Temel özellikler**

- **Gecikme:** Milisaniye ile birkaç saniye arası.
- **Olay (event):** Her kayıt zaman damgalı tek bir olaydır: bir işlem (tick), bir kart harcaması, bir log satırı.
- **Durum (state):** Hareketli ortalama, son 1 dakikanın hacmi gibi hesaplar için sistem ara sonucu bellekte/diskte tutar.
- **Pencere (window):** Sonsuz akışta toplama yapmak için akış zaman dilimlerine bölünür (1 dk, 5 dk).
- **Zorluklar:** Geç veya sırasız gelen olaylar, tekrar eden mesajlar, sistem çökünce durumun korunması.

**Tipik kullanımlar:** Fiyat alarmları, dolandırıcılık tespiti, anlık risk limiti takibi, canlı dashboard'lar, IoT sensörleri, CDC ile veritabanı değişikliklerinin anında taşınması.

**Micro-batch** ikisinin ortasıdır: akış çok küçük parçalara (ör. her 10 sn) bölünüp her parça batch gibi işlenir. Spark Structured Streaming varsayılan olarak böyle çalışır.

## Karşılaştırma

Stream daha hızlıdır ama işletmesi daha pahalı ve karmaşıktır; ihtiyaç saniye düzeyinde değilse batch çoğu zaman doğru tercihtir.

| Boyut | Batch | Stream |
| --- | --- | --- |
| Veri | Sınırlı, durağan küme | Sınırsız, sürekli akış |
| Gecikme | Dakika – saat | Milisaniye – saniye |
| Tetikleme | Zamanlayıcı / dosya gelişi | Her olay |
| Çalışma süresi | İş bitince kapanır | 7/24 açık |
| Durum yönetimi | Gerekmez, veri tamamdır | Pencere, state store, checkpoint |
| Hata kurtarma | İşi yeniden çalıştır | Offset + checkpoint'ten devam |
| Geç gelen veri | Sorun değil | Watermark ile yönetilir |
| Maliyet | Düşük, kaynak kısa süreli | Yüksek, kaynak sürekli |
| Operasyon | Basit | İzleme, ölçekleme, nöbet gerekir |
| Tipik araç | Airflow + Spark/SQL + DWH | Kafka + Flink/Spark Streaming + TSDB/OLAP |

[Sonraki: Temel kavramlar →](02-temel-kavramlar.md)
