---
title: "Wifi Over Terminal"
date: 2020-09-14T17:21:14+03:00
draft: false
categories : [
    "code",
]
tags : [
    "spark",
    "java",
    "rest"
]
---
Selamlar Efendim,

Bugün, bana arada sırada gelen "Nvidia Driver'ı da neymiş yauv" gazının sonucunda yine siyah ekranla kala kaldığım bir maceraya atıldım. Bu macera esnasında daha önce herhangi bir wifi ağına Terminal'den bağlanmadığımı farkettim ve nasıl bağlanabileceğimi araştırıp buraya da not etmek istedim. Şuan çalıştığım makinenin OS'i Debian 10 (buster) yazıya başlamadan bunu da belirteyim.

Buradaki kaynakta Network Manager nasıl kullanılır oldukça güzel bir şekilde anlatmış. Ben de buradaki kaynağı takip ederek uygulamaya başlıyorum.
```sh
nmcli general status
```
Bulunduğumuz cihazın bağlantı durumunu kontrol etmek için yukarıdaki komutu kullanıyoruz.
```sh
nmcli connection show
```
Bu komut ile de daha önce giriş yaptığımız kaydettiğimiz ağları listeliyoruz. 
```sh
nmcli con up $network_name
```
Eğer daha önce kaydettiğimiz ağlardan birisine bağlanmak istiyorsak, yukarıdaki komut'a bağlanacağımız ağın adını vererek bağlanabiliyoruz. Bağlantıyı ilk defa kuracak ise aşağıdaki adımları takip ediyoruz.
```sh
kaygisiz@akua:~$ nmcli device status
DEVICE           TYPE      STATE         CONNECTION      
br-f9614c6c587f  bridge    connected     br-f9614c6c587f 
wlp3s0           wifi      disconnected  --              
```
Bilgisayarımızdaki cihazların durumunu görüntülüyoruz. Buradaki Type'ı wifi olan device adını bir kenara yazıyoruz. Benim durumum için bu wlp3s0.
```sh
kaygisiz@akua:~$ nmcli dev wifi list
IN-USE  SSID               MODE   CHAN  RATE        SIGNAL  BARS  SECURI
        Akua               Infra  1     117 Mbit/s  100     ****  WPA2  
```
Bağlanabileceğimiz wifi ağlarını listeleyip, bağlanmak istediğimiz ağın SSID'sini de aklımızın bir kısmına atıyoruz. Bağlanacağımız ağı, bağlantılar listesine ekliyoruz.
```sh
# nmcli con add con-name $network_name ifname $device_name type wifi ssid $ssidName
kaygisiz@akua:~$ nmcli con add con-name Wifi_Hayrati ifname wlp3s0 type wifi ssid Akua
Connection 'Wifi_Hayrati' ($UUID) successfully added.
```

Burada Wifi_Hayrati olarak  eklediğim değer değişken. Ağı nasıl tanıtmak istiyorsanız o şekilde ekleyebilirsiniz. Babam_Sagolsun, Vefa_Sadece_Semt_Adi_Degil  vb. Ağ sahibinin sizdeki yerine göre doldurabilirsiniz. :)

```sh
# nmcli con modify $network_name wifi-sec.key-mgmt wpa-psk
kaygisiz@akua:~$ nmcli con modify Wifi_Hayrati wifi-sec.key-mgmt wpa-psk
# nmcli con modify $network_name wifi-sec.psk $password
kaygisiz@akua:~$ nmcli con modify Wifi_Hayrati wifi-sec.psk $password
```
Burada ağın koruma şeklinin WPA2 olması durumuna göre tanımlamayı yapıp parolamızı ekliyoruz. Artık ağ bilgilerini kaydettik, sırada bağlanmak var. 
```sh
# nmcli con up $network_name
kaygisiz@akua:~$ nmcli con up Wifi_Hayrati
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/8)
```
Son olarak bağlantıya up diyerek mutlu sona ulaşıyoruz.

CLI güzel hoş fakat GUI için şükretmek gerekiyor. Bol öğrenmeli günlerde görüşmek üzere.
