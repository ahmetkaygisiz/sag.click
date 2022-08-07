---
title: "Oracle FMW Infrastructure Silent Installation"
date: 2021-04-18T18:32:01+03:00
draft: true
---
Selamlar,

Geçmişte aklımda olan fakat zamanı gelmemiş bir çalışma konusu önüme düştü. Oracle FMW, WebLogic vb. gibi ürünlerde installation wizard yani kurulum ekranları gelmeden, gerekli olan bilgileri response file adı verilen dosyalarda belirterek yükleme yapılabilmesini sağlayan kurulum şekline silent installation deniyor. Bu yazıda sadece FMW Infrastructure'ın kurulumunu yapacağız. Sonraki yazılarımın konuları da Container Oracle DB ile rcu çalıştırılması ve SOA domain kurulumu olacak. Şimdi ortam hazırlıklarıyla başlayalım. 

### JDK
[Oracle'dan](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html) java 1.8'in JDK paketini kendimize uygun olanını [jdk-8u281-linux-x64.tar.gz](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html#license-lightbox) indiriyorum. İndirdiğim paketi java pathime çıkarıyorum.
```bash
# paketi java pathine çıkar
tar -xzf jdk-8u281-linux-i586.tar.gz -C /data/java
```
```bash
# linkimizi oluşturuyoruz
ln -s /data/java/jdk1.8.0_281 /data/java/latest
```
.zshrc dosyamda JAVA_HOME'u tanımlayıp export ediyorum. Bunu yapmamızın sebebi her oturum açtığımızda tanımlanan genel sistem değişkenleri arasında JAVA_HOME'un da bizim istediğimiz adresi göstermesi.

```bash
vim ~/.zshrc
# .zshrc uygun bir kısma ekliyoruz şu satırları
export JAVA_HOME=/data/java/latest

export PATH="$PATH:$JAVA_HOME/bin"
# export 
source .zshrc

# kontrol ettim
java -version
# java version "1.8.0_281"
```

### FMW Infrastructure 
[Fusion Middleware Infrastructure Installer](https://www.oracle.com/middleware/technologies/weblogic-server-installers-downloads.html#license-lightbox) (1.5 GB) 12.2.1.4 versiyonunu indiriyorum. Burada ihtiyacımız olan 2 adet konfigürasyon dosyası var. Birincisi oraInventory adresini belirteceğimiz oraInst.loc, ikincisi de fmw response file. Oluşturduğum dosyalar aşağıdaki gibi.

oraInst.loc

Burada inst_group kulumu yapacağımız kullanıcı grup adı, inventory_loc da oraInventory adresimiz olmalıdır.
```bash
inst_group=kaaygisiz
inventory_loc=/data/fmw/oraInventory
```
Response file da değiştirdiğim tek değişken ORACLE_HOME=/data/fmw/12.2.1.4 değeri. Burada kurulumu yapacağımız path'i belirtiyoruz. Burada ben kendi dev ortamımı kurduğum için çoğu değer default. Sizin kendi ihtiyaçlarınıza göre [dökümanlardan](https://docs.oracle.com/en/middleware/fusion-middleware/12.2.1.4/ouirf/sample-response-files-silent-installation-and-deinstallation.html#GUID-19DEEA75-CC63-47D4-BDC7-038E133490E0) takip ederek gerekli değişiklikleri yapabilirsiniz.
```bash
[ENGINE]
 
#DO NOT CHANGE THIS.
Response File Version=1.0.0.0.0
 
[GENERIC]
 
#Set this to true if you wish to skip software updates
DECLINE_AUTO_UPDATES=true

#My Oracle Support User Name
MOS_USERNAME=

#My Oracle Support Password
MOS_PASSWORD=

#If the Software updates are already downloaded and available on your local system,
#then specify the path to the directory where these patches are available and
#set SPECIFY_DOWNLOAD_LOCATION to true
AUTO_UPDATES_LOCATION=

#Proxy Server Name to connect to My Oracle Support
SOFTWARE_UPDATES_PROXY_SERVER=

#Proxy Server Port
SOFTWARE_UPDATES_PROXY_PORT=

#Proxy Server Username
SOFTWARE_UPDATES_PROXY_USER=

#Proxy Server Password
SOFTWARE_UPDATES_PROXY_PASSWORD=

#The oracle home location. This can be an existing Oracle Home or a new Oracle Home
ORACLE_HOME=/data/fmw/12.2.1.4
 
#Set this variable value to the Installation Type selected. 
#e.g. Fusion Middleware Infrastructure, Fusion Middleware Infrastructure With Examples.
INSTALL_TYPE=Fusion Middleware Infrastructure
 
#Provide the My Oracle Support Username. If you wish to ignore Oracle Configuration Manager
#configuration provide empty string for user name.
MYORACLESUPPORT_USERNAME=
 
#Provide the My Oracle Support Password
MYORACLESUPPORT_PASSWORD=
 
#Set this to true if you wish to decline the security updates. 
#Setting this to true and providing empty string for
#My Oracle Support username will ignore the Oracle Configuration Manager configuration
DECLINE_SECURITY_UPDATES=true
 
#Set this to true if My Oracle Support Password is specified
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
 
#Provide the Proxy Host
PROXY_HOST=
 
#Provide the Proxy Port
PROXY_PORT=
 
#Provide the Proxy Username
PROXY_USER=
 
#Provide the Proxy Password
PROXY_PWD=
 
#Type String (URL format) Indicates the OCM Repeater URL 
#which should be of the format [scheme[Http/Https]]://[repeater host]:[repeater port]
COLLECTOR_SUPPORTHUB_URL=
```

Dosyalarım hazır, kurulum paketim hazır. -silent değişkeni ile wizard olmadan kurulum yapacağımızı belirttik. -responseFile ile ihtiyaç duyulan değişken değerlerini şu dosyamdan oku dedik ve son olarak da Oracle ürün bilgilerimizi tutan oraInst.loc dosyasını da -invPtrLoc ile verdik. 

java -jar ile kurulumu başlatıyoruz.

```bash
/data/java/latest/bin/java -jar \
	/data/installations/oracle_installations/fmw_12.2.1.4.0_infrastructure.jar -silent \
	-responseFile /data/installations/oracle_installations/fmw-inf-12.2.1.4.rsp \
	-invPtrLoc /data/installations/oracle_installations/oraInst.loc
```
### OUTPUT

Görüldüğü gibi kurulum başarıyla tamamlandı. 
```text
Copyright (c) 1996, 2019, Oracle and/or its affiliates. All rights reserved.
Reading response file..
Skipping Software Updates
Starting check : CertifiedVersions
Expected result: One of oracle-6, oracle-7, redhat-7, redhat-6, SuSE-11, SuSE-12, SuSE-15
Actual Result: redhat-33
Check complete. The overall result of this check is: Passed
CertifiedVersions Check: Success.


Starting check : CheckJDKVersion
Expected result: 1.8.0_191
Actual Result: 1.8.0_281
Check complete. The overall result of this check is: Passed
CheckJDKVersion Check: Success.


Validations are enabled for this session.
Verifying data
Copying Files
Percent Complete : 10
Percent Complete : 20
Percent Complete : 30
Percent Complete : 40
Percent Complete : 50
Percent Complete : 60
Percent Complete : 70
Percent Complete : 80
Percent Complete : 90
Percent Complete : 100
```

The installation of Oracle Fusion Middleware 12c Infrastructure 12.2.1.4.0 completed successfully.
Logs successfully copied to /data/fmw/oraInventory/logs.
İyi günler olsun.

