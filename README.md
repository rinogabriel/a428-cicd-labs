📌 Langkah-Langkah Membangun CI Pipeline dengan Jenkins
🔹 1. Menjalankan Jenkins di Docker
Untuk memulai, kita akan menjalankan Jenkins di dalam Docker container. Jenkins digunakan untuk mengotomatiskan proses build, test, dan deployment aplikasi.

🛠️ Langkah-langkah:
Buka Terminal/CMD di komputer Anda.

Buat jaringan bridge di Docker dengan perintah berikut:

sh

docker network create jenkins
Jalankan Docker in Docker (DinD) untuk Jenkins, karena kita akan menjalankan aplikasi dalam container di dalam container lain.

sh

docker run --name jenkins-docker --detach --privileged --network jenkins \
--network-alias docker --env DOCKER_TLS_CERTDIR=/certs \
--volume jenkins-docker-certs:/certs/client --volume jenkins-data:/var/jenkins_home \
--publish 2376:2376 --publish 3000:3000 --restart always docker:dind --storage-driver overlay2
Penjelasan parameter penting:

--name jenkins-docker → Menamai container sebagai jenkins-docker

--detach → Menjalankan container di latar belakang.

--privileged → Memungkinkan akses ke Docker daemon.

--network jenkins → Menghubungkan container ini ke jaringan jenkins.

--publish 2376:2376 → Mengekspos Docker daemon ke port 2376.

--restart always → Jika container mati, maka otomatis restart.

Buat Dockerfile untuk menjalankan Jenkins dengan plugin Blue Ocean.

sh

nano Dockerfile
Isi Dockerfile:

dockerfile

FROM jenkins/jenkins:2.346.1-jdk11
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
https://download.docker.com/linux/debian $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean:1.25.5 docker-workflow:1.28"
Build image Jenkins dengan Blue Ocean:

sh

docker build -t myjenkins-blueocean:2.346.1-1 .
Jalankan Jenkins container dengan Blue Ocean UI:

sh

docker run --name jenkins-blueocean --detach --network jenkins \
--env DOCKER_HOST=tcp://docker:2376 --env DOCKER_CERT_PATH=/certs/client \
--env DOCKER_TLS_VERIFY=1 --publish 8080:8080 --publish 50000:50000 \
--volume jenkins-data:/var/jenkins_home --volume jenkins-docker-certs:/certs/client:ro \
--volume "$HOME":/home --restart=on-failure \
--env JAVA_OPTS="-Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true" \
myjenkins-blueocean:2.346.1-1

🔹 2. Menyiapkan Jenkins Wizard
Sebelum menggunakan Jenkins, kita perlu melakukan konfigurasi awal.

🛠️ Langkah-langkah:
Akses Jenkins melalui browser:

arduino

http://localhost:8080
Ambil password administrator dari Jenkins log:

sh

docker logs jenkins-blueocean
Masukkan password di halaman Unlock Jenkins.

Install Suggested Plugins untuk Jenkins.

Buat admin user dan simpan konfigurasi.

Mulai menggunakan Jenkins!

🔹 3. Fork dan Clone React App Repository
Jenkins akan digunakan untuk mengotomatiskan proses build dan test aplikasi React.

🛠️ Langkah-langkah:
Login ke GitHub dan fork repository aplikasi React dari Dicoding Academy.

Clone repository ke komputer lokal:

sh

git clone -b react-app https://github.com/USERNAME-AKUN-GITHUB-ANDA/a428-cicd-labs.git
Buka proyek dengan Visual Studio Code.

🔹 4. Membuat Pipeline Project di Jenkins
Pipeline akan dibuat untuk mengelola proses CI/CD.

🛠️ Langkah-langkah:
Buka Jenkins dan klik Create a job.

Masukkan nama pipeline (misal: react-app).

Pilih Pipeline dan klik OK.

Pada Pipeline script from SCM:

Pilih Git.

Masukkan path repository lokal:

bash

/home/Documents/Belajar_Implementasi_CICD/Jenkins/a428-cicd-labs
Ubah Branch Specifier menjadi:

bash

*/react-app
Klik Save.

🔹 5. Membuat Jenkins Pipeline dengan Jenkinsfile
Jenkinsfile digunakan untuk mendefinisikan pipeline sebagai kode.

🛠️ Langkah-langkah:
Buat file baru bernama Jenkinsfile di root folder proyek.

Tambahkan kode berikut:

groovy

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
    }
}
Simpan dan commit file Jenkinsfile:

sh

git add .
git commit -m “Add initial Jenkinsfile”
Jalankan pipeline di Blue Ocean.

🔹 6. Menambahkan Tahapan Testing
Kita perlu menambahkan tahap Test dalam Jenkinsfile.

🛠️ Langkah-langkah:
Edit Jenkinsfile, tambahkan:

groovy

stage('Test') {
    steps {
        sh './jenkins/scripts/test.sh'
    }
}
Simpan dan commit perubahan:

sh

git add .
git commit -m “Add Test stage”
Jalankan ulang pipeline di Blue Ocean.

🔹 7. Mengelola Jenkins
🛠️ Menjalankan kembali Jenkins jika dihentikan:
sh

docker start jenkins-blueocean jenkins-docker
🛠️ Menghentikan Jenkins:
sh

docker stop jenkins-blueocean jenkins-docker
Akses kembali di:

arduino

http://localhost:8080


🎯 Kesimpulan
Anda telah berhasil membangun CI Pipeline menggunakan Jenkins untuk aplikasi React. Dengan pipeline ini, setiap perubahan pada kode akan di-build dan diuji secara otomatis, sehingga meningkatkan efisiensi dalam pengembangan perangkat lunak. 🚀
