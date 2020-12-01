# GCP Kubernetes Basic

## Membuat Cluster

Setting default compute zone:

```bash
gcloud config set compute/zone 'zone-name'
```

Membuat cluster baru:

```bash
gcloud container clusters create 'cluster-name'
```

## Mendapatkan Authenticaion Credential untuk Cluster

Autentikasi cluster:

```bash
gcloud container clusters get-credentials 'cluster-name'
```

## Melakukan Deploy Aplikasi ke Cluster

Dalam perintah ini akan menjalankan hello-app pada cluster. Sebelum men-deploy aplikasi, perlu membuat server terlebih dulu bernama hello-server dengan:

```bash
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```

Perintah di atas menggunakan image dari Google Container Registry untuk hello-app. Kemudian membuat kubernetes service, summber daya untuk memungkinkan aplikasi diakses dari traffic eksternal:

```bash
kubectl expose deployment hello-server --type=LoadBalancer --port 8080
```

--type akan membuat sebuah compute engine load balancer untuk container.
--port akan menentukan port yang digunakan.

## Melakukan Inspect Server

Menggunakan perintah di bawah untuk memeriksa server:

```bash
kubectl get service
```

## Menghapus Cluster

Jika ingin menghapus cluster yang tidak digunakan, dapat menggunakan perintah:

```bash
gcloud container clusters delete 'cluster-name'
```

