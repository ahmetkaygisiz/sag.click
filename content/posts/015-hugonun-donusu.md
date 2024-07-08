---
title: "Hugo'nun Donusu"
date: 2024-07-06T13:28:11+03:00
draft: false

categories: [
  "theothers"
]
---

Cok uzun aralar blog için pek hoş olmuyormuş. Bu postu hatırlamak ve geri dönüş için bir açık aralık bırakmak için oluşturuyorum. 

Tekrar hatırlamak için neler yaptım?

1 - GitHub'a yeni pcden bir sshkey oluşturdum. Onu sshkeys'e yapıştırdım.

```
ssh-keygen -t ed25519 -C "xxxxx"
```

2 - github'a push yapabilmek için github için oluşturduğum sshkey'i ~/.ssh/config dosyasına aşağıdaki şekilde ekledim.

```bash

Host github.com
  IdentityFile ~/id_github_ed25519
```

3- Hugo'nun versiyonunu indir

```
wget https://github.com/gohugoio/hugo/releases/download/v0.128.2/hugo_extended_0.128.2_Linux-64bit.tar.gz
tar -xvf hugo_extended_0.128.2_Linux-64bit.tar.gzls -rlta
mkdir 0.128.2.lnx
mv hugo LICENSE README.md 0.128.2.lnx/
ln -s 0.128.2.lnx latest 
HUGO_HOME=/mnt/c/Ake/Apps/hugo/latest/
$HUGO_HOME/hugo version
```

4 - Yeni postu oluştur
```bash
$HUGO_HOME/hugo new posts/015-hugonun-donusu.md
```

5 - Kontrol et
```
$HUGO_HOME/hugo server
```


To be continue...


