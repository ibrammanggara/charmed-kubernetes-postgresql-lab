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
Machine  State    Address        Inst id              Base          AZ  Message
0        started  172.31.45.222  manual:              ubuntu@24.04      Manually provisioned machine
1        started  172.31.32.69   manual:54.81.75.173  ubuntu@24.04      Manually provisioned machine
2        started  172.31.40.233  manual:3.81.4.171    ubuntu@24.04      Manually provisioned machine
```

deploy charmed-kubernetes yang diperlukan:

```
juju deploy easyrsa --to 1
juju deploy etcd --to 1
juju deploy kubernetes-control-plane --to 1
juju deploy kubernetes-worker --to 2
juju deploy calico

```

hubungkan worker ke control plane:

```
p
```

lihat status juju:

```
juju status
```
