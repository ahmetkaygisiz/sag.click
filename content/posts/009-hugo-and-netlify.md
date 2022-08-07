---
title: "Hugo and Netlify"
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
Selamlar,

Web sitelerinde genel kullanım çoğunlukla kullanıcılarla etkileşim gerektiren işler oluyor. Fakat bu etkileşime gereksinim duymadığımız Blog/Portfolio/CV gibi sadece bizim değişiklik yapacağımız, boyutları oldukça küçük web sitelerine de ihtiyacımız oluyor. Bu küçük boyutlu web siteleri için hosting ücreti ödemek ise son derece gereksiz. Bizlere ücretsiz statik web siteleri hostingi sunan servislerden bazıları şunlar : Github Pages, Netlify, Firebase vb. Bu yazımda henüz tanışmamış ve bir şekilde yolu benim bloguma düşmüş olanlara Hugo & Netlify'dan bahsedeceğim.

HUGO
Hugo; golang dili ile yazılmış bir statik web site oluşturmaya yarayan bir frameworktür. Kendilerinin en hızlı web site oluşturucu olduklarını iddia ediyorlar. Golang ile yazılmış olması ve [şuradaki](https://gohugo.io/about/what-is-hugo/#how-fast-is-hugo) benchmark videosu bizlere inanmamak için pek bir sebep bırakmıyor. Hızlı bir şekilde hugo kurulumuna başlayalım. Ben debian tabanlı bir işletim sistemi üzerinden gideceğim. Diğer OS'ler için tüm kurulum talimatları için [buraki linki](https://gohugo.io/getting-started/installing/) takip edebilirsiniz. 

Hugo default olarak Debian 10'de kuruluydu fakat versiyon olarak oldukça eskiydi. Hugo versiyonu kontrolü için hugo version komutunu kullanabilirsiniz. Bu versiyonu yükseltmek için [hugo releases](https://github.com/gohugoio/hugo/releases) sayfasına gidip ilgili paketi indiriyoruz. hugo_extended_* ile başlayan bir paketi indirmenizi tavsiye ederim. Yazıya başlamadan önce kullandığım tema aşağıdaki hatayı verdiği için ben de önce normal sonra extended paketi yüklemek mecburiyetinde kaldım.

failed to transform "*.scss" (text/scss): resource "scss/styles/*.scss_a4e369375f17bb89f2019364cbe7a830" not found in file cache
İndirdiğim .deb uzantılı paketi kuruyorum.
```bash
# deb paketini yükledim.
sudo dpkg -i hugo_extended_0.79.0_Linux-64bit.deb

# versiyon kontrolü yapıyorum.
hugo version

# ve çıktıda 0.79 ifadesini görüyorum.
# Hugo Static Site Generator v0.79.0-1415EFDC/extended linux/amd64 BuildDate: 2020-11-27T09:16:17Z
```
Hugo hazır. Uygulamamızı Github üzerinden Netlify'e çekeceğimiz için Github'da boş bir repo oluşturalım. Sonra local'de aşağıdaki adımları takip edelim.
```bash
# hugo ile yeni site oluştur ve bu klasörün içerisine gir
hugo new site blog-with-hugo && $_

# git reposu haline getir.
git init
```
blog-with-hugo isimli dosyamızı web sitemizi ve bunu bir repo haline getirdik. Şimdi [themes.gohugo.io](https://themes.gohugo.io/) adresinden kendi isteğimize göre bir tema seçiyoruz. Seçtiğimiz temanın kendi Github sayfasına da gitmenizi tavsiye ediyorum temanızla ilgili ek bilgileri bulabilirsiniz. Seçtiğimiz temayı bulduğumuz git reposuna submodule olarak ekliyoruz. 
```bash
git submodule add https://github.com/jakewies/hugo-theme-codex.git themes/hugo-theme-codex
```
Sonrasında temadaki talimalatları takip ediyorum. Temada bulunan config.toml dosyasını ana dizindeki config.toml ile değiştiriyorum.
```bash
# kopyala
cp themes/hugo-theme-codex/exampleSite/config.toml .
```
```bash
# düzenle
vim config.toml
```
Dosyanın en başında bulunan themesDir = "../../" ifadesini siliyorum ve kendi bilgilerini girmek istediğim alanları değiştiriyorum. Index Page'i ekliyorum ve ilk blog postumu aşağıdaki komut ile oluşturuyorum. ilk-post sizin postunuzun başlığı oluyor.
```bash
hugo new blog/ilk-post.md
```
Değiştirmek için contents/blog/ilk-post.md dosyasına içeriği yazıyoruz.
```text
---
title: "Ilk Post"
date: 2020-11-30T12:38:56+03:00
# url
slug: "first-post"
description: "Bu Hugo'daki İlk Paylaşımım"
keywords: ["hugo","go","netlify"]
draft: false
tags: ["hugo","go","netlify"]
math: false
toc: true
---
```
```text
## The standard Lorem Ipsum passage
"Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum."
```
Son olarak hugo'yu çalıştırıyorum. Uygulama :1313 portunu kullanıyor.
```bash
hugo server -D
```
localhost:1313'e giderek siteyi kontrol ettik her şey çok güzel gözüküyor. Sırada bunu Github'a göndermek var.
```bash
git add *
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:${your-username}/${your-repo}.git
git push -u origin main
```
Netlify 
Sırada hosting adımı var. Netlify hesabınız yoksa ise https://app.netlify.com/signup adresinden kayıt olup. Panele ilerliyoruz. 

New Site from Git butonuna tıklıyoruz. Continuous Deployment seçenekleri içerisinde Github'ı ya da sizin reponuz hangi platformdaysa onu seçiyoruz. Sonraki adımda kodumuzu gönderdiğimiz repoyu seçiyoruz ve default değerler ile Deploy Site diyoruz. Eğer temanız Netlify'deki versiyonu destekliyorsa siteniz yayınlanmış oluyor. Benimki gibi bir versiyon uyuşmama durumunda aşağıdaki hatayı alacaksınız.

HUGO-THEME-CODEX theme does not support Hugo version 0.54.0. Minimum version required is 0.72.0
Bunun çözümü için Deploy Settings > Environment > Environtment Variables adımlarını takip ediyoruz. Edit Variables diyerek Key-> HUGO_VERSION Value-> 0.79.0 yazıp kaydediyoruz. Tekrar Deploys menüsüne dönüp Trigger Deploy'a basıyoruz.

Ve ta da Netlify Build Complete. Sitemiz yayında.

General > Site Settings > Change Site Name  adımlarını takip ederek sitemizin başındaki random oluşturulmuş kısmı değiştirebiliyoruz.

Sonuç : https://ahmetkaygisiz.netlify.app/

Custom Domain Name Ekleme
Netlify bize site ayarlarında ikinci adım olarak zaten Add a custom domain to your site şeklinde seçenek sunuyor. Bu seçeneğe tıklıyoruz. Benim 3-5 liraya aldığım boşta duran bir domain name'im vardı bu yazı için bunu kullanacağım. Siz de kendinize, domain name satan bir siteden bunu black friday indirimleriyle ucuza alabilirsiniz.

Netlify'de Domain Settings'e gidip yazdığımız Domain Name Options'ına tıklayarak DNS name'i ayarla diyoruz. Burada bize 4 adet nameserver adresi veriliyor. Bunları domain aldığımız siteye gidip nameservers kısmına giriyoruz. Aktifleştirme maksimum 24 saat içerisinde yapılacaktır.

Sonuç : http://ahmetkaygisiz.online

Final
Bu yazının sonuna geldik. Blogger gibi sitelerde hazır sitelerde, size esneklik tanımayan yerlerde çalışmaktansa başlangıçta biraz emek harcayarak sizin de elinizin değdiği bir siteye sahip olmak bence daha güzel. Umarım faydalı olmuştur.

Güzel günler dilerim.