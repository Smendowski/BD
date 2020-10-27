# Bazy Danych / Lab_3 Zadania

Author: @http://github.com/hbery 

##### version: final-1.0

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

ONELINER:
```sql
select cid, sum(lenmsec) from movies_list group by cid order by cid;
```

```sql
select
    cid,
    sum(lenmsec)
from
    movies_list
group by
    cid
order by
    cid;
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

ONELINER:
```sql
select cid, sum(lenmsec) from movies_list where stream>1500000 group by cid order by cid;
```

```sql
select
    cid,
    sum(lenmsec)
from
    movies_list
where
    stream>1500000
group by
    cid
order by
    cid;
```

---

#### **Zadanie 3.** Wyświetlić identyfikator kategorii i sumę czasów trwania filmów należących do poszczególnych kategorii. Do obliczania sumy brać tylko filmy, których wartość parametru stream > 1 500 000. Wyświetlić tylko kategorie, dla których suma czasów trwania filmu jest większa od 900 000 ms.

| cid | sum |
|:---:|:---:|
61 | 5709429
64 | 2283686
| |5436129
(3 rows)

ONELINER:
```sql
select cid, sum(lenmsec) from movies_list where stream>1500000 group by cid having sum(lenmsec)>900000 order by cid;
```

```sql
select
    cid,
    sum(lenmsec)
from
    movies_list
where
    stream>1500000
group by
    cid
having
    sum(lenmsec)>900000
order by
    cid;
```

---

#### **Zadanie 4.** Policz dla ilu epizodów (nie filmów) wprowadzono opis jako tekst (descrtype=1).

|liczba opisow|
|:-----------:|
7
(1 row)

ONELINER:
```sql
select count(*) as "liczba opisow" from episodes_list where descrtype=1 and is_movie=0;
```

```sql
select
    count(*) as "liczba opisow"
from
    episodes_list
where
    descrtype=1
    and
    is_movie=0;
```

---

#### **Zadanie 5.** Wyświetl nazwy kategorii, które mają przynajmniej 2 subkategorie. Jako drugą kolumnę wyświetl ile subkategorii należy do danej kategorii.

| Nazwa kategorii | count |
|:----------------|:-----:|
KT | 3
Education | 6
Amusement | 3
(3 rows)

ONELINER:
```sql
select c.name as "Nazwa kategorii", count(s.cid) from subcategories as s, categories as c where c.cid=s.cid group by c.name having count(s.cid)>=2 order by c.name desc;
```

```sql
select
    c.name as "Nazwa kategorii",
    count(s.cid)
from
    subcategories as s,
    categories as c
where
    c.cid=s.cid
group by
    c.name
having
    count(s.cid)>=2
order by
    c.name desc;
```

---

#### **Zadanie 6.** Wypisz nazwy subkategorii, dla których maksymalny czas trwania epizodów należących do danej subkategorii jest dłuższy od 10 sekund. Dodatkowo wypisz czas trwania najdłuższego epizodu i liczbę epizodów spełniających warunek dla danej subkategorii.

|Subkategoria | max | count|
|:-----------|:---:|:-----:|
Biology | 318690 | 5
Geography | 660330 | 11
Cartoons | 11480 | 1
(3 rows)

ONELINER:
```sql
select s.name as "Subkategoria", max(e.episode_end-e.episode_start), count(s.name) from episodes_list as e, movies_list as m, subcategories as s where m.mid=e.mid and s.sid=m.sid and (e.episode_end-e.episode_start)>10000 and e.is_movie=0 group by s.name having max(e.episode_end-e.episode_start)>10000;
```

```sql
select
    s.name as "Subkategoria",
    max(e.episode_end-e.episode_start),
    count(s.name)
from
    episodes_list as e,
    movies_list as m,
    subcategories as s
where
    m.mid=e.mid
    and
    s.sid=m.sid
    and
    (e.episode_end-e.episode_start)>10000
    and
    e.is_movie=0
group by
    s.name
having
    max(e.episode_end-e.episode_start)>10000;
```

---

#### **Zadanie 7.** Wypisz nazwy subkategorii, dla których maksymalna długość epizodu należącego do danej subkategorii jest mniejsza od 350 sekund. Jako drugą kolumnę wypisz długość najdłuższego epizodu, a jako trzecią kolumnę liczbę epizodów należących do danej subkategorii. Posortuj wyniki według nazwy subkategorii.

| Nazwa podkategorii | max | count |
|:-------------------|:---:|:-----:|
Biology | 318690 | 5
Cartoons | 11480 | 1
wykłady | 6970 | 1
(3 rows)

ONELINER:
```sql
select s.name as "Nazwa podkategorii", max(e.episode_end-e.episode_start), count(s.name) from episodes_list as e, movies_list as m, subcategories as s where m.mid=e.mid and s.sid=m.sid and e.is_movie=0 group by s.name having max(e.episode_end-e.episode_start)<350000 order by s.name;
```

```sql
select
    s.name as "Nazwa podkategorii",
    max(e.episode_end-e.episode_start),
    count(s.name)
from
    episodes_list as e,
    movies_list as m,
    subcategories as s
where
    m.mid=e.mid
    and
    s.sid=m.sid
    and
    e.is_movie=0
group by
    s.name
having
    max(e.episode_end-e.episode_start)<350000
order by
    s.name;
```

---

#### **Zadanie 8.** Policz ile filmów ma różne słowa kluczowe.

| count |
|:-----:|
13|
(1 row)

ONELINER:
```sql
select count(distinct keywords) from episodes_list where is_movie=1;
```

```sql
select
    count(distinct keywords)
from
    episodes_list
where
    is_movie=1;
```

---

#### **Zadanie 9.** Policz ile filmów ma zdefiniowany odnośnik URL (parametr descrhtml).

| count |
|:-----:|
20|
(1 row)

ONELINER:
```sql
select count(descrhtml) from episodes_list where is_movie=1;
```

```sql
select
    count(descrhtml)
from
    episodes_list
where
    is_movie=1;
```

---

#### **Zadanie 10.** Policz ile filmów nie ma zdefiniowanego odnośnika URL (parametr descrhtml).

| count |
|:-----:|
24|
(1 row)

ONELINER:
```sql
select count(*) from episodes_list where descrhtml is null and is_movie=1;
```

```sql
select
    count(*)
from
    episodes_list
where
    descrhtml is null
    and
    is_movie=1;
```

---

#### **Zadanie 11.** Policz ile jest zdefiniowanych różnych odnośników URL dla filmów.

| count |
|:-----:|
10|
(1 row)

ONELINER:
```sql
select count(distinct descrhtml) from episodes_list where is_movie=1;
```

```sql
select
    count(distinct descrhtml)
from
    episodes_list
where
    is_movie=1;
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

ONELINER:
```sql
select title, coalesce(descrhtml, 'URL not defined') as "url" from episodes_list where is_movie=1 order by title;
```

```sql
select
    title,
    coalesce(descrhtml, 'URL not defined') as "url"
from
    episodes_list
where
    is_movie=1
order by
    title;
```

---
