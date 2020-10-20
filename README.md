# Spring Boot で DB マイグレーションツール Flyway を使う

## サンプルコードの実行方法

### MySQL コンテナの起動
```console
docker-compose up -d
```

### API サーバの起動
```console
gradle bootRun
```

### API ドキュメントを開く
```
http://localhost:8080/swagger-ui/index.html
```

***

## Flywayマイグレーションとは
データベースのバージョン管理ツールである。
 
### 特徴
- アプリを新規作成した場合はSQLが全て適用される
- 過去のバージョンのアプリの場合は差分のSQLのみ適用される
- DB更新のデグレが発生しないので、特に複数人で開発する際に役立つ
- アプリケーションを起動するとマイグレーションが実行される

## Flyway を使うには
以下の準備をすることで Flyway を利用することができる。

- build.gradle への追記
- SQL ファイルの作成・配置

## build.gradle への追記
build.gradle の plugins、dependencies に追記し、新たに flyway ブロックを追記する。
```
plugins {
	id "org.flywaydb.enterprise.flyway" version "6.5.7"
}

flyway {
  url      = 'jdbc:mysql://192.168.99.100:13306/covid19'
  user     = 'fsedu'
  password = 'secret'
}

dependencies {
	implementation 'org.flywaydb:flyway-core'
}
```

## SQL ファイルの作成・配置
`/src/main/resources/db/migration` 配下に次の命名規則となる SQL ファイルを配置する。
- 先頭文字が "V"
- アンダースコア区切りのバージョン番号
- 環境（任意）
- マイグレーションログ識別のための名前

例: V1_0_1__dev_create_table.sql
```sql
CREATE TABLE covid_19_case ( 
  id bigint NOT NULL AUTO_INCREMENT
  , name varchar(255) DEFAULT NULL
  , gender varchar(255) DEFAULT NULL
  , age int DEFAULT NULL
  , address varchar(255) DEFAULT NULL
  , city varchar(255) DEFAULT NULL
  , country varchar(255) DEFAULT NULL
  , status varchar(255) DEFAULT NULL
  , created_at date DEFAULT NULL
  , updated_at date DEFAULT NULL
  , PRIMARY KEY (id)
) ENGINE = InnoDB AUTO_INCREMENT = 0 
DEFAULT CHARSET = utf8mb4 
COLLATE = utf8mb4_0900_ai_ci
;
```

例: V1_0_2__dev_initial_data.sql
```sql
insert into covid19.covid_19_case
  (name,gender,age,address,city,country,status,created_at,updated_at)
values
  ('山本一郎', '男', 23, '愛知県', '名古屋市', '日本', '1', '2020/10/20', '2020/10/20')
,
  ('小野田真理子', '女', 11, '和歌山県', '和歌山市', '日本', '2', '2020/10/20', '2020/10/20');
```

準備が整ったのでマイグレーションを実行する。

## マイグレーションコマンド

### ベースラインの作成
```console
gradle flywayBaseline -i
```

### マイグレーションの実行
```console
gradle flywayMigrate
```

### マイグレーション情報の表示
```console
gradle flywayInfo
```
実行結果:
```
> Task :flywayInfo
Schema version: 1.0.2
+-----------+---------+-----------------------+----------+---------------------+----------+----------+
| Category  | Version | Description           | Type     | Installed On        | State    | Undoable |
+-----------+---------+-----------------------+----------+---------------------+----------+----------+
|           | 1       | << Flyway Baseline >> | BASELINE | 2020-10-20 14:28:33 | Baseline | No       |
| Versioned | 1.0.1   | dev create table      | SQL      | 2020-10-20 14:28:43 | Success  | No       |
| Versioned | 1.0.2   | dev initial data      | SQL      | 2020-10-20 14:28:43 | Success  | No       |
+-----------+---------+-----------------------+----------+---------------------+----------+----------+
```

### マイグレーションを戻す
```console
gradle flywayClean
```
