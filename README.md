📥 Kurulum
PostgreSQL’i açın.
yasemin_sireci_proje.sql dosyasını çalıştırın:
Tüm tablo, fonksiyon, trigger, prosedür ve view'lar otomatik olarak oluşacaktır


E-Ticaret Sipariş Süreci Sistem Nasıl İşliyor?

Müşteri ->  Sipariş Ver     ->  orders tablosu INSERT ->   Trigger çalışır: reduce_stock ->  products.stock -= qty  ->    Sipariş Hazırlanır -> Müşteri ürünü teslim alır -> review ekleyebilir          
           (sp_place_order)

Sipariş İptali Veri Akışı

   CALL sp_cancel_order(order_id) ->  orders.status = 'cancelled' -> Trigger çalışır: restore_stock -> products.stock += quantity

 Ürün Fiyat Güncelleme Akışı (Trigger Log)

 UPDATE products SET price = X  -> Trigger: log_product_price_change -> product_price_logs tablosuna eski_fiyat / yeni_fiyat yazılır

1 → Many (1-N) ilişkiler
✔ Bir kategori birçok ürün içerir
✔ “Bir müşteri birden fazla sipariş verebilir”
✔ “Bir müşterinin birden fazla adresi olabilir”
✔ Sipariş belirli bir adrese bağlıdır
✔ Bir ürün birçok sipariş satırında bulunabilir
✔ Bir sipariş içinde birden fazla ürün olabilir
✔ Bir müşteri birçok yorum bırakabilir

Bir sipariş → birden fazla order_item

Bir ürün → birden fazla yorum

✔ Cascade Davranışları

orders.customer_id → ON DELETE CASCADE

shipping_addresses.customer_id → ON DELETE CASCADE

order_items.order_id → ON DELETE CASCADE

Bu sayede müşteri silinirse → gönderi adresi, sipariş ve order_items otomatik silinir.

✔ SET NULL olanlar

products.category_id → kategori silinirse ürün NULL kategoriye düşer

order_items.product_id → ürün silinirse order_item ürün ID NULL olur

orders.shipping_address_id → adres silinirse adres NULL olur

Bir ürün → birçok fiyat logu (price_logs)

