## Modern Web Mimarisi ve Ağ Güvenliği Notları (0x04)

Bu doküman, bir cihazın ağa girişinden itibaren modern web ekosistemindeki veri akışını, protokolleri ve altyapı bileşenlerini güvenlik (Pentesting) perspektifiyle ele alır.

### 1. Ağ Katmanında İlk Adımlar: Cihazın Hayata Dönüşü

Bir bilgisayar ağ kablosunu taktığınızda veya Wi-Fi'ya bağlandığınızda henüz bir kimliği yoktur. İletişimin başlaması için şu süreçler işler:
A. IP Edinme Süreci: DHCP (Dynamic Host Configuration Protocol)

Bilgisayar, yerel ağda kendine bir yer edinmek için DORA sürecini başlatır:

- Discover (Keşif): Cihaz, 0.0.0.0 IP'si ile 255.255.255.255 (Broadcast) adresine bir paket fırlatır: "Burada bana adres verecek bir otorite var mı?"

- Offer (Teklif): DHCP Sunucusu, havuzundan boş bir IP seçer ve teklif eder.

- Request (İstek): Cihaz, "Tamam, bu IP'yi istiyorum" der.

- Acknowledge (Onay): Sunucu kaydı yapar ve cihaz artık ağın bir parçasıdır.

- Pentest Notu: Bir saldırgan ağda sahte bir DHCP sunucusu kurarak (Rogue DHCP), trafiği kendi üzerinden geçirebilir (MITM).

### B. Yerel İletişim: ARP (Address Resolution Protocol)

IP adresleri mantıksal adreslerdir; ancak verinin fiziksel olarak bir kablodan veya frekanstan geçmesi için MAC adresine ihtiyaç vardır.

- ARP Table: Bilgisayarlar her seferinde sormamak için IP-MAC eşleşmelerini RAM'de tutar.

- Süreç: 192.168.1.5 nolu cihaz, 192.168.1.10 ile konuşmak istediğinde önce ARP tablosuna bakar. Yoksa bir ARP Request (Broadcast) gönderir.

- Pentest Notu: ARP Spoofing saldırısı ile hedef cihaza "Gateway benim, MAC adresim de bu" diyerek tüm internet trafiğini üzerinize çekebilirsiniz.

## 2. Web İsteği: İsimden Sunucuya Yolculuk

### A. DNS (Domain Name System) Çözümleme

x.com yazdığımızda makine bunu anlamaz. IP'ye ulaşmak için şu hiyerarşiyi izler:

- Local Cache & Hosts File: İlk durak bilgisayarın kendi hafızası ve /etc/hosts (veya System32\drivers\etc\hosts) dosyasıdır.

- Recursive Resolver: ISP'nin veya Google'ın (8.8.8.8) DNS sunucusu sizin adınıza dünyayı dolaşır.

- Root & TLD Servers: .com, .net gibi uzantıların yetkililerine sorulur.

- Authoritative DNS: Alan adının gerçek sahibinin sunucusuna gidilir ve nihai IP alınır.

- NAT (Network Address Translation): Yerel ağdaki 192.168.x.x IP'niz dışarı çıkarken Router tarafından Public IP'ye dönüştürülür. Bu, IPv4 adres kıtlığı için bir zorunluluktur.

### B. TCP 3-Way Handshake (Güvenli Bağlantı İnşası)

HTTP, güvenilir bir taşıma ister. Bu yüzden veriden önce "el sıkışılır":

- SYN (Synchronize): İstemci: "Bağlantı kurmak istiyorum, sıra numaram X."

- SYN-ACK: Sunucu: "İsteğini aldım, ben de istiyorum, benim sıra numaram Y."

- ACK (Acknowledge): İstemci: "Anlaşıldı, başlıyoruz."

- Güvenlik Katmanı: Eğer site HTTPS ise, bu aşamadan hemen sonra TLS Handshake başlar (Sertifika kontrolü ve şifreleme anahtarı değişimi).

## 3. Modern Sunucu Mimarisi: Zone Analizi

Klasik "Tek Sunucu, Tek Veritabanı" yapısı modern dünyada (yüksek trafikli e-ticaret, sosyal medya vb.) geçerli değildir.

### A. Load Balancing & Reverse Proxy

Binlerce isteği tek bir sunucu karşılayamaz. Bu yüzden en öne bir Load Balancer (Nginx, F5, HAProxy) konur.

- Yatay Ölçeklendirme: İhtiyaç anında yan yana 10 tane daha sunucu (App Server) açılır.

- Health Check: Load balancer, çöken sunucuyu anlar ve trafiği oraya göndermeyi keser.

### B. Session Yönetimi ve Redis (In-Memory DB)

Eğer 1. sunucuda login olduysanız ve Load Balancer sizi 2. sunucuya yönlendirirse, 2. sunucu sizi tanımaz (Session diskteyse).

- Çözüm: Session bilgileri sunucu diskinde değil, çok hızlı çalışan ve RAM üzerinde duran Redis gibi merkezi bir veritabanında tutulur.

### C. CDN ve Statik İçerik Yönetimi

Görseller, JS ve CSS dosyaları ana sunucuyu yormamalıdır.

- CDN (Content Delivery Network): İçerik kullanıcıya en yakın lokasyondaki sunucudan (Edge Server) sunulur.

- Upload Güvenliği: Eskiden yüklenen bir PHP dosyası ile sunucuda kod çalıştırılabiliyordu (Remote Code Execution). Modern yapıda dosyalar doğrudan S3/Cloud Storage gibi izole alanlara gider; sunucu diskiyle temas etmez.

### D. Disaster Recovery (Felaket Kurtarma)

Tüm bu sistem (Load Balancer + App Servers + Redis + DB) bir veri merkezinde durur. Eğer o veri merkezinde yangın çıkarsa, sistem saniyeler içinde başka bir ülkedeki (Region) kopyasına geçer.


### Özet Saldırı İpuçları:

- ARP Spoofing: Yerel ağdaki ARP mekanizmasını manipüle ederek trafiği araya alabilirsiniz.

- DNS Hijacking: DNS sorgularını yanlış yönlendirerek kullanıcıyı sahte sitelere çekebilirsiniz.

- Session Fixation: Session yönetiminin Redis gibi merkezi bir yerde ama güvensiz yapılandırılması sızıntılara yol açabilir.

- Misconfigured CDN: CDN üzerindeki cache kurallarının yanlış yapılandırılması, hassas verilerin (API key, user info) cache'lenmesine neden olabilir.