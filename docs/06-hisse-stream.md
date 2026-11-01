# 6. Hisse senedi örneği — stream hattı

[← İçindekiler](../README.md)

Amaç: seans içinde her işlemi (tick) anında almak, 1 dakikalık mum ve VWAP üretmek, belirlenen fiyat seviyesi kırılınca saniyeler içinde alarm göndermek. Gecikme ihtiyacı saniyelerdir; stream gerekir.

## Akış

1. **Kaynak:** Veri sağlayıcının WebSocket (veya kurumsal tarafta FIX) akışı; her mesaj bir tick: `{sembol, fiyat, adet, event_time}`.
2. **Üretici (producer):** Küçük bir Python servisi WebSocket'i dinler, mesajı şemaya göre (Avro/JSON) paketler ve Kafka'ya yazar.
3. **Kuyruk:** `ticks` topic'i, **anahtar = sembol**, örneğin 12 partition. Aynı sembolün tick'leri aynı partition'a düşer, kendi içinde sıralı kalır. Saklama süresi 7 gün; gerekirse geriye sarılıp yeniden işlenir.
4. **İşleme (Flink veya Spark Structured Streaming):**
    - Event time'a göre 1 dk tumbling window → OHLCV mumu + VWAP.
    - Watermark 5 sn: mum kapandıktan sonra 5 sn içinde gelen geç tick hâlâ sayılır.
    - Kayan 20 dakikalık hacim ortalaması; anlık hacim 3 katını aşarsa "hacim patlaması" olayı.
    - Kullanıcının fiyat seviyeleri (Redis veya küçük bir tablodan) ile karşılaştırma → `alerts` topic'i.
    - Checkpoint her 30 sn; çökme sonrası state kaldığı yerden devam eder.
5. **Yazma (sink):** Mumlar `bars_1m` topic'ine ve TimescaleDB/ClickHouse'a `(sembol, dakika)` anahtarıyla idempotent yazılır.
6. **Tüketiciler:** Alarm servisi `alerts`'i okuyup Telegram'a gönderir; Grafana mumları canlı çizer; arşivleyici ham tick'leri saatlik Parquet dosyalarına (bronze) döker.

## Topic tasarımı

| Topic | Anahtar | İçerik | Tüketen |
| --- | --- | --- | --- |
| `ticks` | sembol | Ham işlemler | Flink, arşivleyici |
| `bars_1m` | sembol | 1 dk OHLCV + VWAP | TSDB sink, dashboard |
| `alerts` | kullanıcı | Seviye kırılımı, hacim patlaması | Telegram botu |

## Altyapı ihtiyacı

Kafka kümesi (en az 3 broker; tek makinede deneme için tek Redpanda düğümü yeterli), schema registry, bir stream işlemci (Flink JobManager + TaskManager), zaman serisi/OLAP veritabanı, Redis, izleme (Prometheus + Grafana, özellikle consumer lag), hepsini ayakta tutacak Docker Compose veya Kubernetes.

## Kod iskeleti

```python
# Spark Structured Streaming ile 1 dk mum (sadeleştirilmiş)
ticks = (spark.readStream.format("kafka")
         .option("subscribe", "ticks").load()
         .select(from_json(col("value").cast("string"), schema).alias("t"))
         .select("t.*"))

bars = (ticks
        .withWatermark("event_time", "5 seconds")
        .groupBy(window("event_time", "1 minute"), "symbol")
        .agg(first("price").alias("open"), max("price").alias("high"),
             min("price").alias("low"),   last("price").alias("close"),
             sum("qty").alias("volume"),
             (sum(col("price") * col("qty")) / sum("qty")).alias("vwap")))

bars.writeStream.outputMode("update") \
    .option("checkpointLocation", "/chk/bars_1m") \
    .foreachBatch(upsert_timescale).start()
```

## Ara çözüm: yarı-stream

Veri sağlayıcı yalnızca dakikalık veya 10 dakikalık REST verisi veriyorsa gerçek stream kurmaya gerek yoktur. Her N dakikada bir çalışan bir zamanlayıcı (micro-batch) aynı mum ve alarm mantığını Kafka olmadan çalıştırır; tek kullanıcılı bir alarm botu için en düşük maliyetli yol budur.

[← Önceki](05-hisse-batch.md) · [Sonraki: Altyapı ihtiyaçları →](07-altyapi-ihtiyaclari.md)
