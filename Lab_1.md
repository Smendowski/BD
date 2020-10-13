# Bazy Danych / Lab_1 Zadania

Author @http://github.com/hbery

version: final-1.0

---

### Setup

```sql
create table studenci (imie varchar(20), nazwisko varchar(20), indeks int4);
\d
\d studenci
insert into studenci values ('Adam', 'Malysz', 1);
insert into studenci values ('James', 'Bond', 2);
insert into studenci values ('Adam', 'Mickiewicz', 3);
```

#### Zadanie 1: Wyświetlić nazwisko i nr. indeksu.

```sql
select nazwisko, indeks from studenci;
```

---

#### Zadanie 2: Wyświetlić nazwisko i imię tych studentów, których indeks=2

```sql
select nazwisko, indeks from studenci where indeks=2;
```

---

#### Zadanie 3: Dopisać nr. tel. dla podanych osób, aby jego długość wynosiła 6-7 znaków.

```sql
alter table studenci add column tel varchar(20);
update studenci set tel=floor(random()*10000000);
```

---

#### Zadanie 4: Dopisać kolumnę „wiek” typ: int4 i uzupełnić tak. aby wiek różnych osób był powyżej i poniżej 25 lat.

```sql
alter table studenci add column wiek int4;
update studenci set wiek=floor(random()*(35-15+1)+15);
```

---

#### Zadanie 5: Wyświetlić osoby starsze niż 25 lat.

```sql
select * from studenci where wiek>25;
```

---

#### Zadanie 6: Wyświetlić nazwiska i imiona posortowane wg. nazwiska.

```sql
select nazwisko, imie from studenci order by nazwisko;
```

---

#### Zadanie 7: Wyświetlić nazwiska i imiona osób posortowane wg. wieku.

```sql
select nazwisko, imie order by wiek;
```

---

#### Zadanie 8: Wykonaj kopię zapasową bazy danych. Następnie usuń bazę danych. Utwórz ją na nowo i załaduj zawartość bazy danych z kopii zapasowej. Sprawdź zawartość odzyskanej bazy danych.

```bash
pg_dump database_name > file_name
dropdb database name
createdb
psql -f file_name
```

---

#### Zadanie 9: Skasować jednym poleceniem wszystkie wiersze w tablicy studenci.

```sql
drop table studenci
```

---
