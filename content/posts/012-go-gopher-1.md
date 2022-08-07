---
title: "Go Gopher-I"
date: 2021-04-11T18:23:40+03:00
draft: false
categories : [
    "code",
]
tags : [
    "golang",
    "go",
    "android"
]
---
Merhaba,

Uzun zamandır kısa kısa göz gezdirip ilerleyemediğim Go öğrenme hedefim için burada bir yazı ile işi resmiyete dökmek istedim. Kısa bir başlangıç da olsa bu yazımda linux'de go kurulum aşamasını, "Hello, World!" kodunu ve bunun farklı ortamlar için nasıl derlenebileceği konusundan bahsedeceğim. 

## İndirme ve Kurulum
1. Gugıl'da golang download yazdığımızda şu adres ile karşılaşıyoruz : https://golang.org/dl/
2. Ortamımıza uygun paketi indiriyoruz. Linux paketi : go1.16.3.linux-amd64.tar.gz
3. Golang path'ini oluşturuyorum ve indirdiğim paketi oraya extract ediyorum.

```bash
mkdir -p /data/installations/golang/1.16.3
tar -C /data/installations/golang/1.16.3 -xzf go1.16.3.linux-amd64.tar.gz
```
4. Bu pathi sistem değişkeni olarak ekleyeceğiz fakat burada görüldüğü gibi versiyona göre dosyaladık. Buraya bir sembolik link oluşturuyoruz ve diyoruz ki en son çıkan go versiyonunu işaret et. Yeni versiyon çıktığında yapmamız gereken sadece sembolik linkin gösterdiği dosyayı değiştirmek olacak.
```bash 
ln -s /data/installations/golang/1.6.3/go /data/installations/golang/latest 
```
5. Şimdi sistem değişkeninde GO_HOME adresimizi belirtiyoruz.
```bash
# bash içim
vim .bashrc

# zsh için
vim .zshrc
	
export GO_HOME="/data/installations/golang/latest"
PATH="$PATH:$GO_HOME/bin"
```

Değişikliklerin uygulanması için .bashrc/.zshrc dosyamızı source ile yeniden tanıtıyoruz.
```bash
source .zshrc
# ya da
source .bashrc
```
Ve kontrol ediyoruz.
```bash
go version
# go version go1.16.3 linux/amd64
```
## Build & Run 
Hello world kodumuzu yazmak için bir klasör oluşturduk ve içerisine girdik.
```bash
mkdir 01_first_go && cd $_
```
Kod editörü ile dosyayı .go uzantılı dosyayı oluşturuyoruz.
```bash
# with vim
vim main.go

# with visual code
code main.go

# with sublime 
subl main.go
```
Selam verdiğimiz main.go dosyasına kodumuzu yazıyoruz ve kaydediyoruz.
```go
package main

// Temel fonksiyonları kullanmamızı sağlayacak, out imkanı da sağlayan kütüphane
import "fmt"

// Uygulamanın başlangıç noktası main() fonksiyonu
func main() {
        
    //  Println ile selamımızı ekrana basıyoruz
	fmt.Println("Hi! I want to be a Gopher!")
}
```
Kodumuzu yürütüyoruz ve verilen selamı alıyoruz.
```bash
go run main.go
# Output : Hi! I want to be a Gopher!
```
Sırada build işlemi var. Aşağıda belirttiğim şekilde build işlemini yapabiliyoruz. 
```bash
# bulunan dizindeki tüm .go uzantılı dosyaları build eder.
go build 
```
```bash
# parametre olarak verilen .go dosyamızı build ediyor
go build main.go
```
Fakat ben go build komutunu yürüttüğümde şu hata ile karşılaştım : go: cannot find main module, but found. Bunun çözümü için  aşağıdaki komutu çalıştırıyoruz ve problem çözülüyor.
```bash
# Aşağıdaki değerin boş olduğunu gördüm.
go env | grep GO111MODULE
```
```bash
# auto değerini ilgili env parametresine atadım.
go env -w GO111MODULE=auto
```
Farklı Ortamlar için Build
Go bize farklı ortamlarda yürütülebilir paketler build edebilmemizi sağlıyor. Default olarak üstte oluşturduğumuz paketin hangi ortam için build edildiğini öğrenebilmek için file komutunu kullanıyoruz.
```bash
file 01_first_go
# Output : ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, Go BuildID=..., not stripped
```
Benim ortamım benzeri 64-bit linux ortamlarda yürütülebilir bir executable dosyayı oluşturduk. Konu farklı ortamlara göre build edilmeye geldiğinde ise işin içerisine $GOOS ve $GOARCH değişkenleri giriyor.

GOOS değişkeni için aşağıdaki OS seçenekleri için kullanılabiliyor. GOARCH ise işlemci mimarisi seçenekleri sunuyor. Daha fazla ayrıntı için https://golang.org/doc/install/source#environment adresini inceleyebilirsiniz. Ben de dedim Android telefonum var neden yürütmeyeyim. 

```bash
# Telefon mimarisine uygun seçenekler ile build ediyorum
GOOS=android GOARCH=arm64 GOARM=7 go build
```
Android için build ettiğimiz paketi yeniden file komutu ile kontrol ettiğimizde ARM aarch64 mimarisinin de eklendiğini görebiliyoruz.
```bash
# yeniden dosya yapısını kontrol ediyoruz.
file 01_first_go
# Output : 01_first_go: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /system/bin/linker64, Go BuildID=, not stripped
```
Burada yine paketim oluştu. Bunu alıp telefonuma attım. Terminal kullanabilmek için de Termux'u telefonuma indirdim ve termux'da aşağıdaki komutları çalıştırdım.
```bash
# executable hale getir.
chmod +x 01_first_go

# ve yürüt
./01_first_go
# Hi! I want to be a Gopher!
```

Kapanış
Gördüğünüz üzere bir selamımızı Go dünyasına verdik. Bu yazı sonrası için teminat sayılsın.

Güzel hafta sonları.
