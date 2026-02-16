# charmed-kubernetes-postgresql-lab

<img src="arsitektur.jpg"/>

## vm1 (juju)
setup ssh (remote vm 2 & 3 by private key)

```scp -i remote.pem remote.pem ubuntu@13.217.97.251:~/.ssh/```

```ssh -i remote.pem ubuntu@13.217.97.251```

```chmod 400 ~/.ssh/remote.pem```

```nano ~/.ssh/config```

isi kan ini:
```
Host 54.161.125.241
  User ubuntu
  IdentityFile ~/.ssh/remote.pem

Host 98.93.111.79
  User ubuntu
  IdentityFile ~/.ssh/remote.pem
```
