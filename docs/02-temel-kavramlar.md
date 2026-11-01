# 2. Temel kavramlar

[← İçindekiler](../README.md)

Stream dünyasındaki zorlukların çoğu zamanla ilgilidir: olay ne zaman oldu, ne zaman geldi, ne zaman hesaba katılmalı.

| Kavram | Anlamı | Hisse senedi örneğinde |
| --- | --- | --- |
| Event time | Olayın gerçekte olduğu an | İşlemin borsada gerçekleştiği saat |
| Processing time | Olayın sisteme ulaştığı an | Tick'in Kafka'ya düştüğü saat |
| Window (pencere) | Akışı zaman dilimlerine bölme: tumbling (örtüşmeyen), sliding (kayan), session | 1 dakikalık OHLCV mumu = tumbling window |
| Watermark | "Bu saatten önceki olaylar artık geldi" kabulü; geç olaylara ne kadar bekleneceği | 1 dk mum kapandıktan sonra 5 sn daha geç tick bekle |
| State | Akış işlemcinin tuttuğu ara sonuç | O dakikanın açılış, en yüksek, en düşük fiyatı |
| Checkpoint | State'in periyodik olarak kalıcı diske yazılması | Flink çökerse mumlar kaldığı yerden devam eder |
| Offset | Kuyruktaki mesajın sıra numarası | Tüketici "THYAO partisinde 10.523. mesajdayım" der |
| Partition | Bir topic'in paralel parçaları; sıra sadece partition içinde garantilidir | Anahtar = sembol; her sembolün tick'leri kendi içinde sıralı |
| Delivery semantics | At-most-once, at-least-once, exactly-once | Hacim toplamında bir tick iki kez sayılmamalı → exactly-once veya idempotent yazım |
| Backpressure | Tüketici yavaş kaldığında üreticiyi yavaşlatma | Açılış seansında tick patlamasında sistem çökmeden yavaşlar |
| Idempotency | Aynı işlemi iki kez yapmanın sonucu değiştirmemesi | `(sembol, zaman)` anahtarıyla upsert |
| Schema registry | Mesaj şemalarının merkezi kaydı ve uyumluluk kontrolü | Tick şemasına yeni alan eklenince tüketiciler bozulmaz |
| CDC | Veritabanı değişikliklerini log'dan olay olarak yakalama | Portföy tablosundaki pozisyon değişikliklerini akışa almak |

[← Önceki](01-batch-ve-stream.md) · [Sonraki: Teknoloji katmanları →](03-teknoloji-katmanlari.md)
