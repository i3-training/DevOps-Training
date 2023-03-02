<a name="readme-top"></a>

# Add Sample HPA on Kubernetes

## Requirements
- Kubernetes Cluster has been deployed
- Metric Server has been installed

## Preparation
- [Installing Metric Server on K8s Cluster]

## Installing Metric Server
Metrics Server adalah sumber metrik resource container yang skalabel dan efisien untuk pipeline penskalaan otomatis bawaan Kubernetes.
Metrics Server mengumpulkan metrik sumber daya dari Kubelet dan memaparkannya di apiserver Kubernetes melalui Metrics API untuk digunakan oleh Horizontal Pod Autoscaler dan Vertical Pod Autoscaler. Metrics API juga dapat diakses oleh kubectl top, membuatnya lebih mudah untuk men-debug pipeline penskalaan otomatis.

Server Metrik dapat diinstal baik langsung dari manifes YAML atau melalui grafik Helm resmi. Untuk menginstal rilis Server Metrik terbaru dari manifes components.yaml, jalankan perintah berikut.
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
## Menjalankan dan Mengekspose php-apache Server
Untuk mendemonstrasikan HorizontalPodAutoscaler, pertama-tama Anda akan memulai Deployment yang menjalankan container menggunakan image sample-hpa, dan memaparkannya sebagai Layanan menggunakan manifest berikut:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
```
Kemudian jalanankan command seperti dibawah ini:
```
kubectl apply -f deployment-hpa.yaml
```

```
deployment.apps/php-apache created
service/php-apache created
```
## Membuat Horizontal Pod Autoscaler
Sekarang setelah server berjalan, buat autoscaler menggunakan kubectl. Ada subperintah kubectl autoscale, bagian dari kubectl, yang membantu Anda melakukan ini.
Kita  akan menjalankan perintah yang membuat HorizontalPodAutoscaler yang memelihara antara 1 dan 10 replika Pod yang dikontrol oleh Deployment php-apache yang kita buat pada langkah pertama instruksi ini.
Secara kasar, pengontrol HPA akan menambah dan mengurangi jumlah replika (dengan memperbarui Deployment) untuk mempertahankan penggunaan CPU rata-rata di semua Pod sebesar 50%.
Deployment kemudian memperbarui ReplicaSet - ini adalah bagian dari cara kerja semua Deployment di Kubernetes - dan kemudian ReplicaSet menambahkan atau menghapus Pod berdasarkan perubahan pada spec-nya.

Membuat HorizontalPodAutoscaler:
```
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

```
horizontalpodautoscaler.autoscaling/php-apache autoscaled
```
Kita dapat memeriksa status terkini dari HorizontalPodAutoscaler yang baru dibuat, dengan menjalankan:
```
kubectl get hpa
```
Hasilnya akan seperti ini :
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache/scale   0% / 50%  1         10        1          18s
```
## Menambahkan load traffic
Selanjutnya, lihat bagaimana autoscaler bereaksi terhadap peningkatan beban. Untuk melakukannya, Kita akan memulai Pod yang berbeda untuk bertindak sebagai klien. Kontainer di dalam Pod klien berjalan dalam loop tak terbatas, mengirimkan kueri ke layanan php-apache.
```
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```
Setelah itu dijalankan hasilnya seperti ini:
```
If you don't see a command prompt, try pressing enter.
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!
```
Kemudian kita cek apakah pod yang telah diberikan beban traffic melakukan scaling secara otomatis:
```
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   39%/50%   1         10        1          55m
php-apache   Deployment/php-apache   250%/50%   1         10        1          55m
php-apache   Deployment/php-apache   250%/50%   1         10        4          56m
php-apache   Deployment/php-apache   84%/50%    1         10        5          56m
php-apache   Deployment/php-apache   71%/50%    1         10        5          56m
php-apache   Deployment/php-apache   66%/50%    1         10        8          56m
php-apache   Deployment/php-apache   60%/50%    1         10        8          57m
php-apache   Deployment/php-apache   61%/50%    1         10        8          57m
php-apache   Deployment/php-apache   57%/50%    1         10        8          57m
php-apache   Deployment/php-apache   59%/50%    1         10        8          58m
php-apache   Deployment/php-apache   52%/50%    1         10        8          58m
php-apache   Deployment/php-apache   62%/50%    1         10        8          58m
php-apache   Deployment/php-apache   61%/50%    1         10        8          58m
php-apache   Deployment/php-apache   53%/50%    1         10        8          59m
php-apache   Deployment/php-apache   63%/50%    1         10        8          59m
```
Dari gambar diatas dapat kita liat pod php-apache melakukan replicas sebanyak 8 karena beban yang diberikan telah mencapai batas 50%

Kemudian untuk menghentikan beban yang diberikan tekan control+C diterminal eksekusi sebelumnya:
```
^Cpod "load-generator" deleted
pod default/load-generator terminated
```
Maka pod akan secara otomatis ada melakukan scaling down seperti gambar dibawah ini :
```
php-apache   Deployment/php-apache   60%/50%    1         10        8          59m
php-apache   Deployment/php-apache   56%/50%    1         10        8          59m
php-apache   Deployment/php-apache   58%/50%    1         10        8          60m
php-apache   Deployment/php-apache   9%/50%     1         10        8          60m
php-apache   Deployment/php-apache   1%/50%     1         10        8          60m
php-apache   Deployment/php-apache   0%/50%     1         10        8          60m
php-apache   Deployment/php-apache   0%/50%     1         10        8          65m
php-apache   Deployment/php-apache   0%/50%     1         10        2          65m
php-apache   Deployment/php-apache   0%/50%     1         10        1          65m
```

