# MDISEC Web Security Eğitim Notları

## Web Security 0x01 | SQL Injection

### SQLI Nedir?
 - Kısaca yazdığınız payloadları web uygulamasını kullanarak database'de kendi SQL sorgunuzu çalıştırabilmenizdir.
 - Önce SQL'i anlamamız gerekir.Örnek temel sql sorguları:
  
  ```sql
  SELECT 1; 
  Sonuç 1 döner.
  ```
  ```sql
  SELECT 2-1;
  Veri tabanı çıkarma işlemini yapar ve sonuç yine 1 döner.
  ```
  ```sql
  SELECT '2-1';
  Veritabanı string olarak algılar ve sonucu 2-1 şeklinde geri verir.
  ```