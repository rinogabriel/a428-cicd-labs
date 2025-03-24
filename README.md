# 🚀 Continuous Integration Pipeline dengan Jenkins

## 📌 Deskripsi

Repositori ini berisi contoh implementasi **Continuous Integration (CI) Pipeline menggunakan Jenkins** untuk aplikasi **React**. Dengan pipeline ini, setiap perubahan pada kode akan **di-build dan diuji secara otomatis**, meningkatkan efisiensi dalam pengembangan perangkat lunak.

## 🔧 Prasyarat
Sebelum memulai, pastikan Anda telah menginstal:
- **Docker** & **Docker Compose**
- **Git**
- **Jenkins**
- **Node.js** & **npm**

## 📂 Struktur Direktori
```
├── jenkins/
│   ├── scripts/
│   │   ├── test.sh
│   ├── Jenkinsfile
├── src/
│   ├── App.js
│   ├── App.test.js
├── package.json
├── README.md
```

## 🚀 Langkah-langkah Implementasi

### 1️⃣ Menjalankan Jenkins di Docker

```sh
docker network create jenkins

docker run --name jenkins-docker --detach --privileged --network jenkins \
--network-alias docker --env DOCKER_TLS_CERTDIR=/certs \
--volume jenkins-docker-certs:/certs/client --volume jenkins-data:/var/jenkins_home \
--publish 2376:2376 --publish 3000:3000 --restart always docker:dind --storage-driver overlay2
```

### 2️⃣ Menjalankan Jenkins dengan Blue Ocean UI
```sh
docker build -t myjenkins-blueocean:2.346.1-1 .

docker run --name jenkins-blueocean --detach --network jenkins \
--env DOCKER_HOST=tcp://docker:2376 --env DOCKER_CERT_PATH=/certs/client \
--env DOCKER_TLS_VERIFY=1 --publish 8080:8080 --publish 50000:50000 \
--volume jenkins-data:/var/jenkins_home --volume jenkins-docker-certs:/certs/client:ro \
--volume "$HOME":/home --restart=on-failure \
--env JAVA_OPTS="-Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true" \
myjenkins-blueocean:2.346.1-1
```

### 3️⃣ Konfigurasi Jenkins
1. **Buka Jenkins**: `http://localhost:8080`
2. **Unlock Jenkins** dengan password dari log:
   ```sh
   docker logs jenkins-blueocean
   ```
3. **Install Suggested Plugins**.
4. **Buat Admin User** dan **Save Configuration**.

### 4️⃣ Clone Repository & Konfigurasi Pipeline

```sh
git clone -b react-app https://github.com/USERNAME-GITHUB/a428-cicd-labs.git
cd a428-cicd-labs
```

1. **Buka Jenkins** → **Create a new job** → Pilih **Pipeline**
2. **Pilih** `Pipeline script from SCM` → **Masukkan Repository URL**
3. **Branch Specifier**: `*/react-app`
4. **Simpan dan Jalankan Pipeline**

### 5️⃣ Membuat Jenkinsfile
Buat file `Jenkinsfile` di root proyek dengan isi berikut:

```groovy
pipeline {
    agent {
        docker {
            image 'node:16-buster-slim'
            args '-p 3000:3000'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
    }
}
```

### 6️⃣ Commit & Push Jenkinsfile
```sh
git add Jenkinsfile
git commit -m "Add Jenkinsfile"
git push origin react-app
```

### 7️⃣ Menjalankan Pipeline
1. **Buka Jenkins → Blue Ocean UI**
2. **Jalankan Pipeline**
3. **Pastikan semua tahap berhasil (Build & Test)** ✅

### 🔄 Mengelola Jenkins
**Menjalankan ulang Jenkins jika dihentikan:**
```sh
docker start jenkins-blueocean jenkins-docker
```
**Menghentikan Jenkins:**
```sh
docker stop jenkins-blueocean jenkins-docker
```

## 🎯 Kesimpulan
Dengan mengikuti langkah-langkah di atas, Anda telah berhasil mengimplementasikan **CI Pipeline dengan Jenkins**. Pipeline ini memastikan bahwa setiap perubahan pada aplikasi diuji dan dibangun secara otomatis, meningkatkan efisiensi dalam pengembangan perangkat lunak. 🚀

---
**📢 Catatan:** Jika mengalami error, cek log Jenkins dan pastikan semua dependensi telah diinstal dengan benar.

Happy coding! 🎉
