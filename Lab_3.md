# Bazy Danych / Lab_3 Zadania

Author: @http://github.com/hbery 

##### version: <span style="color: orange;">draft-0.2</span>

---

#### **Zadanie 1.** Wyświetlić identyfikator kategorii i sumę czasów trwania filmów należących do poszczególnych kategorii.

|cid | sum |
|:--:|:---:|
60 | 11882469
61 | 56491621
63 | 804406
64 | 3309812
| |5436129
(5 rows)

```sql
select cid, sum(lenmsec) from movies_list group by cid;
```

---

#### **Zadanie 2.** Wyświetlić identyfikator kategorii i sumę czasów trwania filmów należących do poszczególnych kategorii. Do obliczania sumy brać tylko filmy, których wartość parametru stream > 1 500 000.

|cid | sum |
|:--:|:---:|
60 | 155532
61 | 5709429
63 | 772848
64 | 2283686
| |5436129
(5 rows)

```sql
select cid, sum(lenmsec) from movies_list where stream>1500000 group by cid;
```

---

#### **Zadanie 3.** Wyświetlić identyfikator kategorii i sumę czasów trwania filmów należących do poszczególnych kategorii. Do obliczania sumy brać tylko filmy, których wartość parametru stream > 1 500 000. Wyświetlić tylko kategorie, dla których suma czasów trwania filmu jest większa od 900 000 ms.

| cid | sum |
|:---:|:---:|
61 | 5709429
64 | 2283686
| |5436129
(3 rows)

```sql
select cid, sum(lenmsec) from movies_list where stream>1500000 group by cid having sum(lenmsec)>900000;
```

---

#### **Zadanie 4.** Policz dla ilu epizodów (nie filmów) wprowadzono opis jako tekst (descrtype=1).

|liczba opisow|
|:-----------:|
7
(1 row)

```sql
select count(*) as "liczba opisow" from episodes_list where descrtype=1 and is_movie=0;
```

---

#### **Zadanie 5.** Wyświetl nazwy kategorii, które mają przynajmniej 2 subkategorie. Jako drugą kolumnę wyświetl ile subkategorii należy do danej kategorii.

| Nazwa kategorii | count |
|:----------------|:-----:|
KT | 3
Education | 6
Amusement | 3
(3 rows)

```sql
select (select categories.name as "Nazwa kategorii" from categories where categories.cid=subcategories.cid), count(subcategories.cid) from subcategories group by subcategories.cid having count(subcategories.cid)>=2;
```

---

#### **Zadanie 6.** Wypisz nazwy subkategorii, dla których maksymalny czas trwania epizodów należących do danej subkategorii jest dłuższy od 10 sekund. Dodatkowo wypisz czas trwania najdłuższego epizodu i liczbę epizodów spełniających warunek dla danej subkategorii. 

|Subkategoria | max | count|
|:-----------|:---:|:-----:|
Biology | 318690 | 5
Geography | 660330 | 11 
Cartoons | 11480 | 1
(3 rows)

```sql
select (select sub.name as "Subkategoria" from subcategories sub where sub.sid=e.sid), max(e.episode_end-e.episode_start), count(e.sid) from episodes_list e group by e.sid having max(e.episode_end-e.episode_start)>10000;
```

---

#### **Zadanie 7.** Wypisz nazwy subkategorii, dla których maksymalna długość epizodu należącego do danej subkategorii jest mniejsza od 350 sekund. Jako drugą kolumnę wypisz długość najdłuższego epizodu, a jako trzecią kolumnę liczbę epizodów należących do danej subkategorii. Posortuj wyniki według nazwy subkategorii.

| Nazwa podkategorii | max | count |
|:-------------------|:---:|:-----:|
Biology | 318690 | 5
Cartoons | 11480 | 1
wykłady | 6970 | 1
(3 rows)

```sql
select (select sub.name as "Nazwa podkategorii" from subcategories sub where sub.sid=e.sid), max(e.episode_end-e.episode_start), count(e.sid) from episodes_list e group by e.sid having max(e.episode_end-e.episode_start)<350000;
```

---

#### **Zadanie 8.** Policz ile filmów ma różne słowa kluczowe.

| count |
|:-----:|
13|
(1 row)

```sql
select count(distinct title) from movies_list;
```

---

#### **Zadanie 9.** Policz ile filmów ma zdefiniowany odnośnik URL (parametr descrhtml).

| count |
|:-----:|
20|
(1 row)

```sql
select count(descrhtml) from movies_list;
```

---

#### **Zadanie 10.** Policz ile filmów nie ma zdefiniowanego odnośnika URL (parametr descrhtml).

| count |
|:-----:|
24|
(1 row)

```sql
select count(*) from movies_list where descrhtml is null;
```

---

#### **Zadanie 11.** Policz ile jest zdefiniowanych różnych odnośników URL dla filmów.

| count |
|:-----:|
10|
(1 row)

```sql
select count(distinct descrhtml) from movies_list;
```

---

#### **Zadanie 12.** Wypisz nazwy filmów i odnośniki URL posortowane według nazwy filmu. Jeśli odnośnik URL nie jest zdefiniowany powinien pojawić się napis "URL not defined" (użyj COALESCE). 

| title | url |
|:---------------------------------------------------------|:------------------------------------
Bolek & Lolek - Australian steppes | ../html_descriptions/bolek_lolek
Bolek & Lolek - Gold town | ../html_descriptions/bolek_lolek
Bolek & Lolek - On tiger's trails | ../html_descriptions/bolek_lolek
Bolek & Lolek - Pharaoh's tomb | ../html_descriptions/bolek_lolek
Bolek & Lolek - Race to the North Pole | ../html_descriptions/bolek_lolek
Bolek & Lolek - Smuggler | ../html_descriptions/bolek_lolek
Course in a box | URL not defined
Fore Systems (advertisement) | URL not defined
Gustavus and the fly | URL not defined ...
...|
(44 rows)

```sql
select title, coalesce(descrhtml, 'URL not defined') as "url" from movies_list order by title;
```

---