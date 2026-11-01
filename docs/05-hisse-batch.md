# 5. Hisse senedi örneği — batch hattı

[← İçindekiler](../README.md)

Amaç: her işlem günü kapanıştan sonra izlenen sembollerin günlük OHLCV verisini çekmek, getiri ve volatilite hesaplamak, DWH'de raporlamaya hazır tutmak. Gecikme ihtiyacı birkaç saattir; batch yeterlidir.

## Akış

1. **Zamanlama:** Airflow DAG'ı her iş günü 19:00'da (borsa kapanışından sonra) tetiklenir.
2. **Alım:** Python görevi veri sağlayıcıdan (yfinance, broker API, Matriks vb.) günün OHLCV verisini çeker, ham dosyayı tarih bazlı klasöre Parquet olarak yazar: `bronze/eod/dt=2026-10-30/`.
3. **Doğrulama:** Eksik sembol, sıfır hacim, tatil günü, aşırı fiyat sıçraması kontrolleri; başarısızsa DAG durur ve alarm verir.
4. **Dönüştürme:** Bölünme/temettü düzeltmesi (adjusted close), günlük log getiri, 20 günlük hareketli ortalama, 20 günlük volatilite, sektör eşlemesi.
5. **Yükleme:** Sonuç DWH'ye `(sembol, tarih)` anahtarıyla upsert edilir; aynı gün iki kez koşsa da tekrar kayıt oluşmaz.
6. **Sunum:** BI aracı gold tablodan raporu yeniler; ertesi sabah e-posta özeti gider.

## Örnek tablo yapısı

| Katman | Tablo | Anahtar kolonlar |
| --- | --- | --- |
| Bronze | `raw_eod_prices` | sembol, tarih, open, high, low, close, volume, kaynak, yükleme_zamanı |
| Silver | `eod_prices` | sembol, tarih, adj_close, log_getiri |
| Gold | `daily_stock_metrics` | sembol, tarih, ma_20, vol_20, sektör, ytd_getiri |

## Altyapı ihtiyacı

Bir orkestratör (Airflow), Python çalıştıracak bir sunucu veya konteyner, bir nesne deposu veya klasör (ham dosyalar), bir DWH veya PostgreSQL (sonuç tabloları), bir BI aracı. Kuyruğa gerek yoktur.

## Kod iskeleti

```python
# Airflow DAG iskeleti (sadeleştirilmiş)
with DAG("eod_prices", schedule="0 19 * * 1-5", catchup=True) as dag:
    extract   = PythonOperator(task_id="extract",   python_callable=fetch_eod)
    validate  = PythonOperator(task_id="validate",  python_callable=check_quality)
    transform = PythonOperator(task_id="transform", python_callable=compute_metrics)
    load      = PythonOperator(task_id="load",      python_callable=upsert_dwh)
    extract >> validate >> transform >> load
```

`catchup=True` sayesinde geçmiş günler için aynı DAG geriye dönük çalıştırılabilir (backfill); batch'in en büyük avantajlarından biri budur.

[← Önceki](04-mimari-desenler.md) · [Sonraki: Hisse senedi stream hattı →](06-hisse-stream.md)
