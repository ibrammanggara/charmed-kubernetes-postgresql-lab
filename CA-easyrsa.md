kubernetes-control-plane belum terhubung ke Certificate Authority (EasyRSA)

## Cek apakah easyrsa ada:
Di controller:

```
juju status | grep easyrsa
```

Jika tidak ada → lanjut langkah:

Deploy EasyRSA:
```
juju deploy easyrsa --to 0
```
cek status:

```
juju status --watch 5s
```

## tunggu sampe active. jka sudah active lanjut:

buat relasi:

```
juju relate easyrsa kubernetes-control-plane
juju relate easyrsa kubernetes-worker
```

## cek status lagi
