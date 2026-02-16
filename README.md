# charmed-kubernetes-postgresql-lab

<img src="arsitektur.jpg"/>

## vm1 (juju)

install juju 
```
sudo snap install juju --classic
juju version
```

setup vm1 in juju controllers 
```
ssh-keygen -t ed25519
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
```

lalu koneksikan vm1 ke controller juju:

```
juju bootstrap manual/ubuntu@{ip-vm1} manual-controller
```

switch controllers:

```
juju switch <model-name>
```

untuk cek controller:

```
juju controllers
juju models
```

lalu buat directory untuk storageclass di vm3:
```
sudo mkdir -p /mnt/postgres/data
sudo chmod 777 /mnt/postgres/data
```

lalu setup ssh di vm1 (remote vm 2 & 3 by private key)

```scp -i remote.pem remote.pem ubuntu@{ip-vm1}:~/.ssh/```

```ssh -i remote.pem ubuntu@{ip-vm1}```

```chmod 400 ~/.ssh/remote.pem```

```nano ~/.ssh/config```

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
Machine  State    Address        Inst id              Base          AZ  Message
0        started  172.31.45.222  manual:              ubuntu@24.04      Manually provisioned machine
1        started  172.31.32.69   manual:54.81.75.173  ubuntu@24.04      Manually provisioned machine
2        started  172.31.40.233  manual:3.81.4.171    ubuntu@24.04      Manually provisioned machine
```

deploy control-plane(vm2) dan kubernetes-worker(vm3):

```
juju deploy kubernetes-control-plane --to 1
juju deploy kubernetes-worker --to 2
```

deploy container runtime:

```
juju deploy containerd
juju integrate kubernetes-control-plane containerd
juju integrate kubernetes-worker containerd
```

deploy networking(calico):

```
juju deploy calico
juju integrate calico kubernetes-control-plane
juju integrate calico kubernetes-worker

```

hubungkan worker ke control plane:

```
juju integrate kubernetes-worker:kube-control kubernetes-control-plane:kube-control
```

lihat status juju:

```
juju status
```

jika sudah di step sini lalu ada yang error, coba cek file problem-solv ;v
