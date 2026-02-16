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

setup ssh (remote vm 2 & 3 by private key)

```scp -i remote.pem remote.pem ubuntu@{ip-vm1}:~/.ssh/```

```ssh -i remote.pem ubuntu@13.217.97.251```

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

