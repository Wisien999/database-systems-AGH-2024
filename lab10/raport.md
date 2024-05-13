> initdb -D ./publisher_db

![alt text](image.png)

> initdb -D ./subscriber_db

![alt text](image-1.png)

W pliku postgresql.conf zmieniamy port i wal_level na logical

> pg_ctl start -D ./publisher_db

![alt text](image-2.png)

> pg_ctl start -D ./subscriber_db

![alt text](image-3.png)

Łączymy się z główną instancją

> psql -p 5433

Tworzymy baze danych

> CREATE DATABASE pub_db;

> \c pub_db

Tworzymy Tabele

> CREATE TABLE pub_tl (id int, name varchar(80))

> INSERT INTO pub_tbl
> SELECT * FROM generate_series(1, 10) AS id, md5(random()::text) AS name;

![alt text](image-4.png)

Przełączamy sie na sub i tworzymy tam baze sub_db

> psql -p 5434

> CREATE DATABASE sub_db;

Używamy polecenia pg_dump by wykonać dump bazy pub_db

> pg_dump -d pub_db -h localhost -p 5433 | /usr/lib/postgresql/15/bin/psql -p 5434 -d sub_db

![alt text](image-5.png)

![alt text](image-6.png)

Jak widać odrazu mamy skopiowane dane

Zatem dropnijmy dane dla potrzeb ćwiczenia:

![alt text](image-7.png)

Tworzymy teraz publikacje na głównej instancji

> psql -p 5433 -d pub_db

> CREATE PUBLICATION mypublication FOR TABLE pub_tbl;

![alt text](image-8.png)

Teraz stowrzmy subskrypcje:

> psql -p 5434 -d sub_db

> CREATE SUBSCRIPTION mysub CONNECTION 'host=localhost port=5433 dbname=pub_db' PUBLICATION mypublication;

![alt text](image-9.png)

Efekt w głównej instancji:

![alt text](image-10.png)

Sprawdzam czy dane zostały zreplikowane:

![alt text](image-11.png)

Dalsza weryfikacja działania:

### Insert


![alt text](image-12.png)


![alt text](image-13.png)

### Update

Przy próbie wykonania:

> UPDATE pub_tbl SET name='test-test' where id=21;

Dostaje error:

![alt text](image-14.png)

Korzystamy z hinta:

> ALTER TABLE pub_tbl REPLICA IDENTITY FULL;

Po tej komendzie możemy zupdtować rekord:

![alt text](image-15.png)

Sprawdźmy sub:

![alt text](image-16.png)

Jak widać jest pewne rozjechanie danych, wcześniej przez przypadek zupdatowałem rekord w sub, ta zmiana nie została oczywiście zreplikowana na pub, natomiast po zupdtowaniu tego samego rekordu w pub, zmiana ta nie została wykonana w sub. Dlatego w pub rekord o id 20 ma name=test-test a w sub name=test.

### Delete

Na pub:

> DELETE FROM pub_tbl WHERE id < 22 AND id > 10;

![alt text](image-17.png)

Sub:

![alt text](image-18.png)

Znowu możemy zaobserwować że rekord o id 20 nie został usunięty, jest jakby wyłączony z synchornizacji. Moglibśmy go ręcznie usunąc ale nie zrobimy tego by móc obserwować jak się będzie zachowywał przy następnych operacjach

### Truncate

Pub:

![alt text](image-19.png)


Sub:

![alt text](image-20.png)

Tej operacji nie przeżył rekord o id 20

### Add Column

Pub: 

![alt text](image-21.png)

Sub: 

![alt text](image-22.png)

Po dodaniu danych do pub, dostajemy takie errory odnośnie subskrypcji:

![alt text](image-23.png)

Wg dokumentacji postgresa (https://www.postgresql.org/docs/10/logical-replication-restrictions.html):

> The database schema and DDL commands are not replicated

Zatem ręcznie dodamy kolumne w sub:

i potej zmianie znowu mamy synchornizacje danych:

![alt text](image-24.png)

![alt text](image-25.png)

Następnie dodajemy nową kolumnę *city* na drugiej instancji:

![alt text](image-26.png)

Dodajmy kilka wierszy na pub:

![alt text](image-27.png)

Jak widać wiersze te zostały dodane na sub:

![alt text](image-28.png)

Jak widać różne schematy tabeli wcale nie muszą przeszkadzać w replikcaji danych. Tak zresztą możemy przeczytać w dokumentacji:

>Note, however, that there is no need for the schemas to be absolutely the same on both sides.

### PG_STAT

pub:

![alt text](image-29.png)

sub:

![alt text](image-30.png)

Dropujemy subskrypcje:

![alt text](image-31.png)

Pub:

![alt text](image-32.png)

