# 3. Teknoloji katmanları

[← İçindekiler](../README.md)

Bir veri platformu altı katmandan oluşur; batch ve stream aynı katmanları farklı araçlarla doldurur. Apache vakfı bu katmanların çoğunda açık kaynak standardı belirler (Kafka, Spark, Flink, Airflow, Iceberg, Druid).

| Katman | Görevi | Batch araçları | Stream araçları |
| --- | --- | --- | --- |
| 1. Kaynak | Verinin üretildiği yer | Dosya (CSV, Excel), API, veritabanı | WebSocket, FIX, IoT, CDC log |
| 2. Alım (ingestion) / kuyruk | Veriyi alıp tampona koymak, üretici ile tüketiciyi ayırmak | SFTP, nesne deposu (S3, MinIO), Airbyte, Python script | **Apache Kafka**, Redpanda, Apache Pulsar, RabbitMQ, AWS Kinesis, Google Pub/Sub; Kafka Connect, Debezium (CDC) |
| 3. İşleme | Temizleme, dönüştürme, toplama | **Apache Spark**, SQL (dbt), Pandas/Polars, Dataiku | **Apache Flink**, Spark Structured Streaming, Kafka Streams, ksqlDB, Apache Beam |
| 4. Depolama | Sonucu saklamak ve sorgulatmak | DWH: Snowflake, BigQuery, Redshift, Oracle, PostgreSQL; Lakehouse: Apache Iceberg, Delta Lake, Hudi (Parquet üzerinde) | Zaman serisi: TimescaleDB, kdb+, InfluxDB; gerçek zamanlı OLAP: ClickHouse, Apache Druid, Apache Pinot; önbellek: Redis |
| 5. Orkestrasyon | İşlerin sırası, zamanlaması, tekrar denemesi | **Apache Airflow**, Dagster, Prefect, cron, n8n | Sürekli çalışır; dağıtım ve yeniden başlatma Kubernetes ile |
| 6. Sunum ve izleme | Raporlama, alarm, sağlık takibi | Power BI, Tableau, Superset, Streamlit | Grafana, Telegram/Slack alarmları, Kafka UI; Prometheus ile gecikme (consumer lag) takibi |

## Kuyruk neden gerekli?

Üretici (borsa akışı) ile tüketici (hesaplama motoru) farklı hızlarda çalışır. Kuyruk araya girerek veriyi kalıcı biçimde tamponlar: tüketici çökerse veri kaybolmaz, kaldığı offset'ten devam eder; aynı veriyi birden fazla tüketici (alarm servisi, arşivleyici, dashboard) bağımsız okur.

## Kafka ile RabbitMQ farkı

Kafka bir **log**'dur; mesajlar okunduktan sonra da saklanır, geriye sarılıp yeniden okunabilir ve yüksek hacme uygundur. RabbitMQ klasik bir **mesaj kuyruğu**dur; mesaj tüketilince silinir, iş dağıtımı ve yönlendirme için uygundur. Piyasa verisi gibi yeniden işlenebilir olay akışlarında Kafka (veya Kafka uyumlu Redpanda) standart tercihtir.

## DWH ne zaman yeterli?

Klasik DWH'ler toplu yükleme ve analitik sorgu için optimize edilir; saniyede binlerce satır tekil insert'e uygun değildir. Bu yüzden stream sonuçları genelde önce zaman serisi veya OLAP veritabanına yazılır, DWH'ye periyodik olarak aktarılır.

[← Önceki](02-temel-kavramlar.md) · [Sonraki: Mimari desenler →](04-mimari-desenler.md)
