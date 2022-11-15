# Writing & Presentation Test Week 7 #

# **Sequelize**

**Sequelize** adalah Node.js promise-based ORM untuk MySQL, PostgreSQL, SQLite, MSSQL dan database SQL lainnya. Sequelize berfungsi  untuk bekerja dengan database dan relasi-relasi di dalamnya. 
**ORM** (Object Relational Maping) adalah sebuah tool yang kita gunakan sebagai cara kita melakukan interaksi dengan database pilihan kita tanpa harus mempelajari domain spesific language kita hanya berinteraksi dengan pertara api yang sudah disediakan.

## Installing

Menginstall Sequelizenya terlebih dahulu dengan menggunakan command berikut :
```h
npm install --save sequelize
```
Anda juga harus menginstal driver secara manual untuk database pilihan Anda:
```h
# One of the following:
$ npm install --save pg pg-hstore # Postgres
$ npm install --save mysql2
$ npm install --save mariadb
$ npm install --save sqlite3
$ npm install --save tedious # Microsoft SQL Server
$ npm install --save oracledb # Oracle Database
```

## Connecting to a database

Untuk terhubung ke database, Anda harus membuat instance Sequelize. Ini dapat dilakukan dengan meneruskan parameter koneksi secara terpisah ke konstruktor Sequelize atau dengan melewatkan URI koneksi tunggal:

```h
const { Sequelize } = require('sequelize');

// Option 1: Passing a connection URI
const sequelize = new Sequelize('sqlite::memory:') // Example for sqlite
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname') // Example for postgres

// Option 2: Passing parameters separately (sqlite)
const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: 'path/to/database.sqlite'
});

// Option 3: Passing parameters separately (other dialects)
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: /* one of 'mysql' | 'postgres' | 'sqlite' | 'mariadb' | 'mssql' | 'db2' | 'snowflake' | 'oracle' */
});
```
### Testing the connection

Anda dapat menggunakan fungsi `.authenticate()` untuk menguji apakah koneksi baik-baik saja:
```h
try {
  await sequelize.authenticate();
  console.log('Connection has been established successfully.');
} catch (error) {
  console.error('Unable to connect to the database:', error);
}
```

### Closing the connection

Sequelize akan membuat koneksi tetap terbuka secara default, dan menggunakan koneksi yang sama untuk semua kueri. Jika Anda perlu menutup koneksi, panggil `sequelize.close()` (yang asinkron dan mengembalikan Promise).

## Model Basics

Model memberi tahu Sequelize beberapa hal tentang entitas yang diwakilinya, seperti nama tabel dalam database dan kolom mana yang dimilikinya (dan tipe datanya).

Model di Sequelize memiliki nama. Nama ini tidak harus sama dengan nama tabel yang diwakilinya dalam database. Biasanya, model memiliki nama tunggal (seperti Pengguna) sedangkan tabel memiliki nama jamak (seperti Pengguna), meskipun ini sepenuhnya dapat dikonfigurasi.

### Model Definition

Model dapat didefinisikan dalam dua cara yang setara dalam Sequelize:

1. Memanggil `sequelize.define` `(modelName, atribut, opsi)`
Memperluas `Model` dan memanggil `init``(atribut, opsi)`
Setelah model didefinisikan, tersedia dalam sequelize.models dengan nama modelnya.

2. Membuat sebuah model untuk merepresentasikan pengguna, yang memiliki `firstName` dan `lastName`. Kami ingin model kami disebut User, dan tabel yang diwakilinya disebut Users dalam database.

Kedua cara untuk mendefinisikan model ini ditunjukkan di bawah ini. Setelah didefinisikan, kita dapat mengakses model kita dengan  `sequelize.models.User`.

Contoh `sequelize.define` :

```h
const { Sequelize, DataTypes } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

const User = sequelize.define('User', {
  // Model attributes are defined here
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING
    // allowNull defaults to true
  }
}, {
  // Other model options go here
});

// `sequelize.define` also returns the model
console.log(User === sequelize.models.User); // true
```

Contoh `Extending Model`:
```h
const { Sequelize, DataTypes, Model } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

class User extends Model {}

User.init({
  // Model attributes are defined here
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING
    // allowNull defaults to true
  }
}, {
  // Other model options go here
  sequelize, // We need to pass the connection instance
  modelName: 'User' // We need to choose the model name
});

// the defined model is the class itself
console.log(User === sequelize.models.User); // true
```

Secara internal, `sequelize.define` memanggil `Model.init`, jadi kedua pendekatan tersebut pada dasarnya setara.

## Model Instances

Seperti yang sudah Anda ketahui, model adalah kelas ES6. Instance kelas mewakili satu objek dari model itu (yang memetakan ke satu baris tabel dalam database). Dengan cara ini, instance model adalah DAO.

Untuk panduan ini, pengaturan berikut akan diasumsikan:
```h
const { Sequelize, Model, DataTypes } = require("sequelize");
const sequelize = new Sequelize("sqlite::memory:");

const User = sequelize.define("user", {
  name: DataTypes.TEXT,
  favoriteColor: {
    type: DataTypes.TEXT,
    defaultValue: 'green'
  },
  age: DataTypes.INTEGER,
  cash: DataTypes.INTEGER
});

(async () => {
  await sequelize.sync({ force: true });
  // Code here
})();
```

### Creating an instance

Meskipun model adalah kelas, Anda tidak boleh membuat instance dengan menggunakan operator baru secara langsung. Penyimpanan itu adalah metode asinkron. Nyatanya, hampir setiap metode Sequelize tidak sinkron; build adalah salah satu dari sedikit pengecualian.

### Updating an instance

Jika mengubah nilai beberapa bidang dari suatu instance, memanggil `save` lagi akan memperbaruinya . Untuk dapat memperbarui beberapa bidang sekaligus dengan metode yang ditetapkan,

`save()` di sini juga akan mempertahankan perubahan lain yang telah dibuat pada instance ini, bukan hanya pada panggilan set sebelumnya.

### Deleting an instance

Untuk dapat menghapus instance dengan memanggil `destroy`.

### Reloading an instance

Untuk memuat ulang instance dari database dengan memanggil `reload`.

## Model Querying - Basics

Sequelize menyediakan berbagai metode untuk membantu menanyakan database Anda untuk data.

Pemberitahuan penting: untuk melakukan kueri siap-produksi dengan Sequelize, pastikan Anda juga telah membaca panduan Transaksi. Transaksi penting untuk memastikan integritas data dan memberikan manfaat lainnya.

Panduan ini akan menunjukkan cara membuat kueri CRUD standar

### Simple INSERT queries

Metode `Model.create()` adalah singkatan untuk membuat instance yang belum disimpan dengan `Model.build()` dan menyimpan instance dengan `instance.save()`.

Dimungkinkan juga untuk menentukan atribut mana yang dapat diatur dalam metode buat.

### Simple SELECT queries

Baca seluruh tabel dari database dengan metode `findAll`.

### Applying WHERE clauses

Opsi `where` digunakan untuk memfilter query. Ada banyak operator yang digunakan untuk klausa `where`, tersedia sebagai Simbol dari `Op`.

#### Basics

Tidak ada operator (dari Op) yang diteruskan secara eksplisit, jadi Sequelize mengasumsikan perbandingan kesetaraan secara default.

Sama seperti Sequelize menyimpulkan operator `Op.eq` pada contoh pertama, di sini Sequelize menyimpulkan bahwa penelepon menginginkan `AND` untuk dua pemeriksaan.

- `OR` dapat dengan mudah dilakukan dengan cara yang serupa.

## Model Querying - Finders

Metode Finder adalah metode yang menghasilkan kueri `SELECT`.

Secara default, hasil dari semua metode finder adalah turunan dari kelas model (sebagai kebalikan dari objek JavaScript biasa). Ini berarti bahwa setelah database mengembalikan hasilnya, Sequelize secara otomatis membungkus semuanya dalam objek contoh yang tepat. Dalam beberapa kasus, ketika hasilnya terlalu banyak, pembungkusan ini bisa menjadi tidak efisien. Untuk menonaktifkan pembungkusan ini dan menerima respons sederhana, berikan `{ raw: true }` sebagai opsi untuk metode finder.

#### findAll

Metode `findAll` sudah diketahui dari tutorial sebelumnya. Ini menghasilkan kueri `SELECT` standar yang akan mengambil semua entri dari tabel (kecuali dibatasi oleh sesuatu seperti klausa `WHERE`, misalnya)

#### findByPk

Metode `findByPk` hanya memperoleh satu entri dari tabel, menggunakan `Primary Key`
 yang disediakan.

 #### findOne
 
 Metode `findOne` memperoleh entri pertama yang ditemukannya (yang memenuhi opsi kueri opsional, jika disediakan).

 #### findOrCreate

 Metode `findOrCreate` akan membuat entri dalam tabel kecuali jika metode tersebut dapat menemukan entri yang memenuhi opsi kueri. Dalam kedua kasus, itu akan mengembalikan sebuah instance (baik instance yang ditemukan atau instance yang dibuat) dan boolean yang menunjukkan apakah instance itu telah dibuat atau sudah ada.
 Opsi `where` dipertimbangkan untuk menemukan entri, dan opsi `default` digunakan untuk menentukan apa yang harus dibuat jika tidak ada yang ditemukan. Jika `default` tidak berisi nilai untuk setiap kolom, Sequelize akan mengambil nilai yang diberikan `where` (jika ada).

 #### findAndCountAll

 Metode `findAndCountAll` adalah metode praktis yang menggabungkan `findAll` dan `count`. Ini berguna saat menangani kueri yang terkait dengan paginasi di mana Anda ingin mengambil data dengan `limit` dan `offset`, tetapi juga perlu mengetahui jumlah record yang cocok dengan kueri.

 Ketika `grup` tidak tersedia, metode `findAndCountAll` mengembalikan objek dengan dua properti:
 - `count` - bilangan bulat - catatan jumlah total yang cocok dengan kueri
- `rows` - array objek - catatan yang diperoleh

Saat `grup` disediakan, metode `findAndCountAll` mengembalikan objek dengan dua properti:

- `count` - larik objek - berisi hitungan di setiap grup dan atribut yang diproyeksikan
- `rows` - array objek - catatan yang diperoleh.

## Getters, Setters & Virtuals
### Getters

Pengambil adalah fungsi `get`() yang didefinisikan untuk satu kolom dalam definisi model:
```h
const User = sequelize.define('user', {
  // Let's say we wanted to see every username in uppercase, even
  // though they are not necessarily uppercase in the database itself
  username: {
    type: DataTypes.STRING,
    get() {
      const rawValue = this.getDataValue('username');
      return rawValue ? rawValue.toUpperCase() : null;
    }
  }
});
```

Pengambil ini, seperti pengambil JavaScript standar, dipanggil secara otomatis saat nilai bidang dibaca:
```h
const user = User.build({ username: 'SuperUser123' });
console.log(user.username); // 'SUPERUSER123'
console.log(user.getDataValue('username')); // 'SuperUser123'
```

Perhatikan bahwa, meskipun `SUPERUSER123` dicatat di atas, nilai yang benar-benar disimpan dalam database masih `SuperUser123`. Kami menggunakan `this.getDataValue('username')` untuk mendapatkan nilai ini, dan mengubahnya menjadi huruf besar.

Seandainya kami mencoba menggunakan `this.username` di pengambil, kami akan mendapatkan loop tak terbatas! Inilah mengapa Sequelize menyediakan metode `getDataValue`.

### Setters

Setter adalah fungsi `set()` yang ditentukan untuk satu kolom dalam definisi model. 

### Combining getters and setters

`Getter` dan `setter` dapat didefinisikan dalam bidang yang sama.

Sebagai contoh, katakanlah kita memodelkan `Post`, yang isinya adalah teks dengan panjang tidak terbatas. Untuk meningkatkan penggunaan memori, katakanlah kita ingin menyimpan versi konten yang di-gzip.

Setiap kali kami mencoba berinteraksi dengan bidang konten model `Post` kami, Sequelize akan secara otomatis menangani pengambil dan penyetel khusus.

### Virtual fields

Bidang virtual adalah bidang yang Sequelize terisi di bawah tenda, tetapi pada kenyataannya bidang itu bahkan tidak ada dalam database.

Sebagai contoh, katakanlah kita memiliki atribut `firstName` dan `lastName` untuk User.

Alangkah baiknya memiliki cara sederhana untuk mendapatkan nama lengkap secara langsung! Kita dapat menggabungkan ide getter dengan tipe data khusus yang disediakan Sequelize untuk situasi seperti ini:  `DataTypes.VIRTUAL`
```h
const { DataTypes } = require("sequelize");

const User = sequelize.define('user', {
  firstName: DataTypes.TEXT,
  lastName: DataTypes.TEXT,
  fullName: {
    type: DataTypes.VIRTUAL,
    get() {
      return `${this.firstName} ${this.lastName}`;
    },
    set(value) {
      throw new Error('Do not try to set the `fullName` value!');
    }
  }
});
```

Bidang `VIRTUAL` tidak menyebabkan kolom dalam tabel ada. Dengan kata lain, model di atas tidak memiliki kolom `fullName.`

## Validations & Constraints

**Validasi** adalah pemeriksaan yang dilakukan di level Sequelize, dalam JavaScript murni. Mereka bisa sangat rumit jika Anda menyediakan fungsi validator khusus, atau bisa menjadi salah satu validator bawaan yang ditawarkan oleh Sequelize. Jika validasi gagal, tidak ada kueri SQL yang akan dikirim ke database sama sekali.

Di sisi lain, **Constraints** adalah aturan yang ditentukan pada level SQL. Contoh kendala paling dasar adalah Kendala Unik. Jika pemeriksaan kendala gagal, kesalahan akan dilemparkan oleh database dan Sequelize akan meneruskan kesalahan ini ke JavaScript (dalam contoh ini, melempar `SequelizeUniqueConstraintError`). Perhatikan bahwa dalam kasus ini, kueri SQL dilakukan, tidak seperti kasus validasi.

### Validators

Validator model memungkinkan Anda menentukan validasi format/konten/warisan untuk setiap atribut model. Validasi dijalankan secara otomatis saat membuat, memperbarui, dan menyimpan. Anda juga dapat memanggil `validate()` untuk memvalidasi instance secara manual.

## Migration

Sama seperti Anda menggunakan sistem kontrol versi seperti Git untuk mengelola perubahan dalam kode sumber Anda, Anda dapat menggunakan `migrasi` untuk melacak perubahan pada database. Dengan migrasi, Anda dapat mentransfer database yang ada ke status lain dan sebaliknya: Transisi status tersebut disimpan dalam file migrasi, yang menjelaskan cara menuju ke status baru dan cara mengembalikan perubahan untuk kembali ke status lama.

Anda memerlukan `Sequelize Command-Line Interface (CLI`). CLI mengirimkan dukungan untuk migrasi dan proyek bootstrap.

Migrasi di Sequelize adalah file javascript yang mengekspor dua fungsi, `up` dan `down`, yang menentukan cara melakukan migrasi dan membatalkannya. Anda menentukan fungsi tersebut secara manual, tetapi Anda tidak memanggilnya secara manual; mereka akan dipanggil secara otomatis oleh CLI. Dalam fungsi-fungsi ini, Anda cukup melakukan kueri apa pun yang Anda butuhkan, dengan bantuan `sequelize.query` dan metode apa pun yang disediakan Sequelize untuk Anda.

### Installing the CLI
```h
npm install --save-dev sequelize-cli
```

### Project bootstrapping

Untuk membuat proyek kosong, Anda harus menjalankan perintah `init`.
```h
npx sequelize-cli init
```

Ini akan membuat folder berikut:

- `config`, berisi file konfigurasi, yang memberi tahu CLI cara terhubung dengan database
- `model`, berisi semua model untuk proyek Anda
- `migrasi`, berisi semua file migrasi
- `seeder`, berisi semua file benih

## Creating the first Model (and Migration)

Setelah Anda mengkonfigurasi file konfigurasi CLI dengan benar, Anda siap untuk membuat migrasi pertama Anda. Ini sesederhana menjalankan perintah sederhana.

Kami akan menggunakan perintah `model:generate`. Perintah ini memerlukan dua opsi:

- `name`: nama model;
- `atributes`: daftar atribut model.

## Running Migrations

Sampai langkah ini, kami belum memasukkan apa pun ke dalam database. Kami baru saja membuat model dan file migrasi yang diperlukan untuk model pertama kami, User. Sekarang untuk benar-benar membuat tabel itu di database, Anda perlu menjalankan perintah `db:migrate`.

# **MongoDB**

**MongoDB** adalah salah satu database open source NoSQL yang cukup populer digunakan. MongoDB menggunakan dokumen dengan format **JSON**.

**NoSQL adalah Not Only SQL** : Artinya kita bisa mengolah database dengan fleksibel dan tidak membutuhkan Query

## Kelebihan dan Kekurangan MongoDB

1. Kelebihan <br>
    - Sistem tidak membutuhkan Tabel
    - Tidak perlu menggunakan Tabel yang terstruktur
    - By Default sudah menggunakan JSON(JavaScript Object Notation), sehingga memudahkan integrasi dengan JavaScript
    - Performa lebih cepat dengan kemampuan menampung banyak data yang bervariasi. <br>
2. Kekurangan
    - Tidak mendukung transaksi
    - Masalah konsistensi data
    - Menggunakan banyak memory
    - Hanya bisa menampung maksimal 16MB disetiap document

**Anatomi MongoDB**
<br>
 Penjelesan mengenai anatomi :<br>
- **Database** adalah wadah untuk menyimpan berbagai macam Collection
- **Collection** adalah tempat kumpulan dari berbagai macam document, sehingga collection sering disamakan dengan tabel pada SQL
- **Document** adalah unit terkecil yang berada pada MongoDB
```h
{
   "_id" : ObjectId("54c955492b7c8eb21818bd09"),
   "alamat" : {
      "street" : "2 Avenue",
      "zipcode" : "10075",
      "building" : "1480",
      "coord" : [ -73.9557413, 40.7720266 ]
   },
   "borough" : "Manhattan",
   "cuisine" : "Italian",
   "grades" : [
      {
         "date" : ISODate("2014-10-01T00:00:00Z"),
         "grade" : "A",
         "score" : 11
      },
      {
         "date" : ISODate("2014-01-16T00:00:00Z"),
         "grade" : "B",
         "score" : 17
      }
   ],
   "name" : "Nurul Hikmah",
   "restaurant_id" : "41704620"
}
```

## MongoDB GUI Tools

Untuk mempermudah kita bisa menggunakan GUI Tools Official dari MongoDB yaitu MongoDB Compass.

## Operasi CRUD MongoDB

Kita bisa mulai dengan menyalakan MongoDB, lalu kita bisa mengakses dengan mengetikan `mongo` pada terminal.

Secara default setelah database kita adalah “test” . Untuk melihat database yang aktif, bisa gunakan `db` pada `MongoDB shell`
<br>
<br>
- Untuk melihat daftar database bisa menggunakan show dbs<br>
- Untuk Database baru gunakan `use (nama db)`<br>
- Untuk menambahkan Collection baru kita bisa gunakan `db.createCollections(“nama”)`<br>
- Untuk menambahkan data pada Collection kita bisa gunakan `db.(nama db).insert` <br>
- Untuk melihat data kita gunakan `db.(nama db).find()`<br>
- Untuk mengupdate data pada Collection kita bisa gunakan `db.(nama db).update`<br>
- Untuk mengupdate data pada Collection kita bisa gunakan `db.(nama db).remove`<br>

## Cara untuk mendesain schema MongoDB
### Schema Design Relational vs MongoDB

Jika kita mendesain schema aplikasi MongoDB kita sama dengan mendesain SQL maka kita akan banyak kehilangan fitur MongoDB.

Karena MongoDB
- Tidak ada Formal Process
- Tidak ada algorithms
- Tidak ada aturan

Sebuah MongoDB schema desain yang baik akan memperhatikan
- Bagaimana kita menstore data
- Menghasilkan perfoma Query yang bagus

Mendesain MongoDB kita ada 2 pendekatan :
1. Embedding, Dengan Embedding kita memasukan semua data yang terkait dalam satu dokumen.
2. Referencing, Dengan Referencing kita hanya memasukan data sebagian saja tidak keseluruhan.

## Relasi dalam MongoDB

1. One-to-One Relationships<br>
Hubungan one-to-one mewakili dari hubungan 2 objek yang berbeda. 
Cara yang disarankan adalah **Embedded**.

2. One-to-Many Relationships<br>
Dengan contoh 1 lagu bisa diputar oleh ratusan atau ribuan user. 
Cara yang disarankan adalah **Reference**.

3. Many-to-Many Relationships
Sekarang bayangkan kita sebagai seorang user bisa memiliki banyak playlist. 
Cara yang disarankan adalah **Reference**.

# **Mongoose**
**Mongoose** adalah library yang bisa dibilang sebagai Object Modelling MongoDB untuk NodeJS. Mongoose bisa digunakan untuk mengelola hubungan antara data, menyediakan validasi, Digunakan untuk menerjemahkan antara objek dalam kode dan representasi Objek tersebut di MongoDB.

## Create Connection
Membuat koneksi dengan menggunakan MongoDB database, yang diletakkan di `.env`.

Kita membiasakan diri dengan menggunakan file `.env` untuk menyimpan URL atau data rahasia yang tidak perlu dilihat di public.

## Defining your Schema
```h
const schema = new Schema({ name: String });
schema.path('name') instanceof mongoose.SchemaType; // true
schema.path('name') instanceof mongoose.Schema.Types.String; // true
schema.path('name').instance; // 'String'
```
Sebuah `SchemaType` berbeda dari sebuah tipe. Dengan kata lain, `mongoose.ObjectId !== mongoose.Types.ObjectId`. SchemaType hanyalah objek konfigurasi untuk Mongoose. Instance dari `mongoose.ObjectId` SchemaType sebenarnya tidak membuat MongoDB ObjectIds, ini hanyalah konfigurasi untuk jalur dalam skema.

## Simple CRUD

Sebelum membuat operasi CRUD, jangan lupa untuk menginstall express untuk routing dan body-parser, untuk menggunakan method Post dan testing API di postman.

Untuk menampilkan keseluruhan data `(READ)` kita bisa menggunakan fungsi `find()`.

Kita menggunakan method `POST` untuk mendaftarkan user, sebelum mendaftarkan kita mengecek dulu apakah user sudah ada atau belum dengan menggunakan `findOne()`, jika sudah ada akan muncul pesan error, jika belum terdaftar maka user akan didaftarkan menggunakan fungsi `create()`.

- Isi form raw dengan tipe JSON, atau bisa juga menggunakan form-data, untuk mendaftarkan user, setelah itu kita bisa menekan tombol send untuk mengirim request `POST`.

- Untuk mendapatkan data user berdasarkan id, dengan fungsi `findById()`.<br>
-  Untuk menghapus satu data berdasarkan ID dengan menggunakan `deleteOne()`. <br>
- Untuk mengedit/update satu data berdasarkan ID dengan menggunakan `findByIdAndUpdate()`.<br>

## Populate

**Populate** ada kaitannya dengan relasi database. Populate adalah proses penggabungan 2 collection atau lebih menjadi satu objek JSON.

- Untuk menambahkan field , denga type Schema.types.ObjectId, dan ref merujuk kepada schema “nama field”.<br>
- Untuk membuat schema baru, dengan "nama_field", untuk di populate dengan (nama_Schema) schema. Buatlah Route dengan method `POST` menuju ke endpoint /(nsms_field), untuk mengisi Data<br>
-  untuk menampilkan keseluruhan data user dengan kode diatas. Kita melakukan `Populate()` dengan address schema, dan akan ditampilkan isi dari address.
- Untuk test menggunakan Postman untuk method `GET` ke endpoint /users, data users akan muncul dengan data address, berarti kita berhasil mempopulate schema users dengan address.

# **Docker**

**Docker** adalah software yang menjalankan suatu aplikasi menggunakan container. Docker men-sharing kernel dari host OS, serta meng-container-kan suatu aplikasi agar dapat dijalankan dimana saja dan kapan saja. Aplikasi yg berjalan di dalam container docker tidak terpengaruh oleh faktor luar karena terisolasi

Docker berfungsi sebagai penyedia layanan virtual bagi aplikasi yg diinstall pada sebuah host. 

## Container VS Virtual Machine

**VM** memakan banyak resource dan waktu utk booting karena melakukan virtualisasi pada host hardware-nya. 

Sedangkan **container** kebalikannya dari vm, container melakukan virtualisasi pada host OS-nya

## Docker Fundamental

- **Docker File** : Blueprint untuk membuat image

- **Image** : Template untuk menjalankan container

- **Container** : Perwujudan dari image

- **Docker Registry** : Tempat untuk upload/download image

## Perintah Dasar Docker

- docker pull
- docker images
- docker run
- docker ps

## Docker File

Merupakan sebuah blueprint untuk  membuat image, kamu juga bisa membuat custom image menggunakan docker file.

Cara :

1. Buat file Dockerfile di dalam project yang kamu buat
2. Tulis beberapa perintah ke dalam dockerfile
3. Jalankan docker file menggunakan perintah 
`docker build -t NAMA_IMAGES:TAG` . 
`docker build -t my-app:1.0` . 

## Docker Compose

Cara untuk menjalankan lebih dari 1 container secara bersamaan dan saling terhubung.

Caranya :

1. Buat file NAMA_FILE.yaml di dalam project yang kamu buat
2. Tulis beberapa perintah ke dalam sana
3. Jalankan menggunakan perintah 
`docker-compose NAMA_FILE.yaml up`