---
title: "İşe Yarar Linux Komutları"
date: 2024-10-26T15:42:23+03:00
draft: false
---

İşe Yarar Linux Komutları
Uzuun bir aranın ardından merhabalar,
Blogum ve kendim için yeniden döndüm buraya. Blog yazmak için yeniden bir enerji ve heyecan hissediyorum bu dönemde. Daha önce standart plan sunan bir hosting sağlayıcıda laravel ile yayınladığım bu sitemi hugoya geçirmek istedim. Eski yazılarımı aktarırken bazı yerleri düzeltmem gerekiyor, onları da zamanla tekrar gözden geçireceğim. Benim için daha sade olacağını düşündüm. Ayrıca hosting konusunda birden fazla ücretsiz imkan da var ilerleyen zamanlar için değiştirmek istersem taşımam daha kolay olur. Şu an S3’e üzerinde host edip, Route53 ile yönlendiriyorum. Genel bir özet sonunda konuma geliyorum.

Farklı zamanlarda işim düştükçe not aldığım linux komutlarını paylaşacağım. Buradaki genel amacım iş değişikliği ya da veri kaybı olması durumunda ben şunu hangi komutla yapmıştım diyip kara kara düşünmemek. Tabi ki Github üzerinde de saklıyorum ama kendi sitem üzerinden copy-paste daha bir keyifli oluyor :D Farklı başlıklar altında gruplandıracağım.

``` bash
### Web
wget $URL get --no-check-certificate --http-user=$USER --http-password=$PASSWORD -r -nH --no-parent --level=1 --no-directories --reject="*.sha1" --progress=bar:force "${URL}" 2>&1

curl -v --cert-type P12 --cert cert.p12:changeit --location 'https://replace_me' \--header 'Content-Type: application/json' --data @request-body.json

curl  -H "Content-Type: text/xml; charset=utf-8" -H "SOAPAction:" -u $uuserna:$pass -d @request_file $URL

curl -v --proxy "http://$P_username:$P_password@$PROXY_HOST:$PROXY_PORT" -X POST $API_URL -H 'Content-Type: application/json' -d  @request-body.json
```

Text & Files
``` bash
# -n line nmber
# -A n line after match
# -B n line before match
# -C both
grep -A5 -B5 -n "11:00:00" server01_access.log00090
grep -C -n "11:00:00" server01_access.log00090
grep -o '[^ ]*$' # sadece dosyaları listele
ls -lrt | grep "Mar 26 17" | grep -o '[^ ]*$'
    
# Büyük boyutlu dosyaları kucuk parcalara böl
split -b 500m server02_osb02_heapdump.tgz heap_dump.tgz.part

# iki line arasındaki kısmı getir. 
awk -v s="$line1" -v e="$line2" 'NR>s&&NR<e' fileName > output.log

#24h (Bunlar bazen cortluyor :D)
awk '/2021-11-03 17:1?/,/2021-11-03 17:2?/' server01.out00855 
awk '/2022-01-17 12:00:0?/,/2022-01-17 13:00:0?/' server01.out | less

#12h
awk '/Jan 14, 2022 12:0?:0? AM/,/Jan 14, 2022 12:12:0? AM/' server01.out 
awk '/2022-02-02 08:32:59/,/2022-02-02 08:46:/' server01.out

sed -n '/Oct 26, 2021 11:20:00 AM/,/Oct 26, 2021 11:35:00 AM/p' server02_osb01.out
```

Process & Java
``` bash
# Process Kill
ps -ef | grep NodeManager | grep -v grep | awk '{print $2}' | xargs kill -9
ps -ef | grep java -v grep | awk '{print $2}' | xargs kill -9

# HeapDump
$JAVA_HOME/bin/jmap -dump:format=b,file=/opt/tmp/heapdump.bin 37320
$JAVA_HOME/bin/jmap -dump:format=b,file=hprof

kill -3 $pid >> tmp_`date`.log
$JAVA_HOME/bin/jstack $pid >> tmp_`date`.log

source ~/.commonenv

# Thread Dump
export JAVA_HOME="/data/jdk/latest"
export PID=`ps -ef | grep java | grep "/search_string/" | awk '{print $2}'`

$JAVA_HOME/bin/jstack $PID > threadDump.log
x=1; while [ $x -le 2 ]; do $JAVA_HOME/bin/jstack $PID >> test_`hostname`_`date '+%Y-%m-%d'`; sleep 10;(( x++ )); done

# GC
$JAVA_HOME/bin/jstat -gcutil $PID 1000 10

# Histogram
$JAVA_HOME/bin/jmap -histo:live,file=/tmp/histo_`date '+%Y%m%d_%H%M'`.data $PID

# Head Dump
$JAVA_HOME/bin/jmap -dump:format=b,file=/path/to/dump.hprof $PID
$JAVA_HOME/bin/jcmd $PID GC.heap_dump /path/to/dump.hprof
    
# JFR - !! duration : 20 dakika !!
$JAVA_HOME/bin/jcmd $PID JFR.start duration=5m filename=/path/to/record_20211210.jfr

#Check Cert
$JAVA_HOME/bin/keytool -list -v -keystore $JAVA_HOME/jre/lib/security/cacerts > /tmp/trusted.txt && less /tmp/trusted.txt
nmap -sV --script ssl-enum-ciphers -p 443 abc.website.com

openssl s_client -showcerts -verify 5 -connect $HOST:$PORT -servername $HOST < /dev/null 2> /dev/null | awk '/BEGIN/,/END/{ if(/BEGIN/){a++}; print}'

openssl s_client -showcerts -verify 5 -connect $HOST:$PORT -servername $HOST
```

``` bash
## Network
# Telnet Alternatifi
nc -w 3 -v $ip $port
curl -v telnet://$IP:$PORT
IPADDR="10.104.177.27";PORT=8080; timeout 3 /bin/bash -c "</dev/tcp/$IPADDR/$PORT" && echo "$IPADDR:$PORT ... Ok" || echo "Host $IPADDR:$PORT is unreachable"

lsof -i:8080

# linux how to see ports in use
lsof -i -P -n | grep LISTEN
netstat -tulpn | grep LISTEN
lsof -i:22 # see a specific port such as 22

nmap -sTU -O IP-address-Here
sudo lsof -i:<port_number>

echo exit | timeout 2s telnet 

# pid'ye göre port bul
lsof -i -P -n | grep $pid

# tcpdump port check	
tcpdump -nnSX port 443
    -nn : Don’t resolve hostnames or port names.
    -S : Get the entire packet.
    -X : Get hex output.
    -t : Give human-readable timestamp output.

# a specific network interface
ifconfig -a 
tcpdump -i $interface_name
tcpdump -i eth0

sudo tcpdump -i any -s 0 -w "`date +'%Y%m%d_%H%M'`_file.pcap"
sudo tcpdump -i any -s 0 -w "`date +'%Y%m%d_%H%M'`_name.pcap"  host dns.com

# with ip
tcpdump host 1.2.3.4
tcpdump src 1.2.3.4
tcpdump dst 1.2.3.4

find HTTP Cookies/headers etc.
tcpdump -vvAls0 | grep 'Set-Cookie|Host:|Cookie:'
```

Service call count
```bash
# $8 : response time kolonu
grep "/v1/test1" access_server0*_osb0*.log | wc -l 
grep "/v1/test1" access_server0*_osb0*.log | awk '{sum+=$8} END { print "Average = ",sum/NR}'

cat access_server0*_osb0*.log | sort -nr | uniq -c
cat access_server0*_osb0*.log | awk '{ if($8>20) print $0 }' | wc -l
cat access.log | awk '{if($6==500 && $3>"10:00" && $3<"10:30") print $0}'

# Bir komutu belirli aralıklarla tekrar et.
watch -n 10 'echo "test" >> file.f' 

wc -w file.f # get char count
wl -l file.f # get line count

# File Viewing
nl : With line numbers
nl -b a $fileName	# bosluklari da numaralandir

# Replace
sed -i "s/Host=/Host=$_IP/;sUser=/User=$_USERNAME/" file.f # replace properties
sed "-i.`date +%F`" "s|$APP_ROOT\/jdk\/jdkSet|$APP_ROOT\/Java\/jdkSet|g"  file.name

# Delete a record 
sed -i '/132.132.23.20/d' ~/.ssh/known_hosts

# Search
find . -mtime +10 | xargs ls -lrtah
find . -mtime-3 -ls

# find and compress last X day logs 
find . -type f \( -name "*.out*" -o -name "*.log*" \) -mtime -1 | xargs tar -cvzf tarName.tgz

find -type f -newermt "2021-03-26T14:00" \! -newermt "2021-03-26T12:30"
find -type f -name '*.jar' -newermt "2021-12-19" \! -newermt "2021-12-25" | xargs ls -lrtah

# delete
find . -type f -name '*jfr.part' | sed -r 's|/[^/]+$||' -mtime +10 -exec rm {} \;
find . -type f *name '*access*'-newerat 2021-06-10 ! -newerat 2021-06-12

/usr/bin/find $_OUTPUT_PATH -name "std_*.tgz" -mtime +3
find /home/randomcat -mtime +11 -type f -exec gzip {} \;

find . name test*
find ../elasticsearch/ -name *mapping*
    : find under another directory

find . -name *.txt
    : seach for files by extension

find . -type f -name "test*"
    : search only files
find . -type d -name "test*"
    : search for only dir names

find . -iname "Test*"
    : case insensitive

find ./test ./numeric -name file_name.txt -type f
    : search multiple directories

find . -type f ( -name "*.txt" -o -name "*.pdf" -o -name "*.doc" )
    : find multiple files with different extensions	

find . -name "*" -exec grep -il "/Middleware/" {} \; > FindResult.txt
find . -type f -exec grep -li "search_certain_text" {} \;
    : find files containing certain text

find / -size +100M -size -200M
    : files with sizes between 100-200MB

find ./ -type d –empty
    : find empty folders

find /path/ -type -f -name '*.txt' -mtime +8
    : find files older than n days.

atime : access time
mtime : mopdification time
ctime : creation time

find . -amin -10 -type f
    : accessed last 10 min.

find -perm 777
    : find files with permission 777

find /home -user appadmin
    : find files owned by a user

find ver -name "*.php" -type f -exec chmod 755 {} ;
    : find *.php files and chmod to 755

find . -type f -perm 644 -exec chmod 655 {} ;
    : yetkisi 644 olan dosyaları bul ve 655 yap.

#find -iname files_name -exec cp {} ~/to/target/folder ; 
echo target1 target2 target3 | xargs -n 1 cp file_txt;

find ~/tmp/dir1/ ~/tmp/dir2/ $HOME/3/ -maxdepth 0 -exec cp ~/numeric/hci {} ;
```

HW
```bash
# boyutu buyuk dosyaları bi şey yap
du -sh * | sort -rh | head -5 | awk '{print $2}' | xargs

# hidden files included
du -sch .[!.]* * | sort -h

# get OS release
cat /etc/*-release
lsb_release -a

lscpu | egrep 'Model name|Socket|Thread|NUMA|CPU\(s\)' 
less /proc/cpuinfo
grep -c ^processor /proc/cpuinfo

# check mount structure
lsblk -a

#
free -g
free -bg
```

```bash
# VM or Physical Machine
systemd-detect-virt # output: none-> Fiziksel, wmware: Sanal
cat /sys/class/dmi/id/product_name

sar -P ALL -f /var/log/sa/sa14
sar -P ALL -f /var/log/sa/sa10 -s 07:00:00 -e 15:00:00  

#cpu 
# sar 2 10
# sar -p 2 10
# sar  -P ALL 2 10
sar -P ALL -f /var/log/sa/sa14

#memory
sar -r 2 10
sar -r -f /var/log/sa/sa14
sar -R 2 10

# load avg.
sar -q 2 10
sar -q -f /var/log/sa/sa14

#io usage
sar -b 2 10
sar -b -f /var/log/sa/sa14

#disk 
sar -d -p 2 10
sar -d -p -f /var/log/sa/sa14
```


``` bash
## Bir komutla ilgili yardım
man $keyword
info $command

man -k $keyword
# Örn :
#>kaaygisiz@pc:~$ man -k tcp
#>ip-mptcp (8)         - MPTCP path manager configuration
#>tcp (7)              - TCP protocol
#...
#>tcpdump (8)          - dump traffic on a network

type $command # komut nereden çalışıyor.
# Örn :
#>kaaygisiz@pc:~$ type whoami
#>whoami is /usr/bin/whoami

which $command/$executable
# Örn :
#>kaaygisiz@pc:~$ which go
#>/snap/bin/go

!$		Represents the parameter from the prev command.
!*		Represents all from the previous command
# Örn:
#>kaaygisiz@pc:~/test$ ls a b c
#>a  b  c

#>kaaygisiz@pc:~/test$ wc !$
#>wc c
#>0 0 0 c

#>kaaygisiz@pc:~/test$ wc !*
#>wc a b c
#>1  1  9 a
#>1  1 25 b
#>0  0  0 c
#>2  2 34 total

basename /a/b/c : output c
dirname /a/b/c  : output /a/b
#>kaaygisiz@pc:~/test$ basename /data/workspace/go/sagclick/README.md 
#>README.md

#>kaaygisiz@pc:~/test$ dirname /data/workspace/go/sagclick/README.md 
#>/data/workspace/go/sagclick

# Log rotate
/usr/sbin/logrotate -s /opt/servers/bin/rotate-status.log -f rotate-catalinaout.conf

```

Genel olarak şimdilik eklemek istediğim şeyler bunlar. Sağdan soldan yine birşeyler çıkar eklerim.

İyi günler.