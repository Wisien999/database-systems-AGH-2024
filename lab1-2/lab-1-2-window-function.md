
# SQL - Funkcje okna (Window functions) 

# Lab 1-2

---
**Imię i nazwisko:**

--- 


Celem ćwiczenia jest zapoznanie się z działaniem funkcji okna (window functions) w SQL, analiza wydajności zapytań i porównanie z rozwiązaniami przy wykorzystaniu "tradycyjnych" konstrukcji SQL

Swoje odpowiedzi wpisuj w miejsca oznaczone jako:

```sql
-- wyniki ...
```

Ważne/wymagane są komentarze.

Zamieść kod rozwiązania oraz zrzuty ekranu pokazujące wyniki, (dołącz kod rozwiązania w formie tekstowej/źródłowej)

Zwróć uwagę na formatowanie kodu

---

## Oprogramowanie - co jest potrzebne?

Do wykonania ćwiczenia potrzebne jest następujące oprogramowanie:
- MS SQL Server - wersja 2019, 2022
- PostgreSQL - wersja 15/16
- SQLite
- Narzędzia do komunikacji z bazą danych
	- SSMS - Microsoft SQL Managment Studio
	- DtataGrip lub DBeaver
-  Przykładowa baza Northwind
	- W wersji dla każdego z wymienionych serwerów

Oprogramowanie dostępne jest na przygotowanej maszynie wirtualnej

## Dokumentacja/Literatura

- Kathi Kellenberger,  Clayton Groom, Ed Pollack, Expert T-SQL Window Functions in SQL Server 2019, Apres 2019
- Itzik Ben-Gan, T-SQL Window Functions: For Data Analysis and Beyond, Microsoft 2020

- Kilka linków do materiałów które mogą być pomocne
	 - https://learn.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql?view=sql-server-ver16
	- https://www.sqlservertutorial.net/sql-server-window-functions/
	- https://www.sqlshack.com/use-window-functions-sql-server/
	- https://www.postgresql.org/docs/current/tutorial-window.html
	- https://www.postgresqltutorial.com/postgresql-window-function/
	-  https://www.sqlite.org/windowfunctions.html
	- https://www.sqlitetutorial.net/sqlite-window-functions/

- Ikonki używane w graficznej prezentacji planu zapytania w SSMS opisane są tutaj:
	- [https://docs.microsoft.com/en-us/sql/relational-databases/showplan-logical-and-physical-operators-reference](https://docs.microsoft.com/en-us/sql/relational-databases/showplan-logical-and-physical-operators-reference)

---
# Zadanie 1 - obserwacja

Wykonaj i porównaj wyniki następujących poleceń.

```sql
select avg(unitprice) avgprice
from products p;

select avg(unitprice) over () as avgprice
from products p;

select categoryid, avg(unitprice) avgprice
from products p
group by categoryid

select avg(unitprice) over (partition by categoryid) as avgprice
from products p;
```

Jaka jest są podobieństwa, jakie różnice pomiędzy grupowaniem danych a działaniem funkcji okna?

### Wyniki

Zapytania zrealizowane w bazie danych **Postgres**
#### Zapytanie 1
```sql
select avg(unitprice) avgprice
from products p;
```

 ![[_img/Pasted image 20240304183527.png]]

To zapytanie policzyło średnią z **unitprice** z wszystkich wierszy w tabeli.
#### Zapytanie 2
```sql
select avg(unitprice) over () as avgprice
from products p;
```
![[_img/Pasted image 20240304183602.png]]
Funkcja okna bez Partition policzyła średnią **unitPrice** z wszystkich wierszów, następnie zwróciła tą wartość dla każdego wiersza w tabeli.
#### Zapytanie 3
```sql
select categoryid, avg(unitprice) avgprice
from products p
group by categoryid
```

![[_img/Pasted image 20240304183631.png]]
Powyższe zapytanie pogrupowało produkty po **categoryId** a następnie dla każdej grupy policzyła średnią z **unitPrice**. Jako wynik zwróciła tabele pogrupowaną po **categoryId**

#### Zapytanie 4
```sql
select avg(unitprice) over (partition by categoryid) as avgprice
from products p;
```

![[_img/Pasted image 20240304183652.png]]
Funkcja okna z partition policzyła średnią dla każdej kategorii, następnie jako wynik zwróciła wszystkie wiersze z wartością odpowiadającej średniej dla kategorii do której dany wiersz należy

---
# Zadanie 2 - obserwacja

Wykonaj i porównaj wyniki następujących poleceń.

```sql
--1)

select p.productid, p.ProductName, p.unitprice,
       (select avg(unitprice) from products) as avgprice
from products p
where productid < 10

--2)
select p.productid, p.ProductName, p.unitprice,
       avg(unitprice) over () as avgprice
from products p
where productid < 10
```


Jaka jest różnica? Czego dotyczy warunek w każdym z przypadków? Napisz polecenie równoważne 
- 1) z wykorzystaniem funkcji okna. Napisz polecenie równoważne 
- 2) z wykorzystaniem podzapytania

### Wyniki

#### Zapytanie 1

```sql
select p.productid, p.ProductName, p.unitprice,
       (select avg(unitprice) from products) as avgprice
from products p
where productid < 10
```

![[_img/Pasted image 20240304185604.png]]

To zapytanie najpierw liczy średnią dla wszystkich produktów, następnie ogranicza wynikową tabele do wierszy z **productId** < 10
#### Zapytanie 2

```sql
select p.productid, p.ProductName, p.unitprice,
       avg(unitprice) over () as avgprice
from products p
where productid < 10
```

![[_img/Pasted image 20240304185721.png]]

Natomiast w tym zapytaniu średnia liczona jest z produktów gdzie **productId** < 10

#### Równoważne 1

(spróbować z with)

```sql
select distinct p.productid, p.ProductName, p.unitprice,  
    avg(sp.unitprice) over () as avgprice  
from products p, (select * from products) sp  
where p.productid < 10;
```

Ponieważ funkcja okna wykonuje się po klauzuli **WHERE**, to dodaliśmy do zapytania kolejną tabele, która nie jest ograniczona przez **WHERE**  i to na niej policzyliśmy średnią

![[_img/Pasted image 20240304194655.png]]
#### Równoważne 2

```sql
select p.productid, p.ProductName, p.unitprice,  
    (select avg(unitprice) from (select * from products sp where sp.productid < 10 )) as avgprice  
from products p  
where productid < 10;
```

Sposobem na uniknięcie korzystania z funkcji okna jest policzenie średniej z tabeli już ograniczonej do **productId** < 10

![[_img/Pasted image 20240304194034.png]]
# Zadanie 3

Baza: Northwind, tabela: products

Napisz polecenie, które zwraca: id produktu, nazwę produktu, cenę produktu, średnią cenę wszystkich produktów.

Napisz polecenie z wykorzystaniem z wykorzystaniem podzapytania, join'a oraz funkcji okna. Porównaj czasy oraz plany wykonania zapytań.

Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

W SSMS włącz dwie opcje: Include Actual Execution Plan oraz Include Live Query Statistics

![w:700](_img/window-1.png)

W DataGrip użyj opcji Explain Plan/Explain Analyze

![w:700](_img/window-2.png)


![w:700](_img/window-3.png)

### Wyniki

```sql
--- subquery)
select p.ProductID, p.ProductName, p.UnitPrice, (select avg(unitprice) from products) as avgprice
from Products p

--- join)
select p.productid, p.productname, p.unitprice, avgprices.avgprice
from Products p
inner join (select avg(unitprice) as avgprice from Products) avgprices on 1=1

--- window) 
select p.productid, p.productname, p.unitprice, avg(p.unitprice) over () as avgprice
from Products p
```
Brak porównania czasu wykonania, bo przy tabeli rozmiaru 77 nie ma ono większego sensu.

### MS SQL Server
#### Query with join
![w:700](_img/zad3-mssql-join-exec-plan.png)
![w:700](_img/zad3-mssql-join-live-query.png)
Cost: 0.007102

#### Query with widnow function
![w:700](_img/zad3-mssql-window-exec-plan.png)
![w:700](_img/zad3-mssql-window-live-query.png)
Cost: 0.0048

#### Query with subquery
![w:700](_img/zad3-mssql-subquery-exec-plan.png)
![w:700](_img/zad3-mssql-subquery-live-query.png)
Cost: 0.007109

Jak widać najmniejszy koszt ma wersja query z funkcją okna.

### PostgreSQL
#### Query with join
![w:700](_img/zad3-postgres-join-exec-plan.png)

#### Query with widnow function
![w:700](_img/zad3-postgres-window-exec-plan.png)

#### Query with subquery
![w:700](_img/zad3-postgres-subquery-exec-plan.png)

Ponownie najmniejszy koszt ma wersja query z funkcją okna.

### SQLite
#### Query with join
![w:700](_img/zad3-sqllite-join-exec-plan.png)

#### Query with widnow function
![w:700](_img/zad3-sqllite-window-exec-plan.png)

#### Query with subquery
![w:700](_img/zad3-sqllite-subquery-exec-plan.png)

Datagrip nie wpiera dokładnej analizy kosztu w przypadku SQLite.

---

# Zadanie 4

Baza: Northwind, tabela products

Napisz polecenie, które zwraca: id produktu, nazwę produktu, cenę produktu, średnią cenę produktów w kategorii, do której należy dany produkt. Wyświetl tylko pozycje (produkty) których cena jest większa niż średnia cena.

Napisz polecenie z wykorzystaniem podzapytania, join'a oraz funkcji okna. Porównaj zapytania. Porównaj czasy oraz plany wykonania zapytań.

Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

```sql
-- subquery
select productid, productname, unitprice, (select AVG(unitprice) FROM products AS P2 where P2.categoryid = P1.categoryid) AS avg_price FROM products AS P1
where unitprice > (select AVG(unitprice) FROM products AS P2 where P2.categoryid = P1.categoryid);

-- join
SELECT
    P1.productid,
    P1.productname,
    P1.unitprice,
    AVG(P2.unitprice) AS avg_price
FROM
    products AS P1
        JOIN
    products AS P2 ON P1.categoryid = P2.categoryid
GROUP BY
    P1.productid, P1.productname, P1.unitprice
HAVING P1.unitprice > AVG(P2.unitprice);

-- window function
SELECT productid, productname, unitprice, avg
FROM
    (SELECT productid, productname, unitprice, AVG(unitprice) over (partition by categoryid) AS avg
     FROM products
    ) AS ss
WHERE unitprice > avg;
```
- **Zapytania**

    ***Subquery*** wykorzystuje podzapytanie do uzyskania średniej ceny dla danej kategorii, a następnie porównuje jednostkową cenę produktu z tą średnią.

    ***Join*** wykorzystuje operację JOIN oraz GROUP BY do uzyskania średniej ceny dla danej kategorii, a następnie porównuje jednostkową cenę produktu z tą średnią.

    ***Window Function*** wykorzystuje funkcję okna do uzyskania średniej ceny dla danej kategorii, a następnie porównuje jednostkową cenę produktu z tą średnią.

- **Czas**

    **PostgreSQL**
    | Zapytanie | subquery  | join  | window function |
    | ---       | ---       | ---   |---              |
    | Czas      | 91ms      | 20ms  | 12ms            |

    **SQL Server**
    | Zapytanie | subquery  | join  | window function |
    | ---       | ---       | ---   |---              |
    | Czas      | 91ms      | 14ms  | 19ms             |

    **SQLite**
    | Zapytanie | subquery  | join  | window function |
    | ---       | ---       | ---   |---              |
    | Czas      | 42ms      | 10ms  | 8ms            |

    Jak wynika z wyników, najszybszym rozwiązaniem okazał się SQLite, a najszybszym zapytaniem zapytanie wykorzystujące funkcje okna.

- **Plany wykonania**
    **PostgreSQL**
    ![alt text](./_img/zad4_1.png)
    ![alt text](./_img/zad4_2.png)
    ![alt text](./_img/zad4_3.png)
    ![alt text](./_img/zad4_4.png)
    ![alt text](./_img/zad4_5.png)
    ![alt text](./_img/zad4_6.png)

    Jak widać koszt oraz czas zapytań jest najlepszy w przypadku funkcji okna.

    Plany wykonania zapytań są proste i przejrzyste.

    **SQL Server**
    ![alt text](./_img/zad4_7.png)
    ![alt text](./_img/zad4_8.png)
    ![alt text](./_img/zad4_9.png)
    ![alt text](./_img/zad4_10.png)
    ![alt text](./_img/zad4_11.png)
    ![alt text](./_img/zad4_12.png)

    Koszt zapytań jest najlepszy w przypadku funkcji okna i wykorzystania join'a. Oba wyniki są sobie bliskie. 
    
    Plany wykonania zapytań są dużo bardziej skomplikowane niż w przypadku PostgreSQL.

    **SQLite**
    Dla tego serwera bazodanowego DataGrip nie pozwala zobaczyć analizy zapytań.

---
# Zadanie 5 - przygotowanie

Baza: Northwind

Tabela products zawiera tylko 77 wiersz. Warto zaobserwować działanie na większym zbiorze danych.

Wygeneruj tabelę zawierającą kilka milionów (kilkaset tys.) wierszy

Stwórz tabelę o następującej strukturze:

Skrypt dla SQL Srerver

```sql
create table product_history(
   id int identity(1,1) not null,
   productid int,
   productname varchar(40) not null,
   supplierid int null,
   categoryid int null,
   quantityperunit varchar(20) null,
   unitprice decimal(10,2) null,
   quantity int,
   value decimal(10,2),
   date date,
 constraint pk_product_history primary key clustered
    (id asc )
)
```

Wygeneruj przykładowe dane:

Dla 30000 iteracji, tabela będzie zawierała nieco ponad 2mln wierszy (dostostu ograniczenie do możliwości swojego komputera)

Skrypt dla SQL Srerver

```sql
declare @i int  
set @i = 1  
while @i <= 30000  
begin  
    insert product_history  
    select productid, ProductName, SupplierID, CategoryID,   
         QuantityPerUnit,round(RAND()*unitprice + 10,2),  
         cast(RAND() * productid + 10 as int), 0,  
         dateadd(day, @i, '1940-01-01')  
    from products  
    set @i = @i + 1;  
end;  
  
update product_history  
set value = unitprice * quantity  
where 1=1;
```


Skrypt dla Postgresql

```sql
create table product_history(
   id int generated always as identity not null  
       constraint pkproduct_history
            primary key,
   productid int,
   productname varchar(40) not null,
   supplierid int null,
   categoryid int null,
   quantityperunit varchar(20) null,
   unitprice decimal(10,2) null,
   quantity int,
   value decimal(10,2),
   date date
);
```

Wygeneruj przykładowe dane:

Skrypt dla Postgresql

```sql
do $$  
begin  
  for cnt in 1..30000 loop  
    insert into product_history(productid, productname, supplierid,   
           categoryid, quantityperunit,  
           unitprice, quantity, value, date)  
    select productid, productname, supplierid, categoryid,   
           quantityperunit,  
           round((random()*unitprice + 10)::numeric,2),  
           cast(random() * productid + 10 as int), 0,  
           cast('1940-01-01' as date) + cnt  
    from products;  
  end loop;  
end; $$;  
  
update product_history  
set value = unitprice * quantity  
where 1=1;
```

Skrypt dla SQLite

```sql
CREATE TABLE IF NOT EXISTS product_history (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    productid INTEGER,
    productname TEXT NOT NULL,
    supplierid INTEGER,
    categoryid INTEGER,
    quantityperunit TEXT,
    unitprice REAL,
    quantity INTEGER,
    value REAL,
    date DATE
);
```

Wygeneruj przykładowe dane:

Skrypt dla SQLite

```sql
WITH RECURSIVE cnt(i) AS (
    SELECT 1
    UNION ALL
    SELECT i+1 FROM cnt WHERE i < 30000
)
INSERT INTO product_history (productid, productname, supplierid, categoryid, quantityperunit, unitprice, quantity, value, date)
SELECT
    productid,
    productname,
    supplierid,
    categoryid,
    quantityperunit,
    ROUND((ABS(RANDOM() % 10)) * unitprice + 10, 2),
    CAST(ABS(RANDOM() % 10) * productid + 10 AS INTEGER),
    0,
    date('1940-01-01', '+' || cnt.i || ' days')
FROM
    products,
    cnt;

UPDATE product_history
SET value = unitprice * quantity;
```

Wykonaj polecenia: `select count(*) from product_history`,  potwierdzające wykonanie zadania

```sql
select count(*) from product_history
```
**PostgreSQL**
![alt text](./_img/zad5_1.png)

**SQL Server**
![alt text](./_img/zad5_2.png)

**SQLite**
![alt text](./_img/zad5_3.png)

---
# Zadanie 6

Baza: Northwind, tabela product_history

To samo co w zadaniu 3, ale dla większego zbioru danych

Napisz polecenie, które zwraca: id pozycji, id produktu, nazwę produktu, cenę produktu, średnią cenę produktów w kategorii do której należy dany produkt. Wyświetl tylko pozycje (produkty) których cena jest większa niż średnia cena.

Napisz polecenie z wykorzystaniem podzapytania, join'a oraz funkcji okna. Porównaj zapytania. Porównaj czasy oraz plany wykonania zapytań.

Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

### Wyniki

```sql
-- subquery
select productid, productname, unitprice, (select AVG(unitprice) FROM product_history AS P2 where P2.categoryid = P1.categoryid) AS avg_price FROM product_history AS P1
where unitprice > (select AVG(unitprice) FROM product_history AS P2 where P2.categoryid = P1.categoryid);

-- join
SELECT
    P1.productid,
    P1.productname,
    P1.unitprice,
    AVG(P2.unitprice) AS avg_price
FROM
    product_history AS P1
        JOIN
    product_history AS P2 ON P1.categoryid = P2.categoryid
GROUP BY
    P1.productid, P1.productname, P1.unitprice
HAVING P1.unitprice > AVG(P2.unitprice);

-- window function
SELECT productid, productname, unitprice, avg
FROM
    (SELECT productid, productname, unitprice, AVG(unitprice) over (partition by categoryid) AS avg
     FROM product_history
    ) AS ss
WHERE unitprice > avg;
```

Dla dwóch milionów rekordów w tabelach `product_history` wykonanie zapytań trwało bardzo długo. Po kilku minutach zdecydowaliśmy się zmniejszyć ilość rekordów w tabelach do miliona.

- **Czas**

    **PostgreSQL**
    | Zapytanie | subquery  | join  | window function |
    | ---       | ---       | ---   |---              |
    | Czas      | > 1m      | > 1m  | 741ms           |

    **SQL Server**
    | Zapytanie | subquery  | join  | window function |
    | ---       | ---       | ---   |---              |
    | Czas      | 280ms     | 350ms | 250ms           |

    **SQLite**
    | Zapytanie | subquery  | join  | window function |
    | ---       | ---       | ---   |---              |
    | Czas      | > 1m      | > 1m  | 692ms           |

    W przypadku PostgreSQL oraz SQLite tylko zapytania wykorzystujące funkcje okna liczyły się w rozsądnym czasie. Pozostałe zajmowały ponad kilka minut, więc zdecydowaliśmy się je przerwać. 
    W przypadku SQLServer każde zapytanie liczyło się szybko i nie było między nimi dużych różnic czasowych.

- **Plany wykonania**
    **PostgreSQL**
    ![alt text](./_img/zad6_1.png)
    ![alt text](./_img/zad6_2.png)

    Plan wykonania zapytań jest prosty i przejrzysty.

    **SQL Server**
    ![alt text](./_img/zad6_3.png)
    ![alt text](./_img/zad6_4.png)
    ![alt text](./_img/zad6_5.png)
    ![alt text](./_img/zad6_6.png)
    ![alt text](./_img/zad6_7.png)
    ![alt text](./_img/zad6_8.png)

    Koszt zapytań jest najlepszy w przypadku funkcji okna.

    **SQLite**
    Dla tego serwera bazodanowego DataGrip nie pozwala zobaczyć analizy zapytań.

---
# Zadanie 7

Baza: Northwind, tabela product_history

Lekka modyfikacja poprzedniego zadania

Napisz polecenie, które zwraca: id pozycji, id produktu, nazwę produktu, cenę produktu oraz
-  średnią cenę produktów w kategorii do której należy dany produkt.
-  łączną wartość sprzedaży produktów danej kategorii (suma dla pola value)
-  średnią cenę danego produktu w roku którego dotyczy dana pozycja
- łączną wartość sprzedaży produktów danej kategorii (suma dla pola value)

Napisz polecenie z wykorzystaniem podzapytania, join'a oraz funkcji okna. Porównaj zapytania. W przypadku funkcji okna spróbuj użyć klauzuli WINDOW.

Porównaj czasy oraz plany wykonania zapytań.

Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

### Wyniki

```sql
SELECT
    ph.productid,
    ph.ProductName,
    ph.date,
    ph.unitprice,
    (SELECT AVG(unitprice) FROM product_history WHERE CategoryID = ph.CategoryID) AS avgPrice,
    (SELECT SUM(value) FROM product_history WHERE CategoryID = ph.CategoryID) AS total,
    (SELECT AVG(unitprice) FROM product_history WHERE productid = ph.productid AND YEAR(date) = YEAR(ph.date)) AS avgYear
FROM
    product_history ph
WHERE
    ph.UnitPrice > (SELECT AVG(unitprice) FROM product_history WHERE CategoryID = ph.CategoryID)
ORDER BY
    ph.productid, ph.date;
```

```sql
SELECT
    ph1.productid,
    ph1.ProductName,
    ph1.unitprice,
    ph1.date,
    AVG(ph2.UnitPrice) AS avgPrice,
    SUM(ph2.value) AS total,
    AVG(ph3.UnitPrice) AS avgYear
FROM product_history ph1
JOIN product_history ph2 ON ph2.CategoryID = ph1.CategoryID
JOIN product_history ph3 ON ph3.productid = ph1.productid AND YEAR(ph1.date) = YEAR(ph3.date)
GROUP BY
    ph1.productid,
    ph1.ProductName,
    ph1.unitprice,
    ph1.date
HAVING ph1.unitprice > AVG(ph2.UnitPrice)
ORDER BY ph1.productid, ph1.date;
```

```sql
SELECT
    nph1.productid,
    nph1.ProductName,
    nph1.unitprice,
    nph1.date,
    nph1.avgPrice,
    nph1.total,
    nph1.avgYear
FROM (
    SELECT
        ph.productid,
        ph.ProductName,
        ph.unitprice,
        ph.date,
        AVG(ph.unitprice) OVER w1 AS avgPrice,
        SUM(ph.value) OVER w1 AS total,
        AVG(ph.unitprice) OVER w2 AS avgYear
    FROM product_history ph
    WINDOW w1 AS (PARTITION BY ph.CategoryID),
    w2 AS (PARTITION BY ph.productid, YEAR(ph.date))
) AS nph1
ORDER BY nph1.productid, nph1.date;
```

- **Czas**

    **PostgreSQL**
    | Zapytanie | subquery  | join  | window function |
    | ---       | ---       | ---   |---              |
    | Czas      | > 1m      | > 1m  | 3 s 822 ms      |

    **SQL Server**
    | Zapytanie | subquery  | join  | window function |
    | ---       | ---       | ---   |---              |
    | Czas      | 439ms     | 357ms | 331ms           |

    **SQLite**
    | Zapytanie | subquery  | join  | window function |
    | ---       | ---       | ---   |---              |
    | Czas      | > 1m      | > 1m  | 534ms           |

    W przypadku PostgreSQL oraz SQLite tylko zapytania wykorzystujące funkcje okna liczyły się w rozsądnym czasie. Pozostałe zajmowały ponad kilka minut, więc zdecydowaliśmy się je przerwać. 
    W przypadku SQLServer każde zapytanie liczyło się szybko i nie było między nimi dużych różnic czasowych.

- **Plany wykonania**
    **PostgreSQL**
    ![alt text](./_img/zad7_1.png)

    Plan wykonania zapytań jest prosty i przejrzyste.

    **SQL Server**
    ![alt text](./_img/zad7_2.png)
    ![alt text](./_img/zad7_3.png)
    ![alt text](./_img/zad7_4.png)

    Koszt zapytań jest najlepszy w przypadku funkcji okna.

    **SQLite**
    Dla tego serwera bazodanowego DataGrip nie pozwala zobaczyć analizy zapytań.

---
# Zadanie 8 - obserwacja

Funkcje rankingu, `row_number()`, `rank()`, `dense_rank()`

Wykonaj polecenie, zaobserwuj wynik. Porównaj funkcje row_number(), rank(), dense_rank()

```sql 
select productid, productname, unitprice, categoryid,  
    row_number() over(partition by categoryid order by unitprice desc) as rowno,  
    rank() over(partition by categoryid order by unitprice desc) as rankprice,  
    dense_rank() over(partition by categoryid order by unitprice desc) as denserankprice  
from products;
```

![w:700](_img/zad8-result.png)

Zadanie

Spróbuj uzyskać ten sam wynik bez użycia funkcji okna

```sql
SELECT 
    p.productid, p.productname, p.unitprice, p.categoryid,
	(
	select count(*) + 1 from products p2
	where (p2.categoryid = p.categoryid and p2.UnitPrice > p.UnitPrice) or (p2.categoryid = p.categoryid and p2.UnitPrice = p.UnitPrice and p2.ProductID < p.ProductID)
	) as rownno,
    (SELECT COUNT(*) + 1
     FROM products p2
     WHERE p2.categoryid = p.categoryid AND p2.unitprice > p.unitprice) AS rankprice,
    (SELECT COUNT(distinct unitprice) + 1
     FROM products p2
     WHERE p2.categoryid = p.categoryid AND p2.unitprice > p.unitprice) AS denserankprice
FROM products p
order by categoryid asc, unitprice desc, productid asc;
```


---
# Zadanie 9

Baza: Northwind, tabela product_history

Dla każdego produktu, podaj 4 najwyższe ceny tego produktu w danym roku. Zbiór wynikowy powinien zawierać:
- rok
- id produktu
- nazwę produktu
- cenę
- datę (datę uzyskania przez produkt takiej ceny)
- pozycję w rankingu

Uporządkuj wynik wg roku, nr produktu, pozycji w rankingu.

### Wyniki

```sql
--- PostgreSQL
WITH RankedPrice AS (
    SELECT
        EXTRACT(year from PH.date) AS Year,
        PH.ProductID,
        PH.ProductName,
        PH.UnitPrice,
        ROW_NUMBER() OVER (PARTITION BY PH.ProductID, EXTRACT(year from PH.date) ORDER BY PH.UnitPrice DESC) AS PriceRank
    FROM
        product_history PH
)

SELECT
    *
FROM
    RankedPrice
ORDER BY
    Year, ProductID, PriceRank;

--- SQLServer
WITH RankedPrice AS (
    SELECT
        YEAR(PH.Date) AS Year,
        PH.ProductID,
        PH.ProductName,
        PH.UnitPrice,
        ROW_NUMBER() OVER (PARTITION BY PH.ProductID, YEAR(PH.Date) ORDER BY PH.UnitPrice DESC) AS PriceRank
    FROM
        product_history PH
)

SELECT
    *
FROM
    RankedPrice
ORDER BY
    Year, ProductID, PriceRank;

--- SQLite
WITH RankedPrice AS (
    SELECT
        strftime('%Y', PH.date) AS Year,
        PH.ProductID,
        PH.ProductName,
        PH.UnitPrice,
        ROW_NUMBER() OVER (PARTITION BY PH.ProductID, strftime('%Y', PH.date) ORDER BY PH.UnitPrice DESC) AS PriceRank
    FROM
        product_history PH
)

SELECT
    *
FROM
    RankedPrice
ORDER BY
    Year, ProductID, PriceRank;
```

- **Czas**

    | Serwer | PostgreSQL  | SQLServer  | SQLite     |
    | ---    | ---         | ---        |---         |
    | Czas   | 14 s 448 ms | 2 s 778 ms | 16 s 16 ms |

    Najwolniejszy okazał się SQLite mając czas zbliżony do PostgreSQL, a najszybszy mający wielokrotnie lepszy czas okazał się SQL Server.

- **Plany wykonania**

    **PostgreSQL**
    ![alt text](./_img/zad9_1.png)

    **SQL Server**
    ![alt text](./_img/zad9_2.png)

    Koszt jest znacznie mniejszy niż w przypadku PostgreSQL.

    **SQLite**
    Dla tego serwera bazodanowego DataGrip nie pozwala zobaczyć analizy zapytań.

Spróbuj uzyskać ten sam wynik bez użycia funkcji okna, porównaj wyniki, czasy i plany zapytań. Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

### Wyniki

```sql
--- PostgreSQL
SELECT
    EXTRACT(year from PH.date) AS Year,
    PH.ProductID,
    PH.ProductName,
    PH.UnitPrice,
    (
        SELECT
            COUNT(DISTINCT PH2.UnitPrice) + 1
        FROM
            product_history PH2
        WHERE
            EXTRACT(year from PH2.date) = EXTRACT(year from PH.date)
            AND PH2.ProductID = PH.ProductID
            AND PH2.UnitPrice > PH.UnitPrice
    ) AS PriceRank
FROM
    product_history PH
ORDER BY
    Year, ProductID, PriceRank;

--- SQLServer
SELECT
    YEAR(PH.date) AS Year,
    PH.ProductID,
    PH.ProductName,
    PH.UnitPrice,
    (
        SELECT
            COUNT(DISTINCT PH2.UnitPrice) + 1
        FROM
            product_history PH2
        WHERE
            YEAR(PH2.date) = YEAR(PH.date)
          AND PH2.ProductID = PH.ProductID
          AND PH2.UnitPrice > PH.UnitPrice
    ) AS PriceRank
FROM
    product_history PH
ORDER BY
    Year, ProductID, PriceRank;

--- SQLite
SELECT
    strftime('%Y', PH.date) AS Year,
    PH.ProductID,
    PH.ProductName,
    PH.UnitPrice,
    (
        SELECT
            COUNT(DISTINCT PH2.UnitPrice) + 1
        FROM
            product_history PH2
        WHERE
            strftime('%Y', PH2.date) = strftime('%Y', PH.date)
          AND PH2.ProductID = PH.ProductID
          AND PH2.UnitPrice > PH.UnitPrice
    ) AS PriceRank
FROM
    product_history PH
ORDER BY
    Year, ProductID, PriceRank;
```

- **Czas**

    | Serwer | PostgreSQL  | SQLServer  | SQLite     |
    | ---    | ---         | ---        |---         |
    | Czas   |  |  |  |


- **Plany wykonania**

    **PostgreSQL**

    **SQL Server**

    **SQLite**

---
# Zadanie 10 - obserwacja

Funkcje `lag()`, `lead()`

Wykonaj polecenia, zaobserwuj wynik. Jak działają funkcje `lag()`, `lead()`

```sql
select productid, productname, categoryid, date, unitprice,  
       lag(unitprice) over (partition by productid order by date)   
as previousprodprice,  
       lead(unitprice) over (partition by productid order by date)   
as nextprodprice  
from product_history  
where productid = 1 and year(date) = 2022  
order by date;  
  
with t as (select productid, productname, categoryid, date, unitprice,  
                  lag(unitprice) over (partition by productid   
order by date) as previousprodprice,  
                  lead(unitprice) over (partition by productid   
order by date) as nextprodprice  
           from product_history  
           )  
select * from t  
where productid = 1 and year(date) = 2022  
order by date;
```

### Wyniki

```sql
-- wyniki ...
```

Zadanie

Spróbuj uzyskać ten sam wynik bez użycia funkcji okna, porównaj wyniki, czasy i plany zapytań. Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

```sql
select PH.productid, PH.productname, PH.categoryid, PH.date, PH.unitprice, (
    select
        PH2.unitprice
    from
        product_history PH2
    where
        PH2.productid = PH.productid and
        PH2.date = dateadd(day, -1, PH.date) and
        year(dateadd(day, -1, PH.date)) = 2022
) as previousprodprice, (
           select
               PH2.unitprice
           from
               product_history PH2
           where
               PH2.productid = PH.productid and
               PH2.date = dateadd(day, 1, PH.date)
       ) as nextprodprice
from product_history PH
where PH.productid = 1 and year(PH.date) = 2022
order by PH.date;

select PH.productid, PH.productname, PH.categoryid, PH.date, PH.unitprice, (
    select
        PH2.unitprice
    from
        product_history PH2
    where
        PH2.productid = PH.productid and
        PH2.date = dateadd(day, -1, PH.date)
) as previousprodprice, (
           select
               PH2.unitprice
           from
               product_history PH2
           where
               PH2.productid = PH.productid and
               PH2.date = dateadd(day, 1, PH.date)
       ) as nextprodprice
from product_history PH
where PH.productid = 1 and year(PH.date) = 2022
order by PH.date;
```

---
# Zadanie 11

Baza: Northwind, tabele customers, orders, order details

Napisz polecenie które wyświetla inf. o zamówieniach

Zbiór wynikowy powinien zawierać:
- nazwę klienta, nr zamówienia,
- datę zamówienia,
- wartość zamówienia (wraz z opłatą za przesyłkę),
- nr poprzedniego zamówienia danego klienta,
- datę poprzedniego zamówienia danego klienta,
- wartość poprzedniego zamówienia danego klienta.

### Wyniki

```sql
with Data as (select
    O.OrderID,
    C.CompanyName,
    O.OrderDate,
    O.Freight + sum(OD.UnitPrice * OD.Quantity * (1 - OD.Discount)) as Cost,
    lag(O.OrderID) over (partition by C.CustomerID order by O.OrderDate) as PrevOrderID,
    lag(O.OrderDate) over (partition by C.CustomerID order by O.OrderDate) as PrevOrderDate
from
    Orders O
join Customers C
    on O.CustomerID = C.CustomerID
join [Order Details] OD
    on O.OrderID = OD.OrderID
group by
    O.OrderID, C.CustomerID, C.CompanyName, O.OrderDate, O.Freight)
select
    Data.*,
    O.Freight + sum(OD.UnitPrice * OD.Quantity * (1 - OD.Discount)) as PrevCost
from
    Data
left join Orders O
    on O.OrderID = PrevOrderID
left join [Order Details] OD
    on O.OrderID = OD.OrderID
group by
    O.Freight, Data.OrderID, Data.CompanyName, Data.OrderDate, Data.Cost, Data.PrevOrderID, Data.PrevOrderDate
order by
    Data.OrderID;
```


---
# Zadanie 12 - obserwacja

Funkcje `first_value()`, `last_value()`

Wykonaj polecenia, zaobserwuj wynik. Jak działają funkcje `first_value()`, `last_value()`. Skomentuj uzyskane wyniki. Czy funkcja `first_value` pokazuje w tym przypadku najdroższy produkt w danej kategorii, czy funkcja `last_value()` pokazuje najtańszy produkt? Co jest przyczyną takiego działania funkcji `last_value`. Co trzeba zmienić żeby funkcja last_value pokazywała najtańszy produkt w danej kategorii

```sql
select productid, productname, unitprice, categoryid,  
    first_value(productname) over (partition by categoryid   
order by unitprice desc) first,  
    last_value(productname) over (partition by categoryid   
order by unitprice desc) last  
from products  
order by categoryid, unitprice desc;
```

### Wyniki

```sql
-- wyniki ...
```

Zadanie

Spróbuj uzyskać ten sam wynik bez użycia funkcji okna, porównaj wyniki, czasy i plany zapytań. Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

```sql
select p.productid, p.productname, p.unitprice, p.categoryid,  
    (select top 1 p2.productname from products p2 where p2.CategoryID=p.CategoryID order by p2.UnitPrice desc)first,
	(select top 1 p2.productname from products p2 where p2.CategoryID=p.CategoryID order by p2.UnitPrice)last
from products  p
order by p.categoryid, p.unitprice desc;
```

---
# Zadanie 13

Baza: Northwind, tabele orders, order details

Napisz polecenie które wyświetla inf. o zamówieniach

Zbiór wynikowy powinien zawierać:
- Id klienta,
- nr zamówienia,
- datę zamówienia,
- wartość zamówienia (wraz z opłatą za przesyłkę),
- dane zamówienia klienta o najniższej wartości w danym miesiącu
	- nr zamówienia o najniższej wartości w danym miesiącu
	- datę tego zamówienia
	- wartość tego zamówienia
- dane zamówienia klienta o najwyższej wartości w danym miesiącu
	- nr zamówienia o najniższej wartości w danym miesiącu
	- datę tego zamówienia
	- wartość tego zamówienia

### Wyniki

```sql
with Data as
         (
         select o.CustomerID as CustomerID, o.OrderID, o.OrderDate, o.Freight+od.UnitPrice*od.Quantity-od.Discount as value
         from orders o
         join [Order Details] od
            on od.OrderID = o.OrderID)
select
    d.CustomerID, d.OrderDate,d.OrderDate,d.value,
    last_value(concat(d.OrderID,' ', d.OrderDate,' ', d.value)) over (partition by d.CustomerID order by d.value desc rows between unbounded preceding and unbounded following ) min_value_order,
    first_value(concat(d.OrderID,' ', d.OrderDate,' ', d.value)) over (partition by d.CustomerID order by d.value desc ) max_value_order
from Data d
```

---
# Zadanie 14

Baza: Northwind, tabela product_history

Napisz polecenie które pokaże wartość sprzedaży każdego produktu narastająco od początku każdego miesiąca. Użyj funkcji okna

Zbiór wynikowy powinien zawierać:
- id pozycji
- id produktu
- datę
- wartość sprzedaży produktu w danym dniu
- wartość sprzedaży produktu narastające od początku miesiąca

### Wyniki

```sql
with Data as 
(select
	id, productid, date,
	sum(unitprice*quantity) over(partition by productid,convert(date,date)) dayValue
from product_history)
select 
	d.*,
	sum(d.dayValue) over(partition by d.productid, datepart(year,d.date),datepart(month,d.date) order by convert(date,date))
from Data d
```

Spróbuj wykonać zadanie bez użycia funkcji okna. Spróbuj uzyskać ten sam wynik bez użycia funkcji okna, porównaj wyniki, czasy i plany zapytań. Przetestuj działanie w różnych SZBD (MS SQL Server, PostgreSql, SQLite)

```sql
select
	ph1.id,
	ph1.productid,
	ph1.date
	,(select SUM(ph2.unitprice*ph2.quantity)  from product_history ph2 where convert(date,ph2.date) = convert(date,ph1.date) and ph1.productid=ph2.productid) dayValue
	,(select
		SUM(ph3.tDayValue) 
	from 
		(select 
			SUM(ph4.unitprice*ph4.quantity) as tDayValue,
			ph4.date as tDate
		from product_history ph4 
		where ph1.productid=ph4.productid
		group by ph4.date
		) as ph3 
	where datepart(year,ph3.tDate) =datepart(year,ph1.date) 
	and  datepart(month,ph3.tDate) =datepart(month,ph1.date) 
	and datepart(day,ph3.tDate)<=datepart(day,ph1.date)
	) as aggregatedValue
from product_history ph1
order by ph1.productid, ph1.date
```

---
# Zadanie 15

Wykonaj kilka "własnych" przykładowych analiz. Czy są jeszcze jakieś ciekawe/przydatne funkcje okna (z których nie korzystałeś w ćwiczeniu)? Spróbuj ich użyć w zaprezentowanych przykładach.

### Wyniki

**Klauzula `RANGE`**
```sql
SELECT 
    productid, 
    productname, 
    unitprice, 
    categoryid,
    FIRST_VALUE(productname) OVER (PARTITION BY categoryid
        ORDER BY unitprice DESC RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS first,
    LAST_VALUE(productname) OVER (PARTITION BY categoryid
        ORDER BY unitprice DESC RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS last
FROM 
    products
ORDER BY 
    categoryid, 
    unitprice DESC;
```
Za pomocą klauzuli `RANGE`, możemy określić zakres danych, który ma być uwzględniony w analizie. W tym przypadku analiza odbywa się dla każdej kategorii produktu, a zakres danych obejmuje wszystkie wartości od początku do bieżącego wiersza. W rezultacie, dla każdej kategorii produktów uzyskujemy pierwszą i ostatnią wartość produktu na podstawie sortowania według jednostkowej ceny w kolejności malejącej.

**Klauzula `ROWS`**
```sql
SELECT 
    productid, 
    productname, 
    unitprice, 
    categoryid,
    FIRST_VALUE(productname) OVER (PARTITION BY categoryid
        ORDER BY unitprice DESC) AS first,
    LAST_VALUE(productname) OVER (PARTITION BY categoryid
        ORDER BY unitprice DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS last
FROM 
    products
ORDER BY 
    categoryid, 
    unitprice DESC;
```
Za pomocą klauzuli `ROWS`, również analiza jest przeprowadzana dla każdej kategorii produktu, ale zakres danych jest definiowany w sposób bardziej szczegółowy. Funkcja FIRST_VALUE obejmuje cały zakres danych, ale LAST_VALUE jest ograniczona do zakresu od początku do bieżącego wiersza. Oznacza to, że dla każdej kategorii produktów uzyskujemy pierwszą wartość produktu dla całego zakresu, ale ostatnią wartość tylko dla danego wiersza.

Punktacja

|         |     |
| ------- | --- |
| zadanie | pkt |
| 1       | 0,5 |
| 2       | 0,5 |
| 3       | 1   |
| 4       | 1   |
| 5       | 0,5 |
| 6       | 2   |
| 7       | 2   |
| 8       | 0,5 |
| 9       | 2   |
| 10      | 1   |
| 11      | 2   |
| 12      | 1   |
| 13      | 2   |
| 14      | 2   |
| 15      | 2   |
| razem   | 20  |
