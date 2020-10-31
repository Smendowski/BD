# Bazdy Danych / Lab_5 Zadania

Author: @https://github.com/Smendowski

---

#### **Zadanie 1.** Wyświetlić nazwę kategorii i sumę czasów trwania filmów należących do poszczególnych kategorii. Użyj CROSS JOIN.
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
<font size="2">Czytelnie formatowanie zapytania</font>
```sql
SELECT  COUNT(*) AS "episode nr",
        e1.title AS "movie",
        c.name AS "category",
        s.name AS "subcategory"
FROM 
    # Dwa razy kożystamy z tabeli espisodes_list bo będziemy
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
<font size="2">Zadanie 1 z użyciem INNER JOIN</font>
```sql
SELECT c.name, sum(m.lenmsec) FROM categories c INNER JOIN movies_list m USING(cid) GROUP BY c.name ORDER BY c.name;
```
<font size="1">Komentarz: Używając INNER JOIN == JOIN pozbywamy się warunku WHERE koniecznego w CROSS JOIN, który zmniejszał czytelność. USING(nazwa_kolumny) spowoduje złączenie tych wszystkich wierszy tabel biorących udział w INNER JOIN, które w kolumnach mają zgodność co do nazwa_kolumny czyli argumentu USING()</font>

<font size="2">Zadanie 2 z użyciem INNER JOIN</font>
```sql

```
#### **Zadanie 4.** Wyświetlić tytuł filmu, NAZWĘ kategorii i NAZWĘ podkategorii, do której należy dany film. Mają byćwyświetlone wszystkie filmy, nawet te nie przypisane do kategorii.

#### **Zadanie 5.** Policzyć ile filmów należy do danej kategorii. W statystyce uwzględnić filmy, które nie należą dożadnej kategorii pod nazwą "--n/a--".

#### **Zadanie 6.**  Wyświetlić listę, zawierającą dwie kolumny: tytuł epizodu, tytuł filmu. Na wyniku mają sie pojawić wszystkie filmy (nawet te, które nie posiadają epizodów).

#### **Zadanie 7.** Dla każdej kategorii wyświetlić sumę długości filmów należących do niej. Dla kategorii, które niezawierają żadnych filmów suma długości ma być równa 0 (nie NULL!!!). Filmy nie należące do żadnej kategorii należy uwzględnić jako osobną kategorię bez nazwy (NULL).