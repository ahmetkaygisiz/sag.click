---
title: "Crontab Price Alarm"
date: 2020-10-31T17:25:35+03:00
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
Selamlar,

Geçen günlerde hepsiburada'da bir ürün almak için belirli indirim günlerinden birini bekliyordum. Ürünün indirime girip girmediğini kontrol etmek çok zor olmasa da bunu nasıl script ile yapabilirim diye düşündüm :D. Yani sorarsanız çok gerekli miydi diye değildi. Fakat önemli olan yolda olmak dedim (çok fazla TEDX'e maruz kaldım) ve bir script hazırlamaya karar verdim. Bu script, crontab'da 1 saatte bir çalıştırılacak ve ürün indirime girdiyse bunun bildirimini bana desktop notification olarak verecekti. 

İlk olarak yapmam gereken şey ilgili ürünün url'ini curl ile çağırıp ürün fiyatını parse etmekti. Ürünün fiyatının bulunduğu html tag class'ını browser'da inspect (ctrl+shift+c) ile aldım ve bir iki deneme yanılma ile kesmem gereken kısmı yani fiyatı aşağıdaki şekilde aldım.

```bash
curl -X GET $_URL | grep "class=\"extra-discount-price\"" | cut -d'>' -f3 | cut -d ',' -f1
```
Fiyatı aldıktan sonra bunu bir değişkene atadım. Bir if koşulu ile gelen değer istediğim fiyattan düşük ise bunu bana bildir dedim.
```bash
if [ $VALUE -lt 300 ]
then
	/usr/bin/notify-send  "Product discount" 
fi
```
Buraya kadar her şey güzel gitmişti çok basit oldu bu iş dedim. Fakat küçük bir problem çıktı. Script kendim yürüttüğümde normal bir şekilde çalışırken, crontab içerisinde notification bildirimini göndermiyordu. Çeşitli yerlere txt dosyalarına çıktı basması için "Burası çalışıyor.", "Burası da çalışıyor." echolar yerleştirerek profesyonel bir şekilde debug yaptım :) Kısa bir gugıl sörç sonrası notify-send komutunun gönderilmesi için DBUS_SESSION_BUS_ADDRESS isimli değişkenin tanımlanması gerektiğini öğrendim. 

Bu scripti generic yapmak için de araştırma yaptım fakat fiyat tespiti için pratik bir çözüm bulamadım. Daha iyisi mutlaka vardır. Scriptin son hali şu şekilde : 
```bash
#!/bin/bash

# For notification problem in crontab 
# https://askubuntu.com/questions/298608/notify-send-doesnt-work-from-crontab
eval "export $(egrep -z DBUS_SESSION_BUS_ADDRESS /proc/$(pgrep -u $LOGNAME plasma)/environ)";

# Url to parse
_URL="https://www.hepsiburada.com/hp-1kf75aa-omen-by-hp-600-12-000-dpi-oyuncu-mouse-p-HBV000008HST3"

# Get price
VALUE=`curl -X GET $_URL | grep "class=\"extra-discount-price\"" | cut -d'>' -f3 | cut -d ',' -f1`

if [ $VALUE -lt 300 ]
then
	/usr/bin/notify-send  "Product discount" 
fi
```
Crontab : 
```bash
0 * * * * DISPLAY=:0 /data/scripts/crontab_price/check_price.sh
```

Bunun ile ilgili farklı fikirler de geliştirilebilir. Örneğin kariyer.net'de belirli bir iş listesi parse edilip txt dosyasına kaydedilerek günlük olarak karşılaştırılması yapılıp, yeni bir fırsat var mı yok mu kontrol edilebilir. Eğer onu da kurcalama hevesi gelirse buraya bir ekleme yaparım.

Bol fikirli günler dilerim.