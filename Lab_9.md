# Bazdy Danych / Lab_9 Zadania

Author: @https://github.com/Smendowski

---


#### **Zadanie 1.** Utworzyć tabelę categories tak, aby kolumna cid była kluczem głównym, zaś kolumna name miała unikalne i niepuste wartości.
```sql
c.dump

DROP TABLE "categories" cascade;
CREATE TABLE "categories" (
        "cid" int4 PRIMARY KEY,
        "name" character varying(30) UNIQUE NOT NULL,
        "cserid" int4
);
COPY "categories" FROM stdin;
```


#### **Zadanie 2.** Utworzyć tabelę subcategories tak, aby sid było kluczem głównym, kolumna cid była kluczem obcym związanym z categories.cid. Kolumna cid w tabeli subcategories ma być automatycznie aktualizaowana w przypadku zmiany cid w tabeli categories. Należy także zapewnić, by dana nazwa podkategorii nie występowała dwa razy w tej samej kategorii (w różnych kategoriach może wystąpić taka sama nazwa podkategorii). Skasowanie kategorii powinno spowodować automatyczne skasowanie podkategorii.
```sql
s.dump
DROP TABLE "subcategories";
CREATE TABLE "subcategories" (
        "sid" int4 PRIMARY KEY,
        "name" character varying(30),
        "cid" int4,
        "sserid" int4,
        foreign key (cid) references categories(cid) on update cascade on delete cascade,
		unique(cid,name),
        unique(cid,sid)
);
COPY "subcategories" FROM stdin
```

#### **Zadanie 3.** Utworzyć tabelę movies_list, w której mid jest kluczem głównym, a pola cid i sid zawierają poprawne wartości identyfikatorów kategorii i podkategorii, przy czym należy zapewnić, że wskazana podkategoria należy do wskazanej kategorii. Domyślną wartością dla tych pól ma być 1. W przypadku usunięcia kategorii bądź podkategorii w polach cid i sid mają być automatycznie wstawiane wartości domyślne. W przypadku aktualizacji wartości identyfikatorów kategorii lub podkategorii w tabelach categories oraz subcategories wartości pól cid oraz sid w movies_list mają być automatycznie aktualizowane.

```sql
m.dump
DROP TABLE "movies_list";
CREATE TABLE "movies_list" (
        "mid" int4 PRIMARY KEY,
        "sysflags" int4,
        "lenbin" int4,
        "stream" int4,
        "compress" character varying(16),
        "dimensions" character varying(16),
        "fps" int4,
        "cid" int4 DEFAULT 1,
        "sid" int4 DEFAULT 1,
        "author" character varying(256),
        "lenmsec" int4,
        "filename" character varying(256),
        "tokenrate" int4,
        "bucketsize" int4,
        "peakbitrate" int4,
        "license" character varying(256),
        "expiry" date,
        "mserid" int4,
        foreign key (cid,sid) references subcategories(cid,sid) on update cascade on delete set default
);
```


#### **Zadanie 4.** Utworzyć tabelę osoby zawierającą nazwisko, imię i nr. pesel. Nazwisko i imię nie mogą być puste. PESEL musi być poprawny. Algorytm sprawdzania poprawności nr. PESEL znajduje się na stronie: http://wipos.p.lodz.pl/zylla/ut/pesel.html 

```sql
CREATE TABLE osoby (nazwiko varchar(30) NOT NULL, imie varchar(30) NOT NULL, pesel varchar(11));

create or replace function check_pesel () returns trigger AS '
    declare
		wagi integer ARRAY[4];
		i int4;
		wk int4;
		k int4;
	begin
	wagi[1] := 1;
	wagi[2] := 3;
	wagi[3] := 7;
	wagi[4] := 9;
	wk := 0;

	if length(NEW.pesel) != 11 then
		raise exception ''Zla dlugosc numeru pesel! Ma byc 11 cyfr'';
      	return NULL;
    else
    	for i in 0 .. 9 loop
    		wk = (wk + substr(NEW.pesel,i+1,1)::integer * wagi[i%4 + 1]) % 10;
     	end loop;
     	k = (10 - wk) % 10;
     	--raise notice ''Wartosc kontrolna to %'', k;
     	if (substr(NEW.pesel,11,1)::integer = k) then
		raise notice ''Insert przebiegl pomyslnie'';
     		return NEW;
     	else
     		raise notice ''Nie udany insert, sprawdz PESEL jeszcze raz'';
		return NULL;
     	end if;
    end if;
    end;
    ' language 'plpgsql';

    CREATE TRIGGER check_pesel_t before insert on osoby for each row execute procedure check_pesel();
```