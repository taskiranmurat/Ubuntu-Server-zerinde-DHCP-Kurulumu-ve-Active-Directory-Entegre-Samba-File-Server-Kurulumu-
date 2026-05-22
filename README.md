# Ubuntu-Server-zerinde-DHCP-Kurulumu-ve-Active-Directory-Entegre-Samba-File-Server-Kurulumu-
Ubuntu Server Üzerinde DHCP Kurulumu  ve Active Directory Entegre Samba File Server Kurulumu 

# Ubuntu Server Üzerinde DHCP Kurulumu ve Active Directory Entegre Samba File Server Kurulumu 

Bu çalışmada Ubuntu Server üzerinde DHCP servisi kurulmuş, Samba File Server yapılandırılmış ve sistem Windows Server üzerinde çalışan Active Directory domain yapısına dahil edilmiştir. Kerberos, SSSD ve Winbind servisleri kullanılarak Ubuntu sunucusunun domain kullanıcılarını doğrulaması sağlanmış, departman bazlı paylaşım klasörleri oluşturularak yetkilendirme işlemleri gerçekleştirilmiştir. Her biri screen shotlarla desteklenmiş ve anlatılmıştır.

Ubuntu server için LTS 24.04.4 seçilmiştir destek süresi standart destek süresi 2029 sona ermesi ise 2039 uzun bir süre alması bizim için iyi bir seçimdir.

## Sistem Mimarisi ve Bileşen Tablosu

| Bileşen / Özellik | Kullanılan Teknoloji / Versiyon |
| :--- | :--- |
| **İşletim Sistemi** | Ubuntu Server 24.04.4 LTS (Noble Numbat) |
| **Sanallaştırma Ortamı** | VMware Workstation |
| **Merkezi Kimlik Doğrulama** | Windows Server Active Directory Entegrasyonu |
| **Dinamik Adresleme Servisi** | ISC-DHCP-Server |
| **Dosya Paylaşım Protokolü** | Samba  |
| **Kimlik Doğrulama Servisleri** | SSSD, Winbind, Kerberos v5 |
| **Yetkilendirme Altyapısı** | AD Gruplarına Göre Departman Bazlı Dosya İzinleri |
| **Sistem Mimarisi** | x86_64 (64-bit) |


## Sanal Makine Oluşturma ve Ubuntu Server Kurulumu

<img width="486" height="521" alt="image20" src="https://github.com/user-attachments/assets/c139c85c-78b4-469d-8686-7785e5cde46f" />

Vmware de bu şekilde sanal makine oluşturulmuştur. Özellikle bridge network seçildi.Updateler için internet bağlantısı gerektiği için kendi fiziksel makinemin internetime ihtiyacım var. İso’yu ayarlardan takıldı ve power butonuna basıldı.

<img width="1701" height="937" alt="image12" src="https://github.com/user-attachments/assets/f6334fd9-f453-4dd2-b88f-3748dc3becab" />

İşletim sistemi dil seçimi ile başlandı. İngilizce ile devam edilmiştir.

<img width="1566" height="858" alt="image25" src="https://github.com/user-attachments/assets/ae8c4bea-d8e0-4398-8989-75be4c0b5de7" />

Daha sonra updateler yapılmıştır. Sonraki bölümde klavye seçimi yapıldı ve standart ubuntu server seçimi yapılmıştır.

<img width="1311" height="223" alt="image1" src="https://github.com/user-attachments/assets/be471aa2-53ec-451e-9d83-6cd60a67ab07" />

Dhcp den ip aldı 192.168.1.106 aldı ve ethernetiin ismi de ens33. 

<img width="904" height="568" alt="image14" src="https://github.com/user-attachments/assets/e0572ebd-d6ef-405d-9645-d7f45a97c75f" />

Disk bölümlendirme bölümünde 1GB boot, 25 GB roota geriye kalanı da swap olarak ayarlanmıştır.

<img width="1228" height="404" alt="image10" src="https://github.com/user-attachments/assets/4ad182b4-61c7-4d04-ab2b-4268da1685fc" />

Profil ayarları yapılmıştır.  

<img width="644" height="672" alt="image3" src="https://github.com/user-attachments/assets/4162c258-9fdc-49f9-b09b-858ba67d4598" />

Kurulum da bittikten sonra ubuntu serverımızı reboota gönderdik isoyu çıkarmayı unutmuyoruz ondan sonra kullanıma hazırdır.


## Ubuntu Serverı Hazırlama

### Saat Ayarlama

<img width="1193" height="671" alt="image19" src="https://github.com/user-attachments/assets/def84ffc-9a8c-4c92-836e-5aa1ff146902" />

Serverlar da saat önemlidir doğru yer ve saat ayarlanmalıdır. **“tzselect”** komutu ile saat ayarlanır. Komutu girdikten sonra avrupa ve Türkiye seçildikten sonra saat ayarlanmış olur. 


### Rootu Açma

**“sudo passwd root”** komutu ile root açılır ve root’un passwordu verilir. 

### Update

**“apt update -y && apt upgrade -y”** ile güncellemeler yapılır

Gerekli olabilir diye de bazı programlar indirebiliriz. **“apt install nano wget git open-vm-tools network-manager”**  bu komut ile indirilebilir. 

* **nano:** Dosya içine girebilmek (düzenleyebilmek) için gerekli pakettir.
* **wget:** Dosya çekmek için gerekli pakettir.
* **open-vm-tools:** VMware ile işletim sistemi arasında sorunsuz iletişim için gerekli pakettir.
* **network manager:** Network bilgilerini girmek için gerekli pakettir.

Her şey tamam bu aşamada reboota gitmek sistem için iyidir.


### Network Bilgilerini Tanımlama


* Server olduğu için statik ip verilecektir ama önce netplan’dan network managera geçiş yapmak istiyorum network manager da grafik arayüz olduğu için. 

* Network managera geçiş yapmak için. **“nano /usr/lib/NetworkManager/conf.d/10-globally-managed–devices.conf** bu komutun içine girip **except:type:ethernet** yani etherneti kontrol et diyoruz. Config dosyası yapılandırdığımız için **“systemctl restart NetworkManager”** komutu ile servisi restart etmek gerekir.

<img width="620" height="794" alt="image21" src="https://github.com/user-attachments/assets/255aa169-61ea-49d3-9ebb-089b29ae3e62" />

**“nmtui”** komutu ile ethernet ekliyoruz ethernet adı “ens33” ve statik ip ve gateway verilmiştir gateway önemli çünkü internetten indirmeler yapacağız. Domain ismi de verilmiştir dns şimdilik 1.1.1.1 verilmiştir indirmelere yapılacağı için  daha sonra join olmadan önce dns serverın ip'si atanacaktır. Daha sonra da etherneti active ederiz. 

<img width="369" height="130" alt="image24" src="https://github.com/user-attachments/assets/411b401a-be2c-43e1-9e5e-3af7fb58f760" />


## DHCP Server Kurulumu

#### Dhcp Nedir?
DHCP (Dynamic Host Configuration Protocol), ağ üzerindeki cihazlara otomatik olarak IP adresi, ağ maskesi, ağ geçidi ve DNS bilgilerini dağıtan bir protokoldür. Bu sayede istemci cihazların ağ ayarlarının manuel olarak yapılandırılmasına gerek kalmaz ve ağ yönetimi daha kolay hale gelir. 

Ubuntu da **“apt install isc-dhcp-server”** komutu ile dhcp server repolardan çekilir. 

Dhcp Server indirildikten sonra config dosyasına gitmek için **“nano /etc/dhcp/dhcpd.conf”** dosyasına girip konfigürasyonları yapacağız. 


<img width="629" height="373" alt="image5" src="https://github.com/user-attachments/assets/308549ea-a469-4306-939b-f169ea4f74c5" />

* **authoritative:** "Bu ağın tek yetkilisi benim" demektir. Eğer ağda başka bir DHCP ( varsa onu susturur ve önceliği kendine alır.
* **default-lease-time;:** Bir cihaza verilen IP adresinin standart kira süresidir (10 dakika). Süre bitince cihaz "bu IP bende kalsın mı?" diye sorar.
* **max-lease-time:** Bir cihazın bu IP adresini en fazla ne kadar süre  elinde tutabileceğini belirler.
* **subnet netmask :** Bu ayarların hangi ağ aralığı için geçerli olduğunu tanımlayan ana çerçevedir.
* **range ;:** Dağıtılacak IP adreslerinin başlangıç ve bitiş sınırıdır.
* **option domain-name :** Cihazların hangi domaine ait olduğunu söyler.
* **option domain-name-servers :** DNS serverı işaret eder. AD ile kurula DNS serverın ip'si atanmıştır.
* **ddns-update-style interim;:** DHCP sunucusunun, IP verdiği cihazların isimlerini DNS servera  otomatik olarak gidip kaydetmesini sağlar.
* **ignore client-updates;:** Cihazların kayıt işlemini tamamen DHCP sunucusunun kontrolüne bırakır.
* **ddns-domainname :** DNS kayıtlarının hangi domain altına ekleneceğini belirler.
* **ddns-rev-domainname:** IP adresinden isim bulmaya yarayan "ters kayıtların" formatını belirler.

Daha sonra **/etc/default/isc-dhcp-server**  gidip **INTERFACESv4="ens33"** yazılmıştır. dhcpd’nin dinlemesi gereken interface yazılır. Ardından **”sudo systemctl restart isc-dhcp-server.service”** servis açılır.

<img width="534" height="420" alt="image22" src="https://github.com/user-attachments/assets/a8eef5bb-7c80-4fac-8112-e20d78ffc682" />

Clientta(windows 11) test edip ip’ye baktığımız da verdiğimiz ilk ip ve dns suffiximiz görünüyor dhcp server başarılı olmuş demektir.


## Samba File Server Kurulumu ve Active Directory’e Entegre Etme


Bu bölümde Ubuntu Server üzerine Samba File Server kurulumu gerçekleştirilecek ve sistem Windows Active Directory domain yapısına entegre edilecektir. 

* **Kerberos, SSSD ve Winbind servisleri kullanılarak Ubuntu sunucusunun domain kullanıcılarının doğrulaması  yapılacak, Active Directory üzerindeki kullanıcı ve gruplar Samba paylaşım izinlerinde kullanılacaktır.**

Mimarisi aşağıdaki gibi olacaktır.

Windows AD (192.168.1.150) (DNS + Kerberos)  →  Ubuntu Server (192.168.1.200) (realm join)  → SSSD veya Winbind → Samba File Server → AD Groups (IK, IT, SATIS, MUHASEBE) 


#### File Server Nedir?

File Server, ağ üzerindeki kullanıcıların dosya ve klasörlere merkezi olarak erişmesini, dosya paylaşımı yapmasını ve verileri güvenli şekilde saklamasını sağlayan sunucu sistemidir. Kullanıcı yetkilendirmeleri sayesinde belirli kullanıcı veya gruplara erişim izinleri verilebilir. 


### Ubuntu DNS ve Windows AD’ye yönlendirme 

<img width="298" height="186" alt="image2" src="https://github.com/user-attachments/assets/3e4c2a26-af5b-46f7-aa26-11b7732723ac" />

**/etc/systemd/resolved.conf** dosyasında  dns serverımızın ip’sini ve domain ismini vererek ubuntuyu dns servera ve AD’ye yönlendirme yapılmıştır. Ubuntu artık dns sorgularını Windows Server’a yollar. AD user bulma, Kerberos,Domain join, Samba  hepsi DNS’e bağımlıdır.


<img width="712" height="254" alt="image13" src="https://github.com/user-attachments/assets/e5ff1f0d-802d-4fac-9b77-d1608944c950" />

**nslookup** ile test edildiğinde serverın ip'si ve domain ismi doğrulanmıştır. Yönlendirme doğru bir şekilde yapılmıştır.


### Gerekli Paketlerin İndirilmesi

"sudo apt install -y realmd sssd sssd-tools libnss-sss libpam-sss adcli samba winbind libnss-winbind libpam-winbind krb5-user packagekit samba-common-bin oddjob oddjob-mkhomedir"

* **realmd** = Linux’u AD domain’e join etmek için
* **sssd** = AD kullanıcılarını Linux’a getirir
* **sssd-tools** = SSSD yönetim/test araçları
* **libnss-sss** = Linux user/group sistemi  SSSD bağlantısı
* **libpam-sss** = AD ile login/authentication sağlar
* **adcli** = Active Directory join işlemleri
* **samba** = Windows dosya paylaşımı 
* **winbind** = AD user ve group bilgilerini çeker
* **libnss-winbind** = Linux user sistemi  Winbind bağlantısı
* **libpam-winbind** = Winbind ile login desteği
* **krb5-user** = Kerberos client araçları (kinit, klist)
* **packagekit** = realm için dependency yönetimi
* **samba-common-bin** = Samba yönetim araçları (testparm, net ads)
* **oddjob** = Otomatik işlem servisi
* **oddjob-mkhomedir** = Login olunca otomatik home klasörü oluşturur

### Kerberos  Yapılandırılması

Kerberos yapılandırması, Ubuntu sunucusunun Windows Active Directory domainiyle güvenli şekilde haberleşmesini sağlar.

Yani Ubuntu: AD server’ı nerede biliyor ,hangi domain’e ait olduğunu biliyor,kullanıcı doğrulamasını (authentication) Kerberos ile yapabiliyor,domain kullanıcılarının şifresini güvenli şekilde kontrol edebiliyor,Samba’nın AD kullanıcılarıyla çalışmasını sağlıyor.

**nano /etc/krb5.conf** dosyasın da ayarlar yapmalıyız.

<img width="373" height="400" alt="image7" src="https://github.com/user-attachments/assets/33e185b8-9510-42d5-9a63-647571ce954a" />

### Kerberos (krb5.conf) Yapılandırması

Ubuntu sunucunun Active Directory Domaine güvenli şekilde bağlanabilmesi için `/etc/krb5.conf` konfigürasyon dosyası görseldeki gibi düzenlenmiştir:

* **[libdefaults]:** Varsayılan etki alanı (`default_realm`) **MURAT.LOCAL** olarak belirlenmiş ve DNS üzerinden KDC (Key Distribution Center) sorgulamaları aktif edilmiştir.
* **[realms]:** **MURAT.LOCAL** domain yapısı için kimlik doğrulama sunucusu (`kdc`) ve yönetim sunucusu (`admin_server`) olarak **192.168.1.150** IP adresi tanımlanmıştır.
* **[domain_realm]:** `murat.local` ve `.murat.local` uzantılı ağ isteklerinin doğrudan **MURAT.LOCAL** Kerberos alanına yönlendirilmesi sağlanmıştır.


<img width="743" height="241" alt="image18" src="https://github.com/user-attachments/assets/e4669d65-bdb5-4bcc-a3a3-d0ef641d36a6" />


Kerberos çalışmış. Ubuntu, AD server’dan Kerberos ticket almış. DNS çalışıyor,Kerberos çalışıyor, AD server bulunuyor,Authentication başarılı olmuş.


### Ubuntu’yu Join Etme

**“sudo realm join murat.local -U Administrator”**  bu komut ile join edilir

<img width="577" height="315" alt="image6" src="https://github.com/user-attachments/assets/b5f65873-c5e0-476a-8c1e-0d514ec14878" />

Başarıyla join edilmiştir.

### SSSD Yapılandırması 

SSSD (System Security Services Daemon) Linux’un dış kimlik sistemleriyle çalışmasını sağlayan bir authentication servisidir. 

<img width="411" height="323" alt="image9" src="https://github.com/user-attachments/assets/1130a1b7-fb4b-4bd7-89ab-70c03c06c1ba" />

Ayarlar bu şekilde yapıldı. Bu ayar AD user/group bilgilerini çeker ,login doğrulaması yapar,kerberos ile çalışır,linux’un AD kullanıcılarını normal kullanıcı gibi görmesini sağlar.
	
**“sudo chmod 600 /etc/sssd/sssd.conf “** root’a ait olmalı dosya o yüzden değlştirdik yetkisini. Ardından dosyayı restart edip açıyoruz servisi.


### NSS Ayarı 

NSS (Name Service Switch), Linux’un kullanıcı, grup ve kimlik bilgilerini nereden alacağını belirleyen sistemdir. Linux kullanıcıyı nerede arayacak,grubu nereden bulacak, local mi AD mi? bunu NSS üzerinden öğrenir.

<img width="373" height="182" alt="image16" src="https://github.com/user-attachments/assets/2da4abcf-2748-4b99-aaed-afc81db00d0b" />

**“sudo nano /etc/nsswitch.conf “** dosyasında bu ayarlar bu olmalı. Bu ayarlar Kullanıcı ve grup bilgilerini sırayla:**local files (/etc/passwd),systemd,SSSD,Winbind** üzerinden ara demektir.

<img width="863" height="96" alt="image4" src="https://github.com/user-attachments/assets/47d2af0e-3e6d-4c9f-8709-9cb50cd67dec" />

Testimiz başarılı her şey yolunda gözüküyor. AD’den bilgileri çekebildik.


### Samba klasörleri Oluşturma

<img width="549" height="131" alt="image23" src="https://github.com/user-attachments/assets/7be155d5-654d-4744-b511-54b923651e30" />

Bu komutla klasörler oluşturulmuştur.

### Yetkiler

<img width="543" height="156" alt="image11" src="https://github.com/user-attachments/assets/2b5ab504-b139-46f7-9585-186459c9d66b" />

Bu işlem ile Samba paylaşım klasörlerinin sahiplik ve erişim izinleri yapılandırılmıştır. chown komutu kullanılarak klasörün grup sahibi Active Directory üzerindeki ilgili departman grubuna atanmış, chmod 2770 komutu ile ise yalnızca yetkili kullanıcıların klasöre erişebilmesi sağlanmıştır. Böylece departman bazlı güvenli dosya paylaşımı gerçekleştirilmiştir. 


### smb.conf  Yapılandırması

<img width="460" height="721" alt="image17" src="https://github.com/user-attachments/assets/213adab2-5c97-4e74-9458-9c7545d23821" />


Bu yapılandırma dosyasında Samba servisi Active Directory domain ortamına uygun şekilde yapılandırılmıştır. 

Global bölümde sunucunun domain bilgileri tanımlanmış, güvenlik yöntemi olarak Active Directory Services (ADS) seçilmiş ve Kerberos tabanlı kimlik doğrulama etkinleştirilmiştir. Winbind servisi kullanılarak Active Directory üzerindeki kullanıcı ve grup bilgilerinin Linux sisteminde tanınması sağlanmıştır. Ayrıca idmap ayarları ile Windows SID bilgileri Linux UID/GID yapısına eşlenmiştir.

Yapılandırmada ACL desteği etkinleştirilerek Windows sistemlerindeki dosya izinlerinin Linux üzerinde uyumlu şekilde çalışması sağlanmıştır. Böylece dosya ve klasör izinleri merkezi olarak yönetilebilir hale getirilmiştir.


Paylaşım bölümlerinde ise her departman için ayrı klasörler tanımlanmıştır. IK, IT, SATIS ve MUHASEBE paylaşımlarına yalnızca ilgili Active Directory gruplarındaki kullanıcıların erişebilmesi sağlanmıştır. valid users parametresi ile yetkili gruplar belirlenmiş, create mask ve directory mask ayarları ile oluşturulan dosya ve klasörlerin erişim izinleri sınırlandırılmıştır. Böylece departman bazlı güvenli ve merkezi dosya paylaşımı yapısı oluşturulmuştur.


<img width="596" height="206" alt="image15" src="https://github.com/user-attachments/assets/32d457aa-0884-4722-880a-a8dd477db408" />

Testimiz başarılı olmuştur samba ad ile üye oldu ve file serverımız başarılı şekilde kurulmuştur.


<img width="792" height="593" alt="image8" src="https://github.com/user-attachments/assets/876cd2e2-9142-4b42-ae57-660cd01d0afc" />

Clientımız da test edildiğinde  başarılı bir şekilde file servera ulaşılabildiğini görebiliyoruz.












































