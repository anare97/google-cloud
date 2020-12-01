# Network dan HTTP Load Balancer

## Melakukan Set Region dan Zone

Perintah ini akan membuat setup zone pada ```us-central1-a``` dan region pada ```us-central1```.

```bash
gcloud config set compute/zone us-central1-a
gcloud config set compute/region us-central1
```

## Setup Web Server

### Membuat Multiple Web Server Instance

Membuat VM di default zone dan memberikan tag berupa ```network-lb-tag```.

```bash
    gcloud compute instances create www1 \
    --image-family debian-9 \
    --image-project debian-cloud \
    --zone us-central1-a \
    --tags network-lb-tag \
    --metadata startup-script="#! /bin/bash
        sudo apt-get update
        sudo apt-get install apache2 -y
        sudo service apache2 restart
        echo '<!doctype html><html><body><h1>www1</h1></body></html>' | tee /var/www/html/index.html"
```

### Membuat Firewall Rule

Membuat traffic dari luar dapat mengakses server yang sudah dibuat.

```bash
gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80
```

Mendapatkan list instance yang sudah dibuat.

```bash
gcloud compute instances list
```

### Pengaturan Layanan Load Balancing

Membuat static eksternal IP untuk load balancing.

```bash
gcloud compute addresses create network-lb-ip-1 \
 --region us-central1
```

Menambahkan HTTP health check.

```bash
gcloud compute addresses create network-lb-ip-1 \
 --region us-central1
```

Menambahkan target pool di region yang sama dengan instance yang dibuat. Jalankan perintah berikut untuk membuat target pool dan menggunakan health check (dibutuhkan agar layanan berfungsi).

```bash
gcloud compute target-pools create www-pool \
    --region us-central1 --http-health-check basic-check
```

Setelah itu masukkan instance yang sudah dibuat ke dalam pool. Perintah berikut memasukkan 3 instance, yaitu ```www1```, ```www2```, dan ```www3```.

```bash
gcloud compute target-pools add-instances www-pool \
    --instances www1,www2,www3 \
    --instances-zone us-central1-b
```

## Membuat HTTP Load Balancer

Pertama, buatlah template untuk load balancer menggunakan perintah berikut.

```bash
gcloud compute instance-templates create lb-backend-template \
   --region=us-central1 \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
   --image-family=debian-9 \
   --image-project=debian-cloud \
   --metadata=startup-script='#! /bin/bash
     apt-get update
     apt-get install apache2 -y
     a2ensite default-ssl
     a2enmod ssl
     vm_hostname="$(curl -H "Metadata-Flavor:Google" \
     http://169.254.169.254/computeMetadata/v1/instance/name)"
     echo "Page served from: $vm_hostname" | \
     tee /var/www/html/index.html
     systemctl restart apache2'
```

Buat sebuah instance group yang dapat diatur menggunakan template tadi.

```bash
gcloud compute instance-groups managed create lb-backend-group \
   --template=lb-backend-template --size=2 --zone=us-central1-a
```

Buat rule dengan nama ```fw-allow-health-check```. Sebuah rule ingress yang membolehkan akses traffic dari sistem health check milik Google Cloud.

```bash
gcloud compute firewall-rules create fw-allow-health-check \
    --network=default \
    --action=allow \
    --direction=ingress \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --target-tags=allow-health-check \
    --rules=tcp:80
```

Karena instance VM sudah berjalan, buat sebuah IP static global untuk akses dari luar ke load balancer.

```bash
gcloud compute addresses create lb-ipv4-1 \
    --ip-version=IPV4 \
    --global
```

Cek bahwa IPv4 sudah didapatkan.

```bash
gcloud compute addresses describe lb-ipv4-1 \
    --format="get(address)" \
    --global
```

Buat health check untuk load balancer.

```bash
gcloud compute health-checks create http http-basic-check \
    --port 80
```

Buat layanan backend.

```bash
gcloud compute backend-services create web-backend-service \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=http-basic-check \
    --global
```

Masukkan instance group yang sudah dibuat sebagai backend ke layanan backedn.

```bash
gcloud compute backend-services add-backend web-backend-service \
    --instance-group=lb-backend-group \
    --instance-group-zone=us-central1-a \
    --global
```

Buat url map untuk melakukan route dari request ke layanan backend.

```bash
gcloud compute url-maps create web-map-http \
    --default-service web-backend-service
```

Buat target HTTP proxy untuk melakukan route request ke url map.

```bash
gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map-http
```

Buat global rule untuk meneruskan request masuk ke proxy.

```bash
gcloud compute forwarding-rules create http-content-rule \
    --address=lb-ipv4-1\
    --global \
    --target-http-proxy=http-lb-proxy \
    --ports=80
```

