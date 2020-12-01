# VM Instance dan Nginx Server

## Membuat VM instance menggunakan ```gcloud``` CLI:

Menggunakan perintah pada CLI console:

```bash
gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-c
```

Dengan menggunakan nama ```gcelab2```, menggunakan mesin tipe ```n1-standard-2``` dan memilih zona di ```us-central1-c```.

Setelah selesai membuat instance, selanjutnya memasang Nginx server. Pertama masuk sebagai root:

```bash
sudo su -
```

Instalasi Nginx:

```bash
apt-get install nginx -y
```

Konfirmasi bahwa Nginx berjalan:

```bash
ps auwx | grep nginx
```

