
# Indeksy,  optymalizator <br>Lab 5

<!-- <style scoped>
 p,li {
    font-size: 12pt;
  }
</style>  -->

<!-- <style scoped>
 pre {
    font-size: 8pt;
  }
</style>  -->


---

**Imię i nazwisko:**

Mateusz Skowron, Bartłomiej Wiśniewski, Karol Wrona

--- 

Celem ćwiczenia jest zapoznanie się z planami wykonania zapytań (execution plans), oraz z budową i możliwością wykorzystaniem indeksów (cz. 2.)

Swoje odpowiedzi wpisuj w miejsca oznaczone jako:

---
> Wyniki: 

```sql
--  ...
```

---

Ważne/wymagane są komentarze.

Zamieść kod rozwiązania oraz zrzuty ekranu pokazujące wyniki, (dołącz kod rozwiązania w formie tekstowej/źródłowej)

Zwróć uwagę na formatowanie kodu

## Oprogramowanie - co jest potrzebne?

Do wykonania ćwiczenia potrzebne jest następujące oprogramowanie
- MS SQL Server,
- SSMS - SQL Server Management Studio    
- przykładowa baza danych AdventureWorks2017.
    
Oprogramowanie dostępne jest na przygotowanej maszynie wirtualnej


## Przygotowanie  

Uruchom Microsoft SQL Managment Studio.
    
Stwórz swoją bazę danych o nazwie XYZ. 

```sql
create database lab5  
go  
  
use lab5  
go
```


## Dokumentacja/Literatura

Obowiązkowo:

- [https://docs.microsoft.com/en-us/sql/relational-databases/indexes/indexes](https://docs.microsoft.com/en-us/sql/relational-databases/indexes/indexes)
- [https://docs.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide](https://docs.microsoft.com/en-us/sql/relational-databases/sql-server-index-design-guide)
- [https://www.simple-talk.com/sql/performance/14-sql-server-indexing-questions-you-were-too-shy-to-ask/](https://www.simple-talk.com/sql/performance/14-sql-server-indexing-questions-you-were-too-shy-to-ask/)

Materiały rozszerzające:
- [https://www.sqlshack.com/sql-server-query-execution-plans-examples-select-statement/](https://www.sqlshack.com/sql-server-query-execution-plans-examples-select-statement/)

<div style="page-break-after: always;"></div>

# Zadanie 1 - Indeksy klastrowane I nieklastrowane

Skopiuj tabelę `Customer` do swojej bazy danych:

```sql
select * into customer from adventureworks2017.sales.customer
```

Wykonaj analizy zapytań:

```sql
select * from customer where storeid = 594  
  
select * from customer where storeid between 594 and 610
```

Zanotuj czas zapytania oraz jego koszt koszt:

---
> Wyniki: 

Zapytanie nr1:
czas: 1/100 sekundy
koszt: ~0.14

Zapytanie nr2:
czas: 1/100 sekundy
koszt: ~0.14

W każdym z zapytań wykonanu full table scan


Dodaj indeks:

```sql
create  index customer_store_cls_idx on customer(storeid)
```

Jak zmienił się plan i czas? Czy jest możliwość optymalizacji?

Po dodaniu indeksu:

Zapytanie nr1:
czas: 1/100 sekundy
koszt: ~0.0065

![alt text](./images/image-kw.png)

Zapytanie nr2:
czas: 1/100 sekundy
cost: ~0.05

![alt text](./images/image-1-kw.png)

Jak widać zamiast full table scan, mamy index scan a następnie RID lookup

O ile czas wykonania znacząco się nie różni, to możemy zauważyć znaczące mniejszenie kosztu wykonania.


Dodaj indeks klastrowany:

```sql
create clustered index clustered_customer_store_cls_idx on customer(storeid)
```

Czy zmienił się plan i czas? Skomentuj dwa podejścia w wyszukiwaniu krotek.

Zapytanie nr1:
czas: 1/100 sekundy
kosz: ~0.003

![alt text](./images/image-2-kw.png)

Zapytanie nr2:
czas: 1/100 sekundy
koszt: ~0.003

![alt text](./images/image-3-kw.png)

W planie mamy tylko clustered index seek. W przypadku zapytania nr1 nie widzimy dużej korzyści z zastosowania clustered index. Rożnice widać za to w przypadku wybieraniu zakresu. Ponieważ clustered index fizycznie sortuje dane na dysku, kolejne wiersze z zakresu są umieszczone blisko siebie.

# Zadanie 2 – Indeksy zawierające dodatkowe atrybuty (dane z kolumn)

Celem zadania jest poznanie indeksów z przechowujących dodatkowe atrybuty (dane z kolumn)

Skopiuj tabelę `Person` do swojej bazy danych:

```sql
select businessentityid  
      ,persontype  
      ,namestyle  
      ,title  
      ,firstname  
      ,middlename  
      ,lastname  
      ,suffix  
      ,emailpromotion  
      ,rowguid  
      ,modifieddate  
into person  
from adventureworks2017.person.person
```
---

Wykonaj analizę planu dla trzech zapytań:

```sql
select * from [person] where lastname = 'Agbonile'  
  
select * from [person] where lastname = 'Agbonile' and firstname = 'Osarumwense'  
  
select * from [person] where firstname = 'Osarumwense'
```

Co można o nich powiedzieć?


---
> Wyniki: 

![alt text](./images/zad2-zap1-noindex.png)
![alt text](./images/zad2-zap2-noindex.png)
![alt text](./images/zad2-zap3-noindex.png)

Dla query szukającym po imieniu zwrócone zostało 5 wierszy.
W każdym z przypadków system bazy danych jest zmuszony do przeskanowania całej tabeli w celu znalezienia wierszy spełniających kryterium.


Przygotuj indeks obejmujący te zapytania:

```sql
create index person_first_last_name_idx  
on person(lastname, firstname)
```

Sprawdź plan zapytania. Co się zmieniło?


---
> Wyniki: 

![alt text](./images/zad2-zap1-index.png)
![alt text](./images/zad2-zap2-index.png)
![alt text](./images/zad2-zap3-index.png)

Jak widać teraz najpierw używany jest indeks a następnie RID lookup.
RID lookup jest wykonywany w celu uzyskania dodatkowych informacji o wierszu, które nie są przechowywane w indeksie.


Przeprowadź ponownie analizę zapytań tym razem dla parametrów: `FirstName = ‘Angela’` `LastName = ‘Price’`. (Trzy zapytania, różna kombinacja parametrów). 

Czym różni się ten plan od zapytania o `'Osarumwense Agbonile'` . Dlaczego tak jest?


---
> Wyniki: 

```sql
select * from [person] where lastname = 'Price';
select * from [person] where lastname = 'Price' and firstname = 'Angela'  ;
select * from [person] where firstname = 'Angela';
```

![alt text](./images/zad2-zap4-index.png)
![alt text](./images/zad2-zap5-index.png)
![alt text](./images/zad2-zap6-index.png)

Dla query szukającym po imieniu zwrócone zostało 86 wierszy.

Do realizacji zapytań używających tylko jednego pola w klauzuli `WHERE` indeks nie jest wykorzystywany.
Prawdopodobnie optymalizator mając do dyspozycji statystyki tabeli i indeksu stwierdził, że znalezienie wszystkich wierszy spełniających klauzulę `WHERE` użycie dostępnego indeksu nie jest opłacalne. System baz danych musi utrzymywać statystyki rozkładu danych z których wynika, że `Angele` jest dużo popularniejszym imieniem od `Osarumwense`.


# Zadanie 3

Skopiuj tabelę `PurchaseOrderDetail` do swojej bazy danych:

```sql
select * into purchaseorderdetail from  adventureworks2017.purchasing.purchaseorderdetail
```

Wykonaj analizę zapytania:

```sql
select rejectedqty, ((rejectedqty/orderqty)*100) as rejectionrate, productid, duedate  
from purchaseorderdetail  
order by rejectedqty desc, productid asc
```

Która część zapytania ma największy koszt?

---
> Wyniki: 

![alt text](./images/zad3_1.png)

![alt text](./images/zad3_2.png)

![alt text](./images/zad3_3.png)

Największy koszt ma część zapytania obejmująca sortowanie. Stanowi ona 85% kosztu całości zapytania.

Jaki indeks można zastosować aby zoptymalizować koszt zapytania? Przygotuj polecenie tworzące index.


---
> Wyniki: 

Można dodać indeks na kolumny `rejectedqty` oraz `productid` wskazując kolejność sortowania tak jak w zapytaniu wraz z sekcją `include`, w której uwzględnimy kolumny, których dodatkowo potrzebujemy, czyli `orderqty` oraz `duedate`, aby nie trzeba było realizować dodatkowych operacji pobierających te wartości.

```sql
create index person_firspurchaseorderdetail_rejectedqty_productid_orderqty_duedate
on purchaseorderdetail (rejectedqty desc, productid asc) include (orderqty, duedate)
```

Ponownie wykonaj analizę zapytania:

---
> Wyniki: 

![alt text](./images/zad3_4.png)
![alt text](./images/zad3_5.png)

Zniknęła najbardziej kosztowna część zapytania obejmująca sortowanie dzięki zastosowaniu indeksu na kolumny po których odbywało się sortowanie, gdyż nie jest ono już potrzebne, bo jest zapownione przez realizację indeksu. Teraz najbardziej kosztowną operacją jest odczyt wartości.
Zmniejszył się także koszt operacji wejścia/wyjścia.
Zmalał całkowity koszt i czas zapytania.

# Zadanie 4

Celem zadania jest porównanie indeksów zawierających wszystkie kolumny oraz indeksów przechowujących dodatkowe dane (dane z kolumn).

Skopiuj tabelę `Address` do swojej bazy danych:

```sql
select * into address from  adventureworks2017.person.address
```

W tej części będziemy analizować następujące zapytanie:

```sql
select addressline1, addressline2, city, stateprovinceid, postalcode  
from address  
where postalcode between n'98000' and n'99999'
```

```sql
create index address_postalcode_1  
on address (postalcode)  
include (addressline1, addressline2, city, stateprovinceid);  
go  
  
create index address_postalcode_2  
on address (postalcode, addressline1, addressline2, city, stateprovinceid);  
go
```


Czy jest widoczna różnica w zapytaniach? Jeśli tak to jaka? Aby wymusić użycie indeksu użyj `WITH(INDEX(Address_PostalCode_1))` po `FROM`:

> Wyniki: 
### WYNIKI START

Wynik zapytania:

![alt text](./images/4-kw-image.png)

Plan:

![alt text](./images/4-kw-image-1.png)

Koszt: ~0.27

Wykonujemy pełny skan tabeli

### Zastosowanie indexu 1
```sql
select addressline1, addressline2, city, stateprovinceid, postalcode  
from address 
WITH(INDEX(Address_PostalCode_1))
where postalcode between '98000' and '99999'
```

Wynik zapytania:

![alt text](./images/4-kw-image-2.png)

Plan:

![alt text](./images/4-kw-image-3.png)

Koszt: ~0.028

### Zastosowanie indexu 2
```sql
select addressline1, addressline2, city, stateprovinceid, postalcode  
from address 
WITH(INDEX(Address_PostalCode_2))
where postalcode between '98000' and '99999'
```

Wynik zapytania: 

![alt text](./images/4-kw-image-4.png)

Plan:

![alt text](./images/4-kw-image-5.png)

Koszt: ~0.028

Nie widać różnicy pomiędzy poszczególnymi zapytaniami, koszt i plan jest taki sam dla obu indeksów.

Różnica w indeksach polega na na sposbie przetrzymywania wartości w b-drzewie. Klucze w include() są przechowywane w liściach b-drzewa, w drugim przypadku wszystkie wartości są przechoywane w każdym z nodeów.


### WYNIKI END
---

Sprawdź rozmiar Indeksów:

```sql
select i.name as indexname, sum(s.used_page_count) * 8 as indexsizekb  
from sys.dm_db_partition_stats as s  
inner join sys.indexes as i on s.object_id = i.object_id and s.index_id = i.index_id  
where i.name = 'address_postalcode_1' or i.name = 'address_postalcode_2'  
group by i.name  
go
```


Który jest większy? Jak można skomentować te dwa podejścia do indeksowania? Które kolumny na to wpływają?

### WYNIKI START
> Wyniki: 

![alt text](./images/4-kw-image-6.png)

Jak widać rozmiar indeksów jest bardzo podobny. Może za to wystąpić różnica w ilości poziomów drzewa. Ponieważ, w nie-liściach nie przechowujemy kolumn zawartych w include(), to jesteśmy w stanie zmieścić więcej kluczy w jednym nodzie. Skutkuje to bardziej płaskim drzewem ale o podobnym fizycznym rozmiarze.

### WYNIKI END

# Zadanie 5 – Indeksy z filtrami

Celem zadania jest poznanie indeksów z filtrami.

Skopiuj tabelę `BillOfMaterials` do swojej bazy danych:

```sql
select * into billofmaterials  
from adventureworks2017.production.billofmaterials
```


W tej części analizujemy zapytanie:

```sql
select productassemblyid, componentid, startdate  
from billofmaterials  
where enddate is not null  
    and componentid = 327  
    and startdate >= '2010-08-05'
```

![alt text](./images/zad5_1.png)
![alt text](./images/zad5_2.png)

Zastosuj indeks:

```sql
create nonclustered index billofmaterials_cond_idx  
    on billofmaterials (componentid, startdate)  
    where enddate is not null
```

Sprawdź czy działa. 

Przeanalizuj plan dla poniższego zapytania:

Czy indeks został użyty? Dlaczego?

> Wyniki: 

![alt text](./images/zad5_3.png)

Jak widać wynik analizy zapytania po utworzniu indeksu jest taki sam jak przed jego utworzeniem, a więc indekse nie został użyty.

Spróbuj wymusić indeks. Co się stało, dlaczego takie zachowanie?

> Wyniki: 

##### Wymuszenie indeksu

```sql
select productassemblyid, componentid, startdate  
from billofmaterials with(index(billofmaterials_cond_idx))  
where enddate is not null  
    and componentid = 327  
    and startdate >= '2010-08-05'
```

![alt text](./images/zad5_4.png)

##### Porównanie

Koszt zapytania przed wymuszeniem indeksu:

![alt text](./images/zad5_5.png)

Koszt zapytania po wymuszeniu indeksu:

![alt text](./images/zad5_6.png)

Po wymuszeniu indeksu, widzimy, że został on wykorzystany, jednak koszt zapytania wzrósł z 0.02 do 0.07.

Bez wymuszania indeksu planer decyduje się na jego nie wykorzystanie, prawdopodobnie dlatego, że zapytanie korzystające z tego indeksu jest mniej wydajne.

---

Punktacja:

|         |     |
| ------- | --- |
| zadanie | pkt |
| 1       | 2   |
| 2       | 2   |
| 3       | 2   |
| 4       | 2   |
| 5       | 2   |
| razem   | 10  |
