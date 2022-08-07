---
title: "Spark Rest Api"
date: 2020-09-02T17:13:38+03:00
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
Merhaba,

Spark Framework'ü çok da uzak olmayan bir tarihte karşıma çıktı ve basitliği hafifliği sebebiyle mutlaka incelemem gerek diye düşündüm. Geçen günlerde terminal'de kendime arada motivasyon sağlamak amacıyla bir komut yazsam güzel olur diye düşünürken arkasına kendi hazırladığım bir servisi dahil edip oradan parse ederek bu isteğime ulaşabilirim hem de Spark'a bir giriş yapmış olurum dedim. Bu sebeple bu blogun konusu olan bir adet Quote API hazırladım. Bu API'de veritabanı olarak postgres, bağlantı için de JDBC kullandım. Objelerin serialize/deserialize işleminde de Gson'ı kullandım. Blog'da sadece Spark ile ilgili kısmı paylaşacağım. Kodun geri kalanını incelemek isteyenler [github](https://github.com/ahmetkaygisiz/quotes-rest-app) repoma göz atabilirler. 

Bir maven projesi açarak işe girişiyoruz. Sonrasında dependencies alanına gerekli bağımlılıkları ekliyoruz.
```xml
<!-- https://mvnrepository.com/artifact/com.sparkjava/spark-core -->
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-core</artifactId>
    <version>2.9.2</version>
</dependency>
```
Bu bağımlığı ekledikten sonra http methodlarını kullanabilir hale geliyorsunuz.
```java
public class App {
    public static void main(String[] args) {
        QuotesService quotesService = new QuotesService();

        try{
            //quotesService.insertDataFromJsonFile("./src/main/resources/quotes.json");
            
            // port(8080);
            
            // GET - Get quotes randomly
            get("/quotes/random", (req,res) -> {
                res.type("application/json");
                return quotesService.getJsonRandomQuote();
            });

            // ... methods ..

        } catch (Exception throwables) {
            throwables.printStackTrace();
        }
    }
}
```
Gördüğünüz gibi bir adet QuotesService'im var. Bu servis içerisinde objelerin json'a dönüştürülmesi, başlangıç için tabloların oluşturulması gibi methodlar bulunduruyor. Başlangıç için veritabanında bir miktar data olsun istediğim için netten bulup /resources dosyasının altına indirdiğim json dosyasını, insertDataFromJsonFile() methodunda gerekli deserialize işlemini yapıp veritabanına kaydettim. 

Spark bize oldukça basit bir yapı sunuyor. Gömülü bir Jetty server'ı bulunuyor. Başlangıç portunu port(int) değeriyle değiştirmemiz mümkün. Controller olarak kullandığımız kod dizini sadece : 

get("/path", (request,response) -> "return string" );
get/post/put/delete methodları, bizim ihtiyacımız olan tüm Http methodlarını bize sağlıyor. Lambda yapısı bulundurduğu için, java 1.8 versiyonuna ihtiyaç duyuyor. response.type() ile gönderdiğimiz verinin tipini belirtebiliyoruz.
```java
// POST - Save a quote
post("/quotes", (req,res) -> {
    res.type("application/json");
    res.status(201);

    // deserialize json
    Quote q = Jsons.jsonToObject(req.body(), Quote.TYPE_TOKEN);
    quotesService.save(q);

    return "Quote saved.";
});
```
Post methodunun response kodunu res.status(code) methoduyla gönderebiliyoruz.  Deserialize işleminin için request ile gönderilen body'i request.body ile alıp static Jsons methoduma gönderiyorum. Gson ile deserialize işlemi içerisinde hangi objeye dönüştürülmesini anlamasını sağlayan Type değerini verdikten sonra objeyi elde etmiş oluyoruz.
```java
// GET - Get quote by id
get("/quotes/:id", (req,res) -> {
    res.type("application/json");

    return  quotesService.getJsonQuoteById(Integer.parseInt(req.params("id")));
});
```
Get methodunda , request parametreleri de req.params fonksiyonu ile alınıyor. delete ve put methodlarının da aşağıda görebilirsiniz.
```java
    // DELETE - Delete quote by id
    delete("/quotes/:id", (req,res) -> {
        res.type("application/json");
        String id = req.params("id");

        quotesService.deleteById(Integer.parseInt(id));
        return "Quote deleted " + id;
    });

    // PUT - Update a quote by
    put("/quotes/:id", (req,res) -> {
        res.type("application/json");
        Quote q = Jsons.jsonToObject(req.body(), Quote.TYPE_TOKEN);

        Quote inDb = quotesService.getQuoteById(Integer.parseInt(req.params("id")));
        inDb.setAuthor(q.getAuthor());
        inDb.setQuote(q.getQuote());

        quotesService.updateQuoteById(inDb);

        return "Quote updated.";
    });
```
Uygulamanın boyutu da aşağıda görüldüğü üzere oldukça küçük.

```bash
kaygisiz@akua:target$ du -sh quotes-rest-app-1.0-SNAPSHOT-jar-with-dependencies.jar 
4.0M	quotes-rest-app-1.0-SNAPSHOT-jar-with-dependencies.jar
```
Güzel günlerde görüşmek dileğiyle.