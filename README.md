# sharding-mongodb

Dokumentasi ini hanya bersifat sementara, dimana masih ada step yang tidak bisa.
case:
install server ubuntu spek ram 1 gb, mongo versi 4 community
***
Disini disediakan ip address private version dan nama server 

Shards : 
- shard1 xx.2.5
- shard2 xx.2.4
***
Config :
- config1 xx.2.3
- config2 xx.2.2
***
Router :
- router xx.2.1
***
ReplSet untuk Shard dikasih nama shard01
ReplSet untuk config dikasih nama configserver
*** 
Catatan : saya menggunakan virtualbox dengan semua server menggunakan 2 interface, interface 1 NAT untuk internet, interface 2 internal virtualbox,
hanya untuk server router saya menggunakan 3 interface, dimana interface 3 server router digunakan untuk host only, untuk koneksi ssh ke router,
kemudian dari router saya bikin `ssh-keygen` dan kemudian saya copy ke semua server tetangga `ssh-copy-id user@host`,
jika sudah maka saya akan mengkontrol semua server tetangga melalui server router.
***

#### Untuk server shard1, shard2, config1, dan config2
- Tambahkan masing-masih ip ke /etc/hosts dengan nama shard1 dan shard2
- Buat folder /data/db dan /data/configdb , ganti chown menjadi user ,contoh `chown developer /data/db /data/congidb`
-------
-------
### Untuk shard1 dan shard2
##### Masuk ke server shard1 dan shard2
- ketik perintah
`mongod --shardsvr --bind_ip 0.0.0.0`

** --bind_ip agar server yang bersangkutan bisa diakses oleh semua server tetangga

** default port adalah 27018
----------
----------
### Untuk config1 dan config2 dengan ReplSet configserver
##### Masuk ke server config1 dan config2
- ketik perintah
`mongod --configsvr --replSet configserver --bind_ip 0.0.0.0`

** --bind_ip agar server yang bersangkutan bisa diakses oleh semua server tetangga

** default port adalah 27019
----------
#### Masuk ke server config yang akan di jadikan primary, disini contoh 'config1'
- ketik perintah `mongo --port 27019` sampai masuk ke `>`
- tambahkan members dengan cara 
```
rs.initiate({ _id:"configserver", configsvr:true, version:1, members:[ {_id:0,host:"xx.2.2:27019"}, {_id:1,host:"xx.2.3:27019"} ] })
```
jika sudah maka ketik perintah `rs.status()` nanti ada pesan ada PRIMARY dan SECONDARY

-------------
-------------

### Untuk router
##### Masuk ke server router
- ketik perintah
`mongos --port 27020 --configdb configserver/xx.2.2:27019,xx.2.3:27019 --bind_ip 0.0.0.0`

** --bind_ip agar server yang bersangkutan bisa diakses oleh semua server tetangga

** port yang di daftarkan 27020
##### Masuk ke server router, dengan ssh yang baru
- ketik perintah `mongo xx.2.1:27020/admin` sampai masuk ke `>`
- tambahkan members dengan cara
`sh.addShard("xx.2.4:27018")` dan `sh.addShard("xx.2.5:27018")`
jika bisa harusnya ada pesan `OK` , jika tidak bisa berarti error sama dengan yang punya saya.
--------
### Masih didalam router
- Tambahkan enable sharding,
>`db.runCommand({enableSharding:"testDB"})` jika berhasil maka OK
- >`sh.shardCollection("testDB.testData",{"name":1})` jika berhasil maka OK
- >`sh.status()` kemudian `sh.getBalancerState()` maka akan `true`
- kemudian keluar dan lihat doscovernya
$`mongostate --discover --host xx.2.4:27018,xx.2.5:27018` jika berhasil maka akan berjalan
- jika sudah silahkan test dengan robomongo konek ke ip router dengan port router
------------
------------
Ada beberapa link yang menginspirasi saya.
- [https://blog.serverdensity.com/automating-partitioning-sharding-and-failover-with-mongodb/](https://blog.serverdensity.com/automating-partitioning-sharding-and-failover-with-mongodb/)
- [https://github.com/alexcomu/mongodb_howto/tree/master/04_sharding](https://github.com/alexcomu/mongodb_howto/tree/master/04_sharding)
- [https://www.youtube.com/watch?v=qYzYp1bPCPg](https://www.youtube.com/watch?v=qYzYp1bPCPg) Part 1
- [https://www.youtube.com/watch?v=Ge6EQXTC8VU](https://www.youtube.com/watch?v=Ge6EQXTC8VU) Part 2

silahkan kunjungi.
