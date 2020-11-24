# Bazy Danych / Lab_8 Zadania

Author: @https://github.com/piotrsladowski

---

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
begin; declare xyz cursor for SELECT e.title, m.compress, c.name FROM episodes_list e, movies_list m, categories c WHERE e.mid=m.mid AND c.cid=m.cid ORDER BY e.title; fetch 5 FROM xyz; close xyz; commit;
```

**Beautiful syntax**

```sql
begin;
declare xyz cursor for 
    SELECT 
        e.title, m.compress, c.name 
    FROM 
        episodes_list e, movies_list m, categories c 
    WHERE 
        e.mid=m.mid 
        AND c.cid=m.cid 
    ORDER BY e.title;
fetch 5 FROM xyz;
close xyz;
commit;
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
SELECT e.title, m.compress, c.name AS category FROM episodes_list e, movies_list m, categories c WHERE m.mid=e.mid AND m.cid=c.cid ORDER BY e.title LIMIT 5;
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
| 6. |  | `INSERT inTO categories (cid, name, cserid) values (50, 'test', 1);` <br>Komenda się nie wykona, brak uprawnień. |
| 7. | **Zadanie:** jednym poleceniem nadać prawo do wykonywania *INSERT* oraz *delete* na tabeli *categories* dla Użytkownika, nie dla wszystkich. | |
| 8. | | Przetestować nadane uprawnienia |
| 9. | **Zadanie:** Nadać komplet uprawnień dla Użytkownika dla tabeli *categories* | |
| 10. | | Przetestować ***SELECT***, ***INSERT***, ***update***, ***delete*** |
| 11. | **Zadanie:** Odwołać prawo do *update*, *delete* i *SELECT* dla tabeli *categories* dla Użytkownika |

---

#### **Zadanie 3.** Należy umożliwić Użytkownikowi dodawanie nowych wierszy do tabeli *categories*, oraz wyświetlanie tylko kolumn *cid* oraz *name*. Użytkownik nie może mieć prawa odczytu do kolumny *cserid*!

**Username do zmiany!**
```sql
GRANT INSERT ON categories TO psladows;
GRANT SELECT(cid,name) ON categories TO psladows;
```

---

#### **Zadanie 4.** Napisać funkcję silnia przyjmującą jako argument warTOść ***int4*** i zwracająca ***int4***.

```sql
CREATE FUNCTION silnia (int4) RETURNS int4 AS'
declare
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

```sql
CREATE FUNCTION srednia () RETURNS float AS '
declare
    	c float;
begin
    	SELECT avg(konto) INTO c FROM konta;
    	return c;
end;
' Language 'plpgsql';
```

---

#### **Zadanie 6.** Zmodyfikować trigger ***x_2t***, tak, aBY działał również po każdym ***INSERT***, ***update*** oraz ***delete***.

```sql
create function x_2 () returns trigger as '
declare
c int4;
begin
select sum(konto) into c from konta;
raise notice ''Suma kont wynosi %'', c;
return NEW;
end;
' language 'plpgsql';
create trigger x_2t after insert or update or delete on konta for each row execute procedure x_2();
```


---


#### **Zadanie 7.** Napisać trigger, który zapewni unikalność wpisów w kolumnie ***konto*** w tabeli ***konta***.

```sql
CREATE FUNCTION x_x() RETURNS trigger AS'
declare                    
r konta%rowtype;
begin
for r in SELECT id,konto FROM konta loop
if NEW.konto = r.konto then
raise notice ''Podana wartosc juz istnieje w tabeli.'';
return null;
end if;
end loop;
return NEW;
end;
'Language 'plpgsql';
create trigger x_7t before update on konta for each row execute procedure x_7();
```

---