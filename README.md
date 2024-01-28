# Sequelize Notlari

Bu notlar, 19 ocak 2024 tarihinden itibaren yazilmaya baslanmis olup, bir back-end projesi gelistirirken yardimci olunmasi icin Tayfun Erbilen tarafindan olusturulmustur. Bu notlarin yazildigi tarihde `v6-stable` versiyonu olup, `v7-alpha` surumu de mevcuttur. Butun notlar `v6-stable` versiyonuna gore hazirlanmistir.

Sequelize promise tabanli Postgres, MySQL, MariaDB, SQLite, Microsoft SQL Server, Oracle Database vs.. icin bir ORM aracidir.

## Giris

### Kurulum

Herhangi bir paket yoneticisi ile `sequelize` paketini kurmaniz yeterlidir.

```
bun add sequelize
npm i sequelize # ya da
yarn add sequelize # ya da
pnpm i sequelize # ya da
```

Ek olarak hangi veritabanini kullaniyorsaniz bunu da manuel olarak yuklemelisiniz.

```
bun add pg pg-hstore # Postgres
bun add mysql2
bun add mariadb
bun add sqlite3
bun add tedious # Microsoft SQL Server
bun add oracledb # Oracle Database
```

### Veritabani Baglantisi

Veritabani baglantisi icin bir Sequelize instance'i olusturmaniz gerekiyor. Bunu ister tek parametrede baglanti URI'i olarak ya da parametre gecerek yapabiliyorsunuz.

```js
import { Sequelize } from 'sequelize';

// Secenek 1: Baglanti URI kullanarak baglanmak
const sequelize = new Sequelize('sqlite::memory:') // sqlite icin ornek
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname') // postgres icin ornek

// Secenek 2: Parametre olarak gecme ornegi (sqlite icin)
const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: 'path/to/database.sqlite'
});

// Secenek 3: Parametre olarak gecme ornegi (diger dbler icin)
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: /* sunlardan birisi gelebilir: 'mysql' | 'postgres' | 'sqlite' | 'mariadb' | 'mssql' | 'db2' | 'snowflake' | 'oracle' */
});
```

Gunun sonunda mysql bir veritabanina soyle baglaniyoruz:

```js
// db.js
const sequelize = new Sequelize('DBNAME', 'DBUSER', 'DBPASS', {
    host: 'localhost',
    dialect: 'mysql'
})
```

### Baglantiyi Test Etmek

Basarili bir sekilde baglanip baglanmadiginizi test etmek icin `authenticate()` metodunu calistirabilirsiniz.

```js
import {sequelize} from "./db";

try {
    await sequelize.authenticate()
    console.log('Veritabanina basariyla baglanildi')
} catch (e) {
    console.log('Veritabani baglanti hastasi:', e)
}
```

Eger basariliysa soyle bir cikti gorursunuz console'da:

```
Executing (default): SELECT 1+1 AS result
Veritabanina basariyla baglanildi
```

Basarisizsa da hatayi goreceksiniz.

### Baglantiyi Kapatma

Sequelize varsayilan olarak tum sorgular icin baglantiyi acik tutarak ayni baglantiyi kullanmaktadir. Baglantiyi kendiniz kapatmak isterseniz `close()` metodunu kullanabilirsiniz.

```js
sequelize.close()
```

> Baglantiyi bir kere kapattiktan sonra bir daha veritabanina erisiminiz olmaz, erismek icin yeni bir Sequelize instance'i olusturmaniz gerekir, unutmayin.

### Yeni Veritabani vs. Mevcut Veritabani

Eger sifirdan bir proje gelistiriyorsaniz ve veritabaniniz henuz bossa, Sequelize veritabaninizdaki tum tablolari otomatik olusturmak icin kullanilabilir.

Eger mevcut bir projeniz ve veritabaniniz varsa sorun degil, Sequelize bunu da basarili sekilde yonetebiliyor.

## Temel Konseptler

### Model (Temeller)

Model, veritabanindaki tabloyu temsil eden bir soyutlamadir. Sequelize'da ise model, `Model` sinifindan extend edilen bir sinif demektir.

Modeller Sequelize'a bir cok sey soyleyebilir, tablo adi, tablonun kolonlari, kolonlarin veri turleri vs.

Her modelin bir ismi olur, bu isim tablo adinizla ayni olmak zorunda degildir. Genelde modeller tekil olarak isimlendirilir, ornegin modelinizin adi `User` olur ancak tablo adiniz `Users` gibi dusunebilirsiniz, bu size kalmis bir tanimlama sekli, istediginiz gibi ayarlayabilirsiniz.

### Model Tanimlama

Modeli 2 farkli birbirine denk yontemle olusturabilirsiniz:

- define() metodu ile tanimlama
- Model sinifindan extend ederek init() metodu ile tanimlama

Kisaca birinde sinifi sequelize yaratirken, digerinde siz yaratip tanim giriyorsunuz, iki yontemde birbiriyle ayni, hangisi kolayiniza geliyorsa.

Yukaridaki iki farkli metod ile kullanicilari temsil eden bir model olusturalim. Model adimiz `User` olsun ve `firstName`, `lastName` adinda 2 kolonu oldugunu varsayalim. Tablo adimiz ise `users` olsun.

#### define() ile Model Tanimlama

```js
import { DataTypes } from 'sequelize';

const User = sequelize.define('User', {
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING
    // allowNull varsayilan olarak true, tanim yapmaniza gerek yok
  }
}, {
  // Model ile ilgili diger ayarlar buraya gelecek
});

// `sequelize.define` ayni zamanda modeli return eder
console.log(User === sequelize.models.User); // true
```

#### Modeli extend Ederek Tanimlama

```js
import { DataTypes, Model } from 'sequelize';

class User extends Model {}

User.init({
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING
  }
}, {
  sequelize, // Baglanti instance'ini belirlememiz gerekiyor
  modelName: 'User' // Model adini belirlememiz gerekiyor
});

console.log(User === sequelize.models.User); // true
```

Fark ettiginiz uzere, tablo adini belirlemedik. Sequelize [inflection](https://www.npmjs.com/package/inflection) adinda bir kutuphane sayesinde cogullastirma islemini kendisi yapiyor. Yani `User` olarak tanımlanan bir modelin tablo karşılığı `users` oluyor. Ya da `Person` olan bir modelin tablo adı `people` oluyor ya da `Category` olan bir modelin tablo adı `categories` oluyor gibi gibi.

Tabi isterseniz model adinizla tablo adinizin ayni olacagini ek bir ayarla belirtebiliyorsunuz:

```js
import { DataTypes } from 'sequelize';

const User = sequelize.define('Users', {
  firstName: {
    type: DataTypes.STRING
  },
  lastName: {
    type: DataTypes.STRING
  }
}, {
  freezeTableName: true
});
```

Eger olusturugunuz her model'e bunu her seferinde eklemek istemezseniz, instance'i olustururken de genel bir ayar ekleyebiliyorsunuz:

```js
const sequelize = new Sequelize('DBNAME', 'DBUSER', 'DBPASS', {
    host: 'localhost',
    dialect: 'mysql',
    define: {
        freezeTableName: true
    }
})
```

Ya da model isimlerim tekil kalsin ben tablo adini her modelde belirtecegim diyorsaniz da soyle yapabiliyorsunuz:

```js
import { DataTypes } from 'sequelize';

const User = sequelize.define('User', {
  firstName: {
    type: DataTypes.STRING
  },
  lastName: {
    type: DataTypes.STRING
  }
}, {
  tableName: 'users'
});
```

### Model Senkronizasyonu

Bir model tanimladiginizda, Sequelize'a veritabanindaki tablo ile ilgili bilgiler veriyorsunuz. Peki ya tablonuz hic yok ise? Ya da varsa ama kolonlari degismis ise?

Iste bu sebepten oturu model senkronizasyonu bulunuyor. Bunu da `sync()` metodu ile yapabiliyorsunuz. Bu metodu da farkli kullanim sekilleri var, soyle ki:

- `User.sync()` - Bu eger tablo yoksa tabloyu olusturur, varsa hicbir sey yapmaz.
- `User.sync({ force: true })` - Bu eger tablo varsa once onu kaldirir, sonra tabloyu olusturur.
- `User.sync({ alter: true })` - Bu veritabanindaki tabloyu analiz eder, hangi kolonlari var, veri tipleri ne vs. diye. Ve model'de bir degisiklik varsa bunu tabloya yansitir. Yani modelde yeni bir kolon tanimladiysaniz, bir kolonun tipini vs. degistirdiyseniz veritabaninda bunu gunceller.

#### Tüm Modelleri Ayni Anda Senkron Etme

Sequelize instance'iniz butun modelleri bildigi icin, bu instance'in altindaki `sync()` metodunu cagirarak butun modelleri tek seferde senkron edebilirsiniz.

```js
await sequelize.sync({ force: true });
console.log("Tum modeller basariyla senkron edildi.");
```

### Tablolari Kaldirma

Modelle iliskili olan tabloyu kaldirmak icin `drop()` metodunu kullanabilirsiniz.

```js
await User.drop();
console.log("User tablosu kaldirildi!");
```

Tum tablolari kaldirmak icin:

```js
await sequelize.drop();
console.log("Tum tablolar kaldirildi!");
```

### Veritabanı guvenligi

Bildiginiz gibi `sync` ve `drop` islemleri biraz riskli. Bunu production seviyesindeki bir veritabaninda yaparken dikkatli olmakta fayda var. Sequelize bu konuda ek bir ayarda getiriyor.

```js
// Bu eger veritabani adi '_test' ile bitiyorsa sync islemi yapmanizi saglar
sequelize.sync({ force: true, match: /_test$/ });
```

### Timestamp

Varsayilan olarak, Sequelize her modele `createdAt` ve `updatedAt` kolonlarini `DataTypes.DATE` tipiyle ekler. Bu sayede bir create ya da update islemi oldugunda Sequelize otomatik olarak bu kolonlari doldurur.

> **Not:** Bu Sequelize seviyesinde gerceklesen bir olaydir. Yani db'den siz bir deger ekleyip guncellediginizde bu tetiklenmez, Sequelize ile yapacaginiz islemler icin bu gecerlidir.

Bu davranisi iptal etmek isterseniz `timestamps: false` ayarini ekleyebilirsiniz.

```js
sequelize.define('User', {
  // ...
}, {
  timestamps: false
});
```

Ayrica istediginiz enable/disable etmeniz mumkun, ve varsayilan isimlerini tanimlayabiliyorsunuz:

```js
sequelize.define('User', {
  // ...
}, {
  timestamps: true,
  createdAt: false, // olusturma tarihini iptal et,
  updatedAt: 'updatedStock' // guncellenme tarihinin kolon adini degistir
});
```

### Kolon Kisa Tanimi

Eger kolonu tanimlarken sadece veri tipini belirtiyorsaniz, baska bir ayariniz yoksa bunu kisa tanim kullanarakta yapabilirsiniz. Yani sunu:

```js
sequelize.define('User', {
  name: {
    type: DataTypes.STRING
  }
});
```

su sekilde kisaltabilirsiniz:

```js
sequelize.define('User', {
  name: DataTypes.STRING
});
```

Eger farkli kolon tanimlari da yapacaksaniz, o zaman mecbur uzun yoldan yapmaya devam :)

### Varsayilan Degerler

Varsayilan olarak, Sequelize bir kolonun varsayilan degerini `NULL` kabul eder. Bu davranisi `defaultValue` tanimini ekleyerek degistirebilirsiniz.

```js
sequelize.define('User', {
  isMarried: {
    type: DataTypes.BOOLEAN,
    defaultValue: true
  }
});
```

Bazi ozel degerler, ornegin `DataTypes.NOW` gibi kabul edilmektedir.

```js
sequelize.define('Foo', {
  bar: {
    type: DataTypes.DATETIME,
    defaultValue: DataTypes.NOW // bu sekilde bir insert islemi sirasinda zaman kaydedilecektir varsayilan deger olarak
  }
});
```

### Veri Turleri

Modelde tanimladiginz her kolonun bir veri turu olmak zorunda. Sequelize size bir cok veri turuyle ilgili destek sagliyor. Bunun icin zaten yukarida bolca yaptigimiz `DataTypes` objesini import edip icinden kullaniyoruz.

```js
import { DataTypes } from "sequelize"
```

#### Strings

```js
DataTypes.STRING             // VARCHAR(255)
DataTypes.STRING(1234)       // VARCHAR(1234)
DataTypes.STRING.BINARY      // VARCHAR BINARY
DataTypes.TEXT               // TEXT
DataTypes.TEXT('tiny')       // TINYTEXT
```

#### Boolean

```js
DataTypes.BOOLEAN            // TINYINT(1)
```

#### Numbers

```js
DataTypes.INTEGER            // INTEGER
DataTypes.BIGINT             // BIGINT
DataTypes.BIGINT(11)         // BIGINT(11)

DataTypes.FLOAT              // FLOAT
DataTypes.FLOAT(11)          // FLOAT(11)
DataTypes.FLOAT(11, 10)      // FLOAT(11,10)

DataTypes.DOUBLE             // DOUBLE
DataTypes.DOUBLE(11)         // DOUBLE(11)
DataTypes.DOUBLE(11, 10)     // DOUBLE(11,10)

DataTypes.DECIMAL            // DECIMAL
DataTypes.DECIMAL(10, 2)     // DECIMAL(10,2)
```

#### Unsigned & Zerofill integers

MySQL ve MariaDB'de `INTEGER`, `BIGINT`, `FLOAT` ve `DOUBLE` türleri unsigned ya da zerofill ya da her ikisi olarak ayarlanabiliyor.

`unsigned` - Sadece pozitif degerler atanmasi icin kullanilir. Normalde INT bir veri turu -2,147,483,648 ile 2,147,483,647 arasında değer alabilir. Ancak UNSIGNED kullanıldığında, bu aralık 0 ile 4,294,967,295 arasında olur. Bu, sayısal alanın kapasitesini negatif sayılar yerine daha büyük pozitif sayılar için kullanmayı mümkün kılar.

`Zerofill` - Bu özellik, bir sayısal alanın belirtilen genişliğe ulaşması için alanın soluna sıfır ekler. Eğer alanın uzunluğu belirtilen genişlikten kısa ise, eksik kalan kısımlar sol taraftan sıfırlarla doldurulur. ZEROFILL kullanıldığında otomatik olarak UNSIGNED özelliği de uygulanır, çünkü negatif sayılar sıfır ile doldurulamaz.

```js
DataTypes.INTEGER.UNSIGNED
DataTypes.INTEGER(5).ZEROFILL
DataTypes.INTEGER.UNSIGNED.ZEROFILL
```

#### Dates

```js
DataTypes.DATE
DataTypes.DATEONLY // sadece tarih
```

#### UUIDs

UUID'ler icin `DataTypes.UUID` kullanabilirsiniz. PostgreSQL ve SQLite icin `UUID` turune, MySQL icin `CHAR(36)` turune denk olacaktir. Sequelize bu alan icin UUID'yi otomatik olusturabilir, varsayilan degere `DataTypes.UUIDV1` ya da `DataTypes.UUIDV4` eklemeniz yeterli.

```js
{
  type: DataTypes.UUID,
  defaultValue: DataTypes.UUIDV4 // Ya da DataTypes.UUIDV1
}
```

Geri kalan veri turlerine [suradan](https://sequelize.org/docs/v6/other-topics/other-data-types/) bakabilirsiniz.

### Kolon Ayarlari

- `allowNull` - Varsayilan olarak sequelize kolonu nullable olarak tanimlar. Eger bu alan zorunlu olacaksa `allowNull: false` kullanilabilir.
- `defaultValue` - Varsayilan bir kolon degeri icin bu ayar kullanilir.
- `unique` - Kolonun benzersiz bir kolon oldugunu tanimlamak icin `unique: true` kullanilabilir.
- `primaryKey` - Primary key tanimi icin bu ayar true olarak kullanilabilir.
- `autoIncrement` - Otomatik artan degerler icin bu ayar true olarak kullanilabilir.
- `field` - Kolonun adini degistirmek icin bu ayar kullanilabilir.
- `comment` - MySQL, MariaDB, PostgreSQL ve MSSQL icin kolona yorum eklemek isterseniz bu ayar kullanilabilir.

Ayrica Foreign Key eklemekte mumkun:

```js
role_id: {
    type: DataTypes.INTEGER,
    references: {
        model: Role,
        key: 'id'
    }
}
```

## Model Instance'i

Sequeleze'de modeller birer ES6 sinifi, yani bildiginiz javascript class'i. Instance'da veritabaninda bir row'un degerini temsil eden bir obje olarak dusunebilirsiniz.

Bundan sonraki ornekleri su yapi uzerinden inceliyor olacagiz:

```js
import { Sequelize, Model, DataTypes } from 'sequelize'
const sequelize = new Sequelize() // db baglantisi ayarlari

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

### Instance Olusturma

Model bir sinif olsada, instance olusturmak icin `new` anahtar kelimesiyle ile sinifi baslatmadan `build` metodunu kullaniyoruz.

```js
const jane = User.build({ name: "Jane" });
console.log(jane instanceof User); // true
console.log(jane.name); // "Jane"
```

Bu sekilde kodumuz henuz veritabani ile bir baglanti gerceklestirmiyor. Bu metod aslinda db'ye uygun bir objeyi bize donduruyor. Gercekten kaydetmek icin `save` metodunu kullaniyoruz.

```js
await jane.save();
console.log('Kullanıcı dbye eklendı!');
```

> `build` ve birkac metod haric Sequelize'de tum metodlar asenkron oldugunu unutmayin.

### create Metodu

Build edip sonra save etmek istemiyorsaniz, dogrudan kayit icin `save()` metodunu kullanabilirsiniz.

```js
// dbye kayit eder
const jane = await User.create({ name: "Jane" });
console.log(jane instanceof User); // true
console.log(jane.name); // "Tayfun"
```

### Loglama

Instance'in sonucunu loglamak icin yardimci bir metod olan `toJSON()` kullanabilirsiniz ya da `JSON.stringify` icinde gosterebilirsiniz.

```js
const jane = await User.create({ name: "Jane" });
// console.log(jane); // Bunu yapmayin
console.log(jane.toJSON()); // Boyle harika!
console.log(jane.stringify(user, null, 4)); // Boyle de harika!
```

### Varsayilan Degerler

Build metodu otomatik olarak varsayilan degerleri de dondurecektir. Yani kolona bir varsayilan deger eklediyseniz ve insert isleminde bu deger icin bir deger gondermediyseniz otomatik olarak varsayilan degeri kabul edip onu dondurecektir.

```js
const jane = User.build({ name: "Jane" });
console.log(jane.favoriteColor); // "green"
```

### Instance'i Guncelleme

Instance'daki herhangi bir degeri degistirdiginizde, `save()` metodunu cagirmaniz kaydetmesi icin yeterli olacaktir.

```js
const jane = await User.create({ name: "Jane" });
console.log(jane.name); // "Jane"
jane.name = "Ada";
// db'de isim hala "Jane"
await jane.save();
// Simdi db'deki isim "Ada" olarak guncellendi
```

Birden fazla degeri tek seferde `set` metoduyla da guncelleyebilirsiniz.

```js
const jane = await User.create({ name: "Jane" });
jane.cash = 3000
jane.set({
  name: "Ada",
  favoriteColor: "blue"
});
// db 'de su an hala degerler guncellenmesi, sadece buna hazirlandi
await jane.save();
// simdi db'de de bu degisiklikler uygulandi
```

Fark ettiginiz uzere, set ile birden fazla degeri guncellemek istedigimizde `save` metoduyla kaydetmemiz gerekiyor, bu da diger degisikliklerin de kaydolmasina sebep oluyor, bunun sadece yaptiginiz degisiklikleri etkilemesini istiyorsaniz `update` metodunu kullanabilirsiniz.

```js
const jane = await User.create({ name: "Jane" });
jane.favoriteColor = "blue"
await jane.update({ name: "Ada" })
// db'de isim "Ada" olarak guncellendi ancak favoriteColor hala "green" degerinde cunku update sadece icindekileri degistirir
await jane.save()
// Artik favoriteColor degisikligi de kaydoldu
```

### Instance Silme

Silme islemi icin `destroy()` metodunu kullanabilirsiniz.

```js
const jane = await User.create({ name: "Jane" });
console.log(jane.name); // "Jane"
await jane.destroy();
// dbden silindi
```

### Instance'i Yeniden Yukleme (reload)

`reload` metoduyla instance'i yeniden yukleyebilirsiniz.

```js
const jane = await User.create({ name: "Jane" });
console.log(jane.name); // "Jane"
jane.name = "Ada";
// db'de isim hala Jane
await jane.reload();
console.log(jane.name); // "Jane"
```

### Belirli Alanlari Guncelleme

`save` metodu cagrildiginda hangi alanlarin guncellenmesi gerektigini belirtebiliyorsunuz. 

```js
const jane = await User.create({ name: "Jane" });
console.log(jane.name); // "Jane"
console.log(jane.favoriteColor); // "green"
jane.name = "Jane II";
jane.favoriteColor = "blue";
await jane.save({ fields: ['name'] });
console.log(jane.name); // "Jane II"
console.log(jane.favoriteColor); // "blue"
// favoriteColor blue basildi cunku objede degeri blue olarak belirlendi
// ancak db'de hala green degerinde
await jane.reload();
console.log(jane.name); // "Jane II"
console.log(jane.favoriteColor); // "green"
```

> `save` metodu cagirildiginda eger bir degisiklik yoksa aninda promise resolve olacaktir ancak bir db query'si olusturmayacaktir.

### Increment ve Decrement

Db'de integer verilerin degerlerini artirmak ya da azaltmak icinde metodlar mevcut:

```js
const jane = await User.create({ name: "Jane", age: 100 });
await jane.increment('age'); // +1 artirir
await jane.increment('age', { by: 2 }) // +2 artirir
```

birden fazla degeri de tek seferde guncelleyebilirsiniz.

```js
const jane = await User.create({ name: "Jane", age: 100, cash: 5000 });
await jane.increment({
  age: 2,
  cash: 100
});

// Eger birden fazla degeri ayni sayida artiracaksaniz da boyle kullanabilirsiniz
await jane.increment(['age', 'cash'], { by: 2 });
```

decrement yani azaltma olayi da tahmin edebileceginiz uzere `increment` yerine `decrement` koyarak gerceklestiriyorsunuz.
