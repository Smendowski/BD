# Bazy Danych / Lab_8 Zadania

Author: @https://github.com/piotrsladowski

---

**`Student` is a generic name for answers.**

## Individual tasks

#### **Zadanie 1.** Wyświetlić po 5 wierszy: tytuł filmu, metodę kompresji i nazwę kategorii, do której należy dany film. Wyniki sortowane po tytule. Można użyć iloczyn kartezjański.

title | compress | category
|:--:|:--:|:--:|
Bolek & Lolek - Australian steppes | MPEG 1 | Amusement
Bolek & Lolek - Gold Town | MPEG 1 | Amusement
Bolek & Lolek - On tiger's trails | MPEG 1 | Amusement
Bolek & Lolek - Pharaoh's Tomb | MPEG 1 | Amusement
Bolek & Lolek - Race to the North Pole | MPEG 1 | Amusement
(5 rows)

**Oneliner**

```sql
BEGIN; DECLARE xyz cursor for SELECT e.title, m.compress, c.name AS "category" FROM episodes_list e, movies_list m, categories c WHERE e.mid=m.mid AND c.cid=m.cid ORDER BY e.title; FETCH 5 FROM xyz; FETCH 5 FROM xyz; CLOSE xyz; COMMIT;
```

**Beautiful syntax**

```sql
BEGIN; 
DECLARE xyz cursor for 
    SELECT 
        e.title, m.compress, c.name AS "category" 
    FROM 
        episodes_list e, movies_list m, categories c 
    WHERE 
        e.mid=m.mid 
        AND c.cid=m.cid 
    ORDER BY e.title; 
FETCH 5 
    FROM xyz; 
FETCH 5 
    FROM xyz; 
CLOSE xyz; 
COMMIT;
```



---

#### **Zadanie 2.** Zrealizować zadanie 1 korzystając z klauzul LIMIT/offset. Wyniki jak w zad 2.

title | compress | category
|:--:|:--:|:--:|
Bolek & Lolek - Australian steppes | MPEG 1 | Amusement
Bolek & Lolek - Gold Town | MPEG 1 | Amusement
Bolek & Lolek - On tiger's trails | MPEG 1 | Amusement
Bolek & Lolek - Pharaoh's Tomb | MPEG 1 | Amusement
Bolek & Lolek - Race to the North Pole | MPEG 1 | Amusement
(5 rows)

**Oneliner**

```sql
SELECT e.title, m.compress, c.name FROM episodes_list e, movies_list m, categories c WHERE e.mid=m.mid AND c.cid=m.cid ORDER BY e.title LIMIT 5; SELECT e.title, m.compress, c.name FROM episodes_list e, movies_list m, categories c WHERE e.mid=m.mid AND c.cid=m.cid order by e.title LIMIT 5 offset 5;
```

**Beautiful syntax**

```sql
SELECT 
    e.title, m.compress, c.name AS category 
FROM 
    episodes_list e, movies_list m, categories c 
WHERE 
    m.mid=e.mid 
    AND m.cid=c.cid 
ORDER BY e.title LIMIT 5;
```

---

## Group tASks

**Uwagi:**
1. Ćwiczenia wykonujemy w parach
2. Jedna osoba pełni rolę administratora bazy danych, na jej bazie danych wykonujemy ćwiczenia.
(Administratorem bazy jest osoba, która ją założyła).
3. Druga osoba jest zwykłym użytkownikiem, zalogowanym do bazy.

Administrator bazy loguje się ze swojego terminala do bazy komendą:

`psql nazwa_bazy`

Użytkownik loguje się ze swojego terminala komendą:

`psql nazwa_bazy włASny_login`


| Lp | AdministraTOr bazy | Użytkownik |
|:--:|:--:|:--:|
| 1. | `SELECT * FROM categories;` | `SELECT * FROM categories;` <br> Zapytanie nie powiedzie się, ze względu na brak <br>uprawnień do tej operacji. |
| 2. | `GRANT SELECT ON categories TO public;` <br> Zezwala na wykonywanie komendy SELECT wszystkim użytkownikom. |
| 3. | | SELECT * FROM categories; |
| 4. | `\z` <br>Wyświetla bieżące uprawnienia na obiektach (tabelach, widokach i sekwencjach) | `\z` |
| 5. | `revoke SELECT ON categories FROM public;` <br>Odwołanie prawa dostępu. | |
| 6. |  | `INSERT INTO categories (cid, name, cserid) values (50, 'test', 1);` <br>Komenda się nie wykona, brak uprawnień. |
| 7. | **Zadanie:** jednym poleceniem nadać prawo do wykonywania *INSERT* oraz *delete* na tabeli *categories* dla Użytkownika, nie dla wszystkich.<br> `grant insert, delete on categories to student;` | |
| 8. | | Przetestować nadane uprawnienia |
| 9. | **Zadanie:** Nadać komplet uprawnień dla Użytkownika dla tabeli *categories* <br> `grant all on categories to student;` | |
| 10. | | Przetestować ***SELECT***, ***INSERT***, ***UPDATE***, ***delete*** |
| 11. | **Zadanie:** Odwołać prawo do *UPDATE*, *delete* i *SELECT* dla tabeli *categories* dla Użytkownika.<br> `revoke UPDATE, delete, SELECT on categories FROM student;` |

---

#### **Zadanie 3.** Należy umożliwić Użytkownikowi dodawanie nowych wierszy do tabeli *categories*, oraz wyświetlanie tylko kolumn *cid* oraz *name*. Użytkownik nie może mieć prawa odczytu do kolumny *cserid*!

**Username do zmiany!**
```sql
GRANT INSERT ON categories TO student;
GRANT SELECT(cid,name) ON categories TO student;
```
Opcja druga

```sql
grant insert, SELECT (cid, name) on table categories to student;
```
---

#### **Zadanie 4.** Napisać funkcję silnia przyjmującą jako argument warTOść ***int4*** i zwracająca ***int4***.

**Oneliner**
```sql
CREATE FUNCTION silnia (int4) RETURNS int4 AS 'DECLARE a int4 default 1; b int4 default 1; BEGIN if $1 < 0 then raise exception ''No negatives''; else for a in 1 .. $1 loop b=b*a; end loop; end if; return b; END;' Language 'plpgsql';
```

**Beautiful syntax**

```sql
CREATE FUNCTION silnia (int4) RETURNS int4 AS'
DECLARE
    a int4 default 1;
    b int4 default 1;
BEGIN
    if $1 < 0 then raise exception ''No negatives'';
    else
    for a in 1 .. $1 loop
    b=b*a;
    end loop;
    end if;
    return b;
END;
' Language 'plpgsql';
```

---

#### **Zadanie 5.** Napisać funkcję, która liczy warTOść średnią z kolumny ***konta.konto***.

**Oneliner**
```sql
CREATE FUNCTION srednia () RETURNS float AS 'DECLARE c float; BEGIN	SELECT avg(konto) INTO c FROM konta; return c; end;' Language 'plpgsql';
```

**Beautiful syntax**
```sql
CREATE FUNCTION srednia () RETURNS float AS '
DECLARE
    	c float;
BEGIN
    	SELECT avg(konto) INTO c FROM konta;
    	return c;
end;
' Language 'plpgsql';
```

---

#### **Zadanie 6.** Zmodyfikować trigger ***x_2t***, tak, aBY działał również po każdym ***INSERT***, ***UPDATE*** oraz ***delete***.

**Beautiful syntax**

```sql
CREATE function x_2 () RETURNS trigger as '
DECLARE
c int4;
BEGIN
SELECT sum(konto) into c FROM konta;
raise notice ''Suma kont wynosi %'', c;
return NEW;
end;
' language 'plpgsql';
CREATE trigger x_2t after insert or UPDATE or delete on konta for each row execute procedure x_2();
```
**Oneliner**

```sql
CREATE function x_2 () RETURNS trigger as 'DECLARE c int4; BEGIN SELECT sum(konto) into c FROM konta; raise notice ''Suma kont wynosi %'', c; return NEW; end;' language 'plpgsql'; CREATE trigger x_2t after insert or UPDATE or delete on konta for each row execute procedure x_2();
```

---

#### **Zadanie 7.** Napisać trigger, który zapewni unikalność wpisów w kolumnie ***konto*** w tabeli ***konta***.

**Beautiful syntax**

```sql
CREATE FUNCTION x_x() RETURNS trigger AS'
DECLARE                    
r konta%rowtype;
BEGIN
for r in SELECT id,konto FROM konta loop
if NEW.konto = r.konto then
raise notice ''Podana wartosc juz istnieje w tabeli.'';
return null;
end if;
end loop;
return NEW;
end;
'Language 'plpgsql';
CREATE trigger x_7t BEFORE UPDATE OR INSERT ON konta for each row execute procedure x_7();
```

**Oneliner**

```sql
CREATE FUNCTION x_x() RETURNS trigger AS 'DECLARE r konta%rowtype; BEGIN for r in SELECT id,konto FROM konta loop if NEW.konto = r.konto then raise notice ''Podana wartosc juz istnieje w tabeli.''; return null; end if; end loop; return NEW; end;' Language 'plpgsql'; CREATE trigger x_7t BEFORE UPDATE OR INSERT ON konta for each row execute procedure x_7();
```

---