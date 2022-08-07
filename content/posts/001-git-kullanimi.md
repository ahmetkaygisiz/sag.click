---
title: "Git Kullanimi"
date: 2020-07-20T16:29:53+03:00
draft: false
categories : [
    "devops",
]
tags : [
    "git"
]
---
<p>Selamlar,

Gökkubbe altında söylenmemiş söz, yazılmamış blog konusu yoktur ama kimse hikayeyi benden dinlemedi* diyerek giriş yapıyorum (*Üsküdar'a Giderken). Genel amacım kendime notlar toparlamak olduğu için eşsiz bir bloga sahip olacağım Bil Geyts, Zukırberk gelip benden alacak teknolojideki güncel haberleri gibi bir iddiam olmadığını beyan ederek başlıyorum ilk yazıma.

Versiyon kontrol sistemi, backend'den fronend'e kod yazıyorum diyen herkesin kullandığı bir sistem. Herhangi bir yazı yazarken, tez hazırlarken 'SON', 'BU SON', 'SON_SON_SON', 'BU DA MI SON DEGIL' isimli dosyaları nereye koyacağınızı şaşırıyorsanız sizin de kesinlikle tanışmanız gereken bir sistem diye düşünüyorum.

Öncelikle konfigürasyon ile işe başlayalım. Remote repomuzu github/bitbutket gibi ortamlarda oluşturduysak hazır ise konfigürasyon yaparak başlıyoruz. </p>

{{< highlight sh >}}
git config --global user.name   "$kullanici_adi"
git config --global user.email  "$mail_adresi"
{{< /highlight  >}}

<p>
Localdeki git'imize hangi depoya gidersen bu giriş bilgilerini kullan komutunu vermiş olduk. Repo olarak kullanacağımız dosyanın içerisine girdikten sonra aşağıdaki komutu çalıştırarak burayı artık bir depo olarak ilan ediyoruz/git'e bildiriyoruz.
</p>

```sh
git init    // Konfigürasyon dosyalarını oluşturup klasörü bir git dosyası haline getirmek için
```

Bu komuttan sonra bulunduğumuz dizinde bir .git klasörü oluşuyor ve içerisinde de konfigürasyon dosyaları bulunuyor. Bunlar local deponuz ve remote deponuz arasında eşitliği sağlamak, branchları logları vb. bilgileri tutmaya yarıyor.

Depomuza ilk dosyayı aşağıdaki şekilde ekliyoruz.
```sh
git add <eklenecek_dosya_adi> // adı belirtilen dosyayı ekle
git add *      // klasörün içerisindeki tüm dosyaları ekle
```
Ben giderim commit'im kalır juniorlar beni hatırlasın diyerek yorumumuzu, açıklamamızı yazıyoruz.
```sh
git commit -m "first commit" // yapılan değişikliği belirtilent
```
Projemiz local'de ortamlara gönderilmeyi bekliyor. Uzak depomuzun adresini ekliyoruz :

```sh
git remote add origin https://github.com/ahmetkaygisiz/tutorial_git.git // remote repoya adresini ekle
```

Eğer farklı bir branch oluşturmadıysanız, default olarak master branch'ı oluşturuluyor. Adresi verdikten sonra kodlarımızı depomuza gönderiyoruz.
```sh
git push -u origin master // değişiklikleri remote repoya gönder
```
Buraya kadar olan herşey çok kolay olsa da bu depoların genel amacı bir projede birden fazla kişinin çalışabilmesini sağlamak olduğu için işin içerisine insan (hele de benim gibi juniorlar) girdiğinde ortalık bir miktar karışabiliyor.

Bu gibi projelerde çalışırken, çalışmaya başlamadan önce yapmanız ilk gereken şey güncel repoyu almak/çekmek olmalıdır. Aksi takdirde sizden önce çalışan birisinin değişikliklerini almazsınız ve siz de yeni eklemeler yaparsanız ortaya karışıklıklar çıkar.

```sh
git pull origin <branch_adi> // Remote repodaki değişiklikleri al
git pull origin master
```

Çalışmalarınızı tamamladınız, tam o sırada çılgın bir fikir geldi aklınıza. Dediniz ben balkon duvarını kırıp mutfağa katacağım... Git'in sunduğu avantajlardan biri de size bunu yapmadan önce farklı bir dal sunması, yani sen gel önce şurda çalış evi kirletme, güzel olursa evde de aynısını yaparız diyor. Bunun için bir branch oluşturuyoruz.

```sh
git checkout -b <yeni_branch> <kaynak_Branch>
git checkout -b balkon_to_mutfaga master
```

Tüm branchları listeleyebilmek için aşağıdaki komutları kullamabiliriz.

```sh
git branch -a   // all - Local ve Remote branchlar
git branch -r    // remote - remote branchlar
git show-branch    // commitler ile birlikte gösterir
```
Branchlar arası 'daldan dala, daldan dala' aşağıdaki komut ile mümkün.

```sh
git checkout <branch_adi>
git checkout master
```

Commit kayıtlarını,geçmişini görüntülemek için git log kullanılabilir.
```sh
git log --oneline // Tek satırda commit historysini göster
```
Commit ID ile de yeni bir branch oluşturabilmemiz mümkün:
```sh
git branch <branch_name> <commit_id> // Yapılan değişiklikler ile yeni bir branch oluştur.
```
Aktif çalıştığımız dizin içerisinde durum nedir diye kontrol etmek için:
```sh
git status
```
Yaptığınız değişiklikler hoşunuza gitmediyse, localde yaptığınız değişiklikleri remote repoya göndermek istemiyorsanız commit etmeden geri almak aşağıdaki komut ile mümkün.
```sh
git reset --hard HEAD // En son yapılan değişiklikleri geri al.
```
Sadece bir dosyada yaptığınız değişikliği geri almak için :
```sh
git checkout -- dosya1.md // En son commit edilmiş haline geri dön.
```

balkon_to_mutfak branchında yaptığınız değişiklikler hoşunuza gitti ve evinize de uygulamak istiyorsunuz. Ana branch'ı ve yan branch'ı birleştirmek için önce birleştirilecek/ezilecek branch'a geçiş yapıyoruz. Sonrasında birleştirmek istediğimiz branch ismi ile değişikliği ana branch'a uyguluyoruz.

```sh
git checkout master
git merge balkon_to_mutfaga
```
Eski branch'a artık ihtiyacımız yok, ondan da şu şekil kurtulabiliriz.

```sh
git branch -d balkon_to_mutfaga              // localde branchı sil
git push origin --delete balkon_to_mutfaga  // remoteda branchı sil.
```

Şimdilik aktaracaklarım bu kadar, eklemeler çıkarmalar olacaktır.
Güzel ve sağlıklı günler dilerim.