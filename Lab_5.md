# Bazdy Danych / Lab_5 Zadania

Author: @https://github.com/Smendowski

---

#### **Zadanie 1.** Wyświetlić nazwę kategorii i sumę czasów trwania filmów należących do poszczególnych kategorii. Użyj CROSS JOIN.

```sql
SELECT c.name, SUM(CASE WHEN c.cid = m.cid THEN m.lenmsec ELSE 0 END)
    FROM categories c CROSS JOIN movies_list m
    GROUP BY c.name
    ORDER BY c.name ASC
    LIMIT 4;
```

```diff
- 'CROSS' JOIN samo w sobie to połączenie bezwarunkowe działające jak iloczyn kartezjański.
+ Sumujemy ze sobą czasy filmów należących tylko do tej samej kategorii.
! 'LIMIT' ogranicza nam liczbę wyświetlanych wierszy. Pomijamy przez to kategorię 'Others'.
```

#### **Zadanie 2.**  Policzyć ile epizodów (jako episode nr) należy do danego filmu. Wypisać tytuły filmów, których te epizody są fragmentami (jako movie). Posortować według tytułu filmu (rosnąco). Dodatkowo wyświetlić nazwę kategorii i podkategorii. Użyj CROSS JOIN.

#### **Zadanie 3.** Wykonaj Zadanie 1 i Zadanie 2 z użyciem INNER JOIN.

#### **Zadanie 4.** Wyświetlić tytuł filmu, NAZWĘ kategorii i NAZWĘ podkategorii, do której należy dany film. Mają byćwyświetlone wszystkie filmy, nawet te nie przypisane do kategorii.

#### **Zadanie 5.** Policzyć ile filmów należy do danej kategorii. W statystyce uwzględnić filmy, które nie należą dożadnej kategorii pod nazwą "--n/a--".

#### **Zadanie 6.**  Wyświetlić listę, zawierającą dwie kolumny: tytuł epizodu, tytuł filmu. Na wyniku mają sie pojawić wszystkie filmy (nawet te, które nie posiadają epizodów).

#### **Zadanie 7.** Dla każdej kategorii wyświetlić sumę długości filmów należących do niej. Dla kategorii, które niezawierają żadnych filmów suma długości ma być równa 0 (nie NULL!!!). Filmy nie należące do żadnej kategorii należy uwzględnić jako osobną kategorię bez nazwy (NULL).