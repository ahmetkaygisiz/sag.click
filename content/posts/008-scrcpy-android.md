---
title: "Scrcpy Android"
date: 2020-11-30T17:25:35+03:00
draft: false
categories : [
    "linux",
]
tags : [
    "linux",
    "crontab",
    "bash"
]
---
Merhaba,

Bu yazıda android telefonu linux üzerinden yönetebilmemizi sağlayan bir uygulamadan bahsedeceğim. Şerrefsizim benim aklıma gelmişti tarzında fakat aklımızdakinden de fazlasını bulduğumuz bir uygulama çünkü proje sahibi Genymobile :) Android'de "Hello World!" yazan herkesin yüksek ihtimal duymuş olduğunu düşünüyorum. Bu uygulama da size bir emülatör gibi kendi cihazınızı desktop üzerinden yönetmenizi sağlıyor.

Uygulamayı yüklemenin birden fazla yolu var. snap ile yüklemek bunlardan en kolayı. Mac/Windows kullananlar veya yok ben kaynağından kullanırım kolaya kaçmam diyenler için de buyrun [link](https://github.com/Genymobile/scrcpy/blob/master/BUILD.md) . Gerekli kurulum işlemleri adımları belirtilmiş. Ben önce snap ile yüklemeyi sonra da doğrudan repo üzerinden nasıl kurulur onu anlatmak istiyorum. 

Snap ile scrcpy Kurulumu
```bash
sudo snap install scrcpy
```
Yukarıdaki komut ile scrcpy'i yükledik. Telefonunuzu usb kablosu ile bilgisayara bağladık. Uygulamayı çalıştırmak için telefonunuzun Developer Options'ı açık olmalı ve o menü altında USB Debugging seçeneğini de açık olmalıdır. Uygulamayı başlatmak için aşağıdaki komutu kullanıyoruz.
```bash
snap run scrcpy
```
Uygulamayı başlattığınızda, telefonunuz sizden bağlantı için onay vermenizi isteyecek. Onayı verdikten sonra telefonunuzu desktop üzerinden yönetebilir hale geliyorsunuz.

Repo ile Kurulum
Bu aşamada scrcpy'nin github reposunu bilgisayarımıza çekip sonra bu projeyi build ederek kurulumu tamamlayacağız. Benim bu seçenek üzerinden gitme sebeplerimden birisi de wireless üzerinden de bağlantı sağlayabilmemizi sağlayan gerekli araçları da yükleyip ortak ağ üzerinden kabloya gerek kalmadan ekran paylaşımı yapabilmek. 
```bash
# bağımlılıkların yüklenmesi
sudo apt install ffmpeg libsdl2-2.0-0 adb

# istemci build bağımlıkları
sudo apt install gcc git pkg-config meson ninja-build \
                 libavcodec-dev libavformat-dev libavutil-dev \
                 libsdl2-dev

# server build bağımlılığı JDK
sudo apt install openjdk-8-jdk
```
Yukarıdaki bağımlılıkları kurduk. Şimdi geldi Android SDK ve CommandLine Tool'larının kurulumuna. Şu [link](https://developer.android.com/studio/index.html) içerisinden altlarda bulunan Command Line Tools başlığı altındaki zip dosyalarından işletim sistemimize uygun olanını indiriyoruz. Android Debugging Bridge (ADB) isimli komut için de şu [adresten](https://developer.android.com/studio/releases/platform-tools.html) gerekli paketi indiriyoruz. 

Bu indirilen dosyaları installation path'inize extract ediyoruz. Örneğin /data/installation/AndroidSDK pathi içerisine ben iki dosyayı çıkarttım. Kurulum öncesinde ANDROID_SDK_ROOT pathini terminal'e tanımlıyoruz. Ben zsh shell'ini kullandığım için aşağıdaki komut ile .zshrc dosyamı açıyorum. Eğer bash kullanıyorsanız ..bashrc dosyasına ortam değişkenini ekleyebilirsiniz.
```bash
export JAVA_HOME=/data/java/latest
export ANDROID_SDK_ROOT="/data/installations/AndroidSDK"
export PATH=$JAVA_HOME/bin:$ANDROID_SDK_ROOT/platform-tools/:$PATH
```
Sistem değişkenlerini tanımlayıp çıkış yaptıktan sonra aşağıdaki şekilde kontrolleri yapmamız faydalı olur. 
```bash
echo $ANDROID_SDK_ROOT
echo $JAVA_HOME
```
Burada build ile ilgili aldığım license hataları sebebiyle aşağıdaki gibi cmdline-tools klasörüne girip sdkmanager ile gerekli lisansların yüklenmesini sağlıyoruz.
```bash
cd /data/installations/AndroidSDK/cmdline-tools/bin/
./sdkmanager --licenses --sdk_root=$ANDROID_SDK_ROOT
```
Projeyi Github'dan çekiyoruz.
```bash
git clone https://github.com/Genymobile/scrcpy && cd scrcpy
```
ve build ediyoruz.
```bash
meson x --buildtype release --strip -Db_lto=true
ninja -Cx
```
BUILD SUCCESS mesajını görene kadar sevinmiyoruz. Build tamamlandığında son olarak scrcpy komutunu yüklüyoruz.
```bash
sudo ninja -Cx install
```
ve ta daaa. Uygulamayı başlatmak için komut satırına aşağıdaki komutu yazıyoruz ve telefonumuzun ekranı ile karşılaşıyoruz.
```bash
scrcpy
```
scrcpy ile Kablosuz Bağlantı
Telefonunuz ile ortak bir ağa bağlandığınızda ekran paylaşımı yapmak için aşağıdaki adımları takip etmeniz yeterli.

0 - Başlangıçta telefonuz bilgisayara usb kablosuyla takılı olmalı<br>
1 - Telefonunuzun IP adresini kontrol edin. (Settings > About Phone > IP Address<br>
2 - Bu komutu yazarak ADB TCPIP portunu belirtin :<br>
```bash
    adb tcpip 5555
```
3- Telefonunuzun IP'sini aşağıdaki $DEVICE_IP kısmına yazın.
```bash
adb connect $DEVICE_IP:5555
# connected to $DEVICE_IP:5555
```
4- Connected ifadesini gördükten sonra usb bağlantısını çıkartıyoruz.
5- ```bash scrcpy``` komutunu giriyoruz.

Kablosuz bağlantı da bu şekilde. Kablosuz bağlantıda bazı gecikmeler olabiliyor normal olarak. Bit hızı ve çözünürlük düşürülerek daha hızlı kullanmak, gecikme yaşamaktan daha iyi bir seçenek olduğu için aşağıdaki komut ile scrcpy'i başlatabilirsiniz.
```bash
scrcpy --bit-rate 2M --max-size 800
```
Bu yazıda benden bu kadar. 
Görüşmek üzere.

Kaynaklar : 
https://github.com/Genymobile/scrcpy/blob/master/BUILD.md
https://www.genymotion.com/blog/open-source-project-scrcpy-now-works-wirelessly