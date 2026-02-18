# charmed-kubernetes-postgresql-lab

<img src="arsitektur.jpg"/>

3VM terdiri:
```
VM1 : juju controller
VM2 : control plane dll..
VM3 : worker
```

## vm1 (juju)

siapkan 3VM dan update paket:

```
sudo apt update && sudo apt upgrade -y
```

install juju 
```
sudo snap install juju --classic
juju version
```

setup VM1 untuk login juju dengan pubkey sendiri:

```
ssh-keygen -t ed25519
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
```

lalu koneksikan VM1 ke controller juju:

```
juju bootstrap manual/ubuntu@{ip-vm1} manual-controller
```
cek models:

```
juju models
```

switch models:

```
juju switch <model-name>
```

### untuk cek controller:

```
juju controllers
juju models
```

lalu setup ssh di vm1 (remote vm 2 & 3 by private key) buka terminal baru yang tersedia privatekey yang digunakan:

```
scp -i remote.pem remote.pem ubuntu@{ip-vm1}:~/.ssh/
```

```
ssh -i remote.pem ubuntu@{ip-vm1}
```

```
chmod 400 ~/.ssh/remote.pem
```

```
nano ~/.ssh/config
```

isi kan ini:
```
Host {ip-vm2}
  User ubuntu
  IdentityFile ~/.ssh/remote.pem

Host {ip-vm3}
  User ubuntu
  IdentityFile ~/.ssh/remote.pem
```
tambah machine:

```
juju add-machine ssh:ubuntu@{ip-vm2}
juju add-machine ssh:ubuntu@{ip-vm3}

```

cek machine:

```
juju machines
```
akan terlihat =
```
ubuntu@juju:~$ juju machines
Machine  State    Address        Inst id               Base          AZ  Message
0        started  172.31.45.222  manual:               ubuntu@24.04      Manually provisioned machine
1        started  172.31.32.69   manual:54.221.54.249  ubuntu@24.04      Manually provisioned machine
2        started  172.31.40.233  manual:13.221.191.162 ubuntu@24.04      Manually provisioned machine
```

deploy charmed-kubernetes yang diperlukan:

```
juju deploy easyrsa --to 1
juju deploy etcd --to 1
juju deploy kubernetes-control-plane --to 1
juju deploy kubernetes-worker --to 2
juju deploy calico
juju deploy containerd
```

relasikan satu per satu:

```
juju relate calico etcd
juju relate easyrsa etcd
juju relate easyrsa kubernetes-control-plane
juju relate easyrsa kubernetes-worker


juju relate etcd kubernetes-control-plane

juju relate kubernetes-control-plane:kube-control kubernetes-worker:kube-control

juju relate calico kubernetes-control-plane
juju relate calico kubernetes-worker

juju relate containerd kubernetes-control-plane
juju relate containerd kubernetes-worker
```

lihat status juju:

```
juju status
```

<img src="status.jpeg"/>

# problem solv

jika pada jaringan default settingan:

```
net.ipv4.conf.all.rp_filter = 1
```
kami menggunakan jaringan vpc pada aws dengan default seperti di atas, maka cara atasi di VM2 dan VM3:

```
sudo nano /etc/sysctl.d/99-calico.conf
```

isi lengkap:

```
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
net.ipv4.conf.<interface-eth0>.rp_filter=0
```

note : ganti pada <interface-eth0> sesuai interface yang digunakan
lalu save dan jalankan :

```
sudo sysctl --system
```

## lanjut setup storageclass untuk postgresql-k8s

di VM3:

```
sudo mkdir -p /mnt/postgres/data
sudo chmod 777 /mnt/postgres/data
```

lalu di VM1:

```
sudo snap install kubectl --classic
juju scp kubernetes-control-plane/0:config config
mv config ~/.kube/config
chmod 600 ~/.kube/config
```

tes:

```
kubectl get nodes
```

result contoh:

```
ubuntu@ip-172-31-46-100:~$ kubectl get nodes
NAME               STATUS   ROLES           AGE   VERSION
ip-172-31-36-198   Ready    control-plane   43m   v1.34.4
ip-172-31-46-67    Ready    <none>          42m   v1.34.4 (ini VM3 = worker)

```

masuk ke folder setup, lalu:

```
kubectl apply -f storageclass.yaml
```
pada file pv-postgres.yaml edit pada values ganti name alamat ip internal (worker) dari output get nodes

```
kubectl apply -f pv-postgres.yaml
```

cek ini, dan status harus available:

```
kubectl get pv
```

## deploy postgresql-k8s dengan storageclass

di VM1:

```
juju deploy postgresql-k8s --trust
```

set storageclass

```
juju config postgresql-k8s storage-class=local-postgres
```











