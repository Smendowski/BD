# Bazy Danych / Lab_3 Zadania

Author: @https://github.com/piotrsladowski

---

#### **Zadanie 1.** Wyświetlić nazwy podkategorii należących do kategorii 'Education' i średnie czasy trwania filmów należących do poszczególnych podkategorii.

|name|avg|
|:--:|:---:|
Biology | 4730854.142857142857
Physics |
Technology | 1027413.5000000000000
Telecommunications | 3715850.000000000000
Mathematics |
Geography | 2741565.000000000000
(6 rows)

```sql
select s.name, (select avg(lenmsec) from movies_list m where m.sid = s.sid) from subcategories s where s.cid=61;
```

---

#### **Zadanie 2.** Wyświetlić tytuły wszystkich filmów nie należących do kategorii 'Education' (stosując podzapytania), posortowane rosnąco:

|title|
|:--:|
Bolek & Lolek - Australian steppes|
Bolek & Lolek - Gold town|
Bolek & Lolek - On tiger's trails|
Bolek & Lolek - Pharaoh's tomb|
Bolek & Lolek - Race to the North Pole|
Bolek & Lolek - Smuggler|
Fore Systems (advertisement)|
Gustavus and the fly|
Gustavus is a muff|
Gustavus is cheating|
Gustavus participates in traffic|
Gustavus wants to marry|
Infomercial|
Intranet (advertisement)|
Lucent 1|
Lucent 2|
Matematyczny model transmisji pakietów w kanale (M/M/1)|
Nowy budynek KT|
Nowy budynek KT|
OVS Introduction|
OVS MPEG1 1536k|
OVS MPEG1 2048k|
OVS MPEG1 2048k PAL|
OVS Sample 1|
OVS Sample 2|
Prezentacja Katedry Telekomunikacji AGH|
The Big Blue|
Wystąpienie rektora AGH|
(28 rows)

```sql
select e.title from episodes_list e where (select cid from movies_list m where m.mid=e.mid)<>61 order by title;
```

---

#### **Zadanie 3.** Wyświetlić nazwy kategorii, w których minimalna długość filmu w bajtach jest większa niż 2000000.

|name|
|:--:|
Amusement|
Education|
KT|
(3 rows)

```sql
select name from categories where (select min(lenbin) from movies_list where movies_list.cid=categories.cid)>2000000;
```

---

#### **Zadanie 4.** Podać statystykę liczby epizodów w filmach. Poniższe wyniki oznaczają, że w bazie jest 40 filmów, dla których nie zdefiniowano epizodów, 2 filmy z jednym epizodem, jeden film z pięcioma epizodami itd.


|no of episodes | no of films|
|:--:|:--:|
 11 | 1
 5 | 1
 1 | 2
 0 | 40
(4 rows)

```sql

```


---


#### **Zadanie 5.** Wypisać ID, nazwę i długość filmów, które są dłuższe od średniej z długości pozostałych filmów. Jako ostatnią kolumnę wyświetlić średnią z długości pozostałych filmów: 

|mid | title | lenmsec | avg lenmsec excl. given mid|
|:--:|:--:|:--:|:--:|
 80 | Image coding (part 2) | 5436129 | 1685774.604651162791 |
122 | Microcosmos (medium quality) | 4534809 | 1706735.534883720930 |
101 | Video coding | 5385605 | 1686949.581395348837 |
 98 | Image coding (part 1) | 5501871 | 1684245.720930232558 |
100 | Image coding (part 4) | 6965989 | 1650196.465116279070 |
119 | Overhead telecommunications lines | 1897170 | 1768075.976744186047 |
124 | Saga of life (medium quality) | 6363459 | 1664208.790697674419 |
111 | Saga of life (high quality) | 6422197 | 1662842.790697674419 |
110 | Southern Africa Safari | 3571616 | 1729135.372093023256 |
103 | Microcosmos (high quality) | 4379470 | 1710348.069767441860 |
121 | Microcosmos (low quality) | 4554451 | 1706278.744186046512 |
123 | Saga of life (low quality) | 3289977 | 1735685.116279069767 |
112 | The World's Deadliest Volcanoes | 2741565 | 1748438.883720930233 |
113 | The Big Blue | 6876310 | 1652282.023255813953 |
(14 rows)

```sql
select m.mid, (select e.title from episodes_list e where e.mid=m.mid limit 1), m.lenmsec, select avg(lenmsec) from movies_list where mid<>m.mid) from movies_list m where lenmsec > (select avg(lenmsec) from movies_list);
```


---

#### **Zadanie 6.** Wypisz nazwy subkategorii, dla których sumaryczny czas trwania filmów należących do danej subkategorii jest dłuższy od 1000 sekund. Dodatkowo wypisz czas trwania najdłuższego filmu i liczbę filmów dla danej subkategorii. Użyj podzapytań.

|name | max | count|
|:--:|:--:|:--:|
budowa | 846550 | 2 |
Cartoons | 607362 | 11 |
wykłady | 757567 | 3 |
Movies | 6876310 | 1 |
Biology | 6422197 | 7 |
Technology | 1897170 | 2 |
Geography | 2741565 | 1 |
Telecommunications | 6965989 | 5 |
(8 rows)

```sql
select s.name, (select max(lenmsec) from movies_list where sid=s.sid), (select count(*) from movies_list where sid=s.sid) from subcategories s where (select sum(lenmsec) from movies_list m where m.sid = s.sid)>1000000;
```

---


#### **Zadanie 7.** Wypisz nazwy subkategorii, dla których średni czas trwania epizodów jest dłuższy od 15 sekund. Dodatkowo wypisz średni czas trwania epizodów i liczbę epizodów należących do danej subkategorii. Użyj podzapytań. W zapytaniu głównym nie używaj iloczynu kartezjańskiego.

| name | avg | count |
|:--:|:--:|:--:|
Biology | 142582.000000000000 | 5 |
Geography | 194136.363636363636 | 11 |
(2 rows)


---

#### **Zadanie 8.** Wykonaj zadanie 7 bez używania iloczynu kartezjańskiego w zapytaniu głównym i w którymkolwiek z podzapytań. 

| name | avg | count |
|:--:|:--:|:--:|
Biology | 142582.000000000000 | 5 |
Geography | 194136.363636363636 | 11 |
(2 rows)


---


#### **Zadanie 9.** Utworzyć widok o nazwie srednie_v (polecenie CREATE VIEW) zawierający nazwy podkategorii należących do kategorii „Education” oraz średnie czasy trwania filmów w poszczególnych podkategoriach. Wyświetlić dane z widoku srednie_v. 

| name | avg |
|:--:|:--:|
Biology | 4730854.142857142857 |
Physics |
Technology | 1027413.5000000000000 |
Telecommunications | 3715850.000000000000 |
Mathematics |
Geography | 2741565.000000000000 |
(6 rows)

---


#### **Zadanie 10.** Utworzyć nową tabelę o nazwie srednie_t (polecenie CREATE TABLE) zawierającą nazwy podkategorii należących do kategorii „Education” oraz średnie czasy trwania filmów w poszczególnych podkategoriach. <br> Wyświetlić dane z tabeli srednie_t.

| name | avg |
|:--:|:--:|
Biology | 4730854.142857142857 |
Physics |
Technology | 1027413.5000000000000 |
Telecommunications | 3715850.000000000000 |
Mathematics |
Geography | 2741565.000000000000 |
(6 rows)

---


#### **Zadanie 11.**  Jeden z filmów należących do podkategorii „Biology” ma czas trwania równy 6363459. Zmienić czas trwania tego filmu na 1 000 000.<br> Następnie wyświetlić zawartość widoku srednie_v:

| name | avg |
|:--:|:--:|
Biology | 3964645.714285714286 |
Physics |
Technology | 1027413.5000000000000 |
Telecommunications | 3715850.000000000000 |
Mathematics |
Geography | 2741565.000000000000 |
(6 rows)

oraz tabeli srednie_t:

| name | avg |
|:--:|:--:|
Biology | 4730854.142857142857 |
Physics |
Technology | 1027413.5000000000000 |
Telecommunications | 3715850.000000000000 |
Mathematics |
Geography | 2741565.000000000000 |
(6 rows)

Dlaczego średnia długość filmów należących do kategorii „Biology” zmieniła się w przypadku widoku
srednie_v a nie zmieniła w tabeli srednie_t ?