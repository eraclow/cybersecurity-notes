## Web Security 0x3: Session Management & CSRF Vulnerability

Bu yazı, HTTP protokolünün doğasından kaynaklanan eksiklikleri, bu eksiklikleri gidermek için kullanılan Session/Cookie mekanizmalarını ve bu mekanizmaların suistimal edilmesiyle ortaya çıkan CSRF (Cross-Site Request Forgery) zafiyetini konu almaktadır.

### HTTP ve Stateless Yapısı

HTTP (HyperText Transfer Protocol), modern internetin temel taşıdır. En önemli özelliği stateless (durumsuz) olmasıdır.

 - Stateless Nedir?: Sunucu, gelen bir isteğin (request) bir önceki istekle bağlantısını bilmez. Her request, sunucu için yeni ve yabancıdır.

 - Problem: Kullanıcı bir kez giriş yaptıktan sonra, gezindiği her alt sayfada neden tekrar kullanıcı adı ve şifre sormuyor?

 - Çözüm: HTTP Header yapısına eklenen Cookie mekanizması.

### Cookie ve Session Mekanizması

  ## Cookie Nedir?

    Sunucu tarafından Set-Cookie header'ı ile tarayıcıya gönderilen küçük veri parçalarıdır. Tarayıcı bu veriyi yerel veri tabanında saklar ve ilgili domain'e yapılan sonraki tüm isteklerde otomatik olarak header'a ekler.

  ## Session (Oturum) Yönetimi

    Session, sunucu tarafında kullanıcıya ait verilerin tutulduğu alandır. Yaygın tutulma yöntemleri:

     - File System: /tmp gibi klasörlerde tutulur. Disk I/O (giriş/çıkış) işlemleri darboğaz (bottleneck) yaratabilir.

     - Database: Ölçeklenebilirlik sağlar ancak her request'te DB sorgusu maliyetlidir.

     - In-Memory (Redis/Memcached): En hızlı yöntemdir. Veri RAM üzerinde tutulur.

     - Client-Side Session: Sunucu yükünü azaltmak için session verisi doğrudan cookie içine yazılır.
  
  
  ## Client-Side Session & Güvenlik

    Veri istemcide tutulduğu için kullanıcı bunu değiştirebilir. Bunu engellemek için:

     - HMAC Signature: Veri, sunucuda saklı bir Secret Key ile imzalanır.

     - İşleyiş: Veri + Secret Key hashlenir. Eğer kullanıcı veriyi değiştirirse, hash tutmayacağı için sunucu isteği reddeder.

     - Encoding: Veri genellikle okunabilirliği sağlamak için Base64 ile encode edilir (Bu bir şifreleme değildir!).

### CSRF (Cross-Site Request Forgery)

CSRF, kurbanın tarayıcısını kullanarak, onun halihazırda oturum açmış olduğu bir web sitesinde isteği dışında işlemler (şifre değiştirme, para transferi vb.) yaptırma saldırısıdır.

## Neden Mümkün?

Tarayıcılar, bir siteye istek atarken o siteye ait cookie'leri otomatik olarak ekler. Sunucu, isteğin kaynağına (kullanıcı gerçekten butona mı bastı yoksa gizli bir script mi çalıştı?) bakmaz; sadece cookie geçerli mi diye bakar.

## Saldırı Senaryosu

    - Kullanıcı banka.com'da oturum açar.

    - Aynı tarayıcıda saldırganın hazırladığı kotu-site.com'a girer.

    - Kötü niyetli site arka planda şu isteği tetikler:

    ```html
    <img src="https://banka.com/transfer?to=hacker&amount=1000" style="display:none;">
    ```
    - Tarayıcı, banka.com cookie'lerini otomatik eklediği için işlem başarıyla gerçekleşir.

### Savunma Mekanizmaları ve SameSite

## CSRF Token (Anti-CSRF)

En yaygın yöntemdir. Sunucu, her oturum veya her form için benzersiz, tahmin edilemez bir Token üretir ve bunu HTML içine (hidden input) gömer.

 - Saldırgan bu token'ı başka bir domain'den okuyamaz (Same-Origin Policy nedeniyle).

 - Token içermeyen veya yanlış içeren istekler reddedilir.

## SameSite Cookie Ölemi

Google tarafından geliştirilen ve cookie'lerin hangi durumlarda gönderileceğini belirleyen bir attribute'tur.

  Strict   Cookie sadece "First-party" (aynı site) isteklerinde gönderilir. Dış linkten tıklayıp gelince bile cookie gitmez.

  Lax      (Modern tarayıcılarda default) Dış sitelerden gelen "Güvenli" isteklerde (Sadece GET ve üst seviye navigasyon) cookie gönderilir.

  None     Cookie her durumda gönderilir. Secure flag'i ile birlikte kullanılmalıdır.