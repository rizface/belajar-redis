![REDIS](https://redis.io/images/redis-white.png)

REDIS / Remote Dictionary Server, adalah sistem basis data key-value berbasis memory (data disimpan di RAM)

## KEY VALUE DATABASE
- key-value adalah paradigman dimana data disimpan dalam format pair key-value
  - nama: fariz -> ini key-value
  - {nama:"fariz",alamat:"indonesia"} -> json juga key-value

- redis meyimpan data di memory, jadi besaran data yang bisa disimpan redis tergantung dari size memori(RAM)

## KAPAN KITA BUTUH REDIS 
- redis tidak wajib digunakan saat aplikasi pertama kali dibuat
- untuk menggunakan redis, kita perlu melihat kasus secara mendetail


### MASALAH YANG BISA DIHADAPI MENGGUNAKAN REDIS
- ketika database utama lambat(PARAH), bisa jadi karena query lambat, dalam kasus ini redis bisa berperan menyimpan hasil query yang pertama kali, jadi request selanjutnya tidak perlu mengambil data langsung dari database tapi mengambil data dari cache redis
- ketika aplikasi lain lambat(INTEGRASI DENGAN APLIKASI ORANG LAIN), response pertama akan disimpan diredis
- ketika butuh delayed job


## REDIS SERVER VS REDIS CLI
- saat install redis sebenarnya ada 2 aplikasi yang ikut terinstall, yaitu redis server dan redis cli
- redis server adalah service untuk redis itu sendiri
- redis cli adalah cli client untuk connect ke redis
- connect redis cli ke redis server
```shell
redis-cli -h <host> -p <port>
```
- ping, bergunan untuk mengecek apakah kita connect ke server, jika balasan PONG maka kita sudah connect

## CONFIGURATION FILE
- secara default redis tidak butuh file config, tapi ini tidak boleh dilakukan dalam production
- pada kenyataanya jika menjalankan redis kita harus menggunakan file konfigurasi agar redis lebih aman
- file config redis bisa didownload [disini](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwj_mf7_h5D1AhW7SmwGHZN2BtUQFnoECAwQAQ&url=https%3A%2F%2Fdownload.redis.io%2Fredis-stable%2Fredis.conf&usg=AOvVaw3_kFM1q_S7L9omTrS8uL23)

script dibawah dijalankan ketika ingin menggunakan file konfigurasi pada redis
```shell
version: '3.8'

services:
  redis-config:
    container_name: redis-config
    image: redis
    ports:
      - "6379:6379"
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf
```

## DATABASE
- redis menggunakan angka sebagai nama database nya
- 0 adalah database default
- jumlah maksimal database bisa diatur pada file config
- select<noDatabase>, digunakan untuk pindah database
```shell
select 1
// response -> ok
select 16
// response -> ERR DB index is out of range
```

## STRINGS

- set, digunakan untuk input data ke redis 
- mset, digunakan untuk input beberapa data
- get, digunakan untuk mendapatkan data yang disimpan pada redis menggunakan key
- mget, digunakan untuk menampilkan value dari beberapa key
- exists, digunakan untuk memeriksa apakah key ada pada database
- del, digunakan untuk mengahapus data
- append, digunakan untuk menambahkan value ke key
- keys, digunakan untuk menampilkan semua key yang memenuhi pattern

```shell
set nama fariz // key : nama, value : Fariz
mset nama fariz alamat indonesia // set nama to fariz and alamat to indonesia
get nama // return nil if key if not exists
mget nama alamat // return value of nama and alamat
exists nama // check if the key is exists, return 1 if key exists, and 0 if key not exists
del nama // delete data base on key,return number of deleted data
keys nama* // will return all of keys that contains nama
keys namaD* // will return all of keys that contains namaD

// before -> nama: fariz
append nama " Al Farizzi"
// after -> nama: "fariz Al Farizzi"
```

## EXPIRATION
- secara default redis akan meyimpan data secara permanen
- di redis ada fitur expiration yang memungkinkan data dihapus otomatis dalam rentang waktu tertentu

### Operasi Expiration Data String
- expire <key> <seconds>, menambahkan expire ke data yang sudah ada 
- setex <key> <seconds> <value>, menambahkan expire pada data yang akan dibuat
- ttl <key>, mendapatkan sisa waktu dari suatu key

```shell
// set nama
set nama fariz
// set expiration to nama
expire nama 10 -> set 10 seconds to nama

// set expire while set key and value
setex namaDua 10 fariz // set 10 seconds to namaDua
```

## INCREMENT & DECREMENT 
increment dan decrement ini banyak digunakan ketika ada perubahan data yang sangat cepat, dan dikhawatirkan menjadi tidak konsisten karena *Race Condition*

### Operasi Increment & Decrement
1. incr key, menambah angka yang sudah ada dengan 1
2. decr key, mengurangi angka yang sudah ada dengan 1
3. incrby <key> <value>, menambah angka yang sudah ada sesuai keinginan
4. decrby <key> <value>, mengurangi angka yang sudah ada sesuai keinginan

```shell
set umur 10

// increment
incr umur // increment umur to 11

// decrement
decr umur // decrement umur to 10

// increment by
incrby umur 10 // increment umur to 20

// decrement by
decrby umur 9 // decrement umur to 11
```

## FLUSH
redis memiliki fungsi untuk menghapus seluruh data yang ada pada database

### Operasi flush
- flushdb, menghapus seluruh data pada database yang sedang kita gunakan
- flushall, menghapus seluruh data pada redis, tanpa mengenal database yang digunakan

```shell
// delete data from database that currently in use
flushdb

// delete data from all of databases
flushall
```

## PIPELINE
pada redis setiap satu request akan secara langsung mendapatkan satu response, dalam beberapa kasus kita perlu melakukan operasi sekaligus banyak jika dilakukan dengan yang cara yang biasa maka akan sangat memakan waktu, bayangkan jika kita perlu input 1.000.000 data maka kita akan mendapatkan 1.000.000 response juga, ini sangat tidak disarankan.
pada kasus seperti ini disarankan menggunakan pipeline, jika menggunakan pipeline maka response akan diberikan diakhir jika semua proses sudah selesai.

```shell
cat *namaFile* | redis-cli -h localhost --pipe
```

## TRANSACTION
transaction adalah proses mengirimkan beberapa perintah, dan akan dianggap berhasil jika semua perintah berhasil, jika ada satu saja perintah yang gagal maka semuanya akan dianggap gagal
- multi, digunakan untuk memulai transaction
- exec, digunakan untuk menerima semua transaction
- discard, digunakan untuk membatalkan semua transaction

```shell
multi // begin transcation
set nama fariz // will return QUEUED
set alamat indonesia // will return QUEUED 

// just run on of commands below
exec // to execute all command
discard // to cancel all command
```

## MONITOR
monitor ada fitur melakukan monitor terhadap semua request yang masuk ke redis server

```shell
monitor // this will make redis start monitor all incoming request
```

## SERVER INFORMATION
- info, mendapatkan data statistic tentang info
- config, mendapatkan config redis
- slowlog, mengembalikan slow query

```shell
info // will return server information

// config
config get <pattern> // will return configuration value
config get databases // example

// slowlog
slowlog get <subcommand> // will return slow query 
slowlog get set // example  
```

## CLIENT CONNECTION
- client list, untuk mendapatkan list client yang tersambung ke redis server
- client id, untuk mendapatkan id yang kita gunakan saat terkoneksi ke redis server
- client kill <ip>:<port>, menghapus koneksi client

```shell
client list // will return list of clients connected to server
// example return value
id=7 
addr=127.0.0.1:48732  // use this when kill client
laddr=127.0.0.1:6379 
fd=7 
name= 
age=104 
idle=0 
flags=N 
db=0 
sub=0 
psub=0 
multi=-1 
qbuf=26 
qbuf-free=40928 
argv-mem=10 
obl=0 
oll=0 
omem=0 
tot-mem=61466 
events=r 
cmd=client 
user=default 
redir=-1

client id // will return client id that we are using
client kil <ip>:<port> // will kill client that using listed ip and port, we can get target ip and port with key "addr"
// example
client kill 127.0.0.1:48732
```

## SECURITY
secara default redis akan menerima request dari mana saja, dan redis bisa di akses dari public secara default.
namun redis memiliki *Second Layer* yaitu mode protected, secara default protectednya aktif, artinya walau redis bisa diakses dari manapun, tapi hanya akan men-eksekusi request dari localhost

- protected-mode, berguna untuk mencegah request dari luar vm untuk dieksekusi
- bind, berguna untuk membatasi siapa yang bisa terkoneksi, apakah terkoneksi dari luar vm atau tidak

```shell
// jika config dibawah di uncomment maka redis hanya akan bisa diakses dari localhost
bind 127.0.0.1 -::1

// jika dicomment maka redis bisa diakses dari manapun
/*
  walaupun binding di comment dan redis menerima koneksi, tapi redis tidak akan melakukan ekseskusi terhadap perintah yang datang dari luar
*/
#bind 127.0.0.1 -::1

```

**Note :**
kenapa 2 container bisa tersambung tanpa harus mengatur network terlebih dahulut ?
itu  dikarenakan jika container dibuat dalam 1 file docker-compose yang sama otomatis berada dalam network yang sama

## AUTHENTICATION
authentication adalah proses verifikasi identitas, ini digunakan untuk verifikasi apakah yang mengakses redis adalah credential yang telah didefinisikan di redis.conf atau bukan.
proses autentikasi di redis sangat cepat, pastikan / disarankan password sepanjang mungkin agar terhindar dari bruteforce

```shell
// cara membuat user baru
user <username> on <authorization> ~* ><password>

// buat user default dulu sebelum bisa autentikasi menggunakan user lain
user default on +@connection 
// example 
user fariz on +@all ~* >rahasia

// cara melakukan autentikasi
redis-cli -h <host> -p <port>
// jika sudah masuk
auth <username> <password>
```

## AUTHORIZATION
authorization adalah proses memberi hak akses ke user yang telah berhasil autentikasi.

// ACL Command
https://redis.io/topics/acl

// ACL Category
https://redis.io/commands/acl-cat

1. ~jobs:*, akun hanya bisa memanipulasi data dengan keys jobs
2. ~*, kun bisa melakukan manipulasi ke semua keys 
3. ACL CAT, digunakan untuk melihat category apa saja yang tersedia
4. ACL CAT <category>, digunakan untuk melihat command apa saja yang tersedia dalam category

```shell
// user fariz bisa melakukan apa saja terhadapa data apapun
user fariz on +@all ~* >rahasia

// user fariz hanya bisa melakukan perintah yang ada dalam category read terhadap data dengan key makanan
user fariz on +@read ~makanan:* >rahasia

// user fariz hanya bisa melakukan perintah mget terhadap data yang memiliki key makanan
user fariz on +mget ~makanan:* >rahasia 

// user fariz tidak bisa melakukan write data apapun
user fariz on -@write ~* > rahasia

// user fariz tidak bisa membaca data apapun
user fariz on -@read ~* >rahasia

// user fariz7 hanya bisa melakukan perintah dalam category write,dengan key makanan
user fariz7 on +@write ~makanan:* >rahasia

// user fariz8 hanya bisa menjalankan perintah mset ke data dengan key makanan
user fariz8 on +mset ~makanan:* >rahasia
```

## EVICTION 
eviction adalah fitur di redis untuk menghapus data secara otomatis(sesuai strategi yang dipilih) jika penyimpanan redis sudah penuh(RAM PENUH)
ada 2 syarat sebelum menggunakan eviction
1. harus melakukan set terhadapat maxmemory pada file config
2. harus memilih strategi apa yang akan digunakan untk menghapus data

```shell
maxmemory <bytes>
// example
maxmemory 100gb
maxmemory 100mb

maxmemory-policy <eviction>
// example
maxmemory-policy allkeys-lfu
maxmemory-policy volatile-ttl
```

### LIST OF EVICTION
*LRU : LEAST RECENTLY USED (DATA YANG TERAKHIR KALI DIGUNAKAN)*
*LFU : LEAST FREQUENTLY USED(DATA YANG PALING JARANG DIGUNAKAN)*

1. volatile-lru, menghapus data yang terakhir digunakan (yang memiliki expire)
2. allkeys-lru, menghapus data yang terakhir yang digunakan (tidak perlu ada expire)
3. volatile-lfu, menghapus data yang paling jarang digunakan (yang memiliki expire)
4. allkeys-lfu, menghapus data yang paling jarang digunakan (tidak perlu ada expire)
5. volatile-ttl, menghapus data yang expire-nya paling dekat
6. noeviction, hanya mengembalikan error