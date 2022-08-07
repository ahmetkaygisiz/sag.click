---
title: "Script - Bulktelnet"
date: 2020-07-24T16:29:53+03:00
draft: true
categories : [ "linux" ]
tags : [
    "bash",
    "script",
    "linux",
]
---
<p>
Merhabalar,<br>
Bugün bir üşengeçliğimin sonucunda ihtiyacı gidermek adına yazmış olduğum küçük bir scriptten bahsedeceğim. Bir önceki işimde ~20 sunucu üzerinden 25 server:port kontrol etmem istediğinde ya title'ımı clickbot olarak değiştirecektim ya da olması gerektiği gibi bunu bir scripte yaptıracaktım. Script yazmak hep hoşuma gitmiştir. Linux sağolsun çok az satırda işinizi hemen pratikleştirecek bir yol bulabiliyorsunuz. Windows bu yüzden tam bir cehennem. Command Prompt işinizi kolaylaştırmak yerine daha ne kadar zorlaştırırım der gibi. Neyse sektör tarafından kabul görecek kadar Linux övdüysem scripte geçiyorum.
</p>

```sh
#!/bin/bash

function readFiles()
{ 	
	_fileName=$i
	
	echo "#### $_fileName ####" >> connected_$_hostName.txt
	echo "#### $_fileName ####" >> failed_$_hostName.txt

	while IFS= read -r line; do

		_host=$(echo $line | tr ':' ' ' | awk '{print $1}')
		_port=$(echo $line | tr ':' ' ' | awk '{print $2}')

		_str=`echo exit | timeout 2s telnet $_host $_port`
		
		if [[ $_str == *Connected* ]] ; then
			echo $line >> connected_$_hostName.txt
	        else
			echo $line >> failed_$_hostName.txt
   		fi	

	done < $_fileName

	echo " " >> connected_$_hostName.txt
	echo " " >> failed_$_hostName.txt	
}

_hostName=$(hostname)
_fileList=${*}
touch connected_$_hostName.txt failed_$_hostName.txt

for i in $_fileList
do
  readFiles $i
done

```

Scritimiz server:port şeklinde verilmiş basit bir txt dosyasını alarak satır satır telnet bağlantısı deniyor. Dosya formatı, scritin özelleştirilmesi size kalmış.


Script ```sh ./bulkConnection.sh file_1.txt ... file_n.txt ```şeklinde yürütülüyor. 

Scripte giriş yaptıktan sonra bulunduğun sunucunun ismini $_hostname değişkenine atıyor.

Verilen dosyaları parametre olarak kabul ediyor.

Hangi bağlantı denemelerinin başarılı olup, hangilerinin başarısız olduğunu tutmak için 2 adet dosya oluşturuyor.

Her bir parametre dosyası için readFiles fonksiyonu çağırılıyor.

while içerisinde host ve port değişkenleri awk ile alınıyor.

Asıl macik'in yapıldığı satırda telnet bağlantısına timeout olarak 2 saniye veriliyor. echo exit ile bu işlemden çıkılıyor ve telnet'in verdiğin console outputu _str değişkenine atanıyor.
```sh
_str=`echo exit | timeout 2s telnet $_host $_port`
```
Bu değişken sıradan bir telnet bağlantısının bağlandığında verdiği *connected* ifadesini içeriyorsa, bu başarılı bir bağlantı olarak, connected_hostname.txt dosyasına bu host:port bilgileri yazılıyor.
Bu küçük fakat beni bir clickbottan ayıran script, tamamiyle bu kadar.

Sağlıklı günleriniz olsun.

