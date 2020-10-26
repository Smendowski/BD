

SELECT title from episodes_list WHERE title like 'T%' AND title not like '%e';

# Bazy Danych / Lab_ Zadania

Author: @http://github.com/marcinswistak 

##### version: final-1.0

---

#### ** Zadanie 1: ** Wyświetlić wszystkie tytuły filmów i epizodów, które zaczynają się od litery 'T' i nie kończąznakiem 'e'.

title
---------------------------------
Technology LSA-PLUS
The World's Deadliest Volcanoes
The first morning
The ladybird
(4 rows)

```sql
SELECT title from episodes_list WHERE title like 'T%' AND title not like '%e';
```

---

#### **Zadanie 2: ** Wyświetlić wszystkie tytuły filmów i epizodów, które zaczynają się od litery 'T' i kończą znakiem 'e'. Użyć tylko jeden warunek.

title
--------------
The bee
The Big Blue
(2 rows)

```sql
SELECT title from episodes_list WHERE title like 'T%e';
```

---

#### **Zadanie 3.**  Na dwa sposoby wyświetlić tytuły filmów i epizodów które zawierają słowo 'The' lub 'the'

title
----------------------------------------
Gustavus and the fly
The World's Deadliest Volcanoes
Southern Africa Safari
Bolek & Lolek - Race to the North Pole
The first morning
The bee
The ladybird
The Big Blue
(8 rows)

```sql
SELECT title from episodes_list WHERE title like '%The%' OR title like '%the%';
```
```sql
SELECT title from episodes_list WHERE title ~ '.*[T][h][e].*' OR title ~'.*[t][h][e].*';
```
---

#### **Zadanie 4.**  Wyświetlić tytuły filmów i epizodów w których występuje cyfra ze zbioru: 0, 1, 2, 3, 5, 6.

title
---------------------------------------------------------
OVS Sample 1
OVS MPEG1 1536k
OVS MPEG1 2048k
OVS MPEG1 2048k PAL
OVS Sample 2
Image coding (part 1)
Image coding (part 2)
Matematyczny model transmisji pakietów w kanale (M/M/1)
Lucent 1
Lucent 2
(10 rows)

```sql
SELECT title from episodes_list WHERE title ~ '.*[012356].*';
```

---

#### **Zadanie 5.** Wyświetlić tytuły filmów i epizodów w których występują koło siebie cztery dowolne cyfry ze zbioru: 0, 1, 2, 3, 5, 6

title
-----------------
OVS MPEG1 1536k
(1 row)

```sql
SELECT title from episodes_list WHERE title ~ '.*[0-356][0-356][0-356][0-356].*';
```

---

#### **Zadanie 6.** Wyświetlić identyfikatory filmów dla których rozmiar pliku (lenbin) zawiera się w granicach 10 000 000 i 30 000 000. Zastosowć operator between.

mid
-----
 70
108
(2 rows)

```sql
SELECT mid from movies_list WHERE lenbin BETWEEN 10000000 AND 30000000;
```

---

#### **Zadanie 7.** Wyświetlić tytuły filmów z zadania 6. Posłużyć się podzapytaniem (nie krzyżować tabel) i operatorem in.

| Nazwa podkategorii | max | count |
|:-------------------|:---:|:-----:|
Biology | 318690 | 5
Cartoons | 11480 | 1
wykłady | 6970 | 1
(3 rows)

```sql
SELECT title from episodes_list WHERE mid IN (SELECT mid from movies_list WHERE lenbin BETWEEN 10000000 AND 30000000);
```

---
