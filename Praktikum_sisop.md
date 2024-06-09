## Langkah 1: Persiapan Server
1. **Setup server**: Gunakan penyedia layanan cloud (seperti AWS, DigitalOcean, atau lainnya) atau siapkan server fisik.
2. **Instalasi Sistem Operasi**: Pasang distribusi Linux seperti Ubuntu Server.
3. **Setting Network:** Setting Network dari Nat ke Bridger Adapter jika menggunakan virtual Box

## Langkah 2: Konfigurasi SSH
1. **Instalasi OpenSSH Server:**
   
   ```bash
   sudo apt update
   sudo apt install openssh-server
   ```

2. **Konfigurasi SSH untuk menggunakan port 9005:**
   
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

- lalu ubah port delfault 22 menjadi 9005 hapus tanda pagarnya
  
   ```
   Port 9005
   ```
   
3. **Restart layanan SSH:**
   ```bash
   sudo systemctl restart ssh
   ```

4. **Cek Ping dan Ping server:**
   
- Cek ip (pilih salah satu)
   ```bash
   ifconfig
   hostname I 
   ip a
   ```

- Uji ping
   ```bash
   ping <ipadressserver>
   ```
   
5. **Coba Masuk ke Server:**
   
   ```bash
   ssh -p 9005 userr@ipadressserver_or_hostname
   ```

- Untuk cek server telah hidup
   ```bash
   sudo service ssh status
   ```

- Jika belum hidup
   ```bash
   sudo service ssh start
   ```

6. **Buat ssh public key di lokal komputer**
   
   ```shell
   ssh-keygen -t rsa -b 4096
   ```
   
- Masuk ke directory ssh dan liat menggunakan ls
  
   ```shell
   cd .ssh
   ```

7. **Copy public key ke ssh server buka menggunakan cat dan copy:**
8. 
- Buka public key di lokal komputer dan copy
   ```shell
   cat ~/.ssh/id_rsa.pub
   ```

- Masuk ke ssh server
   ```shell
   ssh -p 9005 user@ipadress
   ```
   
8. **Buat direktori ~/.ssh dan copy ke     authorized keys:**
   
   ```bash
   mkdir -p ~/.ssh
   nano ~/.ssh/authorized_keys
   ```

9.  **Setel izin ssh authorized keys:**

- Setel izin menggunakan chmod
  (untuk baca, tulius dan eksekusi)
   ```bash
   chmod 700 ~/.ssh 
   ```
-  (untuk baca dan tulis)
   ```bash
   chmod 600 ~/.ssh/authorized_keys
   ```

10.  **Masuk lagi ke konfig dan ubah passwordAuthentication:**
    
- Masuk ke konfig
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

- Hapus pagar dan ubah ke passwordAuthentication yes ke no
   ```
   passwordAuthentication no
   ```

11.  **Coba masuk lagi:**

- Masuk ke server menggunakan ssh atau remote os
   ```bash
   ssh -p 9005 user@ipadress_or_hostname
   ```


## Langkah 3: Membuat User Baru (guest) dan Hanya Bisa Akses ke Home Directory

1. **Membuat user baru:**
   
   ```bash
   sudo adduser guest
   ```
2. **cek user**
   ```bash
   cat etc/passwd
   ```

3. **Buat file 'guest', setting kepemilikan ke guest dan izin directory:**
   
   ```bash
   sudo mkdir /home/guest
   sudo chown guest:guest /home/guest
   sudo chmod 755 /home/guest
   ```
   - cek perizinan di
   ```bash
   ll
   ```
   - atau 
   ```bash
   ls -ld 
   ```

   drwxrwxrwx --> formatnya d(---)(---)(---)
   |Owner|Group|Other|
   |-----|-----|-----|
   |rwx|rwx|rwx|
   |4+2+1|4+2+1|4+2+1|

4. **Edit konfigurasi ssh:**
   
- Masuk ke sshd_config
   ```bash
   sudo /etc/ssh/sshd_config
   ```

- Tambahkan ini ke paling bawah
   ```
   Match User guest
      ChrootDirectory /home
      AllowTcpForwarding no
      X11Forwarding no
   ```
   - AllowTcpForwarding no Dan X11Forwarding no: Menonaktifkan penerusan TCP dan X11 untuk keamanan tambahan. 

5. **Restart ssh:**
   
   ```bash
   sudo systemctl restart ssh
   ```

6. **Verifikasi status ssh:**
   
   ```bash
   sudo service ssh status
   ```

7. **Tes akses direktori home dan guest:**
   
   ```bash
   cd /home/guest
   touch test.txt
   ls
   ```
8. **Tes akses diluar direktori home:**
   
   ```bash
   cd ../.. atau cd /
   ```
   masuk ke folder var atau yang lain selain folder direktori guest
   
   ``` bash
   cd var
   touch test.txt
   ```

   jika tidak bisa, tes akses diluar dirktori guest telah berhasil.
   
## Langkah 4: Setup Web Server dengan HTTPS

1. **install apache2:**
   
   ```bash
   sudo apt install apache2
   sudo systemctl status apache2
   ```
   
2. **Masuk ke dir index.html:**
   
   ```bash
   cd /var/www/html/
   ls
   ```

3. **Test website html dengan local host:**
   
   ```bash
   firefox index.html
   ```
   https://index.html (ubah menjadi) 127.0.0.1/index.html

4. **Ambil (copy) Website php:**
   
   github.com/Rizkirazkafi/testing-website

- Edit file php dan tambahkan (paste)
  
  ```bash
  sudo gedit index.php
  ```

1. **Test website php dengan local host:**
   
   ```bash
   firefox index.php
   ```
   https://index.php (ubah menjadi) 127.0.0.1/index.php

2. **Instal nmap (opsional):**
   ```bash
   sudo apt-get update
   sudo apt-get install nmap
   ```

3. **Cek port (opsional):**
   
   ```bash
   nmap localhost
   ```

4. **Ganti port web server tidak boleh sama dengan port ssh jadi biarkan saja port 80:**
    
   ```bash
   cd etc/apache2
   sudo nano ports.conf
   listen 80
   sudo service apache2 restart
   ```

5.  **Cek lagi menggunakan ipadrees server**
    
   ```bash
   cd /var/www/html
   ls
   firefox index.html
   ```

   https://index.html (ubah menjadi) ipadress/index.

   ```bash
   firefox index.php
   ```

   https://index.php (ubah menjadi) ipadress/index.php
   
## Langkah 5: Setup Storage Utama dan Backup Storage Server

note: jika menggunakan virtual box tambahkan disk di setting tambahkan 2 disk berukuran 10gb

1. **Cek disk yang tersedia:**
   ```bash
   lsblk
   lsblk -f
   ```

2. **Membuat Tabel partisi:**
   ```bash
   sudo cfdisk /dev/sdb
   sudo cfdisk /dev/sdc
   ```

- pilih gpt
- pilih new dan klik enter
- pilih opsi write dan klik enter
- jika selesai pilih opsi quit lau enter
  
3. **Memformat dengan ext4 dan melabelkan disk dengan 'main' dan 'backup':**
   
   ```bash
   sudo mkfs.ext4 -L main /dev/sdb1
   sudo mkfs.ext4 -L backup /dev/sdc1
   ```

4. **Buat Direktori Mount Point :**
   
   ```bash
   sudo mkdir -p /var/www/html
   sudo mkdir -p /mnt/backup
   ```

5. **Pasang Direktori dan Mount disk secara manual untuk memeriksa:**
   
   ```bash
   sudo mount /dev/sdb1 /var/www/html
   sudo mount /dev/sdc1 /mnt/backup
   ```

6. **Tambahkan disk ke fstab untuk mount otomatis saat boot:**
   
   ```bash
   sudo nano /etc/fstab
   ```
   - Tambahkan dan gunakan jarak spasi menggunakan **tab**
   ```
   /dev/sdb1  /var/www/html  ext4  defaults  0  2
   /dev/sdc1  /mnt/backup  ext4  defaults  0  2
   ```
7. **Verifikasi mount:**
   
   ```bash
   df -h 
   ```

8. **Buatlah Automatisasi Backup dengan cara:**
   
- Buat script Backup
   ```bash
   sudo nano /usr/local/bin/backup.sh
   ```

- Tambahkan baris berikut
   ```
   #!/bin/bash
   rsync -av --delete /var/www/html/ /mnt/backup/ &>> /var/log/backup.sh
   ```
   
- Simpan dan tutup file, lalu buat skrip dapat dieksekusi: 
   ```bash
   sudo chmod +x /usr/local/bin/backup.sh
   ```

- Menjalankan Script Secara Manual  : 
  ```bash
  sudo /usr/local/bin/backup.sh
  ```

- Memeriksa Log Backup:
  ```bash
  sudo cat /var/log/backup.sh
  ```

- Menjalankan Script Backup dengan automatis menggunakan Cron job (opsional)
  ```bash
  sudo crontab -e
  ```

- contoh:  baris berikut untuk menjalankan backup setiap hari pada jam 2 pagi: (opsional)
  ```bash
  0 2 * * * /usr/local/bin/backup.sh
  ```

- Cek dan verifikasi data  di dr backup
  ```bash
  ls /mnt/backup
  ```

9. **Jika gagal gunakan perintah ini untuk menghapus partisi:**
   
   ```bash
   sudo fdisk /dev/sdb
   sudo fdisk /dev/sdc
   ```

- tekan 'd' lalu enter untuk menghapus partisi
- tekan 'w' lalu enter untuk write dan keluar

-  Sebelum itu unmount terlebih  dahulu

   ```bash
   sudo umount /dev/sdb <direktori mount>
   sudo umount /dev/sdc <direktori mount>
   ```



   
