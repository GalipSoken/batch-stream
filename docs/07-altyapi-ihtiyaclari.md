# 7. Altyapı ihtiyaçları: ölçeğe göre üç profil

[← İçindekiler](../README.md)

Aynı mimari tek bir makineden kurumsal kümeye kadar büyütülebilir; başlangıçta en küçük profil ile kurup darboğaz görünce bir üst profile geçmek doğru yoldur.

| Bileşen | Küçük (kişisel / PoC) | Orta (ekip) | Kurumsal (banka) |
| --- | --- | --- | --- |
| Kuyruk | Tek düğüm Redpanda veya Kafka (Docker) | 3 broker Kafka, schema registry | Çok bölgeli Kafka (Confluent, MSK), ACL, şifreleme, replikasyon |
| Stream işleme | Python consumer veya tek düğüm Flink | Flink/Spark kümesi | Kubernetes üzerinde Flink operatörü, otomatik ölçekleme |
| Batch işleme | Python + cron / n8n | Airflow + Spark veya dbt | Airflow/Control-M + Spark kümesi, Dataiku |
| Depolama | PostgreSQL + TimescaleDB, Parquet klasörü | ClickHouse + S3/MinIO üzerinde Iceberg | Oracle/Snowflake DWH + lakehouse + kdb+ |
| İzleme | Grafana | Prometheus + Grafana, Kafka UI | Kurumsal izleme, nöbet (on-call), SLA |
| Çalışma ortamı | Docker Compose, tek VM | Birkaç VM veya küçük Kubernetes | OpenShift/Kubernetes, ayrı test-prod ortamları |

## Donanım dışı ihtiyaçlar

- Veri sağlayıcı lisansı (gerçek zamanlı borsa verisi genelde ücretlidir)
- Şema ve veri sözlüğü
- Veri kalite kuralları
- Erişim yetkilendirmesi
- Saklama süresi politikası
- Stream sistemleri için 7/24 operasyon sorumluluğu

[← Önceki](06-hisse-stream.md) · [Sonraki: Karar rehberi →](08-karar-rehberi.md)
