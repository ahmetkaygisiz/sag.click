---
title: "Quarkus Framework Nedir"
date: 2021-01-02T17:51:31+03:00
draft: false
categories : [
    "code",
]
tags : [
    "java",
    "quarkus",
]
---

Yeni yıldan selamlar,

Bu yazıma "Seni yeneceğim 20XX!!!" enerjisiyle giriş yapıyorum. Her ne kadar bu enerji Google Anatycs verileriyle biraz bozulmaya yön tutsa da iyi ki blog açma hedefimi düşük tutup "kendime not alıyorum" şeklinde belirlemişim. Çünkü günlüğümü sağda solda açık bıraksam daha fazla kişi okurdu :D  Neyse kısa bir dertleşme sonrası (kendimle) bugünkü konuya başlayabilirim. 

Quarkus Framework'üyle karşılaşmam aslında tesadüf oldu. Spring Boot'un performansıyla ilgili bir iki gugıllama sonrasında Quarkus ile geliştirilen uygulamanın ayağa kalkış süresinin daha az, containerize edildiğinde daha küçük boyutlu olduğunu ve aynı işi daha kısa sürede bitirdiğini gördüm. Quarkus ile ilgili biraz daha araştırma yaptığımda bu konuyla ilgili Mehmet Cem Yücel Hoca'nın yazısı ile karşılaştım. Gerçekten akıldaki boşlukları dolduran güzel bir yazı size de okumanızı tavsiye ediyorum.

## Just in Time (JIT) ve Ahead of Time (AOT)
Ahead of Time; kodun bulunduğu makinenin üzerinde yürütülebilecek şekilde native makine kodlarına dönüştürülmesini yapan derleyici çeşitidir. Burada "bulunduğu makine" kelimesinden anlayabileceğiniz gibi platforma bağlı derleyicilerdir. Kodun tamamı makine koduna dönüştürülerek hızlı bir şekilde kodun yürütülmesini sağlarlar. C, C++ dilleri bu derleyicileri kullanır.

Just in Time; yüksek seviyeli dillerin kullandığı derleyici çeşitidir. JIT, interpretation ve AOT yaklaşımlarını kullanır. Interpretation adından da anlaşılacağı üzere yorumlayıcıdır. Java'da kod önce bytecode'a dönüştürülür sonrasında JIT compiler ile bu bytecode yürütüleceği zaman dinamik şekilde makine koduna dönüştürülür. AOT yaklaşımı kodların makine koduna dönüştürülme kısmında kullanılırken, kodun analizini yapan, performans arttırmaya yarayan yaklaşım interpretation'dır.

Peki işin içerisine docker girdiğinde, Java'nın platformdan bağımsız olup olmamasının pek de bir önemi kalmıyor gibi. Ben bir paket içerisine uygulamayı hazırlıyorum ve o paketi dağıtıyorum. O zaman AOT yaklaşımıyla C,C++ gibi diller gibi daha yüksek bir performansa ulaşmamız Java ile de mümkün mü?

## Quarkus Nedir?
Quarkus tamamiyle yukarıdaki sorularımıza cevap olarak doğmuş. İlk amacı container yapılarında hızlı şekilde ayağa kalkmak ve daha az yer kaplamak. "OpenJDK Hopspot ve GraalVM ile en iyi Java kütüphanelerini ve standartlarını kullanarak Kubernetes Native bir ortam için oluşturuldum" diyor. Bir JVM alternatifi olan GraalVM'in ne olduğunu öğrenmek için sitesine gittiğinizde AOT ifadesini görüyorsunuz. Bu da Quarkus'un bu JVM'i neden tercih ettiğini az çok açıklıyor. Container ortamında kodların bytecode'a çevrilip yeniden makine koduna dönüştürülme gibi bir ihtiyacı yok AOT yaklaşımıyla bunu daha hızlı şekilde halledebiliyor. Projeyi oluşturduğunuzda aşağıdaki yapı ile geliyor.

![Quarkus](/010-quarkus-tree.png)

Quarkus'un Avantajları
- src/main/docker klasörü altında farklı seçenekler için Dockerfile'lar oluşturulmuş hazır halde bulunuyor.
- RestEasy JAX-RS yapısında bir başlangıç yapısı sunuyor.
- Geliştirdiğiniz uygulamayı old-school yolla JVM ile çalıştırabildiğiniz gibi GraalVM ile native şekilde de çalıştırabiliyorsunuz.
- Extensionları eklemek için ./mvnw quarkus:add-extension -Dextensions="extension-name" komutunu kullanabiliyorsunuz.
- Spring framework'ünü de aynı zamanda destekliyor. Geniş bir extension desteği bulunuyor.
- Fakat benim en fazla sevdiğim şey, geliştirme aşamasında kod içerisinde bir değişiklik yaptığınızda uygulamayı durdur/yeniden başlat yapma mecburiyetinde değiliz. Yazdığımız anda değişik uygulanıyor.

Bu yazıda Quarkus yazılarına küçük bir giriş yapmak istedim. Bir sonraki yazımda Quarkus ve Postgres ile bir JAX-RS CRUD uygulama nasıl geliştirilir onu inceleyeceğiz. Quarkus'un arkasındaki fikir benim hoşuma gitti açıkçası bu yüzden üzerinde uzun bir süre duracağız.

Yeni yılın size güzellikler sunması dileğiyle.