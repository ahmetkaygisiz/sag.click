---
title: "Spring Rest Docs"
date: 2020-08-11T16:29:53+03:00
draft: false
categories : [
    "devops",
]
tags : [
    "git"
]
---
Merhabalar,

Spring Boot ile REST uygulamaları geliştirirken dökümantasyonu yapmaya yardımcı olan Spring REST Docs'u nedir, bize faydaları neler zorlukları neler onları temel olarak ele alacağım. Web Servisleri kullanırken en büyük problem çoğu zaman, hangi değerlerin nasıl kullanılması gerektiği konusunun muamma olması. Middleware olarak görev aldığım dönemde yıllardır kullanılan servislerin dökümanının olmaması, servisi kullanımını açtığımız her geliştiriciye nasıl kullanılmasının belirtilmesini gerektiriyordu. Başta size ağır bir yük gibi gözükse de, uzun vadede size vakit kazandıracak bir durum.

Şimdi gelelim Spring REST Docs avantajlarına : 

Test temelli oluşturulur. Döküman hazırlarken hem testinizi yapmış hem de dökümanınızı hazırlamış olursunuz. 
JSON ve XML formatını destekler.
Hypermedia'yı destekler (Grafik, ses, video, düz metin, linkler).
CURL ve HTTP'yi destekler.
Dezavantajları : 

Manuel bir şekilde asciidoctor ile oluşturulduğu için yazım konusu biraz sizi oyalayabilir. Fakat bunun döküman konusunda size esneklik sağlayacağını da unutmamak gerek.
Yol haritası şu şekildedir.
    TEST -> SNIPPETS -> ASCIIDOC -> FINAL DOCS

Default olarak bir testten 6 adet snippets oluşturulur. Bunlar : 

curl-request
http-request
http-response
httpie-request
request-body
response-body
Şimdi gelelim projemize. Spring Boot Starter adresinden Lombok, Spring Web, Spring Data JPA, H2 Database bağımlılıkları olan bir uygulama oluşturalım ve IDE'de projeyi açalım. Spring REST Docs için gerekli olan 3 bağımlılık daha var. Bunlardan birisi spring-boot-starter-test, diğeri junit ve sonuncusu da spring-restdocs-mockmvc. Aşağıdaki bağımlılıkları da dependencies alanına ekliyoruz. 

```xml
<!-- Added for REST Docs -->
<dependency>
    <groupId>org.springframework.restdocs</groupId>
    <artifactId>spring-restdocs-mockmvc</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13</version>
    <scope>test</scope>
</dependency>
```
Sonrasında, uygulamada dökümantasyonu yapacak/hazırlayacak olan plugin asciidoctor'u plugin alanına ekliyoruz. configuration tagları arasındaki sourceDirectory bizim index.html sayfamızı oluşturacak tasarımı oluşturan index.adoc dosyasını içeriyor. outputDirectory ise package safhasında oluşturulacak snippets ve index.html sayfasının oluşturulacak dizinini belirtiyor. outputDirectory belirtilmezse default olarak maven'de target/generated-snippets dosyasının altında dosyalar oluşturuluyor.
```xml
<plugin>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctor-maven-plugin</artifactId>
    <version>1.5.8</version>
    <executions>
        <execution>
            <id>generate-docs</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>process-asciidoc</goal>
            </goals>
            <configuration>
                <backend>html</backend>
                <doctype>book</doctype>
                <sourceDirectory>${project.basedir}/docs/source</sourceDirectory>
                <outputDirectory>${project.basedir}/docs/output</outputDirectory>
            </configuration>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>org.springframework.restdocs</groupId>
            <artifactId>spring-restdocs-asciidoctor</artifactId>
            <version>${spring-restdocs.version}</version>
        </dependency>
    </dependencies>
</plugin>
```
Şimdi uygulanın geliştirilme aşamasına geldik. main package altında controller, domain repository ve service isimli 4 package oluşturalım. İlk olarak applation.properties değerlerini verelim ve ardından POJO ile başlayalım. domain package'ı içerisnde User class'ını oluşturalım. Lombok ile @Data annotation'ını kullandığım için setter/getter oluşturmamıza gerek yok.

application.properties
```yml
spring.datasource.url=jdbc:h2:mem:userdb
spring.h2.console.enabled=true
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=as
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
```
User.java
```java
@Data
@Entity
@Table
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String email;
    private String firstName;
    private String lastName;
    private String phoneNumber;
}
Sırada Repository var. 

public interface UserRepository extends JpaRepository<User, Long> {

    @Transactional
    public void deleteByUsername(String username);
}
```
UserService ile Repository methodlarını implemente edelim.

```java
@Service
public class UserService {

    final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public void addUser(User user){
        userRepository.save(user);
    }

    public List<User> getAllUsers(){
        return userRepository.findAll();
    }

    public void deleteUserByUsername(String username){
        userRepository.deleteByUsername(username);
    }

    public User getUserById(Long id){
        return userRepository.findById(id).get();
    }
}
```
Son olarak REST Controller'ı yazalım.
```java
@RestController
@RequestMapping("/api/user")
public class UserController {

    final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping
    public List<User> getUsers(){
        return userService.getAllUsers();
    }

    @GetMapping("/{id}")
    public User getUserById(@PathVariable("id") Long id){
        return userService.getUserById(id);
    }

    @DeleteMapping("/{username}")
    public void deleteUserByUsername(@PathVariable("username") String username){
        userService.deleteUserByUsername(username);
    }

    @PostMapping
    public void addUser(@RequestBody User user){
        userService.addUser(user);
    }

    @PutMapping("/{id}")
    public void updateEmployee(@RequestBody User user, @PathVariable Long id) {
        User dbUser = userService.getUserById(id);

        dbUser.setFirstName(user.getFirstName());
        dbUser.setLastName(user.getLastName());
        dbUser.setEmail(user.getEmail());
        dbUser.setPhoneNumber(user.getPhoneNumber());
        dbUser.setUsername(user.getUsername());

        userService.addUser(dbUser);
    }
}
```
Uygulama ayağa kalktığında birkaç veri olsun istediğim için resources altında data.sql adında bir dosya oluşturarak içerisine 5 adet data ekliyorum.
```SQL
INSERT INTO user(id, username, email, first_name, last_name, phone_number) values (1,'akua','ahmetkaygisiz@gmail.com','ahme','ka','1231231212');
INSERT INTO user(id, username, email, first_name, last_name, phone_number) values (2,'ake','ake@gmail.com','ae','kawq','asdfasfwe');
INSERT INTO user(id, username, email, first_name, last_name, phone_number) values (3,'beka','aasq@gmail.com','me','kaqwes','1231231212');
INSERT INTO user(id, username, email, first_name, last_name, phone_number) values (4,'dejas','ahme@gmail.com','hm','kasqwe','1231231212');
INSERT INTO user(id, username, email, first_name, last_name, phone_number) values (5,'yodwe','tkaygisiz@gmail.com','ahmasd','kaasdfsa','1231231212');
```
Uygulama ayağa kalkıyor mu kalmıyor mu kontrolünü yaptıktan sonra sırada test ve dökümantasyon var. test/java/package/ altında UserControllerTest  isimli bir controller oluşturuyorum. ve aşağıdaki eklemeleri yaparak test'e hazır hale getiriyoruz.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserControllerTest {

    @Rule
    public final JUnitRestDocumentation restDocumentation = new JUnitRestDocumentation("docs/output");

    @Autowired
    private WebApplicationContext context;

    private MockMvc mockMvc;

    @Before
    public void setUp() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.context)
                .apply(documentationConfiguration(this.restDocumentation))
                .build();
    }
    
    // Test methods
}
```
JUnit 4 kullanırken dökümantasyon oluşturmak için ilk olarak bir JUnitRestDocumentation değişkeni oluşturup buna @Rule annotation'ını ekliyoruz. Dökümanların çıktı lokasyonlarını parametre olarak bu nesneye verebiliyoruz Parametre geçmezsek default olarak target/generated-snippets folderının altında oluşturuyor.

@Before ile tüm testler yapılmadan önce gerekli dökümantasyon ayarlarının da yapılması için bir setUp fonksiyonu yazıyoruz. Sıra testlerin yapılarak dökümanlarımızın oluşturulmasında.
```java
    @Test
    public void getUserById() throws Exception{
        this.mockMvc.perform(get("/api/user/1")
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andDo(MockMvcResultHandlers.print())
                .andDo(document("user/get-user-by-id"));
    }
```
getUserById() methodunda RestDocumentationRequestBuilder'ın static methodu olan get() ile get çağrısı yapıyoruz. gelen cevabın JSON olacağını belirtip gelen cevabın 200 olmasını bekleyip print() methodu ile output klasörü altında document() fonksiyonunda belirtilen user/get-user-by-id folder'ı altında yukarıda belirttiğim 6 adet snippets'in oluşmasını sağlıyoruz. Diğer dökümante ettiğimiz methodlar da aşağıdaki gibi. Güncelleme ve Ekleme fonksiyonunda @ResponseBody beklediğimiz için bir User oluşturup bunu ObjectMapper ile String olarak .content() fonksiyonuna parametre vererek gönderdik. 
```java
   @Test
    public void getUsers() throws Exception{
        this.mockMvc.perform(get("/api/user")
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andDo(MockMvcResultHandlers.print())
                .andDo(document("user/get-all-users"));
    }

    @Test
    public void deleteUserByUsername() throws Exception{
        this.mockMvc.perform(delete("/api/user/beka"))
                .andExpect(status().isOk())
                .andDo(MockMvcResultHandlers.print())
                .andDo(document("user/delete-user-by-username"));
    }

    @Test
    public void addUser() throws Exception{
        User u1 = new User();
        u1.setUsername("test");
        u1.setEmail("test@tim.com");
        u1.setFirstName("mest");
        u1.setLastName("oldum");
        u1.setPhoneNumber("1231231222");

        String body = (new ObjectMapper()).valueToTree(u1).toString();

        this.mockMvc.perform(post("/api/user")
                .contentType(MediaType.APPLICATION_JSON)
                .content(body))
            .andExpect(status().isOk())
            .andDo(MockMvcResultHandlers.print())
            .andDo(document("user/add-user"));
    }

    @Test
    public void updateUser() throws Exception{
        User u1 = new User();
        u1.setId(5L);
        u1.setUsername("test");
        u1.setEmail("test@tim.com");
        u1.setFirstName("mest");
        u1.setLastName("oldum");
        u1.setPhoneNumber("1231231222");

        String body = (new ObjectMapper()).valueToTree(u1).toString();

        this.mockMvc.perform(put("/api/user/5")
                .contentType(MediaType.APPLICATION_JSON)
                .content(body))
                .andExpect(status().isOk())
                .andDo(MockMvcResultHandlers.print())
                .andDo(document("user/update-user-by-id"));
    }
```
Son olarak docs/source dosyasındaki index.adoc ile dökümanımızın yapısını hazırlayalım.
```text
= User CRUD REST Service API Guide
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: right
:toclevels: 4
:sectlinks:
:sourcedir: ../../docs/output
DocWriter : <ahmetkaygisiz17@gmail.com>

== User CRUD Rest Service
This REST API developed by Ahmet Kaygisiz for learning Tutorial Spring Boot Rest Docs. 
RestDocsTutorial has following functions.

=== Get All Users

[%collapsible]
====
*Curl*
include::{sourcedir}/user/get-all-users/curl-request.adoc[]

*HTTP Request*
include::{sourcedir}/user/get-all-users/http-request.adoc[]


*HTTP Response*
include::{sourcedir}/user/get-all-users/http-response.adoc[]

*HTTPIE Request*
include::{sourcedir}/user/get-all-users/httpie-request.adoc[]

*Request Body*
include::{sourcedir}/user/get-all-users/request-body.adoc[]

*Response Body*
include::{sourcedir}/user/get-all-users/response-body.adoc[]
====

=== Get User By ID
[%collapsible]
====
*Curl*
include::{sourcedir}/user/get-user-by-id/curl-request.adoc[]

*HTTP Request*
include::{sourcedir}/user/get-user-by-id/http-request.adoc[]

*HTTP Response*
include::{sourcedir}/user/get-user-by-id/http-response.adoc[]

*HTTPIE Request*
include::{sourcedir}/user/get-user-by-id/httpie-request.adoc[]

*Request Body*
include::{sourcedir}/user/get-user-by-id/request-body.adoc[]

*Response Body*
include::{sourcedir}/user/get-user-by-id/response-body.adoc[]
====

=== Delete User By Username
[%collapsible]
====
*Curl*
include::{sourcedir}/user/delete-user-by-username/curl-request.adoc[]

*HTTP Request*
include::{sourcedir}/user/delete-user-by-username/http-request.adoc[]

*HTTP Response*
include::{sourcedir}/user/delete-user-by-username/http-response.adoc[]

*HTTPIE Request*
include::{sourcedir}/user/delete-user-by-username/httpie-request.adoc[]

*Request Body*
include::{sourcedir}/user/delete-user-by-username/request-body.adoc[]

*Response Body*
include::{sourcedir}/user/delete-user-by-username/response-body.adoc[]
====

=== Add User
[%collapsible]
====
*Curl*
include::{sourcedir}/user/add-user/curl-request.adoc[]

*HTTP Request*
include::{sourcedir}/user/add-user/http-request.adoc[]

*HTTP Response*
include::{sourcedir}/user/add-user/http-response.adoc[]

*HTTPIE Request*
include::{sourcedir}/user/add-user/httpie-request.adoc[]

*Request Body*
include::{sourcedir}/user/add-user/request-body.adoc[]

*Response Body*
include::{sourcedir}/user/add-user/response-body.adoc[]
====

=== Update User
[%collapsible]
====
*Curl*
include::{sourcedir}/user/update-user-by-id/curl-request.adoc[]

*HTTP Request*
include::{sourcedir}/user/update-user-by-id/http-request.adoc[]

*HTTP Response*
include::{sourcedir}/user/update-user-by-id/http-response.adoc[]

*HTTPIE Request*
include::{sourcedir}/user/update-user-by-id/httpie-request.adoc[]

*Request Body*
include::{sourcedir}/user/update-user-by-id/request-body.adoc[]

*Response Body*
include::{sourcedir}/user/update-user-by-id/response-body.adoc[]
====
```
Biraz uzun gelebilir bunu yazmakla mı uğraşacağım diyebilirsiniz fakat yapılan sadece 4 adet copy-paste ve folder isimlerinin değiştirilmesi. Kendinize ait bir sistem oturttukça bu size daha rahat gelecektir. Uygulamada mvn clean package yürüterek output dosyamızdaki snippets ve index.html dosyamızın oluştuğunu görüyoruz. 

![Example image](/003-spring_rest_docs.jpeg)

Bu yazı bir miktar uzun oldu fakat umarım işe yarar olmuştur. 

İyi çalışmalar dilerim.

Kodlar : https://github.com/ahmetkaygisiz/SpringRESTDocsCRUD

Kaynak : https://docs.spring.io/spring-restdocs/docs/2.0.4.RELEASE/reference/html5/
