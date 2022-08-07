---
title: "Quarkus Crud"
date: 2021-02-10T18:12:48+03:00
draft: false
categories : [
    "code",
]
tags : [
    "java",
    "quarkus",
    "crud"
]
---
Merhabalar,<br>
Quarkus Framework'ünün fantastikliğinden bir önceki yazıda bahsetmiştik. Şimdi bu framework ile basit bir crud uygulamasını nasıl yaparız buna bakalım.

### Projenin Oluşturulması
Projeyi oluşturmadan önce versiyon gereksinimleri aşağıdaki gibi olmalıdır.

- java -version -> 8 veya 11 
- mvn -version ->  3.6.2+ 
- JAVA_HOME ortam değişkeni tanımlı olmalı

Projemizi oluşturmak için aşağıdaki komutu giriyoruz.
```bash
mvn io.io.quarkus:quarkus-maven-plugin:1.10.5.Final:create \
        -DprojectGroupId=com.akua \
        -DprojectArtifactId=quarkus-tutorial \
        -DclassName="com.akua.resources.UserResources" \
        -Dpath="/users"
```
Herhangi bir ekleme yapmadan quarkusun bize sunmuş olduğu dosya yapısı aşağıdaki şekildedir. Görüldüğü gibi bize src/main/docker folder'ı altında hazır 3 adet Dockerfile sunuyor. pom.xml içerisinde de eklentiler, profil, temel gerekli olan bağımlılıklar ve tanımlamalar başlangıç yapısında bizi karşılıyor.

```bash
 kaygisiz λ quarkus-tutorial ➜ tree .
.
|-- README.md
|-- mvnw
|-- mvnw.cmd
|-- pom.xml
`-- src
    |-- main
    |   |-- docker
    |   |   |-- Dockerfile.fast-jar
    |   |   |-- Dockerfile.jvm
    |   |   `-- Dockerfile.native
    |   |-- java
    |   |   `-- com
    |   |       `-- akua
    |   |           `-- resources
    |   |               `-- UserResources.java
    |   `-- resources
    |       |-- META-INF
    |       |   `-- resources
    |       |       `-- index.html
    |       `-- application.properties
    `-- test
        `-- java
            `-- com
                `-- akua
                    `-- resources
                        |-- NativeUserResourcesIT.java
                        `-- UserResourcesTest.java

15 directories, 12 files
```

Geliştireceğimiz uygulama RESTful bir servis olacak ve postgresql kullanacağız. Bunun için bazı bağımlılıkları pom.xml dosyamıza eklemeliyiz. Bu bağımlılıklar sırasıyla:

* jdbc-postgresql : Postgresql Veritabanı Bağımlılığı
* agroal : JDBC Pool Yönetimi
* resteasy-jackson : Serialize/Deserialize

Projeye başlangıçta bu bağımlılıkları -Dextensions= parametresiyle ekleyebileceğimiz gibi  ./mvnw quarkus:add-extension -Dextensions= komutunu kullanarak da ekleyebiliyoruz. Şimdi ihtiyacımız olan parametreleri ekleyelim ve X has been installed çıktılarını görelim.

```bash
./mvnw quarkus:add-extension -Dextensions="jdbc-postgresql,agroal,resteasy-jackson"
```
```bash
 kaygisiz λ quarkus-tutorial ➜ ./mvnw quarkus:add-extension -Dextensions="jdbc-postgresql,agroal,resteasy-jackson"
....
[INFO] --- quarkus-maven-plugin:1.10.5.Final:add-extension (default-cli) @ quarkus-tutorial ---
? Extension io.quarkus:quarkus-resteasy-jackson has been installed
? Extension io.quarkus:quarkus-agroal has been installed
? Extension io.quarkus:quarkus-jdbc-postgresql has been installed
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
...
```
![Uygulama Yapısı](/011-quarkus-tutorial-project-structure.png)
<center>Uygulama Yapısı</center>

Geliştireceğimiz uygulama yapısı yukarıdaki gibi olacak. api'mizin bir String mesaj döneceği Response, temel veri yapımız User, veritabanı ile ilişkilerin yönetileceği Manager ve Repository classları ve son olarak endpointlerin yönetileceği controller yapısındaki resource. 

İlk adım olarak application.properties dosyamıza veritabanı için gerekli değişkenlerimizi giriyoruz.

application.properties
```yaml
quarkus.datasource.db-kind=postgresql
quarkus.datasource.username=postgres
quarkus.datasource.password=${your_password}

quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/temp
quarkus.datasource.jdbc.max-size=16
İkinci adım olarak POJO'muzu oluşturalım.
```
domain/User.java
```java
package com.akua.domain;

import java.util.UUID;

public class User {
    private UUID id;
    private String email;
    private String password;

    public User() { }

    public User(UUID id, String email, String password) {
        this.id = id;
        this.email = email;
        this.password = password;
    }

    public UUID getId() {
        return id;
    }

    public void setId(UUID id) {
        this.id = id;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Object[] toObjectArray(){
        return new Object[] { email, password };
    }
}
```
Veritabanı bağlantılarını oluşturmalıyız. Burada bağlantı yönetimi ve prepared statement yönetimi için base classımız olan DBManager'ı oluşturuyoruz. Burada Dependency Injection @ApplicationScoped ile yapılıyor yani bu nesnenin yönetimi artık framework yönetimine geçiyor. Bu objeden bir tane oluşturuluyor ve @Inject annotation'ı tek oluşturulan bu nesne çağırılıp farklı classlarda kullanılabilir hale geliyor. 

DataSource objesi, application properties'de bilgilerini verdiğimiz veritabanına göre agroal JDBC yöneticisi tarafından oluşturuluyor ve bize @Inject annotation'ı ile bu veritabanı üzerinde işlem yapabilmemize olanak sağlıyor.

repository/manager/DBManager.java
```java
package com.akua.repository.manager;

import javax.enterprise.context.ApplicationScoped;
import javax.inject.Inject;
import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

@ApplicationScoped
public class DBManager {

    @Inject
    private DataSource dataSource;

    private Connection conn;

    public void connect() throws SQLException {
        conn = dataSource.getConnection();
    }

    public void close() throws SQLException {
        if ( conn != null)
            conn.close();
    }

    public PreparedStatement getPreparedStatement(String query) throws SQLException {
        return conn.prepareStatement(query);
    }

    public PreparedStatement getPreparedStatementWithParams(String query, Object[] params) throws SQLException {
        PreparedStatement ps = conn.prepareStatement(query);

        for (int i = 0; i< params.length; i++) {
            ps.setObject(i + 1, params[i]);
        }

        return ps;
    }

    public boolean executePS(String query) throws SQLException {
        PreparedStatement ps = getPreparedStatement(query);
        boolean status = ps.execute();
        ps.close();

        return status;
    }

    public int executePSWithParams(String query, Object[] params) throws SQLException {
        PreparedStatement ps = getPreparedStatementWithParams(query, params);
        int status = ps.executeUpdate();
        ps.close();

        return status;
    }
}
```
User için gerekli bir kaç method'un eklendiği UserDBManager class'ımız da aşağıdaki gibidir.
```java
package com.akua.repository.manager;

import com.akua.domain.User;

import javax.enterprise.context.ApplicationScoped;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

@ApplicationScoped
public class UserDBManager extends DBManager{

    public List<User> getResultSet(String query) throws SQLException {
        PreparedStatement ps = getPreparedStatement(query);
        ResultSet rs = ps.executeQuery();

        return resultSetToOArrayList(rs);
    }

    public List<User> getResultSetWithParams(String query, Object[] params) throws  SQLException {
        ResultSet rs = getPreparedStatementWithParams(query, params).executeQuery();

        return resultSetToOArrayList(rs);
    }

    public List<User> resultSetToOArrayList(ResultSet rs) throws SQLException {
        List<User> dataList = new ArrayList<>();

        while( rs.next() ){
            User user = new User();

            user.setId((UUID) rs.getObject("id"));
            user.setEmail(rs.getString("email"));
            user.setPassword(rs.getString("password"));

            dataList.add(user);
        }
        rs.close();

        return dataList;
    }
}
```

PreparedStatement'ın yürütülüp ve bize sonuç döndüren repository classımız da şu şekil.

repository/UserRepository.java
```java
package com.akua.repository;

import com.akua.domain.User;
import com.akua.repository.manager.UserDBManager;

import javax.enterprise.context.ApplicationScoped;
import java.sql.SQLException;
import java.util.List;
import java.util.UUID;

@ApplicationScoped
public class UserRepository {

    private static final String USERS_FIND_BY_ID = "SELECT * FROM users WHERE id = ?";
    private static final String USERS_FIND_ALL = "SELECT * FROM users";
    private static final String USERS_INSERT = "INSERT INTO users (id, email, password) VALUES(?, ?, ?)";
    private static final String USERS_UPDATE = "UPDATE people SET name = ?, age = ? WHERE id = ?";
    private static final String USERS_DELETE_BY_ID = "DELETE FROM users WHERE id = ?";

    private final UserDBManager manager;

    public UserRepository(UserDBManager manager){
        this.manager = manager;
    }

    public List<User> findAll(){
        try {
            return getQueryResults(USERS_FIND_ALL);
        }catch (SQLException e){
            e.printStackTrace();
        }
        return null;
    }


    public List<User> findById(UUID id) {
        try {
            return getUserById(USERS_FIND_BY_ID, id);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }

    public int insert(User user) {
        try {
            return executeQueryWithParams(USERS_INSERT, new Object[]{
                    user.getId(), user.getEmail(), user.getPassword()});
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return -1;
    }

    public int update(User user){
        try {
            return executeQueryWithParams(USERS_UPDATE, new Object[]{
                    user.getEmail(), user.getPassword(), user.getId()
            });
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return -1;
    }

    public int deleteById(UUID id){
        try {
            return executeQueryWithParams(USERS_DELETE_BY_ID, new Object[]{ id });
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return -1;
    }

    public boolean executeQuery(String query) throws SQLException {
        manager.connect();
        boolean isSucceed = manager.executePS(query);
        manager.close();

        return isSucceed;
    }

    public int executeQueryWithParams(String query, Object[] params) throws SQLException{
        manager.connect();
        int isSucceed = manager.executePSWithParams(query, params);
        manager.close();

        return isSucceed;
    }

    public List<User> getQueryResults(String query) throws SQLException{
        manager.connect();
        List<User> users = manager.getResultSet(query);
        manager.close();

        return  users;
    }

    public List<User> getUserById(String query, UUID id) throws SQLException {
        manager.connect();
        List<User> user = manager.getResultSetWithParams(query, new Object[]{ id });
        manager.close();

        return user;
    }
}
```
Bilgilendirme mesajlarımızı döneceğimiz Response classımız : 

api/Response.java
```java
package com.akua.api;

public class Response {
    String message;

    public Response(String message){
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```
Son olarak uygulamızın etkileşime geçeceği Resourse yapısı da şöyle. Aşağıda da görüldüğü üzere GET, POST, PUT, DELETE işlemlerini yapan bir adet RESTful api yazmış olduk.

resource/UserResource.java
```java
package com.akua.resource;

import com.akua.api.Response;
import com.akua.domain.User;
import com.akua.repository.UserRepository;

import javax.ws.rs.*;
import javax.ws.rs.core.MediaType;
import java.util.List;
import java.util.UUID;

@Path("/users")
public class UserResource {

    private final UserRepository userRepository;

    public UserResource(UserRepository userRepository){
        this.userRepository = userRepository;
    }

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public List<User> all(){
        return userRepository.findAll();
    }

    @GET
    @Path("{id}")
    @Produces(MediaType.APPLICATION_JSON)
    public List<User> get(@PathParam("id") UUID id){
        return userRepository.findById(id);
    }

    @POST
    @Produces(MediaType.APPLICATION_JSON)
    public Response post(User user){
        int status = userRepository.insert(new User(UUID.randomUUID(), user.getEmail(), user.getPassword()));

        if (status == 1)
            return new Response("User created!");
        return new Response("Something goes wrong");
    }

    @PUT
    @Path("{id}")
    @Produces(MediaType.APPLICATION_JSON)
    public Response put(@PathParam("id") UUID id, User user){
        if ( userRepository.findById(id) == null){
            throw new NotFoundException("User not found!");
        }
        int status = userRepository.update(new User(id, user.getEmail(), user.getPassword()));

        if (status == 1)
            return new Response("User updated!");
        return new Response("Something goes wrong");
    }

    @DELETE
    @Path("{id}")
    @Produces(MediaType.APPLICATION_JSON)
    public Response delete(@PathParam("id") UUID id){
        if ( userRepository.findById(id) == null){
            throw new NotFoundException("User not found");
        }
        int status = userRepository.deleteById(id);

        if (status == 1)
            return new Response("User deleted!");
        return new Response("Something goes wrong");
    }
}
```
### Uygulamanın Ayağa Kaldırılması

Quarkus'un en güzel yanlarından biri de development modda başlattığınızda backend'de yaptığınız bir değişikliğin yeniden başlatmaya gerek duymadan uygulanması. Development mod'da başlatmak için aşağıdaki komutu giriyoruz.

```bash
./mvnw compile quarkus:dev
```

### Sonuç ve Hedefler

Temel olarak bir CRUD uygulamayı geliştirdik ve ayağa kaldırdık. Quarkus ile olan maceralarımız burada bitmeyecek. Keşfetmemiz gereken native kısmını da ilerleyen yazılarda kurcalayıp asıl kullanılması gereken alanın neresi olduğunu göreceğiz. Başlangıç yapmış olduk geresini de getiririz umarım :)

Güzel günleri de görmek dileğiyle.
