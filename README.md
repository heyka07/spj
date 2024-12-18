# SISTEM PERTAHANAN JARINGAN - HARDENING SERVER KELOMPOK 8
Disini, kami akan membuat Web Server lalu melakukan penyerangan Brute-Force menggunakan tools hydra serta melakukan Hardening. Anggota Kelompok kami terdiri dari:

Raka Tirta Wahyudi - 23.83.0991.
Zhewa al varihy mandeva - 23.83.1012.
Angga ahdi prasetya - 23.83.1033.

## daftar isi
1. Membuat Web Server
2. Brute-Force before secure
3. Hardening/Mitigasi Server dari serangan
4. Brute-force aftter Secure
5. SSH Key Authentication
   
## 1. Membuat WEB Server
1. Install SSH 
   ```
   sudo apt install openssh-server
   ```
2. pastikan berjalan (bisa dimodif di html)
   ![image](https://github.com/user-attachments/assets/26abdd8d-5b5a-40b9-ad49-24a43d33e062)

3. Konfigurasi Web Server Apache dengan masuk ke
  ```
  sudo nano /var/www/html/index.html
  ```
  konfigurasi sebagai berikut
  ```
  <!DOCTYPE html>
  <html>
  <head>
      <title>Welcome to My Web Server!</title>
  </head>
  <body>
      <h1>Hello, World! This is my first web server.</h1>
  </body>
  </html>
  ```
  ![image](https://github.com/user-attachments/assets/08c49102-9114-4f6f-a5d6-8010276544db)

4. Restart Apache untuk melihat perubahan
  ```
  sudo systemctl restart apache2
  ```
5. Buat Direktori untuk melakkukan simulasi serangan, lalu edit direktori tersebut
  ```
  sudo mkdir /var/www/html/private
  sudo nano /var/www/html/private/info.html
  ```
  lalu bisa ditambahkan
  ```
  <h1>data penting militer tier 1</h1>
  <p>Do not share this information!</p>
  <img src="https://thumb.viva.co.id/media/frontend/thumbs3/2019/12/03/5de5d8b7aa964>7aa964-seekor-anjing-  mengendarai-sepeda-motor_1265_711.jpg" alt="naik ajg">
  ```
6. Akses file tersebut melalui browser, masukkan link <alamat ip kamu>/private/info.html.
  ![image](https://github.com/user-attachments/assets/e656b4dc-e5c2-4033-a64e-6205f5a01300)

## 2. Menyerang Server Before Secure Menggunakan Hydra
1. Lakukan serangan menggunakan tools hydra, pastikan kamu sudah memiliki passlist.txt. Jika serangan berhasil, outpunya akan sesuai dengan gambar yang dibawah.
   ```
   hydra -l <username server kamu> -P passlist.txt ssh://<alamat ip web server kamu>
   ```
   ![WhatsApp Image 2024-12-19 at 02 23 14_227d9517](https://github.com/user-attachments/assets/a9c333cd-a370-4f1e-bfe6-b0e0956ad681)

## 3. Hardening
### 1. Proteksi Direktori /private
1. Buat file Password. Untuk yang bagian admin, dapat kamu ubah sesuai dengan username kamu.
   ```
   sudo htpasswd -c /etc/apache2/.htpasswd admin
   ```
2. Verifikasi File Password
   ```
   cat /etc/apache2/.htpasswd
   ```
   ![image](https://github.com/user-attachments/assets/a7ac106d-add8-4a5a-bbc4-cf9134819d4f)
3. Edit File konfigurasi Apache
   ```
   sudo nano /etc/apache2/apache2.conf
   ```
   lalu tambahkan
   ```
   <Directory /var/www/html/private>
    AuthType Basic
    AuthName "Restricted Content"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
   </Directory>
   ```
   ![image](https://github.com/user-attachments/assets/4de88617-5585-422d-8be5-fc4be1a7c30a)
4. Restart Apache
   ```
   sudo systemctl restart apache2
   ```
5. Ketik <alamat ip kamu>/private pada browser
   Jika Username dan Password yang dimasukkan sesuai, kamu dapat masuk ke direktori /private.
   ![image](https://github.com/user-attachments/assets/896ff0ad-56a4-48f5-9462-b689d80588a8)
   jika gagal maka akan keluar
   ![Screenshot 2024-12-19 030046](https://github.com/user-attachments/assets/6cfe90b3-4264-46e5-9d33-a3153f2be5c9)
## 2. Hide Status Apache dan Port
1. Buka File konfigurasi Apache
   ```
   sudo nano /etc/apache2/conf-available/security.conf
   ```
   tambahkan
   ```
   ServerTokens Prod
   ServerSignature Off
   ```
   ![image](https://github.com/user-attachments/assets/e93ed0f3-24dd-42f4-8b84-6b52bd698251)
2. Restart Apache dan buka kembali direktori /private pada browser
   ```
   sudo systemctl restart apache2
   ```
   ![image](https://github.com/user-attachments/assets/a417be69-1ce2-412f-a11d-6e8298e09546)
## 3. Nonaktifkan Login Root
1. Masuk ke konfigurasi SSH
   ```
   sudo nano /etc/ssh/sshd_config
   ```
   Lalu ubah bagian PermitRootLogin menjadi PermitRootLogin no, Lalu restart SSH.
   ![image](https://github.com/user-attachments/assets/bb64a1b6-eb1d-4376-b1f2-0d88efd541a0)
## 4. Rate-Limiting
1. Install tools Fail2Ban
   ```
   sudo apt install fail2ban -y
   ```
2. Masuk ke file konfigurasi Fail2Ban
   ```
   nano /etc/fail2ban/jail.local
   ```
   lalu masukkan tetks berikut
   ```
   [sshd]
   enabled = true
   port = 22
   maxretry = 3
   bantime = 600
   findtime = 600
   ```
## 4. Brute-force after Hardening
1. Lakukan serangan brute-force menggunakan hydra, jika server kamu berhasil untuk di Hardening, maka Hydra tidak akan bisa menyerang server kamu.
   ```
   hydra -l <username kamu> -P passlist.txt ssh://192.168.100.72
   ```
   ![WhatsApp Image 2024-12-19 at 04 01 42_89da6ab1](https://github.com/user-attachments/assets/836de87a-07e3-46f7-9c98-c50e2366441c)

## 5. jika pasword masih bisa diserang

1. Gunakan SSH Key Authentication (Nonaktifkan Password Login)
Ganti autentikasi berbasis password dengan SSH Key Authentication.
Dengan metode ini, hanya user yang memiliki kunci privat yang cocok dengan kunci publik di server yang bisa login.
Langkah singkatnya:

Generate key pair:
```
ssh-keygen -t rsa -b 4096
```
Salin kunci publik ke server:
```
ssh-copy-id kelompok8@192.168.100.86
```
![image](https://github.com/user-attachments/assets/0b589ee4-2f3f-40f6-a973-e46cd22c2990)

Nonaktifkan password login di /etc/ssh/sshd_config:
```
sudo nano /etc/ssh/sshd_config
```
ubah menjadi PasswordAuthentication no
![image](https://github.com/user-attachments/assets/9e6661e9-504e-462d-87e8-e5e6922bf402)

Restart SSH:
```
sudo systemctl restart sshd
```
2. Batasi Percobaan Login dengan Fail2Ban
Install dan konfigurasi Fail2Ban untuk memblokir IP yang gagal login berkali-kali:
```
sudo apt install fail2ban
```
Konfigurasi SSH di /etc/fail2ban/jail.local:
```
[sshd]
enabled = true
maxretry = 3
bantime = 3600
findtime = 600
```
Restart Fail2Ban:
```
sudo systemctl restart fail2ban
```
4. Ubah Port Default SSH
Port default SSH adalah 22, yang sering menjadi target serangan brute-force. Ubah ke port lain di /etc/ssh/sshd_config:
```
Port 2222
```
Restart SSH:
```
sudo systemctl restart sshd
```
Pastikan firewall mengizinkan port baru tersebut:
```
sudo ufw allow 2222/tcp
```
5. Hanya Izinkan User Tertentu untuk SSH
Batasi akses hanya ke user tertentu dengan menambahkan di /etc/ssh/sshd_config:
```
AllowUsers kelompok8
```
Restart SSH:
```
sudo systemctl restart sshd
```
6. Gunakan Firewall untuk Memfilter Akses
Gunakan UFW (Uncomplicated Firewall) untuk hanya mengizinkan IP tertentu:
```
sudo ufw allow from 192.168.1.0/24 to any port 22
sudo ufw enable
```
Jika IP user sudah diketahui, ini akan mempersempit akses hanya dari jaringan yang dipercaya.

   



   



      
   

