---
title: "Container DB ve RCU"
date: 2022-02-20T18:38:43+03:00
draft: true
---
Merhabalar,

Oracle'ın SOA-OSB gibi ortamlarının kurulum aşamalarından biri olan RCU (Repository Creation Utility) adımı için db bağlantısını yapmak ve can sıkıcı/heves kırıcı Oracle DB kurulumunu en hafif yarayla atlatabilmek için bu yazıyı yazıyorum. Daha önce Oracle DB kurulumu yapmış olsam da bir sürü zincirleme sebepten Ubuntu ve Fedora üzerinde kurulum yaparken problem yaşadım. Bu noktada büyük bir nimet olan Docker yardımıma koştu. Her ne kadar container db'ler ile çalışmamış olsam da hem yeni bir yol bulmak hem de Oracle Support'taki "tek ayağını kaldırarak mouse'u diğer elinle kullanıp kuruluma başlarsan oluyor" çözümlerinden kurtulmak benim için güzel oldu. 

Bir önceki yazımda FMW kurulumu yapmıştık. Şimdi de RCU ile DB bağlantı adımına geçeceğiz. Bu kurulum aşamalarını parça parça yapmamın nedeni SOA/OSB ya da benzer teknolojiler için ortak kurulum adımları olması. Tekrara düşmemek için parça parça ilerlemek istedim. Haydi geçelim şimdi kuruluma.

### Oracle DB Image Çekilmesi
hub.docker.com adresine gidip oracle xe $version diye arattığımızda containerize edilmiş birden fazla kullanıma hazır image karşımıza çıkıyor. Burada ana container'ı çekip üzerinde değişiklik yapıp yeniden kendi etiketinizle image'i yükleyebiliyorsunuz. Ben de bu imagelerden [bir tanesini](https://hub.docker.com/r/pvargacl/oracle-xe-18.4.0) buldum. Sayfaya gittğimizde zaten bize 2 adımda bu image'i ayağa kaldırabileceğimiz söylenmiş.

Öncelikle pull adımını gerçekleştiriyoruz.  Bu pull adımı uzun sürebilir çünkü bu image boyutu 12.7 GB.
```bash
docker pull pvargacl/oracle-xe-18.4.0:latest
```
Image tamamen çekildikten sonra bir kontrol ediyorum.
```bash
☁  ~  docker images                               
REPOSITORY                  TAG       IMAGE ID       CREATED         SIZE
pvargacl/oracle-xe-18.4.0   latest    9f03e56c9bd3   13 months ago   12.7GB
```
Şimdi de bu image'i çalışan bir container haline getirmemiz gerekiyor. Onu da docker run ile çalıştırıyoruz. Yaklaşık bir dakika sonra da docker ps ile sağlıklı şekilde ayağa kalkıp kalkmadığını kontrol ediyoruz.
```bash
docker run --name oracle18 -d -p 1521:1521 pvargacl/oracle-xe-18.4.0
☁  ~  docker ps
CONTAINER ID   IMAGE                       COMMAND                  CREATED              STATUS                        PORTS                                                 NAMES
a78fb39a9002   pvargacl/oracle-xe-18.4.0   "/bin/sh -c 'exec $O…"   About a minute ago   Up About a minute (healthy)   0.0.0.0:1521->1521/tcp, :::1521->1521/tcp, 5500/tcp   oracle18
```
Bu image'i hazırlayan arkadaşımız parolaları yönetebilmemiz için bir script hazırlamış. Bunu doğrudan çalıştırdığımızda sysdba vb. roller için parolayı değiştirmiş oluyoruz. Ben de sadece development ortamında kullanacağım ve kritik bir bilgi olmadığı için basit bir parola set ediyorum.
```bash
#docker exec <container name> ./setPassword.sh <your password>
docker exec oracle18 ./setPassword.sh Welcome1
```
Parolayı değiştirdik, bu script neredeydi nasıl çalıştı onu da bir kontrol edelim. Container içerisine aşağıdaki komut ile terminaline düşelim.
```bash
docker container exec -it oracle18 sh
```
Sonrasında neredeyim hangi dosyalar var diye kontrol ediyorum. Aşağıda görüldüğü gibi kök dizine giriş yaptık ve burada setPassword scripti link olarak verilmiş. Yani bir üst adımda parametre olarak verdiğimiz değer burada setPassword'e verildi ve DB'de parolalar güncellenmiş oldu.
```bash
sh-4.2# pwd
/
sh-4.2# ls -lrta
total 24
...
lrwxrwxrwx.   1 root root   26 Apr  8  2020 setPassword.sh -> /opt/oracle/setPassword.sh
lrwxrwxrwx.   1 root root   19 Apr  8  2020 docker-entrypoint-initdb.d -> /opt/oracle/scripts
dr-xr-x---.   1 root root   16 Apr  8  2020 root
-rwxr-xr-x.   1 root root    0 May 22 08:18 .dockerenv
drwxr-xr-x.   1 root root 2508 May 22 08:18 etc
...
```
Hazır container içerisindeyiz veritabanına da bir bağlanalım.
```bash
sh-4.2# sqlplus sys/Welcome1@//localhost:1521/XE as sysdba

SQL*Plus: Release 18.0.0.0.0 - Production on Sat May 22 08:35:54 2021
Version 18.4.0.0.0

Copyright (c) 1982, 2018, Oracle.  All rights reserved.


Connected to:
Oracle Database 18c Express Edition Release 18.0.0.0.0 - Production
Version 18.4.0.0.0

SQL>
```
### Container Databases (CDB) and Pluggable Databases (PDB)
Her şey güzel veritabanımızı oluşturduk ve bağlandık. E tamam o zaman hemen RCU çalıştırıp işimi halledeyim dedim cahil ben. Bir iki denemeden sonra baktım olmuyor başlığını yazmış olduğum 2 kavram ile karşılaştım. Container Databases ve Pluggable Databases.

Container Database: Çoklu ve daha geniş yapıları desteleyen, içerisinde birden fazla pluggable database gibi yapıları içerebilen, sanal veritabanları oluşturabileceğiniz veritabanı sistemi diyerek anladığım kadarıyla tanımlıyorum.

Pluggable Database: Aslında kullanıcı tarafından bakıldığında normal bir veritabanı gibi gözüken fakat Container DB'ler içerisinde yaşamını sürdüren veritabanı yapısı. Pluggable DB oluşturduğunuzda normal kurulum yapıldığındaki veritabanı sisteminden farklı gözükmüyor. 

RCU bizden Pluggable Database istediği için container db'miz içerisinde bir PDB oluşturmamız gerekiyor. DBBeaver kullanıyorum ben db bağlantıları SQL Editor vs için. Aşağıdaki bilgilerle bağlantı oluşturdum.

- Host/Port : localhost 1521
- Service Name : XE
- Auth : sys as sysdba 
- Pass : Welcome1

PDB oluşturmamız gerekiyor. Aşağıdaki komut ile PDB'leri listeleyebiliyoruz.
```SQL
SELECT name, pdb FROM   v$services ORDER BY name;
```
Alttaki SQL komutu ile yeni bir PDB oluşturuyoruz. SQL success döndükten sonra, bu veritabanına da sys/Welcome1 ile giriş yapıyoruz.
```SQL
CREATE PLUGGABLE DATABASE soapdb
ADMIN USER soaadm IDENTIFIED BY Welcome1
FILE_NAME_CONVERT = ('/opt/oracle/oradata/XE/pdbseed/', 
                     '/opt/oracle/oradata/XE/soapdb/');
```
soaadm kullanıcısına SYSDBA yetkisini veriyoruz.

```SQL
GRANT SYSDBA to soaadm;
```
Veritabanı kısmında yapmamız gereken adımlar tamam şimdi sırada RCU'nun çalıştırılmasında.

### RCU
Burada yapacağım şey sadece RCU'nun çalıştırılması ve bağlantının sağlandığını göstermek. SOA için farklı OSB için farklı tablolar seçileceği için onların kurulumu ayrı ayrı yazılarda anlatırım muhtemelen.

1- RCU'u çalıştırıyoruz.
![RCU-1](/014-rcu-1.png)
2- Go go go.
![RCU-2](/014-rcu-2.png)
3- Repository oluştur diyoruz.
![RCU-3](/014-rcu-3.png)
4- PDB için oluşturduğumuz kullanıcı ve URL bilgilerini giriyoruz.
![RCU-4](/014-rcu-4.png)
5- Görüldüğü gibi bağlantı başarılı oldu. Bu aşamadan sonra da tabloların oluşturulma aşaması geliyor.
![RCU-5](/014-rcu-5.png)

### Sonuç
Bu aşamaları ben uzata uzata yazdım hem öğretmek hem de size aktarabilmek için. Biraz uğraştırıcı gibi gelmiş olabilir belki ama doğrudan Oracle DB kurulumunu denedikten sonra o kadar da problemli bir konu olmadığını görüyorsunuz.

Hevesimizin daim olduğu günlerde/yazılarda görüşmek üzere.
