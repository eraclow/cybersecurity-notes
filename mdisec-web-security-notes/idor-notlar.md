# IDOR (Insecure Direct Object Reference)

## IDOR’u Anlamak İçin Temel Mantık

IDOR’u anlayabilmek için önce **web uygulamalarının temel çalışma mantığını** kavramak gerekir.

Bir web uygulaması, kullanıcıdan **input** (girdi) alarak çalışır.  
Uygulama senden hiçbir direktif almadığı sürece **attack surface daralır**.

Örneğin bir e-ticaret sitesinde:
- Kullanıcıdan teslimat adresi istenir
- Kullanıcı adresi girip onayladığında
- Bu adres veritabanında bir **ID** ile ilişkilendirilir
- Daha sonra bu ID kullanılarak adres tekrar kullanıcıya gösterilir

İşte IDOR’un hikâyesi tam olarak burada başlar.

---

## IDOR Nedir?

**IDOR (Insecure Direct Object Reference)**, uygulamanın kullanıcıdan aldığı bir referans (ID gibi) üzerinden **yetki kontrolü yapmadan** veritabanındaki nesnelere erişim sağlamasıdır.

Bu erişimler şunları kapsayabilir:
- Okuma (read)
- Güncelleme (update)
- Silme (delete)
- Değiştirme (modify)

### Basit Tanım

Normalde görmemem veya işlem yapmamam gereken **başka bir kullanıcıya ait veriye**, sadece ID’yi değiştirerek erişebiliyorsam bu bir **IDOR zaafiyetidir**.

Örnek:
- Başka bir kullanıcıya ait adresi görüntüleyebiliyorsam
- Başka bir kullanıcıya ait adresi silebiliyorsam

→ **IDOR vardır**

---

## Uygulamalı Örnek (Vulnerable E-Ticaret Senaryosu)

Zaafiyete açık bir e-ticaret sitesi olduğunu varsayalım.

### Senaryo Kurulumu

- İki farklı kullanıcı oluşturulur
- Her kullanıcı kendine ait bir adres kaydeder
- Kullanıcılardan biri kendi adresini silmeye çalışır

Bu sırada **Burp Suite** ile istek intercept edilir.

---

## Request Analizi

Adres silme isteği şu şekilde olsun:

```
GET /address/delete/15
```


- Diğer kullanıcının adres ID’si = **12**

---

## IDOR Testi

Şimdi kritik kısım başlar 👇

### Test 1 – Başka Kullanıcının ID’si

- Kendi kullanıcı oturumumuzdayken
- Request içindeki ID’yi **12** yapıyoruz
- Request’i tekrar gönderiyoruz

### Sonuç:
- **302 Redirect** alıyoruz
- Ancak profile döndüğümüzde **authorization failure** görüyoruz

Bu bize şunu gösterir:
- Uygulama ID’nin varlığını kontrol ediyor
- İşlem sırasında **yetki kontrolü yapılıyor**

---

### Test 2 – Random / Var Olmayan ID

Bu sefer ID’yi rastgele, veritabanında olmaması gereken bir değer yapıyoruz:

```
GET /address/delete/9999
```


### Sonuç:
- **404 Not Found**

---

## Buradan Ne Anlıyoruz?

Uygulamanın çalışma mantığı şu şekilde:

1. Önce ID var mı diye kontrol ediliyor
2. ID varsa ilgili fonksiyon çalıştırılıyor
3. Yetki kontrolü yapılıyor
4. Yetkisizse işlem iptal ediliyor

👉 Bu senaryoda **IDOR yok**, ancak IDOR mantığını anlamak için çok iyi bir örnek.

---

## IDOR vs Missing Function Level Access Control

Bu iki kavram **çok sık karıştırılır**, ama aynı şey değildir.

---

## Missing Function Level Access Control Nedir?

Bir kullanıcının **hiç erişmemesi gereken bir fonksiyona** erişebilmesidir.

Örnek:
- `edit` gibi admin-only bir fonksiyon
- Kullanıcı bu endpoint’e erişebiliyorsa → **Missing Function Level Access Control**

---

## Burp Suite ile Nasıl Bulunur?

- Endpoint’lere farklı payload’lar gönderilir
- `200 OK` dönenler analiz edilir
- Hangi fonksiyonlara erişim olduğu belirlenir

---

## IDOR ile Birlikte Görülmesi

Eğer:
- Bir kullanıcı `edit` fonksiyonuna erişebiliyorsa → **Missing Function Level Access**
- Aynı fonksiyonla **başka kullanıcıların verisini de etkileyebiliyorsa** → **Ekstra IDOR**

---

## Önemli Not

Bir yerde IDOR yok diye, uygulamanın başka bir yerinde de yok diyemeyiz.

Örnek:
- Sipariş onaylarken adres ID’sini değiştiriyorum
- Sipariş başka bir adrese gidiyor
- Sipariş geçmişinde o adresin detaylarını görebiliyorum

👉 Bu durum **Second Order IDOR** olarak adlandırılır.

---

## Second Order IDOR Nedir?

Kullanıcı girdisi:
- İlk aşamada zararsız gibi görünür
- Daha sonra başka bir işlemde kullanılır
- Bu ikinci aşamada yetkisiz veri açığa çıkar

## NOT
- Tek bir input birden fazla zaafiyete yol açabilir (IDOR, XSS, SQLi vb.)

---

## Özet

- IDOR, doğrudan nesne referanslarının kontrolsüz kullanımıdır.
- Yetki kontrolü her zaman **server-side** yapılmalıdır.
- ID bazlı işlemler her zaman test edilmelidir.
- Bir input’un sadece tek bir zaafiyeti olmayabilir.

## Makale

Aşağıdaki linkten IDOR ile alakalı her şeyi öğrenebileceğiniz makeleye ulaşabilirsiniz.

https://medium.com/@aysebilgegunduz/everything-you-need-to-know-about-idor-insecure-direct-object-references-375f83e03a87
