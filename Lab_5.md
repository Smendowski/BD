# Bazdy Danych / Lab_5 Zadania

Author: @https://github.com/Smendowski

---

#### **Zadanie 1.**  Wyświetlić nazwę kategorii i sumę czasów trwania filmów należących do poszczególnych kategorii. Użyj CROSS JOIN.
<font size="2">Brzydka wersja zapytania</font>
```sql
SELECT c.name, SUM(CASE WHEN c.cid = m.cid THEN m.lenmsec ELSE 0 END)
    FROM categories c CROSS JOIN movies_list m
    GROUP BY c.name
    ORDER BY c.name ASC
    LIMIT 4;
```
<font size="2">Poprawna wersja zapytania</font>
```sql
SELECT c.name, SUM(m.lenmsec) FROM categories c CROSS JOIN movies_list m WHERE c.cid=m.cid GROUP BY c.name ORDER BY c.name;
```

#### **Zadanie 2.**  Policzyć ile epizodów (jako episode nr) należy do danego filmu. Wypisać tytuły filmów, których te epizody są fragmentami (jako movie). Posortować według tytułu filmu (rosnąco). Dodatkowo wyświetlić nazwę kategorii i podkategorii. Użyj CROSS JOIN.
<font size="2">Czytelnie formatowane zapytanie</font>
```sql
SELECT  COUNT(*) AS "episode nr",
        e1.title AS "movie",
        c.name AS "category",
        s.name AS "subcategory"
FROM 
    # Dwa razy korzystamy z tabeli espisodes_list bo będziemy
    # potrzebowali zarówno pełnych filmów jak i epizodów.
    episodes_list e1
    CROSS JOIN episodes_list e2
    CROSS JOIN categories c
    CROSS JOIN subcategories s
    CROSS JOIN movies_list m
WHERE
    e1.mid=e2.mid
    AND e1.is_movie=1
    AND e2.is_movie=0
    AND m.mid=e1.mid
    AND m.cid=c.cid
    AND m.sid=s.sid   
GROUP BY e1.title, c.name, s.name ORDER BY ASC;
```
<font size="2">Jednolinijkowiec</font>
```sql
SELECT COUNT(*) AS "episode nr", e1.title AS "movie", c.name AS "category" ,s.name AS "subcategory" FROM episodes_list e1 CROSS JOIN episodes_list e2 CROSS JOIN categories c CROSS JOIN subcategories s CROSS JOIN movies_list m WHERE e1.mid=e2.mid AND e1.is_movie=1 AND e2.is_movie=0 AND m.mid=e1.mid AND m.cid=c.cid AND m.sid=s.sid GROUP BY e1.title, c.name, s.name ORDER BY 2;
```
<font size="2">Komentarz: ORDER BY 2 oznacza ORDER BY 2-ga kolumna wymieniona w zapytaniu SELECT</font>


#### **Zadanie 3.** Wykonaj Zadanie 1 i Zadanie 2 z użyciem INNER JOIN.
<font size="3">Zadanie 1 z użyciem INNER JOIN</font>
```sql
SELECT c.name, sum(m.lenmsec) FROM categories c INNER JOIN movies_list m USING(cid) GROUP BY c.name ORDER BY c.name;
```
<div style="text-align: justify"><font size="1">Komentarz: Używając INNER JOIN == JOIN pozbywamy się warunku WHERE koniecznego w CROSS JOIN, który zmniejszał czytelność. USING(nazwa_kolumny) spowoduje złączenie tych wszystkich wierszy tabel biorących udział w INNER JOIN, które w kolumnach mają zgodność co do nazwa_kolumny czyli argumentu USING()</font></div>

<br/>
<font size="3">Zadanie 2 z użyciem INNER JOIN</font>
<br>


**<font size="2">Czytelne formatowanie zapytania</font>**
```sql
SELECT  COUNT(*) AS "episode nr", 
        e1.title, 
        c.name, 
        s.name 
FROM episodes_list e1 
    INNER JOIN episodes_list e2 USING(mid) 
    INNER JOIN movies_list   m  USING(mid)
    INNER JOIN categories    c  USING(cid) 
    INNER JOIN subcategories s  USING(sid) 
WHERE e1.is_movie=1 and e2.is_movie=0       # Zauważamy zaletę. Pozbylismy się nadmiaru
GROUP BY e1.title,c.name,s.name             # warunków Where dzięki USING i INNER JOIN
ORDER BY 2;
```
**<font size="2">Jednolinijkowiec</font>**
```sql
SELECT  COUNT(*) AS "episode nr", e1.title, c.name, s.name FROM episodes_list e1 INNER JOIN episodes_list e2 USING(mid) INNER JOIN movies_list m USING(mid) INNER JOIN categories c USING(cid) INNER JOIN subcategories s USING(sid) WHERE e1.is_movie=1 and e2.is_movie=0       GROUP BY e1.title,c.name,s.name ORDER BY 2;
```
<div style="text-align: justify"><font size="1">Komentarz: Operując na wielu tabelach, użycie INNER JOIN pozwala nam zredukować warunki WHERE zmniejszające czytelność zapytania.</font></div>

#### **Zadanie 4.** Wyświetlić tytuł filmu, NAZWĘ kategorii i NAZWĘ podkategorii, do której należy dany film. Mają byćwyświetlone wszystkie filmy, nawet te nie przypisane do kategorii.
```sql
SELECT e.title, c.name, s.name FROM episodes_list e LEFT JOIN movies_list m USING(mid) LEFT JOIN categories c USING(cid) LEFT JOIN subcategories s USING(sid) WHERE e.is_movie=1;
```
<div style="text-align: justify"><font size="1">Komentarz: LEFT JOIN jest to to samo co INNER JOIN ale dodatkowo do tabeli po "lewej stronie" dodawane są wiersze z tabeli po "prawej stronie" wyrażenia LEFT JOIN nawet jeśli nie znaleziono dopasowania przy USING()</font></div>

#### **Zadanie 5.** Policzyć ile filmów należy do danej kategorii. W statystyce uwzględnić filmy, które nie należą dożadnej kategorii pod nazwą "--n/a--".
```sql
SELECT COUNT(*), COALESCE(c.name, '--n/a--') FROM categories c RIGHT JOIN movies_list m USING(cid) GROUP BY c.name ORDER BY c.name;
```
<div style="text-align: justify"><font size="1">Komentarz: RIGHT JOIN jest to to samo co INNER JOIN ale dodatkowo do tabeli po "prawej stronie" dodawane są wiersze z tabeli po "lewej stronie" wyrażenia RIGHT JOIN nawet jeśli nie znaleziono dopasowania przy USING().</font></div>

#### **Zadanie 6.**  Wyświetlić listę, zawierającą dwie kolumny: tytuł epizodu, tytuł filmu. Na wyniku mają sie pojawić wszystkie filmy (nawet te, które nie posiadają epizodów).
**<font size="2">Czytelne formatowanie zapytania</font>**
```sql
SELECT e1.title AS "movie title", e2.title AS "episode title"
FROM
    (SELECT mid, title FROM episodes_list WHERE is_movie=1) e1
    FULL OUTER JOIN
    (SELECT mid, title FROM episodes_list WHERE is_movie=0) e2
    ON e1.mid = e2.mid;
```

**<font size="2">Jednolinijkowiec</font>**
```sql
SELECT e1.title AS "movie title", e2.title AS "episode title" FROM (SELECT mid, title FROM episodes_list WHERE is_movie=1) e1 FULL OUTER JOIN (SELECT mid, title FROM episodes_list WHERE is_movie=0) e2 ON e1.mid = e2.mid;
```
<div style="text-align: justify"><font size="1">Komentarz: W podzapytaniu pierwszym uzyskujemy tabelę z filmami. W podzaytaniu drugim uzyskujemy tabelę z epizodami. FULL JOIN dokonuje złączenia obu tabel == łączone są wiersze obu tabel jeśli spełniony jest warunek ON na kolumnach. FULL JOIN jest to LEFT JOIN + RIGHT JOIN. Nawet jeśli nie ma dopasowania to przepisujemy wiersze.</font></div>

#### **Zadanie 7.** Dla każdej kategorii wyświetlić sumę długości filmów należących do niej. Dla kategorii, które niezawierają żadnych filmów suma długości ma być równa 0 (nie NULL!!!). Filmy nie należące do żadnej kategorii należy uwzględnić jako osobną kategorię bez nazwy (NULL).
**<font size="2">Czytelne formatowanie zapytania</font>**
```sql
SELECT COALESCE(SUM(m.lenmsec),0) AS "case", c.name AS "name" 
    FROM categories c FULL OUTER JOIN movies_list m
ON m.cid = c.cid GROUP BY c.name ORDER BY c.name;
```
**<font size="2">Jednolinijkowiec</font>**
```sql
SELECT COALESCE(SUM(m.lenmsec),0) AS "case", c.name AS "name" FROM categories c FULL OUTER JOIN movies_list m ON m.cid = c.cid GROUP BY c.name ORDER BY c.name;
```
