# Implementasi Cassandra DB pada Ubuntu
Sebelumnya kita harus menyiapkan vm terlebih dahulu untuk node-node yang akan di pakai dalam tutorial kali ini. Untuk membuat vm kita tinggal melakukan `vagrant up` pada folder dimana kita menyimpan file `Vagrantfile`. Nantinya akan ada 2 vm yang terbuat dimana semuanya menggunakan os `ubuntu 14.04` dan spesifikasi seperti berikut :   

| No | Hostname | IP Adrress |
| :---: | --- | --- |
| 1 | cnode1 | 192.168.1.11 |
| 2 | cnode2 | 192.168.1.12 |

## Daftar Isi
1. [Instalasi Cassandra Single Node](#1-instalasi-casandra-single-node)   
   1.1 [Instalasi Oracle Java Virtual Machine](#11--instalasi-oracle-java-virtual-machine)   
   1.2 [Instalasi Cassandra](#12-instalasi-cassandra)   
   1.3 [Cek Cassandra sudah terinstal](#13-cek-cassandra-sudah-terinstal)   
2. [Instalasi Cassandra Multi Nodes](#2-instalasi-cassandra-multi-nodes)   
   2.1 [Instalasi Cassandra pada setiap node](#21-instalasi-cassandra-pada-setiap-node)   
   2.2 [Menghapus data default](#22-menghapus-data-default)   
   2.3 [Konfigurasi Cassandra cluster](#23-konfigurasi-cassandra-cluster)   
   2.4 [Menyalakan service Cassandra kembali](#24-menyalakan-service-cassandra-kembali)   
   2.5 [Testing](#25-testing)   
3. [Import Data](#3-import-data)   
4. [CRUD pada Cassandra DB](#4-crud-pada-cassandra-db)   
5. [Referensi](#5-referensi)   

## 1. Instalasi Casandra Single Node
Disini kita akan menggunakan `cnode1`
### 1.1  Instalasi Oracle Java Virtual Machine
Cassandra memerlukan Oracle Java SE Runtime Environment (JRE) untuk dijalankan, oleh karena itu kita harus menginstall Oracle JRE terlebih dahulu.
1. Menginstall `software-properties-common` agar bisa menggunakan command untuk menambahkan repository
    ```bash
    sudo apt-get update
    sudo apt-get install software-properties-common
    ```
2. Menambahkan repository java
   ```bash
   sudo add-apt-repository ppa:webupd8team/java
   ```
3. Menginstall Oracle JRE
    ```bash
    sudo apt-get update
    sudo apt-get install oracle-java8-set-default
    ```
4. Cek java sudah terinstall
   ```bash
   java -version
   ```   
   ![gambar 1](https://github.com/tamtama17/Instalasi-Cassandra/blob/master/gambar/gambar1.png)   
### 1.2 Instalasi Cassandra
1. Menambahkan repository Cassandra
   ```bash
   echo "deb http://www.apache.org/dist/cassandra/debian 311x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
   ```
2. Menambahkan keys dari repository Cassandra
   ```bash
   curl https://www.apache.org/dist/cassandra/KEYS | sudo apt-key add -
   ```
3. Mengupdate repository
   ```bash
   sudo apt-get update
   ```
4. Menginstall Cassandra
   ```bash
   sudo apt-get install cassandra
   ```
### 1.3 Cek Cassandra sudah terinstal
1. Cek service Cassandra berjalan
   ```bash
   sudo service cassandra status
   ```
2. Cek status node
   ```bash
   sudo nodetool status
   ```
3. Masuk ke Cassandra
   ```bash
   cqlsh
   ```   
![gambar 2](https://github.com/tamtama17/Instalasi-Cassandra/blob/master/gambar/gambar2.png)   

## 2. Instalasi Cassandra Multi Nodes
### 2.1 Instalasi Cassandra pada setiap node
Sebelumnya instalasi cassandra terlebih dahulu di setiap node. Sudah ada script `bootstrap-cassandra.sh` yang bisa digunakan untuk instalasi Cassandra seperti pada bagian pertama, tinggal dijalankan saja dengan cara :
```bash
bash /vagrant/bootstrap-cassandra.sh
```
Untuk tahap-tahap selanjutnya dilakukan di semua node.
### 2.2 Menghapus data default
Untuk menghapus data default, kita harus mematikan service nya terlebih dahulu dengan cara :
```bash
sudo service cassandra stop
```
Lalu menghapus semua data default dengan cara :
```bash
sudo rm -rf /var/lib/cassandra/data/system/*
```
### 2.3 Konfigurasi Cassandra cluster
File konfigurasi Cassandra berada di directory `/etc/cassandra`, pada file `cassandra.yaml`. Kita harus merubah beberapa config dengan cara :
```bash
sudo nano /etc/cassandra/cassandra.yaml
```
Hal yang perlu dirubah adalah sebagai berikut :
```bash
.....
cluster_name: 'Test Cluster'
.....
seed_provider:
  - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    parameters:
         - seeds: "192.168.1.11,192.168.1.12"
.....
listen_address: 192.168.1.11 # setiap node berbeda, sesuai ip node masing-masing
.....
rpc_address: your_server192.168.1.11 # setiap node berbeda, sesuai ip node masing-masing_ip
.....
endpoint_snitch: GossipingPropertyFileSnitch
.....
```
Lalu, diakhir file tambahkan :
```bash
auto_bootstrap: false
```
Jika sudah selesai simpan configurasi baru.
### 2.4 Menyalakan service Cassandra kembali
Nyalakan kembali service Cassandra dengan cara :
```bash
sudo service cassandra start
```
Cek status service apakah sudah menyala dengan cara :
```bash
sudo service cassandra status
sudo nodetool status
```   
![gambar 3](https://github.com/tamtama17/Instalasi-Cassandra/blob/master/gambar/gambar3.png)   
### 2.5 Testing
Jika multi nodes sudah berhasil seharusnya kita bisa mengakses node 1 dari node lainnya. Cek dengan cara :
```bash
cqlsh [ip node tujuan] 9042
```   
![gambar 4](https://github.com/tamtama17/Instalasi-Cassandra/blob/master/gambar/gambar4.png)      
Terlihat `cnode1` yang memiliki ip `192.168.1.11` bisa mengakses Cassandra `cnode2` dengan ip `192.168.1.12`.

## 3. Import Data
### 3.1 Dataset
Dataset yang digunakan adalah dataset 100 lagu terbaik dari aplikasi `spotify` pada tahun 2018. Dataset bisa didapatkan [disini](https://www.kaggle.com/nadintamer/top-spotify-tracks-of-2018).
Deskripsi data :
1. Spotify URI setiap lagu, dimana setiap lagu berbeda.
2. Judul lagu
3. Artis/penyanyi
4. Fitur audio (seperti tempo, kunci, durasi, dll)
### 3.2 Membuat `KEYSPACE`
`KEYSPACE` pada Cassandra adalah seperti `database` pada MySQL. Cara membuat `keyspace` adalah dengan syntax berikut :
```sql
CREATE KEYSPACE [nama keyspace]
WITH REPLICATION = {
  'class' = '[nama kelas]',
  'replication' = [jumlah replikasi]
};
```
Untuk `class`, defaultnya ada 2 yang sering digunakan pada Cassandra yaitu `SimpleStrategy` dan `NetworkTopologyStrategy`. Untuk melihat semua `keyspace` yang ada gunakan syntax :
```sql
DESCRIBE KEYSPACE;
```   
![gambar 5](https://github.com/tamtama17/Instalasi-Cassandra/blob/master/gambar/gambar5.png)   
### 3.3 Membuat `TABLE`
Untuk membuat `table` pada Cassandra caranya hampir sama pada MySQL dengan cara :
```sql
CREATE TABLE [nama table] (
  [nama kolom] [tipe data],
  [nama kolom] [tipe data],
  ...
  [nama kolom] [tipe data],
  PRIMARY KEY([nama kolom])
);
```   
![gambar 6](https://github.com/tamtama17/Instalasi-Cassandra/blob/master/gambar/gambar6.png)   
### 3.4 Import data csv
Untuk meng-import dataset bertipe csv pada Cassandra adalah dengan cara :
```sql
COPY [nama table]([nama kolom-1],[nama kolom-2],[nama kolom-3],...,[nama kolom-n]) FROM '[nama file]' WITH [opsi copy]
```   
![gambar 7](https://github.com/tamtama17/Instalasi-Cassandra/blob/master/gambar/gambar7.png)   
Cek data sudah masuk   
![gambar 8](https://github.com/tamtama17/Instalasi-Cassandra/blob/master/gambar/gambar8.png)   

## 4. CRUD pada Cassandra DB
Untuk CRUD atau DML pada Cassandra menggunakan bahasa CQL(Cassandra Query Language) yang hampir mirip dengan MySQL.
### 4.1 Create
Syntax CQL untuk membuat data baru :
```sql
INSERT INTO [nama tabel]([nama kolom-1],[nama kolom-2],[nama kolom-3],...,[nama kolom-n])
VALUES ([nilai nama kolom-1],[nilai nama kolom-2],[nilai nama kolom-3],...,[nilai nama kolom-n]);
```   
![gambar 9](https://github.com/tamtama17/Instalasi-Cassandra/blob/master/gambar/gambar9.png)   
### 4.2 Read
Syntax CQL untuk menampilkan data :
```sql
SELECT [nama kolom-1],[nama kolom-2],[nama kolom-3],...,[nama kolom-n]
FROM [nama tabel]
WHERE [klausa where];
```   
![gambar 10](https://github.com/tamtama17/Instalasi-Cassandra/blob/master/gambar/gambar10.png)   
#####Catatan penting :
Untuk menggunakan klausa `where` kita harus menggunakan kolom yang menjadi `primary key`.
### 4.3 Update
Syntax CQL untuk meng-update data :
```sql
UPDATE [nama tabel]
SET [nama kolom-1] = [nilai baru kolom-1], [nama kolom-2] = [nilai baru kolom-2], ..., [nama kolom-n] = [nilai baru kolom-n]
WHERE [klausa where];
```   
![gambar 11](https://github.com/tamtama17/Instalasi-Cassandra/blob/master/gambar/gambar11.png)   
### 4.4 Delete
Syntax CQL untuk menghapus data :
```sql
DELETE
FROM [nama tabel]
WHERE [klausa where];
```   
![gambar 12](https://github.com/tamtama17/Instalasi-Cassandra/blob/master/gambar/gambar12.png)   

## 5. Referensi
- https://www.digitalocean.com/community/tutorials/how-to-install-cassandra-and-run-a-single-node-cluster-on-ubuntu-14-04
- https://www.digitalocean.com/community/tutorials/how-to-run-a-multi-node-cluster-database-with-cassandra-on-ubuntu-14-04
- http://cassandra.apache.org/download/
- http://cassandra.apache.org/doc/latest/
- https://askubuntu.com/questions/593433/error-sudo-add-apt-repository-command-not-found
- https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet