# MDISEC Web Security Eğitim Notları

## Web Security 0x01 | SQL Injection

### SQLI Nedir?
 - Kısaca yazdığınız payloadları web uygulamasını kullanarak database'de kendi SQL sorgunuzu çalıştırabilmenizdir.
 - Önce SQL'i anlamamız gerekir.Örnek temel MySQL sorguları:
  
  ```
  SELECT 1; 
  Sonuç 1 döner.
  ```
  ```
  SELECT 2-1;
  Veritabanı çıkarma işlemini yapar ve sonuç yine 1 döner.
  ```
  ```
  SELECT '2-1';
  Veritabanı string olarak algılar ve sonucu 2-1 şeklinde geri verir.
  ```
  ```
  SELECT '2' - '1';
  Veritabanı değeri integer türüne dönüştürerek çıkarma işlemini gerçekleştirir. Sonuç 1’dir.
  ```
  ```
  SELECT '2' - 'a';
  DB integer tipine cast edemeyeceği için sonuç 2a'dır.
  ```
  ```
  SELECT 'b' + 'a';
  Sonuç 0'dır.
  ```
  ```
  SELECT '2' '1';
  Sonuç 21'dir.Veritabanı string concatenation yapar.
  ```
  ```
  SELECT '2' '1' 'a' - 1;
  Sonuç 20'dir.a - 1 = 0 - 1 = -1.21-1=20
  ```
  ```
  SELECT 2^1;
  Sonuç 3'tür.Bu bir XOR opperand'dır.
  ```

  ## MySQL Davranış Örneği
  Test adında bir DB oluşturup içine USERS diye bir tablo oluşturabiliriz ve column isimlerini ise isim,soyisim şeklinde ayarlayıp isim kısmına 'Mehmet' ve 'Mehmet     ' şeklinde eklemeler yapabiliriz.Mehmet kullanıcısını bul getir şeklinde bir query yazarsak sonunda boşluk olanlar dahil hepsi gelmektedir.DB sonunu trim'liyor.Bize yararı şu: Eğer 'admin     ' şeklinde bir kullanıcı oluşturup bize kullanıcı adı admin olanların parolasını getir dersek tüm admin isimli kullanıcıların verisi gelir.Exact Match açık olursa bu durum engellenir.

  ## Uygulama Kısmı
  
  ## UNION SELECT SQLi

  ## Örnek Olarak Alacağımız Kod ve URL

  ```
  www.x.com/?id=1

  id = request.get('id')
  query = "SELECT * FROM haberler WHERE id="+id
  result = db.execute(query)
  if result.size() > 0:
    for i in result:
        print(i.title)
  else:
    print("Haber Yok")
  ```

  - Kendimize öncelikle bir referans noktası seçmemiz gerekiyor.Bu senaryoda bizim referans noktamız www.x.com/?id=1.

  - ?id=1 query kısmını ?id=2-1 ile değiştirdikten sonra fark ediyoruz ki kendi referans noktamıza geri döndük ve aslında arka tarafta database'e bir çıkarma işlemi yaptırdık.

  - UNION operator'ünü kullanarak kendi sorgumuzu çalıştırmak için query'nin sonucunun column sayısı bizden önceki ve bizim yazacağımız query ile aynı olması lazım.Bunu da ya ' ORDER BY 1-- ya da UNION SELECT 1,2,3,4,5 gibi tekniklerle bulabiliriz.

  - UNION SELECT 1,2,3,4,5 tekniğini kullanarak kendi referans noktamıza döndüğümüz zaman kolon sayısını bulmuş oluyoruz.Kullandığımız UNION SELECT yöntemiyle kolon sayısını bulduktan sonra yazdığımız 1,2,3,4,5 column kısmındaki hangi değeri bir string ile değiştirirsek ekrana output verir onu bulmamız lazım.Bu sayede o kısımı kullanarak helper function'lar ile veya kendi yazacağımız subquery'ler ile veri çıkartmaya başlayabiliriz.Örneğimizde 1 değerindeki alana yazınca string output almış olalım.

  - Eğer bizden önceki sorgunun ekrana basılmamasını istiyorsak ?id=1 kısmını ?id=-2324 gibi bir karşılığı olmayan bir değerle değiştirebiliriz.

  - Her ilişkisel database'de tablo ve kolon adlarını barındıran bir sistem bilgi tablosu vardır.(information_schema.tables gibi)

  - Örnek olarak vereceğim query ile database'deki tablo isimlerini öğrenebiliriz:

  ```
  UNION SELECT table_name,2,3,4,5 FROM information_schema.tables WHERE table_schema=database()

  Genel olarak yazılması gereken payload'ları vermeyeceğim kendiniz rahatlıkla internet üzerinden bulabilirsiniz.
  ```
  
    
  