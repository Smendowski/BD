# Bazy Danych / Lab_7

To MarkDown transcripted @http://github.com/hbery

##### version draft-0.1

> *Commentary: Jak pojawią się zadania to dodam*

---

## Transakcje

Najistotniejszą sprawą przy projektowaniu i korzystaniu z bazy danych jest zachowanie spójności informacji w KAŻDYCH warunkach, również w przypadku równoczesnego dokonywania operacji przez wielu użytkowników. Jednym z mechanizmów, umożliwiających zapewnienie spójności są transakcje.

Transakcja jest wykonywana na zasadzie „wszystko albo nic”, tzn. albo wszystkie komendy w transakcji zostaną poprawnie wykonane, albo (w razie dowolnego błędu) wszystkie dokonane w transakcji zmiany zostaną odwołane.
Domyślnie w systemie Postgres transakcja obejmuje jedno polecenie wydane przez użytkownika (baza zawsze wykonuje polecenia użytkownika w transakcji).

Zatem polecenie select * from categories; jest w rzeczywistości wykonywane jako:
* `begin;`
* `select * from categories;`
* `commit;`

Można jednak wymusić poleceniami BEGIN i COMMIT wykonanie kilku poleceń w ramach jednej transakcji.
Do wykonania poniższych ćwiczeń należy otworzyć dwa okna, w obu uruchomić psql. Zapytania z lewej kolumny należy wykonywać w jednym oknie, zapytania z drugiej - w drugim. Polecenia należy wykonywać kolejno, w odpowiednich oknach. Jeżeli w jednej linijce są dwa polecenia dla różnych okien to można je wykonać w dowolnej kolejności.
Przykład transakcji z rollback:

| Lp | Okno 1 | Okno 2 |
|---:|:-------|:-------|
1.| | `\d`|
2.| `begin;` <br>otwarcie transakcji| 
3.| `create table test as select * from categories;`
4.| `\d` <br>tabela test jest widoczna tylko wewnątrz tej transakcji | `\d` <br>tabela test nie jest widoczna bo transakcja w oknie 1 nie została jeszcze zatwierdzona
5.| `rollback;` <br>odwołanie transakcji
6.| `\d` <br>tabela test nie istnieje, bo transakcja została odwołana | `\d` 

Przykład transakcji z commit: 
Zwrócić uwagę na to, że tabela 'osoby' nie jest widoczna do czasu wykonania 'commit'.

| Lp | Okno 1 | Okno 2 |
|---:|:-------|:-------|
7.| `begin;` |
8.| `create table osoby (imie varchar(40), nazw varchar(40), konto int4);`
9.| `\d` | `\d`
10.| `insert into osoby (imie, nazw, konto) values ('Jan', 'Kos', 3000);`
11.| `select * from osoby;` <br>wprowadzone wiersze są widoczne tylko wewnątrz transakcji | `select * from osoby;` <br>zwraca błąd, bo na zewnątrz niezatwierdzonej transakcji tabela osoby nie istnieje
12.| `insert into osoby (imie, nazw, konto) values ('Ala', 'Waga', 4000);`
13.| `select * from osoby;`
14.| `commit;` <br>zatwierdzenie transakcji
15.| `\d` | `\d`
16.| | `select * from osoby;` <br>zarówno tabela jak i wprowadzone wiersze są widoczne dla wszystkich

Błąd w transakcji powoduje przejście do trybu “Abort state”. Przez błąd należy rozumieć zarówno błąd w składni polecenia, jak też polecenie, które narusza ograniczenia bazy danych. Naruszenie ograniczeń ma miejsce np. wtedy, gdy kolumna w tabeli ma zawierać wartości dodatnie a polecenie próbuje wstawić wartość ujemną. Błędem NIE jest polecenie update (delete) które aktualizuje (kasuje) 0 wierszy (nie znajdzie wiersza pasującego do klauzuli where).
W momencie przejścia do trybu abort state wykonywany jest automatycznie rollback a reszta instrukcji w transakcji jest ignorowna. Takie zachowanie pozwala bez ryzyka wykonać zestaw komend (np. ze skryptu) bez sprawdzania, czy każda z nich się wykonała poprawnie.

| Lp | Okno 1 | Okno 2 |
|---:|:-------|:-------|
17.| `begin;` | `select * from osoby;`
18.| `update osoby set konto=konto-1000 where nazw='Kos';`
19.| `update osoby set konto=konto+1000 where nazw='Waga';`
20.| `select konta from osoby;` <br>__UWAGA: w tej linijce jest celowy błąd - 'konta'__
21.| `select konto from osoby;` <br>komenda zostanie zignorowana - Abort State| `select * from osoby;`
22.| `commit;` <br>koniec transakcji, transakcja została ODWOŁANA ze względu na wcześniejszy błąd
23.| | `select * from osoby;` <br>zawartość tabeli bez zmian

### Blokady wierszy

Modyfikowanie tych samych danych równocześnie przez kilka transakcji może prowadzić do błędów (takich jak błędne wyliczenie stanu konta, sprzedaż jednego towaru kilku osobom). Blokady wierszy są jednym z mechanizmów pozwalających na uniknięcie takiej sytuacji.
<br>
<br>
Wykonanie polecenia „update” powoduje automatyczną blokadę modyfikowanego wiersza. Inne transakcje próbujące zmodyfikować taki wiersz muszą poczekać na zakończenie pierwszej transakcji. Blokada obowiązuje do końca transakcji, w której została założona.

| Lp | Okno 1 | Okno 2 |
|---:|:-------|:-------|
24.| `begin;` | `begin;`
25.| `update osoby set konto=1000 where nazw='Waga';`
26.| | `update osoby set konto=konto+1000 where nazw='Waga';` <br>polecenie oczekuje na zakończenie konkurencyjnej transakcji
27.| `commit;` | po commit w oknie 1 polecenie z poprzedniej linii zostanie wykonane
28.| | `commit;`
29.| | `select * from osoby;`

Poniższy przykład prezentuje często popełniany błąd, polegający na niezablokowaniu wiersza przeznaczonego do modyfikacji już w momencie odczytu aktualnej wartości. Powoduje to, że w obu transakcjach zostanie odczytane dodatnie saldo konta pani Wagi i w konsekwencji zaakceptowanie obu przelewów; po ich wykonaniu pani Waga ma debet (a to nie powinno się zdarzyć - drugi przelew powinien zostać odrzucony).

| Lp | Okno 1 | Okno 2 |
|---:|:-------|:-------|
30.| | `update osoby set konto=1000 where nazw='Waga';` <br>przywrócenie danych w bazie do stanu pierwotnego
31.| `begin;` | `begin;`
32.| `select konto from osoby where nazw='Waga';`
33.|| `select konto from osoby where nazw='Waga';`
34.| `update osoby set konto=konto-1000 where nazw='Waga';`
35.| `update osoby set konto=konto+1000 where nazw='Kos';`
36.| `commit;`
37.|| użytkownik nie wie, że stan konta się zmienił od czasu zapytania z linii 33, dlatego kontynuuje wykonanie przelewu `update osoby set konto=konto-1000 where nazw='Waga';`
38.|| `update osoby set konto=konto+1000 where nazw='Kos';`
39.|| `commit;
40.|| `select * from osoby;` <br>tutaj p. Waga ma już debet 

Rozwiązaniem jest użycie w obu transakcjach „select for update”, który zablokuje wiersz już w momencie odczytu.

| Lp | Okno 1 | Okno 2 |
|---:|:-------|:-------|
41.|| `update osoby set konto=1000 where nazw='Waga';` <br>przywrócenie danych w bazie do stanu pierwotnego
42.| `begin;` | `begin;`
43.| `select konto from osoby where nazw='Waga' for update;` <br>zapewniamy sobie wyłączność na dostęp do wyświetlonych danych
44.|| `select konto from osoby where nazw='Waga' for update;` <br>select jest zablokowany do czasu zakończenia innych transakcji operujących na tym wierszu
45.| `update osoby set konto=konto-1000 where nazw='Waga';`
46.| `update osoby set konto=konto+1000 where nazw='Kos';`
47.| `commit;`
48.|| select został wykonany; <br>p. Waga ma stan konta = 0 więc użytkownik podejmuje decyzję o przerwaniu transakcji `rollback;`

Blokowanie wierszy może prowadzić do powstania zakleszczeń (deadlocks). W takich sytuacjach postgres radzi sobie sam.

| Lp | Okno 1 | Okno 2 |
|---:|:-------|:-------|
49.| `begin;` | `begin;`
50.|| `update osoby set konto=1000 where nazw='Waga';`
51.| `update osoby set konto=2000 where nazw='Kos';`
52.|| `update osoby set konto=1000 where nazw='Kos';`
53.| `update osoby set konto=2000 where nazw='Waga';`
54.| `commit;` | `commit;`

W przedstawionym poniżej przykładzie nie da się bezpośrednio zastosować blokowania rekordów za pomocą „select for update.” Operujemy tutaj dwoma tabelami: kursy i miejsca. Pierwsza z nich zawiera informacje o kursach autobusów, druga o miejscach zarezerwowanych w poszczególnych kursach. Pojawienie się rekordu w tabeli miejsca oznacza, że dane miejsce jest zajęte. Konflikt pomiędzy transakcjami w poniższym przykładzie wynika stąd, że obie transakcje odczytają, że miejsce nr. 1 w kursie 1 jest wolne, a w konsekwencji wstawią dwie rezerwacje tego samego miejsca

| Lp | Okno 1 | Okno 2 |
|---:|:-------|:-------|
55.| Przygotowanie tabel: <br>`create table kursy (kurs int4, opis varchar(40));` <br>`create table miejsca (kurs int4, miejsce int4);` <br>`insert into kursy (kurs, opis) values (1, 'Kurs1');`
56.| `begin;` | `begin;`
57.| Sprawdzamy, czy miejsce jest wolne <br>`select * from miejsca where kurs=1;`
58.| Miejsce jest wolne – rezerwujemy je <br>`insert into miejsca (kurs, miejsce) values (1, 1);`
59.|| Sprawdzamy, czy miejsce jest wolne <br>`select * from miejsca where kurs=1;`
60.|| Miejsce jest wolne – rezerwujemy je <br>`insert into miejsca (kurs, miejsce) values (1, 1);`
61.| `commit;` | `commit;`
62.| `select * from miejsca where kurs=1;`

Opisany konflikt można rozwiązać na dwa sposoby. Pierwszym z nich jest użycie PRZED sprawdzeniem wolnych miejsc komendy LOCK TABLE miejsca in ACCESS EXCLUSIVE MODE, co spowoduje, że nasza transakcja uzyska wyłączność na dostęp do tej tabeli. Wyłączność dotyczy również odczytu danych, co oznacza, że inne transakcje nie są w stanie sprawdzać wolnych miejsc. Jest to istotna wada tego rozwiązania, gdyż przy dużych bazach radykalnie spowolni korzystanie z systemu.
<br>
<br>
Drugie rozwiązanie zakłada, że wszyscy korzystający z bazy działają według tej samej procedury. Jeżeli wszyscy zainteresowani rezerwacją miejsc wykonają przed sprawdzeniem wolnych miejsc „select for update” na wierszu opisującym konkretny kurs, to efektywnie tylko jedna transakcja operująca na danym kursie będzie działać, reszta będzie czekać:

| Lp | Okno 1 | Okno 2 |
|---:|:-------|:-------|
63.| `delete from miejsca;`
64.| `begin;` | `begin;`
65.| `select kurs from kursy where kurs=1 for update;`
66.|| `select kurs from kursy where kurs=1 for update;`
67.| Sprawdzamy, czy miejsce jest wolne <br>`select * from miejsca where kurs=1;`
68.| Miejsce jest wolne – rezerwujemy je <br>`insert into miejsca (kurs, miejsce) values (1, 1);`
69.| `commit;`
70.|| Sprawdzamy, czy miejsce jest wolne <br>`select * from miejsca where kurs=1;`
71.|| Miejsce jest zajęte – wykonujemy rollback <br>`rollback;`
72.| `select * from miejsca where kurs=1;`

Zysk w porównaniu do LOCK TABLE jest podwójny: transakcje operujące na różnych kursach nie blokują się wzajemnie, ponadto transakcje, które w celach czysto informacyjnych uzyskują informacje o wolnych miejscach nie są blokowane wcale.

## Izolacja transakcji

W bazie danych mogą występować trzy niepożądane zjawiska:

* dirty reads - transakcja odczytuje dane wprowadzone ale niezatwierdzone przez inną transakcję.

* non-repeatable reads - zjawisko jest widoczne w sytuacji, gdy transakcja odczytuje dane, czeka chwilę, następnie ponownie odczytuje te same dane i stwierdza, że dane uległy zmianie (przez inną zatwierdzoną transakcję)

* phantom read - transakcja ponownie wykonuje zapytanie zwracające określone wiersze i stwierdza, że pojawiły się nowe wiersze spełniające zadane kryterium (wstawione przez inną, zatwierdzoną transakcję)

Zajwisko „dirty-reads” nie występuje w Postgresie, co można zauważyć w poprzednich przykładach. Występowanie zjawisk „non-repeatable reads” oraz „phantom read” zależy od bieżącego poziomu izolacji transakcji. Domyślnym poziomem izolacji jest poziom „Read Committed”, w którym te zjawiska występują. Można korzystać także z poziomu „Serializable”, w którym te zjawiska są eliminowane.

Tryb serializable emuluje wykonywanie transakcji tak, jakby były wykonywane jedna po drugiej a nie równolegle. Stąd w trybie „Serializable” transakcja „widzi” stan bazy z momentu rozpoczęcia transakcji. Zmiany wprowadzone do bazy przez inne transakcje nie są widoczne. Ponadto operacje na wierszu (UPDATE, DELETE, SELECT FOR UPDATE), który jest aktualizowany przez konkurencyjną transakcję są wstrzymywane do czasu zakończenia tej transakcji. Jeżeli konkurencyjna transakcja zostanie odwołana to bieżąca transakcja będzie działać nadal. Jeżeli konkurencyjna transakcja zostanie zatwierdzona, to bieżąca transakcja zostanie zerwana, ze względu na brak możliwości dalszego emulowania wykonywania transakcji jedna po drugiej.

| Lp | Okno 1 | Okno 2 |
|---:|:-------|:-------|
73.| przykład w domyślnym trybie pracy bazy danych <br>`begin;` | `begin;`
74.| `select * from osoby;` | zwrócić uwagę na stan konta p. Wagi
75.|| `update osoby set konto=1500 where nazw='Waga';`
76.|| `commit;`
77.| `select * from osoby;` <br>stan konta p. Wagi uległ zmianie (non-repeatable read)
78.| `update osoby set konto=2000 where nazw='Waga';` <br>update wykonuje się normalnie
79.| `commit;`

Przykład zerwania transakcji (wygrała transakcja, która wcześniej zrobiła „update”). Zwrócić uwagę na fakt, że oba selecty w oknie 1 podają te same dane mimo commit'a w oknie 2. Jest to właściwe narzędzie do tworzenia raportów, zamraża bowiem spójny obraz bazy danych z chwili rozpoczęcia transakcji.

| Lp | Okno 1 | Okno 2 |
|---:|:-------|:-------|
80.| `begin;` | `begin;`
81.| `set transaction isolation level serializable;` <br>włączamy tryb “serializable”
82.| `select * from osoby;`
83.|| `update osoby set konto=1000 where nazw='Waga';`
84.|| `commit;`
85.| `select * from osoby;` <br>tym razem select podaje wartość taką samą jak poprzednio
86.| `update osoby set konto=2000 where nazw='Waga';` <br>próba aktualizacji nie powiedzie się, ze względu na modyfikację danych w konkurencyjnej transakcji
87.| `rollback;`

Tryb “serializable” – transakcja nie jest zerwana ze względu na rollback tej drugiej transakcji

| Lp | Okno 1 | Okno 2 |
|---:|:-------|:-------|
88.| `begin;` | `begin;`
89.| `set transaction isolation level serializable;`
90.| `select * from osoby;`
91.|| `update osoby set konto=1000 where nazw='Waga';`
92.| `update osoby set konto=2000 where nazw='Waga';` <br>polecenie czeka na zakończenie konkurencyjnej transakcji
93.|| `rollback;`
94.| `commit;`

Przykład użycia trybu „serializable” do poprawnej realizacji przelewu

| Lp | Okno 1 | Okno 2 |
|---:|:-------|:-------|
95.|| `update osoby set konto=1000 where nazw='Waga';`<br>przywrócenie danych w bazie do stanu pierwotnego
96.| `begin;` | `begin;`
97.| `set transaction isolation level serializable;` | `set transaction isolation level serializable;`
98.|| `select konto from osoby where nazw='Waga';`
99.| `select konto from osoby where nazw='Waga';`
100.| `update osoby set konto=konto-1000 where nazw='Waga';`
101.| `update osoby set konto=konto+1000 where nazw='Kos';`
102.| `commit;`
103.|| `update osoby set konto=konto-1000 where nazw='Waga';` <br>tutaj występuje błąd szeregowania (nasze zapytania przeplotły się z inną transakcją) – dalsze polecenia są ignorowane aż do końca transakcji
104.|| `update osoby set konto=konto+1000 where nazw='Kos';`
105.|| `commit;` <br>efektywnie jest to rollback ze względu na błąd szeregowania

Głównym powodem korzystania z trybu SERIALIZABLE jest zapewnienie spójnego obrazu danych przy tworzeniu raportów z dużych baz danych. Należy podkreślić, że do ochrony przed konkurencyjnymi transakcjami ten tryb NIE JEST konieczny, a czasem wręcz jest szkodliwy gdyż w razie kolizji powoduje zrywanie transakcji, co przy dużych i intensywnie użytkowanych bazach może wręcz uniemożliwiać pracę.
Podstawowym mechanizmem ochrony przed konkurencyjnymi transakcjami są blokady wierszy (begin ... select for update ... update ... commit).