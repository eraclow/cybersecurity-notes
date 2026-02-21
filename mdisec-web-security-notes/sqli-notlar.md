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

  ## Örnek Olarak Kullanacağımız Kod ve URL

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

  ## SQL Injection Tespit

  - Kendimize öncelikle bir referans noktası seçmemiz gerekiyor.Bu senaryoda bizim referans noktamız www.x.com/?id=1.

  - ?id=1 query kısmını ?id=2-1 ile değiştirdikten sonra fark ediyoruz ki kendi referans noktamıza geri döndük ve aslında arka tarafta database'e bir çıkarma işlemi yaptırdık.

  - Sadece çıkarma işlemi yapmak zorunda değiliz örnek olarak referans noktası aldığımız URL'deki query ?id=3 olsun.Yukarıda bahsettiğim gibi 2^1 3'e eşit olduğu için bu şekilde de varlığını tespit edebiliriz.

  - Arka tarafta şu şekil bir query çalışıyor olabilir SELECT * FROM haberler WHERE id= ' " + id + " ' ".Bu senaryoda 2-1 bir işe yaramayacaktır çünkü input tırnak içine alınıyor.Bu senaryoda ise ?id=1' AND '1'='1 yazarak yine referans noktamıza dönebiliriz.Bu payload sorgunun tamamını TRUE yaparak filtreyi bypass eder.Query'nin devamında başka sorgular da çalışıyor olabilir bu yüzden sonuna # ya da -- ekleyerek yazdığımız kısımdan sonrasını commentout edebiliriz.

  ## UNION SELECT Kullanımı

  - UNION operator'ünü kullanarak kendi sorgumuzu çalıştırmak için query'deki column sayısının bizden önceki ve bizim yazacağımız query ile aynı olması lazım.Bunu da ya ' ORDER BY 1-- ya da UNION SELECT 1,2,3,4,5 gibi tekniklerle bulabiliriz.

  - UNION SELECT 1,2,3,4,5 tekniğini kullanarak kendi referans noktamıza döndüğümüz zaman kolon sayısını bulmuş oluyoruz.Kullandığımız UNION SELECT yöntemiyle kolon sayısını bulduktan sonra yazdığımız 1,2,3,4,5 column kısmındaki hangi değeri bir string ile değiştirirsek ekrana output verir onu bulmamız lazım.Bu sayede o kısımı kullanarak helper function'lar ile veya kendi yazacağımız subquery'ler ile veri çıkartmaya başlayabiliriz.Örneğimizde 1 değerindeki alana yazınca string output almış olalım.

  - Eğer bizden önceki sorgunun ekrana basılmamasını istiyorsak ?id=1 kısmını ?id=-2324 gibi bir karşılığı olmayan bir değerle değiştirebiliriz.

  - Her ilişkisel database'de tablo ve kolon adlarını barındıran bir sistem bilgi tablosu vardır.(information_schema.tables gibi)

  - Örnek olarak vereceğim query ile database'deki tablo isimlerini öğrenebiliriz:

  ```
  UNION SELECT table_name,2,3,4,5 FROM information_schema.tables WHERE table_schema=database()

  Genel olarak yazılması gereken payload'ları vermeyeceğim kendiniz rahatlıkla internet üzerinden bulabilirsiniz.
  ```

  ## Error-Based SQLi

  Error-based SQL injection, saldırganın bir web uygulamasındaki giriş alanlarına zararlı SQL ifadeleri enjekte etmesi sonucu uygulamanın SQL hataları üretmesine neden olan bir güvenlik açığı ve saldırı türüdür.

  Bu hatalar, veritabanının yapısı, tabloları, sütunları veya yapılandırması hakkında hassas bilgilerin açığa çıkmasına sebep olabilir.

  ## Error-Based SQL Injection Nasıl Çalışır?

  ## Injection noktası (Girdi noktası)

    Saldırgan, kullanıcı girdisinin doğrudan SQL sorgularına eklendiği savunmasız bir alan tespit eder.

     Örneğin:

      - Arama kutusu

      - Giriş (login) formu

      - Filtre parametreleri

      - URL parametreleri

    Bu ifadeler, uygulama tarafından çalıştırıldığında kasıtlı olarak SQL sözdizimi (syntax) hatalarına yol açacak şekilde tasarlanmıştır.

    Bu alanlar yeterince filtrelenmezse SQL injection için uygun hedef haline gelir. 
  
  
  - Error-Based SQLi ile veri çekmek için bazı helper function'lar kullanabiliriz.Örnek olarak ExtractValue() üzerinden yürüyelim.Bu fonksiyon parametre olarak xml alıp parse ederek belirttiğin filtreye göre değer döner.Örnek normal kullanım:

    ```sql
     ExtractValue(@xml, '//b[$@i]')
    ```

  - Önceki inject ettiğimiz URL üzerinden devam edebiliriz.WWW.x.com/?id=ExtractValue(rand(), concat(1,'Mehmet')) yazdığımızda XPATH syntax error: 'Mehmet' şeklinde bir hata alıyoruz.Yazdığımız string hatanın içerisinde geri döndü.Bu sayede subquery yazabiliriz.
  - Örnek payload:

     ```sql
     SELECT ExtractValue(rand(),concat(1,(SELECT database())));
     ```

  - Burada rand() fonksiyonunu kullanmamızın sebebi hata ürettirip kendi yazdığımız subquery'nin sonucunu ekrana basmak.

  ## Boolean-Based SQLi

  ## Referans Olarak Alacağımız Kod:
   
   ```
   if result.size() > 0:
        print("Haber Var")
   else:
        print("Haber Yok")
   ```

  - Bu koddan çıkaracağımız anlam:
      - Select sorgusu ekrana basılmıyor.
      - Eğer query çalıştıysa "Haber Var" yazacak ekranda,çalışmadıysa ise "Haber Yok" yazacak.
      - True,false oynamamız gerekiyor.


  - Burada da kullanabileceğimiz SUBSTRING() adlı bir fonksiyonumuz var.Bu fonksiyona bir string değer verdiğiniz zaman o string değerinin içinden karakter çıkarttırabiliyorsunuz.Örnek kullanım:

    ```
    SUBSTRING('MEHMET', 2, 1)
    Sonuç E döner çünkü buradaki 2 hangi karakterden başlayacağımızı belirtir.1 ise kaç tane getireceğini belirler.
    ```
  - SUBSTRING() fonksiyonunun içine bir subselect ekleyebiliriz:
    
    ```
    SUBSTRING((SELECT table_name FROM ınformation_schema.tables WHERE table_schema=database() LIMIT 1,1),1,1)=a
    LIMIT 1,1 kullanmamızın sebebi ilk tablonun ilk harfini bulmak.
    =a kısmını değiştirerek "Haber Var" yazısını alana kadar devam etmemiz lazım.
    ```
  - Bu süreci hızlandırmak için ASCII() fonksiyonunu kullanabiliriz.Bu fonksiyon bir karakterin ASCII sayı karşılığını döndürür.Binary Search tekniği olarak da adlandırılır.Büyüktür küçüktür oynamak gibi de düşünebilirsiniz.

  - Örnek payload:

    ```
    ASCII(SUBSTRING((SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 1,1), 1, 1)) > 104
    Eğer sonuç 104'den büyükse ASCII tablosundaki 104'den büyük olan numaraların aritmetik ortalamasını alıp aynı işlemi daha az HTTP request ile çözmüş oluyoruz.
    ```
 
  ## Time-Based SQLi

  ## Time-Based SQLi Nedir?
  
  - Time-Based SQL Injection, saldırganın veritabanının yanıt süresini kasıtlı olarak geciktirerek sorgunun doğru (TRUE) ya da yanlış (FALSE) olup olmadığını anlamaya çalıştığı bir blind (kör) SQL injection türüdür.

  - Klasik SQL injection saldırılarından farklı olarak, bu yöntemde veriler doğrudan elde edilmez. Bunun yerine, veritabanında bilinçli olarak gecikmeye neden olan SQL fonksiyonları tetiklenir.

  - Saldırgan, uygulamanın yanıt süresini ölçerek belirli bir koşulun doğru mu yoksa yanlış mı olduğunu anlayabilir. Bu sayede, ekranda herhangi bir çıktı veya hata mesajı olmadan bile sistemden bilgi sızdırmak mümkün olur.

  ## Nasıl Tespit Edilir?

  - Diyelim ki arkada SELECT * FROM haberler WHERE id = 1 şeklinde bir query çalışıyor.
  - ?id=1 AND SLEEP(5) yazınca database 5 saniye uyursa SQL injection var.
  - IF() fonksiyonunu kullanarak true ve false arasındaki zaman farklarını anlayabiliriz.Mesela:
    
    ```
    ?id=1 AND IF(1=1,SLEEP(5),0) #eğer 1 1'e eşitse 5 saniye bekle
    ?id=1 AND IF(1=2,SLEEP(5).0) #1 2'ye eşit olmadığı için 5 saniye uyumayacak
    ```
  
  - Eğer yukarıda da kullandığımız tekniklerle birleştirirsek:
   
   ```
   IF(ASCII(SUBSTRING(database(),1,1)) > 100, SLEEP(5), 0)
   Bu şekilde database'in adının ilk karakterini öğrenmeye çalışabiliriz.
   ```
  
    


      
    
  