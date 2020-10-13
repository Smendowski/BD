# Zadania lab 2

## Autor @http://github.com/marcinswistak
---

### __Zad1.__ Wyświetlić identyfikatory filmów w których w tytule filmu lub w nazwie autora występuje słowo Techno (... WHERE xxxx ~ 'Techno' ... gdzie xxxx jest nazwą kolumny w której szukamy tego słowa).	

```sql
SELECT W1.mid FROM movies_list W1 WHERE  W1.author ~ 'Techno' UNION SELECT W2.mid FROM episodes_list W2 WHERE W2.title ~ 'Techno';
```
***

### __Zad2.__ Jak wyżej, ale wypisać także autora i tytuł każdego znalezionego filmu.

```sql
SELECT W1.mid, W1.author, W2.title from movies_list AS W1 INNER JOIN episodes_list AS W2 ON W1.mid = W2.mid where W1.author ~ 'Techno' OR W2.title ~ 'Techno';
```
***

### __Zad3.__ Załadować ze zrzutu ~rstankie/db/stud2.dump tabelę episodes_list2. Tabela episodes_list2 zawiera dane z tabeli episodes_list z wprowadzonymi drobnymi zmianami. Ustalić, które wiersze zostały zmienione/usunięte/dodane.

```bash
psql mswistak < ~rstankie/db/stud2.dump
```

```sql
SELECT * FROM episodes_list EXCEPT SELECT * FROM episodes_list2;
SELECT * FROM episodes_list2 EXCEPT SELECT * FROM episodes_list;
```

__TO JEST DZIWNE BARDZO DZIWNE!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!__
***

### __Zad4.__ Wyświetlić tytuły filmów (is_movie = 1) z tabeli episodes_list.
```sql
select title from episodes_list where is_movie = 1;
```
***

### __Zad5.__ Wyświetlić tytuł filmu, ID kategorii i ID podkategorii, do której należy dany film.

```sql
SELECT W2.title, W1.cid, W1.sid from movies_list AS W1 INNER JOIN episodes_list AS W2 ON W1.mid = W2.mid where W2.is_movie = 1;
```
***

### __Zad6.__ Wyświetlić tytuł filmu, NAZWĘ kategorii i NAZWĘ podkategorii, do której należy dany film. Zastanów się dlaczego zapytanie zwraca o jeden wiersz mniej niż w zadaniu 5. Znajdź brakujący wiersz i wyjaśnij.

```sql
SELECT 	e.title, c.name, s.name 
	FROM movies_list AS m
		INNER JOIN episodes_list AS e ON e.mid = m.mid
		INNER JOIN categories AS c ON c.cid = m.cid
		INNER JOIN subcategories AS s On s.sid = m.sid
	WHERE e.is_movie = 1;
```

#### *Image coding (part 2)*, ponieważ nie ma w bazie danych cid oraz sid więc nie ma wpisaów odnoścnie katedori oraz podakterorii 

```sql
SELECT W2.title from movies_list AS W1 INNER JOIN episodes_list AS W2 ON W1.mid = W2.mid where W2.is_movie = 1 EXCEPT SELECT e.title FROM movies_list AS m INNER JOIN episodes_list AS e ON e.mid = m.mid INNER JOIN categories AS c ON c.cid = m.cid INNER JOIN subcategories AS s On s.sid = m.sid WHERE e.is_movie = 1;
```
***

### __Zad7.__ Wyświetlić numery i tytuły filmów należących do kategorii 63.

```sql
	SELECT m.mid, e.title from movies_list AS m
		INNER JOIN episodes_list AS e ON m.mid = e.mid
	WHERE m.cid = 63;
```
***

### __Zad8.__ Wyświetlić numery i tytuły filmów należących do kategorii "Education" posortowane po tytułach.

```	sql
	SELECT m.mid, e.title from movies_list AS m
		INNER JOIN episodes_list AS e ON m.mid = e.mid
		INNER JOIN categories AS c ON c.cid = m.cid
	WHERE c.name = 'Education' AND e.is_movie = 1 ORDER BY e.title;
```


### __Zad9.__ Wyświetlić listę epizodów (jako episode) oraz tytuły filmów, których te epizody są fragmentami (jako movie). Posortować według tytułu filmu (rosnąco) oraz wg czasu początku epizodu (malejąco).

```sql
SELECT e.title AS episode, ee.title AS movie from episodes_list e INNER JOIN episodes_list ee ON e.mid = ee.mid WHERE e.is_movie=0 AND ee.is_movie=1 ORDER BY ee.title asc, e.episode_start DESC;
```
***

### __Zad10.__ Jak wyżej tylko dodać jeszcze kolumnę 'start' z czasem początku epizodu w formacie: HH:MM:SS. W tabeli czas podany jest w milisekundach. Zalecane użycie funkcji reltime(int4).

```sql
SELECT reltime(e.episode_start/1000), e.title AS episode, ee.title AS movie from episodes_list e INNER JOIN episodes_list ee ON e.mid = ee.mid WHERE e.is_movie=0 AND ee.is_movie=1 ORDER BY ee.title asc, e.episode_start DESC;
```
***

### __Zad11.__ Jak wyżej, ale wyświetlić dodatkowo nazwę kategorii i podkategorii.

```sql
SELECT reltime(e.episode_start/1000), e.title AS episode, ee.title AS movie, c.name AS category, s.name AS subcategory from movies_list m INNER JOIN episodes_list2 ee ON m.mid = ee.mid INNER JOIN episodes_list e ON m.mid=e.mid INNER JOIN categories c On m.cid=c.cid INNER JOIN subcategories s ON s.sid=m.sid WHERE e.is_movie=0 AND ee.is_movie=1 ORDER BY ee.title asc, e.episode_start DESC;
```
***

### __Zad12.__ Wyświetlić tytuły epizodów, których czas trwania jest większy od 100000. Wyświetlić również czasy trwania epizodów i posortować wg czasu trwania.

```sql
SELECT e.title AS episode_title, e.episode_end-e.episode_start from episodes_list e WHERE (e.episode_end-e.episode_start)>100000 AND e.is_movie=0 ORDER BY (e.episode_end-e.episode_start);
```
***