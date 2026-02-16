Status saat ini menunjukkan cluster tidak akan selesai dengan sendirinya karena ada error konfigurasi jaringan (Calico) yang menyebabkan komponen Kubernetes terblokir. Ini harus diperbaiki secara manual.

Masalah utama:

```
calico: blocked
ignore-loose-rpf config is in conflict with rp_filter value
```
Akibatnya:

```
kubernetes-control-plane: Waiting for certificate authority
kubernetes-worker: Waiting for certificate authority
```
## Penyebab Teknis

Calico membutuhkan pengaturan kernel:

```
net.ipv4.conf.all.rp_filter = 0
net.ipv4.conf.default.rp_filter = 0
```

AWS VPC menggunakan routing asymmetric.
Calico membutuhkan loose reverse path filtering (rp_filter=0) agar paket antar pod tidak diblok kernel.

Langkah Perbaikan (Lakukan di SEMUA NODE)

```
sudo nano /etc/sysctl.d/99-kubernetes.conf
```

isi:

```
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
```

lalu apply:
```
sudo sysctl --system
```
