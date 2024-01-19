# Sequelize Notlari

Bu notlar, 19 ocak 2024 tarihinden itibaren yazilmaya baslanmis olup, bir back-end projesi gelistirirken yardimci olunmasi icin Tayfun Erbilen tarafindan olusturulmustur. Bu notlarin yazildigi tarihde `v6-stable` versiyonu olup, `v7-alpha` surumu de mevcuttur. Butun notlar `v6-stable` versiyonuna gore hazirlanmistir.

Sequelize promise tabanli Postgres, MySQL, MariaDB, SQLite, Microsoft SQL Server, Oracle Database vs.. icin bir ORM aracidir.

## Kurulum

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

## Veritabani Baglantisi

Veritabani baglantisi icin bir Sequelize instance'i olusturmaniz gerekiyor. Bunu ister tek parametrede baglanti URI'i olarak ya da parametre gecerek yapabiliyorsunuz.

```js
const { Sequelize } = require('sequelize');

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
const db = new Sequelize('DBNAME', 'DBUSER', 'DBPASS', {
    host: 'localhost',
    dialect: 'mysql'
})
```

## Baglantiyi Test Etmek

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
