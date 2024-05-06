Zaczynamy od stworzenia folderu:

> mkdir primary_db

I w tym folderze wykonujemy:

> /usr/lib/postgresql/15/bin/initdb -D ./

![alt text](_img/image1-kw.png)

Następnie zmodyfikowałem plik postgresql.conf i ustawiłem pola:

> listen_addres = '0.0.0.0'
> port = 5433

Następnie uruchomiłem instancje komendą:

> /usr/lib/postgresql/15/bin/pg_ctl start -D .

![alt text](_img/image2-kw.png)

Następnie zalgowałem się do tej instancji w celu stworzenia usera repuser:

> CREATE ROLE repuser WITH REPLICATION, LOGIN;

Na koniec pliku dodałem:
```
host    all     repuser     0.0.0.0/0   trust
```

Następnia zrestartowałem serwis:

> /usr/lib/postgresql/15/bin/pg_ctl restart -D .

![alt text](_img/image3-kw.png)

Następnie stworzyłem instancje repliki przy pomocy komendy:

> /usr/lib/postgresql/15/bin/pg_basebackup -h localhost -U repuser -p 5433 -D ./replica_db -R -C --slot=slot_name --checkpoint=fast

![alt text](_img/image4-kw.png)

> ls -al

![alt text](_img/image5-kw.png)

Następnie zmieniłem port w replice i wystartowałem instancje repliki

![alt text](_img/image6-kw.png)

Po zalogowaniu się na primary instance i wywołaniu:

> select * from pg_stat_replication;

![alt text](_img/image7-kw.png)

Natomiast po zalogowaniu się na backup instance i wywołaniu:

> select * from pg_stat_wal_receiver;

![alt text](_img/image8-kw.png)

Następnie przetestowałem replikacje danych:

> psql -p 5433
> CREATE TABLE weather (city varchar(80), temp int);
> INSER INTO WEATHER ('Krakow', 26);

![alt text](_img/image9-kw.png)

Sprawdźmy replike:

> psql -p 5434
> select * from weather

![alt text](_img/image10-kw.png)

Sprawdzenie operacji truncate:

![alt text](_img/image11-kw.png)

![alt text](_img/image12-kw.png)

Sprawdzenie operacji delete:

![alt text](_img/image13-kw.png)

![alt text](_img/image14-kw.png)

Symulacja awari primary servera

przy pomocy komendy:

> pg_ctl stop -D ./primary_db

stopujemy primary server:

![alt text](_img/image15-kw.png)

Jak widać dostajemy też errory z replica server

Zróbmy teraz ręcznego failovera przy pomocy komendy:

> pg_ctl promote -D ./replica_db

![alt text](_img/image16-kw.png)

Multi standby-setup:

Tworzymy dwie repliki:

> pg_basebackup -h localhost -U repuser -p 5433 -D ./replica1 -R -C --slot=slot_name1 --checkpoint1

> pg_basebackup -h localhost -U repuser -p 5433 -D ./replica1 -R -C --slot=slot_name1 --checkpoint1

Zmieńmy porty odpowiednio na `5434` i `5435` i uruchomy repliki:

![alt text](_img/image17-kw.png)

![alt text](_img/image18-kw.png)

Sprawdźmy czy wszystko działa:

![alt text](_img/image19-kw.png)

Jak widać instancja na porcie 5433 nie jest w recovery mode a pozostałe dwie są.

Cascade Setup

> ./pg_basebackup -p 5433 -U repuser -h 127.0.0.1 -D /tmp/replica3 -R -C --slot slot3 --checkpoint fast
i zmiana portu na `5436`

> ./pg_basebackup -p 5436 -U repuser -h 127.0.0.1 -D /tmp/replica4 -R -C --slot slot4 --checkpoint fast
i zmiana portu na `5437`

Teraz zobaczmy czy replikacja działa.

![alt text](_img/image20-bw.png)

![alt text](_img/image21-bw.png)

Jak widać replika o numerze 4 otrzymała dane od bazy primary za pomocą repliki 3.