---
title: "Except Ssh"
date: 2020-09-03T17:18:26+03:00
draft: false
categories : [
    "linux",
]
tags : [
    "linux",
    "expect",
    "scp",
    "ssh"
]
---
Selamlar, 

Bugün size yine geçmişte  yapmış olduğum bir scriptten bahsedeceğim. İhtiyaçlar doğukça yeni fikirler geldiğinden ve VirtualBox'daki VM'imin herhangi bir ihtiyacı olmadığından geçmişten devam ediyorum :)

*Expect komutunun kurumunu şuradaki medium yazımda anlatmıştım.

Expect Komutu ile Remote Makinelerde Script Yürütülmesi
```bash
#!/bin/bash
_prompt=":|#|\\\$"

_HOST=$1
_USER=$2
_PASS=$3
_SCRIPT=$4

_nohupScript="bash $_SCRIPT > /dev/null 2>&1 &"

expect -c "
	spawn ssh $_USER@$_HOST $_nohupScript
        expect {
		      "*asswor*" { 
			   send $_PASS\r
			   exp_continue
			   interact -o -nobuffer -re $_prompt return
		       send "exit"\r
		       interact
		       exit
		      }
	      }
"
```
Bir dosya güncellemesinden sonra uygulama sunucularının yeniden başlatılması gerektiği için bu scripti hazırlamıştım. Scriptte parametreler :

- $1 : Bağlanılacak olan adres
- $2 : Kullanıcı adı
- $3 : Parola
- $4 : O host üzerinde bulunan restart scriptinin absolute path'i

Kendi yazdığınız ana makinenizde bulunan bir scripti de yürütebilirsiniz fakat, bu senaryoda sunucuların sistem parametrelerinin değişken ve kendine özgü olması sebebiyle o makine üzerindeki restart scriptlerinin yürütülmesi gerekliydi. Bu script bir WLST ( WebLogic Scripting Tool ) scripti içerisinden while döngüsü içerisinden parametreleri gönderilerek çağrılıyordu. 

expect komutu, bilgisayarla etkileşim gerektiren adından da anlaşıldığı gibi bilgisayarın bizden birşey beklediği durumlar için kullanılan ve otomasyonlar için çok kullanışlı olan bir komut. expect uzantılı dosyalar ile bir bash scripti içerisinden bu komutu yürütebileceğimiz gibi, -c parametresiyle doğrudan komut satırından gerekli işlemleri de yapabiliyoruz.

spawn, expect komutunun yürütüp beklemeye almasını istediğiniz komutu verdiğiniz kısımdır. spawn komutu, verdiğiniz komutu yürütüp yeni bir process'i bekler yani burada spawn'dan sonra gelen expect { . . . } komutu içerisindeki olası senaryoların oluşmasını ve buradan dönecek cevabı yürütmeyi bekler.

expect { . . . } alanındaki *asswor* kısmı spawn'da yürütülen komut içerisinde bulunuyorsa curlybraces içerisindeki komutları yürütmeye  başlar. Burada birden fazla işlem vermemiz de mümkün. Mesela ssh ile ilk bağlantıda known_hosts dosyasına eklenip eklenmemesini isteme senaryosundaki aşağıdaki gibi çoklu bir expect'i switch-case yapısı benzerliğinde kullanabilirsiniz. 

```bash
expect {
    "*yes*" { send "yes"\r }
    exit
}	
expect {
	"*asswor*" {  send $_PASS\r; exp_continue} 
}
```

*asswor*" { . . . } bloğu içerisinde, öncelikle parola bilgisini gönderiyoruz. exp_continue ile ssh'ın bağlantısının yapılmasına izin veriyoruz. interact ile karşıdaki sunucudaki command promt ile iletişime geçip sunucudan çıkış için exit komutunu gönderiyoruz.

Expect Komutu ile Bulk SCP 
Dosya aktarımını tek tek yapmak gereksiz bir şekilde uzun sürdüğü için script ile yapmak daha mantıklı geldi ve aşağıdaki gibi bir script daha hazırladım.

```bash
#!/bin/bash
_hostFile=$1

function extract_hosts() {
    cat $_hostFile | awk -F ',' '(NF && !/^($|#)/) {print $1, $2, $3, $4, $5}'
}

while IFS=' ' read _HOST _USER _PASS _FILE _DEST
do		
	expect -c "
		set timeout 2
		log_file bulkScp.log    
		
		spawn scp $_FILE $_USER@$_HOST:$_DEST
			expect {
				"*yes*" { send "yes"\r }
				exit
			}	
			expect {
				"*pass*" {  send $_PASS\r; exp_continue} 
			}
		"
done < <(extract_hosts)
```

Host bilgilerini içeren bir adet csv dosyasından aşağıdaki column'lar parse ediliyor ve while içerisinde satır satır expect komutuna gönderiliyor.

$1 : HOSTNAME
$2 : USER
$3 : PASSWORD
$4 : Gönderilecek dosya
$5 : Remote makinede gönderilecek absolute path adresi

expect komutunda spawn ile beklettiğimiz process'e bir timeout değerini de set timeout komutu ile verebiliyoruz. Dilersek, log_file ile işlemlerin bir log dosyasına eklenmesini de sağlayabiliyoruz.

spawn komutu ile scp komutunu başlatıp beklemeye alıyoruz ve ilk defa girilen makinelerde known_hosts dosyasına ekleme yapılması için *yes* değerini gönderiyoruz. Parola sorulduğunda ise parolayı gönderip dosya aktarımının tamamlanmasını sağlıyoruz.

Şimdilik bu scriptler ile aktaracaklarım bu kadar. expect komutu oldukça geniş ve araştırmaya denemeye ihtiyaç duyan bir komut. Eğer bu scriptleri kullanmayı planlıyorsanız da öncelikle test ortamında denemenizi tavsiye ederim.

Hoşçakalın.


