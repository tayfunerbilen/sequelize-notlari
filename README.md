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
const db = new Sequelize('sqlite::memory:') // sqlite icin ornek
const db = new Sequelize('postgres://user:pass@example.com:5432/dbname') // postgres icin ornek

// Secenek 2: Parametre olarak gecme ornegi (sqlite icin)
const db = new Sequelize({
  dialect: 'sqlite',
  storage: 'path/to/database.sqlite'
});

// Secenek 3: Parametre olarak gecme ornegi (diger dbler icin)
const db = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: /* sunlardan birisi gelebilir: 'mysql' | 'postgres' | 'sqlite' | 'mariadb' | 'mssql' | 'db2' | 'snowflake' | 'oracle' */
});
```

Gunun sonunda mysql bir veritabanina soyle baglaniyoruz:

```js
// db.js
const db = new Sequelize('DBNAME', 'DBUSER', 'DBPASS', {
    host: 'localhost',
    dialect: 'mysql'
})
```

### Baglantiyi Test Etmek

Basarili bir sekilde baglanip baglanmadiginizi test etmek icin `authenticate()` metodunu calistirabilirsiniz.

```js
import {db} from "./db";

try {
    await db.authenticate()
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
db.close()
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

const User = db.define('User', {
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
  sequelize: db, // Baglanti instance'ini belirlememiz gerekiyor
  modelName: 'User' // Model adini belirlememiz gerekiyor
});

console.log(User === sequelize.models.User); // true
```

Fark ettiginiz uzere, tablo adini belirlemedik. Sequelize [inflection](https://www.npmjs.com/package/inflection) adinda bir kutuphane sayesinde cogullastirma islemini kendisi yapiyor. Yani `User` olarak tanımlanan bir modelin tablo karşılığı `users` oluyor. Ya da `Person` olan bir modelin tablo adı `people` oluyor ya da `Category` olan bir modelin tablo adı `categories` oluyor gibi gibi.

Tabi isterseniz model adinizla tablo adinizin ayni olacagini ek bir ayarla belirtebiliyorsunuz:

```js
import { DataTypes } from 'sequelize';

const User = db.define('Users', {
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
const db = new Sequelize('DBNAME', 'DBUSER', 'DBPASS', {
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

const User = db.define('User', {
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

