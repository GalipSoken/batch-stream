# 8. Karar rehberi: hangisi ne zaman

[← İçindekiler](../README.md)

Kural basittir: sonuç birkaç dakika gecikse kimse zarar görmüyorsa batch (veya micro-batch) seçin; gecikme doğrudan para veya risk kaybı demekse stream seçin.

- **Batch seçin:** Raporlama, regülatör dosyaları, model eğitimi, gün sonu metrikleri, geçmiş verinin yeniden hesaplanması, veri kaynağı zaten günlük/aylık yayımlanıyorsa (BDDK, TÜİK gibi).
- **Micro-batch seçin:** Dakikalık güncel dashboard'lar, REST API'den periyodik çekilen veri, tek kullanıcılı alarm sistemleri.
- **Stream seçin:** Saniye altı alarm, dolandırıcılık tespiti, anlık limit/risk kontrolü, çok sayıda tüketicinin aynı olayları bağımsız kullandığı durumlar.
- **İkisini birlikte kullanın:** Stream anlık karar için, batch aynı verinin gece kesinleşmiş ve düzeltilmiş halini DWH'ye yazmak için. Hisse senedi örneğinde mumlar gün içinde stream'den, düzeltilmiş kapanış ise gece batch'ten gelir.

## Stream'e geçmeden önce üç soru

1. Veri kaynağı gerçekten anlık mı?
2. Sonucu saniyede tüketecek biri var mı?
3. Sistemi 7/24 izleyecek bir ekip var mı?

[← Önceki](07-altyapi-ihtiyaclari.md) · [Sonraki: Sözlük →](09-sozluk.md)
