# charmed-kubernetes-postgresql-lab

<img src="arsitektur.jpg"/>

## vm1 (juju)
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
lalu koneksikan vm 2 dan 3 ke controller juju:

```
juju bootstrap manual/ubuntu@{ip-vm2} manual-controller
```

```
juju bootstrap manual/ubuntu@{ip-vm3} manual-controller
```
untuk cek controller:

```
juju controllers
juju models
```

switch controllers:
```
juju switch <model-name>
```

