---
title: "Pi K8s Cluster"
date: 2025-05-31T21:30:38+03:00
draft: false
---
Merhabalar,
Bugün çok önceleri istediğim fakat yapamadığım k8s pi cluster için altyapıyı hazırlayarak başlayacağız. Bu blog öncesi kurulum yapmıştım fakat birkaç kez silip yeniden kurmam gerekti. Bunu süreçleri bir yere not etmemim iyi olacağını düşündüm. Hem de eğer buraya yazarsam biraz daha özenirim dedim. Bu sebeple buradayız. 

Şu an elimde 2 adet Rasberry Pi 5 var. Biri 4gb diğer 8gb ram'e sahip bunlara ek bir 8gb'lık daha gelecek. 8gb olan modelleri worker, 4gb olanı da master node olarak kullanacağım. Şu an için bir ssd planlamadım eğer bir yavaşlık sorunu yaşarsam ileride ssd ile master node'u upgrade ederiz. 

1. Pi'ler için aldığım 64gb'lık hafıza kartlarını formatlamak için Rasberry Pi Imager'ı kendi PC'me yükleyip açıyorum.

    1. İlk seçenekte rasberry pi modelini seçiyorum.
    2. OS olarak en lite olan Rasberry PI OS (64Bit) ile ilerliyorum.
    3. Hafıza kartı seçiyorum
    4. Next ile ilerliyorum.
    ![Pi01](/021/21-01-piimager.png)
    5. Config özelleştirmesi için "Edit Settings"i seçiyoruz.
    ![Pi01](/021/21-02-piimager.png)
    6. Hostname belirliyorum. 
    7. Pi'ye ssh yapabilmek için username ve password belirleyebiliyoruz. İstersek harici bir sshkey ile de bağlantı sağlanabiliyor. Ben password ile devam edeceğim.
    8. İlk bağlantıyı ethernet ile yapabileceğimiz gibi wifi ile de yapabiliyoruz aldığım modelde. Burada ağ bilgilerini giriyorum.
    9. Timezone ve klavye tipini belirliyorum.
    ![Pi01](/021/21-03-piimager.png)
    10. Yaptığımız değişiklikleri uygulamak için Yes diyoruz.
    ![Pi01](/021/21-04-piimager.png)
    11. Daha önce yapılan bir config varsa ya da üzerine yazmak için onaylayıp işlemin tamamlanmasını bekliyorum.
    ![Pi01](/021/21-05-piimager.png)
    12. Format işi tamamlandı Continue dedim ve sd kartı çıkarttım. Aynı işlemleri ikinci node için de uyguladım.
    ![Pi01](/021/21-06-piimager.png)

2. Pi'lere kartları taktık ve gücü verdik fakat hangi IP'ye bağlanacağız sorusuyla karşılaştık. Bunu bulmanın birden fazla yolu var. Ben modem arayüzünden ip'leri görmek istedim. Ben default gateway adresimi güncellemiştim farklı VPN'lerde çakışan bloklar olduğu için. O yüzden arayüze http://192.168.50.1/ adresinden ulaşıyorum.
![Pi01](/021/21-07-getipaddresses.png)

3. IP adreslerimizi elde ettik o zaman ssh yapabiliriz.
![Pi01](/021/21-08-connectmachines.png)
    
Bu ipler iyi güzel fakat sabit olmayacak. Bunları komutlar ile sabitlemek istiyorum.
``` bash
    # NODE1
    # set ip address & gateway ip
    sudo nmcli con mod preconfigured ipv4.addresses 192.168.51.250/24
    sudo nmcli con mod preconfigured ipv4.gateway 192.168.50.1
    
    # check the conn
    nmcli con show
    
    # reboot
    sudo shutdown -r now

    # connect
    ssh pc@192.168.50.250

    # NODE2 
    # set ip address as 192.168.50.251 like above & connect 
    ssh pc@192.168.50.251
```

4. Kubernetes kurulumu öncesi ön hazırlığımızı yapıyoruz.
    1. Upgrade'i çalıştırıyoruz.
    ```bash
    # update packages
    sudo apt update -y
    sudo apt upgrade -y
    ```

    2. Swap'ın kapalı olması gerekiyor bu sebeple swap'ı kontrol ediyoruz.
    ```bash
    # 
    cat /etc/fstab
    free -m 

    # swap servisini kontrol edelim.
    sudo systemctl status dphys-swapfile 

    # swapı kapatalım
    sudo swapoff -a

    # swap servisini kapatıp kaldıralım
    sudo systemctl stop dphys-swapfile
    sudo systemctl disable dphys-swapfile
    sudo apt-get purge dphys-swapfile -y

    # makineyi reboot edelim
    sudo shutdown -r now
    ```

    ```bash
    # swapları kontrol ettim ve 0 olduğunu gördüm.
    pc@pi01:~ $ free -m
                    total        used        free      shared  buff/cache   available
    Mem:            4050         210        3770          10         134        3840
    Swap:              0           0           0

    pc@pi02:~ $ free -m
                total        used        free      shared  buff/cache   available
    Mem:            8063         276        7729          13         138        7787
    Swap:              0           0           0
    ```

    3. Docker yüklü mü kontrol et, eğer bir çıktı olsaydı kaldırmam gerekecekti çünkü containerd yükleyeceğim.
    ```bash
        dpkg -l | grep -i docker
        # Eğer bir çıktı olsaydı kaldıracaktım çünkü containerd kurulumu yapmak istiyoruz.
        # sudo apt-get remove docker docker-engine docker.io containerd runc
    ```

4. IP yönlendirmesini açıyoruz. Bu farklı nodelardeki pod-to-pod haberleşmesi için gerekiyor.
```bash
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.ipv4.ip_forward = 1
    EOF
    
    # /etc/sysctl.d/ dizinindeki komutları uyguluyoruz.
    sudo sysctl --system
```

5. Container runtime olarak containerd kuracağım.

```bash
sudo apt update
sudo apt install -y containerd
systemctl status containerd

containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

# output 
pc@pi01:~ $ grep -i SystemdCgroup /etc/containerd/config.toml
SystemdCgroup = true

sudo systemctl restart containerd
sudo systemctl status containerd

# 
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

#
sudo systemctl restart containerd
```

6. pi için cgroup'un enable edilmesi için aşağıdaki komutu boot sırasında çalışması için cmdline dosyasına ekliyoruz.
```bash
sudo sed -i '$ s/$/ cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1/' /boot/firmware/cmdline.txt

# etc/host'a node iplerimi tanımladım.
vim /etc/hosts
#hosts
192.168.50.250  pi-k8s-master01.local
192.168.50.251  pi-k8s-node01.local

# reboot 
sudo shutdown -r now
```

7. Master Node'da k8s kubeadm kubelet kubectl kurulumu yapıyoruz.
```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl

TOKEN=$(sudo kubeadm token generate)

sudo kubeadm init --token=${TOKEN} --pod-network-cidr=10.244.0.0/16 --control-plane-endpoint "pi-k8s-master01.local:6443" --upload-certs
```

```bash
# kubectl'i çalıştırabilmek için kubeconfig dosyasını home dizinine alıyoruz.
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# flannel'ı yükle
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

8. Worker Node'umda da kubelet kubeadm kubectl'i de yükleyip kubectl join ile cluster'a worker'ı dahil ediyoruz.
```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl

# Yukarıda masterdan aldığımız token'ı worker node'da çalıştırıyoruz.
kubeadm join pi-k8s-master01.local:6443 --token 38in2k.wlb9is6v8mhvxq0p --discovery-token-ca-cert-hash sha256:ea03bb57dbfdecbf6951415bfd720fa09a2f61804f1a55b323ebaca147bcbd9a
```

9. Nodeları kontrol ediyoz ve son durum böyle.
![Pi01](/021/21-09-final-status.png)

Başladık devamı da gelmesi dileğiyle.
Güzel günlere.
