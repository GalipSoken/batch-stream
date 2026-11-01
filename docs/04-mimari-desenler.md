# 4. Mimari desenler

[← İçindekiler](../README.md)

Batch ve stream'i bir arada kullanmanın yerleşik dört yolu vardır; bugün en yaygını Kappa ile lakehouse'un birleşimidir.

| Desen | Fikir | Artı | Eksi |
| --- | --- | --- | --- |
| **Lambda** | Aynı veri iki yoldan akar: hızlı ama yaklaşık stream katmanı + yavaş ama kesin batch katmanı; sonuçlar sunumda birleşir | Hem hız hem kesinlik | Aynı mantık iki kez yazılır ve bakılır |
| **Kappa** | Her şey stream; yeniden hesaplama gerekirse Kafka log'u baştan oynatılır | Tek kod tabanı | Uzun geçmişi log'da tutmak ve yeniden oynatmak maliyetli |
| **Micro-batch** | Akış küçük parçalarda batch gibi işlenir (Spark Structured Streaming) | Batch koduna yakın, kolay | Gecikme saniyeler düzeyinde, milisaniye değil |
| **Lakehouse** | Ham ve işlenmiş veri Iceberg/Delta tablolarında; aynı tabloya hem stream yazılır hem batch sorgulanır | Tek depolama, ucuz nesne deposu, zaman yolculuğu | Kurulum ve tablo bakımı (compaction) bilgi ister |

## Medallion düzeni

Katmanlı veri düzeni olarak **medallion** yaklaşımı sık kullanılır:

- **Bronze:** Ham olaylar, değiştirilmeden.
- **Silver:** Temizlenmiş, tekilleştirilmiş.
- **Gold:** İş metrikleri, raporlamaya hazır.

[← Önceki](03-teknoloji-katmanlari.md) · [Sonraki: Hisse senedi batch hattı →](05-hisse-batch.md)
