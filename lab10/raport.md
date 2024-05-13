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