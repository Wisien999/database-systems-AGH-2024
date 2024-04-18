
# Indeksy,  optymalizator <br>Lab 6-7

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


    
Stwórz swoją bazę danych o nazwie lab6. 

```sql
create database lab5  
go  
  
use lab5  
go
```

## Dokumentacja

Obowiązkowo:
- [https://docs.microsoft.com/en-us/sql/relational-databases/indexes/indexes](https://docs.microsoft.com/en-us/sql/relational-databases/indexes/indexes)
- [https://docs.microsoft.com/en-us/sql/relational-databases/indexes/create-filtered-indexes](https://docs.microsoft.com/en-us/sql/relational-databases/indexes/create-filtered-indexes)

# Zadanie 1

Skopiuj tabelę Product do swojej bazy danych:

```sql
select * into product from adventureworks2017.production.product
```

Stwórz indeks z warunkiem przedziałowym:

```sql
create nonclustered index product_range_idx  
    on product (productsubcategoryid, listprice) include (name)  
where productsubcategoryid >= 27 and productsubcategoryid <= 36
```

Sprawdź, czy indeks jest użyty w zapytaniu:

```sql
select name, productsubcategoryid, listprice  
from product  
where productsubcategoryid >= 27 and productsubcategoryid <= 36
```

Sprawdź, czy indeks jest użyty w zapytaniu, który jest dopełnieniem zbioru:

```sql
select name, productsubcategoryid, listprice  
from product  
where productsubcategoryid < 27 or productsubcategoryid > 36
```


Skomentuj oba zapytania. Czy indeks został użyty w którymś zapytaniu, dlaczego? Czy indeks nie został użyty w którymś zapytaniu, dlaczego? Jak działają indeksy z warunkiem?


---
> Wyniki: 

Execution Plan dla zapytania 1:

![alt text](_img/zad1-zap1.png)

Execution Plan dla zapytania 2:

![alt text](_img/zad1-zap2.png)


Tylko pierwsze zapytanie wykorzystuje indeks. W drugim zapytaniu warunek jest przeciwieństwem warunku indeksu, dlatego indeks nie jest używany. Indeks z warunkiem działa tylko wtedy, gdy warunek jest spełniony. W przeciwnym przypadku indeks nie jest używany.



# Zadanie 2 – indeksy klastrujące

Celem zadania jest poznanie indeksów klastrujących![](file:////Users/rm/Library/Group%20Containers/UBF8T346G9.Office/TemporaryItems/msohtmlclip/clip_image001.jpg)

Skopiuj ponownie tabelę SalesOrderHeader do swojej bazy danych:

```sql
select * into salesorderheader2 from adventureworks2017.sales.salesorderheader
```
![alt text](./_img/zad2_1.png)

Wypisz sto pierwszych zamówień:

```sql
select top 100 * from salesorderheader2  
order by orderdate
```

![alt text](./_img/zad2_2.png)
![alt text](./_img/zad2_3.png)
![alt text](./_img/zad2_analysis-1.png)

Stwórz indeks klastrujący według OrderDate:

```sql
create clustered index order_date2_idx on salesorderheader2(orderdate)
```

![alt text](./_img/zad2_4.png)

Wypisz ponownie sto pierwszych zamówień. Co się zmieniło?

![alt text](./_img/zad2_5.png)
![alt text](./_img/zad2_6.png)
![alt text](./_img/zad2_analysis-2.png)

---
> Wyniki: 

W wynikach nie zmieniło się nic. 

W analizie zapytań, możemy zauważyć, że w pierwszym przypadku, gdzie nie mamy indeksu jest realizowana operacja sortowania, która jest bardzo kosztowa i stanowi większą część kosztu zapytania. W drugim przypadku, dzięki zastosowaniu indeksu klastrującego na kolumnę, po której sortujemy w naszym zapytaniu pozbywamy się konieczności sortowania, czyli najbardziej kosztownej operacji, a więc koszt zapytania się zmniejsza.

Sprawdź zapytanie:

```sql
select top 1000 * from salesorderheader2  
where orderdate between '2010-10-01' and '2011-06-01'
```


Dodaj sortowanie według OrderDate ASC i DESC. Czy indeks działa w obu przypadkach. Czy wykonywane jest dodatkowo sortowanie?


---
> Wyniki: 

```sql
select top 1000 * from salesorderheader2  
where orderdate between '2010-10-01' and '2011-06-01'
order by OrderDate asc
```
![alt text](./_img/zad2_7.png)
![alt text](./_img/zad2_8.png)

Widzimy, że indeks został wykorzystany, na co wskazuje operacja `Clustered Index Seek`. Nie jest wykonywane dodatkowe sortowanie.

```sql
select top 1000 * from salesorderheader2  
where orderdate between '2010-10-01' and '2011-06-01'
order by OrderDate desc
```
![alt text](./_img/zad2_9.png)
![alt text](./_img/zad2_10.png)

Widzimy, że indeks został wykorzystany, na co wskazuje operacja `Clustered Index Seek`. Nie jest wykonywane dodatkowe sortowanie.

Indeks działa w obu przypadkach dzięki czemu w obu przypadkach unikamy konieczności sortowania.

# Zadanie 3 – indeksy column store


Celem zadania jest poznanie indeksów typu column store![](file:////Users/rm/Library/Group%20Containers/UBF8T346G9.Office/TemporaryItems/msohtmlclip/clip_image001.jpg)

Utwórz tabelę testową:

```sql
create table dbo.saleshistory(  
 salesorderid int not null,  
 salesorderdetailid int not null,  
 carriertrackingnumber nvarchar(25) null,  
 orderqty smallint not null,  
 productid int not null,  
 specialofferid int not null,  
 unitprice money not null,  
 unitpricediscount money not null,  
 linetotal numeric(38, 6) not null,  
 rowguid uniqueidentifier not null,  
 modifieddate datetime not null  
 )
```

Załóż indeks:

```sql
create clustered index saleshistory_idx  
on saleshistory(salesorderdetailid)
```



Wypełnij tablicę danymi:

(UWAGA    `GO 100` oznacza 100 krotne wykonanie polecenia. Jeżeli podejrzewasz, że Twój serwer może to zbyt przeciążyć, zacznij od GO 10, GO 20, GO 50 (w sumie już będzie 80))

```sql
insert into saleshistory  
 select sh.*  
 from adventureworks2017.sales.salesorderdetail sh  
go 100
```

Sprawdź jak zachowa się zapytanie, które używa obecny indeks:

```sql
select productid, sum(unitprice), avg(unitprice), sum(orderqty), avg(orderqty)  
from saleshistory  
group by productid  
order by productid
```

Załóż indeks typu ColumnStore:

```sql
create nonclustered columnstore index saleshistory_columnstore  
 on saleshistory(unitprice, orderqty, productid)
```

Sprawdź różnicę pomiędzy przetwarzaniem w zależności od indeksów. Porównaj plany i opisz różnicę.


---
> Wyniki: 

Zapytania zostały wykonane na tabeli z 52 mln wierszy

### Clustered index

![alt text](_img/image-3k.png)

![alt text](_img/image-1-3k.png)

![alt text](_img/image-2-3k.png)

![alt text](_img/image-6-3k.png)

Czas wykonania około: 8s

### Columnstore index

Zakładnie column store indexa trwało około: 41 sek

Czas wykonania: 0s 

![alt text](_img/image-3-3k.png)

![alt text](_img/image-5-3k.png)

![alt text](_img/image-4-3k.png)

![alt text](_img/image-7-3k.png)

### Porównanie

Jak widać columnstore mimo dość długiego procesu zakładania indeksu, drastycznie zwiększa prędkość odczytu. Estimated Subtree Cost zwykłego indeksu to 360, columnstora to 6. 

# Zadanie 4 – własne eksperymenty

Należy zaprojektować tabelę w bazie danych, lub wybrać dowolny schemat danych (poza używanymi na zajęciach), a następnie wypełnić ją danymi w taki sposób, aby zrealizować poszczególne punkty w analizie indeksów. Warto wygenerować sobie tabele o większym rozmiarze.

Do analizy, proszę uwzględnić następujące rodzaje indeksów:
- Klastrowane (np.  dla atrybutu nie będącego kluczem głównym)
- Nieklastrowane
- Indeksy wykorzystujące kilka atrybutów, indeksy include
- Filtered Index (Indeks warunkowy)
- Kolumnowe

## Analiza

Proszę przygotować zestaw zapytań do danych, które:
- wykorzystują poszczególne indeksy
- przy wymuszeniu indeksu działają gorzej, niż bez niego (lub pomimo założonego indeksu, tabela jest w pełni skanowana)

Odpowiedź powinna zawierać:

- Schemat tabeli
- Opis danych (ich rozmiar, zawartość, statystyki)
- Trzy indeksy
- Opis indeksu
- Przygotowane zapytania, wraz z wynikami z planów (zrzuty ekranow)
- Komentarze do zapytań, ich wyników
- Sprawdzenie, co proponuje Database Engine Tuning Advisor (porównanie czy udało się Państwu znaleźć odpowiednie indeksy do zapytania)


> Wyniki: 

```sql
--  ...
```




|         |     |     |     |
| ------- | --- | --- | --- |
| zadanie | pkt |     |     |
| 1       | 2   |     |     |
| 2       | 2   |     |     |
| 3       | 2   |     |     |
| 4       | 10  |     |     |
| razem   | 16  |     |     |
