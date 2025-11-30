# E-Ticaret Yönetim Sistemi

Bu proje, temel bir e-ticaret yönetim sistemi için PostgreSQL veritabanı yapısını içerir. Sistem, kategoriler, ürünler, müşteriler, siparişler ve ilgili işlemleri yönetir. Fiyat değişikliklerini loglama, stok yönetimi ve sipariş süreçleri fonksiyonlar, triggerlar ve prosedürler ile desteklenmektedir.

---

## 📥 Kurulum

1. PostgreSQL’i açın.
2. `yasemin_sireci_proje.sql` dosyasını çalıştırın:
   - Tüm tablolar, fonksiyonlar, triggerlar, prosedürler ve view’lar otomatik olarak oluşturulacaktır.

---

##  🗂 Tablolar ve İlişkiler
- **categories:** Ürün kategorilerini tutar
- **products:** Ürün bilgileri, fiyat, stok vb. 
- **customers:** Müşteri bilgileri
- **shipping_addresses:** Müşteri teslimat adresleri
- **orders:** Sipariş bilgileri
- **order_items:** Sipariş ürün detayları
- **reviews:** Ürün değerlendirmeleri
- **product_price_logs:** Ürün fiyat değişiklikleri logları

**🔗 1 → Many (1-N) ilişkiler:**
✔ Bir kategori birçok ürün içerir  
✔ Bir müşteri birden fazla sipariş verebilir  
✔ Bir müşterinin birden fazla adresi olabilir  
✔ Sipariş belirli bir adrese bağlıdır  
✔ Bir ürün birçok sipariş satırında bulunabilir   
✔ Bir sipariş içinde birden fazla ürün olabilir  
✔ Bir müşteri birçok yorum bırakabilir  
✔ Bir ürün içinde birden fazla yorum bırakabilir   
✔ Bir ürün için birçok fiyat logu (price_logs) olabilir  

**⚠ Cascade Davranışları:** - `orders.customer_id` → ON DELETE CASCADE,
- `shipping_addresses.customer_id` → ON DELETE CASCADE,
- `order_items.order_id` → ON DELETE CASCADE,
-- Bu sayede müşteri silinirse → gönderi adresi, sipariş ve order_items otomatik silinir.

**SET NULL olanlar:** - `products.category_id` → kategori silinirse ürün NULL kategoriye düşer,
- `order_items.product_id` → ürün silinirse order_item ürün ID NULL olur ,
- `orders.shipping_address_id` → adres silinirse adres NULL olur

---

## 🔧  Fonksiyonlar

- `calculate_order_total(order_id)`: Siparişin toplam tutarını hesaplar.  
- `customer_lifetime_value(customer_id)`: Müşterinin toplam harcamasını hesaplar (yalnızca teslim edilen siparişler).  
- `stock_status(product_id)`: Ürünün stok durumunu "Bol / Orta / Az / Tükendi" olarak döndürür.  

---

## ⚡ Triggerlar

- **Stok Azaltma:** Sipariş oluşturulduğunda ürün stokunu otomatik düşürür.  
- **Fiyat Değişikliği Loglama:** Ürün fiyatı değiştiğinde eski ve yeni fiyatları loglar.  
- **Stok Geri Yükleme:** Sipariş iptal edildiğinde stokları geri yükler.  

---

## 🛠  Stored Procedure’ler

- `sp_place_order(customer_id, product_id, quantity)`: Stok kontrolü yaparak yeni sipariş oluşturur.  
- `sp_cancel_order(order_id)`: Siparişi iptal eder ve stokları geri yükler.  

---

## 📊  View’lar

- `vw_category_sales`: Kategorilere göre satış istatistikleri.  
- `vw_customer_order_summary`: Müşterilerin toplam sipariş sayısı, harcaması ve ortalama sepet tutarı.  

---

##  📦 E-Ticaret Sipariş Akışı Süreci Sistem Nasıl İşliyor?

1. **Sipariş Oluşturma Veri Akışı**
Müşteri -> Sipariş Ver (sp_place_order) -> orders tablosu INSERT -> Trigger çalışır: reduce_stock -> products.stock -= qty -> Sipariş Hazırlanır -> Müşteri ürünü teslim alır -> review ekleyebilir
   
2. **Sipariş İptali Veri Akışı**
CALL sp_cancel_order(order_id) -> orders.status = 'cancelled' -> Trigger çalışır: restore_stock -> products.stock += quantity

3. **Ürün Fiyat Güncelleme Akışı (Trigger Log)**
UPDATE products SET price = X -> Trigger: log_product_price_change -> product_price_logs tablosuna eski_fiyat / yeni_fiyat yazılır
